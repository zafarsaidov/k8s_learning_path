# 🧪 Kubernetes Practical Lesson – Persistent Storage, Configs, Secrets, Stateful Apps

## Task 1: Deploy an Application with a Persistent Volume
* Create a PersistentVolume using hostPath or a local path
* Create a PersistentVolumeClaim that binds to the PV
* Deploy a Pod that uses the PVC to mount a volume at /data
* Inside the container, create a file to test persistence



## Task 2: Use Secrets and ConfigMaps
* Create a ConfigMap with the app’s configuration (e.g., APP_MODE=prod)
* Create a Secret with a dummy username and password
* Mount both into the application:
* As environment variables
* As volume files



## Task 3: Deploy a Stateful Application with VolumeClaimTemplates
* Create a StorageClass (local or manual)
* Create a StatefulSet with 2 replicas for a simple app (like Redis or a sample NGINX)
* Mount separate volumes to each pod using PVC templates
* Validate that each pod gets its own persistent storage
* Scale down and back up to observe pod naming and volume persistence

## 🧠 Extra Tasks
* Mount PVC as read-only
* Make StatefulSet use dynamic provisioning (StorageClass + provisioner like local-path)
* Inject different config/secrets per pod (via Downward API or projected volumes)
* Backup volume data to a separate pod via initContainers