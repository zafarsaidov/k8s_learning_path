[Main](../README.md)
---

# 🧪 Kubernetes Lab: etcd Backup and Restore

🎯 Objectives

Students will:
* Locate etcd data directory and certificate paths
* Take a manual snapshot of etcd data
* Simulate a failure by deleting data
* Manually restore the snapshot
* Restart the control plane and validate recovery

🛑 WARNING: This lab is intended for non-production clusters only (lab VMs, kind, or kubeadm clusters).

---

🧰 Prerequisites
* You must be able to access the control plane node (e.g., via SSH or root shell)
* etcdctl must be installed (usually included with kubeadm or can be downloaded)
* You must know the location of:
* --data-dir used by etcd
* Certificate files used by etcd (ca.crt, server.crt, server.key)
* Cluster must be kubeadm-based or similar, where etcd runs as a static pod

---

## 🔍 Step 1: Locate etcd Configuration

✅ Task:

Check the etcd manifest to find data directory and certificate paths.
```bash
cat /etc/kubernetes/manifests/etcd.yaml
```
🔍 Look for:
* --data-dir=/var/lib/etcd
* --cert-file, --key-file, --trusted-ca-file

Make note of:
```bash
ETCD_DATA_DIR=/var/lib/etcd
ETCD_CERT=/etc/kubernetes/pki/etcd/server.crt
ETCD_KEY=/etc/kubernetes/pki/etcd/server.key
ETCD_CA=/etc/kubernetes/pki/etcd/ca.crt
```
You’ll use these in the etcdctl command.

## 🛠 Step 2: Set Environment Variables

```bash
export ETCDCTL_API=3
export ETCDCTL_CACERT=$ETCD_CA
export ETCDCTL_CERT=$ETCD_CERT
export ETCDCTL_KEY=$ETCD_KEY
```
Optional: Set endpoint (default for static pod):
```bash
export ETCDCTL_ENDPOINTS=https://127.0.0.1:2379
```

---

## 💾 Step 3: Take a Snapshot
```bash
etcdctl snapshot save /root/etcd-backup.db
```
Expected output:
```txt
Snapshot saved at /root/etcd-backup.db
```
Check snapshot:
```bash
etcdctl snapshot status /root/etcd-backup.db
```

---

## 💣 Step 4: Simulate Disaster (Data Deletion)

⚠️ This deletes etcd data directory. Do NOT do this on production.
```bash
mv /var/lib/etcd /var/lib/etcd.bak
mkdir /var/lib/etcd
```
Restart kubelet if needed:
```bash
systemctl restart kubelet
```
Run:
```bash
kubectl get pods
```
You’ll see that the Kubernetes API server no longer works because etcd is empty.

## 🔄 Step 5: Restore from Snapshot
```bash
etcdctl snapshot restore /root/etcd-backup.db \
  --data-dir=/var/lib/etcd
```
This recreates a fresh etcd data directory from the snapshot.

---

## 🔁 Step 6: Restart etcd Pod

Since etcd runs as a static pod, kubelet will automatically detect restored data.

Restart kubelet:
```bash
systemctl restart kubelet
```
Wait a few seconds and check:
```bash
docker ps | grep etcd       # or crictl ps | grep etcd
kubectl get nodes
kubectl get pods -A
```
✅ If the restore was successful, the cluster should be back to normal.

---

## 📋 Step 7: Validation
* Can you query cluster state?
```bash
kubectl get all -A
```
* Try making some changes:
```bash
kubectl create ns test-backup
kubectl get ns
```


---

## 🧹 Step 8: Cleanup (Optional)
```bash
rm -rf /var/lib/etcd.bak
rm /root/etcd-backup.db
kubectl delete ns test-backup
```

---

## 📘 Summary

| Step |	Description |
| --- | --- |
| 1 |	Locate etcd certs and data dir |
| 2 |	Set etcdctl environment |
| 3 |	Take a snapshot |
| 4 |	Simulate etcd failure |
| 5 |	Restore snapshot |
| 6 |	Restart etcd |
| 7 |	Validate cluster recovery |



[Main](../README.md)
---