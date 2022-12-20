---
title: "[Openstack] Deploy Openstack Manual but With ansible, Openstack Ussuri Version"
date: 2022-12-20
categories: [ngoprek, server, cloud, openstack, ceph, storage]
tags:
  - Jekyll
  - update
---

#### Spesification

| Hostname | VCPUs | RAM | DISK |
| --- | --- | --- | --- |
| lab-r01-oscontroller-01 | 8 | 32 | 100Gb x2 |
| lab-r02-oscontroller-02 | 8 | 32 | 100Gb x2 |
| lab-r03-oscontroller-03 | 8 | 32 | 100Gb x2 |
| lab-r01-oscompute-01 | 16 | 32 | 100Gb x6 |
| lab-r01-oscompute-02 | 16 | 32 | 100Gb x6 |
| lab-r01-oscompute-02 | 16 | 32 | 100Gb x6 |


#### Network Mapping
#### Network Mapping

| Compute | Interface Name | Role | IP |
| --- | --- | --- | --- |
| lab-r01-oscontroller-01 | ens3 | Management Network & Public API Network | 172.18.251.11 |
| lab-r01-oscontroller-01 | ens4 | Internal and Admin API Network | 172.18.236.11 |
| lab-r01-oscontroller-01 | ens5 | Self-service Network | 172.18.237.11 |
| lab-r01-oscontroller-01 | ens6 | External Provider Prod Network | not-assigned |
| lab-r01-oscontroller-01 | ens7 | External Provider Dev Network | not-assigned |
| lab-r01-oscontroller-01 | ens8 | Storage Ceph Public Network | 172.18.238.11 |
| lab-r01-oscontroller-01 | ens9 | Storage Ceph Cluster Network | 172.18.239.11 |
| lab-r02-oscontroller-02 | ens3 | Management Network & Public API Network | 172.18.251.12 |
| lab-r02-oscontroller-02 | ens4 | Internal and Admin API Network | 172.18.236.12 |
| lab-r02-oscontroller-02 | ens5 | Self-service Network | 172.18.237.12 |
| lab-r02-oscontroller-02 | ens6 | External Provider Prod Network | not-assigned |
| lab-r02-oscontroller-02 | ens7 | External Provider Dev Network | not-assigned |
| lab-r02-oscontroller-02 | ens8 | Storage Ceph Public Network | 172.18.238.12 |
| lab-r02-oscontroller-02 | ens9 | Storage Ceph Cluster Network | 172.18.239.12 |
| lab-r03-oscontroller-03 | ens3 | Management Network & Public API Network | 172.18.251.13 |
| lab-r03-oscontroller-03 | ens4 | Internal and Admin API Network | 172.18.236.13 |
| lab-r03-oscontroller-03 | ens5 | Self-service Network | 172.18.237.13 |
| lab-r03-oscontroller-03 | ens6 | External Provider Prod Network | not-assigned |
| lab-r03-oscontroller-03 | ens7 | External Provider Dev Network | not-assigned |
| lab-r03-oscontroller-03 | ens8 | Storage Ceph Public Network | 172.18.238.13 |
| lab-r03-oscontroller-03 | ens9 | Storage Ceph Cluster Network | 172.18.239.13 |
| lab-r01-oscompute-01 | ens3 | Management Network & Public API Network | 172.18.251.21 |
| lab-r01-oscompute-01 | ens4 | Internal and Admin API Network | 172.18.236.21 |
| lab-r01-oscompute-01 | ens5 | Self-service Network | 172.18.237.21 |
| lab-r01-oscompute-01 | ens6 | External Provider Prod Network | 172.18.216.21 |
| lab-r01-oscompute-01 | ens7 | External Provider Dev Network | 172.18.224.21 |
| lab-r01-oscompute-01 | ens8 | Storage Ceph Public Network | 172.18.238.21 |
| lab-r01-oscompute-01 | ens9 | Storage Ceph Cluster Network | 172.18.239.21 |
| lab-r01-oscompute-02 | ens3 | Management Network & Public API Network | 172.18.251.22 |
| lab-r01-oscompute-02 | ens4 | Internal and Admin API Network | 172.18.236.22 |
| lab-r01-oscompute-02 | ens5 | Self-service Network | 172.18.237.22 |
| lab-r01-oscompute-02 | ens6 | External Provider Prod Network | 172.18.216.22 |
| lab-r01-oscompute-02 | ens7 | External Provider Dev Network | 172.18.224.22 |
| lab-r01-oscompute-02 | ens8 | Storage Ceph Public Network | 172.18.238.22 |
| lab-r01-oscompute-02 | ens9 | Storage Ceph Cluster Network | 172.18.239.22 |
| lab-r01-oscompute-03 | ens3 | Management Network & Public API Network | 172.18.251.23 |
| lab-r01-oscompute-03 | ens4 | Internal and Admin API Network | 172.18.236.23 |
| lab-r01-oscompute-03 | ens5 | Self-service Network | 172.18.237.23 |
| lab-r01-oscompute-03 | ens6 | External Provider Prod Network | 172.18.216.23 |
| lab-r01-oscompute-03 | ens7 | External Provider Dev Network | 172.18.224.23 |
| lab-r01-oscompute-03 | ens8 | Storage Ceph Public Network | 172.18.238.23 |
| lab-r01-oscompute-03 | ens9 | Storage Ceph Cluster Network | 172.18.239.23 |


