---
title: "[RemoteDesktop] Setup RDP Protocol with Apache Guacamole on Oracle 8"
date: 2023-01-11
categories: [ngoprek, server, cloud, storage, rdp, guacamole ]
tags:
  - Jekyll
  - update
---

## Guide ## Prerequisites

# Topology

![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2guacamole.png)

# Install Apache Guacamole Native Oracle 8 <br>

![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2guacamole.png)

# Install Dependencies
<details><summary>Click to expand</summary>
<br>

```
dnf update
dnf install epel-release
dnf install -y vim wget unzip make cmake wget gcc zlib-devel compat-openssl10
dnf config-manager --enable ol8_codeready_builder
dnf -y install cairo-devel libuv-devel libjpeg-turbo-devel libjpeg-devel libpng-devel libtool uuid-devel freerdp-devel pango-devel libvncserver-devel pulseaudio-libs-devel openssl-devel libvorbis-devel libwebp-devel libssh2-devel libwebsockets-devel libtheora opus lame-libs
dnf config-manager --disable devel
```

Install other libraries from source <br>

```
curl -s https://api.github.com/repos/seanmiddleditch/libtelnet/releases/latest | grep browser_download_url  | cut -d '"' -f 4 | wget -qi -
tar -xf libtelnet-*.tar.gz
cd libtelnet-*/
./configure
make && sudo make install
```
</details>

# Install Java
<details><summary>Click to expand</summary>
<br>

1. Fetch java-11-openjdk <br>
```
yum install java-11-openjdk-devel
```

2. Create bash file and set java env variables <br>
```
vim /etc/profile.d/java11.sh
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which javac))))
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

3. Source to java.sh <br>
```
source /etc/profile.d/java11.sh
```

</details>

# Install Apache Tomcat
<details><summary>Click to expand</summary>
<br>

1. Create tomcat user and group <br>
```
sudo groupadd --system tomcat
sudo useradd -d /usr/share/tomcat -r -s /bin/false -g tomcat tomcat
```

2. Install tomcat <br>
```
yum -y install wget
export VER="9.0.70"
wget https://archive.apache.org/dist/tomcat/tomcat-9/v${VER}/bin/apache-tomcat-${VER}.tar.gz
tar xvf apache-tomcat-${VER}.tar.gz -C /usr/share/
ln -s /usr/share/apache-tomcat-$VER/ /usr/share/tomcat
chown -R tomcat:tomcat /usr/share/tomcat
chown -R tomcat:tomcat /usr/share/apache-tomcat-$VER/ 
```

3. Configure tomcat <br>
```
vim /etc/systemd/system/tomcat.service
---
[Unit]
Description=Tomcat
After=syslog.target network.target

[Service]
Type=forking
User=tomcat
Group=tomcat

Environment=JAVA_HOME=/usr/lib/jvm/jre-openjdk
Environment='JAVA_OPTS=-Djava.awt.headless=true'

Environment=CATALINA_HOME=/usr/share/tomcat
Environment=CATALINA_BASE=/usr/share/tomcat
Environment=CATALINA_PID=/usr/share/tomcat/temp/tomcat.pid

ExecStart=/usr/share/tomcat/bin/catalina.sh start
ExecStop=/usr/share/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
```

4. Start and enable <br>
```
systemctl daemon-reload
systemctl restart tomcat
systemctl enable tomcat
systemctl status tomcat
```

5. Configure tomcat proxy <br>
```
yum -y install httpd 
vim /etc/httpd/conf.d/tomcat_manager.conf
---
<VirtualHost *:443>
    ServerAdmin root@localhost
    ServerName ap-ol8-2
    DefaultType text/html
    ProxyRequests off
    ProxyPreserveHost On
    ProxyPass / http://cx-guacamole.btech.id:8080/
    ProxyPassReverse / http://cx-guacamole.btech.id:8080/
