[Main](../README.md)
---

# Kubernetes Practice Lab: Helm Chart Release Lifecycle (Using Bitnami RabbitMQ)

This lab guides students through the Helm chart release lifecycle: install → upgrade → rollback → uninstall, using the **Bitnami RabbitMQ** chart.

---

## 🎯 Objectives

* Understand the Helm chart release lifecycle
* Practice installing, upgrading, rolling back, and uninstalling
* Use `helm status`, `helm history`, and `helm diff`

---

## 📦 Step 1: Add Bitnami Helm Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

---

## 🔍 Step 2: Explore Chart & Default Values

Show chart info:

```bash
helm show chart bitnami/rabbitmq
```

Save and edit values:

```bash
helm show values bitnami/rabbitmq > rabbitmq-values.yaml
```

(Optional) Enable metrics and ingress in the values file:

```yaml
metrics:
  enabled: true

ingress:
  enabled: true
  hostname: rabbit.local
```

---

## 🚀 Step 3: Install the Chart

```bash
helm install rabbitmq bitnami/rabbitmq \
  --namespace rabbit-lab \
  --create-namespace \
  -f rabbitmq-values.yaml
```

Watch rollout:

```bash
kubectl get pods -n rabbit-lab -w
```

---

## 📈 Step 4: Upgrade the Release

Make a small change in `rabbitmq-values.yaml`, e.g.:

```yaml
extraEnvVars:
  - name: MY_CUSTOM_ENV
    value: "initial-value"
```

Then upgrade:

```bash
helm upgrade rabbitmq bitnami/rabbitmq -n rabbit-lab -f rabbitmq-values.yaml
```

Verify the change:

```bash
kubectl describe pod -n rabbit-lab -l app.kubernetes.io/name=rabbitmq
```

---

## 🕰️ Step 5: View History and Roll Back

List history:

```bash
helm history rabbitmq -n rabbit-lab
```

Rollback to revision 1:

```bash
helm rollback rabbitmq 1 -n rabbit-lab
```

Verify rollback:

```bash
kubectl get all -n rabbit-lab
```

---

## 🧼 Step 6: Uninstall the Release

```bash
helm uninstall rabbitmq -n rabbit-lab
```

Optionally clean up namespace:

```bash
kubectl delete ns rabbit-lab
```

---

## 🔍 Step 7: Optional Tools for Better Lifecycle Management

Install Helm Diff plugin:

```bash
helm plugin install https://github.com/databus23/helm-diff
```

Use to preview upgrade differences:

```bash
helm diff upgrade rabbitmq bitnami/rabbitmq -n rabbit-lab -f rabbitmq-values.yaml
```

---

## ✅ Summary

You’ve practiced the full Helm chart release lifecycle:

* Install with custom values
* Upgrade and confirm changes
* Rollback to previous revision
* Track history of deployments
* Use advanced tools like Helm Diff

🛠 This lab helps reinforce Helm's power for safe, trackable, and repeatable application management in Kubernetes.


[Main](../README.md)
---