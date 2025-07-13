[Main](../README.md)
---

# 🧪 Kubernetes Final Lab Project — Full Lifecycle Practice

Namespace: `final-lab`

---

## ✅ Step 1: Initialize Namespace & Secrets

```bash
kubectl create ns final-lab
kubectl create secret generic pg-secret \
  --from-literal=PG_USER=admin \
  --from-literal=PG_PASSWORD=adminpass \
  --namespace=final-lab
```

---

Or create yaml file for appliying
```yaml
# postgres-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: final-lab
type: Opaque
data:
  POSTGRES_PASSWORD: cGFzc3dvcmQ=  # base64 of "password"
```
## 🐘 Step 2: Deploy PostgreSQL

📄 `postgres.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pg-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/postgres"
  storageClassName: manual
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pg-pvc
  namespace: final-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: final-lab
spec:
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
        image: postgres:14
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: PG_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: PG_PASSWORD
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: pgdata
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: pgdata
        persistentVolumeClaim:
          claimName: pg-pvc
```

```bash
kubectl apply -f postgres.yaml
```

---

## 🔥 Step 3: Deploy Flask App (Initial Version)

📄 `app.py`

```python
from flask import Flask, jsonify
import os, psycopg2

app = Flask(__name__)

@app.route("/healthz")
def health():
    return "OK", 200

@app.route("/write")
def write():
    try:
        conn = psycopg2.connect(
            dbname="postgres",
            user=os.environ['PG_USER'],
            password=os.environ['PG_PASSWORD'],
            host=os.environ['PG_HOST'],
            port="5432"
        )
        cur = conn.cursor()
        cur.execute("CREATE TABLE IF NOT EXISTS hits (ts TIMESTAMP);")
        cur.execute("INSERT INTO hits VALUES (NOW());")
        conn.commit()
        cur.close()
        conn.close()
        return "Written to DB", 200
    except Exception as e:
        return str(e), 500

@app.route("/read")
def read():
    try:
        conn = psycopg2.connect(
            dbname="postgres",
            user=os.environ['PG_USER'],
            password=os.environ['PG_PASSWORD'],
            host=os.environ['PG_HOST'],
            port="5432"
        )
        cur = conn.cursor()
        cur.execute("SELECT * FROM hits ORDER BY ts DESC LIMIT 5;")
        result = cur.fetchall()
        cur.close()
        conn.close()
        return jsonify(result), 200
    except Exception as e:
        return str(e), 500
```

📄 `Dockerfile`

```Dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py .
RUN pip install flask psycopg2-binary
CMD ["python", "app.py"]
```

🔨 Build & Push:

```bash
docker build -t <youruser>/final-flask:v1 .
docker push <youruser>/final-flask:v1
```

📄 `flask-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask
  namespace: final-lab
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask
  template:
    metadata:
      labels:
        app: flask
    spec:
      containers:
      - name: flask
        image: <youruser>/final-flask:v1
        ports:
        - containerPort: 5000
        env:
        - name: PG_USER
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: PG_USER
        - name: PG_PASSWORD
          valueFrom:
            secretKeyRef:
              name: pg-secret
              key: PG_PASSWORD
        - name: PG_HOST
          value: postgres.final-lab.svc.cluster.local
```

```bash
kubectl apply -f flask-deploy.yaml
kubectl port-forward deployment/flask 5000:5000 -n final-lab
```

✅ Check: `curl localhost:5000/write` and `curl localhost:5000/read`


Service for Flask

```yaml
# flask-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: flask
  namespace: final-lab
spec:
  selector:
    app: flask
  ports:
  - port: 5000
    targetPort: 5000
```

Verify Deployment
```bash
kubectl get all -n final-lab
kubectl port-forward svc/flask 5000:5000 -n final-lab
```
Visit:
```bash
http://localhost:5000/health
http://localhost:5000/write
http://localhost:5000/read
```


## ✅ Step 4: Add Probes, Sidecar & Init Container

Update Flask deployment YAML to include:

