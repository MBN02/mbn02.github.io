---
layout: post
title: "Local AI Stack : Ollama + NVIDIA GPU Operator + Open WebUI"
date: 2025-11-19 11:12:00 +0800
categories: AI
tags: AI
image:
  path: /assets/img/headers/ollama.png
---

*If you have a workstation/server with nvidia GPU and would like to have a full AI-ready stack running with large language model's and Open WebUI - all within your own Kubernetes cluster for AI model deployment and testing then this video is for you.*

{% include embed/youtube.html id='a-EqyRsnJSc' %}

🎞️ [Watch Video](https://youtu.be/a-EqyRsnJSc)

## Install nvidia drivers for your GPU
```sh
# Check your GPU's PCI address
lspci -nn | grep -i nvidia
```
```sh
# check the list of drivers
sudo ubuntu-drivers devices
```
```sh
# Install the recommended driver
sudo apt install nvidia-driver-590-open
```

## Install MicroK8s

```sh
# Install MicroK8s
sudo snap install microk8s --classic --channel=1.35/stable
```
### Add your user to the microk8s group
```sh
{
  sudo usermod -aG microk8s $USER
  sudo mkdir .kube
  sudo chown -f -R $USER ~/.kube
  newgrp microk8s
}
```
### Verify installation
```sh
microk8s status --wait-ready
```
## Install kubectl
```sh
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## Enable Essential Addons

```sh
# Enable DNS (required)
microk8s enable dns

# Enable storage (for persistent volumes)
microk8s enable hostpath-storage

# Enable MetalLB
microk8s enable metallb

# 192.168.1.80-192.168.1.90
```

### Configure kubectl Alias
```sh
microk8s config > ~/.kube/config
```

### Verify MetalLB Configuration
```sh
# Check MetalLB pods
kubectl get pods -n metallb-system

# View MetalLB IP pool
kubectl get ipaddresspool -n metallb-system
kubectl get l2advertisement -n metallb-system
```

## Configure Persistent Storage
```sh
kubectl get storageclass
```

### Addons
```sh
microk8s enable hostpath-storage
```

```sh
kubectl apply -f hostpath-storage/storage-class.yaml
```

## Install helm 3 and enable it

```sh
{
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
}
```
```sh
microk8s enable helm3
```


## Prepare for NVIDIA GPU Operator

### Create GPU Operator namespace
```sh
kubectl create namespace gpu-operator
```
### Install NVIDIA GPU Operator
```sh
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia
helm repo update
helm show values nvidia/gpu-operator > nvidia-values.yaml
```

### Create values file for MicroK8s:
```sh
toolkit:
   env:
   - name: CONTAINERD_CONFIG
     value: /var/snap/microk8s/current/args/containerd-template.toml
   - name: CONTAINERD_SOCKET
     value: /var/snap/microk8s/common/run/containerd.sock
   - name: RUNTIME_CONFIG_SOURCE
     value: "file=/var/snap/microk8s/current/args/containerd.toml"
```

### Install GPU Operator:
```sh
helm install gpu-operator nvidia/gpu-operator \
  -n gpu-operator \
  -f nvidia-values.yaml \
  --wait
```

### Verify GPU Operator Installation

```sh
# Check all GPU operator pods are running
kubectl get pods -n gpu-operator

# Check GPU nodes
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,GPU:.status.allocatable.nvidia\.com/gpu"

# Verify NVIDIA runtime
kubectl describe node | grep -A 5 "Allocatable"
```

### Verification: Running Sample GPU Applications

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: cuda-vectoradd
spec:
  restartPolicy: OnFailure
  containers:
  - name: cuda-vectoradd
    image: "nvcr.io/nvidia/k8s/cuda-sample:vectoradd-cuda11.7.1-ubuntu20.04"
    resources:
      limits:
        nvidia.com/gpu: 1
EOF
```

```sh
kubectl logs pod/cuda-vectoradd
```

## Deploy Ollama LLM in Kubernetes

### Create Namespace and Storage

```sh
kubectl create namespace ollama
```

Apply:
```sh
kubectl apply -f ollama/ollama-pvc.yaml
```

## Deploy Ollama

```sh
kubectl apply -f ollama/ollama-deployment.yaml

# Wait for pod to be ready
kubectl wait --for=condition=ready pod -l app=ollama -n ollama --timeout=300s
```

### Load Model into Ollama

```sh
# Get pod name
POD_NAME=$(kubectl get pod -n ollama -l app=ollama -o jsonpath='{.items[0].metadata.name}')

#To load a model into Ollama, simply use
kubectl exec -n ollama $POD_NAME -- ollama pull gemma3:4b

# Verify model is loaded
kubectl exec -n ollama $POD_NAME -- ollama list
```

### Test Ollama Service

```sh
kubectl port-forward -n ollama svc/ollama 11434:11434
```

### In another terminal, test the API
```sh
curl http://localhost:11434/api/generate -d '{
  "model": "gemma3:4b",
  "prompt": "Explain crewai in one sentence",
  "stream": false
}'
```

## Deploy Open WebUI (formerly Ollama WebUI)

### Add Open WebUI Helm Repository

```sh
helm repo add open-webui https://open-webui.github.io/helm-charts
helm repo update
helm upgrade --install openwebui open-webui/open-webui --values=open-webui-values.yaml -n ollama
```

```sh
ollamaUrls:
  - http://ollama.ollama.svc.cluster.local:11434
```

“Now, open your browser and access `http://<your-node-ip>:80` to start chatting with your models via a clean web interface.”

### 🔗 Reference Links:

- [Ollama](https://ollama.com/)

- [GitHub-Repo](https://github.com/mkbntech/local-ai-stack.git)

- [Open WebUI](https://docs.openwebui.com/)
