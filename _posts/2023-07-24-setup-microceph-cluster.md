---
title: "[Ceph] Setup Ceph cluster with Microceph"
date: 2023-07-24
categories: [ngoprek, server, cloud, ceph, openstack]
tags:
  - Jekyll
  - update
---

## Brief
MicroCeph is a lightweight way of deploying a Ceph cluster using just a few commands, resulting in reliable and resilient distributed storage. MicroCeph is aimed at small scale private-clouds.

we're here used ceph quincy version

Check out MicroCeph's documentation at https://canonical-microceph.readthedocs-hosted.com/en/latest/.

## Nodes
```
- am-ceph1 (OSD Disk: vdb,vdc,vdd) 
- am-ceph2 (OSD Disk: vdb,vdc,vdd)
- am-ceph3 (OSD Disk: vdb,vdc,vdd)
```
## Initial Setup 
To get started install the MicroCeph snap with the following command on each node to be used in the cluster:
```
snap install microceph --channel=quincy/stable && snap refresh --hold microceph
```

Connect the microceph snap to the hardware-observe and block-devices so disks on os will knew by snap package interface:
```
snap connect microceph:hardware-observe
snap connect microceph:block-devices
```

### Then bootstrap init the cluster from the first node:
```
ssh am-ceph1
microceph init
Please choose the address MicroCeph will be listening on [default=10.20.10.68]: 10.20.14.68
Would you like to create a new MicroCeph cluster? (yes/no) [default=no]: yes
Would you like to add additional machines to the cluster? [default=empty]: am-ceph2
eyJuYW1lIjoiYW0tY2VwaDIiLCJzZWNyZXQiOiJjNjhmODllMTE2MTIyMWFmNzQ3MTlhNzQ3MDYyY2RiYWEzZmFkYTdkNjhkMTljY2E1ZmE1MDI3NTkxM2QyZGI2IiwiZmluZ2VycHJpbnQiOiI4OGMzNzFhNDJlMGY3MDIxZTgyYThhM2RlZjJiZDQxMDFiODQwYmRhNGM3YWYwOTQ0NTBiZjc0OTMzM2QzMWVmIiwiam9pbl9hZGRyZXNzZXMiOlsiMTAuMjAuMTAuNjg6NzQ0MyJdfQ==
# move to am-ceph2
Would you like to add additional machines to the cluster? [default=empty]: am-ceph3
eyJuYW1lIjoiYW0tY2VwaDIiLCJzZWNyZXQiOiJjMjcyMTU2M2Y4YTVhYTRiOWJlYzM5MTMxZmJiMzY1ZWIzN2MyNzJiNzJhZmI2MTQ2NjBlN2M1ZGQ0Y2U0ZTc2IiwiZmluZ2VycHJpbnQiOiI4OGMzNzFhNDJlMGY3MDIxZTgyYThhM2RlZjJiZDQxMDFiODQwYmRhNGM3YWYwOTQ0NTBiZjc0OTMzM2QzMWVmIiwiam9pbl9hZGRyZXNzZXMiOlsiMTAuMjAuMTAuNzA6NzQ0MyIsIjEwLjIwLjEwLjY4Ojc0NDMiXX0
# move to am-ceph3
# skip enter path disk
Show local disks available, option to directly enter path ? [default=empty]: 
```

```
ssh am-ceph2
microceph init
Please choose the address MicroCeph will be listening on [default=10.20.10.69]: 10.20.14.69
Would you like to create a new MicroCeph cluster? (yes/no) [default=no]: no
Please enter your join token: eyJuYW1lIjoiYW0tY2VwaDIiLCJzZWNyZXQiOiJjNjhmODllMTE2MTIyMWFmNzQ3MTlhNzQ3MDYyY2RiYWEzZmFkYTdkNjhkMTljY2E1ZmE1MDI3NTkxM2QyZGI2IiwiZmluZ2VycHJpbnQiOiI4OGMzNzFhNDJlMGY3MDIxZTgyYThhM2RlZjJiZDQxMDFiODQwYmRhNGM3YWYwOTQ0NTBiZjc0OTMzM2QzMWVmIiwiam9pbl9hZGRyZXNzZXMiOlsiMTAuMjAuMTAuNjg6NzQ0MyJdfQ==
# skip enter path disk
Show local disks available, option to directly enter path ? [default=empty]: 
```

