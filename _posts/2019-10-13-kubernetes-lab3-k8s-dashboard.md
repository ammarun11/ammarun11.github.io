---

title : "[Kubernetes] Dashboard Lab k8s (Part 3)"
categories: [ngoprek, server, cloud, docker, container, orchestration]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Oke lab ini kita akan memasang Dashbord mili sii kubernetes

## Kubernetes Dashboard #####

### Eksekusi di pod Master #####

### 1. Instal Kubernetes Dashboard

```shell
mkdir ~/certs
kubectl create secret generic kubernetes-dashboard-certs --from-file=$HOME/certs -n kube-system
kubectl apply -f https://raw.githubusercontent.com/nolsatuid/labs/master/k9-adm/script/kubernetes-dashboard.yaml
kubectl -n kube-system get pods
kubectl -n kube-system get svc
```

### 2. Set Permissive RBAC Permissions 
```shell
kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
```

### 3. Menampilkan service account token
```shell
kubectl get secrets
kubectl describe secret [NAMATOKEN]
```

### Browsing ke IP pod master dan port kubernetes-dashboard (HTTPS)
Login dengan https://IP:30000 , lalu isikan token secret and Sign In

![k8s-dashboard](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/k8s-dashboard.png)

---
# Happy,  Enjoy ngoprek~
