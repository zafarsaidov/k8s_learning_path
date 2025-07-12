[Main](../README.md)
---

# 🧪 Practice Lesson: PodSecurityContext & Security Best Practices in Kubernetes

🎯 Objectives

By the end of this lesson, students will:
* Understand the purpose of securityContext at pod and container levels
* Run pods with non-root users
* Control file permissions using fsGroup and runAsGroup
* Use readOnlyRootFilesystem, capabilities, allowPrivilegeEscalation, and more
* Apply best practices like dropping Linux capabilities and non-root enforcement

🧱 Key Concepts

| Setting |	Purpose |
| ---- | ---- |
| runAsUser / runAsGroup |	Run processes as specific UID/GID |
| fsGroup |	Group ownership of volumes |
| allowPrivilegeEscalation |	Prevent gaining more privileges inside container |
| readOnlyRootFilesystem |	Makes root filesystem read-only |
| capabilities |	Drop/keep Linux capabilities |


## 🛠️ Tasks & Use Cases

### 1️⃣ Task: Run a Pod as Non-root User

📄 nonroot-pod.yaml
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: nonroot
spec:
  securityContext:
    runAsUser: 1001
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
```
```bash
kubectl apply -f nonroot-pod.yaml
kubectl exec -it nonroot -- id
```
✅ Output: UID=1001, GID=3000, groups=2000

### 2️⃣ Task: Use readOnlyRootFilesystem

📄 readonly-root.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "touch /tmp/test && sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
```
✅ This will crash because touch will fail. Modify it:
```yaml
command: ["sh", "-c", "touch /tmp/test && echo OK && sleep 3600"]
volumeMounts:
- name: tmp
  mountPath: /tmp
volumes:
- name: tmp
  emptyDir: {}
```
✅ Now it will run correctly with writable /tmp.

### 3️⃣ Task: Drop Linux Capabilities

📄 drop-cap.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: drop-cap
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "ping 127.0.0.1 || echo failed && sleep 3600"]
    securityContext:
      capabilities:
        drop: ["ALL"]
      allowPrivilegeEscalation: false
```
✅ ping fails because CAP_NET_RAW is dropped.

### 4️⃣ Task: Enforce User + Group + Volume Ownership

📄 volume-ownership.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test
spec:
  securityContext:
    runAsUser: 1001
    fsGroup: 2000
  volumes:
  - name: data
    emptyDir: {}
  containers:
  - name: busy
    image: busybox
    command: ["sh", "-c", "ls -l /data && sleep 3600"]
    volumeMounts:
    - mountPath: /data
      name: data
```
✅ Volume /data will be owned by GID 2000.
```bash
kubectl exec -it volume-test -- ls -ld /data
```

### 5️⃣ Task: Combine All Best Practices

📄 secure-app.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-app
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
    fsGroup: 3000
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
    securityContext:
      readOnlyRootFilesystem: true
      allowPrivilegeEscalation: false
      capabilities:
        drop: ["ALL"]
```
✅ Check the output:
```bash
kubectl exec -it secure-app -- id
```
### 6️⃣ Bonus: Try Without SecurityContext and With HostPath

📄 insecure-pod.yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: root-access
spec:
  containers:
  - name: root
    image: busybox
    command: ["sh", "-c", "id && sleep 3600"]
    volumeMounts:
    - name: host
      mountPath: /host
  volumes:
  - name: host
    hostPath:
      path: /
      type: Directory
```
✅ This gives root access to node filesystem — highlight why securityContext matters!

### 🔍 Verification Commands

| Command |	Purpose |
| --- | --- |
| kubectl describe pod <name> |	View pod and container security context |
| kubectl exec <pod> -- id |	Check UID/GID inside container |
| kubectl logs <pod> |	See output/errors for tests |
| kubectl get pod <pod> -o jsonpath='{.spec.securityContext}' |	Extract context config |


## 🧹 Cleanup

```bash
kubectl delete pod nonroot readonly-pod drop-cap volume-test secure-app root-access
```

[Main](../README.md)
---