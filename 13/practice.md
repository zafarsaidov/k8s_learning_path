[Main](../README.md)
---

# Kubernetes Practice: Step-by-Step Full Application Deployment with Advanced Features

Namespace used: `kube-lab`

---

## ✅ Prerequisites (only once)

```bash
kubectl create ns kube-lab
```

---

## 🏡 Step 1: Deploy PostgreSQL and Redis with PVCs

### 1.1 Create StorageClass (local-path if not already present)

```yaml
# storageclass-local.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```bash
kubectl apply -f storageclass-local.yaml
```

### 1.2 Create PersistentVolume and PersistentVolumeClaim for Postgres

```yaml
# pv-postgres.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/postgres"
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
  namespace: kube-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```

### 1.3 Deploy PostgreSQL

```yaml
# postgres.yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
  namespace: kube-lab
type: Opaque
data:
  POSTGRES_PASSWORD: cGFzc3dvcmQ=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
  namespace: kube-lab
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
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: postgres-storage
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

### 1.4 Deploy Redis with PVC

```yaml
# redis.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv
spec:
  capacity:
    storage: 512Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/redis"
  storageClassName: local-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: redis-pvc
  namespace: kube-lab
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi
  storageClassName: local-storage
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: kube-lab
spec:
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
        ports:
        - containerPort: 6379
        volumeMounts:
        - mountPath: /data
          name: redis-storage
      volumes:
      - name: redis-storage
        persistentVolumeClaim:
          claimName: redis-pvc
```

### ✅ Verification

```bash
kubectl get all -n kube-lab
kubectl get pvc,pv -n kube-lab
kubectl logs deployment/postgres -n kube-lab
kubectl logs deployment/redis -n kube-lab
```

---

## ✅ Next Step

Now that Postgres and Redis are deployed and persistent, we will move to creating the **Flask app** with:

* `/healthz`, `/readyz`, `/db-write`, `/db-read`, `/cache`
* ConfigMap and Secret usage
* Dockerfile for image
* Probes, init, sidecar, and more

## 🧪 Step 2: Flask App - Dockerfile, Code, Deployment v1

We’ll now develop a basic version of the Flask app that integrates with Postgres and Redis but starts with a single container and a few endpoints.

---

### 2.1 Flask App Code

📁 `app.py`

```python
from flask import Flask, jsonify
import psycopg2, redis, os, time

app = Flask(__name__)
start_time = time.time()

@app.route("/healthz")
def healthz():
    return "OK", 200

@app.route("/readyz")
def readyz():
    if time.time() - start_time > 10:
        return "Ready", 200
    return "Not Ready", 503

@app.route("/db-write")
def db_write():
    try:
        conn = psycopg2.connect(
            dbname=os.getenv("PG_DB"),
            user=os.getenv("PG_USER"),
            password=os.getenv("PG_PASSWORD"),
            host=os.getenv("PG_HOST"),
            port=os.getenv("PG_PORT", "5432")
        )
        cur = conn.cursor()
        cur.execute("CREATE TABLE IF NOT EXISTS logs (ts TIMESTAMP);")
        cur.execute("INSERT INTO logs VALUES (NOW());")
        conn.commit()
        cur.close()
        conn.close()
        return "Inserted timestamp", 200
    except Exception as e:
        return str(e), 500

@app.route("/db-read")
def db_read():
    try:
        conn = psycopg2.connect(
            dbname=os.getenv("PG_DB"),
            user=os.getenv("PG_USER"),
            password=os.getenv("PG_PASSWORD"),
            host=os.getenv("PG_HOST"),
            port=os.getenv("PG_PORT", "5432")
        )
        cur = conn.cursor()
        cur.execute("SELECT * FROM logs ORDER BY ts DESC LIMIT 5;")
        rows = cur.fetchall()
        cur.close()
        conn.close()
        return jsonify(rows), 200
    except Exception as e:
        return str(e), 500

@app.route("/cache")
def cache():
    try:
        r = redis.Redis(host=os.getenv("REDIS_HOST"), port=6379)
        r.incr('hits')
        return f"Cache hits: {r.get('hits').decode()}"
    except Exception as e:
        return str(e), 500
```

