# my-argocd-poc
This is a POC on Argocd Deployment with Github Workflow

## Setup Guide Overview

1. Setup japan k8s locally
2. Setup singapore k8s locally
3. Setup mgmt k8s locally
4. Helm install argocd in mgmt k8s
5. Login with argocd cli
6. Do Active active
7. Do Active passive
8. Try argo workflow
9. Try KRO

```bash
sudo apt-get install docker -y
# Ensure you run post installation to allow docker to run in your user
# https://docs.docker.com/engine/install/linux-postinstall/
sudo apt-get install kind -y
sudo apt-get install kubectl -y
# install k9s, kubectx, kubens
```

## Setup japan k8s locally
``` bash
kind create cluster --name japan-k8s
```

## Setup singapore k8s locally
``` bash
kind create cluster --name singapore-k8s
```

## Setup mgmt k8s locally
``` bash
kind create cluster --name mgmt-k8s
```

## Helm install argocd in mgmt k8s
``` bash

```

## Login with argocd cli

