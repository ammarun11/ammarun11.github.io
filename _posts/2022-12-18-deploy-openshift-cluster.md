---
title: "[Openshift] Deploy Openshift 4.7"
date: 2022-12-18
categories: [ngoprek, server, cloud, kubernetes, openshift]
tags:
  - Jekyll
  - update
---
## Guide
##### Topology

![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1-openshift-topology.png)

##### Daftar vms:
* 10.60.60.161 lab-bastion DNS Server, DHCP Server, NTP Server, PXE Server, LoadBalancer
* 10.60.60.162 lab-bootstrap VM kosong tanpa OS
* 10.60.60.163 lab-master1 VM kosong tanpa OS
* 10.60.60.164 lab-master2 VM kosong tanpa OS
* 10.60.60.165 lab-master3 VM kosong tanpa OS
* 10.60.60.166 lab-worker1 VM kosong tanpa OS
* 10.60.60.167 lab-worker2 VM kosong tanpa OS

#### Deploy cluster openshift 4.7


1\. SSH to `lab-bastion` host and install bind with related packages

```bash
ssh root@labX.lab.id -pXXXXX
dnf update -y && dnf install epel-release -y
dnf install -y vim bind bind-utils wget screen htop git
```

2\. Define zone in the named.conf file 

```bash
mv /etc/named.conf /etc/named.conf.bak
vim /etc/named.conf
...
options {
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	secroots-file	"/var/named/data/named.secroots";
	recursing-file	"/var/named/data/named.recursing";
	allow-query     { localhost;10.60.60.0/24; };
  listen-on port 53 { any; };

	recursion yes;
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
	dnssec-enable yes;
	dnssec-validation yes;

	managed-keys-directory "/var/named/dynamic";

	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";

	include "/etc/crypto-policies/back-ends/bind.config";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
	type hint;
	file "named.ca";
};

zone "openshift.pod0.io" {
        type master;
        file "dynamic/forward.db";
};

zone "60.60.10.in-addr.arpa" {
        type master;
        file "dynamic/reverse.db";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
...
```

3\. And then create a zone file named forward.db

```bash
vim /var/named/dynamic/forward.db
...
$TTL 1W
@       IN      SOA     ns1.openshift.pod0.io. root (
                        2020092301      ; serial
                        3H                  ; refresh (3 hours)
                        30M                 ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns1.openshift.pod0.io.
        IN      MX 10   smtp.openshift.pod0.io.
;
; 
ns1     IN      A       10.60.60.161
smtp    IN      A       10.60.60.161
;
helper  IN      A       10.60.60.161
helper  IN      A       10.60.60.161
;
; The api points to the IP of your load balancer
api             IN      A       10.60.60.161
api-int         IN      A       10.60.60.161
;
; The wildcard also points to the load balancer
*.apps          IN      A       10.60.60.161
;
; Create entry for the bootstrap host
bootstrap       IN      A       10.60.60.162
;
; Create entries for the master hosts
master1         IN      A       10.60.60.163
master2         IN      A       10.60.60.164
master3         IN      A       10.60.60.165
;
; Create entries for the worker hosts
worker1         IN      A       10.60.60.166
worker2         IN      A       10.60.60.167
;
; The ETCd cluster lives on the masters...so point these to the IP of the masters
etcd1  IN      A       10.60.60.163
etcd2  IN      A       10.60.60.164
etcd3  IN      A       10.60.60.165
;
; The SRV records are IMPORTANT....make sure you get these right...note the trailing dot at the end...
_etcd-server-ssl._tcp   IN      SRV     0 10 2380 etcd1.openshift.pod0.io.
_etcd-server-ssl._tcp   IN      SRV     0 10 2380 etcd2.openshift.pod0.io.
_etcd-server-ssl._tcp   IN      SRV     0 10 2380 etcd3.openshift.pod0.io.
;
...
```

4\. Create reverse zone file named reverse.db 

```bash
vim /var/named/dynamic/reverse.db
...
$TTL 1W
@       IN      SOA     ns1.openshift.pod0.io. root (
                        2020092301      ; serial
                        3H                  ; refresh (3 hours)
                        30M                 ; retry (30 minutes)
                        2W                  ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns1.openshift.pod0.io.
;
; syntax is "last octet" and the host must have fqdn with trailing dot
163       IN      PTR     master1.openshift.pod0.io.
164       IN      PTR     master2.openshift.pod0.io.
165       IN      PTR     master3.openshift.pod0.io.
;
162       IN      PTR     bootstrap.openshift.pod0.io.
;
161       IN      PTR     api.openshift.pod0.io.
161       IN      PTR     api-int.openshift.pod0.io.
;
166       IN      PTR     worker1.openshift.pod0.io.
167       IN      PTR     worker2.openshift.pod0.io.
;
...
```

