---

title : "[Cisco] Konfigurasi Router Cisco dengan telnetlib python"
categories: [ngoprek, automation, routing , cisco]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

### Halo Sobat ambyarr, di tulisan yang singkat ini saya akan lab penggunaan library telnetlib punya python untuk konfigurasi Router Cisco.


**Requirements**
* GNS3 / GNS3VM (Kita pake GNS3)
* Image IOS cisco apa bae saya pake c7200

---

## Disini kita akan memakai applience Network Automation yang tersedia di GNS3 `Marketplace`.

![telnetlib-applience](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/telnetlib-applience-automation.png)

**Start a new blank project, click File > Import Appliance, lalu pilih networkautomation appliance file (.gns3a).**


![telnetlib-installna](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/telnetlib-installNA.png)


**Disini saya menginstall nya di gns3 local, jadi resource yang di gunakan langsung menggunakan perangkat laptop. dan si Node NetworkAutomation ini terinstall di `docker`**.

```shell
root@ilmi:/home/tholib# docker image ls
REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
node-docker-alpine            latest              44fc11081331        2 months ago        199MB
adosztal/network_automation   latest              5a75f0437a0a        5 months ago        597MB

root@ilmi:/home/tholib# sudo docker ps
CONTAINER ID        IMAGE                                COMMAND                  CREATED             STATUS              PORTS               NAMES
eb8ad0174086        adosztal/network_automation:latest   "/gns3/init.sh sh -c…"   About an hour ago   Up About an hour                        xenodochial_payne
```

**Lihat disitu tersedia image NetworkAutomation yang jika kita jalankan di GNS3 maka Image tersebut akan start/run ke docker secara otomatis**

## **Configuration**


![telnetlib-toplogy](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/telentlib-toplogy.png)

**Kita beri terlebih dahulu ke Node Router dan Networkautomation dengan Network yang sama**

* Disini saya menggunakan ip 192.168.10.0/24

>Router1 Untuk menset ip & telnet pada router 
```shell
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)# interface f0/0
R1(config-if)#ip add 192.168.10.2 255.255.255.0
R1(config-if)#no sh
R1(config-if)#exit
R1(config)#username palo secret palo
R1(config)#username palo privilege 15
R1(config)#line vty 0 4
R1(config-line)#login local
```

>NetworkAutomation
```shell
root@NetworkAutomation-2:~# ifconfig eth0 192.168.10.1
root@NetworkAutomation-2:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 92:9a:f9:ac:7b:40  
          inet addr:192.168.10.1  Bcast:192.168.10.255  Mask:255.255.255.0
          inet6 addr: fe80::909a:f9ff:feac:7b40/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:89 errors:0 dropped:5 overruns:0 frame:0
          TX packets:62 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:19000 (19.0 KB)  TX bytes:3807 (3.8 KB)
lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

***Setelah itu pastikan kedua node tersebut berhasil saling test ping***

Oke lalu kita membuat script python nya untuk mengkonfigurasi si router

>ke networkAutomation

```python
root@NetworkAutomation-2:~# nano telnetlibv1.py
import telnetlib

host = "192.168.10.2"
user = "palo"
password = "palo"

tn = telnetlib.Telnet(host)

tn.read_until("Username: ")
tn.write(user + "\n")

tn.read_until("Password: ")
tn.write(password + "\n")

tn.write("conf t\n")
tn.write("int lo0\n")
tn.write("ip address 1.1.1.1 255.255.255.0\n")
tn.write("end\n")
tn.write("exit\n")

print tn.read_all() 
root@NetworkAutomation-2:~# ls
telnetlibv1.py
```

***JALANKAN***

* Node Network Automation

```shell
root@NetworkAutomation-2:~# python telnetlibv1.py 

R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int lo0
R1(config-if)#ip address 1.1.1.1 255.255.255.0
R1(config-if)#end
R1#exit
```

Oke untuk melihat konfig nya berhasil atau tidak kita lihat node si router dah kita show ip nya.

* Node Router

```shell
R1#show ip interface brief 
Interface                  IP-Address      OK? Method Status                Protocol
FastEthernet0/0            192.168.10.2    YES NVRAM  up                    up      
FastEthernet0/1            unassigned      YES NVRAM  administratively down down        
Loopback0                  1.1.1.1         YES manual up                    up      
R1#
```

***Berhasil***

---
**KESIMPULAN**
Kita bisa membuat script otomasi untuk mengkonfig router cisco dengan library python yaitu telnetlib. telnetlib menggunakan protokol telnet yang dimiliki oleh router.



**Referensi**
* [https://pypi.org/project/telnetlib3/](https://github.com/adosztal/gns3-containers)
* [https://pypi.org/project/telnetlib3/](https://pypi.org/project/telnetlib3/)




# Happy,  Enjoy ngoprek~
