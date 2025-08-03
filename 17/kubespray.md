git clone https://github.com/kubernetes-sigs/kubespray.git -b v2.28.0 kubespray_lesson

cd kubespray_lesson

python -m venv my_project_venv

source my_project_venv/bin/activate

pip install -r requirements.txt

cp -r inventory/sample inventory/mycluster

ansible-playbook -i inventory/mycluster/ cluster.yml -b -v \
  --private-key=~/.ssh/private_key






helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --values ingress/values.yml

helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --values ingress/cert.yml --version v1.14.7

k delete crd certificaterequests.cert-manager.io
k delete crd certificates.cert-manager.io
k delete crd challenges.acme.cert-manager.io
k delete crd clusterissuers.cert-manager.io
k delete crd issuers.cert-manager.io
k delete crd orders.acme.cert-manager.io