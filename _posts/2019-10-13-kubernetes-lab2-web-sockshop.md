---

title : "[Kubernetes] Deploy Sock-shop Lab k8s (Part 2)"
categories: [ngoprek, server, cloud, docker, container, orchestration]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Lab ini kita akan mendeploy sebuah web toko online (sock-shop) buatan weave network, web ini dibangun microservices dengan kubernetes. 

Bisa kalian lihat lihat sumber code nya [https://github.com/microservices-demo/microservices-demo](https://github.com/microservices-demo/microservices-demo)

## Deploy Web Sock-shop (Toko Online kaos kaki) #####

#### Eksekusi di semua pod ###


```shell
kubectl create namespace sock-shop
kubectl apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
kubectl -n sock-shop get svc front-end
kubectl -n sock-shop get pods
```

```shell
root@pod-master :~# kubectl -n sock-shop get pods

NAME                            READY   STATUS    RESTARTS   AGE
carts-56c6fb966b-q7hqv          1/1     Running   0          10m
carts-db-5678cc578f-4tnn4       1/1     Running   0          10m
catalogue-644549d46f-hj4cn      1/1     Running   0          10m
catalogue-db-6ddc796b66-qqtnj   1/1     Running   0          10m
front-end-5594987df6-gn7tm      1/1     Running   0          10m
orders-749cdc8c9-97bvv          1/1     Running   0          10m
orders-db-5cfc68c4cf-4n5c4      1/1     Running   0          10m
payment-54f55b96b9-bwbdc        1/1     Running   0          10m
queue-master-6fff667867-lq9lj   1/1     Running   0          10m
rabbitmq-bdfd84d55-d589q        1/1     Running   0          10m
shipping-78794fdb4f-cpvz5       1/1     Running   0          10m
user-77cff48476-tkfkp           1/1     Running   0          10m
user-db-99685d75b-hvc2h         1/1     Running   0          10m

root@pod-master :~# kubectl -n sock-shop get svc front-end
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
front-end   NodePort   10.105.121.67   <none>        80:30001/TCP   11m
```
tampilkan sock-shop di browser jika sudah sukses terdeploy. karena saya membuat nya di azure pod master yang memiliki ip public jadi saya hanya perlu untuk mengakses ip public saya lalu port expose front-end 
`ippublic`:30001

![k8s-sockshop](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/k8s-sock-shop.png)

Delete sock-shop namespaces
```shell
kubectl delete namespace sock-shop
```

---
# Happy,  Enjoy ngoprek~
