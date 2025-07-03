[Main](../README.md)
---

# 🧪 Practice Lesson: Resource Requests, Limits, ResourceQuota, and LimitRange

## 🎯 Objectives
* Understand how resource requests and limits work
* Deploy pods with/without resource definitions
* Apply and test namespace ResourceQuota and LimitRange

## 📚 Part 1: Resource Requests and Limits

### 🧪 Task 1: Deploy Pods with Resource Requests and Limits

Create a pod with defined resource requests and limits:

pod-resources.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: stress-pod
spec:
  containers:
    - name: stress
      image: polinux/stress
      args: ["--cpu", "1"]
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "200m"
          memory: "256Mi"
```
```bash
kubectl apply -f pod-resources.yaml
kubectl describe pod stress-pod
```
✅ Check: Pod will be scheduled only on nodes with at least 100m CPU and 128Mi memory available.

### 🧪 Task 2: Observe Behavior Without Limits

pod-no-limits.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-limits
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sh", "-c", "while true; do echo RUNNING; sleep 5; done"]
```
```bash
kubectl apply -f pod-no-limits.yaml
```
✅ Observe: No constraints — could use a lot of resources if under pressure.

## 📦 Part 2: LimitRange

### 🧪 Task 3: Create a Namespace and LimitRange
```bash
kubectl create ns dev-team
```
limitrange.yaml
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limitrange
  namespace: dev-team
spec:
  limits:
    - default:
        cpu: 500m
        memory: 256Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      type: Container
```
```bash
kubectl apply -f limitrange.yaml
```
### 🧪 Task 4: Deploy Pod Without Resources to See Auto-applied Limits

pod-auto-resources.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: auto-resources
  namespace: dev-team
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
```
```bash
kubectl apply -f pod-auto-resources.yaml
kubectl describe pod auto-resources -n dev-team
```
✅ Check: Resource requests/limits auto-assigned by LimitRange.

## 📊 Part 3: ResourceQuota

### 🧪 Task 5: Apply ResourceQuota in the Namespace

resourcequota.yaml
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: dev-team
spec:
  hard:
    pods: "5"
    requests.cpu: "500m"
    requests.memory: "512Mi"
    limits.cpu: "1"
    limits.memory: "1Gi"
```
```bash
kubectl apply -f resourcequota.yaml
```
### 🧪 Task 6: Try to Deploy Pods to Exceed Quota

Create this pod and attempt to run it multiple times:

big-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: big-pod
  namespace: dev-team
spec:
  containers:
    - name: stress
      image: polinux/stress
      args: ["--cpu", "1"]
      resources:
        requests:
          cpu: "300m"
          memory: "300Mi"
        limits:
          cpu: "500m"
          memory: "600Mi"
```
```bash
kubectl apply -f big-pod.yaml -n dev-team
```
Try creating 2-3 copies and see rejections after quota exceeds

## ✅ Check:
* Pods beyond quota are rejected with a FailedScheduling event.
* Use kubectl describe quota dev-quota -n dev-team to see usage.

## 🔍 Commands for Verification

| Command |	Purpose |
| ---- | ---- |
| `kubectl top pod` |	See live resource usage (metrics-server required) |
| `kubectl describe pod <name>` |	See assigned requests/limits |
| `kubectl describe limitrange -n <ns>` |	Inspect default resource policy |
| `kubectl describe quota -n <ns>` |	View quota usage and limits |
| `kubectl get pod -n <ns>` |	Check pod status |
| `kubectl logs <pod>` |	View pod output |


## 🧹 Cleanup
```bash
kubectl delete ns dev-team
kubectl delete pod stress-pod no-limits
```

## 🎁 Bonus Task
* Try a Deployment instead of a Pod using resource limits.
* Create a LimitRange that restricts max CPU > 1 and see pod rejections.
* Set a quota for configmaps or secrets to limit excessive object creation.


[Main](../README.md)
---