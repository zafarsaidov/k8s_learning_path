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
- [Pods](./3/pods.md), [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/), [Deployments](./3/deployments.md)
- [Labels, selectors](./3/labels.md)
- [YAML structure and object creation](./3/yaml.md)

## Day 4: Services & Networking (Part 1)
- [ClusterIP, NodePort, LoadBalancer](./4/service.md)
- [CoreDNS and service discovery](./4/coredns.md)

## Day 5: Services & Networking (Part 2)
- [Ingress Controllers and ingress rules](./5/ingress.md)
- [Network Policies](./5/policy.md)
- CNI plugins overview

## Day 6: Practice – Build, Push, Deploy
- Build Docker image for a simple app
- Push to container registry
- [Deploy app with Deployment + Service + Ingress](./6/tasks.md)

## Day 7: Storage & Stateful Applications
- [Volumes, PVs, PVCs](./7/volume.md)
- StorageClasses
- [StatefulSets, headless services](./7/statefulset.md)
- [DaemonSets: intro, use cases (logging agents, monitoring)](./7/daemonset.md)

## Day 8: Configuration & Secrets
- Environment variables and mounted files
- [ConfigMaps and Secrets](./8/secrets.md)

## Day 9: Practice – Volumes & Secrets
- Deploy app with persistent volume
- Mount secrets/configs
- [Deploy stateful app with PVCs](./9/tasks.md)

## Day 10: Health Checks & Basic Scheduling
- [Liveness, Readiness, Startup probes](./10/probes.md)
- [NodeSelector, Affinity, Anti-affinity](./10/scheduler.md)

## Day 11: Advanced Scheduling
- [Taints and Tolerations](/11/taint.md)
- [Resource requests and limits](./11/limits.md)
- [Pod priority and preemption](/11/priority.md)
- [CronJobs: scheduling jobs, retries, concurrency, cleanup](/11/crons.md)

## Day 12: Security in Kubernetes
- RBAC: roles, bindings
- Service accounts
- PodSecurityContext and security best practices

## Day 13: Practice – Probes, Scheduling & Security
- Add probes to deployments
- Apply resource limits and affinity rules
- [Apply RBAC policies and PodSecurityContext](./13/practice.md)

## Day 14: Helm Basics
- [Installing Helm and adding repos](./14/install.md)
- [Installing community charts](./14/community.md)
- [Chart release lifecycle (install, upgrade, rollback)](./14/life.md)

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