5\. Enable and start bind service

```bash
systemctl enable named
systemctl restart named
systemctl status named
```

6\. Change DNS Server on Bastion

```bash
nmtui
```


```bash
vim /etc/resolv.conf
...
#nameserver 8.8.8.8
nameserver 10.60.60.161
...
```

7\. Verify DNS Server works

```bash
nslookup api.openshift.pod0.io
ping yahoo.com -c 3
```

##### Install OpenShift Cluster - Setup DHCP Server

1\. Install dhcp-server package on bastion host

```bash
dnf install dhcp-server -y
```

2\. Configure DHCP Leases

```bash
mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak
vim /etc/dhcp/dhcpd.conf
...
ddns-update-style interim;
ignore client-updates;
authoritative;
allow booting;
allow bootp;
allow unknown-clients;

# internal subnet for my DHCP Server
subnet 10.60.60.0 netmask 255.255.255.0 {
range 10.60.60.200 10.60.60.210;
option domain-name-servers 10.60.60.161;
option routers 10.60.60.1;
option broadcast-address 10.60.60.255;
default-lease-time 600;
max-lease-time 7200;

host bootstrap.openshift.pod0.io { hardware ethernet 52:54:00:e7:03:e9; fixed-address 10.60.60.162; }
host master1.openshift.pod0.io { hardware ethernet 52:54:00:5c:d1:42; fixed-address 10.60.60.163; }
host master2.openshift.pod0.io { hardware ethernet 52:54:00:c1:66:b2; fixed-address 10.60.60.164; }
host master3.openshift.pod0.io { hardware ethernet 52:54:00:e9:62:73; fixed-address 10.60.60.165; }
host worker1.openshift.pod0.io { hardware ethernet 52:54:00:25:c7:aa; fixed-address 10.60.60.166; }
host worker2.openshift.pod0.io { hardware ethernet 52:54:00:f8:d0:ea; fixed-address 10.60.60.167; }

deny unknown-clients;

# IP of PXE Server
next-server 10.60.60.161;
if exists user-class and option user-class = "iPXE" {

filename "http://helper.openshift.pod0.io:8080/boot.ipxe";

} else {

filename "undionly.kpxe";

}
}
...
```

> **NOTE: **  Change the MAC Adresses with your own.

3\. Restart and enable dhcp service

```bash
systemctl enable dhcpd
systemctl restart dhcpd
systemctl status dhcpd
```

##### Install OpenShift Cluster - Install TFTP Server

1\. Install TFTP Server packages

```bash
dnf install -y tftp-server ipxe-bootimgs
```

2\. Create the standard tree for the TFTP server and link the image

```bash
mkdir -p /var/lib/tftpboot
ln -s /usr/share/ipxe/undionly.kpxe /var/lib/tftpboot
```

3\. Start and enable TFTP Service

```bash
systemctl enable tftp
systemctl start tftp
systemctl status tftp
```

##### Install OpenShift Cluster - Setup Matchbox

1\. Download and extract Matchbox

```bash
curl -LO https://github.com/poseidon/matchbox/releases/download/v0.8.3/matchbox-v0.8.3-linux-amd64.tar.gz
tar xvzf matchbox-v0.8.3-linux-amd64.tar.gz 
```

2\. Move matchbox binary

```bash
cd matchbox-v0.8.3-linux-amd64
cp matchbox /usr/local/bin/
```

3\. Create matchbox user for matchbox service

```bash
useradd -U matchbox
cat /etc/passwd | grep matchbox
```

4\. Create configuration directory for matchbox

```bash
mkdir -p /var/lib/matchbox/{assets,groups,ignition,profiles}
chown -R matchbox:matchbox /var/lib/matchbox
ls /var/lib/matchbox/
```

5\. Create, enable and start matchbox systemd service

```bash
cp contrib/systemd/matchbox-local.service /etc/systemd/system/matchbox.service

systemctl daemon-reload
systemctl enable matchbox
systemctl start matchbox
systemctl status matchbox
```

