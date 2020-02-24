---

title : "[Fluentd] Monitoring fluentd to prometheus Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyar, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Monitoring fluentd ke prometheus


## !! Ekseskusi di node0 !!
### Install prometheus plugin
```shell
sudo td-agent-gem install fluent-plugin-prometheus
```
### Tambahkan konfigurasi td-agent.conf
```shell
sudo vim /etc/td-agent/td-agent.conf

---
<source>
   @type prometheus
   bind 0.0.0.0
   port 24231
</source>

<filter docker.**>
   @type prometheus
   <metric>
      name docker_command_log
      type counter
      desc total docker commands
      #key log
   </metric>
</filter>

<match docker.*>
   @type copy
   <store>
      @type stdout
   </store>
</match>
---

sudo systemctl restart td-agent.service
```
---
## !! Eksekusi di Node 1 !!

### Install prometheus 
```shell
sudo -i

cd /opt

wget https://github.com/prometheus/prometheus/releases/download/v2.9.2/prometheus-2.9.2.linux-amd64.tar.gz

tar xvfz prometheus-2.9.2.linux-amd64.tar.gz

cd prometheus-2.9.2.linux-amd64
```
Tambahkan text berikut untuk target fluentd yang ada di node0
```shell
vim prometheus.yml
---
  - job_name: 'fluentd'
    static_configs:
    - targets: ['10.0.0.10:24231']  # IP node0 fluentd
---
```
### Start prometheus 

./prometheus --config.file=/opt/prometheus-2.9.2.linux-amd64/prometheus.yml

**Akses prometheus http://[ip-public-node1]:9090/targets**

![fluentd-target](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3-fl-adm-J.png)


### Lakukan curl metrics fluentd ke ip Node0
```shell
curl http://[10.0.0.10]:24231/metrics

root@pod03-node1:~# curl 10.0.0.10:24231/metrics
# TYPE docker_command_log counter
# HELP docker_command_log total docker commands
```

# Eksekusi di Node0

### Jalankan container docker
```shell
sudo docker run -ti --name test --log-driver=fluentd --log-opt tag="docker.{{.ID}}" ubuntu /bin/bash
```

### Masukan perintah
```shell
echo 1
echo 2
echo "Hallo, Nolsatu 03"
```

### Query prometheus

**Akses http://[ip-public-node1]:9090/graph**

Pada form expression ketikkan: `docker_command_log`
Klik execute

**HASIL GRAPHnya**
![fluentd-graph](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3-fl-adm-M.png)

Hasil log nya.
```shell
root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log


2019-11-09 17:26:49 +0000 [info]: #0 [input_forward] listening port port=24224 bind="0.0.0.0"
2019-11-09 17:26:49 +0000 [info]: #0 fluentd worker is now running worker=0
2019-11-09 17:29:25.000000000 +0000 docker.e5b15a62701f: {"container_name":"/test","source":"stdout","log":"\u001b]0;root@e5b15a62701f: /\u0007root@e5b15a62701f:/# ","partial_message":"true","partial_id":"b62db46369eaf2ab02372ee97ad9d06f957585105396cf3d333cb216e62cbabe","partial_ordinal":"1","partial_last":"false","container_id":"e5b15a62701f6efc6f642c8c3f9adfe604a22300ccf72e3ed26b320a933b0ee6"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"log":"\u001b]0;root@a58c00198394: /\u0007root@a58c00198394:/# \r\u001b[K\u001b]0;root@a58c00198394: /\u0007root@a58c00198394:/# echo 1\r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test","source":"stdout"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test","source":"stdout","log":"1\r"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"log":"\u001b]0;root@a58c00198394: /\u0007root@a58c00198394:/# echo 2\r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test","source":"stdout"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"source":"stdout","log":"2\r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"source":"stdout","log":"\u001b]0;root@a58c00198394: /\u0007root@a58c00198394:/# echo \"Hallo, Nolsatu 03\"\r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test"}
2019-11-09 17:29:34.000000000 +0000 docker.a58c00198394: {"log":"Hallo, Nolsatu 03\r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test","source":"stdout"}
2019-11-09 17:29:35.000000000 +0000 docker.a58c00198394: {"source":"stdout","log":"\u001b]0;root@a58c00198394: /\u0007root@a58c00198394:/# \r","container_id":"a58c0019839446c57c11f3fc0bc66d97e14221cb070e92a3d408c716a22f0357","container_name":"/test"}
```
**Referensi**
* [https://docs.fluentd.org/deployment/monitoring-prometheus](https://docs.fluentd.org/deployment/monitoring-prometheus)


# Happy,  Enjoy ngoprek~
