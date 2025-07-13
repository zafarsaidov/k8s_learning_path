[Main](../README.md)
---

# 🧪 Kubernetes Final Lab Project — Full Lifecycle Practice

Namespace: `final-lab`

---

## 🏗 **Step 1: Build and Push App Image**

### ✅ Task

1. **Flask App (`app.py`)**

```python
from flask import Flask, request, jsonify
import os
import psycopg2
import redis

app = Flask(__name__)

# PostgreSQL setup
DB_HOST = os.getenv("POSTGRES_HOST", "localhost")
DB_NAME = os.getenv("POSTGRES_DB", "mydb")
DB_USER = os.getenv("POSTGRES_USER", "myuser")
DB_PASS = os.getenv("POSTGRES_PASSWORD", "mypassword")

conn = psycopg2.connect(
    host=DB_HOST,
    database=DB_NAME,
    user=DB_USER,
    password=DB_PASS
)
cur = conn.cursor()

# Redis setup
REDIS_HOST = os.getenv("REDIS_HOST", "localhost")
redis_client = redis.StrictRedis(host=REDIS_HOST, port=6379, decode_responses=True)

@app.route('/')
def home():
    return "Welcome to the Flask App!"

@app.route('/db/write', methods=['POST'])
def write_db():
    data = request.json.get("data")
    cur.execute("INSERT INTO test_table (data) VALUES (%s);", (data,))
    conn.commit()
    return jsonify({"message": "Data inserted to DB"}), 201

@app.route('/db/read')
def read_db():
    cur.execute("SELECT data FROM test_table;")
    rows = cur.fetchall()
    return jsonify(rows)

@app.route('/cache/write', methods=['POST'])
def write_cache():
    key = request.json.get("key")
    value = request.json.get("value")
    redis_client.set(key, value)
    return jsonify({"message": f"Cached {key}"}), 201

@app.route('/cache/read')
def read_cache():
    key = request.args.get("key")
    value = redis_client.get(key)
    return jsonify({"value": value})
```

You should ensure that the test_table exists in the DB with `CREATE TABLE test_table (id SERIAL PRIMARY KEY, data TEXT);`


2. **`requirements.txt`**

```
flask
psycopg2-binary
redis
```

3. **Dockerfile**

```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

4. **Build & Push**

```bash
docker build -t your-dockerhub-username/flask-app:latest .
docker login
docker push your-dockerhub-username/flask-app:latest
```

5. **Test locally**

```bash
docker run -p 5000:5000 your-dockerhub-username/flask-app:latest
curl localhost:5000/
```

---

## 🚀 **Step 2: Deploy Basic App**

### ✅ PostgreSQL

```yaml
# postgresql-deployment.yaml
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
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          value: "mypassword"
        - name: POSTGRES_USER
          value: "myuser"
        - name: POSTGRES_PASSWORD
          value: "mypassword"
        - name: POSTGRES_DB
          value: "mydb"
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: postgres-storage
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes: ["ReadWriteOnce"]
  resources:
    requests:
      storage: 1Gi
```

### ✅ Redis

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis:7
---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  selector:
    app: redis
  ports:
  - port: 6379
```

### ✅ Flask App Deployment

```bash
kubectl create secret generic db-secret \
  --from-literal=PG_USER=admin \
  --from-literal=PG_PASSWORD=password \
  --namespace=final-lab
```

---

Or create yaml file for appliying
```yaml
# postgres-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: final-lab
type: Opaque
data:
  POSTGRES_PASSWORD: cGFzc3dvcmQ=  # base64 of "password"
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
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
        image: your-dockerhub-username/flask-app:latest
        ports:
        - containerPort: 5000
        env:
        - name: POSTGRES_HOST
          value: "postgres"
        - name: POSTGRES_DB
          value: "mydb"
        - name: POSTGRES_USER
          value: "myuser"
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
            name: db-secret
            key: POSTGRES_PASSWORD
        - name: REDIS_HOST
          value: "redis"

---
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask
  ports:
  - port: 80
    targetPort: 5000
    protocol: TCP
    name: http
  type: ClusterIP
```

### ✅ Verify

```bash
kubectl port-forward svc/flask-service 8080:80
curl localhost:8080
```

---

## 🔍 **Step 3: Health Checks & Init Containers**

### ✅ Add to Flask Deployment

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 5
  periodSeconds: 10
livenessProbe:
  httpGet:
    path: /
    port: 5000
  initialDelaySeconds: 10
  periodSeconds: 10
```

### ✅ Init Containers Example

```yaml
initContainers:
- name: wait-for-postgres
  image: busybox
  command: ['sh', '-c', 'until nc -z postgres 5432; do echo waiting for postgres; sleep 2; done;']
- name: wait-for-redis
  image: busybox
  command: ['sh', '-c', 'until nc -z redis 6379; do echo waiting for redis; sleep 2; done;']
```

---

## 🗃 **Step 4: Config via ConfigMap & Secrets**

### ✅ Create ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  REDIS_HOST: redis
```

### ✅ Create Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  POSTGRES_PASSWORD: bXlwYXNzd29yZA== # "mypassword"
```

### ✅ Inject to Flask Deployment

```yaml
envFrom:
- configMapRef:
    name: app-config
- secretRef:
    name: db-secret
