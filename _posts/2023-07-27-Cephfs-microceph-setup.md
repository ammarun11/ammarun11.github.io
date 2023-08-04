---
title: "[Ceph] Setup Microceph RadosGW"
date: 2023-07-27
categories: [ngoprek, server, cloud, ceph, openstack]
tags:
  - Jekyll
  - update
---

## Brief
All native protocols on Microceph (RBD, CephFS and RGW) are supported, 
Once you have your Ceph Object Storage service up and running, you may administer the service with user management, access controls, quotas and usage tracking among other features.


## Enable RadosGW Microceph
```
microceph enable rgw

microceph status
---output
MicroCeph deployment summary:
- am-ceph1 (10.20.14.68)
  Services: mds, mgr, mon, rgw, osd
  Disks: 3
- am-ceph2 (10.20.14.69)
  Services: mds, mgr, mon, osd
  Disks: 3
- am-ceph3 (10.20.14.70)
  Services: mds, mgr, mon, osd
  Disks: 3

ceph -s
---output
  cluster:
    id:     a7bdc86e-5cce-4697-96cc-c4b29a5a50e2
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum am-ceph1,am-ceph2,am-ceph3 (age 28h)
    mgr: am-ceph1(active, since 18h), standbys: am-ceph2, am-ceph3
    osd: 9 osds: 9 up (since 26h), 9 in (since 28h)
    rgw: 1 daemon active (1 hosts, 1 zones)

  data:
    pools:   12 pools, 305 pgs
    objects: 4.56k objects, 23 GiB
    usage:   67 GiB used, 113 GiB / 180 GiB avail
    pgs:     305 active+clean
```

### Create RadosGW access user 
```
microceph.radosgw-admin key create --uid=test --key-type=s3 --access-key fooAksesKunci --secret-key fooKunciRahasia

---output
{
    "user_id": "test",
    "display_name": "testuser",
    "email": "",
    "suspended": 0,
    "max_buckets": 1000,
    "subusers": [],
    "keys": [
        {
            "user": "test",
            "access_key": "fooAksesKunci",
            "secret_key": "fooKunciRahasia"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "default_storage_class": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "check_on_raw": false,
        "max_size": -1,
        "max_size_kb": 0,
        "max_objects": -1
    },
    "temp_url_keys": [],
    "type": "rgw",
    "mfa_ids": []
}
```


### Install awscli
```
apt install -y awscli
```

### List user and zone
```
radosgw-admin user list
radosgw-admin zone list
```

### Configure aws client user
```
aws configure --profile=test
---
AWS Access Key ID [None]: fooAksesKunci
AWS Secret Access Key [None]: fooKunciRahasia
Default region name [None]: default
Default output format [None]: json
```

### Create bucket
```
aws --profile test --endpoint-url http://am-ceph1.internal.ammarun.id s3api create-bucket --bucket bucket0

aws --profile test --endpoint-url http://am-ceph1.internal.ammarun.id s3 ls
---output
2023-07-25 16:25:22 bucket0

radosgw-admin bucket list
--output
[
    "bucket0"
]
```

### create sample object then upload to bucket
```
for a in {1..20}
do
  fallocate -l 100M isiember$a
  aws --profile test --endpoint-url http://am-ceph1.internal.ammarun.id s3 cp isiember$a s3://bucket0
done
```

### List object in bucket

```
aws --profile test --endpoint-url http://am-ceph1.internal.ammarun.id s3 ls s3://bucket0
---output
2023-07-25 16:36:25  104857600 isiember1
2023-07-25 16:36:46  104857600 isiember10
2023-07-25 16:36:47  104857600 isiember11
2023-07-25 16:36:49  104857600 isiember12
2023-07-25 16:36:51  104857600 isiember13
2023-07-25 16:36:52  104857600 isiember14
2023-07-25 16:36:55  104857600 isiember15
2023-07-25 16:36:57  104857600 isiember16
2023-07-25 16:36:58  104857600 isiember17
2023-07-25 16:37:00  104857600 isiember18
2023-07-25 16:37:02  104857600 isiember19
2023-07-25 16:36:28  104857600 isiember2
2023-07-25 16:37:04  104857600 isiember20
2023-07-25 16:36:30  104857600 isiember3
2023-07-25 16:36:32  104857600 isiember4
2023-07-25 16:36:34  104857600 isiember5
2023-07-25 16:36:36  104857600 isiember6
2023-07-25 16:36:38  104857600 isiember7
2023-07-25 16:36:40  104857600 isiember8
2023-07-25 16:36:42  104857600 isiember9
```