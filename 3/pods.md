[Main](../README.md)
---

## Pod Phases
Pod phases represent the current status of a pod in its lifecycle. The primary pod phases include:

### Pending
A pod is in the “Pending” phase when it has been accepted by the Kubernetes system but one or more of the containers have not been created yet. This could be due to scheduling issues or insufficient resources on the nodes.

__Scenario__: Deploy a pod that requests more resources than available on the node, leading to a “Pending” phase.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pending-pod
spec:
  containers:
  - name: busybox
    image: busybox
    resources:
      requests:
        memory: "10Gi" # Request more memory than available
        cpu: "4"
    command: ["sleep", "3600"]
```
Command to Apply:

```bash
kubectl apply -f pending-pod.yaml
```

__Explanation:__ This pod requests 10Gi of memory and 4 CPUs. If your nodes don’t have these resources available, the pod will stay in the “Pending” phase.

### Running

A pod is in the “Running” phase when all its containers are successfully created and at least one container is still running, or is in the process of starting or restarting.

__Scenario:__ Deploy a simple pod that starts successfully and enters the “Running” phase.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: running-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
Command to Apply:
```bash
kubectl apply -f running-pod.yaml
```

__Explanation:__ This pod runs an Nginx server. Once deployed, it will likely transition to the “Running” phase almost immediately, assuming your cluster has enough resources.

### Succeeded

A pod enters the “Succeeded” phase when all its containers have terminated successfully with an exit code of 0, and they will not be restarted. This phase is common for pods with a finite task, such as batch jobs.

__Scenario:__ Deploy a pod with a simple job that completes successfully.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: succeeded-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["echo", "Hello, Kubernetes!"]
```
Command to Apply:
```bash
kubectl apply -f succeeded-pod.yaml
```
__Explanation:__ This pod runs a simple command and exits. After completing, it moves to the “Succeeded” phase.

### Failed

A pod is in the “Failed” phase if any of its containers has terminated with a non-zero exit code or if a container failed to restart according to its restart policy.

__Scenario:__ Deploy a pod that runs a command designed to fail.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: failed-pod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["false"]  # This command will fail
```
Command to Apply:
```bash
kubectl apply -f failed-pod.yaml
```
__Explanation:__ This pod is configured to run a command that returns a non-zero exit code, causing the pod to enter the “Failed” phase.


### Unknown

The “Unknown” phase is when the state of the pod cannot be determined, typically due to communication issues between the node and the Kubernetes API.

### Pod Statuses: Completed vs. Error

In Kubernetes, pods can be in various statuses that reflect their current state. Two key statuses are “Completed” and “Error.”

#### Completed

__Meaning:__ The pod’s containers finished their work successfully and exited without issues.

__Use Case:__ This status is typical for jobs or tasks that are designed to run to completion, such as batch processing or data transformation tasks.

__Indicators__

__Status:__ Shows as “Completed” when all containers in the pod have exited with an exit code of 0.

__Example:__ A pod running a script that processes data and then exits cleanly will have the status “Completed.”

Command to Check:
```bash
kubectl get pod <pod-name>
```

#### Error
__Meaning:__ The pod’s containers encountered an issue and did not complete their tasks successfully. This status is used when a container in the pod exits with a non-zero exit code.

__Use Case:__ This status indicates a problem with the container’s execution, such as a configuration error or runtime issue.

__Indicators__

__Status:__ Shows as “Error” when containers have exited with a non-zero exit code and Kubernetes is unable to recover or restart them successfully.

__Example:__ A pod running a command that fails (e.g., a command with incorrect parameters) will have the status “Error.”

### Summary
“Completed” Status: Indicates successful execution of the pod’s tasks, with all containers exiting cleanly.

“Error” Status: Indicates that there were issues with the pod’s execution, resulting in one or more containers exiting with an error code.


[Main](../README.md)
---