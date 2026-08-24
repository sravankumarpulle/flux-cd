## 1. (flux-k8s-single-env) branch contains single env with k8s yaml files poc
  #### Step 1: Install Chocolatey (User/Portable Mode)
  #### Step 2: Install Flux CLI Using Chocolatey
  #### Step 3: Install Azure CLI and Required Extensions
  #### Step 4: Create GitHub Personal Access Token (PAT)
  #### Step 5: Prepare GitHub Repository
  #### Step 6: Bootstrap FluxCD with GitHub

## 2. (flux-k8s-multi-env) branch contains multi env and namespaces default and prod with k8s yaml files poc

---------------------------------------
# Fluxcd Installation on Kuberenetes cluster 
## Prerequiresets install Helm cli

```
helm repo add fluxcd-community https://fluxcd-community.github.io/helm-charts
helm repo list
helm repo update
kubectl create ns flux-system
helm install fluxcd fluxcd-community/flux2 -n flux-system   # wait for few mins if case doesn't work execute below upgrade command 
helm list -n flux-system
helm status fluxcd -n flux-system
kubectl get pods -n flux-system
helm upgrade fluxcd fluxcd-community/flux2 -n flux-system
kubectl get pods -n flux-system
```