</VirtualHost>
```

6. Enable and restart <br>
```
systemctl restart httpd && sudo systemctl enable httpd
```

7. Access tomcat <br>
```
cx-guacamole.btech.id
```

</details>

# Install Guacamole server from source
<details><summary>Click to expand</summary>
<br>

```
wget https://downloads.apache.org/guacamole/1.4.0/source/guacamole-server-1.4.0.tar.gz
tar -xvf guacamole-server-1.4.0.tar.gz
./configure --with-init-dir=/etc/init.d
make
make install
ldconfig
systemctl daemon-reload
systemctl start guacd
systemctl enable guacd
systemctl status guacd
```
[![](https://book.btech.id/uploads/images/gallery/2023-01/scaled-1680-/BqqMXoi0vTXVlUoB-image-1673286532612.png)](https://book.btech.id/uploads/images/gallery/2023-01/BqqMXoi0vTXVlUoB-image-1673286532612.png) <br>

</details>

# Install Guacamole Client 
<details><summary>Click to expand</summary>
<br>

```
mkdir /etc/guacamole
wget https://downloads.apache.org/guacamole/1.4.0/binary/guacamole-1.4.0.war
mv guacamole-1.4.0.war /etc/guacamole/guacamole.war
sudo ln -s /etc/guacamole/guacamole.war /usr/share/tomcat/webapps/
```

GUACAMOLE_HOME is the name given to Guacamole’s configuration directory, which is located at `/etc/guacamole` by default. <br>

1. Create GUACAMOLE_HOME environment variable <br>
```
echo "GUACAMOLE_HOME=/etc/guacamole" | sudo tee -a /etc/default/tomcat
```

2. Create /etc/guacamole/guacamole.properties <br>
```
vim /etc/guacamole/guacamole.properties
guacd-hostname: localhost
guacd-port:    4822
user-mapping:    /etc/guacamole/user-mapping.xml
auth-provider:    net.sourceforge.guacamole.net.basic.BasicFileAuthenticationProvider
```

3. Link to tomcat <br>
```
sudo ln -s /etc/guacamole /usr/share/tomcat/.guacamole
```

</details>

# Setup Guacamole Authentication
<details><summary>Click to expand</summary>
<br>

1. Generate md5 password <br>
```
echo -n gladiators88 | openssl md5
```

2. Edit file `user-mapping.xml` <br>
```
vim /etc/guacamole/user-mapping.xml
---
<user-mapping>
  
    <!-- Per-user authentication and config information -->

    <!-- A user using md5 to hash the password
         guacadmin user and its md5 hashed password below is used to 
             login to Guacamole Web UI-->
    <authorize 
            username="admin"
            password="ae6cc7f6a8258fffb39db27023135698"
            encoding="md5">
    </authorize>

</user-mapping>
```


3. Restart tomcat and guacd
```
systemctl restart tomcat guacd
```

4. Stop and disable firewall
```
systemctl stop firewalld
systemctl start firewalld
```

5. Access guacamole web <br>
```
http://cx-guacamole.btech.id:8080/guacamole
```

## Notes : <br>
- On guacamole native, we cant create connection from web like deploy guacamole from docker <br>

- Guacamole native <br>


- We can add several connections to server through user-mapping.xml configuration <br>

```
<user-mapping>
  
    <!-- Per-user authentication and config information -->

    <!-- A user using md5 to hash the password
         guacadmin user and its md5 hashed password below is used to 
             login to Guacamole Web UI-->
    <authorize 
            username="admin"
            password="ae6cc7f6a8258fffb39db27023135698"
            encoding="md5">

        <connection name="Oracle 8">
            <protocol>rdp</protocol>
            <param name="hostname">172.16.1.20</param>
            <param name="port">3390</param>
            <param name="username">root</param>
            <param name="password">gladiators88</param>
            <param name="ignore-cert">true</param>
        </connection>

        <connection name="Centos 7">
            <protocol>rdp</protocol>
            <param name="hostname">172.16.1.22</param>
            <param name="port">3389</param>
            <param name="username">root</param>
            <param name="ignore-cert">true</param>
        </connection>

    </authorize>

</user-mapping>
---
systemctl restart guacd tomcat
```

<!--  -->

The connections we add on user-mapping.xml will appear. <br>

</details>


## Enable RDP TLS

## Client
1. Generate certificate

```
mkdir certs2 && cd certs2
openssl req -x509 -newkey rsa:2048 -nodes -keyout xrdpkey.pem -out xrdpcert.pem -days 3650
cp xrdpkey.pem xrdpcert.pem /etc/xrdp/
```

2. Trust certificate

```
cp xrdpcert.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust s
```

3. Update xrdp configuration

```
nano /etc/xrdp/xrdp.ini

---
security_layer=tls
certificate=/etc/xrdp/xrdpcert.pem
key_file=/etc/xrdp/xrdpkey.pem
ssl_protocols=TLSv1, TLSv1.1, TLSv1.2, TLSv1.3
---

systemctl restart xrdp
```

## Server
1. Edit user-mapping.xml on guacamole server <br>

```
nano /etc/guacamole/user-mapping.xml 
---
        <connection name="Oracle 8 Public - 103.93.57.24:3389">
            <protocol>rdp</protocol>
            <param name="hostname">103.93.57.24</param>
            <param name="port">3389</param>
            <param name="username">root</param>
            <param name="password">gladiators88</param>
            <param name="security">tls</param>             <---- ## Change from `any` to `tls`
            <param name="ignore-cert">true</param>
        </connection>
---
systemctl restart tomcat guacd
```

2. Access guacamole <br>
```
https://cx-guacamole.btech.id/guacamole
```

3. Test Connection <br>

![Dashboard3](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3guacamole.png)
