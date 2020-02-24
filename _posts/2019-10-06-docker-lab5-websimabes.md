---

title : "[Docker] Web Simabes Lab Docker (Part 5)"
categories: [ngoprek, server, cloud, docker, container]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

Oke meneruskan lab sebelumnya dimana kita membangun web kalkulator sekarang kita mencoba untuk ngoprek Web (Simabes Sistem Informasi Manajemen Bengkel Sederhana)

# WEB Simabes

#### Ekseskusi di `pod-pod1`
#### 1. Setting up mysql server
```shell
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
**Cari bind-address ganti nilainya dari 127.0.0.1 menjadi 0.0.0.0 atau langsung menggunakan IP address dari server.**
* bind-address = 127.0.0.1, server hanya menerima koneksi TCP/IP yang masuk melalui 127.0.0.1.
* bind-address = 0.0.0.0, server menerima koneksi TCP/IP yang masuk melalui semua IPv4 yang ada pada interface jaringan.
* Dapat memasang lebih dari satu bind-address seperti gambar di bawah ini.

```shell 
sudo systemctl restart mysql
```

```shell
$ mysql -u root -p
mysql > CREATE DATABASE mydb;
mysql > CREATE USER 'palo'@'%' IDENTIFIED BY 'rahasia';
mysql > GRANT ALL PRIVILEGES ON mydb.* TO 'palo'@'%';
mysql > FLUSH PRIVILEGES; 
```

#### Ekseskusi di `pod-pod0`
#### 1. Menjalankan docker container phpmyadmin untuk meremote database yang ada di pod-pod1

```shell
docker run --name phpmyadminpalo -d -e PMA_HOST=10.1.40.18 -p 8044:80 phpmyadmin/phpmyadmin 
```
Test On Browser dengan di tunnel dengan login akun mysql yang kita telah buat di pod-pod1.

![docker-phpmyadmin](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker-phmyadmin.png)

---

#### 2. Lakukan git clone ke file web simabes dan edit beberapa codingannya.

```shell 
root@pod0:~/gaskeun# ls
Dockerfile  apache-config.conf  simabes
root@pod0:~/gaskeun# git clone https://github.com/winardiaris/simabes.git
root@pod0:~/gaskeun# nano simabes/inc/koneksi.php 
```
Edit Script koneksi.php sesuaikan dengan database kita.
```php
<?php
        ob_start();
  
        // koneksi ke database
        // user yang dicantumkan di sini harus memiliki hak membuat database
        $dbserver="10.1.40.18";
        $dbusername="palo";
        $dbpassword="rahasia";
        $dbname="mydb";

        //koneksi
        mysql_connect($dbserver,$dbusername,$dbpassword)  or die(mysql_error());
        mysql_select_db($dbname) or die  (mysql_error());
?>
```
Import script simabes_example.sql
ke mysql pod-pod1 dengan phpmyadmin

![Docker-mysql](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/docker-mysql.png)

tabel database untuk web simabes akan otomatis ter import ke database kita.

Buat file apache-config.conf
```shell
root@pod0:~/gaskeun# nano apache-config.conf 
<VirtualHost *:80>
  ServerAdmin me@mydomain.com
  DocumentRoot /var/www/simabes

  <Directory /var/www/simabes/>
      Options Indexes FollowSymLinks MultiViews
      AllowOverride All
      Order deny,allow
      Allow from all
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```


#### 3. Script Dockerfile untuk Web Simabes dan Eksekusi

```shell
FROM ubuntu:latest
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
RUN sed -i "s/error_reporting = .*$/error_reporting = E_ERROR | E_WARNING | E_PARSE/" /etc/php/5.6/apache2/php.ini

# Manually set up the apache environment variables
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid

# Expose apache.
EXPOSE 80

WORKDIR /var/www/

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf
ADD . /var/www/
```

Jalankan

```shell
root@pod0:~# docker build -t simabespalo .
root@pod0:~# docker run -d -p 9191:80 simabespalo 
```
Lalu jalankan di web browser dengan cara tunneling 

![Docker-simabes](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/Docker-simabes.png)

# Enjoy Happy,  Ngoprek ~