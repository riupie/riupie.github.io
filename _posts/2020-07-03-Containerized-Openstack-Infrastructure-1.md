---
layout: post
title:  "Container based Openstack Deployment using Ceph Ansible and Kolla Ansible 1"
comments: true
date:   2020-07-03
desc: "Openstack Series"
keywords: "linux,openstack,docker,ceph storage"
categories: [Cloud]
tags: [ceph,openstack,cloud]
icon: icon-shell
---

In this article series, I will do a PoC how to deploy Openstack Cluster on top container based infrastructure. Openstack Cluster will be integrated with Ceph Cluster including images, instance and volume.
There are two tools I use:
1. Ceph Ansible to provision Ceph Cluster (Podman container)
2. Kolla Ansible to provision Openstack CLuster. All services (Docker container).

I will use 3 servers (1 Controller Node+ 2 Compute Node) and 3 network subnets with difference usage.
1. Provider Network: Provide floating IP for Openstack Instance. There is should be no IP address attached to this network ranges.
2. Storage Network: Network for ceph services communication.
3. Openstack Network: Network for openstack internal services communication.

![Design Topology](/static/assets/img/blog/openstack-kolla/2020-os_topology.png)


### IP Information

<table style="width: 260px; height: 92px;">
<tbody>
<tr style="height: 26px;">
<td style="width: 66px; height: 26px; text-align: center;"><strong>Hostname</strong></td>
<td style="width: 74px; height: 26px; text-align: center;"><strong>Storage IP</strong></td>
<td style="width: 98px; height: 26px; text-align: center;"><strong>Openstack IP</strong></td>
</tr>
<tr style="height: 13px;">
<td style="width: 66px; height: 13px; text-align: center;">server0</td>
<td style="width: 74px; height: 13px; text-align: center;">10.50.50.10</td>
<td style="width: 98px; height: 13px; text-align: center;">10.51.51.10</td>
</tr>
<tr style="height: 13px;">
<td style="width: 66px; height: 13px; text-align: center;">server1</td>
<td style="width: 74px; height: 13px; text-align: center;">10.50.50.11</td>
<td style="width: 98px; height: 13px; text-align: center;">10.51.51.11</td>
</tr>
<tr style="height: 13px;">
<td style="width: 66px; height: 13px; text-align: center;">server2</td>
<td style="width: 74px; height: 13px; text-align: center;">10.50.50.12</td>
<td style="width: 98px; height: 13px; text-align: center;">10.51.51.12</td>
</tr>
</tbody>
</table>
</br>


# Deploy Ceph Cluster

In this first stage, I will deploy Ceph Cluster using Ceph Ansible. Ceph Ansible has two different approach when deploy Ceph Cluster: 1) Systemd based and 2) Container based. In here, I will use container based deployment which is use Podman as container runtime.

### 1. Install required packages
```
yum install git python3-pip vim -y
pip3 install ansible
```
### 2. Clone Ceph Ansible 
```
git clone https://github.com/ceph/ceph-ansible
cd ceph-ansible
git checkout stable-5.0
```

### 3. Copy required files
```
cp site-container.yml.sample site-container.yml
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
```
### 4. Create inventory


```
vim inventory 
```
```
[mons]
10.50.50.1[0:2]

[mgrs]
10.50.50.1[0:2]

[grafana-server]
10.50.50.10

[osds]
10.50.50.1[0:2]

[mdss]
10.50.50.10

[clients]
10.50.50.10
```

### 5. Configure ceph ansible variable
```
vim group_vars/all.yml
```
```
ceph_origin: repository
ceph_repository: community
ceph_repository_type: cdn
ceph_stable_release: octopus
containerized_deployment: true
ceph_docker_image: ceph/daemon
ceph_docker_image_tag: latest-octopus
ceph_docker_registry: docker.io

copy_admin_key: True
cephx: true

public_network: 10.50.50.0/24
cluster_network: 10.50.50.0/24
monitor_interface: enp1s0
dashboard_admin_password: yasmin88
grafana_admin_password: yasmin88
```
```
vim group_vars/osds.yml
```
```
osd_scenario: non-collocated
osd_objectstore: bluestore
devices:
  - /dev/vdb
  - /dev/vdc
dedicated_devices:
  - /dev/vdd
  - /dev/vdd
  - /dev/vdd
```
```
vim site-container.yml
```
```
...
- hosts:
  - mons
  - osds
  - mdss
  - clients
  - mgrs
  - grafana-server
...

```
### 6. Deploy containerized ceph cluster
```
ansible-playbook -i inventory site-container.yml
```


### 7. Install epel-release and ceph client on each node
```
yum install epel-release -y
rpm -ivh https://download.ceph.com/rpm-octopus/el8/noarch/ceph-release-1-1.el8.noarch.rpm
yum install ceph-common -y
```
## 8. Verify Ceph Cluster
```
$ ceph -s

  cluster:
    id:     b9a5eeab-b69a-4779-a990-fd76d7856205
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum server0,server1,server2 (age 27h)
    mgr: server1(active, since 8d), standbys: server2, server0
    mds: cephfs:1 {0=server0=up:active}
    osd: 6 osds: 6 up (since 8d), 6 in (since 8d)
 
  task status:
    scrub status:
        mds.server0: idle
 
  data:
    pools:   7 pools, 145 pgs
    objects: 972 objects, 3.1 GiB
    usage:   134 GiB used, 226 GiB / 360 GiB avail
    pgs:     145 active+clean
```
---
