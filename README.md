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

### Helm install argocd in mgmt k8s
``` bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
# Pull down the helm chart values file
helm show values argo/argo-cd --version 9.2.3 > mgmt-k8s/helm/argocd/values.yaml
# Pull down chart version Chart.yaml
helm show chart argo/argo-cd --version 9.2.3 > mgmt-k8s/helm/argocd/Chart.yaml
kubectl create namespace argocd
helm upgrade --install argocd argo/argo-cd -n argocd -f mgmt-k8s/helm/argocd/values.yaml --version 9.2.3

# To obtain the secret
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
# Port forward argocd server
kubectl port-forward svc/argocd-server 8080:443 -n argocd
# Login with username: admin and password from the secret above

# Install argocd cli
# Download from https://localhost:8080/download/argocd-linux-amd64
cp ~/Downloads/argocd-linux-amd64 /usr/local/bin/argocd
chmod +x /usr/local/bin/argocd
argocd version # you will see error under argocd-server because we are not logged in
```

## Login with argocd cli

``` bash
argocd login localhost:8080 --username admin
WARNING: server certificate had error: error creating connection: tls: failed to verify certificate: x509: certificate signed by unknown authority. Proceed insecurely (y/n)? y
Password: 
'admin:login' logged in successfully
Context 'localhost:8080' updated

argocd version
argocd: v3.2.3+2b6251d
  BuildDate: 2025-12-24T12:10:11Z
  GitCommit: 2b6251dfedb54de40596272a73ed1fb19d740219
  GitTreeState: clean
  GoVersion: go1.25.0
  Compiler: gc
  Platform: linux/amd64
argocd-server: v3.2.3+2b6251d
  BuildDate: 2025-12-24T12:10:11Z
  GitCommit: 2b6251dfedb54de40596272a73ed1fb19d740219
  GitTreeState: clean
  GoVersion: go1.25.0
  Compiler: gc
  Platform: linux/amd64
  Kustomize Version: v5.7.0 2025-06-28T07:00:07Z
  Helm Version: v3.18.4+gd80839c
  Kubectl Version: v0.34.0
  Jsonnet Version: v0.21.0

```

## Setup Envoy Gateway
```bash
# 1. Clear any existing manually installed Gateway API CRDs to avoid Helm conflicts
kubectl delete -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml

# 2. Install Envoy Gateway via Helm
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.6.1 -n envoy-gateway-system --create-namespace

# 3. Wait for the controller to be ready
kubectl wait --timeout=5m -n envoy-gateway-system deployment/envoy-gateway --for=condition=Available
```


# Common Errors

### Too many open filesstream closed
- coredns and kube-proxy was unable to start, stuck in Crashloopback    

Solution
```bash
sudo sysctl -w fs.inotify.max_user_watches=524288 && sudo sysctl -w fs.inotify.max_user_instances=512

# make it permanent
sudo tee /etc/sysctl.d/99-kind-inotify.conf <<EOF
fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=512
EOF
sudo sysctl --system
```
The kind containers must be recreated AFTER the sysctl changes are applied for them to inherit the new limits. Simply recreating before applying sysctl won't work.