6\. Download rhcos initramfs, kernel, and metal

```bash
cd /var/lib/matchbox/assets

wget -c https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.7/4.7.7/rhcos-4.7.7-x86_64-live-initramfs.x86_64.img
wget -c https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.7/4.7.7/rhcos-4.7.7-x86_64-live-kernel-x86_64
wget -c https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.7/4.7.7/rhcos-4.7.7-x86_64-metal.x86_64.raw.gz
wget -c https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.7/4.7.7/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img
```

7\. Create Matchbox Bootstrap profiles

```bash
vim /var/lib/matchbox/profiles/bootstrap.json
...
{
"id": "bootstrap",
"name": "OCP 4.7 - Bootstrap",
"ignition_id": "bootstrap.ign",
"boot": {
"kernel": "/assets/rhcos-4.7.7-x86_64-live-kernel-x86_64",
"initrd": ["/assets/rhcos-4.7.7-x86_64-live-initramfs.x86_64.img"],
"args": [
"rd.neednet=1",
"console=tty0",
"console=ttyS0",
"coreos.inst=yes",
"coreos.inst.insecure=yes",
"coreos.inst.install_dev=vda",
"coreos.inst.image_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-metal.x86_64.raw.gz",
"coreos.inst.ignition_url=http://10.60.60.161:8080/ignition?mac=${mac:hexhyp}",
"coreos.live.rootfs_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img"
]
}
}
...
```

8\. Create Matchbox Master profiles

```bash
vim /var/lib/matchbox/profiles/master.json
...
{
"id": "master",
"name": "OCP 4.7 - Master",
"ignition_id": "master.ign",
"boot": {
"kernel": "/assets/rhcos-4.7.7-x86_64-live-kernel-x86_64",
"initrd": ["/assets/rhcos-4.7.7-x86_64-live-initramfs.x86_64.img"],
"args": [
"rd.neednet=1",
"console=tty0",
"console=ttyS0",
"coreos.inst=yes",
"coreos.inst.insecure=yes",
"coreos.inst.install_dev=vda",
"coreos.inst.image_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-metal.x86_64.raw.gz",
"coreos.inst.ignition_url=http://10.60.60.161:8080/ignition?mac=${mac:hexhyp}",
"coreos.live.rootfs_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img"
]
}
}
...
```

9\. Create Matchbox Worker profiles

```bash
vim /var/lib/matchbox/profiles/worker.json
...
{
"id": "worker",
"name": "OCP 4.7 - Worker",
"ignition_id": "worker.ign",
"boot": {
"kernel": "/assets/rhcos-4.7.7-x86_64-live-kernel-x86_64",
"initrd": ["/assets/rhcos-4.7.7-x86_64-live-initramfs.x86_64.img"],
"args": [
"rd.neednet=1",
"console=tty0",
"console=ttyS0",
"coreos.inst=yes",
"coreos.inst.insecure=yes",
"coreos.inst.install_dev=vda",
"coreos.inst.image_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-metal.x86_64.raw.gz",
"coreos.inst.ignition_url=http://10.60.60.161:8080/ignition?mac=${mac:hexhyp}",
"coreos.live.rootfs_url=http://10.60.60.161:8080/assets/rhcos-4.7.7-x86_64-live-rootfs.x86_64.img"
]
}
}
...
```

10\. Create groups for bootstrap node

```bash
cat <<EOF >> /var/lib/matchbox/groups/bootstrap.json
{
"id": "bootstrap",
"name": "OCP 4.7 - Bootstrap server",
"profile": "bootstrap",
"selector": {
"mac": "52:54:00:e7:03:e9"
}
}
EOF
```

11\. Create groups for master1 node

```bash
cat <<EOF >> /var/lib/matchbox/groups/master1.json 
{
"id": "master1",
"name": "OCP 4.7 - Master 1",
"profile": "master",
"selector": {
"mac": "52:54:00:5c:d1:42"
}
}
EOF
```

12\. Create groups for master2 node

```bash
cat <<EOF >> /var/lib/matchbox/groups/master2.json 
{
"id": "master2",
"name": "OCP 4.7 - Master 2",
"profile": "master",
"selector": {
"mac": "52:54:00:c1:66:b2"
}
}
EOF
```

13\. Create groups for master3 node

