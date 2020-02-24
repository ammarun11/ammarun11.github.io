---

title : "[Docker] Docker Compose Lab Docker (Part 7)"
categories: [ngoprek, server, cloud, docker, container]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Lab ini kita akan ngoprek Docker menggunakan Compose, bisa di bilang compose ini seperti automation nya docker jadi dengan compose kita bisa membuat container menjadi lebih cepat dan efesien.

## Default Bridge Network #####

#### Eksekusi di node `pod-pod0` ###
#1. Unduh Compose & Set permisson executable
```shell
curl -L https://github.com/docker/compose/releases/download/1.20.1 docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

```shell
root@pod0:~# docker-compose --version
docker-compose version 1.20.1, build 5d8c71b
```
## Compose & Wordpress ##### 

#### Disini Kita akan membanung Web Wordpress dengan Docker menggunakan Compose.

Buat direktori my_wordpress dan masuk ke direktori tersebut
```shell
mkdir /latihan/my_wordpress
cd /latihan/my_wordpress
```
Buat file docker-compose.yml
```shell
root@pod0:~/latihan/my_wordpress# vim docker-compose.yml

version: '3.2'
services:
   db:
     image: mysql:5.7
     volumes:
       - dbdata:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: [username]
       MYSQL_PASSWORD: [password]

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: [username]
       WORDPRESS_DB_PASSWORD: [password]
volumes:
    dbdata:
```

Jalankan compose dan lihat hasilnya
```shell
docker-compose up -d
docker container ls
```

```shell
root@pod0:~/latihan/my_wordpress# docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED              STATUS              PORTS                                               NAMES
b2c718be67d1        wordpress:latest        "docker-entrypoint.s…"   About a minute ago   Up 58 seconds       0.0.0.0:8000->80/tcp                                mywordpress_wordpress_1
20061a28cec4        mysql:5.7               "docker-entrypoint.s…"   About a minute ago   Up 59 seconds       3306/tcp, 33060/tcp                                 mywordpress_db_1
```

![compose-wp](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/compose-wp.png)

Hapus container, default network dan database wordpress

```shell
docker-compose down --volumes
```

## Compose & app.py #####

Membuat web dengan flask dan redis menggunakan compose
Buat direktori my_app dan masuk ke direktori tersebut

```shell
mkdir /latihan/my_app
cd /latihan/my_app
```
Buat File app.py
```shell
root@pod0:~/latihan/my_app# vim app.py

import time

import redis
from flask import Flask


app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

Buat file requirements.txt
```shell
root@pod0:~/latihan/my_app# vim requirements.txt

flask
redis
```

Lalu buat Dockerfile
```shell
root@pod0:~/latihan/my_app# vim Dockerfile

FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

Buat file docker-compose.yml
```shell
root@pod0:~/latihan/my_app# vim docker-compose.yml

version: '3.2'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

Jalankan compose
```shell
docker-compose up -d
```
Uji browsing dengan cara tunelling

![compose-app](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/compose-app.png)

Menampilkan compose aktif & Menampilkan environment variable di service web
```shell
docker-compose ps
docker-compose run web env
```

```shell
root@pod0:~/latihan/my_app# docker-compose ps
    Name                   Command               State           Ports         
-------------------------------------------------------------------------------
myapp_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp              
myapp_web_1     python app.py                    Up      0.0.0.0:5000->5000/tcp
root@pod0:~/latihan/my_app# docker-compose run web env
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=50450cc97d7b
TERM=xterm
LANG=C.UTF-8
GPG_KEY=97FC712E4C024BBEA48A61ED3A5CA953F73C700D
PYTHON_VERSION=3.4.10
PYTHON_PIP_VERSION=19.0.3
HOME=/root
```


#9. Menampilkan environment variable di service web
docker-compose run web env

```shell
docker network inspect bridge
```

Masuk ke container alpine1
```shell
docker attach alpine1
```
```shell
root@pod0:~# docker attach alpine1
/ # ip add 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
26: eth0@if27: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:11:00:04 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=54 time=5.623 ms
64 bytes from 8.8.8.8: seq=1 ttl=54 time=5.452 ms
64 bytes from 8.8.8.8: seq=2 ttl=54 time=5.402 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 5.402/5.492/5.623 ms
/ # ping 172.17.0.5 
PING 172.17.0.5 (172.17.0.5): 56 data bytes
64 bytes from 172.17.0.5: seq=0 ttl=64 time=0.255 ms
64 bytes from 172.17.0.5: seq=1 ttl=64 time=0.088 ms
64 bytes from 172.17.0.5: seq=2 ttl=64 time=0.082 ms
64 bytes from 172.17.0.5: seq=3 ttl=64 time=0.082 ms
^C
--- 172.17.0.5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.082/0.126/0.255 ms
/ # 
```

> Keluar dari container alpine1 tanpa menutup shell tekan Ctrl+P, Ctrl+Q

Hapus kedua container
```shell
docker container rm -f alpine1 alpine2
```
---

## User-Defined Bridge Network #####

membuat network bridge sendiri untuk menghubungkan alpine2 & 3 saja sedangkan alpine1 by default hanya menggunakan bridge bawaan makan hanya akan terhubung ke alpine 3

```shell
## embuat bridge network alpine-net
docker network create --driver bridge alpine-net

## Tampilkan detil network alpine-net
docker network inspect alpine-net
```

```shell
## Tampilkan daftar network
root@pod0:~# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0581e60d8891        alpine-net          bridge              local
1448af1458d5        bridge              bridge              local
9ddfa1d104bd        host                host                local
1f488f5ddfce        none                null                local
```

