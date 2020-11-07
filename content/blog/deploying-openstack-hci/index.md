---
draft: false
resources: []
title: Deploying HCI Openstack using Kolla Ansible
date: '2020-10-17'
categories:
  - cloud
description: Openstack HCI Implementation Guide
series: []
tags:
  - linux
  - openstack
  - ceph

---

<img src=imgs/2020-os_topology.png>

### IP Information
<table class="table table-striped">
<tbody>
<tr>
<td><strong>Hostname</strong></td>
<td><strong>Storage Interface (enp1s0)</strong></td>
<td><strong>Openstack Interface (enp7s0)</strong></td>
</tr>
<tr>
<td>server0</td>
<td>10.50.50.10</td>
<td>10.51.51.10</td>
</tr>
<tr>
<td>server1</td>
<td>10.50.50.11</td>
<td>10.51.51.11</td>
</tr>
<tr>
<td>server2</td>
<td>10.50.50.12</td>
<td>10.51.51.12</td>
</tr>
</tbody>
</table>
</br>

# A. Deploy Ceph Cluster
In this first stage, I will deploy Ceph Cluster using Ceph Ansible. Ceph Ansible has two different approach when deploy Ceph Cluster: 1) Systemd based and 2) Container based. In here, I will use container based deployment which is use Podman as container runtime.

### 1. Install required packages
```bash
yum install git python3-pip vim -y
pip3 install ansible
```
### 2. Clone Ceph Ansible 
```bash
git clone https://github.com/ceph/ceph-ansible
cd ceph-ansible
git checkout stable-5.0
```

### 3. Copy required files
```bash
cp site-container.yml.sample site-container.yml
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
```
### 4. Create inventory
```bash
vim inventory 
```
```yaml
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
```bash
vim group_vars/all.yml
```
```yaml
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
```bash
vim group_vars/osds.yml
```
```yaml
osd_objectstore: bluestore
devices:
  - /dev/vdb
  - /dev/vdc
```
```bash
vim site-container.yml
```
```yaml
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
```bash
ansible-playbook -i inventory site-container.yml
```

### 7. Install epel-release and ceph client on each node
```bash
yum install epel-release -y
rpm -ivh https://download.ceph.com/rpm-octopus/el8/noarch/ceph-release-1-1.el8.noarch.rpm
yum install ceph-common -y
```
### 8. Verify Ceph Cluster
```bash
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

# B. Create Ceph Pool and Client for Openstack

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

# Openstack Cluster Deployment

### 0. Preflight
Ensure each server could be accessed without password. If not, create ssh keypair then share its public key to each servers. I do this from server0.
```
ssh-keygen
```
After keypair is generated, copy its public key to server0,server1 and server2.
```
ssh-copy-id root@server0
ssh-copy-id root@server1
ssh-copy-id root@server2

```

### 1. Install kolla ansible
```
pip3 install kolla-ansible==10.0
```

### 2. Create kolla directory
```
mkdir /etc/kolla
```

### 3. Copy configuration to kolla directory
```
cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/
```
### 4. Setup ansible.cfg
```
mkdir /etc/ansible
vim /etc/ansible/ansible.cfg
```
```
[defaults]
host_key_checking=False
pipelining=True
forks=100
```

### 5. Setup inventory
```
cd
/usr/local/share/kolla-ansible/ansible/inventory/multinode .
vim multinode
```
```
[control]
server[0]

[network]
server[0]

[compute]
server[1:2]

[monitoring]
server[0:2]

[storage]
server[0:2]
```

### 6. Generate Openstack Services Password
```
kolla-genpwd
```
### 7. Setup Global Variables

In here, I use 2 VIP address for internal services access (10.51.51.100) and public access (10.51.51.101). Both of them should be on the same subnet. Parameter `neutron_external_interface` is used to define interface that will be used as provider network (for floating IP).
```
vim /etc/kolla/globals.yml
```
```
---
kolla_base_distro: "centos"
kolla_install_type: "binary"
openstack_release: "ussury"
kolla_internal_vip_address: "10.51.51.100"
kolla_external_vip_address: "10.51.51.101"
kolla_enable_tls_external: "yes"
storage_interface: "enp1s0"
network_interface: "enp7s0"
neutron_external_interface: "enp8s0"
neutron_plugin_agent: "ovn"
enable_openstack_core: "yes"
enable_cinder: "yes"
enable_haproxy: "yes"
enable_neutron_provider_networks: "yes"
nova_compute_virt_type: "kvm"


enable_ceph: "no"
glance_backend_ceph: "yes"
cinder_backend_ceph: "yes"
nova_backend_ceph: "yes"
```

## Openstack Services Configuration

### 1. Create Glance Configuration Directory
```
mkdir -p /etc/kolla/config/glance/
```

