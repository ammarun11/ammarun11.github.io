---
title: "[LXC Containers] Installation lab at Ubuntu"
date: 2020-02-19
categories: [ngoprek, server, cloud, lxc, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Tentang Linux Containers (LXC dibaca lex-cee)

Linux containers merupakan teknologi di level virtualisasi yang sangat ringan. tidak seperti beberapa teknologi level virtualisasi yang lainnya dimana membutuhkan hardware/resource nya sendiri.

tidak ada hardware/resource tersendiri untuk membuat container, karena container berbagi kernel yang membuat resources seperti CPU,RAM, Hard disk, dan Network akan mengikuti seperti Host OS dari container tersebut.

lxc di bangun dari yang namanya kernel linux yaitu cgroups dan namespaces
, dimana kernel tersebutlah yang mengontrol semua sumber daya maupun services yang ada pada Linux. 

Dan hal tersebut dimanfaatkan LXC untuk membuat sistem virtualisasi yang terisolasi antar services nya dengan kernel namespaces dan untuk mengontrol resource seperti CPU,RAM dan Harddisk LXC memanfaatkan cgroups (Control Groups). [Sumber bacaan Linux containers](https://medium.com/rahasak/linux-containers-lxc-b86bba9ab117)

<img src="https://www.opennebula.org/wp-content/uploads/2019/05/hyp2.png">

Virtualization vs Container via [opennebula.org](https://opennebula.org/lxc-containers-for-opennebula)

### Instalasi LXC

```shell
sudo apt-get install lxc lxctl lxc-templates
```

### Cek Konfigurasi
```shell
sudo lxc-checkconfig
```
Hasilnya
```shell
Kernel configuration not found at /proc/config.gz; searching...
Kernel configuration found at /boot/config-4.4.0-38-generic
--- Namespaces ---
Namespaces: enabled
Utsname namespace: enabled
Ipc namespace: enabled
Pid namespace: enabled
User namespace: enabled
Network namespace: enabled
Multiple /dev/pts instances: enabled
--- Control groups ---
Cgroup: enabled
Cgroup clone_children flag: enabled
Cgroup device: enabled
Cgroup sched: enabled
Cgroup cpu account: enabled
Cgroup memory controller: enabled
Cgroup cpuset: enabled
--- Misc ---
Veth pair device: enabled
Macvlan: enabled
Vlan: enabled
Bridges: enabled
Advanced netfilter: enabled
CONFIG_NF_NAT_IPV4: enabled
CONFIG_NF_NAT_IPV6: enabled
CONFIG_IP_NF_TARGET_MASQUERADE: enabled
CONFIG_IP6_NF_TARGET_MASQUERADE: enabled
CONFIG_NETFILTER_XT_TARGET_CHECKSUM: enabled
FUSE (for use with lxcfs): enabled
--- Checkpoint/Restore ---
checkpoint restore: enabled
CONFIG_FHANDLE: enabled
CONFIG_EVENTFD: enabled
CONFIG_EPOLL: enabled
CONFIG_UNIX_DIAG: enabled
CONFIG_INET_DIAG: enabled
CONFIG_PACKET_DIAG: enabled
CONFIG_NETLINK_DIAG: enabled
File capabilities: enabled
Note : Before booting a new kernel, you can check its configuration
usage : CONFIG=/path/to/config /usr/bin/lxc-checkconfig
```

### Ada beberapa template bawaan cara melihatnya
```
sudo ls /usr/share/lxc/templates/
```

Hasilnya
```
lxc-alpine     lxc-cirros    lxc-openmandriva  lxc-sparclinux
lxc-altlinux   lxc-debian    lxc-opensuse      lxc-sshd
lxc-archlinux  lxc-download  lxc-oracle        lxc-ubuntu
lxc-busybox    lxc-fedora    lxc-plamo	     lxc-ubuntu-cloud
lxc-centos     lxc-gentoo    lxc-slackware
```

Untuk mengunduh template nya akan membutuhkan beberapa waktu tergantung size dan kecepatan internet kita.

### command
```
sudo lxc-create -n <container-name> -t <template>
sudo lxc-create -n mongodb-server -t ubuntu
```
mongodb-server sebagai nama container dengan menggunakan template  ubuntu
list tempalte tersedia di folder /usr/share/lxc/templates

### List Containers
```
sudo lxc-ls --fancy
```
hasilnya
```
NAME            STATE    IPV4       IPV6  GROUPS  AUTOSTART  
-----------------------------------------------------
mongodb-server  RUNNING  10.0.3.221  -     -       NO
```
### Start Container
```
sudo lxc-start -n <container-name> -d
sudo lxc-start -n mongodb-server -d
```
> -d artinya container akan jalan otomatis saat host os nya pun dinyalakan

### Melihat Info Container
```
sudo lxc-info -n <container-name>
sudo lxc-info -n mongodb-server
```
hasilnya
```
Name:           mongodb-server
State:          RUNNING
PID:            20906
IP:             10.0.3.221
CPU use:        1.15 seconds
BlkIO use:      23.55 MiB
Memory use:     36.46 MiB
KMem use:       0 bytes
Link:           vethHPCB07
 TX bytes:      1.30 KiB
 RX bytes:      3.84 KiB
 Total bytes:   5.14 KiB
```
### Masuk ke console/bash container
```
sudo lxc-console -n mongodb-server
```

### Stop Container
Dari host machine
```
sudo lxc-stop -n <container-name>
sudo lxc-stop -n mongodb-server
```

### Delete Container
```
sudo lxc-destroy -n <container-name>
sudo lxc-destroy -n mongodb-server
```

**Referensi**
* [https://lzone.de/cheat-sheet/LXC](https://lzone.de/cheat-sheet/LXC)
* [https://medium.com/rahasak/linux-containers-lxc-e539a8d95ead](https://medium.com/rahasak/linux-containers-lxc-e539a8d95ead)
* [https://medium.com/rahasak/containers-c0bdc5898585](https://medium.com/rahasak/containers-c0bdc5898585)

# Happy,  Enjoy Ngoprek ~