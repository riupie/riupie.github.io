---
layout: post
title:  "Openstack DVR-HA "
comments: true
date:   2020-07-03
desc: "Openstack Series"
keywords: "linux,openstack,docker,ceph storage"
categories: [Cloud]
tags: [ceph,openstack,cloud]
icon: icon-shell
---


# Environment
```
- Centos 8
- Ansible 2.9
- Python 3.6
```
# Deploy Ceph Cluster

### 1. Install required packages
```
yum install epel-release git python3-pip vim -y
yum update -y
yum install ansible screen -y
pip3 install netaddr

```

### 2. Share public key from deployer node to each ceph nodes
```
sshpass -p yasmin@88 ssh-copy-id 10.150.150.10
sshpass -p yasmin@88 ssh-copy-id 10.150.150.11
sshpass -p yasmin@88 ssh-copy-id 10.150.150.12
sshpass -p yasmin@88 ssh-copy-id 10.150.150.20
sshpass -p yasmin@88 ssh-copy-id 10.150.150.21
```

### 4. Clone Ceph Ansible 
```
git clone https://github.com/ceph/ceph-ansible
cd ceph-ansible
git checkout stable-5.0
```

### 5. Copy required files
```
cp site-container.yml.sample site-container.yml
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
```
### 6. Create inventory
```
vim inventory 
```
```
[mons]
10.150.150.1[0:2]

[mgrs]
10.150.150.1[0:2]

[grafana-server]
10.150.150.10

[osds]
10.150.150.1[0:2]
10.150.150.2[0:1]
```

### 7. Configure ceph ansible variable
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

ntp_service_enabled: true
ntp_daemon_type: chronyd

public_network: 10.151.151.0/24
cluster_network: 10.150.150.0/24
monitor_interface: ens9
dashboard_admin_password: yasmin88
grafana_admin_password: yasmin88

openstack_glance_pool:
  name: "images"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "rbd"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False
openstack_cinder_pool:
  name: "volumes"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "rbd"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False
openstack_nova_pool:
  name: "vms"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "rbd"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False
openstack_cinder_backup_pool:
  name: "backups"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "rbd"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False

openstack_cephfs_data_pool:
  name: "manila_data"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "cephfs"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False
openstack_cephfs_metadata_pool:
  name: "manila_metadata"
  pg_num: "{{ osd_pool_default_pg_num }}"
  pgp_num: "{{ osd_pool_default_pg_num }}"
  rule_name: "replicated_rule"
  type: 1
  erasure_profile: ""
  expected_num_objects: ""
  application: "cephfs"
  size: "{{ osd_pool_default_size }}"
  min_size: "{{ osd_pool_default_min_size }}"
  pg_autoscale_mode: False

openstack_pools:
  - "{{ openstack_glance_pool }}"
  - "{{ openstack_cinder_pool }}"
  - "{{ openstack_nova_pool }}"
  - "{{ openstack_cinder_backup_pool }}"
  - "{{ openstack_cephfs_data_pool }}"
  - "{{ openstack_cephfs_metadata_pool }}"

openstack_keys:
  - { name: client.glance, caps: { mon: "profile rbd", osd: "profile rbd pool={{ openstack_cinder_pool.name }}, profile rbd pool={{ openstack_glance_pool.name }}"}, mode: "0600" }
  - { name: client.cinder, caps: { mon: "profile rbd", osd: "profile rbd pool={{ openstack_cinder_pool.name }}, profile rbd pool={{ openstack_nova_pool.name }}, profile rbd pool={{ openstack_glance_pool.name }}"}, mode: "0600" }
  - { name: client.cinder-backup, caps: { mon: "profile rbd", osd: "profile rbd pool={{ openstack_cinder_backup_pool.name }}"}, mode: "0600" }
```
```
vim group_vars/osds.yml
```
```
osd_objectstore: bluestore
devices:
  - /dev/vdb
  - /dev/vdc
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
  - mgrs
  - grafana-server
...

```

### Update All Nodes
### 8. Deploy containerized ceph cluster
```
screen -R ceph
ansible-playbook -i inventory site-container.yml
```

### 9. Install epel-release and ceph client on each node
```
rpm -ivh https://download.ceph.com/rpm-octopus/el8/noarch/ceph-release-1-1.el8.noarch.rpm
yum install ceph-common -y
```
### 10. Verify Ceph Cluster
```
$ ceph -s
  cluster:
    id:     31aaeb98-c2f2-416b-a373-d1ed290a6f46
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ag-controller0,ag-controller1,ag-controller2 (age 86m)
    mgr: ag-controller2(active, since 82m), standbys: ag-controller1, ag-controller0
    osd: 15 osds: 15 up (since 84m), 15 in (since 84m)
 
  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 0 B
    usage:   15 GiB used, 585 GiB / 600 GiB avail
    pgs:     1 active+clean
