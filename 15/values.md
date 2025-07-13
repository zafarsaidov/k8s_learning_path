[Main](../README.md)
---

# Helm Practice: values.yaml and Template Helpers

This lab is focused entirely on understanding `values.yaml` and the `_helpers.tpl` file in Helm.

---

## 🎯 Objectives

* Understand how `values.yaml` is used in templates
* Create and reference template helpers
* Practice using `--set` and `--values` overrides
* Use `helm template` to debug rendering
* Learn commonly used filters like `nindent`, `quote`, `toYaml`

---

## 🛠 Step 1: Create a New Helm Chart

```bash
helm create webapp
cd webapp
```

---

## 📂 Step 2: Explore Chart Structure

```text
webapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── _helpers.tpl
```

✅ `values.yaml` holds default values for parameters used in templates.

---

## 🧾 Step 3: Edit values.yaml

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: "1.25"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
fullnameOverride: ""
nameOverride: ""
```

---

## 🔧 Step 4: Create \_helpers.tpl

```gotpl
{{- define "webapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" -}}
{{- end }}

{{- define "webapp.fullname" -}}
{{- if .Values.fullnameOverride -}}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" -}}
{{- else -}}
{{- printf "%s-%s" .Release.Name (include "webapp.name" .) | trunc 63 | trimSuffix "-" -}}
{{- end -}}
{{- end }}
```

### 🔍 Explanation

* `define` ... `end`: Declares a named template that can be reused.
* `default .Chart.Name .Values.nameOverride`: If `nameOverride` is empty, use the chart name.
* `trunc 63`: Ensures the name doesn't exceed 63 characters (Kubernetes label limit).
* `trimSuffix "-"`: Removes trailing dashes.
* `printf "%s-%s"`: Combines release name and app name.
* `include "webapp.name" .`: Calls another defined template and passes current context (`.`).

These helpers ensure consistent naming throughout your templates.

---

## 📦 Step 5: Use helpers in deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "webapp.fullname" . }}
  labels:
    app: {{ include "webapp.name" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "webapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "webapp.name" . }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - containerPort: {{ .Values.service.port }}
```

---

## 🧪 Step 6: Install chart and override values

```bash
helm install webapp . -n helm-lab --create-namespace \
  --set replicaCount=3 \
  --set image.tag=1.26 \
  --set nameOverride=nginx-lab
```

✅ Verify overrides:

```bash
helm get values webapp -n helm-lab
helm get manifest webapp -n helm-lab
```

---

## 🔍 Step 7: Use `helm template` to render chart

```bash
helm template webapp . \
  --set replicaCount=1 \
  --set image.repository=nginx \
  --set image.tag=1.27
```

✅ This shows the rendered Kubernetes manifests without installing.

---

## 🔤 Step 8: Use Helm Filters (Functions)

Update `deployment.yaml` to use filters:

```yaml
labels:
  app: {{ include "webapp.name" . | quote }}
  chart: {{ .Chart.Name | upper }}
  values: {{ .Values | toYaml | nindent 2 }}
```

### 🔍 Filter Explanations

* `quote`: Wraps value in double quotes.
* `upper`: Converts string to uppercase.
* `toYaml`: Converts a map/object to YAML format.
* `nindent N`: Adds N spaces before every line of the YAML output (used for nesting).

These filters make complex objects easier to embed into YAML.

---

## 🧹 Step 9: Cleanup

```bash
helm uninstall webapp -n helm-lab
kubectl delete ns helm-lab
```

---

## ✅ Summary

You practiced:

* Using and overriding `values.yaml`
* Creating helper functions in `_helpers.tpl`
* Referencing helpers in templates
* Using filters: `quote`, `toYaml`, `nindent`
* Rendering charts with `helm template`
* Installing charts with dynamic values

🎯 This foundational knowledge sets you up for using conditions, loops, and advanced templating in future lessons.


[Main](../README.md)
---