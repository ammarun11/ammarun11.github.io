---
title: "[Proxmox] Preview Setup Cluster In Proxmox"
date: 2023-05-18
categories: [ngoprek, server, cloud, proxmox, instance]
tags:
  - Jekyll
  - update
---

## Nodes
```
- pve-proxmox-1
- pve-proxmox-2
- pve-proxmox-3
```

## Prerequites
- Each node (pve-proxmox-1, pve-proxmox-2, and pve-proxmox-3) has proxmox VE installed
- Each node can be accessed by its GUI
- Each node can be connected to each other
- Each node has chrony installed

## Setup
### Via Dashboard
- Login to one of the proxmox dashboards (pve-proxmox-1)
- Click Datacenter Section -> Cluster -> Create Cluster

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/vr4yQXXDSCxJmsPf-image-1658119426171.png)](https://book.btech.id/uploads/images/gallery/2022-07/vr4yQXXDSCxJmsPf-image-1658119426171.png)
- Enter the cluster name (cluster-prox) and select the network (172.168.1.101) to be used, and click Create

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/NKlvwUMnozyeWWwB-image-1658119442809.png)](https://book.btech.id/uploads/images/gallery/2022-07/NKlvwUMnozyeWWwB-image-1658119442809.png)

- On Cluster Information, click Join Information, then click Copy Information

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/9naqYK6rRJUCi4ds-image-1658119458550.png)](https://book.btech.id/uploads/images/gallery/2022-07/9naqYK6rRJUCi4ds-image-1658119458550.png)

- Login to other node proxmox dashboard (pve-proxmox-2 and pve-proxmox-3)
- Click Datacenter Section -> Cluster -> Join Cluster

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/v6sYnWWomeaiCG4j-image-1658119467399.png)](https://book.btech.id/uploads/images/gallery/2022-07/v6sYnWWomeaiCG4j-image-1658119467399.png)

- Paste the encoded cluster information that was copied earlier, then enter the password from the Proxmox node, click Join
- Wait until the joining cluster process is complete
- Verify, Enter one of the Proxmox node dashboards, Click the Datacenter Section -> Cluster

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/7TnQ8TLCJhEFgpxD-image-1658119476933.png)](https://book.btech.id/uploads/images/gallery/2022-07/7TnQ8TLCJhEFgpxD-image-1658119476933.png)

### With Command-Line
- Create cluster
```
pvecm create cluster-prox
```
- Cek Status
```
pvecm status
```
- add another nodes to cluster
```
# login to server another nodes
pvecm add {ip-address-node-new}
pvecm add 172.168.1.101
```
- Verify
```
root@pve-proxmox-1:~# pvecm status
Cluster information
-------------------
Name:             cluster-prox
Config Version:   3
Transport:        knet
Secure auth:      on

Quorum information
------------------
Date:             Sat Jul 16 09:16:29 2022
Quorum provider:  corosync_votequorum
Nodes:            3
Node ID:          0x00000001
Ring ID:          1.47
Quorate:          Yes

Votequorum information
----------------------
Expected votes:   3
Highest expected: 3
Total votes:      3
Quorum:           2  
Flags:            Quorate 

Membership information
----------------------
    Nodeid      Votes Name
0x00000001          1 172.168.1.101 (local)
0x00000002          1 172.168.1.102
0x00000003          1 172.168.1.103
```

[![](https://book.btech.id/uploads/images/gallery/2022-07/scaled-1680-/tl9xtkwI4Sdy1gyn-image-1658119510943.png)](https://book.btech.id/uploads/images/gallery/2022-07/tl9xtkwI4Sdy1gyn-image-1658119510943.png)