[Main](../README.md)
---

# CoreDNS

## 🎯 Learning Objectives

By the end of this session, students will understand:
* What CoreDNS is and its role in Kubernetes.
* How DNS-based service discovery works in Kubernetes.
* How to troubleshoot and test DNS resolution in a cluster.


## ⚙️ Practice: CoreDNS Hands-On Guide

### ✅ Step 1: Verify CoreDNS is Running

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```
Expected output:

```
NAME                       READY   STATUS    RESTARTS   AGE
coredns-xxx                1/1     Running   0          5d
```

### ✅ Step 2: Inspect CoreDNS ConfigMap

```bash
kubectl -n kube-system get configmap coredns -o yaml
```

Example snippet:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
          pods insecure
          fallthrough in-addr.arpa ip6.arpa
        }
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

### ✅ Step 3: Test DNS Resolution in the Cluster

#### Deploy a test Pod with DNS tools:

```bash
kubectl run dnsutils --image=busybox:1.28 --restart=Never -it --rm -- nslookup kubernetes
```

You should see a response resolving kubernetes.default.svc.cluster.local.
#### Try FQDN resolution:
```bash
nslookup nginx-service.default.svc.cluster.local
```
Replace nginx-service with your actual service.

### ✅ Step 4: Create a Service and Test DNS Resolution

#### Create Deployment & Service:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: echo
  template:
    metadata:
      labels:
        app: echo
    spec:
      containers:
      - name: echo
        image: hashicorp/http-echo
        args:
        - "-text=hello"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: echo
spec:
  selector:
    app: echo
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5678
```
Apply:
```bash
kubectl apply -f echo-deployment.yaml
```
#### Run a client pod and resolve the echo service:
```bash
kubectl run curlpod --image=radial/busyboxplus:curl -it --rm
# Inside pod:
nslookup echo
curl echo
```

### ✅ Step 5: Troubleshooting DNS Issues

Check DNS logs:
```bash
kubectl logs -n kube-system -l k8s-app=kube-dns
```
Common issues:
* CoreDNS crashloop → usually a syntax error in Corefile.
* No response → network policies, iptables misconfig.
* Use dig or nslookup to test from multiple pods.

## 🧠 Bonus Tasks
* Modify CoreDNS ConfigMap to log DNS queries:
```json
.:53 {
  log
  errors
  ...
}
```
Then apply:
```bash
kubectl -n kube-system apply -f coredns-config.yaml
kubectl rollout restart deployment coredns -n kube-system
```
* Scale deployment to zero and check resolve services

## 🧼 Cleanup

```bash
kubectl delete deployment echo
kubectl delete svc echo
kubectl delete pod curlpod
```

## ✅ Summary

| Concept | Example |
| ------ | ------|
| DNS server | CoreDNS |
| Service name FQDN | my-svc.my-namespace.svc.cluster.local |
| CoreDNS config | ConfigMap in kube-system |
| Debug tool | nslookup, dig, curl in a busybox pod |



[Main](../README.md)
---