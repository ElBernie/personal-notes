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

# Enable insecure mode (optional)

```bash
kubectl edit deployment.apps/argocd-server -n argocd
```

Add the following lines to the file:

```yaml
- argocd-server
- --insecure
```

# Generate admin password

invalidate the current password:

```bash
kubectl patch secret argocd-secret -n argocd -p '{"data": {"admin.password": null, "admin.passwordMtime": null}}'
```

restart pods:

````bash
kubectl delete pods -n argocd -l app.kubernetes.io/name=argocd-server
```
get the new password:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
````

# Get API token

    ```bash

curl http://172.16.225.146:8080/api/v1/session -d $'{"username":"admin","password":"k1Dr1TUlNw-FmZcm"}' -H "Content-Type: application/json"

```

```
