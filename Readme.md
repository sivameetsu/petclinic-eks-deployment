
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
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true

# upgrade the palybook
helm upgrade cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true

```



deployment file preparation with certificate manager 

```bash

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  revisionHistoryLimit: 4
  paused: false
  replicas: 2
  minReadySeconds: 10
  selector:
    matchLabels:
      role: webserver
    matchExpressions:
      - {key: version, operator: In, values: [v1, v2, v3]}
  template:
    metadata:
      name: web
      labels:
        role: webserver
        version: v1
        tier: frond-end
    spec:
      containers:
        - name: web
          image: nginx
          ports:
            - containerPort: 80
              protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    role: web-service
spec:
  selector:
    role: webserver
    version: v1
  type: ClusterIP
  ports:
    - port: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: petclinic.fourtimes.ml
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-dev"
    kubernetes.io/tls-acme: "true"
    # ingress.kubernetes.io/force-ssl-redirect: "true"
    # nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
    - hosts:
      - petclinic.fourtimes.ml
      secretName: petclinic.fourtimes.ml
  rules:
  - host: petclinic.fourtimes.ml
    http:
      paths:
      - backend:
          service:
            name: web-service
            port:
              number: 80
        path: /
        pathType: ImplementationSpecific
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-dev
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: jinojoe@gmail.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-dev
    # Enable the HTTP-01 challenge provider
    solvers:
    - selector: {}
      http01:
        ingress:
          class: nginx
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: petclinic.fourtimes.ml
spec:
  secretName: petclinic.fourtimes.ml
  privateKey:
    rotationPolicy: Always
  issuerRef:
    name: letsencrypt-dev
    kind: ClusterIssuer
    group: cert-manager.io
  commonName: petclinic.fourtimes.ml
  dnsNames:
    - petclinic.fourtimes.ml


```
