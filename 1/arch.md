[Main](../README.md)
---

# Kubernetes Components


## CONTROL PLANE COMPONENTS

These components make global decisions about the cluster (like scheduling), and detect and respond to cluster events (like starting a new pod when a deployment’s replicas field is unsatisfied).

---
1. kube-apiserver
* 📌 Role: The front-end (entry point) for the Kubernetes control plane.
* 📄 Function:
	* Exposes the Kubernetes API (REST interface).
	* Accepts commands from kubectl, other control plane components, or clients.
	* Validates and processes REST requests, updates etcd, and communicates with other components.

✅ Highly available — you can run multiple kube-apiserver instances.

---

2. etcd
* 📌 Role: The key-value store used by Kubernetes to store all cluster data.
* 📄 Function:
	* Stores the state of the entire cluster: nodes, pods, configs, secrets, etc.
	* Highly consistent and reliable.
	* It’s the single source of truth for Kubernetes.

💡 Think of etcd as the “database” of Kubernetes.

---

3. kube-scheduler
* 📌 Role: Schedules pods onto nodes.
* 📄 Function:
	* Watches for unscheduled pods.
	* Selects a node based on resource availability, taints, affinity rules, etc.
	* Writes its decision (node assignment) back to the API server.

✅ Only assigns pods to nodes, doesn’t actually start them.

---

4. kube-controller-manager
* 📌 Role: Runs controller processes.
* 📄 Function:
	* A controller is a loop that watches the desired state in the cluster and ensures the actual state matches.
	* Manages:
	* Node controller
	* Replication controller
	* Deployment controller
	* Namespace controller
	* Service account controller

💬 All these controllers are bundled into a single binary (kube-controller-manager), running as one process.

---

5. cloud-controller-manager (optional, for cloud-based clusters)
* 📌 Role: Manages cloud-specific control logic.
* 📄 Function:
	* Interacts with cloud provider APIs.
	* Manages:
	* Node lifecycle in the cloud (e.g., adding/removing VMs)
	* Load balancers
	* Storage volumes

☁️ Required only when running Kubernetes on a cloud like AWS, GCP, Azure, etc.

---

## NODE (WORKER) COMPONENTS

These run on every node (physical or virtual machine) in the cluster and are responsible for managing containers.

---

6. kubelet
* 📌 Role: The agent that runs on each node.
* 📄 Function:
	* Receives instructions from the API server.
	* Ensures containers in pods are running and healthy.
	* Watches the PodSpecs assigned to its node and reports back status.

💬 It does not create containers directly — it talks to container runtimes (like containerd or Docker).

---

7. kube-proxy
* 📌 Role: Manages networking for pods and services.
* 📄 Function:
	* Maintains network rules on each node.
	* Implements service discovery and load balancing.
	* Forwards traffic to the correct pod IPs.

🧠 Uses iptables, ipvs, or eBPF under the hood to route traffic.

---

8. Container Runtime
* 📌 Role: Runs containers on each node.
* 📄 Function:
	* Starts, stops, and manages containers.
	* Interfaces with the kubelet.

🔧 Examples:
* containerd ✅ (default in many distributions)
* CRI-O
* Docker (deprecated for production use with Kubernetes ≥ 1.24)

---

## OTHER IMPORTANT INTERNAL COMPONENTS


---
9. CoreDNS
* 📌 Role: Internal DNS service for the cluster.
* 📄 Function:
	* Resolves internal service names (e.g., my-service.default.svc.cluster.local) into IPs.
	* Allows service discovery.



10. Add-ons (Optional but Common)
* Add functionality like monitoring, logging, dashboard, etc.
* Metrics-server – collects resource usage metrics.
* Kubernetes Dashboard – web UI.
* Flannel / Calico / Cilium – for container networking (CNI plugins).
* Ingress Controller – like NGINX Ingress, to manage external access.


[Main](../README.md)
---