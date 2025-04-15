# Kubernetes Learning Path

1. Introduction to Kubernetes: Understand what Kubernetes is, its use cases, and its architecture.
* What is Kubernetes?
* Kubernetes vs Docker
* Core components of Kubernetes (Control Plane, Nodes)
* Kubernetes architecture overview
* Cluster setup options (Minikube, bare metal and managed kubernetes)

2. Working with Kubernetes CLI (kubectl): Learn how to interact with Kubernetes using kubectl.
* Installing and configuring kubectl
* Connecting to a cluster
* Understanding and using kubectl get, describe, logs, and exec
* Namespaces and context switching

3. Pods and Deployments: Understand how to run containers in Kubernetes using Pods and Deployments.
* What is a Pod?
* Multi-container Pods
* ReplicaSets and Deployments
* Rolling updates and rollbacks
* YAML manifest structure

4. Services and Networking: Learn how services expose applications and how networking works in Kubernetes.
* ClusterIP, NodePort, LoadBalancer, and ExternalName services
* DNS in Kubernetes
* Ingress and Ingress Controllers
* Network Policies
* CNI plugins overview (Calico, Flannel)

5. Persistent Volumes and Persistent Storage: Handle stateful applications and persistent data in Kubernetes.
* Volumes, emptyDir, hostPath
* PersistentVolume (PV) and PersistentVolumeClaim (PVC)
* StorageClasses and dynamic provisioning
* StatefulSets overview

6. Configuration and Secrets Management: Use ConfigMaps and Secrets for app configuration.
* Environment variables and volume mounts
* Creating and mounting ConfigMaps
* Creating and using Secrets
* Managing sensitive data securely

7. Health Checks and Probes: Understand how Kubernetes monitors app health.
* Liveness and Readiness Probes
* Startup probes
* Debugging failed probes

8. Kubernetes Scheduling and Affinity: Learn how Kubernetes schedules Pods and how to influence placement.
* Scheduler basics
* Node Affinity and Anti-affinity
* Taints and Tolerations
* Resource requests and limits

9. Scaling and Autoscaling: Scale applications manually and automatically.
* Horizontal Pod Autoscaler (HPA)
* Vertical Pod Autoscaler (VPA)
* Cluster Autoscaler
* Resource metrics

10. Security in Kubernetes: Implement security best practices in the cluster.
* RBAC (Role-Based Access Control)
* Service accounts
* Network policies
* SecurityContext and PodSecurity Standards
* Using tools like kube-bench and kube-hunter

11. Logging and Monitoring: Monitor applications and collect logs.
* Using kubectl logs and log forwarding
* Centralized logging with Fluentd, Loki, or Elasticsearch
* Metrics server, Prometheus, and Grafana
* Kubernetes dashboard

12. CI/CD with Kubernetes: Integrate CI/CD pipelines with Kubernetes deployments.
* GitOps vs traditional CI/CD
* Deploying with GitLab CI, Jenkins, or ArgoCD
* Image building and scanning
* Canary and Blue/Green deployments

14. Helm Basics: Learn to use Helm to manage Kubernetes applications.
* What is Helm?
* Helm architecture and installation
* Installing and upgrading charts
* Working with official charts (Bitnami, etc.)
* Chart repository management

14. Helm Templating and Chart Development: Create and customize Helm charts.
* Helm chart structure and templating syntax
* Values files and parameterization
* Using functions and conditionals
* Creating reusable templates and subcharts
* Helm hooks and lifecycle events

15. Kubernetes in Production: Understand how to run and manage Kubernetes in production environments.
* Multi-tenancy and resource quotas
* Backup and disaster recovery (backup etcd)
* High availability setup
* Upgrading clusters safely

16. Final Project & Capstone: Apply all learned concepts in a real-world project.
* Design a microservice architecture
* Build and deploy the app using Helm
* Configure scaling, monitoring, and logging
* Present and demo the solution
