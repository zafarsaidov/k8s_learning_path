[Main](../README.md)
---

# 🧪 Practice Lesson: RBAC – Roles, Bindings, and Access Verification (Advanced)

## Create a new user in Kubernetes Cluster

### Generate Certificates for the User
Generate a private key for devops using RSA algorithm (4096 bits):

```bash
openssl genrsa -out devops.pem
```

This command will generate an RSA private key.

Create a Certificate Signing Request (CSR) for devops:
```bash
openssl req -new -key devops.pem -out devops.csr -subj "/CN=devops"
```
### Create a Certificate Signing Request (CSR)
Obtain the base64-encoded representation of the CSR:
```bash
cat devops.csr | base64 | tr -d "\n"
```
Here we encode the CSR to be used in the CertificateSigningRequest.

Use the output to create a CertificateSigningRequest resource:
```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: devops
spec:
  request: <base64_encoded_csr>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # one day
  usages:
  - digital signature
  - key encipherment
  - client auth
EOF
```
CSR status
```bash
kubectl get csr
```
### Sign the Certificate Using the Cluster Certificate Authority
Approve the CertificateSigningRequest for devops:
```bash
kubectl certificate approve devops
```

Check the CSR status:
```bash
kubectl get csr                         
kubectl describe csr/devops
```

### Create a Configuration Specific to the User
Extract the signed certificate from the CertificateSigningRequest:
```bash
kubectl get csr/devops -o jsonpath="{.status.certificate}" | base64 -d > devops.crt
```
Create new user config file:

Use the kubectl config set-cluster command to set up the cluster information:
```bash
kubectl --kubeconfig ~/.kube/config-devops config set-cluster preprod --insecure-skip-tls-verify=true --server=https://KUBERNETES-API-ADDRESS
```
Replace KUBERNETES-API-ADDRESS with the actual API server address of your Kubernetes cluster.

Set user credentials:

Use the kubectl config set-credentials command to set up the user credentials:
```bash
kubectl --kubeconfig ~/.kube/config-devops config set-credentials devops --client-certificate=devops-user.crt --client-key=devops.pem --embed-certs=true
```
Replace devops-user.crt and devops.pem with the paths to your user certificate and private key files respectively.

Set context information:

Use the kubectl config set-context command to set up the context information:
```bash
kubectl --kubeconfig ~/.kube/config-devops config set-context default --cluster=preprod --user=devops
```
Use the context:

Finally, use the kubectl config use-context command to use the newly created context:
```bash
kubectl --kubeconfig ~/.kube/config-devops config use-context default
```
Now, your config-devops file is configured with the necessary cluster, user, and context information. You can use it with kubectl commands by passing --kubeconfig ~/.kube/config-devops. Make sure to replace placeholder values with your actual configuration details.

Example:
```bash
kubectl --kubeconfig ~/.kube/config-devops get pods
```
We’ve successfully created a new user named “devops”. However, when we try to access the pods using this user, we encounter a Forbidden error.

This error occurs because we haven’t assigned any permissions to the “devops” user yet; we’ve only created the user.

Now, let’s proceed to create new roles and role bindings and associate them with the “devops” user using the following steps.

## Writing RBAC Rules
Let’s create RBAC rules and apply to the above user:
### Allow read-only access to pods in specific namespace

Create a file named pod-reader-role.yaml with the following content:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: kube-system
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
Apply the Role using:
```bash
kubectl apply -f pod-reader-role.yaml
```
This Role allows getting and listing pods in the kube-system namespace.

Create a file named devops-pod-reader-rolebinding.yaml with the following content:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devops-pod-reader
  namespace: kube-system
subjects:
- kind: User
  name: devops
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
Apply the RoleBinding using:
```bash
kubectl apply -f devops-pod-reader-rolebinding.yaml
```
This RoleBinding binds the “devops” user to the “pod-reader” Role in the kube-system namespace.

Now, the “devops” user should be able to list and get pods in the kube-system namespace using the provided kubeconfig i.e., ~/.kube/config-devops .

We’ve set up a new role and role binding for the user “devops”. Now, let’s try accessing the pods in the kube-system namespace with the following command.

```bash
kubectl --kubeconfig ~/.kube/config-devops get pods -n kube-system
```

We’ve successfully listed the pods in the kube-system namespace. However, if you attempt to access other resources and resources outside of the kube-system namespace, you’ll receive a Forbidden error.

