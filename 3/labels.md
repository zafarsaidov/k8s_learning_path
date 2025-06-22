[Main](../README.md)
---

## Labels and selectors

[Documentation](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

🎯 Objective

__Understand:__
* How to assign labels to Kubernetes resources.
* How to filter resources using label selectors.
* How Services, Deployments, and kubectl use label selectors.

## Practice Guide: Pod Labels & Selectors

### ✅ Step 1: Create Pods with Labels

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: web
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: web
    tier: backend
spec:
  containers:
  - name: httpd
    image: httpd
```

Apply the file:
```bash
kubectl apply -f pod-labels.yaml
```


### ✅ Step 2: View Labels on Pods

```bash
kubectl get pods --show-labels
```

Expected output shows label columns.

Or:
```bash
kubectl get pod frontend -o jsonpath="{.metadata.labels}"
```

### ✅ Step 3: Filter Pods Using Label Selectors

```bash
kubectl get pods -l app=web
kubectl get pods -l tier=frontend
kubectl get pods -l 'app=web,tier=frontend'
```

### ✅ Step 4: Label an Existing Pod (Add/Modify)

```bash
kubectl label pod backend env=prod
kubectl get pod backend --show-labels
```

Overwrite a label:

```bash
kubectl label pod backend env=dev --overwrite
```
Remove a label:
```bash
kubectl label pod backend env-
```


### ✅ Step 5: Use Label Selectors in a Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: web
    tier: frontend
  ports:
  - port: 80
    targetPort: 80
```
Apply:
```bash
kubectl apply -f service-selector.yaml
kubectl get endpoints frontend-service
```

__Explanation:__ This service routes traffic only to the pod with both labels app=web and tier=frontend.


### ✅ Step 6: Advanced Selector Examples (MatchExpressions)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug
  labels:
    role: test
    env: dev
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sleep", "3600"]

# deployment-match-expressions.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-select
spec:
  replicas: 1
  selector:
    matchExpressions:
    - key: env
      operator: In
      values:
      - dev
      - test
  template:
    metadata:
      labels:
        app: demo
        env: dev
    spec:
      containers:
      - name: nginx
        image: nginx

```

### 🧼 Cleanup

```bash
kubectl delete -f pod-labels.yaml
kubectl delete -f service-selector.yaml
kubectl delete pod debug
kubectl delete deployment demo-select
```


[Main](../README.md)
---