```bash
cat <<EOF >> /var/lib/matchbox/groups/master3.json 
{
"id": "master3",
"name": "OCP 4.7 - Master 3",
"profile": "master",
"selector": {
"mac": "52:54:00:e9:62:73"
}
}
EOF
```

14\. Create groups for worker1 node

```bash
cat <<EOF >> /var/lib/matchbox/groups/worker1.json 
{
"id": "worker1",
"name": "OCP 4.7 - Worker 1",
"profile": "worker",
"selector": {
"mac": "52:54:00:25:c7:aa"
}
}
EOF
```

15\. Create groups for worker2 node

```bash
cat <<EOF >> /var/lib/matchbox/groups/worker2.json 
{
"id": "worker2",
"name": "OCP 4.7 - Worker 2",
"profile": "worker",
"selector": {
"mac": "52:54:00:f8:d0:ea"
}
}
EOF
```

##### Install OpenShift Cluster - Configure NTP Server

1\. Install chrony package

```bash
dnf install -y chrony
```

2\.  Configure chrony

```bash
vim /etc/chrony.conf
...
#comment below line
#pool 2.centos.pool.ntp.org iburst

#add below lines
server 0.id.pool.ntp.org
server 1.id.pool.ntp.org
server 2.id.pool.ntp.org
server 3.id.pool.ntp.org

#Modify below line
# Allow NTP client access from local network.
allow 10.60.60.0/24
...
```

3\.  Enable and restart service

```bash
systemctl enable chronyd
systemctl restart chronyd
systemctl status chronyd
```

4\.  Verify NTP Server

```bash
chronyc sources
```

5\.  Set timezone to Asia/Jakarta

```bash
timedatectl set-timezone Asia/Jakarta
```


##### Install OpenShift Cluster - Configure HAProxy and Rsyslog

1\. Install haproxy packages

```bash
dnf install -y haproxy rsyslog
```

2\. Configure HAProxy

```bash
mv /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.bak
vim /etc/haproxy/haproxy.cfg
...
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

    # utilize system-wide crypto-policies
    ssl-default-bind-ciphers PROFILE=SYSTEM
    ssl-default-server-ciphers PROFILE=SYSTEM

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend openshift-api-server
    bind api.openshift.pod0.io:6443
    default_backend openshift-api-server
    mode tcp
    option tcplog
frontend machine-config-server
    bind api-int.openshift.pod0.io:22623
    default_backend machine-config-server
    mode tcp
    option tcplog
frontend ingress-http
    bind *:80
    default_backend ingress-http
    mode tcp
    option tcplog
frontend ingress-https
    bind *:443
    default_backend ingress-https
    mode tcp
    option tcplog

#---------------------------------------------------------------------
# static backend for serving up API, MSC, HTTP and HTTPS
#---------------------------------------------------------------------
backend openshift-api-server
    balance source
    mode tcp
    server bootstrap.openshift.pod0.io 10.60.60.162:6443 check
    server master1.openshift.pod0.io 10.60.60.163:6443 check
    server master2.openshift.pod0.io 10.60.60.164:6443 check
    server master3.openshift.pod0.io 10.60.60.165:6443 check
backend machine-config-server
    balance source
    mode tcp
    server bootstrap.openshift.pod0.io 10.60.60.162:22623 check
    server master1.openshift.pod0.io 10.60.60.163:22623 check
    server master2.openshift.pod0.io 10.60.60.164:22623 check
    server master3.openshift.pod0.io 10.60.60.165:22623 check
backend ingress-http
    balance source
    mode tcp
    server worker1.openshift.pod0.io 10.60.60.166:80 check
    server worker2.openshift.pod0.io 10.60.60.167:80 check
backend ingress-https
    balance source
    mode tcp
    server worker1.openshift.pod0.io 10.60.60.166:443 check
    server worker2.openshift.pod0.io 10.60.60.167:443 check
...
```

3\. Enable haproxy log. Comment out below line

```bash
vim /etc/rsyslog.conf
...
module(load="imudp") # needs to be done just once
input(type="imudp" port="514")
...
```


```bash
vim /etc/rsyslog.d/haproxy.conf
...
#Add this line
local2.*    /var/log/haproxy.log
...
```

4\. Setsebool to allow haproxy socket to open on any port

```bash
setsebool -P haproxy_connect_any=1
```

