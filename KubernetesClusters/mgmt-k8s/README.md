# ArgoCD Management Cluster (mgmt-k8s)

This repository contains the configuration and setup guide for the ArgoCD Management Cluster (`mgmt-k8s`) running locally on Kind.

---

## 🚀 Setup Overview

1.  **Environment Preparation**: Install prerequisites and fix system limits.
2.  **Cluster Creation**: Spin up the `mgmt-k8s` Kind cluster.
3.  **ArgoCD Installation**: Deploy ArgoCD via Helm.
4.  **Networking & Ingress**: Setup Envoy Gateway and `cloud-provider-kind` for local access.
5.  **Access**: Configure DNS and retrieve credentials.

---

## 1. Prerequisites & Environment Prep

### Install Local CLI Tools
```bash
# Docker & Kind
sudo apt-get update
sudo apt-get install docker.io kind kubectl -y

# Helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh && ./get_helm.sh && rm get_helm.sh

# Productivity Tools (k9s, kubectx, kubens)
wget https://github.com/derailed/k9s/releases/latest/download/k9s_linux_amd64.deb && sudo apt install ./k9s_linux_amd64.deb && rm k9s_linux_amd64.deb
sudo apt install kubectx -y
curl -sS https://webi.sh/kubens | sh
```

### Fix System Limits (Required for Kind)
Kind may fail to start `coredns` or `kube-proxy` due to `inotify` limits.

```bash
sudo sysctl -w fs.inotify.max_user_watches=524288
sudo sysctl -w fs.inotify.max_user_instances=512

# Make permanent
sudo tee /etc/sysctl.d/99-kind-inotify.conf <<EOF
fs.inotify.max_user_watches=524288
fs.inotify.max_user_instances=512
EOF
sudo sysctl --system
```
> [!IMPORTANT]
> Apply these limits **before** creating the Kind cluster.

---

## 2. Cluster Creation

Create the management cluster:
```bash
docker network remove kind
   - run this if your kind cluster is unable to pull down public images
kind create cluster --name mgmt-k8s
```
---

## 3. Networking & Exposure

### Setup Envoy Gateway (Gateway API)
We use Envoy Gateway to handle ingress traffic via the Kubernetes Gateway API.

1. **Install Envoy Gateway**:
```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm --version v1.5.6 -n envoy-gateway-system --create-namespace
```

2. **Apply Gateway Configuration**:
Ensure [gatewayclass.yaml](gateway-api-setup/gatewayclass.yaml) and [argocd-gateway.yaml](helm/argocd/argocd-gateway.yaml) are applied.
> [!NOTE]
> The `GatewayClass` must use `controllerName: gateway.envoyproxy.io/gatewayclass-controller`.

### Enable LoadBalancer via cloud-provider-kind
Since Kind doesn't have a built-in LoadBalancer, we use `cloud-provider-kind` to assign external IPs.

1. **Install**:
   ```bash
   go install sigs.k8s.io/cloud-provider-kind@latest
   sudo install ~/go/bin/cloud-provider-kind /usr/local/bin
   ```

2. **Run** (Keep this running in a separate terminal):
   ```bash
   cloud-provider-kind --gateway-channel standard
   ```


---
## 4. ArgoCD Installation

Deploy ArgoCD using the provided Helm values:

```bash
kubectl create namespace argocd
helm upgrade --install argocd argo/argo-cd \
  -n argocd \
  -f mgmt-k8s/helm/argocd/values.yaml \
  --version 9.2.3
```

## 5. Accessing ArgoCD

### DNS Configuration
Retrieve the External IP of the Envoy service and add it to your `/etc/hosts`:

```bash
# Find the IP
kubectl get svc -n envoy-gateway-system

# Add to /etc/hosts
# Example: 172.18.0.3 argocd.mgmt.local
```

### Retrieve Admin Password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Access URL
Open **[http://argocd.mgmt.local](http://argocd.mgmt.local)** in your browser.

### DNS Configuration
Retrieve the External IP of the Envoy service and add it to your `/etc/hosts`:

```bash
# Find the IP
kubectl get svc -n envoy-gateway-system

# Add to /etc/hosts
# Example: 172.18.0.6 artifactory.mgmt.local
```

## Common CLI Commands

### Login via ArgoCD CLI
```bash
argocd login argocd.mgmt.local --username admin
```

### Port-Forward fallback (if Gateway is down)
```bash
kubectl port-forward svc/argocd-server 8080:443 -n argocd
```