Services|VIP
-|-
Internal VIP|172.18.236.100
External VIP|172.18.251.100
Horizon|172.18.251.100/horizon

Interface Name|Role|CIDR|IP
-|-|-|-
ens3|Management Network & Public API Network|172.18.251.0/24|172.18.251.11-13
ens4|Internal and Admin API Network|172.18.236.0/24|172.18.236.11-13
ens5|Self-service Network|172.18.237.0/24|172.18.237.11-13
ens6|External Provider Prod Network|172.18.216.0/24|172.18.216.0-254
ens7|External Provider Dev Network|172.18.224.0/24|172.18.224.0-254
ens8|Storage Ceph Public Network|172.18.238.0/24|172.18.238.0-254
ens9|Storage Ceph Cluster Network|172.18.239.0/24|172.18.239.0-254


#### Ceph Deployment
###### 1. Copy SSH key each node
###### 2. Configure `/etc/host`

```bash
172.18.251.11 lab-r01-oscontroller-01
172.18.251.12 lab-r02-oscontroller-02
172.18.251.13 lab-r03-oscontroller-03
172.18.251.21 lab-r01-oscompute-01
172.18.251.22 lab-r01-oscompute-02
172.18.251.23 lab-r01-oscompute-03
```

###### 3. Update & Upgrade package

```bash
apt update -y && apt install python3 python3-pip
```

###### 4. Install Ansible

```bash
pip3 install ansible
```

###### 5. Git Clone Repo

```bash
git clone https://go.lab.id/ammar/ansible-iac-ceph-openstack.git
```

###### 6. Adjust Vars with enviroment (`/ansible-iac-ceph-openstack-main/Ceph-Manual-Ansible/group_vars/all.yml`)

```bash
cluster_name: ceph
cluster_storage_interface: ens9 # lab environment
public_storage_interface: ens8 # lab environment
region_name: dc # lab environment
public_network: 172.18.238.0/24
cluster_network: 172.18.239.0/24


### SSH COPY-id
#if you wanna use this feature make sure you fiel all the ssh variable
ssh_password: testing123
ssh_user: ops


### Ceph dashboard
ceph_dashboard: "no" # yes or no
ceph_username: admin
ceph_password: zHhTKL2:GLcD898^

######
# OSD
######

#### this variable will create osd data
devices:
  - data: "/dev/vdb"
  - data: "/dev/vdc"
  - data: "/dev/vdd"
  - data: "/dev/vde"
  - data: "/dev/vdf"
```
 
###### 7. adjust Inventory with  environment (`/ansible-iac-ceph-openstack-main/Ceph-Manual-Ansible/inventory/hosts`)

```bash
[deployer]
172.18.251.11

[ceph-nodes]
172.18.238.1[1:3]
172.18.238.2[1:3]

[mons]
172.18.238.1[1:3]

[osds]
172.18.238.2[1:3]

[mgrs]
172.18.238.1[1:3]
```

###### 8. Disable role task not used (`/ansible-iac-ceph-openstack-main/Ceph-Manual-Ansible/roles/mgr/tasks/main.yml`)

