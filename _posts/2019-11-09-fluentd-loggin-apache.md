---

title : "[Fluentd] Logging Apache Web Server Lab Fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### Logging Apache Web Server

## Install Apache
```shell
sudo apt install -y apache2

sudo systemctl start apache2
sudo systemctl status apache2

sudo usermod -aG adm td-agent
```
```shell
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
```shell
root@pod03-node0:~# curl localhost
```

**lalu di tail file log milik apache**

```shell
root@pod03-node0:~# tail -f /var/log/td-agent/td-agent.log 
2019-11-12 03:57:15 +0000 [warn]: #0 'pos_file PATH' parameter is not set to a 'tail' source.
2019-11-12 03:57:15 +0000 [warn]: #0 this parameter is highly recommended to save the position to resume tailing.
2019-11-12 03:57:15 +0000 [info]: #0 starting fluentd worker pid=3203 ppid=3195 worker=0
2019-11-12 03:57:15 +0000 [info]: #0 following tail of /var/log/apache2/access.log
2019-11-12 03:57:15 +0000 [info]: #0 [input_debug_agent] listening dRuby uri="druby://127.0.0.1:24230" object="Fluent::Engine" worker=0
2019-11-12 03:57:15 +0000 [info]: #0 [input_forward] listening port port=24224 bind="0.0.0.0"
2019-11-12 03:57:15 +0000 [info]: #0 fluentd worker is now running worker=0
2019-11-12 03:57:25.000000000 +0000 apache.access: {"host":"10.0.0.20","user":null,"method":"GET","path":"/","code":200,"size":11173,"referer":null,"agent":"curl/7.58.0"}
2019-11-12 03:57:27.000000000 +0000 apache.access: {"host":"10.0.0.20","user":null,"method":"GET","path":"/","code":200,"size":11173,"referer":null,"agent":"curl/7.58.0"}
2019-11-12 03:57:31.000000000 +0000 apache.access: {"host":"127.0.0.1","user":null,"method":"GET","path":"/","code":200,"size":11173,"referer":null,"agent":"curl/7.58.0"}
```

# Happy,  Enjoy ngoprek~
