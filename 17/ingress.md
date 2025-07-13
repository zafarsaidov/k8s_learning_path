[Main](../README.md)
---

# Ingress-nginx

Add helm repos

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo add jetstack https://charts.jetstack.io

helm repo update
```

Create values files 

```bash
helm show values ingress-nginx/ingress-nginx > ingress.yml

helm show values jetstack/cert-manager > cert-manager.yml
```

Change this variables:
```yaml
# ingress-nginx--4.11.2.yaml
controller:
  # ...
  hostNetwork: true

  # ...
  hostPort:
  # ...
    enabled: true
  # ...
    ports:
      http: 80
      https: 443

  # ...
  ingressClassResource:
    enabled: true
    default: true

  # ...
  kind: DaemonSet # Deployment

  # ...
  tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Equal"
      value: ""
      effect: "NoSchedule"

  # ...
  nodeSelector:
    kubernetes.io/os: linux
    group: control-plane

  # ...
  service:
    # ...
    enabled: true
    # ...
    external:
      enabled: true
    # ...
    type: ClusterIP

  # ...
  nodePorts:
    http: "80"
    https: "443"

  # ... optional
  metrics:
    enabled: true
```

For cert-manager enable installing CRD

Install
```bash
helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --values ingress.yml

helm upgrade --install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --values cert-manager.yml --version v1.14.7
```

Create cluster issuer
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: zafar@zsaidov.uz
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          ingressClassName: nginx
```

[Main](../README.md)
---