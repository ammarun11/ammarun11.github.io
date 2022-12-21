---
title: "[GCP] Google Kubernetes Engine Setup"
date: 2022-12-21
categories: [ngoprek, server, cloud, kubernetes, gke, devops, ]
tags:
  - Jekyll
  - update
---

## Guide ## Prerequisites

1. Set default project
```
$ gcloud config set project playground-s-11-73835160

---
Updated property [core/project].
---
```

2. Set compute zone  
Compute zone [list](https://cloud.google.com/compute/docs/regions-zones#available)
```
$ gcloud config set compute/zone asia-southeast1-a

---
Updated property [compute/zone].
---
```

3. Enable Google Container Services
```
$ gcloud services enable container.googleapis.com

---
Operation "operations/acf.p2-98174380011-113d48c4-96d5-457e-9060-da78eaeaf63b" finished successfully.
---
```
4. Get Cluster configuration  
```
$ gcloud container get-server-config

---
Fetching server config for asia-southeast1-a
channels:
- channel: RAPID
  defaultVersion: 1.19.7-gke.2503
  validVersions:
  - 1.20.4-gke.400
  - 1.19.8-gke.1000
  - 1.19.7-gke.2503
- channel: REGULAR
  defaultVersion: 1.18.12-gke.1210
  validVersions:
  - 1.18.15-gke.1501
  - 1.18.12-gke.1210
- channel: STABLE
  defaultVersion: 1.17.17-gke.1101
  validVersions:
  - 1.17.17-gke.1101
  - 1.17.17-gke.1100
  - 1.16.15-gke.7801
defaultClusterVersion: 1.17.17-gke.1101
defaultImageType: COS
validImageTypes:
- COS
- UBUNTU
- COS_CONTAINERD
- UBUNTU_CONTAINERD
- WINDOWS_SAC
- WINDOWS_LTSC
validMasterVersions:
- 1.18.16-gke.500
- 1.18.16-gke.300
validNodeVersions:
- 1.18.16-gke.500
- 1.18.16-gke.300
---
```

5. Create GKE cluster  
For deploy spesific version `--cluster-version` atau `--release-channel`.

- use `--cluster-version`
```
$ gcloud container clusters create gke-1 --num-nodes=1 --cluster-version=1.17.17-gke.1101 --image-type=UBUNTU

---
NAME   LOCATION           MASTER_VERSION    MASTER_IP       MACHINE_TYPE  NODE_VERSION      NUM_NODES  STATUS
k8s-1  asia-southeast1-a  1.17.17-gke.1101  35.185.185.239  e2-medium     1.17.17-gke.1101  1          RUNNING
---
```

- use `--release-channel`
```
$ gcloud container clusters create k8s-1 --release-channel rapid --num-nodes=1

---
NAME   LOCATION           MASTER_VERSION   MASTER_IP       MACHINE_TYPE  NODE_VERSION     NUM_NODES  STATUS
k8s-1  asia-southeast1-a  1.19.7-gke.2503  35.240.168.191  e2-medium     1.19.7-gke.2503  1          RUNNING
---
```

6. Get cluster credentials
```
$ gcloud container clusters get-credentials k8s-1

---
Fetching cluster endpoint and auth data.
kubeconfig entry generated for k8s-1.
---
```

7. Test cluster credentials
```
$ kubectl get nodes

---
NAME                                   STATUS   ROLES    AGE     VERSION
gke-k8s-1-default-pool-92a66944-mjfd   Ready    <none>   8m12s   v1.17.17-gke.1101
---
```

8. Create hello deployment
```
$ kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:1.0

---
deployment.apps/hello-server created
---
```

9. Expose deployment
```
$ kubectl expose deployment hello-server --type LoadBalancer --port 80 --target-port 8080
---
service/hello-server exposed
---
```

10. Access `hello-server` deployment.
```
$ kubectl get svc hello-server

---
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
hello-server   LoadBalancer   10.43.241.119   34.126.118.29   80:31021/TCP   92s
---

Access to using web browser or curl : 34.126.118.29
```

11. Clean up resources
```
$ kubectl delete deployment hello-server

---
deployment.apps "hello-server" deleted
---

$ gcloud container clusters delete k8s-1
---
The following clusters will be deleted.
 - [k8s-1] in [asia-southeast1-a]

Do you want to continue (Y/n)?  y

Deleting cluster k8s-1...done.
Deleted [https://container.googleapis.com/v1/projects/playground-s-11-73835160/zones/asia-southeast1-a/clusters/k8s-1].
---
```

## Build and Deploying Apps
01. Create Apps 
```
mkdir lac-app && cd lac-app 
vim app.py
---
from flash import Flask
import os
import socket

app = Flask(__name__)

@app.route("/")
def hello():
    html = "<h1 style=text-align:center;margin:20px;>Greeting from linux academy</h1>"
    return html

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
---

vim Dockerfile
---
FROM python:2.7-slim
WORKDIR /app
ADD . /app

RUN pip install -r requirements.txt

EXPOSE 80

CMD ["python", "app.py"]
---

vim requirements.txt
---
Flask
---
```

02. Build App

```
docker build -t lac-app-image 
```

03. Configure docker auth 

```
gcloud auth configure-docker
```

04. Tag image 

```
docker tag lac-app-image gcr.io/playground-s-11-94c871ea/lac-app-image:v1
```

05. Push image 

```
docker push gcr.io/playground-s-11-94c871ea/lac-app-image:v1
```

06. Create deployment
```
kubectl create deployment lac-app --image=gcr.io/playground-s-11-94c871ea/lac-app-image:v1
```

07. Expose deployment 

```
$ kubectl expose deployment lac-app --type LoadBalancer --port 80 --target-port 80
```

## Node Pools
1. Create node pool
```
$ gcloud container node-pools create k8s-pool --cluster k8s-1

---
Creating node pool k8s-pool...done.
Created [https://container.googleapis.com/v1/projects/playground-s-11-73835160/zones/asia-southeast1-a/clusters/k8s-1/nodePools/k8s-pool].
NAME      MACHINE_TYPE  DISK_SIZE_GB  NODE_VERSION
k8s-pool  e2-medium     100           1.17.17-gke.1101
---
```

2. List polls
```
$ gcloud container node-pools list --cluster k8s-1

---
NAME          MACHINE_TYPE  DISK_SIZE_GB  NODE_VERSION
default-pool  e2-medium     100           1.17.17-gke.1101
k8s-pool      e2-medium     100           1.17.17-gke.1101
---
```

3. Describe pools

```
$ gcloud container node-pools describe k8s-pool --cluster k8s-1

---
config:
  diskSizeGb: 100
  diskType: pd-standard
  imageType: COS
  machineType: e2-medium
  metadata:
    disable-legacy-endpoints: 'true'
  oauthScopes:
  - https://www.googleapis.com/auth/devstorage.read_only
  - https://www.googleapis.com/auth/logging.write
  - https://www.googleapis.com/auth/monitoring
  - https://www.googleapis.com/auth/service.management.readonly
  - https://www.googleapis.com/auth/servicecontrol
  - https://www.googleapis.com/auth/trace.append
  serviceAccount: default
  shieldedInstanceConfig:
    enableIntegrityMonitoring: true
initialNodeCount: 3
instanceGroupUrls:
- https://www.googleapis.com/compute/v1/projects/playground-s-11-73835160/zones/asia-southeast1-a/instanceGroupManagers/gke-k8s-1-k8s-pool-30f4b3cc-grp
locations:
- asia-southeast1-a
management:
  autoRepair: true
  autoUpgrade: true
name: k8s-pool
podIpv4CidrSize: 24
selfLink: https://container.googleapis.com/v1/projects/playground-s-11-73835160/zones/asia-southeast1-a/clusters/k8s-1/nodePools/k8s-pool
status: RUNNING
upgradeSettings:
  maxSurge: 1
version: 1.17.17-gke.1101
---
```

4. Get cluster node information
```
$ gcloud container clusters get-credentials k8s-1
$ kubectl get nodes

---
NAME                                   STATUS   ROLES    AGE     VERSION
gke-k8s-1-default-pool-ee4a1364-2nj0   Ready    <none>   10m     v1.17.17-gke.1101
gke-k8s-1-k8s-pool-30f4b3cc-01n5       Ready    <none>   3m17s   v1.17.17-gke.1101
gke-k8s-1-k8s-pool-30f4b3cc-mbqk       Ready    <none>   3m17s   v1.17.17-gke.1101
gke-k8s-1-k8s-pool-30f4b3cc-r9f5       Ready    <none>   3m15s   v1.17.17-gke.1101
---
```

5. Resize Node Pools size
```
$ gcloud container clusters resize k8s-1 --node-pool k8s-pool --num-nodes 2

---
Pool [k8s-pool] for [k8s-1] will be resized to 2.

Do you want to continue (Y/n)?  Y

Resizing k8s-1...done.
Updated [https://container.googleapis.com/v1/projects/playground-s-11-73835160/zones/asia-southeast1-a/clusters/k8s-1].
---
```

List nodes

```
$ kubectl get nodes

---
NAME                                   STATUS   ROLES    AGE   VERSION
gke-k8s-1-default-pool-ee4a1364-2nj0   Ready    <none>   19m   v1.17.17-gke.1101
gke-k8s-1-k8s-pool-30f4b3cc-01n5       Ready    <none>   12m   v1.17.17-gke.1101
gke-k8s-1-k8s-pool-30f4b3cc-mbqk       Ready    <none>   12m   v1.17.17-gke.1101
---
```

6. Deploy pod to specific Node Pools  

Create Pods

```
$ vim nginx.yaml

---
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    cloud.google.com/gke-nodepool: k8s-pool
---

$ kubectl apply -f nginx.yaml
```

Show Pods
```
$ kubectl get po -o  wide

---
NAME    READY   STATUS    RESTARTS   AGE     IP          NODE                               NOMINATED NODE   READINESS GATES
nginx   1/1     Running   0          2m50s   10.40.2.2   gke-k8s-1-k8s-pool-30f4b3cc-mbqk   <none>           <none>
---
```

## Windows Node Pools
1. Enable Google Containers API
```
gcloud services enable container.googleapis.com
```

2. Create Cluster
```
gcloud container clusters create k8s-1 --enable-ip-alias --num-nodes=2 --release-channel stable
```

3. Create Windows Node-pool
```
gcloud container node-pools create win-pool --cluster=k8s-1 --image-type=windows_ltsc --no-enable-autoupgrade --machine-type=n1-standard-2 --num-nodes=1
```

4. Get Cluster Credentials
```
gcloud container clusters get-credentials k8s-1
```

5. List node
```
kubetcl get nodes -o wide

---
NAME                                   STATUS   ROLES    AGE     VERSION             INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                             KERNEL-VERSION    CONTAINER-RUNTIME
gke-k8s-1-default-pool-4f6ea0e1-nn63   Ready    <none>   18m     v1.17.17-gke.1101   10.148.0.4    35.198.251.170   Container-Optimized OS from Google   4.19.150+         docker://19.3.6
gke-k8s-1-default-pool-4f6ea0e1-vlf8   Ready    <none>   18m     v1.17.17-gke.1101   10.148.0.5    35.247.167.48    Container-Optimized OS from Google   4.19.150+         docker://19.3.6
gke-k8s-1-win-pool-6eb842ff-d4vm       Ready    <none>   5m45s   v1.17.17-gke.1101   10.148.0.7    35.198.232.130   Windows Server 2019 Datacenter       10.0.17763.1757   docker://19.3.14
gke-k8s-1-win-pool-6eb842ff-k8xv       Ready    <none>   5m51s   v1.17.17-gke.1101   10.148.0.6    35.197.145.163   Windows Server 2019 Datacenter       10.0.17763.1757   docker://19.3.14
gke-k8s-1-win-pool-6eb842ff-lvsl       Ready    <none>   5m53s   v1.17.17-gke.1101   10.148.0.8    35.240.232.98    Windows Server 2019 Datacenter       10.0.17763.1757   docker://19.3.14
---
```

6. Create windows apps deployment
```
vim iis.yaml

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iis
  labels:
    app: iis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iis
  template:
    metadata:
      labels:
        app: iis
    spec:
      nodeSelector:
        kubernetes.io/os: windows
      containers:
      - name: iis-server
        image: mcr.microsoft.com/windows/servercore/iis
        ports:
        - containerPort: 80
---

kubectl apply -f iis.yaml
```


7. Wait the deployment ready
```
kubectl get deployment --watch
```

8. Expose deployment
```
kubectl expose deployment iis --type=LoadBalancer --name iis --port 80 --target-port 80
```

9. Access service external IP
```
kubectl get svc iis
```