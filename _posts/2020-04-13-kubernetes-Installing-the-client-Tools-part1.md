---
title: "[Kubernetes The Hard Way] Installing Client Tools - Part 1"
date: 2020-04-13
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Installing Client Tools (Kubernetes The Hardway)

Lanjutkeunn, Lab part 1 ini kita akan menyiapkan dari sisi client yaitu HOST OS saya sendiri yang menjalan kan node container k8s nah kita akan menggunakan [Public Key Infrastructure Infrastructure](https://en.wikipedia.org/wiki/Public_key_infrastructure)
Untuk media komunikasi dari sisi client ke tiap node container controller dan worker.

In this lab you will install the command line utilities required to complete this tutorial: [cfssl](https://github.com/cloudflare/cfssl), [cfssljson](https://github.com/cloudflare/cfssl), and [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl).

## Linux Install CFSSL
```s
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```

### Verification
Verify cfssl version 1.2.0 or higher is installed:
```s
cfssl version
```
> output
```s
Version: 1.2.0
Revision: dev
Runtime: go1.6
```

## Linux Install Kubectl

```s
wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

## Verification
Verify kubectl version 1.12.0 or higher is installed:
```s
kubectl version --client
```
> output
```s
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-27T17:05:32Z", GoVersion:"go1.10.4", Compiler:"gc", Platform:"linux/amd64"}
```

Lanjutnye: [Provisioning a CA and Generating TLS Certificates - Part 2](http://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Provisioning-a-CA-and-Generating-TLS-Certificates-part2/)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
