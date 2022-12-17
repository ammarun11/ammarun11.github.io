---
title: "[Ceph] Troubleshoot Ceph PG Incomplete"
date: 2022-12-17
categories: [ngoprek, server, cloud, ceph, storage]
tags:
  - Jekyll
  - update
---
## Investigating

You can check globally using `ceph -s` and you can see there is incomplete or inactive but for more detail you can see by using this command. On this command result osd 100 is the primary of pg `19.64`
```
ceph pg dump_stuck inactive

output:
ok
PG_STAT  STATE       UP           UP_PRIMARY  ACTING       ACTING_PRIMARY
19.64    incomplete  [100,85,89]         100  [100,85,89]             100
```

Or you can use this command for more details
```
ceph pg ls incomplete

output:
PG     OBJECTS  DEGRADED  MISPLACED  UNFOUND  BYTES       OMAP_BYTES*  OMAP_KEYS*  LOG   STATE       SINCE  VERSION     REPORTED      UP               ACTING           SCRUB_STAMP                      DEEP_SCRUB_STAMP
19.64      297         0          0        0  1245708288            0           0  1328  incomplete    24m  69498'1328  204713:96240  [100,85,89]p100  [100,85,89]p100  2022-04-04T04:47:37.785708+0000  2021-10-21T06:24:54.701161+0000
```

You can also check where is the PG lives
```
root@juju-bf8473-19-lxd-0:~# ceph osd lspools
1 default.rgw.buckets.data
2 default.rgw.control
3 default.rgw.data.root
4 default.rgw.gc
5 default.rgw.log
6 default.rgw.intent-log
7 default.rgw.meta
8 default.rgw.usage
9 default.rgw.users.keys
10 default.rgw.users.email
11 default.rgw.users.swift
12 default.rgw.users.uid
13 default.rgw.buckets.extra
14 default.rgw.buckets.index
15 .rgw.root
16 gnocchi
17 cinder-ceph
18 glance
19 scbench
20 cinder-ceph-ssd
21 device_health_metrics
22 testbench
23 rbd-kubernetes
24 k8s-uat
```
>PG start with pool id follow by other numbers, for example in this case pg 19.64 lives on sbench

## First Workaround
Set flag on ceph cluster
```
sudo ceph osd set noout
sudo ceph osd set norebalance
sudo ceph osd set nobackfill
sudo ceph osd set norecover
```

Check the pg query
```
ceph pg {pg.id} query | grep -Ew 'state|peer'
```

Try restart the osd one by one
```
ceph osd find {osd.id}
systemctl restart ceph-osd@{osd.id}
```

After that check the cluster status

## Second Workaround
>If the first workaround still not helping try use this but you must aware that there is possibility of data will be loss and you still need to set the ceph flags

Stop the primary osd daemon
```
systemctl stop ceph-osd@{osd.id}
```

After that mark complete the pg
```
ceph-objectstore-tool --data-path /var/lib/ceph/osd/ceph-{osd.id} --op mark-complete --pgid {pg.id}
```

Then you can start the osd daemon again and check the ceph cluster
```
systemctl start ceph-osd@{osd.id}
```

## References
- [https://www.oreilly.com](https://www.oreilly.com/library/view/mastering-proxmox-/9781788397605/42d80c67-10aa-4cf2-8812-e38c861cdc5d.xhtml)
- [https://lists.ceph.io](https://lists.ceph.io/hyperkitty/list/ceph-users@ceph.io/thread/7F7J6TGSKOP52GB3IL4XNYIZZQA7QFV4/)
- [https://medium.com/opsops](https://medium.com/opsops/recovering-ceph-from-reduced-data-availability-3-pgs-inactive-3-pgs-incomplete-b97cbcb4b5a1)
# Happy,  Enjoy Ngoprek ~
