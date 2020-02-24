---

title : "[Fluentd] lab setup Installation fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---
## Overview

### Halo Sobat ambyarr, di tulisan kali ini saya akan lab fluentd apa sih itu fluentd?

### Fluentd adalah pengumpul log open-source yang sepenuhnya gratis dan memungkinkan kita untuk memiliki arsitektur ' Log di sekala sistem atau aplikasi yang kita buat ' dengan [125+ jenis sistem](https://www.fluentd.org/plugins) .

![fluentd-architecture](https://blobscdn.gitbook.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-LR7OsqPORtP86IQxs6E%2F-LWNPJuIG9Ym5ELlFCti%2F-LWNPOPNQ1l9hvoJ2FIp%2Ffluentd-architecture.png?generation=1547671545415964&alt=media)

> Fluentd membuat log sebagai JSON, format yang populer saat ini di mana dapat dibaca dengan mudah. dan fluentd Ini ditulis dengan bahasa C dengan sedikit bumbu Ruby yang memberi pengguna fleksibilitas. Skalabilitas Fluentd telah terbukti di lapangan: pengguna terbesarnya saat ini mengumpulkan log dari 50.000+ server.



**Setup LAB**

**Node pod3-node0** 
* IP Address: 10.0.0.10/24 
* Gateway: 10.0.0.1
* Hostname: pod3-node0
* Ubuntu 18.04(ns.1-2-10)

**Node podX-node1** 
* IP Address: 10.X.X.20/24
* Gateway: 10.X.X.1
* Hostname: podX-node1
* Ubuntu 18.04(ns.1-2-10)

## Environtment yang saya gunakan disini yaitu public cloud milik `Microsoft Azure`.


![fluentd-topologi](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3-fl-adm-A.png)

## !! Ekseskusi di node0 !! ##

**Install Fluentd (td-agent)**
```shell
sudo apt update && sudo apt -y upgrade && sudo apt autoremove -y
sudo apt -y install chrony vim git

sudo systemctl start chronyd
sudo systemctl status chronyd
sudo chronyc sources
```

**Increase max # of File Descriptors**
```shell
ulimit -n
```
```shell
sudo vim /etc/security/limits.conf

#isi seperti dibawah
root    soft    nofile  65536
root    hard    nofile  65536
*       soft    nofile  65536
*       hard    nofile  65536
```

**Reboot pod3-node0**
```shell
reboot
```

**Jalankan Service Fluentd**
```shell
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent3.sh | sh

sudo systemctl start td-agent.service
sudo systemctl status td-agent.service
```

**Uji Coba log via HTTP**
```shell
curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test

tail -f /var/log/td-agent/td-agent.log
```

hasil uji coba log 
```shell
root@pod03-node0:~# curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log
2019-11-09 10:30:10 +0000 [warn]: #0 [output_td] secondary type should be same with primary one primary="Fluent::Plugin::TreasureDataLogOutput" secondary="Fluent::Plugin::FileOutput"
2019-11-09 10:30:10 +0000 [info]: adding match pattern="debug.**" type="stdout"
2019-11-09 10:30:10 +0000 [info]: adding source type="forward"
2019-11-09 10:30:10 +0000 [info]: adding source type="http"
2019-11-09 10:30:10 +0000 [info]: adding source type="debug_agent"
2019-11-09 10:30:10 +0000 [info]: #0 starting fluentd worker pid=9042 ppid=9033 worker=0
2019-11-09 10:30:10 +0000 [info]: #0 [input_debug_agent] listening dRuby uri="druby://127.0.0.1:24230" object="Fluent::Engine" worker=0
2019-11-09 10:30:10 +0000 [info]: #0 [input_forward] listening port port=24224 bind="0.0.0.0"
2019-11-09 10:30:10 +0000 [info]: #0 fluentd worker is now running worker=0
2019-11-09 10:32:49.258334709 +0000 debug.test: {"json":"message"}
```


# Happy,  Enjoy ngoprek~
