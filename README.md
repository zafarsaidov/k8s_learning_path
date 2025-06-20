# Kubernetes Learning Path

## Day 1: Intro to Containerization & Kubernetes
- [Linux namespaces & cgroups](./1/ns_cgroups.md)
- [Docker basics](./1/docker.md)
- [Kubernetes architecture, control plane & node components](./1/arch.md)

## Day 2: kubectl & Core Concepts
- [Installing kubectl & accessing clusters](./2/kubectl.md)
- [Namespaces, contexts, basic kubectl usage](https://kubernetes.io/docs/reference/kubectl/quick-reference/#:~:text=kubectl%20%2DA-,Kubectl%20context%20and%20configuration,-Set%20which%20Kubernetes)
- [Understanding resources and object metadata](https://kubernetes.io/docs/reference/kubectl/quick-reference/#:~:text=special%2Duser%3ANoSchedule-,Resource%20types,-List%20all%20supported)

## Day 3: Pods, Deployments & Manifests
- Pods, ReplicaSets, Deployments
- Labels, selectors
- YAML structure and object creation

## Day 4: Services & Networking (Part 1)
- ClusterIP, NodePort, LoadBalancer
- CoreDNS and service discovery

## Day 5: Services & Networking (Part 2)
- Ingress Controllers and ingress rules
- Network Policies
- CNI plugins overview

## Day 6: Practice – Build, Push, Deploy
- Build Docker image for a simple app
- Push to container registry
- Deploy app with Deployment + Service + Ingress

## Day 7: Storage & Stateful Applications
- Volumes, PVs, PVCs
- StorageClasses
- StatefulSets, headless services
- DaemonSets: intro, use cases (logging agents, monitoring)

## Day 8: Configuration & Secrets
- ConfigMaps and Secrets
- Environment variables and mounted files
- Secure and dynamic configuration

## Day 9: Practice – Volumes & Secrets
- Deploy app with persistent volume
- Mount secrets/configs
- Deploy stateful app with PVCs

## Day 10: Health Checks & Basic Scheduling
- Liveness, Readiness, Startup probes
- NodeSelector, Affinity, Anti-affinity

## Day 11: Advanced Scheduling
- Taints and Tolerations
- Resource requests and limits
- Pod priority and preemption
- CronJobs: scheduling jobs, retries, concurrency, cleanup

## Day 12: Security in Kubernetes
- RBAC: roles, bindings
- Service accounts
- PodSecurityContext and security best practices

## Day 13: Practice – Probes, Scheduling & Security
- Add probes to deployments
- Apply resource limits and affinity rules
- Apply RBAC policies and PodSecurityContext

## Day 14: Helm Basics
- Installing Helm and adding repos
- Installing community charts
- Chart release lifecycle (install, upgrade, rollback)

## Day 15: Helm Templating
- Chart structure and templating logic
- values.yaml, template helpers
- conditionals, loops, and hooks

## Day 16: Practice – Deploy with Helm
- Create a custom Helm chart
- Use templates and values.yaml
- Release and manage app via Helm

## Day 17: Cluster Setup with Kubespray & Ops
- Deploy a production-ready cluster with Kubespray
- Inventory setup and Ansible run
- Configure kubeconfig access
- etcd Backup: take a snapshot, restore manually
- Review system DaemonSets, CronJobs, and addons

## Day 18: Project
- Deploy multi-component app
- Use Helm, probes, configs, PVCs, security, scheduling


