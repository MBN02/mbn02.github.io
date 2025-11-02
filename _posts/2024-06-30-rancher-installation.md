---
layout: post
title: "Install Rancher with Lets Encrypt on Kubernetes"
date: 2025-11-02 13:25:00 +0800
categories: rancher
tags: rancher
image:
  path: /assets/img/headers/rancher.png
---

*Rancher is a Kubernetes management tool to deploy and run clusters anywhere and on any provider.*

### Prerequisites:

- Kubernetes cluster
- Helm 3.x
- Domain name and ability to perform DNS changes
- Port 80 & 443 must be accessible for Let's Encrypt to verify and issue certificates

#### Pick a subdomain and create a DNS entry pointing to the IP Address that will be assigned to the Rancher Server.


#### Install nginx ingress controller
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

### Install cert-manager with Helm

#### Add the Helm repository:

```sh
helm repo add jetstack https://charts.jetstack.io --force-update
```

#### Update the helm chart repository:
```sh
helm repo update
```

#### Install cert-manager:

```sh
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.19.1 \
  --set crds.enabled=true
```

### Install Rancher:

#### Create `cattle-system` namesapce
```sh
kubectl create ns cattle-system
```

#### Add the `Helm repository`

```sh
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
```

#### `Update` the helm chart repository:
```sh
helm repo update
```

#### Retrieve the package from rancher repository, and download it locally:

```sh
helm fetch rancher-stable/rancher --untar
```

#### Deploy `Rancher`:

```sh
helm install rancher rancher-latest/rancher --namespace cattle-system \
   --set hostname=your_hostname \
   --set bootstrapPassword=Password \
   --set ingress.tls.source=letsEncrypt \
   --set letsEncrypt.email=email@address \
   --set letsEncrypt.ingress.class=nginx \
   --set ingress.ingressClassName=nginx \
   --set replicas=1 \
   --values rancher/values.yaml
```
#### Verify that the Rancher Server is Successfully Deployed

```sh
kubectl -n cattle-system rollout status deploy/rancher
```

```sh
kubectl get pods -n cattle-system -w
```

### Access Rancher User Interface
```sh
https://rancher.url
```
### Reference Links:

- [Installing Helm](https://helm.sh/docs/intro/install/)

- [Cert-manager](https://cert-manager.io/docs/)

- [Rancher](https://ranchermanager.docs.rancher.com/getting-started/installation-and-upgrade/install-upgrade-on-a-kubernetes-cluster)
