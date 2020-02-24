---

title : "[Kubernetes] Latihan Deploy Nginx app Lab k8s (Part 6)"
categories: [ngoprek, server, cloud, docker, container, orchestration]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Kali ini kita akan membuat service nginx dengan studi kasus

Buat sebuah deployment dengan image nginx versi dan replica 1, nama deployment tersebut adalah nginx-X juga pastikan deployment tersebut dideploy di pod worker

kita perlu membuat konfigurasi file untuk deployment dan services
```shell
vim nginx-latihan.yml

...
apiVersion:                    extensions/v1beta1
kind:                          Deployment
metadata:
  name:                        nginx-11
spec:
  replicas:                    1
  template:
    metadata:
      labels:
        app:                   nginx-11
    spec:
      nodeSelector: 
        kubernetes.io/hostname: pod-worker
      containers:
      - name:                  webserver
        image:                 nginx:1.16.1-alpine
        ports:
        - containerPort:       80
        volumeMounts:
        - mountPath:           "/usr/share/nginx/html"
          name:                webservercontent
      volumes:
      - name:                  webservercontent
        persistentVolumeClaim:
          claimName:           nfs
...
```

```shell
vim nginx-svc.yml

...
apiVersion: v1
kind: Service
metadata:
  name: nginx-11
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx-11
...
```

Jalankan file konfigurasinyas
```shell
kubectl create -f nginx-latihan.yaml
kubectl create -f nginx-svc.yaml
```

Cek pod yang berjalan lalu Jabarkan
```shell
kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
nfs-server-6dcf498cdc-2hhqb       1/1     Running   0          68m
nginx-11-6c54d45f9b-jnfsk         1/1     Running   0          12m
nginx-with-pvc-7b4d694967-kd5cl   1/1     Running   0          62m
webserver-79bc4fff6f-d8lsj        1/1     Running   1          3h26m
webserver-79bc4fff6f-lztft        1/1     Running   1          3h26m
webserver-79bc4fff6f-nfgnd        1/1     Running   1          3h26m
```

Berikut hasil nya ~ 

```shell
root@pod-master :~# kubectl describe pod nginx-11-6c54d45f9b-jnfsk
Name:           nginx-11-6c54d45f9b-jnfsk
Namespace:      default
Priority:       0
Node:           pod-worker/10.0.0.5
Start Time:     Sun, 13 Oct 2019 18:21:54 +0000
Labels:         app=nginx-11
                pod-template-hash=6c54d45f9b
Annotations:    <none>
Status:         Running
IP:             10.244.1.25
Controlled By:  ReplicaSet/nginx-11-6c54d45f9b
Containers:
  webserver:
    Container ID:   docker://f8070cf88d15f683ca107e695cf5f1537728877c6778ee545209243f6794a491
    Image:          nginx:1.16.1-alpine
    Image ID:       docker-pullable://nginx@sha256:096c4b3464e2e465f20e9d704f1a0f8d27584df4d6758b6d00a14911cc9bb888
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 13 Oct 2019 18:22:00 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from webservercontent (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-rbxjd (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  webservercontent:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  nfs
    ReadOnly:   false
  default-token-rbxjd:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-rbxjd
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  kubernetes.io/hostname=pod-worker
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age    From                 Message
  ----    ------     ----   ----                 -------
  Normal  Scheduled  6m59s  default-scheduler    Successfully assigned default/nginx-11-6c54d45f9b-jnfsk to pod-worker
  Normal  Pulled     6m54s  kubelet, pod-worker  Container image "nginx:1.16.1-alpine" already present on machine
  Normal  Created    6m53s  kubelet, pod-worker  Created container webserver
  Normal  Started    6m53s  kubelet, pod-worker  Started container webserver
```

```shell
root@pod-master :~# kubectl get replicaset
NAME                        DESIRED   CURRENT   READY   AGE
nfs-server-6dcf498cdc       1         1         1       66m
nginx-11-6c54d45f9b         1         1         1       10m
nginx-with-pvc-7b4d694967   1         1         1       60m
webserver-79bc4fff6f        3         3         3       3h24m
```

> Sudah selesai hasilnya kita membuat deployment nginx dengan replica 1 dan mendeploy nya di pod-worker


# Happy,  Enjoy ngoprek~
