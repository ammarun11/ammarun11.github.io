---

title : "[Cisco] Static Routing"
categories : [network, cisco, routing]
---

# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم

---

## Introduction to Static Routing   

Static Routing merupakan routing yang pemilihan jalurnya ditetapkan oleh
Administratornya sendiri. Static routing ini lebih aman dibanding Dynamic Routing. Kalau
membangun jaringan usahakan jika routernya belum terlalu banyak jangan terburu – buru untuk menggunakan Dynamic Routing karena jika menggunakan Dynamic Routing kinerja dari sebuah router akan menjadi lebih berat

`Cara konfigurasi Static Routing bisa dengan 3 cara yaitu :`

- `Exit Interface` : Dengan cara ini ketika sudah memasukan IP Network dan Subnetmask dari Router maka kita akan menggunakan interface router tetangga sebagai jalur keluarnya. Agar Router  mengetahui Routing Table yang ada pada Router

- `Next Hop`  :  Next-Hop : Perbedaannya dengan Exit Interface adalah Cuma untuk Next –Hop ini memasukan IP yang directly connect dengan Router.

- `Exit Interface dan Next Hop ` : Gabungan dari Exit Interface dan Next-Hop. Konfigurasinya juga gabungan dari Exit Interface dan Next-Hop.

--- 

###  Lab 1 Static Routing with Exit Interface

![static-extint](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/static-extint.png)

* **Berikut Konfigurasi Router 1**

``` BASH
Router>enable
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#hostname Router1
Router1(config)#interface fa0/1
Router1(config-if)#ip address 100.100.100.1 255.255.255.0
Router1(config-if)#no shutdown
Router1(config-if)#exit
Router1(config)#interface fa0/0
Router1(config-if)#ip address 192.168.10.1 255.255.255.0
Router1(config-if)#no shutdown
Router1(config-if)#exit
Router1(config)#
```

* **Lalu Konfigurasi Router 2**s

``` BASH
Router>enable
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#hostname Router2
Router2(config)#interface fa0/1
Router2(config-if)# ip address 192.168.10.2 255.255.255.0
Router2(config-if)#no shutdown
Router2(config-if)#exit
Router2(config)#interface fa0/0
Router2(config-if)# ip address 200.200.200.1 255.255.255.0
Router2(config-if)#no shutdown
Router2(config-if)#exit
Router2(config)#
```

## Selanjutnya kita konfigurasi static Routing

* **Berikut Konfigurasi Router 1**

``` BASH
Router1(config)#ip route 200.200.200.0 255.255.255.0
FastEthernet 0/0
```

* **Lalu Konfigurasi Router 2**

``` BASH
Router2(config)#ip route 100.100.100.0 255.255.255.0
FastEthernet 0/1
```

Setelah dilakukan Routing, berikut Routing Table nya.

![static-staticextint](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/static-hasilextint.png)

> Sudah saling terhubung, silahkan masukan IP Address pada PC lalu lakukan test ping yang berbeda network.

---

###  Lab 2 Static Routing with Next Hop

![static-nexthop](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/static-nexthop.png)

Masih menggunakan topologi seperti sebelumnya, untuk konfigurasi IP Address sama. Tetapi kita sekarang menggunakan Next-Hop. Pertama hapus terlebih dahulu Static Routing sebelumnya.

Sebelum masuk ke konfigurasi dengan next hop yang harus kita lakukan terlebih dahulu melepas protokol routing yang ada di topologi sebelumnya

* **Berikut Konfigurasi Router 1**

``` BASH
Router1(config)#no ip route 200.200.200.0 255.255.255.0
FastEthernet 0/0
```

* **Lalu Konfigurasi Router 2**s

``` BASH
Router2(config)#no ip route 100.100.100.0 255.255.255.0
FastEthernet 0/1
```

Setelah terhapus sekarang kita check dulu Routing Table nya. Berikut tampilan dari Router1

![static-downroute](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/static-downroute.png)

Disitu terlihat tidak ada tanda S yang menandakan terdapat routing static di routing tabel, maka kita telah berhasil menghapus protokol routing di routing tabel Lakukan hal tsb di Router 2.

* **Sekarang kita konfigurasi Static Routing-nya. Berikut konfigurasi dari Router1**

``` BASH
Router1(config)#ip route 200.200.200.0 255.255.255.0
192.168.10.2
```

* **Lalu Konfigurasi Router 2**

``` BASH
Router2(config)#ip route 100.100.100.0 255.255.255.0
192.168.10.1
```

> Jika sudah dikonfigurasi, lihatlah Routing Table, maka sudah ada kode S yang berartikan sudah ada Static Routing. Lalu percobaan test ping dari PC1 ke PC2 ataupun sebaliknya.

---

###  Lab 3 Static Routing with  Exit Interface and Next-Hop

![static-nexthopextint](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/static-extandnexthop.png)

Masih menggunakan topologi seperti sebelumnya, pengetesanya sama seperti
sebelumnya. Tetapi LAB ini belum support di Cisco Packet Tracer, yang penting paham terlebih
dahulu saja.

Sebelum masuk ke konfigurasi dengan next hop yang harus kita lakukan terlebih dahulu melepas protokol routing yang ada di topologi sebelumnya

* **Berikut Konfigurasi Router 1**

``` BASH
Router1(config)#no ip route 200.200.200.0 255.255.255.0 192.168.10.2
```

* **Lalu Konfigurasi Router 2**s

``` BASH
Router2(config)#no ip route 100.100.100.0 255.255.255.0 192.168.10.1
```

Setelah terhapus sekarang kita check dulu Routing Table nya. Pastikan protokol routing static hilang di routing tabel Router 1 & 2.

* **Sekarang kita konfigurasi Static Routing-nya. Berikut konfigurasi dari Router1**

``` BASH
Router1(config)#ip route 200.200.200.0 255.255.255.0
FastEthernet 0/0 192.168.10.2
```

* **Lalu Konfigurasi Router 2**

``` BASH
Router1(config)#ip route 100.100.100.0 255.255.255.0
FastEthernet 0/1 192.168.10.1
```

> Jika sudah dikonfigurasi, lihatlah Routing Table, maka sudah ada kode S yang berartikan sudah ada Static Routing. Lalu percobaan test ping dari PC1 ke PC2 ataupun sebaliknya.

---

# Happy,  Enjoy ngoprek ~






