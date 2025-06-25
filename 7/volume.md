[Main](../README.md)
---

# Kubernetes Volumes

## emptyDir Volume

📝 Task

Create a pod that uses an emptyDir volume shared between two containers.

✅ Example YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "hello from writer" > /data/file && sleep 3600']
    volumeMounts:
    - mountPath: /data
      name: shared-data
  - name: reader
    image: busybox
    command: ['sh', '-c', 'sleep 5 && cat /data/file && sleep 3600']
    volumeMounts:
    - mountPath: /data
      name: shared-data
  volumes:
  - name: shared-data
    emptyDir: {}
```

## hostPath Volume

✅ Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo Hello > /host/log.txt && sleep 3600']
    volumeMounts:
    - mountPath: /host
      name: host-volume
  volumes:
  - name: host-volume
    hostPath:
      path: /tmp/data
      type: DirectoryOrCreate
```

## Create Persistent Volume (PV)

Use local node storage manually

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 500Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
  persistentVolumeReclaimPolicy: Retain
```

## Create Persistent Volume Claim (PVC)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 200Mi
```

## Create Pod Deployment with PVC

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pvc-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: pvc-demo
  template:
    metadata:
      labels:
        app: pvc-demo
    spec:
      containers:
      - name: app
        image: busybox
        command: ['sh', '-c', 'echo Hello from PVC > /data/hello.txt && sleep 3600']
        volumeMounts:
        - mountPath: /data
          name: vol
      volumes:
      - name: vol
        persistentVolumeClaim:
          claimName: local-pvc
```

## StorageClass + Dynamic PVC (with Local Provisioner or External)

🎯 StorageClass Example (Local Volume Provisioner)

__Prerequisite:__ Configure local volume provisioner or use hostPath in kind/minikube.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### 📝 Dynamic PVC Example
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 100Mi
```

### 🧪 Pod using dynamic PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dynamic-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ['sh', '-c', 'echo using dynamic PVC > /mnt/msg && sleep 3600']
    volumeMounts:
    - mountPath: /mnt
      name: vol
  volumes:
  - name: vol
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

## Clean Up

```bash
kubectl delete pod,deploy,pvc,pv,sc --all
```

## 🎁 BONUS TASKS
* Retention Check:
    * Set PV reclaimPolicy to Retain
    * Delete the PVC → check if volume is still there.
* Data Persistence:
    * Re-deploy the pod and verify data is retained.
* Multi-pod Access:
    * Deploy 2 pods sharing a ReadWriteMany (if available) volume.
* Volume Backup:
    * kubectl cp files from PVC-mounted path.
* ReadOnlyMount:
    * Mount a volume with readOnly: true and try to write.



[Main](../README.md)
---