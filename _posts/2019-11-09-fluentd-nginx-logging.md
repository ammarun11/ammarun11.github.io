---

title : "[Fluentd] Logging Nginx Web Server dengan Docker Container Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Logging Nginx dengan Docker Container


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
---
<source>
      @type syslog
      format nginx
      tag docker.nginx
</source>

<match docker.nginx>
   @type stdout
</match>
---
sudo systemctl restart td-agent.service
```

Jalankan Container dengan perintah berikut

```shell
sudo docker run -d -p 9000:80 --log-driver=fluentd nginx:latest
```

Hasil log nya.
```shell
root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log

2019-11-09 16:31:32 +0000 [info]: #0 fluentd worker is now running worker=0
2019-11-09 16:31:37.000000000 +0000 docker.8b0379f6b4b1: {"log":"10.0.0.10 - - [09/Nov/2019:16:31:37 +0000] \"GET / HTTP/1.1\" 304 0 \"-\" \"Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:69.0) Gecko/20100101 Firefox/69.0\" \"-\"","container_id":"8b0379f6b4b112b0de5474f6bacd8e9db9962fa38e7d3cca199d3fbecc116648","container_name":"/boring_chandrasekhar","source":"stdout"}
2019-11-09 16:31:59.000000000 +0000 docker.8b0379f6b4b1: {"source":"stdout","log":"172.17.0.1 - - [09/Nov/2019:16:31:59 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"curl/7.58.0\" \"-\"","container_id":"8b0379f6b4b112b0de5474f6bacd8e9db9962fa38e7d3cca199d3fbecc116648","container_name":"/boring_chandrasekhar"}
2019-11-09 16:33:29.000000000 +0000 docker.8b0379f6b4b1: {"container_id":"8b0379f6b4b112b0de5474f6bacd8e9db9962fa38e7d3cca199d3fbecc116648","container_name":"/boring_chandrasekhar","source":"stdout","log":"10.0.0.20 - - [09/Nov/2019:16:33:29 +0000] \"GET / HTTP/1.1\" 200 612 \"-\" \"curl/7.58.0\" \"-\""}
```

**Referensi**
* [https://docs.fluentd.org/container-deployment/docker-logging-driver#getting-started](https://docs.fluentd.org/container-deployment/docker-logging-driver#getting-started)

# Happy,  Enjoy ngoprek~
