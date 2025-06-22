[Main](../README.md)
---

# Service


## 🎯 Objective

By the end of this practice, students will understand:
* What Kubernetes Services are and why we use them.
* How to expose Pods using ClusterIP and NodePort Services.
* How to access services internally and externally.




## 🛠️ Practice Setup

### ✅ Step 1: Deploy a Sample Application

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx
        ports:
        - containerPort: 80
```
Apply the file:
```bash
kubectl apply -f pod-deployment.yaml
kubectl get pods -l app=web
```

## 🌐 Part 1: ClusterIP Service

### ✅ Step 2: Create a ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-clusterip
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  type: ClusterIP
```
Apply and verify:
```bash
kubectl apply -f clusterip-service.yaml
kubectl get svc web-clusterip
```

### ✅ Step 3: Access Service Internally (Using a Test Pod)

```bash
kubectl run curlpod --image=radial/busyboxplus:curl -it --restart=Never -- /bin/sh
# Inside pod:
curl web-clusterip
```

## 🌍 Part 2: NodePort Service

### ✅ Step 4: Create a NodePort Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-nodeport
spec:
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
    protocol: TCP
  type: NodePort
```
Apply and verify:
```bash
kubectl apply -f nodeport-service.yaml
kubectl get svc web-nodeport
```

### ✅ Step 5: Access NodePort From Browser or curl

Get node IP:

```bash
kubectl get nodes -o wide
```

Access in browser or curl:

```bash
curl http://<NODE-IP>:30080
```


### 🧼 Cleanup

```bash
kubectl delete -f pod-deployment.yaml
kubectl delete -f clusterip-service.yaml
kubectl delete -f nodeport-service.yaml
kubectl delete pod curlpod
```

## 🧠 Bonus Challenges
* Change the selector of a service and watch it stop working.
* Create two deployments (web and api) and expose both with different services.
* Create a ClusterIP service with port 8080 and targetPort 80.



[Main](../README.md)
---