```bash
---- 
#- name: Enable msgr2
 # shell: "ceph mon enable-msgr2"
#  tags: mgr
------
```

###### 9. Check Connection each Node

```bash
ansible all -m ping
```

###### 10. Boostrap Node

```bash
ansible-playbook bootstrap.yml
```

###### 11. Deploy Ansible ceph

```bash
ansible-playbook site.yml
```

###### 12. add config if mon 2 & 3 not added at config `/etc/ceph/ceph.conf`

```bash
mon initial members = lab-r01-oscontroller-01, lab-r02-oscontroller-02, lab-r03-oscontroller-03
mon host = 172.18.238.11, 172.18.238.12, 172.18.238.13
```

###### 13. Restart Ceph Mon 2,3 & 1 (non-leader first)

```bash
systemctl restart ceph-mon@<hostname>.service
```	

#### Openstack Deployment
###### 1. adjust Vars at environment (`/ansible-iac-ceph-openstack-main/OS-Ansible/group_vars/all.yml`)

```bash
---
#########
# GLOBAL
#########
instance: lab
region_name: dc
exec_user: root

#######
# VIP
#######
internal_vip_address: 172.18.236.100
admin_vip_address: "{{ internal_vip_address }}"
public_vip_address: 172.18.251.100

internal_vip_hostname: internal.{{ region_name }}.lab.co.id
admin_vip_hostname: admin.{{ region_name }}.lab.co.id
public_vip_hostname: public.{{ region_name }}.lab.co.id
#internal_vip_prefix: 24 (Default value)

############
# INTERFACE
############
api_interface: ens4
public_api_interface: ens3
overlay_interface: ens5
production_interface: ens6
developer_interface: ens7


##########
# MARIADB
##########
mysql_old_root_password: ""
mysql_root_password: "mysql!lab"
mysql_default_port: 3306
mysql_clustercheck_password: "clustercheck!lab"
mysql_keystone_password: "keystone!lab"
mysql_glance_password: "glance!lab"
mysql_cinder_password: "cinder!lab"
mysql_nova_password: "nova!lab"
mysql_placement_password: "placement!lab"
mysql_neutron_password: "rahasia"
mysql_heat_password: "heat!lab"
mysql_octavia_password: "octavia!lab"
mysql_magnum_password: "magnum!lab"

################
# OPENSTACK USER
################
rabbitmq_openstack_password: "rabbitmqlab"
pacemaker_hacluster_password: "pacemakerlab"
openstack_admin_password: "rahasia"
openstack_glance_password: "glance!lab"
openstack_cinder_password: "cinder!lab"
openstack_nova_password: "nova!lab"
openstack_placement_password: "placement!lab"
openstack_neutron_password: "rahasia"
openstack_heat_password: "heat!lab"
openstack_heat_domain_admin_password: "heat_domain_admin!lab"
openstack_octavia_password: "octavia!lab"
openstack_magnum_password: "magnum!lab"
openstack_magnum_domain_admin_password: "magnum_domain_admin!lab"


#########
# SECRET
#########
metadata_proxy_secret: "skskdjqoqlaoslod"
rbd_secret_uuid: "f2912ae6-c71c-4fa0-bfb7-4a30798bdf38"


##########
# OCTAVIA
##########
amphora_root_password: amphora
ca_private_key_passphrase: "octavia!lab"
```

###### 2. add Template `50-server.cnf.j2` at folder below (`/ansible-iac-ceph-openstack-main/OS-Ansible/roles/mariadb/templates/`)

```bash
[mysqld]
bind-address = {{ hostvars[inventory_hostname]['ansible_' ~ api_interface]['ipv4']['address'] }}
character-set-server = utf8
collation-server = utf8_general_ci
```

###### 3. add task on mariadb for generate file `50-server.cnf` (`/ansible-iac-ceph-openstack-main/OS-Ansible/roles/mariadb/tasks/ubuntu.yml`)