### Probes

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: {{ .Values.containerPort }}
  initialDelaySeconds: 5
  periodSeconds: 10
readinessProbe:
  httpGet:
    path: /readyz
    port: {{ .Values.containerPort }}
  initialDelaySeconds: 5
  periodSeconds: 5
startupProbe:
  httpGet:
    path: /readyz
    port: {{ .Values.containerPort }}
  failureThreshold: 10
  periodSeconds: 2
```

### Init Container

```yaml
initContainers:
- name: wait-db
  image: busybox
  command: ['sh', '-c', 'until nc -z postgres.final-project.svc.cluster.local 5432; do sleep 2; done']
```

### Sidecar

```yaml
- name: sidecar-logger
  image: busybox
  command: ['sh', '-c', 'tail -F /logs/app.log']
  volumeMounts:
    - name: log-dir
      mountPath: /logs
```

Add shared volume:

```yaml
volumes:
- name: log-dir
  emptyDir: {}
```

---

## ✅ Step 5: Add Affinity, NodeSelector, Taints & Tolerations

Add to pod spec:

```yaml
nodeSelector:
  node-role.kubernetes.io/worker: ""

affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: node-role.kubernetes.io/worker
              operator: Exists

podAntiAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    podAffinityTerm:
      labelSelector:
        matchExpressions:
        - key: app
          operator: In
          values:
          - postgres
      topologyKey: kubernetes.io/hostname

tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "flask"
  effect: "NoSchedule"
```

---

## ✅ Step 6: Add Resource Limits

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "250m"
    memory: "256Mi"
```

---

## ✅ Step 7: ConfigMap and Secret

### config.yaml

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: flask-config
  namespace: final-project
data:
  ENV: production
```

### secret.yaml

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: flask-secret
  namespace: final-project
type: Opaque
data:
  API_KEY: c2VjcmV0
```

Mount in Deployment:

```yaml
envFrom:
- configMapRef:
    name: flask-config
- secretRef:
    name: flask-secret
```

---

## ✅ Step 8: Helm Chart Creation

```bash
helm create flask-chart
```

Clean up chart:

* keep only `deployment.yaml`, `service.yaml`, `ingress.yaml`
* move values to `values.yaml`

---

## ✅ Step 9: Write values.yaml

```yaml
image:
  repository: <dockerhub_user>/flask-app
  tag: v1
  pullPolicy: IfNotPresent

imagePullSecrets:
  - name: regcred

containerPort: 5000

resources:
  limits:
    cpu: 250m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi

env:
  enabled: true
  values:
    - name: PG_HOST
      value: postgres.final-project.svc.cluster.local
    - name: REDIS_HOST
      value: redis.final-project.svc.cluster.local

configMaps:
  enabled: true
  list:
    - flask-config

secrets:
  enabled: true
  list:
    - flask-secret

probes:
  enabled: true
  liveness:
    path: /healthz
    port: 5000
  readiness:
    path: /readyz
    port: 5000

initContainers:
  enabled: true

sidecar:
  enabled: true

affinity:
  enabled: true
  rules:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
            - key: node-role.kubernetes.io/worker
              operator: Exists
```

---

## ✅ Step 10: Package Helm and Push to GitLab

```bash
helm package flask-chart
curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
     --upload-file flask-chart-0.1.0.tgz \
     https://gitlab.com/api/v4/projects/<project_id>/packages/helm/stable
```

---

## ✅ Step 11: Add GitLab Repo and Install from Chart

```bash
helm repo add my-gitlab https://gitlab.com/api/v4/projects/<project_id>/packages/helm/stable
helm repo update
helm install flask-app my-gitlab/flask-chart -n final-project
```

---

## ✅ Step 12: Upgrade and Rollback

```bash
# Update app version in values.yaml
helm upgrade flask-app my-gitlab/flask-chart -n final-project --set image.tag=v2

# Rollback
helm rollback flask-app 1 -n final-project
```

---

## ✅ Step 13: Check History and Cleanup

```bash
helm history flask-app -n final-project
helm uninstall flask-app -n final-project
```


[Main](../README.md)
---