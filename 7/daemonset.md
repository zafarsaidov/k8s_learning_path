[Main](../README.md)
---

# DaemonSets

## 🎯 Learning Objectives
* Understand what a DaemonSet is and its use cases
* Deploy a DaemonSet to run an agent on every node
* Verify that it runs one pod per node
* Explore real-world DaemonSets (e.g., log collectors, monitoring agents)

## 🧠 Theory: What Is a DaemonSet?

A DaemonSet ensures that a copy of a pod runs on all (or some) nodes in a cluster.

🛠 Common Use Cases
* Log collection (e.g., Fluentd, Filebeat)
* Node monitoring (e.g., Prometheus Node Exporter)
* System-level configuration agents
* Networking plugins (CNI agents)

When a new node joins the cluster, the DaemonSet automatically schedules a pod on it.

## ⚙️ Syntax Example
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
  labels:
    app: node-logger
spec:
  selector:
    matchLabels:
      name: node-logger
  template:
    metadata:
      labels:
        name: node-logger
    spec:
      containers:
      - name: logger
        image: busybox
        command: [ "/bin/sh", "-c", "while true; do echo Hello from $(hostname) >> /var/log/node.log; sleep 10; done" ]
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
```

## 🧪 Practice Tasks

### Create the DaemonSet

Save the above manifest as daemonset.yaml, then:
```bash
kubectl apply -f daemonset.yaml
```
### Confirm Pod Scheduling
```bash
kubectl get pods -o wide -l name=node-logger
```
✅ You should see one pod per node, each on a different node.

### Check Log File Written on Host

To verify that logs are written to the node’s /var/log/node.log:

```bash
kubectl exec -it <daemon-pod-name> -- tail /var/log/node.log
```
This uses a hostPath mount — files written here are visible on the host.

### 🔍 Real-World DaemonSet Example

Example: Install Prometheus Node Exporter as a DaemonSet

```yaml
kubectl apply -f https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/main/manifests/setup/prometheus-node-exporter-daemonset.yaml
```

## 🔄 Cleanup
```bash
kubectl delete daemonset node-logger
```

[Main](../README.md)
---