```bash
- name: Create mariadb 50-server.cnf configuration file
  template:
	src: 50-server.cnf.j2
	dest: /etc/mysql/mariadb.conf.d/50-server.cnf
	mode: '0644'

## Disable task below

#- name: change mariadb bind-address
#  ini_file:
#   path: /etc/mysql/mariadb.conf.d/50-server.cnf
#	section: mysqld
#	option: bind-address
#	value: "{{ hostvars[inventory_hostname]['ansible_' ~ api_interface]['ipv4']['address'] }}"
#	mode: '0644'
#	backup: yes

#- name: change mariadb default character set to utf8
#  ini_file:
#	path: /etc/mysql/mariadb.conf.d/50-server.cnf
#	section: mysqld
#	option: character-set-server
#	value: utf8
#	mode: '0644'

#- name: change mariadb default character set to utf8
#  ini_file:
#	path: /etc/mysql/mariadb.conf.d/50-server.cnf
#	section: mysqld
#	option: collation-server
#	value: utf8_general_ci
#	mode: '0644'
```

###### 4. Deploy Ansible Openstack

```bash
ansible-playbook deploy.yml
```

###### 5. Reclustering ovn
- Backup configuration existing dan add configuration Haproxy. Edit on every controller


```bash
 global
  chroot  /var/lib/haproxy
  daemon
  group  haproxy
  maxconn  4000
  pidfile  /var/run/haproxy.pid
  user  haproxy

defaults
  log  global
  maxconn  4000
  option  redispatch
  retries  3
  timeout  http-request 10s
  timeout  queue 1m
  timeout  connect 10s
  timeout  client 480m
  timeout  server 480m
  timeout  check 10s

# DASHBOARD

 listen dashboard_cluster
  bind 172.18.251.100:80
  bind 172.18.251.100:443 ssl crt /etc/ssl/certs/cert.dc.lab.co.id.pem
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:80 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:80 check inter 2000 rise 2 fall 5 backup
	server lab-r03-oscontroller-03 172.18.236.13:80 check inter 2000 rise 2 fall 5 backup

# GALERA

 listen galera_cluster
  bind 172.18.236.100:3306
  balance  source
  option  httpchk
	server lab-r01-oscontroller-01 172.18.236.11:3306 check port 9200 inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:3306 backup check port 9200 inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:3306 backup check port 9200 inter 2000 rise 2 fall 5


# GLANCE

 listen glance_api_cluster
  bind 172.18.236.100:9292
  bind 172.18.251.100:9292
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:9292 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:9292 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:9292 check inter 2000 rise 2 fall 5

 listen glance_registry_cluster
  bind 172.18.236.100:9191
  bind 172.18.251.100:9191
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:9191 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:9191 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:9191 check inter 2000 rise 2 fall 5

# KEYSTONE

 listen keystone_cluster
  bind 172.18.236.100:5000
  bind 172.18.251.100:5000
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:5000 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:5000 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:5000 check inter 2000 rise 2 fall 5

# NOVA COMPUTE

 listen nova_compute_api_cluster
  bind 172.18.236.100:8774
  bind 172.18.251.100:8774
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8774 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8774 check inter 2000 rise 2 fall 5 backup
	server lab-r03-oscontroller-03 172.18.236.13:8774 check inter 2000 rise 2 fall 5 backup

# NOVA METADATA

 listen nova_metadata_api_cluster
  bind 172.18.236.100:8775
  bind 172.18.251.100:8775
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8775 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8775 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:8775 check inter 2000 rise 2 fall 5

# CINDER

 listen cinder_api_cluster
  bind 172.18.236.100:8776
  bind 172.18.251.100:8776
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8776 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8776 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:8776 check inter 2000 rise 2 fall 5

# PLACEMENT

 listen placement_internal_api_cluster
  bind 172.18.236.100:8778
  bind 172.18.251.100:8778
  balance  source
  option  tcpka
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8778 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8778 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:8778 check inter 2000 rise 2 fall 5

# NOVA VNC

 listen nova_vncproxy_cluster
  bind 172.18.251.100:6080
  #ssl crt /etc/ssl/certs/cert.dc.lab.co.id.pem
  balance  source
  option  tcpka
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:6080 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:6080 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:6080 check inter 2000 rise 2 fall 5

# NEUTRON

 listen neutron_api_cluster
  bind 172.18.236.100:9696
  bind 172.18.251.100:9696
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:9696 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:9696 check inter 2000 rise 2 fall 5 backup
	server lab-r03-oscontroller-03 172.18.236.13:9696 check inter 2000 rise 2 fall 5 backup

# HEAT

listen heat_api
  bind 172.18.236.100:8004
  bind 172.18.251.100:8004
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8004 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8004 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:8004 check inter 2000 rise 2 fall 5

listen heat_cfn
  bind 172.18.236.100:8000 transparent
  bind 172.18.251.100:8000 transparent
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:8000 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:8000 check inter 2000 rise 2 fall 5
	server lab-r03-oscontroller-03 172.18.236.13:8000 check inter 2000 rise 2 fall 5

# OCTAVIA

listen octavia_api_cluster
  bind 172.18.236.100:9876
  bind 172.18.251.100:9876
  balance  source
  option  tcpka
  option  httpchk
  option  tcplog
	server lab-r01-oscontroller-01 172.18.236.11:9876 check inter 2000 rise 2 fall 5
	server lab-r02-oscontroller-02 172.18.236.12:9876 check inter 2000 rise 2 fall 5 backup
	server lab-r03-oscontroller-03 172.18.236.13:9876 check inter 2000 rise 2 fall 5 backup

# HAPROXY Status Page

listen stats
  bind *:1945
  mode http
  stats enable
  stats hide-version
  stats uri /stats
  stats refresh 10s
  stats show-node
```

