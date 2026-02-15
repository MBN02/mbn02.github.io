---
layout: post
title: "Install an external Ceph cluster using ansible"
date: 2025-12-21 11:12:00 +0800
categories: storage
tags: ceph
image:
  path: /assets/img/headers/ceph.png
---

*Ceph provides scalable, distributed object storage that is self-healing and eliminates single points of failure, making it ideal for handling massive data volumes in cloud environments. Its architecture uses daemons like Monitors (MONs) for cluster maps, OSDs for data storage and replication, and the CRUSH algorithm for decentralized data placement.*

{% include embed/youtube.html id='5ze73hRDamU' %}

🎞️ [Watch Video](https://youtu.be/5ze73hRDamU)

## Pre-requisits

- 3 ubuntu 22.04 LTS nodes 
- Ansible controller node


### Set hostname

```sh
hostnamectl set-hostname ceph1
hostnamectl set-hostname ceph2
hostnamectl set-hostname ceph3
```

### Update hosts file

```sh
# vim /etc/hosts

192.168.122.51  ceph1
192.168.122.52  ceph2
192.168.122.53  ceph3
```

### Copy the ssh-key to all the nodes from our ansible controller node
```sh
ssh-copy-id -i id_ed25519 root@192.168.122.51
```

### clone the cephadm-ansible repo to local
git clone https://github.com/ceph/cephadm-ansible.git

cd cephadm-ansible
 
### Create a hosts file to define the managed ceph nodes 


```sh
# vim hosts
[ceph_servers]
ceph1 ansible_host=192.168.122.51
ceph2 ansible_host=192.168.122.52
ceph3 ansible_host=192.168.122.53

[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_ssh_private_key_file=/home/mkbntech/.ssh/id_ed25519
ansible_user=root
```

### Test the connectivity from our ansible controller node to ceph nodes by running 
```sh
ansible all -i hosts -m ping
```

### Run the  cephadm-preflight.yml playbook.
```sh
ansible-playbook -i hosts cephadm-preflight.yml
```

### ssh into ceph master node and create a `initial_config.yaml` file

```sh
vim initial_config.yaml
```
```sh
---
service_type: host
addr: 192.168.122.51
hostname: ceph1
---
service_type: host
addr: 192.168.122.52
hostname: ceph2
---
service_type: host
addr: 192.168.122.53
hostname: ceph3
---
service_type: mon
placement:
  hosts:
    - ceph1
    - ceph2
    - ceph3    
---
service_type: mgr
placement:
  hosts:
    - ceph1
    - ceph2
    - ceph3
---
service_type: osd
service_id: default_drive_group
placement:
  hosts:
    - ceph1
    - ceph2
    - ceph3
data_devices:
  paths:
    - /dev/vdb
---
```

## bootstrap the ceph cluster using cephadm

```sh
cephadm bootstrap --mon-ip=192.168.122.51 --apply-spec=config.yaml --initial-dashboard-password=P@ssw0rd --dashboard-password-noupdate
```

### verify the infra status by running 
```sh
ceph -s                  # This command provides a high-level overview of the cluster's health,

ceph orch ls             # lists all orchestrated services (daemons like mons, mgrs, osds, rgws) managed by Ceph's Orchestrator

ceph orch host ls        # It displays all hosts registered in the orchestrator inventory
```

### Run the following command to check current pools
```sh
ceph osd pool ls
ceph osd pool ls detail

```
### create new pool for rbd 

```sh
ceph osd pool create {pool-name} replicated  
```

### A Pool should be associated with an application pool before they can be used:
```sh
ceph osd pool application enable {pool-name} {application-name}

```
### check new pools 
```sh
ceph osd pool ls
ceph osd pool ls detail
```

#### Create CEPHFS storage

```sh
ceph fs volume create cephfs
#create cephfs data and metadata pool automatically

#or manual

ceph osd pool create cephfs_data 32
ceph osd pool create cephfs_metadata 1

#check data pools
ceph osd pool ls
ceph osd pool ls detail
cephfs.cephfs.meta
cephfs.cephfs.data

#Eanable data pool
ceph osd pool application enable cephfs_data cephfs
ceph osd pool application enable cephfs.cephfs.meta cephfs

ceph osd pool ls detail

#make file system from this data pool
ceph fs new cephfs-dev cephfs_metadata cephfs_data

#control file system
ceph fs ls

#if need, delete file system: "ceph fs rm  hepapi-cephfs --yes-i-really-mean-it"

#create mds
on ceph dashboard: services--> create mds : 2 or cli: ceph fs volume create cephfs
```

### 🔗 Reference Links:

- [Ceph Docs](https://docs.ceph.com/en/latest/)

- [Cephadm-ansible](https://github.com/ceph/cephadm-ansible)
