[Main](../README.md)
---

## Environment Variables, ConfigMaps, and Secrets

## 🎯 Objectives

By the end of this practice, students will:
* Set and verify environment variables in containers
* Create and use ConfigMaps (via env, envFrom, and volume mounts)
* Create and use Secrets (imperatively and declaratively)
* Use Secrets for Docker registry authentication

## Using Environment Variables in Containers

### ✅ Task: Create a Pod with Custom Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    env:
    - name: APP_ENV
      value: production
    - name: VERSION
      value: "1.0"
```
Apply and verify:
```bash
kubectl apply -f env-pod.yaml
kubectl exec -it env-demo -- env | grep APP_ENV
```

## ConfigMaps

### 📌 Create a ConfigMap

Imperatively:
```bash
kubectl create configmap app-config --from-literal=APP_MODE=debug --from-literal=LOG_LEVEL=info
```
Declaratively:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: debug
  LOG_LEVEL: info
```
Apply:
```bash
kubectl apply -f configmap.yaml
```

### 📌 Use ConfigMap in a Pod (via envFrom)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
```
Check values:
```bash
kubectl exec -it configmap-demo -- env | grep APP_MODE
```

### 📌 Use ConfigMap in a Pod (via Volume)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "cat /etc/config/APP_MODE && sleep 3600"]
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config
  volumes:
  - name: config-vol
    configMap:
      name: app-config
```

## Secrets

### 📌 Create a Secret

Imperative:
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```
Declarative:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=      # base64 of "admin"
  password: c2VjcmV0MTIz  # base64 of "secret123"
```
List & View:
```bash
kubectl get secrets
kubectl describe secret db-secret
kubectl get secret db-secret -o yaml
```

### 📌 Use Secret in a Pod (via env)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo $DB_USER && echo $DB_PASS && sleep 3600"]
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

### 📌 Use Secret as Volume
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "cat /etc/secret/username && sleep 3600"]
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secret
      readOnly: true
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```

## Docker Registry Secret (Pull Private Image)

Create Docker Registry Secret:
```bash
kubectl create secret docker-registry myregistrykey \
  --docker-username=<username> \
  --docker-password=<password> \
  --docker-email=<email> \
  --docker-server=<registry>
```
Use it in a Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-pod
spec:
  containers:
  - name: myapp
    image: <private-registry>/<image>:<tag>
  imagePullSecrets:
  - name: myregistrykey
```

## 🔄 Cleanup
```bash
kubectl delete pod env-demo configmap-demo configmap-volume-demo secret-env-demo secret-volume-demo private-pod
kubectl delete configmap app-config
kubectl delete secret db-secret myregistrykey
```

## 🧠 Bonus Tasks
* Update ConfigMap and trigger rollout (used with Deployments)
* Mount Secret with specific permission
* Expose ENV from both ConfigMap and Secret



[Main](../README.md)
---