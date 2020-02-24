---

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
## Installation Instructions
### Arch Linux
###### From Arch Repository
```
sudo pacman -S docker
```
### CentOS 7
###### From Docker Repository
```
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce
```
###### Using Convenience Script from Docker
```
curl -fsSL https://get.docker.com | sudo sh -
```
### Ubuntu 18.04
###### From Docker Repository
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce
```
Docker service enabled and running by default
###### Using Convenienct Script from Docker
```
curl -fsSL https://get.docker.com | sudo sh -
```
## Starting and Enabling docker service
On Ubuntu 18.04 the service is enabled and running after installation.
```
sudo systemctl enable docker
sudo systemctl start docker
```
## Running docker commands as non-root user
```
getent group docker
sudo gpasswd -a <user> docker
getent group docker
```
Log out and log back in
## Remove Docker
### Arch Linux
```
sudo pacman -Rns docker
```
### CentOS 7
```
sudo yum remove -y docker-ce
sudo rm -f /etc/yum.repos.d/docker-ce.repo
```
### Ubuntu 18.04
```
sudo apt-get purge -y docker-ce
sudo apt-get autoremove
```
###### Remove group and direcory
```
sudo groupdel docker
sudo rm -rf /var/lib/docker*
```


#### 2. Uji instalasi docker
```
sudo docker version
```

![docker-version](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker_version.png)

```shell
sudo docker run hello-world
sudo docker image ls
sudo docker container ls -a
```

> Menampilkan semua container (Active  ataupun exit)
```shell
sudo docker container ls -a
```

![Docker-Run](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker-run.png)


# Enjoy Happy,  Ngoprek ~







