[Main](../README.md)
---

# Kubernetes Practice Lab: Helm Installation and Usage

This independent lab introduces Helm, the Kubernetes package manager. Students will learn how to install Helm, add repositories, explore charts, and manage applications via Helm.

---

## 🎯 Objectives

* Install Helm
* Add popular Helm chart repositories
* Explore available charts
* Install and manage a Helm release (Redis or RabbitMQ)
* Uninstall a release and clean up
* Practice common Helm commands

---

## ⛳ Prerequisites

* A running Kubernetes cluster (e.g., Minikube, Kind, cloud-based)
* `kubectl` installed and configured
* Internet access for downloading Helm binaries and Helm chart repositories

---

## 🔧 Step 1: Install Helm CLI

### MacOS

```bash
brew install helm
```

### Linux

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

### Verify Installation

```bash
helm version
```

---

## 📦 Step 2: Add Helm Repositories

Add some common and widely-used repositories:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

Check added repos:

```bash
helm repo list
```

---

## 🔍 Step 3: Explore Charts

Search for charts by name:

```bash
helm search repo redis
helm search repo rabbitmq
```

Inspect a chart’s default values:

```bash
helm show values bitnami/redis
```

---

## 🚀 Step 4: Install a Helm Chart

### Example: Install Redis with default values

```bash
helm install my-redis bitnami/redis \
  --namespace helm-lab \
  --create-namespace
```

Check deployment status:

```bash
kubectl get all -n helm-lab
```

List Helm releases:

```bash
helm list -n helm-lab
```

---

## ⚙️ Step 5: Upgrade and Uninstall Helm Release

### Upgrade (e.g., change replica count or set password)

```bash
helm upgrade my-redis bitnami/redis \
  --namespace helm-lab \
  --set architecture=replication \
  --set replica.replicaCount=2
```

### Uninstall

```bash
helm uninstall my-redis -n helm-lab
```

Delete the namespace:

```bash
kubectl delete ns helm-lab
```

---

## 🔁 Step 6: Frequently Used Helm Commands

### Preview rendered manifests

```bash
helm template my-redis bitnami/redis > rendered.yaml
```

### Dry-run an install

```bash
helm install --dry-run --debug my-redis bitnami/redis
```

### Show default values

```bash
helm show values bitnami/redis
```

### Show chart README

```bash
helm show readme bitnami/redis
```

### List all installed charts

```bash
helm list -A
```

### Get values of a release

```bash
helm get values my-redis -n helm-lab
```

### Save custom values to file and install

```bash
helm show values bitnami/redis > myvalues.yaml
# edit myvalues.yaml
helm install custom-redis bitnami/redis -f myvalues.yaml --namespace helm-lab
```

---

## 🧪 Optional Tasks

1. Install RabbitMQ using Helm with custom values
2. Use `helm diff` (plugin) to preview changes before upgrade
3. Practice rollback:

   ```bash
   helm rollback my-redis 1 -n helm-lab
   ```

---

## ✅ Conclusion

You’ve successfully used Helm to:

* Install the CLI
* Add and update repositories
* Install, upgrade, and uninstall charts
* Customize deployments with Helm values
* Practice common Helm commands




[Main](../README.md)
---