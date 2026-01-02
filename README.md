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

## Prerequisites setup for local cli
```bash
sudo apt-get install docker -y
# Ensure you run post installation to allow docker to run in your user
# https://docs.docker.com/engine/install/linux-postinstall/
sudo apt-get install kind -y
sudo apt-get install kubectl -y
# install k9s
wget https://github.com/derailed/k9s/releases/latest/download/k9s_linux_amd64.deb && apt install ./k9s_linux_amd64.deb && rm k9s_linux_amd64.deb
# install kubectx, kubens
sudo apt install kubectx
curl -sS https://webi.sh/kubens | sh; source ~/.config/envman/PATH.env

# install helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh
rm get_helm.sh
```

## Setup singapore k8s locally
``` bash
kind create cluster --name singapore-k8s
```

## Setup japan k8s locally
``` bash
kind create cluster --name japan-k8s
```

## Setup mgmt k8s locally
``` bash
kind create cluster --name mgmt-k8s
```

### Install Gateway API CRDs
``` bash
# https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-a-gateway-controller

# Install Gateway API
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml
kubectl get crds

```

### Helm install argocd in mgmt k8s
``` bash

```

## Login with argocd cli

