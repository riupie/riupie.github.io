---
layout: post
title:  "Container based Openstack Deployment using Ceph Ansible and Kolla Ansible 2"
comments: true
date:   2020-07-04
desc: "Openstack Series"
keywords: "linux,openstack,docker,ceph storage"
categories: [Cloud]
tags: [ceph,openstack,cloud]
icon: icon-shell
---
# Create Ceph Pool and Client for Openstack

After Ceph Cluster deployed in [previos post](https://rahmatawe.com/cloud/2020/07/03/Containerized-Openstack-Infrastructure-1.html), in this second stage we have to create volume pools that will be used by Openstack services such as Glance, Nova and Cinder or depends on your Openstack architecture.

### 1. Create Openstack pool
```
ceph osd pool create volumes
ceph osd pool create images
ceph osd pool create backups
ceph osd pool create vms

rbd pool init volumes
rbd pool init images
rbd pool init backups
rbd pool init vms
```

### 2. Share ceph config to each server
```
ssh 10.50.50.11 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh 10.50.50.12 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
```

### 3. Setup Client Configuration
```
ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups' mgr 'profile rbd pool=backups'
```

### 4. Setup Ceph Client
```
ceph auth get-or-create client.glance | ssh 10.50.50.10 sudo tee /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder | ssh 10.50.50.10 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.50.50.10 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring

ceph auth get-or-create client.glance | ssh 10.50.50.11 sudo tee /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder | ssh 10.50.50.11 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.50.50.11 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring

ceph auth get-or-create client.glance | ssh 10.50.50.12 sudo tee /etc/ceph/ceph.client.glance.keyring
ceph auth get-or-create client.cinder | ssh 10.50.50.12 sudo tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-or-create client.cinder-backup | ssh 10.50.50.12 sudo tee /etc/ceph/ceph.client.cinder-backup.keyring
```

### 5. Verify Ceph Pool List
```
$ ceph osd lspools

1 device_health_metrics
2 cephfs_data
3 cephfs_metadata
6 volumes
7 images
8 backups
9 vms

```
---