---
layout: post
title: "Kuberntes Cluster Deployment with Ansible"
date: 2025-11-15 11:12:00 +0800
categories: ansible
tags: ansible
image:
  path: /assets/img/headers/k8s-ansible.png
---

*Automate the setup of a Kubernetes cluster using Ansible. We’ll build a single master node and two worker nodes — all provisioned and configured through automation.*

{% include embed/youtube.html id='qAs_BKX5YoM' %}

🎞️ [Watch Video](https://youtu.be/qAs_BKX5YoM)

## Prerequisites:

- Ubuntu 22.04 or compatible Linux distribution
- Ansible installed (for running Ansible playbooks)
- Kubernetes knowledge


### Create a `inventory` file:

```bash
[control_plane]
master-node ansible_host=192.168.X.A

[workers]
worker-node1 ansible_host=192.168.X.B
worker-node2 ansible_host=192.168.X.C

[all:vars]
ansible_python_interpreter=/usr/bin/python3

[control_plane:vars]
ansible_ssh_private_key_file= /root/.ssh/id_rsa
ansible_user=root

[workers:vars]
ansible_ssh_private_key_file= /root/.ssh/id_rsa
ansible_user=root
```

## To set up a gitlab instance, run the following Ansible playbook:

```yaml
---
- name: Setup Kubernetes Cluster
  hosts: all
  become: true
  roles:
    - k8s-cluster-with-ansible
```

## Executing the role to install gitlab server

```yaml
ansible-playbook -i inventory setup_kubernetes.yml
```

### 🔗 Reference Links:

- [Ansible Installation](https://docs.ansible.com/projects/ansible/latest/installation_guide/index.html)

- [GitHub-Repo](https://github.com/mkbntech/k8s-cluster-with-ansible.git)
