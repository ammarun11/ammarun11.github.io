---
title: "[Kubernetes The Hard Way] Bootstrapping the etcd Cluster - Part 5"
date: 2020-04-13
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Bootstrapping the etcd Cluster (Kubernetes The Hardway)

Lanjutkeunn, Lab part 5
Kubernetes components are stateless and store cluster state in [etcd](https://github.com/coreos/etcd). In this lab you will bootstrap a three node etcd cluster and configure it for high availability and secure remote access.

## Prerequisites
---
The commands in this lab must be run on each controller instance: controller-0, controller-1, and controller-2

## Running commands in parallel with tmux
tmux can be used to run commands on multiple compute instances at the same time. See the Running commands in [parallel with tmux](https://blog.amjith.com/synchronize-panes-in-tmux) section in the Prerequisites lab.

## Bootstrapping an etcd Cluster Member
---
Download and Install the etcd Binaries
Download the official etcd release binaries from the coreos/etcd GitHub project:

```s
wget -q --show-progress --https-only --timestamping \
  "https://github.com/coreos/etcd/releases/download/v3.3.9/etcd-v3.3.9-linux-amd64.tar.gz"
```
Extract and install the `etcd` server and the `etcdctl` command line utility:
```s
{
  tar -xvf etcd-v3.3.9-linux-amd64.tar.gz
  sudo mv etcd-v3.3.9-linux-amd64/etcd* /usr/local/bin/
}
```

### Configure the etcd Server
```s
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:

Controler-0
```s
INTERNAL_IP=10.240.0.88
```
Controller-1
```s
INTERNAL_IP=10.240.161
```
Controller-2
```s
INTERNAL_IP=10.240.39
```

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:
```s
ETCD_NAME=$(hostname -s)
```

Create the `etcd.service` systemd unit file:
```s
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.240.0.88:2380,controller-1=https://10.240.0.161:2380,controller-2=https://10.240.0.39:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Start the etcd Server
---
```s
{
  sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
}
```
> Remember to run the above commands on each controller node: controller-0, controller-1, and controller-2.

## Verification
---
List the etcd cluster members:
```s
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem
```
> output
```s
6c4787f2569330ed, started, controller-1, https://10.240.0.161:2380, https://10.240.0.161:2379
af41a37119e396e0, started, controller-2, https://10.240.0.39:2380, https://10.240.0.39:2379
c8c1068e70981ee2, started, controller-0, https://10.240.0.88:2380, https://10.240.0.88:2379
```

Lanjutnye: [Bootstrapping the Kubernetes Control Plane - Part 6](https://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Bootstrapping-the-Kubernetes-Control-Plane-Part6/)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
