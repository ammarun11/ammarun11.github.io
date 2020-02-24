---

title : "[Docker] Volumes Lab Docker (Part 3)"
categories: [ngoprek, server, cloud, docker, container]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

Lab kali ini kita akan mencoba Volumes, seperti mount file static kedalam container yang sedang berjalan di node local maupun beda node yang berbeda.

#### Eksekusi di node `pod-pod0` ###

## Volumes 

#### 1. Membuat volume & Melihat Detail nya

```shell
root@pod0:~# sudo docker volume create my-vol
my-vol
root@pod0:~# docker volume ls
DRIVER              VOLUME NAME
local               my-vol
root@pod0:~# docker inspect my-vol
[
    {
        "CreatedAt": "2019-10-06T14:42:33Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
root@pod0:~# 
```
#### 2. Jalankan container dengan volume dan uji coba image nya via browser
```shell
root@pod0:~# sudo docker run -d --name=nginxtest -v my-vol:/usr/share/nginx/html nginx:latest
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
b8f262c62ec6: Pull complete 
e9218e8f93b1: Pull complete 
7acba7289aa3: Pull complete 
Digest: sha256:aeded0f2a861747f43a01cf1018cf9efe2bdd02afd57d2b11fcc7fcadc16ccd1
Status: Downloaded newer image for nginx:latest
811631bf89f96b821adc60f3bc23e5c493b47c7f49c018941cd9b8406f037118
```
```shell 
root@pod0:~# sudo docker inspect nginxtest | grep -i ipaddress
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```
```shell
root@pod0:~# curl http://172.17.0.2
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@pod0:~# 
```

---

#### 3. Buat file index.html dan pindahkan ke direktori source volume

```shell 
root@pod0:~# sudo echo "This is from my-vol source directory." > index.html
root@pod0:~# sudo mv index.html /var/lib/docker/volumes/my-vol/_data
```

```shell
root@pod0:~# curl http://172.17.0.2
This is from my-vol source directory.
```

#### 4. Jalankan container dengan read-only volume

```shell
root@pod0:~# sudo docker run -d --name=nginxtest-rovol -v my-vol:/usr/share/nginx/html:ro nginx:latest
58415d2ac2ced7f40fc2267ca5f15c5c0a4f8b41880e3df9017983b2540af52c
```

```shell
root@pod0:~# sudo docker inspect nginxtest-rovol | grep -i RW
                "RW": false
```
---

## Volume Driver
#### 5. SSH ke pod-pod1. Buat folder /share dan exit kembali

```shell
ssh -l ubuntu 10.1.40.18
sudo mkdir /share
sudo chmod 777 /share
exit
```
#### 6. Instal plugin sshfs 
```shell
sudo docker plugin install --grant-all-permissions vieux/sshfs
sudo docker plugin ls
sudo docker plugin disable [PLUDIN ID]
sudo docker plugin set vieux/sshfs sshkey.source=/root/.ssh/
sudo docker plugin enable [PLUDIN ID]
sudo docker plugin ls
```

```shell
root@pod0:~# docker plugin ls
ID                  NAME                 DESCRIPTION               ENABLED
b0e65502c3d3        vieux/sshfs:latest   sshFS plugin for Docker   true
```

#### 7. Membuat volume dengan driver sshfs & Uji Jalankan
```shell
sudo docker volume create --driver vieux/sshfs -o sshcmd=root@10.1.40.18:/share  -o allow_other sshvolume
```

```shell
root@pod0:~# sudo docker run -d --name=nginxtest-ssh -p 8090:80 -v sshvolume:/usr/share/nginx/html nginx:latesta3b6f5ea53b4768802bbcd6480a1206eece95e5f2de6295ee4b07f743d9bb194
```

#### 8. SSH ke pod-pod1.
```shell
ssh -l root 10.1.40.18
sudo sh -c "echo 'Hello, I am palopalepalo' > /share/index.html"
sudo cat /share/index.html
exit
```

#### 9. Eksekusi di pod-pod0
```shell
sudo docker ps
curl http://localhost:8090

Hello, I am palopalepalo
```
# Enjoy Happy,  Ngoprek ~