```

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
ssh 10.150.150.11 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh 10.150.150.12 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh 10.150.150.20 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh 10.150.150.21 sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf

```

### 3. Setup Client Configuration
```
ceph auth get-or-create client.glance mon 'profile rbd' osd 'profile rbd pool=images' mgr 'profile rbd pool=images'
ceph auth get-or-create client.cinder mon 'profile rbd' osd 'profile rbd pool=volumes, profile rbd pool=vms, profile rbd-read-only pool=images' mgr 'profile rbd pool=volumes, profile rbd pool=vms'
ceph auth get-or-create client.cinder-backup mon 'profile rbd' osd 'profile rbd pool=backups' mgr 'profile rbd pool=backups'
```

### 4. Setup Ceph Client
```
for SERVERS in 10.150.150.10 10.150.150.11 10.150.150.12;do \
ceph auth get-or-create client.glance | ssh $SERVERS sudo tee /etc/ceph/ceph.client.glance.keyring; \
ceph auth get-or-create client.cinder | ssh $SERVERS sudo tee /etc/ceph/ceph.client.cinder.keyring; \
ceph auth get-or-create client.cinder-backup | ssh $SERVERS sudo tee /etc/ceph/ceph.client.cinder-backup.keyring;done
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
cp /usr/local/share/kolla-ansible/ansible/inventory/multinode .
vim multinode
```
```
[control]
10.151.151.1[0:2]

[network]
10.151.151.1[0:2]

[compute]
10.151.151.1[0:2]
10.151.151.2[0:1]

[monitoring]
10.151.151.1[0:2]

[storage]
10.151.151.1[0:2]
10.151.151.2[0:1]
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
# Openstack version
kolla_base_distro: "centos"
kolla_install_type: "source"
openstack_release: "ussuri"

#VRRP IP
kolla_internal_vip_address: "10.152.152.100"
kolla_external_vip_address: "10.151.151.100"
kolla_enable_tls_external: "yes"

#Define Inteface
storage_interface: "ens9"
kolla_external_vip_interface: "ens9"
network_interface: "ens10"
neutron_external_interface: "ens11"

# Services to be installed. Core services: glance, keystone, neutron, nova, heat, and horizon
enable_openstack_core: "yes"
enable_cinder: "yes"
enable_haproxy: "yes"
enable_masakari: "yes"
enable_octavia: "yes"

#Neutron Features
neutron_plugin_agent: "openvswitch"
enable_neutron_provider_networks: "yes"
enable_neutron_qos: "yes"
enable_neutron_agent_ha: "yes"
enable_neutron_port_forwarding: "yes"
enable_neutron_dvr: "yes"

#Disable internal Ceph and logging
enable_ceph: "no"
enable_central_logging: "no"
enable_fluentd: "no"

# Virtualization Type
nova_compute_virt_type: "kvm"

# Ceph Integration with Image, Volume and VMs
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
cp /etc/ceph/ceph.conf /etc/kolla/config/cinder/
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
[DEFAULT]
resume_guests_state_on_host_boot = True

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
heat_metadata_server_url = http://10.152.152.100:8000
heat_waitcondition_server_url = http://10.152.152.100:8000/v1/waitcondition
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
url = http://10.152.152.100:8004/v1/%(tenant_id)s

[clients_keystone]
endpoint_type = internalURL
insecure = true
auth_uri = http://10.152.152.100:5000

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

### 11. Generate Self Signed Certificate for Openstack
```
kolla-ansible -i multinode certificates
```

### 12. Deployment OpenStack
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

## Load Balancer as a Service (LBaaS) Octavia
### 1. Build Amphora Image
```bash
yum install -y debootstrap qemu-img
git clone https://opendev.org/openstack/octavia -b stable/ussuri
grep octavia_ca_password /etc/kolla/passwords.yml
sed -i 's/not-secure-passphrase/R6W9WpGAfOAqiXjEFT2IQEBn8PSHJQyISv2B1utE/g' octavia/bin/create_single_CA_intermediate_CA.sh
pip3 install diskimage-builder
cd octavia/diskimage-create
screen -R build-image
./diskimage-create.sh

