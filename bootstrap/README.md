# Bootstrap

Run once on a fresh cluster.

## Install Argo CD

```sh
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade --install argo-cd argo/argo-cd \
  -n argocd --create-namespace \
  --version 9.5.0 \
  -f bootstrap/argocd-values.yaml
```

## Apply App-of-Apps

```sh
kubectl apply -f apps/root.yaml
```
