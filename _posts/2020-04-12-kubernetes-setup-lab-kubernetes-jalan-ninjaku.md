---
title: "[Kubernetes The Hard Way] Setup Lab Kubernetes Jalan Ninjaku"
date: 2020-04-12
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Kubernetes Jalan Ninjaku (Kubernetes The Hardway)

Okrayy pada tutorial kali ini kita akan mengkonfigurasi atau menjalankan kubernetes namun dengan cara the hardway, jadi kita membangun sebuah sistem kubernetes tanpa bantuan automasisasi atau library yang bisa langsung tuiing jdii tuh kebernetes cluster nya semisal minikube dan kubeadm (ya walaupun ini perlu di konfigurasi lgi sih hehehee)..

Tujuan Kubernetes The Hard Way ini untuk membantu kita untuk memahami bagaimana services, ecosystem dan cara kerja kubernetes secara menyeluruh jadi mantul buat yang merasa kubernetes ini adalah jalan ninjaku :D.

> The results of this tutorial should not be viewed as production ready, and may receive limited support from the community, but don't let that stop you from learning! -Kelsey Higtower-

* Sumber utama : [kubernetes-the-hard-way](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)

alat dan bahan ambyarr: 

* [Kubernetes v1.12.0](https://github.com/kubernetes/kubernetes)
* [Containerd Container Runtime v1.2.0-rc.0](https://github.com/containerd/containerd)
* [gVisor sandbox runtime ](https://github.com/google/gvisor)
* [CNI Container Networking v0.6.0](https://github.com/containernetworking/cni)
* [etcd v3.3.9](https://github.com/etcd-io/etcd)
* [CoreDNS v1.2.2](https://github.com/coredns/coredns)
* [LXC Container v3.0.3](https://linuxcontainers.org)

* Dan untuk membangun tiap node nya saya menggunakan [LXC Container](https://ammarun.my.id/ngoprek/server/cloud/lxc/container/lxd-containers-hypervisor/) yang sebelumnya sudah pernah saya tulis.

### Membuat tiap node k8s
Pertama kita membuat profile lxc untuk tiap node container k8s,

```s
lxc profile create k8s
lxc profile edit k8s

###
config:
  limits.cpu: "2"
  limits.memory: 2GB
  limits.memory.swap: "false"
  linux.kernel_modules: ip_tables,ip6_tables,netlink_diag,nf_nat,overlay
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=proc:rw
    sys:rw"
  security.nesting: "true"
  security.privileged: "true"
description: LXD profile for Kubernetes
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: k8s
used_by: []
###
```

Kedua kita membuat tiap node container k8s,
```s
lxc launch images:centos/7 haproxy 
lxc launch ubuntu:18.04 controller-0 --profile k8s
lxc launch ubuntu:18.04 controller-1 --profile k8s
lxc launch ubuntu:18.04 controller-2 --profile k8s

lxc launch ubuntu:18.04 worker-0 --profile k8s
lxc launch ubuntu:18.04 worker-1 --profile k8s
lxc launch ubuntu:18.04 worker-2 --profile k8s
```

cek node dengan `lxc list`,
```s
lxc list

###
+--------------+---------+---------------------+------+------------+-----------+
|     NAME     |  STATE  |        IPV4         | IPV6 |    TYPE    | SNAPSHOTS |
+--------------+---------+---------------------+------+------------+-----------+
| controller-0 | RUNNING | 10.240.0.88 (eth0)  |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| controller-1 | RUNNING | 10.240.0.161 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| controller-2 | RUNNING | 10.240.0.39 (eth0)  |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| haproxy      | RUNNING | 10.240.0.62 (eth0)  |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-0     | RUNNING | 10.240.0.79 (eth0)  |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-1     | RUNNING | 10.240.0.245 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-2     | RUNNING | 10.240.0.175 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
###
```

> Disini saya membuat sebanyak 7 node Container haproxy(Centos7) akan berperan sebagai loadbalancer ke setiap node controller(Ubuntu:18.04) dan worker(Ubuntu:18.04). 

Pertama kita setup Loadbalancer HAproxy nya duls :

Masuk ke container haproxy dan install paket haproxy
```s
root@palo:/home/ngoprek# lxc exec haproxy bash 
[root@haproxy ~]# yum install -y haproxy net-tools nano
```

Edit file konfigurasi haproxy, dan hapus bagian frontend sampai bawah lalu tambahkan,
```s
nano /etc/haproxy/haproxy.cfg 

###
frontend k8s
  bind 10.240.0.62:6443
  mode tcp
  default_backend k8s

backend k8s
  balance roundrobin
  mode tcp
  option tcplog
  option tcp-check
  server controller-0 10.240.0.88:6443 check
  server controller-1 10.240.0.161:6443 check
  server controller-2 10.240.0.39:6443 check
###
```

Up service haproxy dan check,
```s
systemctl enable haproxy
systemctl start haproxy
systemctl status haproxy
netstat -nltp

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 10.240.0.62:6443       0.0.0.0:*               LISTEN      586/haproxy         
```
oke sudah selesai di setup haproxy nya lanjut ke node controller dan node worker ya lainnya,

Lab ini akan saya bagi beberapa part supaya tidak numpuk karena banyak service2 yang dibutuhkan uhuy :

* [Setup Lab Kubernetes Jalan Ninjaku](#)
* [Installing the Client Tools - Part 1](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Installing-the-client-Tools-part1/)
* [Provisioning the CA and Generating TLS Certificates - Part 2](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Provisioning-a-CA-and-Generating-TLS-Certificates-part2/)
* [Generating Kubernetes Configuration Files for Authentication - Part 3](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Generating-Kubernetes-Configuration-Files-for-Authentication-Part3/)
* [Generating the Data Encryption Config and Key - Part 4](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Generating-the-Data-Encryption-Config-and-Key-Part4/)
* [Bootstrapping the etcd Cluster - Part 5](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Bootstrapping-the-etcd-Cluster-Part5/)
* [Bootstrapping the Kubernetes Control Plane - Part 6](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Bootstrapping-the-Kubernetes-Control-Plane-Part6/)
* [Bootstrapping the Kubernetes Worker Nodes - Part 7](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Bootstrapping-the-Kubernetes-Worker-Nodes-Part7/)
- [Provisioning Pod Network Routes - Part 8](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-part8/)
- [Deploying the DNS Cluster Add-on - Part 9](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-part9/)
- [Smoke Test](docs/13-smoke-test.md)
- [Cleaning Up](docs/14-cleanup.md)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
