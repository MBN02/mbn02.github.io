---
layout: post
title: "Install Rancher with Lets Encrypt on Kubernetes"
date: 2025-11-08 12:12:00 +0800
categories: rancher
tags: rancher
image:
  path: /assets/img/headers/rancher.png
---

*Rancher is a Kubernetes management tool to deploy and run clusters anywhere and on any provider.*

{% include embed/youtube.html id='OGaLWmiTfuc' %}

🎞️ [Watch Video](https://youtu.be/OGaLWmiTfuc)

## Prerequisites:

- Kubernetes cluster
- Helm 3.x
- Domain name and ability to perform DNS changes

## Install nginx ingress controller
```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.13.3/deploy/static/provider/baremetal/deploy.yaml
```

#### Change the 'ingress-nginx-controller' service type to LoadBalancer
```sh
kubectl edit svc ingress-nginx-controller -n ingress-nginx
```

#### Create an A record with the IP Address of 'ingress-nginx-controller' service in your domain ragistrar. 
```sh
nslookup subdomain_name
```

## Install cert-manager with Helm

### Add the Helm repository:

```sh
helm repo add jetstack https://charts.jetstack.io --force-update
```

### Update the helm chart repository:
```sh
helm repo update
```

### Install cert-manager:

```sh
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.19.1 \
  --set crds.enabled=true
```
## Generate a private CA and use it with Rancher via cert-manager
 
### Create a config file ca.cnf with CA extensions:

```sh
[ req ]
default_bits       = 2048
distinguished_name = req_distinguished_name
x509_extensions    = v3_ca
prompt             = no

[ req_distinguished_name ]
CN = rancher-private-ca

[ v3_ca ]
basicConstraints = CA:TRUE
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
```

### Generate CA key and cert
```sh
openssl req -x509 -newkey rsa:2048 -sha256 -days 3650 \
  -keyout ca.key -out ca.crt -config ca.cnf -nodes
```

## Generate Rancher Server TLS Cert Signed by Private CA

### Create a CSR config rancher-csr.cnf
```sh
[req]
default_bits       = 2048
prompt             = no
default_md         = sha256
distinguished_name = dn
req_extensions     = req_ext

[dn]
CN = rancher.mkbn.in

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = rancher.mkbn.in
```

### Generate private key and CSR
```sh
openssl genrsa -out tls.key 2048

openssl req -new -key tls.key -out tls.csr -config rancher-csr.cnf
```

### Sign CSR with your CA
```sh
openssl x509 -req -in tls.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out tls.crt -days 365 -sha256 -extfile rancher-csr.cnf -extensions req_ext

```

### Create cattle-system namesapce

```sh
kubectl create ns cattle-system
```

## Create Kubernetes Secrets

### Create CA certificate secret (generic).This secret is for Rancher to trust your private CA.

```sh
kubectl -n cattle-system create secret generic tls-ca --from-file=cacerts.pem=ca.crt
kubectl -n cattle-system create secret generic tls-ca-additional --from-file=cacerts.pem=ca.crt

```
### Create TLS secret for Rancher ingress.This secret holds your Rancher TLS cert and key signed by your private CA.

```sh
kubectl -n cattle-system create secret tls tls-rancher-ingress --cert=tls.crt --key=tls.key
```

### Create cert-manager ClusterIssuer Using CA
```sh
cat <<EOF | kubectl apply -f -
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: rancher-ca-issuer
spec:
  ca:
    secretName: tls-ca
EOF
```
## Install Rancher:

### Add the rancher stable helm repository
```sh
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

### Add the Helm repository

```sh
helm repo update
```

### Retrieve the package from rancher repository, and download it locally
```sh
helm fetch rancher-stable/rancher --untar
```

### Deploy Rancher
```sh
helm upgrade --install rancher rancher-stable/rancher --namespace cattle-system \
  --set hostname=rancher.mkbn.in \
  --set bootstrapPassword=P@ssw0rd \
  --set ingress.tls.source=secret \
  --set ingress.tls.secretName=tls-rancher-ingress \
  --set ingress.ingressClassName=nginx \
  --set privateCA=true \
  --set additionalTrustedCAs=true \
  --set-string "additionalTrustedCASecrets[0]=tls-ca" \
  --set replicas=3
```

### Verify that the Rancher Server is Successfully Deployed

```sh
kubectl get pods -n cattle-system -w
```

```sh
kubectl -n cattle-system rollout status deploy/rancher
```

## Access Rancher User Interface
```sh
https://rancher.url
```
### 🔗 Reference Links:

- [Installing Helm](https://helm.sh/docs/intro/install/)

- [Cert-manager](https://cert-manager.io/docs/)

- [Rancher](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster)
