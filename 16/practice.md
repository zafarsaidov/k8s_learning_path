[Main](../README.md)
---
# 🧪 Helm Templating Practice Lesson (Step-by-Step)

This lab introduces Helm templating through progressive steps. Each step builds upon the previous to produce a production-grade Helm chart.

Namespace used: `helm-lab`

---

## 🟢 Step 0: Create Namespace

```bash
kubectl create ns helm-lab
```

---

## 🟩 Step 1: Create Basic Helm Chart

```bash
helm create flaskchart
cd flaskchart
rm -rf templates/*  # Start fresh
```

Create a minimal `Chart.yaml`:

```yaml
apiVersion: v2
name: flaskchart
version: 0.1.0
appVersion: "1.0"
description: A basic Flask app chart
```

---

## 🟨 Step 2: Write Basic Deployment Template

Create file: `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "flaskchart.fullname" . }}
  namespace: {{ .Release.Namespace }}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{ include "flaskchart.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "flaskchart.name" . }}
    spec:
      containers:
      - name: app
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
        ports:
        - containerPort: {{ .Values.service.port }}
```

Values for `values.yaml`:

```yaml
image:
  repository: myrepo/flaskapp
  tag: latest

service:
  port: 5000
```

Deploy:

```bash
helm install flaskapp . -n helm-lab
```

---

## 🟧 Step 3: Add Optional Init Container

Update deployment template:

```yaml
      {{- if .Values.initContainers.enabled }}
      initContainers:
      - name: wait-for-db
        image: busybox
        command: ['sh', '-c', 'sleep 5']
      {{- end }}
```

Values:

```yaml
initContainers:
  enabled: false
```

Change `enabled` to `true` to enable.

---

## 🟨 Step 4: Add Conditional Environment Variables

Update container section:

```yaml
        env:
        {{- if .Values.envVars }}
        {{- range .Values.envVars }}
        - name: {{ .name }}
          value: {{ .value | quote }}
        {{- end }}
        {{- end }}
```

Values:

```yaml
envVars:
  - name: ENV
    value: dev
  - name: DEBUG
    value: "true"
```

---

## 🟩 Step 5: Add Optional ConfigMap and Secret References

```yaml
        envFrom:
        {{- if .Values.envFrom.configMaps }}
        {{- range .Values.envFrom.configMaps }}
        - configMapRef:
            name: {{ . }}
        {{- end }}
        {{- end }}
        {{- if .Values.envFrom.secrets }}
        {{- range .Values.envFrom.secrets }}
        - secretRef:
            name: {{ . }}
        {{- end }}
        {{- end }}
```

Values:

```yaml
envFrom:
  configMaps: []
  secrets: []
```

---

## 🟨 Step 6: Add Conditional Probes

```yaml
        {{- if .Values.probes.enabled }}
        livenessProbe:
          httpGet:
            path: {{ .Values.probes.liveness.path }}
            port: {{ .Values.probes.liveness.port }}
          initialDelaySeconds: 5
        readinessProbe:
          httpGet:
            path: {{ .Values.probes.readiness.path }}
            port: {{ .Values.probes.readiness.port }}
          initialDelaySeconds: 3
        {{- end }}
```

Values:

```yaml
probes:
  enabled: false
  liveness:
    path: /healthz
    port: 5000
  readiness:
    path: /readyz
    port: 5000
```

---

## 🟫 Step 7: Add Optional Affinity

```yaml
        {{- if .Values.affinity }}
        affinity:
{{ toYaml .Values.affinity | nindent 10 }}
        {{- end }}
```

Values:

```yaml
affinity: {}
```

---

## 🟦 Step 8: Add Service Template

`templates/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "flaskchart.fullname" . }}
spec:
  selector:
    app: {{ include "flaskchart.name" . }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.port }}
```

---

## 🟪 Step 9: Add Optional Ingress

Create `templates/ingress.yaml`

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "flaskchart.fullname" . }}
spec:
  rules:
  - host: {{ .Values.ingress.host }}
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: {{ include "flaskchart.fullname" . }}
            port:
              number: {{ .Values.service.port }}
{{- end }}
```

Values:

```yaml
ingress:
  enabled: false
  host: flask.local
```

---

## ⏏️ Step 10: Package Helm Chart and Push to GitLab

```bash
helm package .
```

Create GitLab project with Helm registry. Use:

```bash
curl --header "JOB-TOKEN: $CI_JOB_TOKEN" \
     --upload-file flaskchart-0.1.0.tgz \
     "https://gitlab.com/api/v4/projects/<project_id>/packages/helm/api/stable/charts"
```

---

## 🧩 Step 11: Install Chart from GitLab Registry

Add GitLab repo:

```bash
helm repo add myrepo https://gitlab.com/api/v4/projects/<project_id>/packages/helm/stable
helm repo update
helm search repo myrepo
helm install flaskapp myrepo/flaskchart -n helm-lab
```

---

## 🔄 Step 12: Upgrade and Rollback

Update image tag and repackage:

```yaml
image:
  tag: v2
```

```bash
helm package .
# Push to GitLab registry again (repeat Step 10)
```

Upgrade:

```bash
helm upgrade flaskapp myrepo/flaskchart --version 0.2.0 -n helm-lab
```

Rollback:

```bash
helm rollback flaskapp 1 -n helm-lab
```

History:

```bash
helm history flaskapp -n helm-lab
```

Delete:

```bash
helm uninstall flaskapp -n helm-lab
```

---

🎉 Done! You’ve created a complete, reusable Helm chart with full templating support and GitLab publishing integration.

✅ All features are progressively introduced and enabled via values.


[Main](../README.md)
---