📁 `requirements.txt`

```
flask
psycopg2-binary
redis
```

---

### 2.2 Dockerfile

```Dockerfile
# Dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

---

### 2.3 Build and Push Image

```bash
docker build -t <your_dockerhub_user>/flask-app:v1 .
docker push <your_dockerhub_user>/flask-app:v1
```

---

### 2.4 Deploy Initial Flask App to Kubernetes

```yaml
# flask-app-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
  namespace: kube-lab
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
        image: <your_dockerhub_user>/flask-app:v1
        ports:
        - containerPort: 5000
        env:
        - name: PG_DB
          value: mydb
        - name: PG_USER
          value: myuser
        - name: PG_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: POSTGRES_PASSWORD
        - name: PG_HOST
          value: postgres.kube-lab.svc.cluster.local
        - name: REDIS_HOST
          value: redis.kube-lab.svc.cluster.local
```

---

### ✅ Verification

```bash
kubectl apply -f flask-app-deploy.yaml
kubectl get pods -n kube-lab
kubectl port-forward deployment/flask-app 5000:5000 -n kube-lab
```

Visit:

* `http://localhost:5000/healthz`
* `http://localhost:5000/readyz`
* `http://localhost:5000/db-write`
* `http://localhost:5000/db-read`
* `http://localhost:5000/cache`

---

## 🔄 Step 3: Add Probes, Init Container, and Sidecar

We will now enhance our Flask deployment gradually with Kubernetes features.

---

### 3.1 Add Liveness, Readiness, and Startup Probes

Update `flask-app-deploy.yaml` to include:

```yaml
        livenessProbe:
          httpGet:
            path: /healthz
            port: 5000
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /readyz
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
        startupProbe:
          httpGet:
            path: /readyz
            port: 5000
          failureThreshold: 30
          periodSeconds: 1
```

Apply:

```bash
kubectl apply -f flask-app-deploy.yaml
kubectl describe pod -n kube-lab
```

---

### 3.2 Add Init Container to Check Postgres and Redis

Update the Deployment to include:

```yaml
      initContainers:
      - name: check-db
        image: busybox
        command: ['sh', '-c', 'until nc -z postgres.kube-lab.svc.cluster.local 5432; do echo waiting for postgres; sleep 2; done']
      - name: check-redis
        image: busybox
        command: ['sh', '-c', 'until nc -z redis.kube-lab.svc.cluster.local 6379; do echo waiting for redis; sleep 2; done']
```

Apply and test pod startup:

```bash
kubectl delete pod -l app=flask -n kube-lab
kubectl get pods -n kube-lab
```

---

### 3.3 Add Sidecar for Logging (Writes logs to shared volume)

Update pod spec:

```yaml
        volumeMounts:
        - name: shared-logs
          mountPath: /logs
      volumes:
      - name: shared-logs
        emptyDir: {}
      containers:
      - name: logger
        image: busybox
        command: ["sh", "-c", "tail -F /logs/app.log"]
        volumeMounts:
        - name: shared-logs
          mountPath: /logs
```

Update Flask app to write logs to `/logs/app.log`.
Rebuild and push Docker image:

```bash
docker build -t <your_dockerhub_user>/flask-app:v2 .
docker push <your_dockerhub_user>/flask-app:v2
```

Update deployment image to `v2`.


In app.py, modify your Flask app to write logs into a file at /logs/app.log (assuming this path is mounted with emptyDir and shared with a sidecar):

