[Main](../README.md)
---

# kubectl

## Install kubectl

[Documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) 

kubectl is the command-line tool used to interact with Kubernetes clusters. It allows users to manage and monitor applications, inspect cluster resources, and more. Some common commands include listing resources, describing resources, and creating and deleting resources. 

## Commonly used kubectl commands: 
### kubectl get:

Retrieves and displays information about one or more resources. 

```bash
kubectl get pods: Lists all pods. 
kubectl get nodes: Lists all nodes. 
kubectl get services: Lists all services. 
kubectl get deployments: Lists all deployments. 
kubectl get namespaces: Lists all namespaces. 
kubectl get all --all-namespaces: Lists all resources across all namespaces. 
```

### kubectl describe:
Displays detailed information about a specific resource. 
```bash
kubectl describe pod <pod_name>: Describes a specific pod. 
kubectl describe node <node_name>: Describes a specific node. 
kubectl describe service <service_name>: Describes a specific service. 
```

### kubectl create:
Creates a new resource. 

```bah
kubectl create deployment <deployment_name> --image=<image_name>: Creates a new deployment. 
kubectl create namespace <namespace_name>: Creates a new namespace. 
```

### kubectl apply:
Applies changes to resources based on a YAML or JSON file, or a directory containing such files. 
```bash
kubectl apply -f <filename>.yaml: Applies changes from the specified file. 
kubectl apply -k <directory_name>: Applies changes from the specified directory, using Kustomize if present. 
```

### kubectl delete:
Deletes resources. 
```bash
kubectl delete pod <pod_name>: Deletes a pod. 
kubectl delete service <service_name>: Deletes a service. 
kubectl delete deployment <deployment_name>: Deletes a deployment. 
kubectl delete namespace <namespace_name>: Deletes a namespace. 
```

### kubectl logs:
Retrieves logs from a pod. 
```bash
kubectl logs <pod_name>: Retrieves logs for a pod. 
kubectl logs -f <pod_name>: Streams logs in real-time. 
kubectl logs --since=1h <pod_name>: Retrieves logs from the last hour. 
```

### kubectl exec:
Executes a command inside a container. 
```bash
kubectl exec -it <pod_name> -- bash: Opens an interactive bash shell inside the pod. 
```

### kubectl port-forward:
Forwards a local port to a port on a pod or service. 
```bash
kubectl port-forward pod/<pod_name> <local_port>:<container_port>: Forwards a local port to a container port. 
```

### kubectl scale:
Scales the number of replicas in a deployment. 
```bash
kubectl scale deployment/<deployment_name> --replicas=<replica_count>: Scales the deployment. 
```

### kubectl rollout:
Used for managing deployments and their rollouts. 
```bash
kubectl rollout status deployment/<deployment_name>: Checks the status of a deployment's rollout. 
kubectl rollout history deployment/<deployment_name>: Shows the history of rollouts for a deployment. 
kubectl rollout restart deployment/<deployment_name>: Restarts a deployment. 
```

### kubectl version:
Shows the client and server versions of Kubernetes. 

### kubectl cluster-info:
Shows information about the Kubernetes cluster.

### kubectl config:
Manages Kubernetes configuration files. 
```bash
kubectl config get-contexts: Lists available contexts in your kubeconfig. 
kubectl config use-context <context_name>: Sets the current context. 
```


[Main](../README.md)
---