---
title: "[DevOps] Integrate Gitlab and ArgoCD"
date: 2022-12-21
categories: [ngoprek, server, cloud, kubernetes, openshift, devops]
tags:
  - Jekyll
  - update
---

## Guide ## Prerequisites
- ArgoCD Server able to connect to GitLab url
- Have at least one file of kubernetes manifest
- Run yaml on master node

#### 1. Create Repository to store ArgoCD manifest file and kubernetes deployment 
#### 2. Create the guestbook for the first time by create application.yaml file
application.yaml example

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://gitlab.lab.id/sd/argocd-config.git
    targetRevision: HEAD
    path: dev
  destination: 
    server: https://kubernetes.default.svc
    namespace: myapp

  syncPolicy:
    syncOptions:
    - CreateNamespace=true

    automated:
      selfHeal: true
      prune: true
```
#### 3. Deploy application.yaml using kubectl for the first time, later you have to make changes on the repo

```
kubectl apply -f application.yaml
```

![Dashboard1](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/1-argoxgitlab.png)
#### 4. The guestbook will be synced automatically with selected repository and check every 3 minutes

## Manual pull
#### 1. Select one of application
#### 2. Click Refresh and then sync
![Dashboard2](https://raw.githubusercontent.com/ammarun11/ammarun11.github.io/master/static/img/_posts/2-argoxgitlab.png)
