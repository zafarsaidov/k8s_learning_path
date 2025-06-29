[Main](../README.md)
---

# 🧪 Kubernetes Practice: nodeName, nodeSelector, Affinity, Anti-Affinity

## Label Nodes for Scheduling

```bash
kubectl get nodes
kubectl label nodes <node1> disk=ssd
kubectl label nodes <node2> zone=zone-a
kubectl label nodes <node2> env=prod
```
✅ Check:
```bash
kubectl get nodes --show-labels
```

## Schedule Pod Using nodeName

Task: Create a pod that runs only on node1.

📄 pod-nodename.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodename
spec:
  containers:
  - name: nginx
    image: nginx
  nodeName: <node1>
```
✅ Verify:
```bash
kubectl get pod pod-nodename -o wide
```

## Use nodeSelector

Task: Schedule a pod to any node with disk=ssd.

📄 pod-nodeselector.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeselector
spec:
  containers:
  - name: nginx
    image: nginx
  nodeSelector:
    disk: ssd
```
✅ Verify:
```bash
kubectl get pod pod-nodeselector -o wide
```

## Use Required nodeAffinity

Task: Schedule to node with zone=zone-a.

📄 pod-nodeaffinity.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - zone-a
  containers:
  - name: nginx
    image: nginx
```
✅ Verify:
```bash
kubectl get pod pod-nodeaffinity -o wide
```

## Use Preferred nodeAffinity

Task: Prefer node with env=prod.

📄 pod-nodeaffinity-preferred.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nodeaffinity-pref
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - preference:
          matchExpressions:
          - key: env
            operator: In
            values:
            - prod
        weight: 100
  containers:
  - name: nginx
    image: nginx
```
✅ Verify:
```bash
kubectl get pod pod-nodeaffinity-pref -o wide
```

## Use podAffinity (Co-locate Pods)

Task:
* Create backend pod with label app: backend.
* Then schedule frontend pod that wants to be on the same node.

📄 backend.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx
```
📄 frontend-affinity.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: backend
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: nginx
    image: nginx
```
✅ Verify:
```bash
kubectl get pods -o wide
```

## Use podAntiAffinity (Avoid Same Node)

📄 isolated.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: isolated
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            app: backend
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: nginx
    image: nginx
```
✅ Verify:
```bash
kubectl get pods -o wide
```

## Combine Node + Pod Affinity

Task: Deploy a pod that:
* Runs only on nodes with zone=zone-a
* Wants to be on the same node as another pod with label tier=api

📄 combined-affinity.yaml:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: complex
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: zone
            operator: In
            values:
            - zone-a
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchLabels:
            tier: api
        topologyKey: "kubernetes.io/hostname"
  containers:
  - name: nginx
    image: nginx
```
✅ Test by deploying a pod with tier=api label.

## 🧹 Cleanup
```bash
kubectl delete pod pod-nodename pod-nodeselector pod-nodeaffinity pod-nodeaffinity-pref backend frontend isolated complex
kubectl label node <node1> disk-
kubectl label node <node2> zone- env-
```

## 🎁 Bonus Exercises
* Try scheduling a pod with nodeSelector to a node without the label — observe the pod stay Pending.
* Schedule 2 pods using podAntiAffinity with topologyKey: zone, not hostname.
* Combine preferred and required affinity types.
* Write a Deployment that only schedules on nodes with disktype=nvme.



[Main](../README.md)
---