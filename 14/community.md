[Main](../README.md)
---

# Kubernetes Practice Lab: Installing Community Helm Charts (Kubernetes Dashboard Example)

This lab demonstrates how to discover, configure, and install a real-world community Helm chart. We'll use the official Kubernetes Dashboard chart as our main example.

---

## 🎯 Objectives

* Search for official/community Helm charts
* Read chart documentation and understand values
* Install the Kubernetes Dashboard using Helm
* Configure access (ServiceAccount + token)
* Practice inspecting Helm releases

---

## ⛳ Prerequisites

* Helm installed (`helm version`)
* Running Kubernetes cluster
* `kubectl` configured

---

## 📦 Step 1: Add Community Helm Repository

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm repo update
```

Verify the chart is available:

```bash
helm search repo kubernetes-dashboard
```

---

## 🔍 Step 2: Explore Chart Values

Check default values:

```bash
helm show values kubernetes-dashboard/kubernetes-dashboard > dashboard-values.yaml
```

Edit `dashboard-values.yaml` if necessary. Example changes:

```yaml
service:
  type: NodePort
  nodePort: 30001

metricsScraper:
  enabled: true

protocolHttp: true

extraArgs:
  - --enable-skip-login
  - --disable-settings-authorizer
```

---

## 🚀 Step 3: Install the Chart

```bash
helm install k8s-dashboard kubernetes-dashboard/kubernetes-dashboard \
  --namespace k8s-dashboard \
  --create-namespace \
  -f dashboard-values.yaml
```

Wait for rollout:

```bash
kubectl get pods -n k8s-dashboard -w
```

---

## 🌐 Step 4: Access the Dashboard

Forward port locally:

```bash
kubectl port-forward svc/k8s-dashboard-kubernetes-dashboard 9090:80 -n k8s-dashboard
```

Open in browser: [http://localhost:9090](http://localhost:9090)

---

## 🔐 Step 5: Create ServiceAccount + Token (Optional Secure Access)

Create a new ServiceAccount:

```bash
kubectl create serviceaccount dashboard-admin -n k8s-dashboard
```

Bind to cluster-admin role:

```bash
kubectl create clusterrolebinding dashboard-admin \
  --clusterrole=cluster-admin \
  --serviceaccount=k8s-dashboard:dashboard-admin
```

Get token:

```bash
kubectl create token dashboard-admin -n k8s-dashboard
```

Use this token to log into the Dashboard.

---

## ⚙️ Step 6: Manage the Release

List Helm releases:

```bash
helm list -n k8s-dashboard
```

Upgrade example:

```bash
helm upgrade k8s-dashboard kubernetes-dashboard/kubernetes-dashboard -n k8s-dashboard -f dashboard-values.yaml
```

Uninstall:

```bash
helm uninstall k8s-dashboard -n k8s-dashboard
```

---

## ✅ Optional Tasks

* Modify values to use Ingress instead of NodePort
* Disable skip-login and enable proper RBAC
* Explore other charts from `kubernetes-dashboard` repo

---

## ✅ Summary

You’ve successfully:

* Added and searched for a community Helm chart
* Installed Kubernetes Dashboard with custom values
* Accessed it locally and configured access
* Managed the release using Helm commands

This experience prepares you to deploy real-world apps from the Helm ecosystem.



[Main](../README.md)
---