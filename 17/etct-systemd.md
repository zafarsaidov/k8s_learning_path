[Main](../README.md)
---

# 🧪 Kubernetes Lab: etcd Backup and Restore (Systemd-Based etcd)

🎯 Objectives

Students will:
* Understand the etcd systemd service setup
* Locate configuration files and certificates
* Take a backup (snapshot) of etcd
* Simulate failure by stopping etcd and removing data
* Restore etcd from snapshot
* Validate cluster recovery

---

🛠 Prerequisites
* A Linux machine with etcd installed and running via systemd
* Access to sudo and the etcdctl binary (version 3)
* Kubernetes control plane must depend on the systemd-based etcd service
* You must know or be able to locate:
* etcd data directory (usually /var/lib/etcd)
* etcd systemd service file (e.g., /etc/systemd/system/etcd.service)
* paths to cert, key, and ca files


---

## 🔍 Step 1: Inspect etcd systemd Service
```bash
systemctl status etcd
```
View full configuration:
```bash
cat /etc/systemd/system/etcd.service
```
Look for:
```bash
ExecStart=/usr/local/bin/etcd \
  --name <node-name> \
  --data-dir=/var/lib/etcd \
  --listen-client-urls=https://127.0.0.1:2379 \
  --advertise-client-urls=https://127.0.0.1:2379 \
  --cert-file=/etc/etcd/pki/server.crt \
  --key-file=/etc/etcd/pki/server.key \
  --trusted-ca-file=/etc/etcd/pki/ca.crt
```
✍️ Record:
```bash
ETCD_DATA_DIR=/var/lib/etcd
ETCD_CERT=/etc/etcd/pki/server.crt
ETCD_KEY=/etc/etcd/pki/server.key
ETCD_CA=/etc/etcd/pki/ca.crt
ETCD_ENDPOINT=https://127.0.0.1:2379
```

---

## 🔐 Step 2: Set Environment for etcdctl
```bash
export ETCDCTL_API=3
export ETCDCTL_CACERT=$ETCD_CA
export ETCDCTL_CERT=$ETCD_CERT
export ETCDCTL_KEY=$ETCD_KEY
export ETCDCTL_ENDPOINTS=$ETCD_ENDPOINT
```
Verify access:
```bash
etcdctl member list
```

---

## 💾 Step 3: Take a Snapshot
```bash
sudo etcdctl snapshot save /root/etcd-snapshot.db
```
Check snapshot status:
```bash
etcdctl snapshot status /root/etcd-snapshot.db
```

---

## 💣 Step 4: Simulate Failure

🛑 Stop the etcd service:
```bash
sudo systemctl stop etcd
```
Backup current data dir and wipe it:
```bash
sudo mv /var/lib/etcd /var/lib/etcd.bak
sudo mkdir /var/lib/etcd
```
Start the etcd service:
```bash
sudo systemctl start etcd
```
🧪 Try this (expect failure):
```bash
kubectl get nodes
```

---

## 🔄 Step 5: Restore Snapshot

Restore with correct data dir:
```bash
sudo etcdctl snapshot restore /root/etcd-snapshot.db \
  --data-dir=/var/lib/etcd
```
If the original etcd data was created with a name (e.g., --name my-node), you must rename the restored dir:
```bash
sudo mv /var/lib/etcd/default.etcd /var/lib/etcd
```

---

## 🔁 Step 6: Restart etcd
```bash
sudo systemctl start etcd
```
Verify:
```bash
systemctl status etcd
```
Wait a few seconds, then test Kubernetes API:
```bash
kubectl get nodes
kubectl get pods -A
```

---

## ✅ Step 7: Validation

Create new test object:
```bash
kubectl create ns restored-test
kubectl get ns
```
You may also check logs:
```bash
journalctl -u etcd -f
```

---

## 🧹 Step 8: Cleanup (Optional)
```bash
sudo rm -rf /var/lib/etcd.bak
sudo rm /root/etcd-snapshot.db
kubectl delete ns restored-test
```

---

## 📘 Summary

| Step |	Description |
| --- | --- |
| 1 |	Inspect systemd service |
| 2 |	Export credentials |
| 3 |	Save snapshot |
| 4 |	Stop etcd and remove data |
| 5 |	Restore snapshot |
| 6 |	Restart etcd |
| 7 |	Verify recovery |
| 8 |	Cleanup |


---

✅ By the end of this lab, students will be confident working with etcd in systemd-based clusters — including inspecting, backing up, and manually restoring data.


[Main](../README.md)
---