```bash
kubectl --kubeconfig ~/.kube/config-devops get cm -n kube-system
```
### Allow read-only access to pods in all namespace

You can define a ClusterRole and ClusterRoleBinding in YAML format using below steps to allow the “devops” user to list pods across all namespaces.

Create a file named pod-list-clusterrole.yaml with the following content:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-lister
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```
Apply the ClusterRole using:
```bash
kubectl apply -f pod-list-clusterrole.yaml
```
This ClusterRole allows getting and listing pods across all namespaces.

Create a file named devops-pod-lister-clusterrolebinding.yaml with the following content:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: devops-pod-lister
subjects:
- kind: User
  name: devops
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-lister
  apiGroup: rbac.authorization.k8s.io
```
Apply the ClusterRoleBinding using:
```bash
kubectl apply -f devops-pod-lister-clusterrolebinding.yaml
```
This ClusterRoleBinding binds the “devops” user to the “pod-lister” ClusterRole, allowing the user to list pods across all namespaces.

Now, the “devops” user should be able to list pods across all namespaces using the provided kubeconfig i.e., ~/.kube/config-devops .

We’ve set up a new clusterRole and clusterRoleBinding for the user “devops”. Now, let’s try accessing the pods in all namespaces with the following command.

```bash
kubectl --kubeconfig ~/.kube/config-devops get po -A
```

We’ve successfully listed the pods in all namespaces. However, if you attempt to access other resources(other than pods) in any namespace, you’ll receive a Forbidden error.

```bash
kubectl --kubeconfig ~/.kube/config-devops get cm -A  
```

Other commands for checking access to resources

```bash
kubectl auth can-i list pods --as=devops
kubectl auth can-i create pods --as=devops

```


## Create Service Account and Extract Token

```bash
kubectl create serviceaccount app-sa -n rbac-lab
SECRET=$(kubectl get sa app-sa -n rbac-lab -o jsonpath='{.secrets[0].name}')
TOKEN=$(kubectl get secret $SECRET -n rbac-lab -o jsonpath='{.data.token}' | base64 -d)
CA=$(kubectl get secret $SECRET -n rbac-lab -o jsonpath='{.data.ca\.crt}' | base64 -d)
```

If there is no secret
```bash
cat > app-sa-secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: app-sa-token-g955r
  annotations:
    kubernetes.io/service-account.name: app-sa
type: kubernetes.io/service-account-token
EOF
```

## Create Kubeconfig for Service Account
```bash
SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
kubectl config set-cluster dev-cluster \
  --server=$SERVER \
  --certificate-authority=<(echo "$CA") \
  --embed-certs=true \
  --kubeconfig=app-sa.kubeconfig

kubectl config set-credentials app-sa \
  --token=$TOKEN \
  --kubeconfig=app-sa.kubeconfig

kubectl config set-context sa-context \
  --cluster=dev-cluster \
  --user=app-sa \
  --namespace=rbac-lab \
  --kubeconfig=app-sa.kubeconfig

kubectl config use-context sa-context --kubeconfig=app-sa.kubeconfig
```


## Create Role and RoleBinding for the Service Account

```yaml
# role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: configmap-manager
  namespace: rbac-lab
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create", "get", "list"]
---
# rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-app-sa
  namespace: rbac-lab
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: rbac-lab
roleRef:
  kind: Role
  name: configmap-manager
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
```

✅ Verify:
```bash
kubectl auth can-i create configmaps --as=system:serviceaccount:rbac-lab:app-sa -n rbac-lab
kubectl auth can-i delete pods --as=system:serviceaccount:rbac-lab:app-sa -n rbac-lab
```

✅ Using app-sa kubeconfig:
```bash
kubectl auth can-i create configmaps --kubeconfig=app-sa.kubeconfig
```
## BONUS CHALLENGES
* Try impersonating system:unauthenticated and check what’s allowed.
* Create a Role that only allows listing pods with a specific label.
* Use kubectl --as to simulate access for other roles (e.g., --as=system:admin).

## 🧹 Cleanup

* kubectl delete ns rbac-lab
* kubectl delete csr devuser
* kubectl delete clusterrole devuser-reader
* kubectl delete clusterrolebinding bind-devuser
* rm -f devuser.* devuser.kubeconfig app-sa.kubeconfig



[Main](../README.md)
---