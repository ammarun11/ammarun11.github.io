---
title: "[Kubernetes The Hard Way] setup lab Kubernetes jalan ninjaku"
date: 2020-02-19
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Kubernetes Jalan Ninjaku (Kubernetes The Hardway)

Okrayy pada tutorial kali ini kita akan mengkonfigurasi atau menjalankan kubernetes namun dengan cara the hardway, jadi kita membangun sebuah sistem kubernetes tanpa bantuan automasisasi atau library yang bisa langsung tuiing jdii tuh kebernetes cluster nya semisal minikuber dan kubeadm..

Tujuan Kubernetes The Hard Way ini untuk membantu kita untuk memahami bagaimana services, ecosystem dan cara kerja kubernetes secara menyeluruh.

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

```s
root@palo:/home# lxc version
Client version: 3.0.3
Server version: 3.0.3
root@palo:/home# lxc list
+--------------+---------+---------------------+------+------------+-----------+
|     NAME     |  STATE  |        IPV4         | IPV6 |    TYPE    | SNAPSHOTS |
+--------------+---------+---------------------+------+------------+-----------+
| controller-0 | RUNNING | 10.240.0.208 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| controller-1 | RUNNING | 10.240.0.215 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| controller-2 | RUNNING | 10.240.0.232 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| haproxy      | RUNNING | 10.240.0.254 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-0     | RUNNING | 10.240.0.244 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-1     | RUNNING | 10.240.0.248 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
| worker-2     | RUNNING | 10.240.0.210 (eth0) |      | PERSISTENT | 0         |
+--------------+---------+---------------------+------+------------+-----------+
```

> Disini saya membuat sebanuak 7 node Container haproxy(Centos7) akan berperan sebagai loadbalancer ke setiap node controller(Ubuntu:18.04) dan worker(Ubuntu:18.04). 

Lab ini akan saya bagi beberapa part supaya tidak numpuk karena banyak service2 yang dibutuhkan uhuy :

* [Installing the Client Tools - Part 1](docs/02-client-tools.md)

<!-- * [Provisioning Compute Resources](docs/03-compute-resources.md)
* [Provisioning the CA and Generating TLS Certificates](docs/04-certificate-authority.md)
* [Generating Kubernetes Configuration Files for Authentication](docs/05-kubernetes-configuration-files.md)
* [Generating the Data Encryption Config and Key](docs/06-data-encryption-keys.md)
* [Bootstrapping the etcd Cluster](docs/07-bootstrapping-etcd.md)
* [Bootstrapping the Kubernetes Control Plane](docs/08-bootstrapping-kubernetes-controllers.md)
* [Bootstrapping the Kubernetes Worker Nodes](docs/09-bootstrapping-kubernetes-workers.md)
* [Configuring kubectl for Remote Access](docs/10-configuring-kubectl.md)
* [Provisioning Pod Network Routes](docs/11-pod-network-routes.md)
* [Deploying the DNS Cluster Add-on](docs/12-dns-addon.md)
* [Smoke Test](docs/13-smoke-test.md)
* [Cleaning Up](docs/14-cleanup.md) -->


**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~