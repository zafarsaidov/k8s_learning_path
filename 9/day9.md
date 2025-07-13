## ✅ Task 1: Persistent Volume for App

1.1 PersistentVolume (pv.yaml)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: app-pv
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
1.2 PersistentVolumeClaim (pvc.yaml)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
1.3 Application Pod with PVC (app-pod.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-with-pv
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "echo Hello > /data/file.txt && sleep 3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: app-pvc
```
Check:
```bash
kubectl exec -it app-with-pv -- cat /data/file.txt
```



## ✅ Task 2: ConfigMaps and Secrets

2.1 ConfigMap
```bash
kubectl create configmap app-config --from-literal=APP_MODE=production
```
2.2 Secret
```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=secret123
```
2.3 Pod with envFrom and volume mounts (app-env.yaml)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-secret-demo
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["sh", "-c", "env | grep APP_MODE && cat /secrets/username && sleep 3600"]
    envFrom:
    - configMapRef:
        name: app-config
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
    volumeMounts:
    - name: secret-vol
      mountPath: "/secrets"
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```
Check:
```bash
kubectl exec -it config-secret-demo -- env | grep DB_USER
kubectl exec -it config-secret-demo -- cat /secrets/username
```


## ✅ Task 3: Deploy a Stateful App

3.1 StorageClass (optional, for dynamic provisioning)
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
3.2 Create manual PVs for StatefulSet (e.g., 2 nodes)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/redis-0"
  persistentVolumeReclaimPolicy: Retain
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis-pv-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data/redis-1"
  persistentVolumeReclaimPolicy: Retain
```
3.3 Headless Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
    - port: 6379
```
3.4 StatefulSet (statefulset.yaml)
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: "redis"
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
      - name: redis
        image: redis
        ports:
        - containerPort: 6379
        volumeMounts:
        - name: redis-data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Check:
```bash
kubectl get pods -l app=redis
kubectl get pvc
kubectl exec -it redis-0 -- redis-cli set foo bar
kubectl exec -it redis-0 -- redis-cli get foo
```
Scale Down & Up:
```bash
kubectl scale statefulset redis --replicas=1
kubectl scale statefulset redis --replicas=2
```
Observe Persistent Volumes:
```bash
kubectl get pv
kubectl get pvc
```


## 🧹 Clean-Up
```bash
kubectl delete pod app-with-pv config-secret-demo
kubectl delete pv app-pv redis-pv-0 redis-pv-1
kubectl delete pvc app-pvc
kubectl delete secret db-secret
kubectl delete configmap app-config
kubectl delete statefulset redis
kubectl delete svc redis
```


## Extra tasks

### 1. Mount PVC as Read-Only


```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: readonly-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
```
✅ Pod using PVC (read-only)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readonly-test
spec:
  containers:
  - name: reader
    image: busybox
    command: ["sh", "-c", "cat /data/file.txt; sleep 3600"]
    volumeMounts:
    - name: shared-storage
      mountPath: /data
      readOnly: true
  volumes:
  - name: shared-storage
    persistentVolumeClaim:
      claimName: readonly-pvc
```
You must pre-populate the volume beforehand using a write-enabled pod.

### 2. StatefulSet with Dynamic Provisioning using local-path

✅ Step 1: Apply local-path provisioner (if not present)

If you’re using Rancher Local Path Provisioner:
```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```
This installs:
* StorageClass named local-path
* Local path dynamic provisioner

Or manually define your own StorageClass with provisioner local-path.

✅ Step 2: StatefulSet with dynamic PVCs
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: app
spec:
  serviceName: "app"
  replicas: 2
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
      - name: busybox
        image: busybox
        command: ["sh", "-c", "echo Hello Stateful >> /data/file.txt; sleep 3600"]
        volumeMounts:
        - name: data
          mountPath: /data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: local-path
      resources:
        requests:
          storage: 100Mi
```
Check that PVCs are dynamically created:
```bash
kubectl get pvc
kubectl get pv
```

### 🔹 3. Backup Volume Data to Separate Pod via initContainer

🎯 Objective

Use an initContainer in a pod to backup data from a PVC into a backup location (e.g., /backup/).

Assume your PVC has some application data in /data.

✅ Pod with backup initContainer
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-backup
spec:
  volumes:
  - name: data-vol
    persistentVolumeClaim:
      claimName: app-pvc
  - name: backup-vol
    emptyDir: {}
  initContainers:
  - name: backup
    image: busybox
    command: ["sh", "-c", "cp -r /data/* /backup/"]
    volumeMounts:
    - name: data-vol
      mountPath: /data
    - name: backup-vol
      mountPath: /backup
  containers:
  - name: viewer
    image: busybox
    command: ["sh", "-c", "ls /backup; sleep 3600"]
    volumeMounts:
    - name: backup-vol
      mountPath: /backup
```
initContainer copies everything from the data volume to an emptyDir, then the main container can inspect it or send it to S3, etc.


### ✅ Verification Commands

Check PVC usage
```bash
kubectl describe pvc
```
View pod logs or content
```bash
kubectl exec -it readonly-test -- cat /data/file.txt
kubectl exec -it app-backup -- ls /backup
```
