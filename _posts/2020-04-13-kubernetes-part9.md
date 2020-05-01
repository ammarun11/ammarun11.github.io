---
title: "[Kubernetes The Hard Way] Deploying the DNS Cluster Add-on
 - FinalPart 9"
date: 2020-04-14
categories: [ngoprek, server, cloud, kubernetes, container]
tags:
  - Jekyll
  - update
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
## Deploying the DNS Cluster Add-on (Kubernetes The Hardway)


Lanjutkeunn, Lab FinalPart 9
In this lab you will deploy the [DNS add-on](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/) which provides DNS based service discovery, backed by [CoreDNS](https://coredns.io/), to applications running inside the Kubernetes cluster.

## The DNS Cluster Add-on

<!-- cek namespaces system
```s
root@palo-server0:~# kubectl -n kube-system get all
No resources found.
``` -->

Deploy the `coredns` cluster add-on:
```s
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns.yaml
```
> output

```s
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.extensions/coredns created
service/kube-dns created
```

cek namespaces system,
```s
kubectl -n kube-system get all
```
> Output 

```s
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-788cdf8b7b-dxpxp   1/1     Running   0          46s
pod/coredns-788cdf8b7b-fbfd6   1/1     Running   0          46s

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.32.0.10   <none>        53/UDP,53/TCP,9153/TCP   46s

NAME                      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2         2         2            0           46s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-788cdf8b7b   2         2         0       46s
```

## Verification
Create a busybox deployment:
```s
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```
List the pod created by the busybox deployment:
```s
kubectl get pods -l run=busybox
```
> output

```s
NAME                      READY   STATUS    RESTARTS   AGE
busybox-bd8fb7cbd-vflm9   1/1     Running   0          10s
```

Retrieve the full name of the busybox pod:
```s
POD_NAME=$(kubectl get pods -l run=busybox -o jsonpath="{.items[0].metadata.name}")
```
Execute a DNS lookup for the kubernetes service inside the busybox pod:
```s
kubectl exec -ti $POD_NAME -- nslookup kubernetes
```

> output

```s
Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```


Lanjutnye: [SMOKE-TEST](#)
## Gaskunnnn

**Referensi**
* [https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.12.0)
* [Kubernetes The Hard Way Just me and Opensource](https://www.youtube.com/watch?v=2bVK-e-GuYI&t=560s)

# Happy,  Enjoy Ngoprek ~
