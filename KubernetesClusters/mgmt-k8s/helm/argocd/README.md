# How to deploy argocd

```bash
helm upgrade --install argocd argo/argo-cd -n argocd -f values.yaml --version 9.2.3 --create-namespace
```