- Restart Haproxy with PCS. Execute on of controller


```bash
pcs resource restart lb-haproxy-clone
```
- check VIP active at which controller, then delete the resource. Execute on of controller


```bash
## check VIP active at which controller , look at MASTER. then marked or take a note. dc itu controller3.
pcs status

## Hapus Resoure
pcs resource delete ovndb_servers-master
```

- Stop service ovn-central on all controller and make sure the status actived.


```bash
systemctl stop ovn-central

systemctl status ovn-central
```

- Edit configuration default ovn-central on each Controller
1. Controller 1 (Master)


```bash
# nano /etc/default/ovn-central
. . .
OVN_CTL_OPTS=" \
--db-nb-addr=172.18.236.11 \
--db-nb-cluster-local-addr=172.18.236.11 \
--db-nb-create-insecure-remote=yes \
--ovn-northd-nb-db=tcp:172.18.236.11:6641,tcp:172.18.236.12:6641,tcp:172.18.236.13:6641 \
--db-sb-addr=172.18.236.11 \
--db-sb-cluster-local-addr=172.18.236.11 \
--db-sb-create-insecure-remote=yes \
--ovn-northd-sb-db=tcp:172.18.236.11:6642,tcp:172.18.236.12:6642,tcp:172.18.236.13:6642"
. . .
```

2. Controller 2 (Slave)


```bash
# nano /etc/default/ovn-central
. . .
OVN_CTL_OPTS=" \
--db-nb-addr=172.18.236.12 \
--db-nb-cluster-local-addr=172.18.236.12 \
--db-nb-cluster-remote-addr=172.18.236.11 \
--db-nb-create-insecure-remote=yes \
--ovn-northd-nb-db=tcp:172.18.236.11:6641,tcp:172.18.236.12:6641,tcp:172.18.236.13:6641 \
--db-sb-addr=172.18.236.12 \
--db-sb-cluster-local-addr=172.18.236.12 \
--db-sb-cluster-remote-addr=172.18.236.11 \
--db-sb-create-insecure-remote=yes \
--ovn-northd-sb-db=tcp:172.18.236.11:6642,tcp:172.18.236.12:6642,tcp:172.18.236.13:6642"
. . .
```

3. Controller 3 (Slave)


```bash
# nano /etc/default/ovn-central
. . .
OVN_CTL_OPTS=" \
--db-nb-addr=172.18.236.13 \
--db-nb-cluster-local-addr=172.18.236.13 \
--db-nb-cluster-remote-addr=172.18.236.11 \
--db-nb-create-insecure-remote=yes \
--ovn-northd-nb-db=tcp:172.18.236.11:6641,tcp:172.18.236.12:6641,tcp:172.18.236.13:6641 \
--db-sb-addr=172.18.236.13 \
--db-sb-cluster-local-addr=172.18.236.13 \
--db-sb-cluster-remote-addr=172.18.236.11 \
--db-sb-create-insecure-remote=yes \
--ovn-northd-sb-db=tcp:172.18.236.11:6642,tcp:172.18.236.12:6642,tcp:172.18.236.13:6642"
. . .
```

