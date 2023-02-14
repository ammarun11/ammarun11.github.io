---
title: "[Openstack] Resize root partition in VM Openstack"
date: 2023-02-14
categories: [ngoprek, server, cloud, openstack, instance]
tags:
  - Jekyll
  - update
---

## Guide Resize Mounted Volume in VM Openstack

Eventually we need to resize volume root /dev/sda, which is the root partition; the problem is root partition cannot detach, so how we can resize the size of the root partition? Here's your answer dude

### Resize From OpenStack Side
```bash
## set the status to available to a volume
openstack volume set --state available {volume id}

## resize the volume
openstack volume set --size 40 {volume id}

## check size and status again
openstack volume show {volume id}

## set status to in-use again
openstack volume set --state in-use {volume id}
```

### Resize Volume from VM Side
```bash
## Grow part (Contoh akan resize root partisi)
sudo growpart /dev/vda 1

## Check apakah sudah grow sesuai volume baru
lsblk

## Resize ‘/’ partition to fill all space
sudo resize2fs /dev/vda1

## (option) If your filesystem is XFS, it can be grown while mounted using the xfs_growfs command:

sudo xfs_growfs /

## Verify 
sudo df -H
```

if `no space left on the block device`, running command:
```
$ sudo mount -o size=10M,rw,nodev,nosuid -t tmpfs tmpfs /tmp
```

After the process unmount `/tmp` 
```
sudo umount /tmp
```

#### IF LVM Volume  :

#### 1. Refresh LVM VG (LVM from lsblk)
```
vgchange --refresh
```
#### 2. Resize pv
```
pvresize /dev/sdX
```
#### 3. Extend lv
```
lvextend -l +100%FREE /dev/<vg_name>/<lv_name>
```
#### 4. Mount /dev/<vg_name>/<lv_name>
```
mount /dev/<vg_name>/<lv_name> /mnt
```
#### 5. Filesystem grow 
###### - Jika filesystem XFS 
```
xfs_growfs /dev/<vg_name>/<lv_name>
```
###### - Jika filesystem . . . .  Lanjutkan
```

