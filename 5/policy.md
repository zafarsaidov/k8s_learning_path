[Main](../README.md)
---

# Network policy

## ✅ Step 1: Setup Namespace and Pods
```bash
kubectl create namespace netpol-demo
```
## ✅ Deploy 2 apps: frontend and backend

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: netpol-demo
  labels:
    app: backend
spec:
  containers:
  - name: backend
    image: hashicorp/http-echo
    args: ["-text=Hello from backend"]
    ports:
    - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: netpol-demo
spec:
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 5678
---
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: netpol-demo
  labels:
    app: frontend
spec:
  containers:
  - name: curl
    image: curlimages/curl
    command: ["sleep", "3600"]
```
Apply:
```bash
kubectl apply -f netpol-demo.yaml
```

## ✅ Step 2: Test Without Network Policy
```bash
kubectl exec -n netpol-demo frontend -- curl backend:80
# Expected output: Hello from backend
```

## ✅ Step 3: Create a Deny-All Ingress Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
  namespace: netpol-demo
spec:
  podSelector: {}  # Applies to all Pods
  policyTypes:
  - Ingress
```
Apply:
```bash
kubectl apply -f deny-all.yaml
```
Test again:
```bash
kubectl exec -n netpol-demo frontend -- curl backend:80
# Expected: connection refused or timeout
```

## ✅ Step 4: Allow Ingress Only from Frontend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
  policyTypes:
  - Ingress
```
Apply:
```bash
kubectl apply -f allow-frontend.yaml
```
Test again:
```bash
kubectl exec -n netpol-demo frontend -- curl backend:80
# Expected: Hello from backend
```

## ✅ Step 5: Deny All Egress from Frontend (Optional)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-egress
  namespace: netpol-demo
spec:
  podSelector:
    matchLabels:
      app: frontend
  egress: []
  policyTypes:
  - Egress
```
Apply:
```bash
kubectl apply -f deny-egress.yaml
```
Try pinging external IP:
```bash
kubectl exec -n netpol-demo frontend -- curl http://1.1.1.1
# Should be blocked
```

## 🧪 Bonus Tasks
* Allow egress only to specific IP (e.g., DNS or API).
* Use namespaceSelector to allow traffic only from Pods in a trusted namespace.
* Combine ingress and egress rules in one policy.

## 🧼 Cleanup
```bash
kubectl delete ns netpol-demo
```

## 🧠 Summary Table

| Feature	| Description |
| ------ | -------|
| podSelector |	Which pods the policy applies to |
| ingress |	Controls incoming connections |
| egress |	Controls outgoing traffic |
| policyTypes |	Ingress, Egress or both |
| Requires CNI |	Yes, e.g., Calico, Cilium, Weave |

[Documentation](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

[Main](../README.md)
---