[Main](../README.md)
---

# YAML structure and object creation

## 🎯 Objective

By the end of this lesson, learners will be able to:
* Understand basic YAML syntax and structure.
* Write valid Kubernetes YAML manifests.
* Create Pods, Deployments, and Services from YAML.
* Use kubectl to create, inspect, and delete objects.


## 🧱 Part 1: YAML Basics (Theory)

🧾 YAML Rules:
* Indentation matters (use spaces, not tabs).
* Lists use -.
* Key/value pairs use :.

__Example YAML snippet__
```yaml
name: nginx
enabled: true
ports:
  - 80
  - 443
```


## ⚙️ Part 2: Practice – Creating Kubernetes Objects from YAML


### ✅ Step 1: Create a Pod Using YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
```

Apply:
```bash
kubectl apply -f pod.yaml
kubectl get pods
kubectl describe pod nginx-pod
```


### ✅ Step 2: Create a Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```
Apply:
```bash
kubectl apply -f deployment.yaml
kubectl get deployments
kubectl get pods -l app=nginx
```


### ✅ Step 3: Expose with a Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
Apply:
```bash
kubectl apply -f service.yaml
kubectl get svc
kubectl describe svc nginx-service
```

### 🛠 Step 4: Inspecting & Troubleshooting
```bash
kubectl get all
kubectl describe pod nginx-pod
kubectl logs <pod-name>
```


### 🧼 Cleanup
```bash
kubectl delete -f pod.yaml
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
```

### 📘 Bonus Tasks
* Create a YAML for a Pod with multiple containers.
* Add environment variables into a container using YAML.




### ✅ Summary of Key Kubernetes YAML Fields

| Field | Purpose | Example |
| --- | --- | --- |
| apiVersion | Specifies the API version | v1, apps/v1 |
| kind | Type of Kubernetes object | Pod, Deployment, Service |
| metadata | Metadata like name, labels | name: my-app |
| spec | Main configuration block | containers, replicas, etc. |
| template | Used in Deployment, Job, etc. | Pod template for replication |

---

## Imperative way

[Imperative Commands](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/imperative-command/)


[Main](../README.md)
---