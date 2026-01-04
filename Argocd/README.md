# ArgoCD Setup

## Argocd CLI
Since i expose my argocd to port 80 via cloud-provider-kind, i can use argocd cli to login

```bash
argocd login argocd.mgmt.local:80 --insecure --plaintext
Username: admin
Password: 
'admin:login' logged in successfully
Context 'argocd.mgmt.local:80' updated
```

## Add/Register kubernetes clusters

Adding of kubernetes cluster can only be done via the CLI
- https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-management/#adding-a-cluster

```bash
argocd cluster add kind-mgmt-k8s --insecure --plaintext --grpc-web
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `kind-mgmt-k8s` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" already exists in namespace \"kube-system\"","time":"2026-01-04T17:02:17+08:00"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" updated","time":"2026-01-04T17:02:17+08:00"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" updated","time":"2026-01-04T17:02:17+08:00"}
{"level":"info","msg":"Using existing bearer token secret \"argocd-manager-long-lived-token\" for ServiceAccount \"argocd-manager\"","time":"2026-01-04T17:02:17+08:00"}
Cluster 'https://172.18.0.2:6443' added
```
If you encounter this error during adding of clusters
```bash
{"level":"fatal","msg":"rpc error: code = Unknown desc = error getting server version: failed to get server version: Get \"https://127.0.0.1:44289/version?timeout=32s\": dial tcp 127.0.0.1:44289: connect: connection refused","time":"2026-01-04T16:57:59+08:00"}
```

Solution

```bash
kubectl get endpoints kubernetes -n default --context=kind-mgmt-k8s
NAME         ENDPOINTS         AGE
kubernetes   172.18.0.2:6443   43h

vim ~/.kube/config
Update the cluster to use the correct IP
# server: https://127.0.0.1:44289
server: https://172.18.0.2:6443
```

Add the clusters into argocd
```bash
argocd cluster add kind-mgmt-k8s --insecure --plaintext --grpc-web
argocd cluster add kind-japan-k8s --insecure --plaintext --grpc-web
argocd cluster add kind-singapore-k8s --insecure --plaintext --grpc-web
``` 
![registered clusters](argocd-clusters.png)

"Unknown" status is normal and expected for your management cluster after argocd cluster add succeeds. It shows "Unknown" because no Applications are deployed yet—ArgoCD only marks clusters "Successful" once they have monitored apps.
​

## Create a new Project for each region
Example: 
`argocd proj create myproject -d https://kubernetes.default.svc,mynamespace`

- Singapore
    - `argocd proj create singapore -d kind-singapore-k8s,prod --grpc-web`
- Japan
    - `argocd proj create japan -d kind-japan-k8s,prod --grpc-web`

