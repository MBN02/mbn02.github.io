---
layout: post
title: "Meet Longhorn a cloud native distributed block storage for Kubernetes"
date: 2025-11-05 12:12:00 +0800
categories: kubernetes
tags: longhorn
image:
  path: /assets/img/headers/longhorn.jpg
---

### Cloud native distributed block storage for Kubernetes

{% include embed/youtube.html id='ts7LAnM744Y' %}

🎞️ [Watch Video](https://youtu.be/ts7LAnM744Y)

### Prerequisites:

**Minimum Hardware requirements:**

- 3 nodes
- 4 vCPUs per node
- 4 GiB per node
- SSD/NVMe or similar performance block device on the node for storage


### Installation Requirements:

- A container runtime compatible with Kubernetes (Docker v1.13+, containerd v1.3.7+, etc.)
- Kubernetes >= v1.21
- `open-iscsi` is installed, and the `iscsid` daemon is running on all the nodes.
- RWX support requires that each node has a NFSv4 client installed.
- The host filesystem supports the `file extents` feature to store the data. Currently longhorn support:
    - ext4
    - XFS
- `bash, curl, findmnt, grep, awk, blkid, lsblk` must be installed.
- `Mount propagation` must be enabled.

### Install dependencies:

Install `nfs-common, open-iscsi` & ensure `daemon` is running on all the nodes.

```sh
{
sudo apt update
sudo apt install -y nfs-common open-iscsi
sudo systemctl enable open-iscsi --now
systemctl status iscsid
}
```

**Run the Environment Check Script:**

```sh
# For AMD64 platform
curl -sSfL -o longhornctl https://github.com/longhorn/cli/releases/download/v1.10.0/longhornctl-linux-amd64

# For ARM platform
curl -sSfL -o longhornctl https://github.com/longhorn/cli/releases/download/v1.10.0/longhornctl-linux-arm64

chmod +x longhornctl
./longhornctl check preflight --kubeconfig=.kube/config

```

### Installing Longhorn with Helm:
*Helm v3.0+ must be installed on your workstation.*

Add the Longhorn Helm repository:
```sh
helm repo add longhorn https://charts.longhorn.io
```

Fetch the latest charts from the repository:
```sh
helm repo update
```

Retrieve the package from longhorn repository, and download it locally:

```sh
helm fetch longhorn/longhorn --untar
```

Install Longhorn in the longhorn namespace:

```sh
helm install longhorn longhorn/longhorn --values longhorn/values.yaml -n longhorn-system --create-namespace --version 1.10.0
```

To confirm that the deployment succeeded, run:
```sh
kubectl -n longhorn-system get pod
```

### Enabling basic authentication with ingress for longhorn UI
*Authentication is not enabled by default for kubectl and Helm installations.*

*Note : Create a basic authentication file `auth`. It’s important the file generated is named auth (actually - that the secret has a key data.auth), otherwise the Ingress returns a 503.*

```sh
USER=<USERNAME_HERE>; PASSWORD=<PASSWORD_HERE>; echo "${USER}:$(openssl passwd -stdin -apr1 <<< ${PASSWORD})" >> auth
```
### Create a secret:

```sh
kubectl -n longhorn-system create secret generic basic-auth --from-file=auth
```

### Create the ingress resource:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: longhorn-frontend
  namespace: longhorn-system
  annotations:
    nginx.ingress.kubernetes.io/auth-type: basic
    nginx.ingress.kubernetes.io/auth-secret: basic-auth
    nginx.ingress.kubernetes.io/auth-realm: "Authentication Required"
    # cert-manager.io/cluster-issuer: "letsencrypt-prod"
    # nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - longhorn.mkbn.in
      secretName: tls-longhorn-frontend
  rules:
    - host: longhorn.mkbn.in
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: longhorn-frontend
              port:
                number: 80
EOF
```
### Accessing the Longhorn UI:

```sh
https://longhorn.mkbn.in
```

### Accessing without ingress:

Get the Longhorn’s external service IP:

```sh
kubectl -n longhorn-system get svc
```
Use `CLUSTER-IP` of the `longhorn-frontend` to access the Longhorn UI using port forward:

```sh
kubectl port-forward svc/longhorn-frontend 8080:80 -n longhorn-system
```

### Create a demo StatefulSet using the default storage class:

Check out the [github repo](https://github.com/mkbntech/longhorn) for code sample.

```sh
kubectl apply -f gitea-demo/gitea.yaml
```


🔗 Reference Links:

- [longhorn](https://longhorn.io/)

- [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)