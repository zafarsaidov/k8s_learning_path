[Main](../README.md)
---

# 🧪 Practice Lesson: Service Account Use Case – Accessing Kubernetes API from a Pod


### 🎯 Objective

Students will simulate a real-world use case where:
* A pod uses a service account to authenticate with the Kubernetes API
* The pod lists configmaps in its namespace
* Access is granted only via RBAC using a Role + RoleBinding
* Students verify the permissions using the mounted token in the pod


## 🧱 Real-World Scenario

Imagine you have a pod that runs an internal tool which needs to list ConfigMaps in its namespace. You want to allow that — but restrict everything else. You’ll do this securely using a dedicated service account, RBAC, and in-cluster API access.

### 1️⃣ Create Namespace

```bash
kubectl create ns sa-demo
```


### 2️⃣ Create a Service Account for the Application
```bash
kubectl create serviceaccount configmap-reader -n sa-demo
```

### 3️⃣ Create Role & RoleBinding for the Service Account

📄 configmap-reader-role.yaml
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-configmaps
  namespace: sa-demo
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-configmap-reader
  namespace: sa-demo
subjects:
- kind: ServiceAccount
  name: configmap-reader
  namespace: sa-demo
roleRef:
  kind: Role
  name: read-configmaps
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f configmap-reader-role.yaml
```

### 4️⃣ Create a Pod that Uses the Service Account and Talks to the K8s API

📄 pod-api-client.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-client
  namespace: sa-demo
spec:
  serviceAccountName: configmap-reader
  containers:
  - name: client
    image: curlimages/curl:latest
    command:
      - sh
      - -c
      - |
        TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
        curl -sSk \
          -H "Authorization: Bearer $TOKEN" \
          https://kubernetes.default.svc/api/v1/namespaces/sa-demo/configmaps
```
```bash
kubectl apply -f pod-api-client.yaml
```
✅ Check pod logs:
```bash
kubectl logs api-client -n sa-demo
```
Expected output: JSON list of ConfigMaps (may be empty if none exist).

### 5️⃣ Try a Pod That Tries to List Secrets (Should Fail)

📄 pod-illegal-access.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: api-client-secrets
  namespace: sa-demo
spec:
  serviceAccountName: configmap-reader
  containers:
  - name: client
    image: curlimages/curl:latest
    command:
      - sh
      - -c
      - |
        TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
        curl -sSk \
          -H "Authorization: Bearer $TOKEN" \
          https://kubernetes.default.svc/api/v1/namespaces/sa-demo/secrets
```
```bash
kubectl apply -f pod-illegal-access.yaml
kubectl logs api-client-secrets -n sa-demo
```
Expected output: 403 Forbidden

### 🔍 Verification

| Command |	Purpose |
| ---- | ---- |
| kubectl describe sa configmap-reader -n sa-demo	| Verify SA is created |
| kubectl get role,rolebinding -n sa-demo	| Confirm RBAC |
| kubectl logs <pod>	| Confirm API access |
| kubectl auth can-i list secrets --as=system:serviceaccount:sa-demo:configmap-reader -n sa-demo | Simulate access |


## 🧹 Cleanup

```bash
kubectl delete ns sa-demo
```


[Main](../README.md)
---