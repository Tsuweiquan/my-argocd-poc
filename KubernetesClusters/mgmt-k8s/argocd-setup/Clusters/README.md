# Add Clusters into ArgoCD

```
argocd login argocd.mgmt.local:80 --insecure --plaintext
argocd cluster add kind-singapore-k8s --insecure --plaintext --grpc-web
```

You might get error like this:

```
{"level":"fatal","msg":"rpc error: code = Unknown desc = error getting server version: failed to get server version: Get \"https://127.0.0.1:46261/version?timeout=32s\": dial tcp 127.0.0.1:46261: connect: connection refused","time":"2026-01-09T21:55:11+08:00"}
```
To fix this, you need to patch the ~/.kube/config file to replace the 127.0.0.1:46261 entry with the actual IP address of the cluster.
To obtain the IP
```
kubectl get endpoints kubernetes -n default --context=kind-singapore-k8s
NAME         ENDPOINTS         AGE
kubernetes   172.18.0.4:6443   9m10s
```

Then patch the ~/.kube/config file to replace the 127.0.0.1:46261 entry with the actual IP address of the cluster.

```
# Rerun the above command
argocd cluster add kind-singapore-k8s --insecure --plaintext --grpc-web
``` 