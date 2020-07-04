---
layout: post
title:  "Container based Openstack Deployment using Ceph Ansible and Kolla Ansible 3"
comments: true
date:   2020-07-05
desc: "Openstack Series"
keywords: "linux,openstack,docker,ceph storage"
categories: [Cloud]
tags: [ceph,openstack,cloud]
icon: icon-shell
---

# Openstack Cluster Deployment

I have created Ceph volume in the [late post](https://rahmatawe.com/cloud/2020/07/04/Containerized-Openstack-Infrastructure-2.html). Then, in this post I will deploy Openstack Cluster using Kolla Ansible 10.0. It should be noted that each Kolla Ansible version used for spesific Openstack release. Kolla Ansible 9.x is used to deploy Openstack Train and Kolla Ansible 10.0 is used to deploy Openstack Ussuri. Prior to its release note, Kolla Ansible has support for OVN as ML2 mechanism. By default, Kolla Ansible will used Openvswitch but in here I will try to use OVN.

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

### Note
* By default OVN will use Geneve as protocol for overlay network. In Openvswitch usually we use VXLAN.
* There is parameter `neutron_ovn_distributed_fip` on globals.yml to activate distributed floating IP feature from OVN. Bye default Kolla Ansible disabled it (on another deployment tool it will be activated by default). When I activated it, I can't access Internet (maybe there are some configuration should be added).
