---
title: "[Ceph] Upgrade Ceph Octopus 15 to Pacific 16 with cephadm"
date: 2022-12-18
categories: [ngoprek, server, cloud, ceph, storage]
tags:
  - Jekyll
  - update
---
## Guide

1.  Check version ceph
```
ceph versions
```

2.  View the ceph release and read the release notes

Visit [Ceph Release](https://docs.ceph.com/en/latest/releases/index.html)

3.  do upgrade ceph

```
ceph orch upgrade start --ceph-version <versi_ceph>

# Example
# ceph orch upgrade start --ceph-version 16.2.10
```

4.  Watch the process of upgrade

Use commands below for checking the process upgrade ceph

```
ceph orch upgrade status

ceph -W cephadm

ceph orch ps

ceph versions
```

#### Before upgrade
![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1-ceph-octopus.png) 

#### Process upgrade
![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2-ceph-upgrade-process.png) 

5.  Verification upgraded

![Dashboard3](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/3-ceph-upgrade-success.png)


## Referensi

-   [Ceph Release](https://docs.ceph.com/en/latest/releases/index.html)
-   [cephadm upgrade guide](https://docs.ceph.com/en/latest/cephadm/upgrade/)