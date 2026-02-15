---
layout: post
title: "RKE2 Cluster Installation With Ansible"
date: 2025-12-24 11:12:00 +0800
categories: kubernetes
tags: rke2
image:
  path: /assets/img/headers/rke2.jpg
---

*RKE2 is Rancher's enterprise-ready next-generation Kubernetes distribution.It delivers upstream‑compatible Kubernetes with built‑in security hardening, compliance‑friendly by defaults, and a simple operational model that scales cleanly from data center to edge.*

{% include embed/youtube.html id='zvY70Rw4f3U' %}

🎞️ [Watch Video](https://youtu.be/zvY70Rw4f3U)

## Pre-requisits

- 6 Ubuntu 24.04 LTS on all nodes [ 3 servers and 3 agent nodes]
- Ansible controller node


### Set hostname on all the nodes using hostnamectl set-hostaname command

```sh
hostnamectl set-hostname master-01
hostnamectl set-hostname master-02
```

### Update /etc/hosts file
```sh
192.168.122.101 master-01
192.168.122.102 master-02
192.168.122.103 master-03
192.168.122.201 worker-01
192.168.122.202 worker-02
192.168.122.203 worker-03
```

## Install ansible

```sh
sudo apt update
sudo apt install ansible -y
```

## Set up passwordless SSH

### Generate ssh key

```sh
ssh-keygen
```

### Copy ssh keys to master and worker nodes

```sh
{
declare -a NODES=(192.168.122.101 192.168.122.102 192.168.122.103 192.168.122.201 192.168.122.202 192.168.122.203)

for node in ${NODES[@]}; do
  ssh-copy-id -i ~/.ssh/id_ed25519 root@$node
done
}
```

### Install the RKE2 deployment role 

```sh
ansible-galaxy install lablabs.rke2
```

### Navigate to the ~/.ansible directory

```sh
cd ~/.ansible
```

### Create inventory file in ~/.ansible

```sh
vim inventory
```

```sh
[masters]
master-01 ansible_host=192.168.122.101 rke2_type=server
master-02 ansible_host=192.168.122.102 rke2_type=server
master-03 ansible_host=192.168.122.103 rke2_type=server

[workers]
worker-01 ansible_host=192.168.122.201 rke2_type=agent
worker-02 ansible_host=192.168.122.202 rke2_type=agent
worker-03 ansible_host=192.168.122.203 rke2_type=agent

[k8s_cluster:children]
masters
workers

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=/root/.ssh/id_ed25519
ansible_user=root
```

### Create `playbook.yaml` file in `~/.ansible`

```sh
- name: Deploy RKE2
  hosts: all
  become: yes
  vars:
    rke2_ha_mode: true
    rke2_api_ip : 192.168.122.100
    rke2_download_kubeconf: true
    #rke2_ha_mode_keepalived: false
    rke2_server_node_taints:
      - 'CriticalAddonsOnly=true:NoExecute'
  roles:
    - role: lablabs.rke2
```

#### Confirm inventory is working
```sh
ansible all -i inventory -m ping
```

```sh
ansible-playbook -i inventory playbook.yaml
```
#### If you're ssh'ing to the other machines as a non-root user, run the following instead:
```sh
ansible-playbook -i inventory playbook.yaml -K
```

#### Manage our cluster with kubectl --kubeconfig ~/rke2.yaml, or we can do the following to shorten our commands:

```sh
export KUBECONFIG=~/rke2.yaml
```

#### Confirm our cluster is running and with correct internal IP addresses

```sh
kubectl get nodes -o wide
```

#### Check the health of our pods

```sh
kubectl get pods -A
```

### 🔗 Reference Links:

- [lablabs/rke2 ansible role](https://galaxy.ansible.com/lablabs/rke2)

- [rke2 docs](https://docs.rke2.io/)
