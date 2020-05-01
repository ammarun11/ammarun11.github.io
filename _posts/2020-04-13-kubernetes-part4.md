---
title: "[Kubernetes The Hard Way] Generating the Data Encryption Config and Key - Part 4"
date: 2020-04-13
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Generating the Data Encryption Config and Key (Kubernetes The Hardway)

Lanjutkeunn, Lab part 4
Kubernetes stores a variety of data including cluster state, application configurations, and secrets. Kubernetes supports the ability to [encrypt](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data) cluster data at rest.

In this lab you will generate an encryption key and an [encryption config](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration) suitable for encrypting Kubernetes Secrets.

## The Encryption Key
---
Generate an encryption key:
```s
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## The Encryption Config File
---
Create the encryption-config.yaml encryption config file:
```s
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

Copy the `encryption-config.yaml` encryption config file to each controller instance:

```s
for instance in controller-0 controller-1 controller-2; do
  lxc file push encryption-config.yaml ${instance}/root/
done
```
Lanjutnye: [Bootstrapping the etcd Cluster - Part 5](https://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Bootstrapping-the-etcd-Cluster-Part5/)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
