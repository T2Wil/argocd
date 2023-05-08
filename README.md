
# ArgoCD & image-updater
A declarative, GitOps continuous delivery tool for Kubernetes.<br>
We use ArgoCD to automate the deployments in specified target environments.

### This chart is made up of 2 chart dependencies:

- `ArgoCD` from [official ArgoCD chart](https://github.com/argoproj/argo-helm/tree/master/charts/argo-cd)
- `Argocd-image-updater`: which is an argocd plugin to allow automatic image updates.  Refer to this [official documentation](https://github.com/argoproj-labs/argocd-image-updater/blob/master/docs/configuration/images.md)



# Installation process
```
# Step 1: Install dependencies (Argocd and argocd-image-updater plugin)
~ helm dependency update .

# Step 2: If docker images to be pulled are in private registry
# Generate secret like below
~ kubectl create secret docker-registry my-registry-secret \
  --docker-server=my-registry.example.com \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email@example.com \
  --dry-run=client -o yaml > secret.yaml

# Step 3: From step 2 in secret.yaml, copy ".dockerconfigjson" to  "dockerConfigJson" in Values.yaml file

# Step 4: Install ArgoCD request crds
~ mv templates templates-temp
~ helm -n argo install argocd . -f values.yaml --create-namespace

# Step 5: Install argocd
~ mv templates-temp templates
~ helm -n argo upgrade argocd . -f values.yaml --create-namespace

```

<br/>



# Get password
- `kubectl -n argo get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

# Some issues and how to fix them
```
# When ingress and TLS are enabled sometimes "ERR_TOO_MANY_REDIRECTS" is returned while accessing exposed ingress.
# This is due to due to conflicting internal tls
# Fix it with the below command

~ kubectl patch configmap/argocd-cmd-params-cm \
  -n argo \
  --type merge \
  -p '{"data":{"server.insecure":"true"}}'

# restart argocd-server to pick the changes
kubectl -n argo  rollout restart deploy argocd-server
```

# Useful links
[ArgoCD declarative setup](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/)

Adding a service to argocd:
- [How to create an argo project](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#projects)
- [How to create an argo application](https://argo-cd.readthedocs.io/en/stable/operator-manual/declarative-setup/#applications)
