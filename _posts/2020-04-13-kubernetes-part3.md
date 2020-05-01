---
title: "[Kubernetes The Hard Way] Generating Kubernetes Configuration Files for Authentication - Part 3"
date: 2020-04-13
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Generating Kubernetes Configuration Files for Authentication (Kubernetes The Hardway)

Lanjutkeunn, Lab part 4 
this lab you will generate [Kubernetes configuration files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), also known as kubeconfigs, which enable Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

### Kubernetes Public IP Address
Each kubeconfig requires a Kubernetes API Server to connect to. To support high availability the IP address assigned to the external load balancer fronting the Kubernetes API Servers will be used.

Using IP LoadBalancer HAproxy,
```s
KUBERNETES_PUBLIC_ADDRESS=10.240.0.62
```

### The kubelet Kubernetes Configuration File
When generating kubeconfig files for Kubelets the client certificate matching the Kubelet's node name must be used. This will ensure Kubelets are properly authorized by the Kubernetes Node Authorizer.

Generate a kubeconfig file for each worker node:
```s
for instance in worker-0 worker-1 worker-2; do
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```
>Results:
```s
worker-0.kubeconfig
worker-1.kubeconfig
worker-2.kubeconfig
```

### The kube-proxy Kubernetes Configuration File
Generate a kubeconfig file for the kube-proxy service:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_PUBLIC_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```
>Results:
```s
kube-proxy.kubeconfig
```

### The kube-controller-manager Kubernetes Configuration File
Generate a kubeconfig file for the kube-controller-manager service:
```s
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```
>Results:
```s
kube-controller-manager.kubeconfig
```

### The kube-scheduler Kubernetes Configuration File
Generate a kubeconfig file for the kube-scheduler service:
```
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
>Results:
```s
kube-scheduler.kubeconfig
```

### The admin Kubernetes Configuration File
Generate a kubeconfig file for the admin user:
```s
{
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
Results:
```s
admin.kubeconfig
```

## Distribute the Kubernetes Configuration Files
Copy the appropriate kubelet and kube-proxy kubeconfig files to each worker instance:
```s
for instance in worker-0 worker-1 worker-2; do
  lxc file push ${instance}.kubeconfig kube-proxy.kubeconfig ${instance}/root/
done
```

Copy the appropriate kube-controller-manager and kube-scheduler kubeconfig files to each controller instance:

```s
for instance in controller-0 controller-1 controller-2; do
  lxc file push admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig ${instance}/root/
done
```

> Verfication
```s
lxc exec controller-0 ls
lxc exec worker-0 ls
```

Lanjutnye: [Generating the Data Encryption Config and Key - Part 4](https://ammarun.my.id/ngoprek/server/cloud/kubernetes/container/kubernetes-Generating-the-Data-Encryption-Config-and-Key-Part4/)

## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
