---
layout : post
title : "[Fluentd] Logging Apache Web Server Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Logging Apache Web Server

## Install Apache
```BASH
sudo apt install -y apache2

sudo systemctl start apache2
sudo systemctl status apache2

sudo usermod -aG adm td-agent
```
```BASH
sudo vim /etc/td-agent/td-agent.conf 

---
<source>
      @type tail
      format apache2
      tag apache.access
      path /var/log/apache2/access.log
</source>

<match apache.access>
   @type stdout
</match>
---

sudo systemctl restart td-agent.service
```



# Akses localhost dan perlihatkan access log apache (td-agent.log)
```BASH
root@pod03-node0:~# curl localhost
```

**lalu di tail file log milik apache**

```BASH
root@pod03-node0:~# tail /var/log/apache2/access.log
127.0.0.1 - - [09/Nov/2019:13:26:44 +0000] "GET / HTTP/1.1" 200 11173 "-" "curl/7.58.0"
127.0.0.1 - - [09/Nov/2019:13:27:36 +0000] "GET / HTTP/1.1" 200 11173 "-" "curl/7.58.0"
127.0.0.1 - - [09/Nov/2019:13:27:38 +0000] "GET / HTTP/1.1" 200 11173 "-" "curl/7.58.0"
```

# Happy Enjoy ngoprek~