> Notes :
> 1. Any differen on configuration each master and slave which is at **db-sb-cluster-remote-addr** and **db-nb-cluster-remote-addr** filled IP Controller which become master OVN on resource **ovndb_servers-master** previously.
> 2. Master must start early (Step 7).

- Backup and delete file DB OVN existing. on all controller

```bash
cd /var/lib/ovn/

## Backup file DB existing
cp ovnnb_db.db ovnnb_db.db.bak
cp ovnsb_db.db ovnsb_db.db.bak

## Delete file DB existing
rm .ovnnb_db.db.~lock .ovnsb_db.db.~lock ovnnb_db.db ovnsb_db.db
```

- Start service ovn-central alternately start from master.
systemctl start ovn-central.service

systemctl status ovn-central
systemctl status ovn-northd
systemctl status ovn-sb-ovsdb
systemctl status ovn-nb-ovsdb

###### 6. Deploy octavia

```bash
ansible-playbook deploy.yml -t octavia
```
>Note: if had a error when deploy octavia dashboard you can do step below

- check file apt cache for packet python3-octavia-dashboard

```bash
ls -l /var/lib/dpkg/info | grep python3-octavia-dashboard
```

- move the package python3-octavia-dashboard to folder `/tmp`

```bash
sudo mv /var/lib/dpkg/info/python3-octavia-dashboard.* /tmp/ 
```

- delete packet python3-octavia-dashboard

```bash
sudo dpkg --remove --force-remove-reinstreq python3-octavia-dashboard 
```

- Install package python3-octavia-dashboard

```bash
apt install python3-octavia-dashboard 
```

- Restart service apache2

```bash
sudo systemctl restart apache2 
```

###### 7. Deploy Horizon

```bash
ansible-playbook deploy.yml -t horizon
```

###### 8. Create admin-openrc

```bash
# vim admin-openrc
. . .
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=rahasia
export OS_AUTH_URL=http://internal.dc.lab.co.id:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
. . .
```

###### 9. Create octavia-openrc

```bash
# vim octavia-openrc
. . .
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=service
export OS_USERNAME=octavia
export OS_PASSWORD=octavia!lab
export OS_AUTH_URL=http://internal.dc.lab.co.id:5000
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
export OS_VOLUME_API_VERSION=3
. . .
```

###### 10. Build Amphora image for Octavia
- Create new task on ansible role octavia (`/root/ansible-iac-ceph-openstack-main/OS-Ansible/roles/octavia/tasks/ubuntu-image.yml`) contents as below ,

```bash
- name: clone octavia repository
  git:
    repo: https://github.com/openstack/octavia
    dest: /root/octavia
    version: stable/ussuri
  when: inventory_hostname == groups['controller'][0]
- name: Initiate virtualenv and install disk-builder amphora pkg
  pip:
    virtualenv: /root/octavia-disk-builder
    virtualenv_python: python3.6
    requirements: /root/octavia/diskimage-create/requirements.txt
  when: inventory_hostname == groups['controller'][0]
- name: Install disk-builder amphora depends pkg
  apt:
    name: "{{ octavia.disk.packages }}"
    state: latest
  when: inventory_hostname == groups['controller'][0]
- name: Create ubuntu amphora image
  shell: source /root/octavia-disk-builder/bin/activate && bash /root/octavia/diskimage-create/diskimage-create.sh -i ubuntu-minimal -s 3 -r {{ amphora_root_password }}
  args:
    executable: /bin/bash
    chdir: /root/octavia/diskimage-create
  when: inventory_hostname == groups['controller'][0]
- name: Create amphora image to openstack
  os_image:
    cloud: octavia
    state: present
    name: amphora
    container_format: bare
    disk_format: qcow2
    tags: amphora
    filename: /root/octavia/diskimage-create/amphora-x64-haproxy.qcow2
    is_public: no
  when: inventory_hostname == groups['controller'][0]
```

- then edit the file task main.yml on role octavia become as below: 

```bash
---
#- import_tasks: ubuntu-octavia.yml
- import_tasks: ubuntu-image.yml
```

- Run ansible playbook at file `/root/ansible-iac-ceph-openstack-main/OS-Ansible/octavia.yml`

```bash
ansible-playbook octavia.yml
```