```python
import logging

# Setup logging
log_file = '/logs/app.log'
logging.basicConfig(filename=log_file, level=logging.INFO, format='%(asctime)s - %(message)s')

@app.route("/db-write")
def db_write():
    try:
        # existing db connection and insert logic...
        logging.info("Inserted timestamp into DB")
        return "Inserted timestamp", 200
    except Exception as e:
        logging.error(f"DB Write Error: {e}")
        return str(e), 500
```

---

✅ You now have a resilient app with:

* Health and readiness gates
* Controlled startup
* Logging via sidecar

## ⚙️ Step 4: Apply Scheduling Policies and Resource Limits

Now we’ll use Kubernetes scheduling strategies to control how and where our application runs.

---

### 4.1 Apply Node Affinity (Require node with label)

Label a node:

```bash
kubectl label nodes <node-name> zone=lab
```

Update Deployment:

```yaml
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: zone
                operator: In
                values:
                - lab
```

---

### 4.2 Add Pod Anti-Affinity

Ensure Redis and Flask do not schedule together:

```yaml
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - redis
            topologyKey: "kubernetes.io/hostname"
```

---

### 4.3 Taints and Tolerations

Add a taint to a node:

```bash
kubectl taint nodes <node-name> dedicated=flask:NoSchedule
```

Update deployment:

```yaml
      tolerations:
      - key: "dedicated"
        operator: "Equal"
        value: "flask"
        effect: "NoSchedule"
```

---

### 4.4 Apply Resource Requests and Limits

Update container spec:

```yaml
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"
```

---

### ✅ Verification

```bash
kubectl describe pod -n kube-lab
kubectl get nodes --show-labels
kubectl get pods -n kube-lab -o wide
```

Check that pods obey placement and limits are visible in descriptions.

---

## 🔐 Step 5: RBAC, ServiceAccounts & Scoped Access

Let’s secure access to our application with Role-Based Access Control and custom kubeconfigs.

---

### 5.1 Create ServiceAccount

```bash
kubectl create serviceaccount flask-sa -n kube-lab
```

Update your `flask-app-deploy.yaml`:

```yaml
      serviceAccountName: flask-sa
```

---

### 5.2 Create Role and RoleBinding

```yaml
# flask-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: flask-reader
  namespace: kube-lab
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: flask-binding
  namespace: kube-lab
subjects:
- kind: ServiceAccount
  name: flask-sa
  namespace: kube-lab
roleRef:
  kind: Role
  name: flask-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply it:

```bash
kubectl apply -f flask-role.yaml
```

Verify:

```bash
kubectl auth can-i list pods --as=system:serviceaccount:kube-lab:flask-sa -n kube-lab
```

---

### 5.3 Create Kubeconfig for ServiceAccount

```bash
SECRET=$(kubectl get sa flask-sa -n kube-lab -o jsonpath='{.secrets[0].name}')
TOKEN=$(kubectl get secret $SECRET -n kube-lab -o jsonpath='{.data.token}' | base64 -d)
CA=$(kubectl get secret $SECRET -n kube-lab -o jsonpath='{.data.ca\.crt}')
SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

cat <<EOF > flask-sa-kubeconfig
apiVersion: v1
kind: Config
clusters:
- name: lab
  cluster:
    certificate-authority-data: $CA
    server: $SERVER
contexts:
- name: flask-sa-context
  context:
    cluster: lab
    user: flask-sa
users:
- name: flask-sa
  user:
    token: $TOKEN
current-context: flask-sa-context
EOF
```

Test with:

```bash
KUBECONFIG=flask-sa-kubeconfig kubectl get pods -n kube-lab
```

---

✅ You’ve successfully scoped access for the Flask app via a service account with limited RBAC.

## 🔐 Step 6: PodSecurityContext and Security Best Practices

In this step, we enhance the security posture of our pods using `securityContext` and pod-level controls.

---

### 6.1 Add `securityContext` to Flask Container

Update your container spec:

```yaml
        securityContext:
          runAsUser: 1000
          runAsGroup: 3000
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

✅ Explanation:

