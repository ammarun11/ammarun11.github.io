---
title: "[Openstack] Integrate CEPH in Kolla OPENSTACK"
date: 2020-09-10
categories: [ngoprek, server, cloud, security]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Integrate CEPH in Kolla OPENSTACK

### CEPH x OPENSTACK
Detail Virtual Machine : 
* am-os01 (10.10.176.110) : 8 vCPU, 12 GB RAM, 40 GB HDD1 ,20 GB HDD2 ,20 GB HDD3 ,20 GB HDD4
* am-os02 (10.10.176.111) : 8 vCPU, 12 GB RAM, 40 GB HDD1 ,20 GB HDD2 ,20 GB HDD3 ,20 GB HDD4
* am-os03 (10.10.176.112) : 8 vCPU, 12 GB RAM, 40 GB HDD1 ,20 GB HDD2 ,20 GB HDD3 ,20 GB HDD4

### Add DNS
> nano /etc/hosts
```s
10.10.176.110 am-os01
10.10.176.111 am-os02
10.10.176.112 am-os03
```
### am-os01 ###

### Create ssh key on am-os01 and copy public key to all node
> ssh-keygen
```bash
for i in {1..3}; do 
  ssh-copy-id -i ~/.ssh/id_rsa.pub root@am-os0$i; 
done
```

### Verify connectivity
```bash
ping -c 3 am-os01
ping -c 3 am-os02
ping -c 3 am-os03
```

### Prepare disk for CEPH on am-os0*
```bash
for i in {1..3}; do
  for node in os0 ; do
    ssh root@am-${node}${i} parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_BS 1 -1
    ssh root@am-${node}${i} parted /dev/vdc -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_BS 1 -1
    ssh root@am-${node}${i} parted /dev/vdd -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP_BS 1 -1
  done
done
```

### Verify Disk
```bash
ssh am-os01 lsblk
ssh am-os02 lsblk
ssh am-os03 lsblk
```
### Update package node
```bash
for i in {1..3}; do
  for node in os0; do
    ssh root@am-${node}${i} apt-get install python-dev libffi-dev gcc libssl-dev python-selinux python-setuptools python-pip python3-pip -y
  done
done
```

```bash
pip install -U pip
pip install ansible
pip install kolla-ansible==
pip install kolla-ansible==9.0.1
```
### Configure kolla-ansible
> sudo mkdir -p /etc/kolla

> sudo chown $USER:$USER /etc/kolla

> cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla

> cp /usr/local/share/kolla-ansible/ansible/inventory/* .


### configure ansible 
> sudo mkdir /etc/ansible

> nano /etc/ansible/ansible.cfg
```bash
[defaults]
host_key_checking=False
pipelining=True
forks=100
```
### Generate password.yml
```
kolla-genpwd
```

### Configure inventory
> nano multinode

```
[control]
am-os01

[network]
am-os01

[compute]
am-os0[1:3]

[monitoring]
am-os0[1:3]

[storage]
am-os0[1:3]
```
### setup globals.yaml kolla-ansible

> nano /etc/kolla/globals.yml

```bash
---
kolla_base_distro: "ubuntu"
kolla_install_type: "source"
openstack_release: "train"
kolla_internal_vip_address: "10.10.176.100"
kolla_external_vip_address: "10.10.176.200"
network_interface: "ens3"
neutron_external_interface: "ens4"
storage_interface: "ens5"
neutron_plugin_agent: "openvswitch" 
enable_ceph: "yes"
enable_cinder: "yes"
enable_cinder_backup: "no"
enable_neutron_provider_networks: "yes"
ceph_pool_pg_num: 30
ceph_pool_pgp_num: 30
cinder_backend_ceph: "yes"
glance_backend_ceph: "yes"
glance_backend_file: "no"
nova_compute_virt_type: "kvm"
```

### Verify ssh connectivity all node
```bash
ansible all -i multinode -m ping
```
### Bootstrap all node 
```
kolla-ansible -i multinode bootstrap-servers
kolla-ansible -i multinode prechecks
kolla-ansible -i multinode deploy
kolla-ansible -i multinode post-deploy
```

### Install Openstack CLI
```
apt -y install python3-openstackclient
```
### Verify Openstack
```
source /etc/kolla/admin-openrc.sh
openstack hypervisor list
```
### inti-runonce edit network with ext-net
# inti-runonce dan edit network with ext-net
```
cp /usr/local/share/kolla-ansible/init-runonce .
```
# ubah variable network sesuaikan dengan network external klen

> nano init-runonce

```
./init-runonece
```

### Add Ceph Repository
```bash
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
echo deb https://download.ceph.com/debian-nautilus/ $(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/ceph.list

apt-update -y

apt install ceph-common
```


### Verify
```bash
ceph -s
rados df
```
## DEMO ##

### Creating Openstack glance a image on ceph RBD ###
```bash
wget -c https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img
openstack image create --disk-format qcow2 --file focal-server-cloudimg-amd64.img --public Ubuntu20.04.img
```
### Verify
```bash
openstack image list
rbd -p images ls
```
### Creating Openstack Cinder Volume (Storaed on Ceph RBD) -Attaching this volume to openstack instance

```bash
openstack server create  --image $image  --flavor m1.tiny  --key-name mykey  --network demo-net $nameVM
```

### Mount volume 
```bash
mkfs.xfs /dev/vdb
mkdir -p /mnt/palo

mount /dev/vdb /mnt/palo
```

### - Openstack instance LIVE MIGRATION

## CEK compute & instance dan pilih instance yang akan di pindah ke compute yang berbeda
```
openstack hypervisor list
```
```
nova hypervisor-servers am-controller0*
```
# masuk ke instance jalankan proses berikut 
```
dd if=/dev/zero of=/mnt/palo/file1 bs=1M count=9000
```
```
nova live-migration $idvm $nameHypervisor
```

*sambil di pantau target compute yg bakal pindah 
```
watch hypervisor-server 
```


**Referensi**
* [https://docs.openstack.org/kolla-ansible/rocky/reference/ceph-guide.html](https://docs.openstack.org/kolla-ansible/rocky/reference/ceph-guide.html)
* [hhttps://docs.openstack.org/kolla-ansible/train/](hhttps://docs.openstack.org/kolla-ansible/train/s)

# Happy,  Enjoy Ngoprek ~