### 2. Copy ceph configuration for glance to kolla
```
cp /etc/ceph/ceph.conf /etc/kolla/config/glance/
cp /etc/ceph/ceph.client.glance.keyring /etc/kolla/config/glance/
```

### 3. Create Cinder Configuration Directory
```
mkdir -p /etc/kolla/config/cinder/cinder-volume
mkdir /etc/kolla/config/cinder/cinder-backup
```
### 4. Copy ceph configuration for cinder to kolla
```
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/cinder/cinder-volume/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/cinder/cinder-backup/
cp /etc/ceph/ceph.client.cinder-backup.keyring /etc/kolla/config/cinder/cinder-backup/
```

### 5. Create Nova Configuration Directory
```
mkdir /etc/kolla/config/nova
```
### 6. Copy ceph configuration for nova to kolla
```
cp /etc/ceph/ceph.conf /etc/kolla/config/nova/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/nova/
cp /etc/ceph/ceph.client.cinder.keyring /etc/kolla/config/nova/ceph.client.nova.keyring
```

### 7. Add Glance Configuration
```
vim /etc/kolla/config/glance/glance-api.conf
```
```
[glance_store]
stores = rbd
default_store = rbd
rbd_store_pool = images
rbd_store_user = glance
rbd_store_ceph_conf = /etc/ceph/ceph.conf
```

### 8. Add Cinder Configuration
```
cat /etc/kolla/passwords.yml |grep cinder_rbd_secret_uuid
vim /etc/kolla/config/cinder.conf
```
```
[DEFAULT]
enabled_backends=rbd-1
backup_ceph_conf=/etc/ceph/ceph.conf
backup_ceph_user=cinder-backup
backup_ceph_chunk_size = 134217728
backup_ceph_pool=backups
backup_driver = cinder.backup.drivers.ceph.CephBackupDriver
backup_ceph_stripe_unit = 0
backup_ceph_stripe_count = 0
restore_discard_excess_bytes = true

[rbd-1]
rbd_ceph_conf=/etc/ceph/ceph.conf
rbd_user=cinder
backend_host=rbd:volumes
rbd_pool=volumes
volume_backend_name=rbd-1
volume_driver=cinder.volume.drivers.rbd.RBDDriver
rbd_secret_uuid = 3f2ee568-97a8-4416-9865-a4567aa69b53 #add here
```
### 9. Add nova configuration
```
vim /etc/kolla/config/nova.conf
```
```
[libvirt]
images_rbd_pool=vms
images_type=rbd
images_rbd_ceph_conf=/etc/ceph/ceph.conf
rbd_user=cinder
```

### 10. Add Heat Configuration
```
vim /etc/kolla/config/heat.conf
```
```
[DEFAULT]
heat_metadata_server_url = http://10.51.51.100:8000
heat_waitcondition_server_url = http://10.51.51.100:8000/v1/waitcondition
server_keystone_endpoint_type = internal

[clients]
insecure = true

[clients_cinder]
endpoint_type = internalURL
insecure = true

[clients_glance]
endpoint_type = internalURL
insecure = true

[clients_heat]
endpoint_type = internalURL
insecure = true
url = http://10.51.51.100:8004/v1/%(tenant_id)s

[clients_keystone]
endpoint_type = internalURL
insecure = true
auth_uri = http://10.51.51.100:5000

[clients_neutron]
endpoint_type = internalURL
insecure = true

[clients_nova]
endpoint_type = internalURL
insecure = true

[clients_octavia]
endpoint_type = internalURL
insecure = true
```
### 11. Add neutron configuration
```
vim /etc/kolla/config/neutron.conf
```
```
[DEFAULT]
core_plugin = neutron.plugins.ml2.plugin.Ml2Plugin
service_plugins=trunk,ovn-router
router_scheduler_driver=neutron.scheduler.l3_agent_scheduler.ChanceScheduler
notify_nova_on_port_status_changes=True
notify_nova_on_port_data_changes=True
```
```
mkdir /etc/kolla/config/neutron
vim /etc/kolla/config/neutron/ml2_conf.ini
```
```
[ml2]
mechanism_drivers = ovn

[securitygroup]
enable_security_group=True
firewall_driver=neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

### 12. Generate Self Signed Certificate for Openstack
```
kolla-ansible -i multinode certificates
```

### 13. Deployment OpenStack
```
kolla-ansible -i multinode bootstrap-servers
kolla-ansible -i multinode prechecks
kolla-ansible -i multinode deploy
kolla-ansible -i multinode post-deploy
```

### 18. Testing Fungsional
It is a script that will create private network,public network, cirros image, and flavors. You can download it in [here](https://gist.github.com/riupie/0fa2f94707c1a61cab85d15b7673deab).
```
./init-runonce
```
