---
title: "[Kubernetes The Hard Way] POD Network Routes - Part 8"
date: 2020-04-14
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## POD Network Routes (Kubernetes The Hardway)

Lanjutkeunn, Lab part 8
Pods scheduled to a node receive an IP address from the node's Pod CIDR range. At this point pods can not communicate with other pods running on different nodes due to missing network routes.

In this lab you will create a route for each worker node that maps the node's Pod CIDR range to the node's internal IP address.

## Running commands in parallel with tmux
tmux can be used to run commands on multiple compute instances at the same time. See the Running commands in [parallel with tmux](https://blog.amjith.com/synchronize-panes-in-tmux) section in the Prerequisites lab.

Pantau pods yang akan dibuat nanti,
```s
watch kubectl get pods -o wide
```

Membuat pods busybox untuk uji coba network nya sebelum di routing antar POD_CIDR,
```s
kubectl run myshell -it --rm --image busybox -- sh
```
```s
kubectl run myshell2 -it --rm --image busybox -- sh
```
> Output hasil watch kubectl 

```s
Every 2.0s: kubectl get pods -o wide                                palo-server0: Tue Apr 14 02:10:35 2020:18
NAME                        READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   
myshell-58f4c7bc6b-szbrd    1/1     Running   0          6s      10.200.2.4   worker-2   <none>
myshell2-6f8cc4f49d-sff8w   1/1     Running   0          3m40s   10.200.0.2   worker-0   <none>
```

Lalu test ping antar IP beda POD tersebut maka 100% akan packet loss.
test myshell > myshell2
```s
If you don't see a command prompt, try pressing enter.
/ # 
/ # ping 10.200.0.2 -c 3
PING 10.200.0.2 (10.200.0.2): 56 data bytes

--- 10.200.0.2 ping statistics ---
3 packets transmitted, 0 packets received, 100% packet loss
/ # 
```

Close balik ke HOST OS

Routing Time 
```s
route add -net 10.200.0.0 netmask 255.255.255.0 gw 10.240.0.79
route add -net 10.200.1.0 netmask 255.255.255.0 gw 10.240.0.245
route add -net 10.200.2.0 netmask 255.255.255.0 gw 10.240.0.175
```

Cek
```s
ip route show
```
> Output 

```s
root@palo-server0:~# ip route show
default via 10.0.0.1 dev eth0
10.0.0.0/24 dev eth0 proto kernel scope link src 10.0.0.4
10.200.0.0/24 via 10.240.0.79 dev lxdbr0
10.200.1.0/24 via 10.240.0.245 dev lxdbr0
10.200.2.0/24 via 10.240.0.175 dev lxdbr0
10.240.0.0/24 dev lxdbr0 proto kernel scope link src 10.240.0.1
168.63.129.16 via 10.0.0.1 dev eth0
169.254.169.254 via 10.0.0.1 dev eth0
```

Membuat pods busybox untuk uji coba network nya sesudah di routing antar POD_CIDR,
```s
kubectl run myshell -it --rm --image busybox -- sh
```
```s
kubectl run myshell2 -it --rm --image busybox -- sh
```
> output

```s
Every 2.0s: kubectl get pods -o wide                            palo-server0: Tue Apr 14 02:18:44 2020

NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE 
myshell-58f4c7bc6b-6r5qh    1/1     Running   0          91s   10.200.1.3   worker-1   <none>
myshell2-6f8cc4f49d-qxwrm   1/1     Running   0          68s   10.200.2.5   worker-2   <none>
```

Verification 

```s
/ # ping 10.200.2.5
PING 10.200.2.5 (10.200.2.5): 56 data bytes
64 bytes from 10.200.2.5: seq=0 ttl=61 time=0.142 ms
64 bytes from 10.200.2.5: seq=1 ttl=61 time=0.142 ms
64 bytes from 10.200.2.5: seq=2 ttl=61 time=0.097 ms
64 bytes from 10.200.2.5: seq=3 ttl=61 time=0.099 ms
^C
--- 10.200.2.5 ping statistics ---
4 packets transmitted, 4 packets received, 0% packet loss
round-trip min/avg/max = 0.097/0.120/0.142 ms
/ # 
```


DONE Sekarang POD tiap Nodes terhubung.



Lanjutnye: [Deploying the DNS Cluster Add-on - FinalPart 9](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-part9/)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
