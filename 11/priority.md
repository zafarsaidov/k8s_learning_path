[Main](../README.md)
---

# 🧪 Practice Lesson: Pod Priority and Preemption


🎯 Learning Objectives

By the end of this lesson, students will be able to:
* Define and use PriorityClass
* Observe pod preemption behavior when cluster resources are limited
* Test eviction scenarios by applying priorities



🧱 Theory in Practice (Quick Summary)
* PriorityClass defines the importance of a pod.
* When resources are limited, higher priority pods can preempt (evict) lower priority pods.
* Preemption only happens if there is no other way to schedule the pod.

## 1️⃣ Create Three Priority Classes

priorities.yaml
```yaml
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "For critical workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: medium-priority
value: 50000
globalDefault: false
description: "For regular workloads"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 1000
globalDefault: false
description: "For background jobs"
...
```
```bash
kubectl apply -f high-priority.yaml
kubectl apply -f medium-priority.yaml
kubectl apply -f low-priority.yaml
kubectl get priorityclass
```

## 2️⃣ Fill Up the Cluster with Low Priority Pods

Create 5 pods using a memory-heavy image.

low-load.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: low-load-1
spec:
  priorityClassName: low-priority
  containers:
    - name: stress
      image: polinux/stress
      args: ["--vm", "1", "--vm-bytes", "200Mi", "--vm-hang", "1"]
      resources:
        requests:
          memory: "200Mi"
          cpu: "100m"
        limits:
          memory: "200Mi"
          cpu: "200m"
```
```bash
for i in {1..5}; do
  kubectl apply -f low-load.yaml -n default --dry-run=client -o yaml | \
  sed "s/low-load-1/low-load-$i/g" | kubectl apply -f -
done
```

✅ Check: Pods should be scheduled and consume cluster memory.
```bash
kubectl top pods
kubectl get pods -o wide
```

## 3️⃣ Attempt to Schedule High Priority Pod

high-priority-pod.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: high-priority-pod
spec:
  priorityClassName: high-priority
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          memory: "300Mi"
          cpu: "100m"
```
```bash
kubectl apply -f high-priority-pod.yaml
```

✅ Expected Result: Some low-priority pods will be evicted to free space for the high-priority pod.
```bash
kubectl get pods
kubectl describe pod high-priority-pod
kubectl get events --sort-by=.metadata.creationTimestamp
```

## 4️⃣ Try a Medium Priority Pod (No Preemption)

medium-priority-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: medium-priority-pod
spec:
  priorityClassName: medium-priority
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      resources:
        requests:
          memory: "400Mi"
          cpu: "100m"
```
```bash
kubectl apply -f medium-priority-pod.yaml
```
✅ If there’s no space, it will remain Pending, and no preemption will happen, because preemption only occurs if the incoming pod has higher priority.

## 🔍 Useful Verification Commands

| Command | Purpose |
| ---- | ---- |
| `kubectl get priorityclass` |	See all priorities |
| `kubectl top pods` |	Monitor resource usage |
| `kubectl describe pod <name>` |	Check preemption and events |
| `kubectl get events --sort-by=.metadata.creationTimestamp` |	See scheduling/preemption messages |
| `kubectl delete pod <name>` |	Free up space manually |


## 🎁 Bonus Tasks
* Create a Deployment with high-priority pods and scale it up — see which pods are evicted.
* Set preemptionPolicy: Never in the high-priority pod to prevent eviction of others.
```yaml
preemptionPolicy: Never
```
* Observe that pods remain Pending.

## 🧹 Cleanup
```bash
kubectl delete pod high-priority-pod medium-priority-pod
kubectl delete pod -l name=low-load
kubectl delete priorityclass high-priority medium-priority low-priority
```


[Main](../README.md)
---