[Main](../README.md)
---

# 🧪 Final Kubernetes Practice Lab: Build and Harden a Complete Project

## 🎯 Objective:

Deploy and improve a simple application step by step, practicing every major Kubernetes feature: deployment, storage, probes, scheduling, RBAC, networking, security, Helm, and more.

## 📦 Project Idea:

A **Flask** application with PostgreSQL + Redis backend.

* `/` - returns "Hello, World!"
* `/db` - adds/selects timestamp from PostgreSQL
* `/cache` - increments count in Redis

---

## 🏗 Step 1: Build and Push App Image

### 🔧 Task:

1. Write a simple Flask app with 3 endpoints
2. Create Dockerfile
3. Build and push image to DockerHub

✅ Verify: Run image locally with Docker

---

## 🚀 Step 2: Deploy Basic App

### 🔧 Task:

1. Deploy PostgreSQL (static manifest, with PVC)
2. Deploy Redis (PVC optional)
3. Deploy Flask app Deployment and ClusterIP service

✅ Verify: Port-forward Flask and test all endpoints

---

## 🔍 Step 3: Add Health Checks and Init Containers

### 🔧 Task:

1. Add readiness and liveness probes to app
2. Add initContainers to wait for PostgreSQL and Redis

✅ Verify: Simulate Redis delay, see initContainers behavior

---

## 🗃 Step 4: Inject Config via ConfigMap and Secrets

### 🔧 Task:

1. Move DB credentials and Redis host to ConfigMap + Secret
2. Inject them via env vars and envFrom

✅ Verify: Describe pod and confirm env vars injected

---

## 📦 Step 5: Add Logging Sidecar

### 🔧 Task:

1. Use `emptyDir` shared volume for logs
2. App writes to `/logs/app.log`, sidecar tails it

✅ Verify: See logs from sidecar container

---

## ⚖ Step 6: Add Affinity and Tolerations

### 🔧 Task:

1. Add `nodeAffinity` to run app on labeled nodes
2. Add `podAntiAffinity` so Redis and Flask don't co-locate
3. Taint one node and use toleration

✅ Verify: Check pod placement

---

## 📈 Step 7: Set Resource Limits

### 🔧 Task:

1. Add CPU & memory `requests` and `limits` for all pods
2. Add `LimitRange` and `ResourceQuota` to namespace

✅ Verify: Run `kubectl describe` on pods and ns

---

## 🔐 Step 8: Add RBAC and ServiceAccount

### 🔧 Task:

1. Create SA for Flask app
2. Create Role/RoleBinding for pod and svc read

✅ Verify: Use `kubectl auth can-i`

---

## 🔒 Step 9: Apply PodSecurityContext

### 🔧 Task:

1. Set `runAsUser`, `readOnlyRootFilesystem`, drop capabilities
2. Add `fsGroup` and check volume ownership

✅ Verify: `kubectl exec` into pod, run `id`

---

## 🧠 Step 10: StatefulSet for PostgreSQL

### 🔧 Task:

1. Convert Postgres Deployment to StatefulSet
2. Use PVC with `volumeClaimTemplates`
3. Use `headless` service

✅ Verify: Pod name DNS works, PVCs are retained after delete

---

## ⏰ Step 11: Add CronJob

### 🔧 Task:

1. Create CronJob that hits `/db` every 5 min
2. Set history limit and concurrencyPolicy

✅ Verify: List jobs and check logs

---

## 🎛 Step 12: Add Ingress

### 🔧 Task:

1. Create Ingress object for app
2. Configure IngressClass and DNS if needed

✅ Verify: Access app via browser or curl

---

## 📦 Step 13: Package Everything into Helm Chart

### 🔧 Task:

1. Create chart structure
2. Template Deployment, Service, Ingress, Config, Secret
3. Add values.yaml and document it

✅ Verify: `helm install` and access app

---

## 🔄 Step 14: Upgrade and Rollback via Helm

### 🔧 Task:

1. Change title text or version of app
2. Bump Helm version and upgrade
3. Perform `helm rollback`

✅ Verify: See updated and restored behavior

---

## 📁 Step 15: Push Helm Chart to GitLab Registry

### 🔧 Task:

1. Create `.gitlab-ci.yml` pipeline
2. Package chart and push to registry
3. Use private GitLab Helm repo for install

✅ Verify: `helm repo add` and install by version

---

## 🎓 Bonus Practices:

* Simulate pod failure and observe probes/restart
* Add priorityClass and test preemption
* Create NetworkPolicy for PostgreSQL
* Backup and restore etcd snapshot (in test cluster)
* Use port-forward, logs, exec, top, events effectively
* Use `kubectl auth can-i`, describe, explain

---

✅ After completing this lab, students will have:

* Deployed and managed a real app in Kubernetes
* Used 90% of daily used features
* Practiced all core topics in context
* Reinforced Helm, YAML, and GitOps principles


[Main](../README.md)
---