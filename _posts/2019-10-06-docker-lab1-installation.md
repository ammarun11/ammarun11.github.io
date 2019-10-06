---
layout : post
title : "[Docker] Installation Lab Docker (Part 1)"
categories: [ngoprek, server, cloud, docker, container]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

Persiapkan Lab Node yang akan kita gunakan disini saya menggunakan private cloud yaitu Openstack, dengan 2 Node :

**pod-pod0**
* Interface: eth0
* IP Address: 10.1.40.12/24
* Gateway: 10.1.40.1
* DNS Resolver: 10.1.40.1

**pod-pod1**
* Interface: eth0
* IP Address: 10.1.40.18/24
* Gateway: 10.1.40.1
* DNS Resolver: 10.1.40.1

-----
## Install Docker #####
#######################

#### Eksekusi di node `pod-pod0` ###

#### 1. Instal Docker & Menampilkan versi docker
```BASH
sudo apt update
sudo apt -y install docker.io
sudo systemctl status docker
sudo docker version
```
![docker-version]()

#### 2. Uji instalasi docker

```BASH
sudo docker run hello-world
sudo docker image ls
sudo docker container ls -a
```

> Menampilkan semua container (Active  ataupun exit)
```BASH
sudo docker container ls -a
```

![Docker-Run]()


# Enjoy Happy Ngoprek ~







