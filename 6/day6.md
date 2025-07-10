
## Dockerfile

```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
EXPOSE 5000
CMD ["python", "app.py"]
```

```bash
docker build -t <your-username>/flask-pg-app:latest .

docker login
docker push <your-username>/flask-pg-app:latest
```

## Kubernetes Manifests

🧩 PostgreSQL Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_DB
          value: "usersdb"
        - name: POSTGRES_USER
          value: "user"
        - name: POSTGRES_PASSWORD
          value: "password"
        ports:
        - containerPort: 5432
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432

```


🧩 Flask App Deployment + Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-app
        image: <your-username>/flask-pg-app:latest
        ports:
        - containerPort: 5000
        env:
        - name: DB_HOST
          value: "postgres"
        - name: DB_PORT
          value: "5432"
        - name: DB_NAME
          value: "usersdb"
        - name: DB_USER
          value: "user"
        - name: DB_PASSWORD
          value: "password"
---
apiVersion: v1
kind: Service
metadata:
  name: flask-app
spec:
  selector:
    app: flask-app
  ports:
  - port: 80
    targetPort: 5000

```


🔐 NetworkPolicy (only app can access PostgreSQL)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: postgres-policy
spec:
  podSelector:
    matchLabels:
      app: postgres
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: flask-app
    ports:
    - protocol: TCP
      port: 5432
  policyTypes:
  - Ingress

```


🌐 Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: flask.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: flask-app
            port:
              number: 80
```

🛠️ Make sure to add this to /etc/hosts if you’re running locally:

127.0.0.1 flask.local



🧪 Commands to Check Deployments and Policies
```bash
kubectl get pods
kubectl get svc
kubectl get deployments
kubectl get ingress
kubectl describe ingress flask-ingress
kubectl get netpol
kubectl describe netpol postgres-policy
kubectl logs -l app=flask-app
kubectl exec -it <flask-pod> -- curl postgres:5432

```


✅ Test Your App
```bash
curl -X POST flask.local/add -H "Content-Type: application/json" -d '{"name": "Zafar"}'
curl flask.local/users
```

🏁 Summary for Students

| Task |	Complete? |
|-----|-----|
| Build and push Docker image |	✅ |
| Deploy PostgreSQL with credentials |	✅ |
| Deploy Flask app using ENV vars |	✅ |
| Apply NetworkPolicy to restrict access |	✅ |
| Create Services and Ingress |	✅ |
| Test with curl and kubectl logs/exec |	✅ |

