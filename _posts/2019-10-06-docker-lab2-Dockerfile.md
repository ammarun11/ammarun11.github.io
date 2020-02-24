---

title : "[Docker] Dockerfile Lab Docker (Part 2)"
categories: [ngoprek, server, cloud, docker, container]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

Lab selanjutnya kita akan mencoba untuk menjalankan Docker container dengan menggunakan image yang ada di Docker Registry yaitu Docker hub.

#### Eksekusi di node `pod-pod0` ###

#### 1. Buka Docker Hub dan cari image whalesay

[https://hub.docker.com](https://hub.docker.com).

#### 2. Jalankan image whalesay
```shell
sudo docker run docker/whalesay cowsay palopalepalo
```
![docker-whalesay](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker-whalesay.png)

---

#### Docker container dengan `Dockerfile` ###

#### 1. Buka Docker Hub dan cari

```shell 
sudo mkdir -p /latihan/latihan01
sudo chown -R ubuntu:ubuntu /latihan
cd /latihan/latihan01
```

#### 2. Buat file Dockerfile
```shell
vim Dockerfile

# Use whalesay image as a base image
FROM docker/whalesay:latest

# Install fortunes
RUN apt -y update && apt install -y fortunes

# Execute command
CMD /usr/games/fortune -a | cowsay
```

#### 3. Bangun image dari Dockerfile lalu jalankan
```shell
sudo docker build -t docker-whale
sudo docker image ls
sudo docker run docker-whale
```

#### 4. Menghapus Image & Container 
```shell
sudo docker image rm [IMAGE_ID]
sudo docker rm [CONTAINER_ID]
```

# Enjoy Happy,  Ngoprek ~

