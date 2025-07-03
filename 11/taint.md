[Main](../README.md)
---

# 🧪 Practice Lesson: Taints and Tolerations

🎯 Objectives:
* Understand how taints prevent pod scheduling
* Learn how tolerations allow pods to bypass taints
* Practice real-world use cases (dedicated node, soft restrictions, NoExecute evictions)

🔍 Quick Refresher
* Taint: Applied to a node. Prevents pods from being scheduled unless they “tolerate” it.
* Toleration: Applied to a pod. Allows it to tolerate taints and be scheduled to tainted nodes.

## 1️⃣ Task: Taint a Node and Try to Schedule a Pod

✅ Steps:

Get nodes
```bash
kubectl get nodes
```
Pick a node (e.g., worker2)
```bash
kubectl taint nodes worker2 key1=value1:NoSchedule
```
Now create a test pod without toleration:

no-toleration-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: no-toleration
spec:
  containers:
    - name: nginx
      image: nginx
```
```bash
kubectl apply -f no-toleration-pod.yaml
kubectl describe pod no-toleration
```
## 🔍 Check:
* Pod will remain in Pending state.
* Describe will show “taint violation”.

## 2️⃣ Task: Add Toleration and Reapply

toleration-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "key1"
      operator: "Equal"
      value: "value1"
      effect: "NoSchedule"
```
```bash
kubectl apply -f toleration-pod.yaml
kubectl get pod toleration-pod -o wide
```
## 🔍 Check:
* Pod should now be scheduled to worker2.

## 3️⃣ Task: Dedicated Node for Critical Workloads

Label a node:
```bash
kubectl label node worker2 dedicated=monitoring
kubectl taint node worker2 dedicated=monitoring:NoSchedule
```

Create a monitoring pod with toleration and node selector:

monitoring-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monitoring
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "monitoring"
      effect: "NoSchedule"
  nodeSelector:
    dedicated: "monitoring"
```
```bash
kubectl apply -f monitoring-pod.yaml
```

✅ Now try scheduling a pod without toleration — it should not go to the tainted node.

## 4️⃣ Task: NoExecute Taint with Eviction

Apply NoExecute taint:
```bash
kubectl taint nodes worker2 test=evictme:NoExecute
```
Run a pod with no toleration on that node using nodeName:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: will-be-evicted
spec:
  nodeName: worker2
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
```
```bash
kubectl apply -f will-be-evicted.yaml
```

✅ Pod will start, then be evicted after a few seconds.

Now try tolerating it:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: tolerated
spec:
  nodeName: worker2
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
  tolerations:
    - key: "test"
      operator: "Equal"
      value: "evictme"
      effect: "NoExecute"
      tolerationSeconds: 30
```
✅ Pod stays for 30 seconds before eviction.

## 5️⃣ Bonus Task: Multiple Taints and Combined Tolerations

Taint node with multiple keys:
```bash
kubectl taint nodes worker2 role=backend:NoSchedule
kubectl taint nodes worker2 disk=ssd:NoSchedule
```
Create pod with partial toleration:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: partial
spec:
  containers:
    - name: nginx
      image: nginx
  tolerations:
    - key: "role"
      operator: "Equal"
      value: "backend"
      effect: "NoSchedule"
```
✅ Pod will still not schedule. Now add second toleration to succeed.

## 🔍 Commands for Verifying

| Command |	Purpose |
| ---- | ---- |
| `kubectl get pods -o wide` |	Check pod scheduling |
| `kubectl describe pod <pod>` |	See taint/toleration mismatch |
| `kubectl describe node <node>` |	View taints on node |
| `kubectl get nodes --show-labels` |	View node labels |
| `kubectl taint nodes <node> <key>=<value>:<effect>` |	Add taint |
| `kubectl taint nodes <node> <key>-` |	Remove taint |


## 🧹 Cleanup
```bash
kubectl delete pod --all
kubectl taint nodes <node> key1-
kubectl taint nodes <node> dedicated-
kubectl taint nodes <node> test-
kubectl taint nodes <node> role-
kubectl taint nodes <node> disk-
kubectl label node <node> dedicated-
```

[Main](../README.md)
---