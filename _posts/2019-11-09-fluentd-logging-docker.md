---

title : "[Fluentd] Logging Docker Container Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Logging Docker Container


### Install docker
```shell
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update && sudo apt install docker-ce

sudo systemctl status docker
```
### Add td-agent.conf
```shell
sudo vim /etc/td-agent/td-agent.conf

<match docker.*>
   @type stdout
</match>
```
```shell
sudo systemctl restart td-agent.service
```

Jalankan Container dengan perintah berikut

```shell
sudo docker run --log-driver=fluentd --log-opt tag="docker.{{.ID}}" ubuntu echo 'Hello World, My name is palopalepalo'
```

Hasil log nya.
```shell
root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log

2019-11-09 15:27:14.000000000 +0000 docker.31dcf1dc69d0: {"log":"Hello World, My name is palopalepalo","container_id":"31dcf1dc69d020ac803567d1c754029b348f2f826468ee918dcf14decb4ed2d3","container_name":"/flamboyant_bell","source":"stdout"}
```

**Referensi**
* [https://docs.fluentd.org/container-deployment/docker-logging-driver#getting-started](https://docs.fluentd.org/container-deployment/docker-logging-driver#getting-started)

# Happy,  Enjoy ngoprek~
