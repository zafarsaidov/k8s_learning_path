[Main](../README.md)
---

# Helm Practice Lab: Conditionals, Loops, and Hooks

This hands-on lab explores advanced Helm templating features:

* Conditionals (`if`, `else`, `with`)
* Loops (`range`)
* Lifecycle Hooks (`helm.sh/hook`, `hook-weight`, `hook-delete-policy`)

Each concept is demonstrated with examples from simple to more complex use cases.

---

## 🎯 Objectives

* Understand Helm `if`, `else`, `with` blocks
* Use `range` to iterate over values
* Write templates with conditional rendering
* Use Helm lifecycle hooks to control execution timing

---

## 🧱 Step 1: Scaffold chart

```bash
helm create advanced-webapp
cd advanced-webapp
```

---

## 📁 Initial Deployment Template

Create or edit `templates/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "advanced-webapp.fullname" . }}
  labels:
    app: {{ include "advanced-webapp.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "advanced-webapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "advanced-webapp.name" . }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

This will serve as the base deployment file, and we’ll incrementally enhance it in each task.

---

## 🔁 Step 2: Conditionals (`if`, `else`, `with`)

### ✅ Use Case: Enable/Disable Sidecar

In `values.yaml`:

```yaml
sidecar:
  enabled: true
  name: log-watcher
  image: busybox
```

In `deployment.yaml`:

```yaml
      {{- if .Values.sidecar.enabled }}
      - name: {{ .Values.sidecar.name }}
        image: {{ .Values.sidecar.image }}
        command: ["sh", "-c", "tail -F /var/log/app.log"]
      {{- end }}
```

✅ Try disabling sidecar:

```bash
helm template . --set sidecar.enabled=false
```

---

### ✅ Use Case: `with` block

```yaml
{{- with .Values.image }}
image: "{{ .repository }}:{{ .tag }}"
imagePullPolicy: {{ .pullPolicy }}
{{- end }}
```

✅ `with` sets context to nested object and shortens paths.

---

## 🔂 Step 3: Loops (`range`)

### ✅ Use Case: Add custom labels

In `values.yaml`:

```yaml
customLabels:
  team: devops
  owner: alice
  environment: lab
```

In `deployment.yaml`:

```yaml
metadata:
  labels:
    {{- range $key, $val := .Values.customLabels }}
    {{ $key }}: {{ $val | quote }}
    {{- end }}
```

✅ Use `range` to iterate key/value pairs.

---

### ✅ Use Case: Multiple ports

In `values.yaml`:

```yaml
containerPorts:
  - name: http
    containerPort: 80
  - name: metrics
    containerPort: 9100
```

In container spec:

```yaml
ports:
  {{- range .Values.containerPorts }}
  - name: {{ .name }}
    containerPort: {{ .containerPort }}
  {{- end }}
```

✅ You can `range` over lists too.

---

## 🔁 Step 4: Combine Conditionals + Loops

Use this in `service.yaml`:

```yaml
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "advanced-webapp.fullname" . }}
spec:
  selector:
    app: {{ include "advanced-webapp.name" . }}
  ports:
    {{- range .Values.service.ports }}
    - port: {{ .port }}
      targetPort: {{ .targetPort }}
      protocol: {{ .protocol | default "TCP" }}
    {{- end }}
{{- end }}
```

---

## ⛓ Step 5: Helm Hooks

Hooks allow you to run templates at specific release lifecycle points.

### ✅ Use Case: Create init Job to prepare DB

In `templates/job-init-db.yaml`:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: init-db
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: postgres:14
        command: ["sh", "-c", "echo Running DB migrations..."]
      restartPolicy: Never
```

### 🔍 Explanation

* `pre-install`, `pre-upgrade`: Job runs before install/upgrade
* `hook-delete-policy`: Deletes old job before new one is created, and after success

✅ Apply and verify:

```bash
helm install advapp . --dry-run --debug
helm install advapp .
kubectl get jobs
```

---

## 📋 Summary

✅ You practiced:

* Helm conditionals (`if`, `else`, `with`)
* Loops (`range`) for lists and maps
* Combining `range` with `if` for dynamic templates
* Creating lifecycle hooks with annotations

🎓 These techniques allow flexible and robust Helm chart development.

Use `helm template .` to test every combination safely before install!


[Main](../README.md)
---