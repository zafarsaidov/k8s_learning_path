[Main](../README.md)
---

# 🧪 Kubernetes Probes Practice Lesson: Liveness, Readiness, Startup

## 🧠 Goal

Students will:
* Understand the difference between liveness, readiness, and startup probes.
* Learn how to define different probe types: httpGet, tcpSocket, exec.
* Observe how Kubernetes handles container restarts, pod availability, and traffic routing based on probes.

## 🛠️ Setup: Simple HTTP Server in Python (Flask)

Use the following code to simulate a simple server with custom endpoints for probes.

app.py
```python
from flask import Flask
import time
import os

app = Flask(__name__)
start_time = time.time()

@app.route('/healthz')
def healthz():
    return "OK", 200

@app.route('/ready')
def ready():
    if time.time() - start_time > 15:
        return "READY", 200
    return "NOT READY", 500

@app.route('/startup')
def startup():
    if time.time() - start_time > 10:
        return "STARTED", 200
    return "STARTING", 500

@app.route('/')
def index():
    return "Hello from Probe App!", 200
```

🐳 Dockerfile
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```
🧪 Build and Push
```bash
docker build -t <your-dockerhub>/probe-app:latest .
docker push <your-dockerhub>/probe-app:latest
```

## 📦 Kubernetes Deployment with Probes

probes-deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: probe-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: probe-app
  template:
    metadata:
      labels:
        app: probe-app
    spec:
      containers:
      - name: probe-app
        image: <your-dockerhub>/probe-app:latest
        ports:
        - containerPort: 5000
        livenessProbe:
          httpGet:
            path: /healthz
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 5000
          initialDelaySeconds: 5
          periodSeconds: 5
        startupProbe:
          httpGet:
            path: /startup
            port: 5000
          failureThreshold: 10
          periodSeconds: 2
```
Replace <your-dockerhub> accordingly.

## ✅ Apply the Deployment
```bash
kubectl apply -f probes-deployment.yaml
```

## 🔍 Verification

General Pod Status
```bash
kubectl get pods
kubectl describe pod -l app=probe-app
```
Watch Events
```bash
kubectl get events --sort-by='.lastTimestamp'
```
Logs
```bash
kubectl logs -l app=probe-app
```

## ⚡ TCP & Exec Probe Examples

### TCP Probe

Modify the livenessProbe section:
```yaml
livenessProbe:
  tcpSocket:
    port: 5000
  initialDelaySeconds: 5
  periodSeconds: 5
```
### Exec Probe

Assuming a container where a file /tmp/healthy indicates health:

Modify app to create a health file:
```python
@app.route('/break')
def break_app():
    os.remove("/tmp/healthy")
    return "App is broken", 500

@app.route('/fix')
def fix_app():
    with open("/tmp/healthy", "w") as f:
        f.write("ok")
    return "App is fixed", 200
```
Then update the container CMD:
```dockerfile
CMD ["sh", "-c", "touch /tmp/healthy && python app.py"]
```
And YAML:
```bash
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

## 🧪 Simulate Failures

Break the readiness:
```bash
kubectl exec -it <pod> -- curl localhost:5000/break
```
✅ Observe:
```bash
kubectl get pods
kubectl describe pod <pod>
```
Kubernetes will stop sending traffic to the pod but won’t restart it.

## Break liveness

Modify app so /healthz returns 500 and redeploy.

Kubernetes will restart the container.

## 🧹 Cleanup
```bash
kubectl delete deployment probe-app
```

## 🏁 Bonus Task

Ask students to:
* Combine HTTP liveness, TCP readiness, and Exec startup in one deployment.
* Observe behavior with a delay (e.g. sleep 30 before startup file is created).



[Main](../README.md)
---