```

### 2. Upload amphora image to service project
```bash
openstack image create amphora-x64-haproxy --project service --container-format bare --disk-format qcow2 --private --tag amphora --file amphora-x64-haproxy.qcow2
```

### 3. Create Amphora Flavor
```bash
openstack flavor create amphora --project service --vcpus 1 --ram 1024 --disk 2 --id 200 --private
```
### 4. Generate Certificate
```bash
cd ~/octavia/bin
./create_single_CA_intermediate_CA.sh
mkdir /etc/kolla/config/octavia
cp single_ca/etc/octavia/certs/* /etc/kolla/config/octavia
```
### 5. Create Security Group for Octavia
```bash
openstack security group create lb-mgmt-sec-grp --project service
openstack security group rule create --protocol icmp lb-mgmt-sec-grp --project service
openstack security group rule create --protocol tcp --dst-port 22 --ingress lb-mgmt-sec-grp --project service
openstack security group rule create --protocol tcp --dst-port 9443 --ingress lb-mgmt-sec-grp --project service
openstack security group rule create --protocol udp --dst-port 5555 --egress lb-mgmt-sec-grp --project service
openstack security group rule create --protocol tcp --dst-port 80 --ingress lb-mgmt-sec-grp --project service


openstack security group create lb-health-mgr-sec-grp --project service
openstack security group rule create --protocol udp --dst-port 5555 lb-health-mgr-sec-grp --project service
```
### 6. Create Amphora Management Network
```bash
openstack network create lb-mgmt-net --project service
openstack subnet create --subnet-range $OCTAVIA_MGMT_SUBNET --allocation-pool start=$OCTAVIA_MGMT_SUBNET_START,end=$OCTAVIA_MGMT_SUBNET_END --network lb-mgmt-net lb-mgmt-subnet --project service

SUBNET_ID=$(openstack subnet show lb-mgmt-subnet --project service -f value -c id) 
PORT_FIXED_IP="--fixed-ip subnet=$SUBNET_ID,ip-address=$OCTAVIA_MGMT_PORT_IP"

MGMT_PORT_ID=$(openstack port create --security-group lb-health-mgr-sec-grp --device-owner Octavia:health-mgr --host=$(hostname) -c id -f value --network lb-mgmt-net $PORT_FIXED_IP octavia-health-manager-listen-port)

MGMT_PORT_MAC=$(openstack port show -c mac_address -f value $MGMT_PORT_ID)

ovs-vsctl -- --may-exist add-port br-int o-hm0 -- set Interface o-hm0 type=internal -- set Interface o-hm0 external-ids:iface-status=active -- set Interface o-hm0 external-ids:attached-mac=$MGMT_PORT_MAC -- set Interface o-hm0 external-ids:iface-id=$MGMT_PORT_ID

ip link set dev o-hm0 address $MGMT_PORT_MAC  
vim dhclient.conf
...
request subnet-mask,broadcast-address,interface-mtu;
do-forward-updates false;
...
dhclient -v o-hm0 -cf dhclient.conf

ip address
ip link 
ovs-vsctl show
```

### 7. Add new parameter to globals.yml
```
#Octavia tuning
enable_octavia: "yes"
octavia_loadbalancer_topology: "SINGLE"
octavia_amp_flavor_id: 200
octavia_amp_boot_network_list: 9e24417f-a8e9-4627-a726-5773f327782b
octavia_amp_secgroup_list: e45a8aff-dfe7-4edb-b058-989a37cb6f17
```
### 8. Create RC file for Octavia user
```
openstack project list
SERVICE_PROJECT_ID=10fca916e30545fa864bee64a738406e
KEYSTONE_OCTAVIA_PASSWORD=`grep octavia_keystone_password /etc/kolla/passwords.yml | cut -f2 -d" "`
```
```bash
cat <<EOF > octavia-openrc
export OS_PROJECT_DOMAIN_ID=default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=service
export OS_PROJECT_ID=$SERVICE_PROJECT_ID
export OS_USERNAME=octavia
export OS_PASSWORD=$KEYSTONE_OCTAVIA_PASSWORD
export OS_AUTH_URL=https://10.151.151.100:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export OS_REGION_NAME="RegionOne"
export OS_CACERT=/etc/kolla/certificates/haproxy-ca.crt
EOF
```
### 5. Create Keypair for Amphora
```bash
source octavia-openrc
openstack keypair create --public-key /root/.ssh/id_rsa.pub octavia_ssh_key
```

---