Buat 3 container: container alpine1 terhubung ke network default bridge, container alpine2 terhubung ke network alpine-net dan container alpine3 terhubung ke kedua network default bridge dan alpine-net
```shell
docker run -dit --name alpine1 alpine ash
docker run -dit --name alpine2 --network alpine-net alpine ash
docker run -dit --name alpine3 alpine ash
docker network connect alpine-net alpine3
docker container ls
```

```shell
root@pod0:~# docker run -dit --name alpine1 alpine ash
fa5eca2f462fdb546888ef1850738ef6727ca3573db4bba2ba6252bf802eecba
root@pod0:~# docker run -dit --name alpine2 --network alpine-net alpine ash
3d142e9a81a0a50e1beedba7ddbf6ee896f943be5076bd3137e6eed69101a79d
root@pod0:~# docker run -dit --name alpine3 alpine ash
102253236cfdb5a6c196e85e6067ad9f44c42ab2d919e7d38fc45452ae327f07
root@pod0:~# docker network connect alpine-net alpine3
root@pod0:~# docker container ls
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS                  PORTS                                               NAMES
102253236cfd        alpine                  "ash"                    1 second ago        Up Less than a second                                                       alpine3
3d142e9a81a0        alpine                  "ash"                    2 seconds ago       Up Less than a second                                                       alpine2
fa5eca2f462f        alpine                  "ash"                    2 seconds ago       Up 1 second                                                                 alpine1
```

Tampilkan detail network bridge dan alpine-net

```shell
docker network inspect bridge
docker network inspect alpine-net
```

Masuk ke container alpine3 dan uji ping ke alamat ip alpine1 dan ke nama container alpine1 dan alpine2 
```shell
docker attach alpine3
ping -c 3 172.17.0.4 || alpine1
ping -c 3 alpine1 || gagal 
ping -c 3 alpine2 || Sukses
```

```shell
/ # ping -c 3 172.17.0.4
PING 172.17.0.4 (172.17.0.4): 56 data bytes
64 bytes from 172.17.0.4: seq=0 ttl=64 time=0.133 ms
64 bytes from 172.17.0.4: seq=1 ttl=64 time=0.089 ms
64 bytes from 172.17.0.4: seq=2 ttl=64 time=0.089 ms

--- 172.17.0.4 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.089/0.103/0.133 ms
/ # ping -c 3 alpine1
ping: bad address 'alpine1'
/ # ping -c 3 alpine2
PING alpine2 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.060 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.137 ms
64 bytes from 172.18.0.2: seq=2 ttl=64 time=0.122 ms

--- alpine2 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 0.060/0.106/0.137 ms
/ # 
```

Masuk ke container alpine2 dan uji ping ke alamat ip container alpine1 (gagal karena beda bridge network dan beda subnet) dan ke internet (sukses)

```shell
docker attach alpine2
ping -c 3 172.17.0.4
ping -c 3 8.8.8.8phmyadmin
```
```shell
root@pod0:~# docker attach alpine2
/ # ping -c 3 172.17.0.4
PING 172.17.0.4 (172.17.0.4): 56 data bytes
--- 172.17.0.4 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
/ # ping -c 3 8.8.8.8
PING 8.8.8.8 (8.8.8.8): 56 data bytes
64 bytes from 8.8.8.8: seq=0 ttl=54 time=6.304 ms
64 bytes from 8.8.8.8: seq=1 ttl=54 time=5.522 ms
64 bytes from 8.8.8.8: seq=2 ttl=54 time=5.685 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 5.522/5.837/6.304 ms
/ # 
```

Hapus semua container dan network alpine-net

```shell
docker container rm -f alpine1 alpine2 alpine3
docker network rm alpine-net
```

## HOST NETWORK NGINX #####

Di lab ini kita akan membangun Container nginx yang langsung terhubung ke host kita dan berjalan di port 80 

Jalankan container dari image nginx dan uji coba 
```shell
docker run --rm -itd --network host --name my_nginx nginx
curl http://localhost
ip add
netstat -tulpn | grep :80
```

```shell
root@pod0:~# docker run --rm -itd --network host --name my_nginx nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
b8f262c62ec6: Already exists 
e9218e8f93b1: Pull complete 
7acba7289aa3: Pull complete 
Digest: sha256:aeded0f2a861747f43a01cf1018cf9efe2bdd02afd57d2b11fcc7fcadc16ccd1
Status: Downloaded newer image for nginx:latest
24cafe8865559d2ef2f647c46e2783ad256aa14165e9d00e1198c53e5edb0310
root@pod0:~# curl http://localhost
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
root@pod0:~# ip add
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:28:4a:16 brd ff:ff:ff:ff:ff:ff
    inet 10.1.40.12/24 brd 10.1.40.255 scope global ens3
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe28:4a16/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:e3:23:b4:61 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e3ff:fe23:b461/64 scope link 
       valid_lft forever preferred_lft forever
9: veth58779c1@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 4a:4a:a7:e3:ee:25 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::484a:a7ff:fee3:ee25/64 scope link 
       valid_lft forever preferred_lft forever
25: veth0bfdbbc@if24: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether fa:15:b5:c4:01:7d brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::f815:b5ff:fec4:17d/64 scope link 
       valid_lft forever preferred_lft forever
root@pod0:~# netstat -tulpn | grep :80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      21740/nginx: master
tcp6       0      0 :::8044                 :::*                    LISTEN      14262/docker-proxy
```

Hapus container my_nginx
```shell
docker container rm -f my_nginx
```

# Happy,  Enjoy Ngoprek ~
