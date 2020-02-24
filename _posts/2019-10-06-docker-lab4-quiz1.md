---

title : "[Docker] Web Kalkulator Lab Docker (Part 4)"
categories: [ngoprek, server, cloud, docker, container]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

Lab kali ini kita akan mencoba Improve dari lab, jadi kita akan menjalankan web server apache2 yang menjalankan file static kalkulator. dan menggunakan Database SQL untuk RDBMS nya Mysql sebagai penyimpan data user untuk website simabes.

Jadi sini kita akan ngorep 2 web sekaligus !

#### Eksekusi di node `pod-pod0` ###

## WEB KALKULATOR & Push Image di Docker Hub

#### 1. Buat Dockerfile untuk menjalankan aplikasi yang bersumber dari [https://github.com/rivawahyuda/kalkulator](https://github.com/rivawahyuda/kalkulator)

```shell
root@pod0:~/kalkulator# nano Dockerfile 

FROM ubuntu:latest
MAINTAINER NaufalAmmar <naufalammarun11@gmail.com>

ENV DEBIAN_FRONTEND noninteractive

# Install basics
RUN apt-get update
RUN apt-get install -y software-properties-common && \
add-apt-repository ppa:ondrej/php && apt-get update

# Install PHP 5.6
RUN apt-get install -y php5.6 php5.6-mysql php5.6-mcrypt php5.6-cli php5.6-gd php5.6-curl

# Enable apache mods.
RUN a2enmod php5.6
RUN a2enmod rewrite

# Update the PHP.ini file, enable <? ?> tags and quieten logging.
RUN sed -i "s/short_open_tag = Off/short_open_tag = On/" /etc/php/5.6/apache2/php.ini
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/5.6/apache2/php$

# Manually set up the apache environment variables
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Expose apache.
EXPOSE 80

# Install git and git clone file web.
RUN apt-get -y install git
WORKDIR /var/www/
RUN git clone https://github.com/rivawahyuda/kalkulator.git

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf

# By default start up apache in the foreground, override with /bin/bash for interative.
CMD /usr/sbin/apache2ctl -D FOREGROUND
```

Run Test Kalkulator 
```shell
root@pod0:~/kalkulator# docker build -t kalkulatorpalo .
root@pod0:~/kalkulator# docker run -d -p 4000:80 kalkulatorpalo
root@pod0:~/kalkulator# curl localhost:4000
```

Test On browser dengan Tunneling Web

![docker-kalkulator](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker-kalkulator.png)

#### 2. Push image kalkulatorpalo 1 ke docker hub
Docker Login
```shell
root@pod0:~/kalkulator# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: palopalepalo
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded 
```
```shell
root@pod0:~/kalkulator# sudo docker tag kalkulatorpalo palopalepalo/kalkulatorpalo:quiz1
root@pod0:~/kalkulator# sudo docker push palopalepalo/kalkulatorpalo:quiz1
The push refers to repository [docker.io/palopalepalo/kalkulatorpalo]
9179611aeb18: Pushed 
164d002d23d8: Pushed 
6ccf6491359e: Pushed 
d358e7a4f7d2: Pushed 
14121515420d: Pushed 
1f3122b32fad: Pushed 
678ef804045f: Pushed 
7d54735ef9b3: Pushed 
0817fa21a532: Pushed 
4598c17cdb29: Pushed 
e80c789bc6ac: Mounted from library/ubuntu 
6c3332381368: Mounted from library/ubuntu 
ef1a1ec5bba9: Mounted from library/ubuntu 
a1aa3da2a80a: Mounted from library/ubuntu 
quiz1: digest: sha256:e1d7226511127f975b07c474ecd693922f43f95075eb6cb2dc62763666c17ba1 size: 3248
```

# Happy,  Enjoy ngoprek ~
