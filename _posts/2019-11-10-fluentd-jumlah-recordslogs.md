---

title : "[Fluentd] Menghitung Jumlah Keseluruhan Records Log Lab fluentd"
categories: [ngoprek, cloud , log, cloud native]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan kali ini saya akan melanjutkan lab fluentd.

### menambahkan metric untuk menghitung jumlah keseluruhan records log.


sudo systemctl status docker
```
### Add td-agent.conf
```shell
sudo vim /etc/td-agent/td-agent.conf
---
# Monitoring fluentd ke prometheus 
<source>
  @type prometheus
  bind 0.0.0.0
  port 24231
</source>

<filter docker.**>
  @type prometheus
  <metric>
    name fluentd_input_status_num_records_total
    type counter
    desc The total number of incoming records
    #key log
    <labels>
      tag ${tag}
      hostname ${hostname}
    </labels>
  </metric>
</filter>

<match docker.*>
  @type copy
  <store>
     @type stdout
  </store>
<store>
    @type prometheus
    <metric>
      name fluentd_output_status_num_records_total
      type counter
      desc The total number of outgoing records
      <labels>
        tag ${tag}
        hostname ${hostname}
      </labels>
    </metric>
  </store>
</match>

---
sudo systemctl restart td-agent.service
```

Jalankan Container dengan perintah berikut

```shell
sudo docker run -d --log-driver=fluentd --log-opt tag="docker.{{.ID}}" nginx:latest echo 'Iki buat Latihan Lur' -p 9000:9000
```


**Referensi**
* [https://docs.fluentd.org/deployment/monitoring-prometheus#step-4-check-the-configuration](https://docs.fluentd.org/deployment/monitoring-prometheus#step-4-check-the-configuration)

# Happy,  Enjoy ngoprek~
