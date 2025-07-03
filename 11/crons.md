[Main](../README.md)
---

# 🧪 Practice Lesson: CronJobs in Kubernetes


## 🎯 Objectives

By the end of this lesson, students will:
* Create and schedule CronJobs
* Understand retry behavior and concurrency policies
* Configure history retention and cleanup
* Monitor and debug job executions


## 🧱 Concepts Refresher

| Term | Meaning |
| ---- | ---- |
| CronJob |	A Kubernetes object that runs Jobs on a time-based schedule |
| Job |	Executes a pod to completion |
| Concurrency Policy |	Controls overlapping jobs: Allow, Forbid, Replace |
| StartingDeadlineSeconds |	Prevents missed schedules from starting after a delay |
| Successful/Failed History Limits |	Controls how many job pods to keep |

## 🧪 Practice Tasks

### 1️⃣ Task: Create a Basic CronJob (every minute)

📄 cronjob-basic.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["sh", "-c", "date; echo Hello from the cluster"]
          restartPolicy: OnFailure
```
```bash
kubectl apply -f cronjob-basic.yaml
```
✅ Verify:
```bash
kubectl get cronjob
kubectl get jobs --watch
kubectl logs job/<job-name>   # replace with real job name
```

### 2️⃣ Task: Observe Retries on Failure

📄 cronjob-failure.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: failme
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      backoffLimit: 3
      template:
        spec:
          containers:
          - name: fail
            image: busybox
            command: ["sh", "-c", "exit 1"]
          restartPolicy: OnFailure
```
```bash
kubectl apply -f cronjob-failure.yaml
```
✅ Check retries:
```bash
kubectl describe job -l job-name=failme
kubectl get pods --selector=job-name=<name> -o wide
```

### 3️⃣ Task: Test Concurrency Policies

✅ Allow (default): All jobs run even if previous not finished

🧪 Forbid: New job is skipped if previous is still running

📄 cronjob-forbid.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: sleep-job
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: sleeper
            image: busybox
            command: ["sleep", "90"]
          restartPolicy: OnFailure
```
```bash
kubectl apply -f cronjob-forbid.yaml
```
✅ Observe:
* Job will skip if previous one not done
* Use kubectl get jobs to see fewer jobs

### 🧪 Replace: Replace running job with new one

📄 cronjob-replace.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: replace-job
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Replace
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: replacer
            image: busybox
            command: ["sleep", "90"]
          restartPolicy: OnFailure
```
✅ Watch jobs:
```bash
kubectl get jobs --watch
```
* Only one job should remain active; old one gets terminated.

### 4️⃣ Task: Limit Job History (Cleanup)

📄 cronjob-cleanup.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: cleanup-job
spec:
  schedule: "*/1 * * * *"
  successfulJobsHistoryLimit: 1
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleaner
            image: busybox
            command: ["date"]
          restartPolicy: OnFailure
```
✅ Observe:
```bash
kubectl get jobs --watch
```
Only 1 successful and 1 failed job retained


### 5️⃣ Task: Prevent Missed Jobs with startingDeadlineSeconds

📄 cronjob-strict.yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: strict-start
spec:
  schedule: "*/1 * * * *"
  startingDeadlineSeconds: 10
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["sleep", "70"]
          restartPolicy: OnFailure
```
✅ Simulate pause:
```bash
kubectl scale deployment/kube-controller-manager --replicas=0 -n kube-system
# wait 2 minutes
kubectl scale deployment/kube-controller-manager --replicas=1 -n kube-system
```
Observe: Missed schedule won’t run due to deadline.

### 🔍 Verification Commands

| Command | Purpose |
| ---- | ---- |
| `kubectl get cronjobs` |	View CronJobs |
| `kubectl get jobs` |	View job executions |
| `kubectl get pods --selector=job-name=<name>` |	See pods per job |
| `kubectl logs job/<job>` | 	Inspect job logs |
| `kubectl describe cronjob <name>` |	Check concurrency, deadline, history config |


### 🧹 Cleanup
```bash
kubectl delete cronjob hello failme sleep-job replace-job cleanup-job strict-start
kubectl delete job --all
```

### 🎁 Bonus Challenges
* Deploy a CronJob that posts logs to a webhook
* Create a CronJob that backs up a volume to another pod using initContainers
* Combine CronJob with ResourceQuota and LimitRange in a namespace


[Main](../README.md)
---