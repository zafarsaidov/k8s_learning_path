
[Main](../README.md)
---

# StatefulSets

## 🎯 Objectives
* Understand the purpose and behavior of StatefulSets
* Use persistent volumes and a local StorageClass
* Observe predictable pod names, DNS, and scaling order
* Ensure data persistence across pod restarts

## Create a Local StorageClass

```yaml

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```
Apply:
```bash
kubectl apply -f storageclass.yaml
```

## Create a Headless Service

Headless service is required for stable network identity of StatefulSet pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web
spec:
  clusterIP: None  # Important for headless service
  selector:
    app: nginx
  ports:
  - port: 80
    name: web
```

## Create a StatefulSet

This example deploys NGINX with 3 replicas, each with its own persistent volume.

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx
  serviceName: "web"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "local-storage"
      resources:
        requests:
          storage: 1Gi
```
Apply:
```bash
kubectl apply -f statefulset.yaml
```

## Examine Pod Creation Order

```bash
kubectl get pods -l app=nginx --watch
```
You will see pods named web-0, web-1, web-2 created in order.

## Check Pod DNS Names

```bash
kubectl exec web-0 -- nslookup web-0.web
kubectl exec web-1 -- nslookup web-1.web
```
Each pod is reachable via:
```
<statefulset-name>-<ordinal>.<service-name>
Example: web-0.web, web-1.web
```

## Verify Volume Data Integrity

Create a test file in web-0:
```bash
kubectl exec web-0 -- /bin/sh -c "echo web-0 > /usr/share/nginx/html/index.html"
```
Delete and recreate pod:
```bash
kubectl delete pod web-0
```
Check if data is still present after pod recreation:
```bash
kubectl exec web-0 -- cat /usr/share/nginx/html/index.html
```
✅ You should see “web-0”, confirming volume persistence.

## Scale StatefulSet

```bash
kubectl scale statefulset web --replicas=5
kubectl get pods -l app=nginx
```
Then scale down:
```bash
kubectl scale statefulset web --replicas=2
kubectl get pods
```
__Observe:__

* Pods are terminated in reverse order (e.g., web-4, web-3 first)
* Volumes (PVCs) for terminated pods are retained unless manually deleted

## Cleanup
```bash
kubectl delete statefulset web
kubectl delete svc web
kubectl delete pvc -l app=nginx
kubectl delete sc local-storage
```


[Main](../README.md)
---