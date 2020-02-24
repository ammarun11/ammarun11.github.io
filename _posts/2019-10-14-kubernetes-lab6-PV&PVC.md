---

title : "[Kubernetes] Presistent Volume (PV) & Presistent Volume Claim (PVC) Lab k8s (Part 6)"
categories: [ngoprek, server, cloud, docker, container, orchestration]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

# PV & PVC #####

This example describes how to create Web frontend server, an auto-provisioned persistent volume on GCE or Azure, and an NFS-backed persistent claim.

Demonstrated Kubernetes Concepts:

* Persistent Volumes to define persistent disks (disk lifecycle not tied to the Pods).
* Services to enable Pods to locate one another.

ref :
[https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs](https://github.com/kubernetes/examples/tree/master/staging/volumes/nfs)


### Di pod worker, buat direktori /data #####
```shell
sudo mkdir /data
```
### Unduh file manifest untuk deploy NFS server #####
```shell
wget -c https://raw.githubusercontent.com/nolsatuid/labs/master/k9-adm/script/nfs-server.yaml
```
```shell
vim nfs-server.yaml

---
    spec:
      nodeSelector: 
        kubernetes.io/hostname: node0
```
ganti `node0` dengan nama host `pod-worker` (kubectl get nodes) masing-masing.

### Di node master, jalankan file nfs-server.yaml #####
```shell
kubectl create -f nfs-server.yaml
kubectl describe deployment nfs-server
kubectl describe services nfs-server
```
 Catat ClusterIP dari nfs-server 10.97.142.135

### PV Provisioning. Download nfs-pv.yaml dan edit IP server dengan ClusterIP nfs-server #####
```shell
wget -c https://raw.githubusercontent.com/nolsatuid/labs/master/k9-adm/script/pv.yaml
```
```shell
vim pv.yaml
.....
  nfs:
    # FIXME: use the right IP
    server: use IP from nfs-server ClusterIP
path: "/exports"
....
```
```shell
kubectl create -f pv.yaml
kubectl get pv
```

## PVC ###
```shell
kubectl create -f https://raw.githubusercontent.com/nolsatuid/labs/master/k9-adm/script/pvc.yaml
kubectl get pvc
```

### Instal paket nfs-common di semua node ###
```shell
sudo apt install -y nfs-common
```

### Setup apps using pvc ###
```shell
kubectl create -f https://raw.githubusercontent.com/nolsatuid/labs/master/k9-adm/script/nginx.yaml
```

### Buat service untuk nginx (nginx-svc.yaml) ###
```shell
vim nginx-svc.yaml
...
apiVersion: v1
kind: Service
metadata:
  name: nginx-server
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
...

kubectl create -f nginx-svc.yaml
```

### tambahkan index.html di folder /data di pod worker ###
```shell
sudo su
echo "this is index file from palopalepalo-11" >> /data/index.html
```

### Testing #####
curl http://pod-master:[XXXXX]
---
![k8s-pv&pvc](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/k8s-pv.png)

# Happy,  Enjoy ngoprek~
