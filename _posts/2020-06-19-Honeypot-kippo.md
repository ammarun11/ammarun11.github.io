---
title: "[Security] Deployment Kippo Honeypot On Linux Ubuntu"
date: 2020-06-19
categories: [ngoprek, server, cloud, security]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Deployment Kippo Honeypot On Linux Ubuntu


## Honeypot
Honeypot merupakan salah satu solusi yang dapat diberikan
karena honeypot merupakan sebuah sistem umpan atau aplikasi
simulasi yang mensimulasikan seluruh jaringan untuk memikat
penyerang dengan menyamarkan diri sebagai sistem yang
rentan

implementasi
Kita bakal menggunakan metode honeypot medium interaction memanfaatkan tools aplikasi, yaitu Kippo yang dapat membuat layanan palsu sebagai target serangan dan mencatat aktivitas yang
dianggap dapat membahayakan sistem dan jaringan, dengan
adanya interaksi lebih lanjut ketika penyerang melakukan
mengeksploitasi dan masuk dalam honeypot yang bisa kita trace log nya.


## Requirements
- Ubuntu 16.04
- [Cara Install dari repo resminyee klik disini yee ](https://github.com/desaster/kippo/wiki/Running-Kippo)


## Konfigurasi

Saya sarankan temen temen ikuti cara instalasi di atas jdi kita langsung ke konfig nya aja, 
lalu kita buat user biasa dulu karena progral kippo berjalanan di non-root user

By default layanan services ssh berjalan di port 22, dan si honeypot ini akan memakai port tersebut untuk menjebak si penyerang maka dari itu kita harus mengganti port ssh aslinya dari port 22 ke yang lain, cara merubahkan ke file,

```s
sudo nano /etc/ssh/sshd_config

# Port 22 ubah ke 1945
Port 1945
```

Setelah done restart service ssh:
```s
$ sudo service ssh restart
```

Buat user non-root untuk honeypot kippo
```s
sudo adduser kippo
```
Tambahkan user kippo ke priveleged ALL
```s
sudo nano /etc/sudoers

# User privilege specification
root    ALL=(ALL:ALL) ALL
kippo   ALL=(ALL:ALL) ALL
```
Sekarang pindah ke user honeypot untuk instalasi Kippo. kita jalankan kippo di virtualenv yang kita buat dari fitur python 

```s
su kippo
$ cd ~
$ git clone https://github.com/desaster/kippo.git
$ cd kippo
$ virtualenv env
$ . ./env/bin/activate
```

Your shell prompt will now display the currently active virtualenv:
```s
(env)$
```

Install the required python packages
````s
(env)$ pip install -U pip
(env)$ pip install setuptools==44.0.0
(env)$ pip install twisted==15.1.0
(env)$ pip install pyasn1
(env)$ pip install pycrypto
````
15.1.0 is the last twisted version that kippo works with

Kippo sendiri secara default berjalan di port 2222. Hal tersebut dikarenakan pada dasarnya Kippo hanya bisa dijalankan oleh non-root user sementara non-root user sendiri tidak diijinkan membuka port dibawah 1024.

Pertama, kita edit edit dulu di bagian kippo.cfg .
```s
(env)$ cp kippo.cfg.dist kippo.cfg
(env)$ nano kippo.cfg
```

Edit file config kippo nya
```s
ssh_port = 4672
hostname = Finance
log_path = log
download_path = dl
download_limit_size = 1000000
```

Untuk konfigurasi lain silahkan dibaca baca sendiri keterangannya. Disini saya menggunakan port 4672 untuk ssh honeypot. Download path adalah lokasi dimana file yang didownload oleh attacker misalnya dengan wget akan disimpan. Sehingga saat hacker mengirim exploit atau malware tidak akan bisa dieksekusi. Download limit digunakan untuk membatasi besarnya download dalam ukuran bytes. Defaultnya tidak dibatasi dan menurut saya tidak baik juga karena bisa saja attacker mengirim file dalam jumlah besar yang menyebabkan server menjadi penuh. Hostname adalah nama komputer yang akan ditampilkan kepada attacker.
[Sumber Linuxsec.org](https://www.linuxsec.org/2017/05/kippo-ssh-honeypot.html)

Selanjutnya adalah melakukan redirrect dari port 22 ke port 4672 dengan iptables.
```s
(env)$ sudo iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-port 4672
(env)$ sudo iptables -I INPUT 5 -p tcp --dport 4672 -j ACCEPT
```

Perintah pertama digunakan untuk meredirect port 22 ke 4672 sehingga saat attacker melakukan akses ke port default ssh tetap bisa tersambung. Sementara perintah kedua digunakan untuk mengijinkan koneksi di port 4672. -I INPUT 5 maksudnya adalah rule uptables tersebut akan diletakkan pada baris kelima. Hal ini saya lakukan untuk menghindari rule baru tersimpan di bawah rule DROP. [Sumber Linuxsec.org](https://www.linuxsec.org/2017/05/kippo-ssh-honeypot.html)

Sampai disini Kippo sudah siap dijalankan. by default Kippo memberi password honeypot atau server palsunya 123456

Starting kippo in foreground jadi secara interactive *saya saranin sih pake ini:

Activate the virtualenv we created before:
```s
(env)$ twistd -n -y kippo.tac
```

OR 

Starting kippo in background:
Dan command ini untuk jalanain honeypot kippi di background services
```s
(env)$ ./start.sh env
```

## HASIL SIMULASI 
Di sisi Server akan menjalankan Kippo honeypot secara Interactive
```s
(env) kippo@ubuntu:~/kippo$ twistd -n -y kippo.tac
Generating new RSA keypair...
Done.
Generating new DSA keypair...
Done.
2020-06-19 14:10:45+0700 [-] Log opened.
2020-06-19 14:10:45+0700 [-] twistd 15.1.0 (/home/kippo/kippo/env/bin/python2 2.7.12) starting up.
2020-06-19 14:10:45+0700 [-] reactor class: twisted.internet.epollreactor.EPollReactor.
2020-06-19 14:10:45+0700 [-] HoneyPotSSHFactory starting on 4672
2020-06-19 14:10:45+0700 [-] Starting factory <kippo.core.ssh.HoneyPotSSHFactory instance at 0x7f57c76231b8>
```

Di Sisi penyerang
```s
C:\Users\cinta>ssh root@192.168.10.1 -p 22
The authenticity of host '192.168.10.1 (192.168.10.1)' can't be established.
RSA key fingerprint is SHA256:w9tOzSOJ1oIL4LW1J50Wj6Mjiv4XNYvjv1f8TOLFg4I.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.1' (RSA) to the list of known hosts.
Password:
root@Finance:~# echo "Hello World"
```

Nah disini si penyerang bisa memasukan command command untuk melakukan hacking pada server tsb namun yang dia serang merupakan server palsu yang dimana tidak akan berpengaruh terhadap server aslinya. Nah command yang di masukkan oleh si penyerang dapat ter log di server aslinya nah disini sysadmin atau netsec dapat mempelajari perilaku penyerang tsb tujuan nya untuk apa dan tools atau malware/virus apa yang digunakan untuk meneyarang server.

Tampilan log disisi Server
```s
2020-06-19 14:17:57+0700 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,1,192.168.10.2] Updating realfile to honeyfs//etc/passwd
2020-06-19 14:18:24+0700 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,1,192.168.10.2] CMD: echo "Hello World""
2020-06-19 14:18:38+0700 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,1,192.168.10.2] CMD: echo "Hello World"
2020-06-19 14:18:38+0700 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,1,192.168.10.2] Command found: echo "Hello World"
```


**Referensi**
* [https://linuxconfig.org/deployment-of-kippo-ssh-honeypot-on-ubuntu-linux](https://linuxconfig.org/deployment-of-kippo-ssh-honeypot-on-ubuntu-linux)
* [https://www.linuxsec.org/2017/05/kippo-ssh-honeypot.html](https://www.linuxsec.org/2017/05/kippo-ssh-honeypot.htmls)
* [https://github.com/desaster/kippo/wiki/Running-Kippo](https://github.com/desaster/kippo/wiki/Running-Kippos)

# Happy,  Enjoy Ngoprek ~