* `runAsUser` ensures your app doesn't run as root.
* `readOnlyRootFilesystem` improves immutability and security.
* `drop ALL` reduces kernel capabilities.

---

### 6.2 Add Pod-Level `securityContext`

```yaml
      securityContext:
        fsGroup: 2000
```

✅ Explanation:

* `fsGroup` sets a group ID for shared volume files like logs or DB.


---

### ✅ Verification

Check that container is not root:

```bash
kubectl exec -n kube-lab -it <flask-pod> -- id
```

Check volume permissions:

```bash
kubectl exec -n kube-lab -it <flask-pod> -- ls -l /logs
```

Describe pod for securityContext validation:

```bash
kubectl describe pod <flask-pod> -n kube-lab
```

---

✅ With this, your Flask app runs with restricted privileges and stronger filesystem boundaries.

## ⏰ Step 7: CronJobs, PriorityClass, Preemption & Cleanup

Now we’ll explore automation and workload prioritization strategies.

---

### 7.1 Create a CronJob to Periodically Query `/db-read`

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-query-job
  namespace: kube-lab
spec:
  schedule: "*/5 * * * *"  # Every 5 minutes
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: curl-db
            image: curlimages/curl
            args: ["http://flask-app.kube-lab.svc.cluster.local:5000/db-read"]
          restartPolicy: OnFailure
```

✅ Apply and check logs:

```bash
kubectl apply -f db-query-cronjob.yaml
kubectl get cronjobs -n kube-lab
kubectl get jobs -n kube-lab
kubectl logs job/<job-name> -n kube-lab
```

---

### 7.2 Create PriorityClasses

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
preemptionPolicy: PreemptLowerPriority
globalDefault: false
description: "High priority for critical jobs"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
preemptionPolicy: Never
globalDefault: false
```

Apply and annotate pods:

```yaml
  priorityClassName: high-priority
```

---

### 7.3 Simulate Preemption

1. Create a low-priority workload with lots of CPU.
2. Deploy high-priority pod that requires the same resources.
3. Observe eviction of low-priority pods.

---

### 7.4 Enable Cleanup Policy on CronJob

```yaml
    successfulJobsHistoryLimit: 2
    failedJobsHistoryLimit: 1
```

✅ Inspect job history:

```bash
kubectl get jobs -n kube-lab --sort-by=.metadata.creationTimestamp
```

✅ Clean up:

```bash
kubectl delete cronjob db-query-job -n kube-lab
```

---

✅ You’ve successfully implemented timed automation, workload priority, and preemptive strategies.

## ✅ Final Step: Wrap-up, Validation & Cleanup

Now that you've implemented all Kubernetes features step-by-step, it's time to test everything together and clean up properly.

---

### 🔍 Full Application Check

```bash
kubectl get all -n kube-lab
kubectl get pvc,pv -n kube-lab
kubectl describe deployment flask-app -n kube-lab
kubectl logs deployment/flask-app -n kube-lab
```

Check endpoints:

```bash
kubectl port-forward deployment/flask-app 5000:5000 -n kube-lab
```

Visit:

* `http://localhost:5000/healthz`
* `http://localhost:5000/readyz`
* `http://localhost:5000/db-write`
* `http://localhost:5000/db-read`
* `http://localhost:5000/cache`

---

### 📜 Validate Advanced Features

* Probe behaviors (simulate startup delay)
* Init container logs
* Sidecar logs from `/logs/app.log`
* Affinity & node placement via `kubectl get pods -o wide`
* Taints/tolerations behavior
* RBAC access with kubeconfig
* App running as non-root
* CronJob scheduling
* Resource limits with `kubectl top pod`

---

### 🧹 Cleanup (Optional)

```bash
kubectl delete ns kube-lab
kubectl delete priorityclass high-priority low-priority
kubectl delete storageclass local-storage
```

✅ Your application followed a complete production-grade deployment pattern with the best Kubernetes practices.


[Main](../README.md)
---