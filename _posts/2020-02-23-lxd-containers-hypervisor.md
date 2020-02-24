---
title: "[LXC Containers] Play with LXD Container Hypervisor"
date: 2020-02-19
categories: [ngoprek, server, cloud, lxc, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Linux Container Hypervisor (LXD dibaca lex-dee)

Proyek LXD didirikan dan saat ini dipimpin oleh Canonical Ltd dengan kontribusi dari berbagai perusahaan lain dan kontributor individu.

LXD merupakan management sistem container yang lebih user experience. LXD menawarkan penggunakaan mirip dengan teknologi virtual machine tetapi menggunakan Linux container sebagai gantinya.

Sebelumnya kita sudah membahas LXC, nah si LXD ini murapakan next level nya dari LXC yang dimana dengan LXD lebih mudah memanagement container dan di design berbasiskan daemon dan berkomunikasi melalui REST API.

Jika LXC menggunakan sebuah template untuk memabangin containernya berbeda dengan LXD yang menggunakan image dan ukuran nya lebih kecil di bandingkan template yang dimiliki LXC.

Some of the biggest features of LXD are:

- Secure by design (unprivileged containers, resource restrictions and much more)
- Scalable (from containers on your laptop to thousand of compute nodes)
- Intuitive (simple, clear API and crisp command line experience)
- Image based (with a wide variety of Linux distributions published daily)
- Support for Cross-host container and image transfer (including live migration with CRIU)
- Advanced resource control (cpu, memory, network I/O, block I/O, disk usage and kernel resources)
- Device passthrough (USB, GPU, unix character and block devices, NICs, disks and paths)
- Network management (bridge creation and configuration, cross-host tunnels, ...)
- Storage management (support for multiple storage backends, storage pools and storage volumes)

### Instalasi LXC

```shell
sudo add-apt-repository ppa:ubuntu-lxc/lxd-stable
sudo apt-get update
sudo apt-get install lxd -y 
```

Aktifkan Services nya dengan systemctl
```
sudo systemctl enable lxd
sudo systemctl start lxd
sudo systemctl status lxd
```

### Create group lxd for user *without premission sudo 
```
getent group lxd
sudo gpasswd -a usermu lxd
getent group lxd 
groups
```
### Inisialisasi
```
lxc init
```
```
ubuntu@server:~$ lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: dir
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```

### Cek versi 
```
lxc version
```
```
ubuntu@server:~$ lxc version 
Client version: 3.0.3
Server version: 3.0.3
```

### Cek repository registry container LXC & Images
```
lxc remote list
lxc image list 
lxc image list images: (List images on LXC)
```

### Spesific
```
lxc image list images:centos
```

### Create container & delete
```
lxc launch ubuntu:18.04 namacontainer
lxc list
lxc stop namacontainer
lxc start namacontainer
lxc delete namacontainer --force (Menghapus secara paksa )
```
```
ubuntu@server:~$ lxc list
+--------------+---------+-----------------------+----------------------------------------------+------------+-----------+
|     NAME     |  STATE  |         IPV4          |                     IPV6                     |    TYPE    | SNAPSHOTS |
+--------------+---------+-----------------------+----------------------------------------------+------------+-----------+
| konteubuntu2 | RUNNING | 10.135.190.157 (eth0) | fd42:5b2:d003:1fd4:216:3eff:fe2c:b4d8 (eth0) | PERSISTENT | 0         |
+--------------+---------+-----------------------+----------------------------------------------+------------+-----------+
| mykonte      | RUNNING |                       | fd42:5b2:d003:1fd4:216:3eff:fe84:394 (eth0)  | PERSISTENT | 0         |
+--------------+---------+-----------------------+----------------------------------------------+------------+-----------+
```


### Move container to other host
```
lxc stop namacontainer
lxc move namacontainer mykonte
lxc start mykonte
```

### Login to container
```
lxc exec namacontainer bash
free -m
```
```
ubuntu@server:~$ lxc exec konteubuntu2 bash
root@konteubuntu2:~# free -m
              total        used        free      shared  buff/cache   available
Mem:           1993          34        1771           0         187        1959
Swap:             0           0           0
root@konteubuntu2:~# 
```

### Login container as a user
```
lxc exec namacontainer su - ubuntu
```
### Ping another container
```
ping mykonte
```
```
root@konteubuntu2:~# ping -c 3 mykonte
PING mykonte(mykonte.lxd (fd42:5b2:d003:1fd4:216:3eff:fe84:394)) 56 data bytes
64 bytes from mykonte.lxd (fd42:5b2:d003:1fd4:216:3eff:fe84:394): icmp_seq=1 ttl=64 time=0.054 ms
64 bytes from mykonte.lxd (fd42:5b2:d003:1fd4:216:3eff:fe84:394): icmp_seq=2 ttl=64 time=0.110 ms
64 bytes from mykonte.lxd (fd42:5b2:d003:1fd4:216:3eff:fe84:394): icmp_seq=3 ttl=64 time=0.126 ms

--- mykonte ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 0.054/0.096/0.126/0.032 ms
```

### Cek kernel, container akan memakai kernel dari Host
```
uname -r 
```
```
ubuntu@server:~$ uname -r
4.15.0-88-generic
ubuntu@server:~$ lxc exec konteubuntu2 bash 
root@konteubuntu2:~# uname -r
4.15.0-88-generic
```

### Cek config on container
```
lxc config show namacontainer
lxc info namacontainer
```

### Cek pid
```
pstree -p PIDContainermu
```
```
ubuntu@server:~$ lxc info konteubuntu2 | grep Pid
Pid: 3009
ubuntu@server:~$ pstree -p 3009
systemd(3009)─┬─accounts-daemon(3369)─┬─{accounts-daemon}(3379)
              │                       └─{accounts-daemon}(3401)
              ├─agetty(3425)
              ├─atd(3404)
              ├─cron(3372)
              ├─dbus-daemon(3377)
              ├─networkd-dispat(3376)───{networkd-dispat}(3517)
              ├─polkitd(3426)─┬─{polkitd}(3431)
              │               └─{polkitd}(3433)
              ├─rsyslogd(3375)─┬─{rsyslogd}(3392)
              │                └─{rsyslogd}(3393)
              ├─sshd(3427)
              ├─systemd-journal(3108)
              ├─systemd-logind(3374)
              ├─systemd-network(3275)
              ├─systemd-resolve(3294)
              ├─systemd-udevd(3110)
              └─unattended-upgr(3445)───{unattended-upgr}(3549)
```

### Cek & create,Edit & using Profile
Nah di penguaan container ini kita bisa meng custom profile untuk setiap containernya, fungsi dari profile tersebut yaitu berisikan config atau seperti profile policy untuk container.


```
lxc profile list
lxc profile show default
lxc profile copy default custom
lxc profile edit custom
```

Edit di bagian line config : {}
```
config:
  limits.memory: 512MB
```

### Perbedaan
```
ubuntu@server:~$ lxc profile show default 
config: {}
description: Default LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: contebr0
    type: nic
  root:
    path: /
    pool: palo
    type: disk
name: default
used_by:
- /1.0/containers/mykonte
- /1.0/containers/konteubuntu2
```

Profile yang telah di edit, disini kita limit memory container yang menggunakan profile custom hanya bisa mengunakan memory 512MB
```
ubuntu@server:~$ lxc profile show custom 
config:
  limits.memory: 512MB
description: Default LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: contebr0
    type: nic
  root:
    path: /
    pool: palo
    type: disk
name: custom
used_by: []
```

Tested kita create container baru dengan profile custom
```
lxc launch ubuntu:18.04 mykonte2 --profile custom
lxc exec mykonte2 bash
```

```
ubuntu@server:~$ lxc exec mykonte2 bash
root@mykonte2:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            488          37         333           0         117         451
Swap:             0           0           0
root@mykonte2:~# 
```
> terlihat penggunakan totaly memory dari container hanya sampai 512MB

### Set limit Container langsung
```
lxc config set mykonte limits.memory 512MB
lxc config set mykonte limits.cpu 1
lxc config set mykonte boot.autostart 1
lxc config show | less
```

### File push & Pull
#### PUSH
```
echo hellow world > myfile
lxc file push myfile myvm/root/
lxc exec mmyvm bash
ls
```
#### PULL
```
lxc file pull myvm/root/myfile .
ls
```
### Snapshot file & restored
#### Membuat 5 folder untuk test backup & restored
```
lxc exec namacontainer bash
for i in $(seq 5); do mkdir $i; done
```
```
ubuntu@server:~$ lxc exec mykonte bash 
root@konteubuntu:~# for i in $(seq 5); do mkdir $i; done
root@konteubuntu:~# ls
1  2  3  4  5
```
Lakukan snapshot
```
lxc snapshot mykonte snap1
lxc list
```
```
ubuntu@server:~$ lxc snapshot mykonte snap1
ubuntu@server:~$ lxc list
+----------+---------+-----------------------+---------------------------------------------+------------+-----------+
|   NAME   |  STATE  |         IPV4          |                    IPV6                     |    TYPE    | SNAPSHOTS |
+----------+---------+-----------------------+---------------------------------------------+------------+-----------+
| mykonte  | RUNNING | 10.135.190.237 (eth0) | fd42:5b2:d003:1fd4:216:3eff:fe84:394 (eth0) | PERSISTENT | 1         |
+----------+---------+-----------------------+---------------------------------------------+------------+-----------+
| mykonte2 | RUNNING | 10.135.190.201 (eth0) | fd42:5b2:d003:1fd4:216:3eff:fe4f:2ba (eth0) | PERSISTENT | 0         |
+----------+---------+-----------------------+---------------------------------------------+------------+-----------+
```
> Dan disitu terlihat SNAPSHOTS tersedia 1, sekarang kita test dengan menghapus file yang ada di container 'mykonte' lalu me restore nya. 
```
ubuntu@server:~$ lxc exec mykonte bash
root@konteubuntu:~# ls
1  2  3  4  5
root@konteubuntu:~# rm -rf *
```

### Restored maka file yang di hapus tadi akan kembali
```
lxc restore mykonte snap1
```
```
ubuntu@server:~$ lxc restore mykonte snap1
ubuntu@server:~$ lxc exec mykonte bash
root@konteubuntu:~# ls
1  2  3  4  5
```

**Referensi**
* [https://lxd.readthedocs.io/en/stable-3.0/containers/](https://lxd.readthedocs.io/en/stable-3.0/containers/)
* [https://linuxcontainers.org/lxd/introduction/](https://linuxcontainers.org/lxd/introduction/)

# Happy,  Enjoy Ngoprek ~