```

### ✅ Verify

```bash
kubectl describe pod flask-app-xyz
```

---

## 📦 **Step 5: Logging Sidecar**

### ✅ Modify Flask Deployment

```yaml
volumes:
- name: log-volume
  emptyDir: {}

containers:
- name: flask
  ...
  volumeMounts:
  - name: log-volume
    mountPath: /logs

- name: log-sidecar
  image: busybox
  command: ['sh', '-c', 'tail -F /logs/app.log']
  volumeMounts:
  - name: log-volume
    mountPath: /logs
```

---

## ⚖ **Step 6: Affinity and Tolerations**

### ✅ Affinity

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: node-role
          operator: In
          values:
          - app

  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
    - labelSelector:
        matchLabels:
          app: redis
      topologyKey: "kubernetes.io/hostname"
```

### ✅ Tolerations
✅ Taint a Node
```bash
kubectl taint nodes <node-name> dedicated=flask:NoSchedule
```
Your pod will now only be scheduled on this node if it has a matching toleration.
```yaml
tolerations:
- key: "dedicated"
  operator: "Equal"
  value: "flask"
  effect: "NoSchedule"
```

---

## 📈 **Step 7: Resource Limits**

### ✅ Add to Pod Specs

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

### ✅ LimitRange & ResourceQuota

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limits
spec:
  limits:
  - default:
      cpu: 500m
      memory: 512Mi
    defaultRequest:
      cpu: 100m
      memory: 128Mi
    type: Container
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: app-quota
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

---

## 🔐 **Step 8: RBAC & ServiceAccount**

### ✅ ServiceAccount + Role + RoleBinding

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flask-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: flask-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: flask-rb
subjects:
- kind: ServiceAccount
  name: flask-sa
roleRef:
  kind: Role
  name: flask-role
  apiGroup: rbac.authorization.k8s.io
```

### ✅ Attach to Deployment

```yaml
serviceAccountName: flask-sa
```

### ✅ Verify

```bash
kubectl auth can-i get pods --as system:serviceaccount:default:flask-sa
```

---

## 🔒 **Step 9: PodSecurityContext**

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  readOnlyRootFilesystem: true
  capabilities:
    drop: ["ALL"]
```

### ✅ Verify

```bash
kubectl exec -it flask-app-xxx -- id
```

---

## 🧠 **Step 10: PostgreSQL StatefulSet**

### ✅ StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
spec:
  serviceName: "postgres"
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
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: POSTGRES_PASSWORD
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
```

### ✅ Headless Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - port: 5432
```

---

## ⏰ **Step 11: CronJob**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hit-db
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: curl
            image: curlimages/curl
            args: ["http://flask-service/db"]
          restartPolicy: OnFailure
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  concurrencyPolicy: Forbid
```

---

## 🎛 **Step 12: Ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: flask-ingress
spec:
  rules:
  - host: flask.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: flask-service
            port:
              number: 80
```

> ✅ For local dev: Add `127.0.0.1 flask.local` to `/etc/hosts`

---

## 📦 **Step 13: Helm Chart**

### ✅ Structure

```
flask-chart/
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
├── values.yaml
├── Chart.yaml
```

templates/deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "flask-chart.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "flask-chart.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "flask-chart.name" . }}
    spec:
      containers:
      - name: flask
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        ports:
        - containerPort: 5000
        env:
        - name: POSTGRES_HOST
          value: {{ .Values.postgres.host | quote }}
        - name: POSTGRES_DB
          value: {{ .Values.postgres.db | quote }}
        - name: POSTGRES_USER
          value: {{ .Values.postgres.user | quote }}
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: {{ .Values.postgres.secretName }}
              key: POSTGRES_PASSWORD
        - name: REDIS_HOST
          value: {{ .Values.redis.host | quote }}
```

templates/service.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "flask-chart.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ include "flask-chart.name" . }}
  ports:
  - port: {{ .Values.service.port }}
    targetPort: 5000
```
templates/ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "flask-chart.fullname" . }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ include "flask-chart.fullname" . }}
            port:
              number: {{ .Values.service.port }}
```
templates/secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.postgres.secretName }}
type: Opaque
stringData:
  POSTGRES_PASSWORD: {{ .Values.postgres.password }}
```


### ✅ Sample `values.yaml`

```yaml
replicaCount: 1

image:
  repository: your-dockerhub/flask-app
  tag: latest

service:
  type: ClusterIP
  port: 80

postgres:
  host: postgres
  db: mydb
  user: myuser
  password: mypassword
  secretName: db-secret

redis:
  host: redis

ingress:
  host: flask.local
```

### ✅ Verify

```bash
helm install flask-app ./flask-chart
```

---

## 🔄 **Step 14: Upgrade & Rollback**

### ✅ Change version and upgrade

```bash
# Change version in Chart.yaml or values.yaml
helm upgrade flask-app ./flask-chart
```

### ✅ Rollback

```bash
helm rollback flask-app 1
```

---

## 📁 **Step 15: Push to GitLab Registry**

### ✅ `.gitlab-ci.yml`

```yaml
stages:
  - package
  - deploy

package_chart:
  stage: package
  script:
    - helm package ./flask-chart
    - helm push flask-chart-0.1.0.tgz oci://registry.gitlab.com/yourgroup/yourproject

deploy:
  stage: deploy
  script:
    - helm repo add myrepo oci://registry.gitlab.com/yourgroup/yourproject
    - helm install flask-app myrepo/flask-chart --version 0.1.0
```

---


[Main](../README.md)
---