```
ssh am-ceph3
microceph init
Please choose the address MicroCeph will be listening on [default=10.20.10.70]: 10.20.14.70
Would you like to create a new MicroCeph cluster? (yes/no) [default=no]: no
Please enter your join token: eyJuYW1lIjoiYW0tY2VwaDIiLCJzZWNyZXQiOiJjMjcyMTU2M2Y4YTVhYTRiOWJlYzM5MTMxZmJiMzY1ZWIzN2MyNzJiNzJhZmI2MTQ2NjBlN2M1ZGQ0Y2U0ZTc2IiwiZmluZ2VycHJpbnQiOiI4OGMzNzFhNDJlMGY3MDIxZTgyYThhM2RlZjJiZDQxMDFiODQwYmRhNGM3YWYwOTQ0NTBiZjc0OTMzM2QzMWVmIiwiam9pbl9hZGRyZXNzZXMiOlsiMTAuMjAuMTAuNzA6NzQ0MyIsIjEwLjIwLjEwLjY4Ojc0NDMiXX0
# skip enter path disk
Show local disks available, option to directly enter path ? [default=empty]: 
```

Check the cluster status with the following command:
```
microceph.ceph status
or
ceph -s
```

## Adding OSDs 

### Exec on all node (am-ceph1, am-ceph2, am-ceph3)
```
microceph disk add /dev/vdb --wipe
microceph disk add /dev/vdc --wipe
microceph disk add /dev/vdd --wipe
```

Check the cluster status with the following command:
```
# microceph.ceph status
or
# ceph -s

  cluster:
    id:     a7bdc86e-5cce-4697-96cc-c4b29a5a50e2
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum am-ceph1,am-ceph2,am-ceph3 (age 9m)
    mgr: am-ceph1(active, since 9m), standbys: am-ceph2, am-ceph3
    osd: 9 osds: 9 up (since 9m), 9 in (since 9m)

  data:
    pools:   1 pools, 1 pgs
    objects: 0 objects, 15 GiB
    usage:   1 GiB used, 180 GiB / 180 GiB avail
    pgs:     1 active+clean
```

## Operational Ceph Block Storage
### Create block device image
#### Create pool
```
ceph osd pool create pool-test1 8 
rbd pool init pool-test1
rbd create pool-test1/block0 --size 1028
```

#### check block rbd that are contained in pool-test1
```
rbd list pool-test1
rbd info pool-test1/block0

rbd image 'block0':
        size 1.0 GiB in 257 objects
        order 22 (4 MiB objects)
        snapshot_count: 0
        id: 3fd3bcd87005b
        block_name_prefix: rbd_data.3fd3bcd87005b
        format: 2
        features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
        op_features:
        flags:
        create_timestamp: Mon Jul 24 22:32:13 2023
        access_timestamp: Mon Jul 24 22:32:13 2023
        modify_timestamp: Mon Jul 24 22:32:13 2023
```

#### Maping image to block device
##### ceph-mon filled by node-mon on am-ceph1
```
rbd map pool-test1/block0 -k /etc/ceph/ceph.client.admin.keyring
rbd showmapped

id  pool        namespace  image   snap  device
0   pool-test1             block0  -     /dev/rbd0
```
#### Format filesystem
```
mkfs.xfs /dev/rbd0
```

#### Mount block device
```
mkdir /mnt/block0
mount /dev/rbd0 /mnt/block0
df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            7.9G     0  7.9G   0% /dev
tmpfs           1.6G  1.3M  1.6G   1% /run
/dev/vda1        39G  2.6G   37G   7% /
---
/dev/rbd0      1018M   40M  979M   4% /mnt/block0
```

