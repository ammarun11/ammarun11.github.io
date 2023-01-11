---
title: "[LoadBalancing] Configure HAProxy and KeepAlived"
date: 2023-01-12
categories: [ngoprek, server, cloud, storage ]
tags:
  - Jekyll
  - update
---

## Guide ## Prerequisites

## Configure HAProxy and KeepAlived

**Note: Configure on lab-oddo1 & lab-oddo2**

### 1\. Install packages

```
apt install –y haproxy keepalived
```

### 2\. Configure KeepAlived

#### lab-oddo1

```
#/etc/keepalived/keepalived.conf 
```

```
vrrp_script chk_haproxy {
```

```
  script "killall -0 haproxy"
```

```
  interface ens3                      # Network card
```

```
    10.60.60.170/24 dev ens3        # The VIP address
```

```
systemctl restart keepalived
```

#### lab-oddo2

```
#/etc/keepalived/keepalived.conf 
```

```
vrrp_script chk_haproxy {
```

```
  script "killall -0 haproxy"
```

```
  interface ens3                      # Network card
```

```
    10.60.60.170/24 dev ens3        # The VIP address
```

```
systemctl restart keepalived
```

#### Check KeepAlived VIP

![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/0keepalive.png)

KeepAlived VIP is assigned on lab-oddo1

### 3\. Configure HAProxy

**Note: Configure on lab-oddo1 & lab-oddo2**

```
#/etc/haproxy/haproxy.cfg 
```

```
    http-check expect status 200 
```

```
    default-server inter 3s fall 3 rise 2 on-marked-down shutdown-sessions 
```

```
    server postgresql_10.60.60.87_db1 10.60.60.87:5432 maxconn 100 check port 8008 
```

```
    server postgresql_10.60.60.88_db2 10.60.60.88:5432 maxconn 100 check port 8008
```

```
    server postgresql_10.60.60.89_db3 10.60.60.89:5432 maxconn 100 check port 8008
```

```
    default_backend odoo-app-server 
```

```
    server web1 10.60.60.83:8069 check   
```

```
    server web2 10.60.60.84:8069 check backup
```

```
systemctl restart haproxy
```

### Access HAProxy Stats

`htttp://10.60.60.170:9000`

![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1keepalive.png)

