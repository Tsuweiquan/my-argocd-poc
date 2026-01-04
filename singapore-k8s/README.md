# Singapore K8s Cluster

This repository contains the configuration and setup guide for the Singapore K8s cluster running locally on Kind.

---

## 1. Prerequisites & Environment Prep

### Install kind cluster
```bash
kind create cluster --name singapore-k8s
```

### Install Envoy Gateway
```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.1 \
  -n envoy-gateway-system \
  --create-namespace
```

### Install GatewayClass and Gateway
```bash
kubectl apply -f gateway-api-setup/gatewayclass.yaml
kubectl apply -f gateway-api-setup/internal-gateway.yaml
```

### Install SG App
```bash
helm upgrade --install sg-app . -f values.yaml -n prod --create-namespace
```



