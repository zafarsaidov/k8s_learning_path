[Main](../README.md)
---

# Ingress Controller & Rules


## 🧱 Ingress Structure
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```


## ✅ Step 1: Deploy NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```
Wait for controller pod:
```bash
kubectl get pods -n ingress-nginx
```
Check external IP:
```bash
kubectl get svc -n ingress-nginx
```

## ✅ Step 2: Deploy Sample Services

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web1
  template:
    metadata:
      labels:
        app: web1
    spec:
      containers:
      - name: web1
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Web1"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: web1
spec:
  selector:
    app: web1
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web2
  template:
    metadata:
      labels:
        app: web2
    spec:
      containers:
      - name: web2
        image: hashicorp/http-echo
        args:
        - "-text=Hello from Web2"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: web2
spec:
  selector:
    app: web2
  ports:
  - port: 80
    targetPort: 5678
```
Apply:
```bash
kubectl apply -f services.yaml
```

## ✅ Step 3: Create an Ingress Resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /web1
        pathType: Prefix
        backend:
          service:
            name: web1
            port:
              number: 80
      - path: /web2
        pathType: Prefix
        backend:
          service:
            name: web2
            port:
              number: 80
```
Apply:
```bash
kubectl apply -f ingress.yaml
```

## ✅ Step 4: Test Routing
Get the Ingress Controller external IP:
```bash
kubectl get svc ingress-nginx-controller -n ingress-nginx
```
Test from browser or curl:
```bash
curl http://<EXTERNAL-IP>/web1
curl http://<EXTERNAL-IP>/web2
```

## 🧪 Bonus Exercises
* Add TLS with a self-signed certificate using Ingress.tls.
* Route based on host: instead of path: (e.g., web1.example.com).
* Test error handling when backend service is down.



## 🧼 Cleanup
```bash
kubectl delete -f ingress.yaml
kubectl delete -f services.yaml
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
```

[Documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)


[Main](../README.md)
---