5\. Allow haproxy to connect to unbind IP Address

```bash
vim /etc/sysctl.d/99-sysctl.conf
...
#Add this line
net.ipv4.ip_nonlocal_bind=1
...
```

6\. Enable and restart haproxy

```bash
systemctl enable haproxy
systemctl restart haproxy
systemctl status haproxy
```

7\. Enable and restart rsyslog

```bash
systemctl enable rsyslog
systemctl restart rsyslog
systemctl status rsyslog
```

##### Install OpenShift Cluster - Provisioning OpenShift Cluster

1\. Generate SSH Keypair

```bash
ssh-keygen -t rsa -b 2048 -N '' -f /root/.ssh/id_rsa
```

2\. Obtaining the installation program

```bash
# Download and install openshift client
curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz | tar -xzf - -C /usr/local/bin/ oc
ls /usr/local/bin/
```


```bash
# Download openshift-installer
curl -s https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz | tar -xzf - -C /usr/local/bin/ openshift-install
ls /usr/local/bin/
```


```bash
# Create a directory to save openshift configuration
rm -rf /root/ocp47 && mkdir /root/ocp47 && cd /root/ocp47
```

3\.  Pull secret from  [https://cloud.redhat.com/openshift/install/pull-secret](https://cloud.redhat.com/openshift/install/pull-secret)


4\. Create installation configuration file

```bash
key=`cat ~/.ssh/id_rsa.pub`
tee -a install-config.yaml<<-EOF
apiVersion: v1
baseDomain: pod0.io
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: openshift
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
pullSecret: 'PUT_THE_PULL_SECRET_HERE'
sshKey: $key
EOF
```

5\. Backup installation configuration file

```bash
cp /root/ocp47/install-config.yaml /root/ocp47/install-config.yaml.bak
ls
```

6\. Generate manifests files

```bash 
openshift-install create manifests --dir=/root/ocp47
```

7\. Prevent Pods from being scheduled on the control plane machines. Set `mastersSchedulable` to false.

```bash
vim /root/ocp47/manifests/cluster-scheduler-02-config.yml
...
apiVersion: config.openshift.io/v1
kind: Scheduler
metadata:
  creationTimestamp: null
  name: cluster
spec:
  mastersSchedulable: false
  policy:
    name: ""
status: {}
...
```

8\. Generate Ignition Files

```bash
openshift-install create ignition-configs --dir=/root/ocp47 
```

9\. Directory tree

```bash
tree
```

10\. Copy ignition files to matchbox directory

```bash
ls -la /root/ocp47/
rm -rf /var/lib/matchbox/ignition/*.ign
cp /root/ocp47/*.ign /var/lib/matchbox/ignition
```

11\. Set ownership of directory /var/lib/matchbox

```bash
chown -R matchbox:matchbox /var/lib/matchbox
chmod o+r /var/lib/matchbox/ignition/*.ign
```

12\. Power On bootstrap,master and worker nodes

13\. Check bootstrapping progress

```bash
openshift-install --dir=/root/ocp47 wait-for bootstrap-complete --log-level=info                     
```

14\. Remove bootstrap server from loadbalancer

```bash
vim /etc/haproxy/haproxy.cfg
...
#    server bootstrap.openshift.pod0.io 10.60.60.162:6443 check
#    server bootstrap.openshift.pod0.io 10.60.60.162:22623 check
...
systemctl restart haproxy
```

15\. After bootstrapping complete, check installation progress.

```bash
openshift-install --dir=/root/ocp47 wait-for install-complete --log-level=debug
```

16\. Login Cluster via virtual user

```bash
YOUR_PASSWORD=`cat ocp47/auth/kubeadmin-password`
oc login -u kubeadmin -p ${YOUR_PASSWORD} https://api.openshift.pod0.io:6443
```
18\. Approving CSR

```bash
oc get csr
oc get csr | grep Pending | awk '{print $1}' | xargs oc adm certificate approve

##### Make sure that all aproved
```

19\. Accessing the web console

```bash
#SSH Tunnel to labX.lab.id

# Execute in local devices
vim /etc/hosts
...
10.60.60.161 console-openshift-console.apps.openshift.pod0.io
...

# Open in browser
https://console-openshift-console.apps.openshift.pod0.io

user: kubeadmin
password: ${YOUR_PASSWORD}
```

## Verify
![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2-openshift-verify.png)