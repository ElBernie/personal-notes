# Requirements

- a k3s or k8s cluster

# Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

# Port forward the ArgoCD server

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

by default the service will be available only for host machine, to make it available for other machines, you can use the following command:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443 --address 0.0.0.0
```
