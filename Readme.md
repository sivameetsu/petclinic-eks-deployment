
Download the credentials

```bash

aws eks update-kubeconfig --region us-east-1 --name prod-dev

```

`Requirements`

* 1. kubectl
* 2. helm
* 3. docker engine

`kubernetes certificate attachement`

* Certificat manager instal via helm charts
* Domain name purchase

    * Domain fourtimes.ml

_create the ingress-nginx controller_

```bash

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/aws/deploy.yaml

```

certificate manager

```bash

helm repo add jetstack https://charts.jetstack.io
helm repo update
helm show values jetstack/cert-manager > values.yml
kubectl create ns cert-manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager -f values.yml

```
