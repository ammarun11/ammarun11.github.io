---

title : "[Kubernetes] Deploying Stand-alone app With CLI Lab k8s (Part 5)"
categories: [ngoprek, server, cloud, docker, container, orchestration]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Sebelumnya kita udah coba deploy app demo yaitu sock-shop nah kali ini kita buat deploy dengan stand-alone.

## Deploying Stand-Alone Application - CLI #####


### 1. Membuat file YAML Deployment webserver.yaml
```shell
vim webserver.yaml

...
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: webserver
    spec:
      containers:
      - name: webserver
        image: nginx:alpine
        ports:
        - containerPort: 80
...
```

### 2. Membuat deployment webserver
```shell
kubectl create -f webserver.yaml
kubectl get deployments
kubectl get replicasets
kubectl get pods
```
```shell
root@pod-master :~# kubectl create -f webserver.yaml 
deployment.extensions/webserver created
root@pod-master :~# kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
webserver   3/3     3            3           43s
root@pod-master :~# kubectl get replicasets
NAME                   DESIRED   CURRENT   READY   AGE
webserver-79bc4fff6f   3         3         3       54s
root@pod-master :~# kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
webserver-79bc4fff6f-d8lsj   1/1     Running   0          60s
webserver-79bc4fff6f-lztft   1/1     Running   0          60s
webserver-79bc4fff6f-nfgnd   1/1     Running   0          60s
```

### 3. Membuat file YAML Service ServiceType NodePort webserver-svc.yaml
```shell
vim webserver-svc.yaml

...
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: webserver 
...
```

### 4. Membuat service
```shell
kubectl create -f webserver-svc.yaml
kubectl get service
kubectl describe svc web-service
```

```shell
root@pod-master :~# kubectl get service
NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes    ClusterIP   10.96.0.1     <none>        443/TCP        3d2h
web-service   NodePort    10.100.27.5   <none>        80:32600/TCP   68s
root@pod-master :~# kubectl describe svc web-service
Name:                     web-service
Namespace:                default
Labels:                   run=web-service
Annotations:              <none>
Selector:                 app=webserver
Type:                     NodePort
IP:                       10.100.27.5
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32600/TCP
Endpoints:                10.244.1.9:80,10.244.2.8:80,10.244.2.9:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### 5. Test browsing IP master port maping service
```shell
curl http://pod-master:[YYYYY]
```
![k8s-app-stand-alone](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/k8s-app-stand-alone.png)

---
# Happy,  Enjoy ngoprek~
