---
title: "[DevOps] ArgoCD Installation"
date: 2022-12-21
categories: [ngoprek, server, cloud, kubernetes, openshift, devops, argocd]
tags:
  - Jekyll
  - update
---
## Guide
### 1.  Install ArgoCD
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2.  Access ArgoCD API Server
choose one of the following techniques to expose the Argo CD API server:

- Port Forward:
```
kubectl port-forward svc/argocd-server -n argocd --address 0.0.0.0 8080:443
```
- Node Port:
```
kubectl expose deployment argocd-server --type=NodePort --name=argocd-server-nodeport -n argocd
kubectl patch service argocd-server-nodeport -n argocd --type='json' --patch='[{"op": "replace", "path": "/spec/ports/0/nodePort", "value":30000}]'
```

### 3.  Login Using The CLI
- get password
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
- login CLI with password
```
argocd login <ARGOCD_SERVER>
```
- Change the password
```
argocd account update-password
```
### 4.  Add tls certificate for connecting to Git repositories
1. Open Argocd Server UI
2. Go to setting
3. Click Certificates
4. Add TLS Certificate
5. Input git cert
### 5.  Add hostalises gitlab to argocd-repo deployment yaml

    hostAliases:
    - hostnames:
      - "gitlab.btech.id"
      ip: 192.168.1.19

### 6.  Install Argocd CLI
    curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
    chmod +x /usr/local/bin/argocd

### 7.  Login to Argocd CLI
    Argocd login hostname

### 8. Setup a credential template for all repos under https://docker-build/repos
    argocd repocreds add https://gitlab.btech.id/repo --username username --password password --upsert
    argocd repo add https://gitlab.btech.id/calvin/argocd-config --username username --password password --upsert

## References
- [Argocd Official Page](https://argo-cd.readthedocs.io/en/stable/getting_started/)