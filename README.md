# FluxCD Kubernetes POC

This repository contains Proof of Concept (POC) implementations for deploying and managing Kubernetes workloads using **FluxCD GitOps**.

## Branches

### 1. `flux-k8s-single-env`

This branch contains a **single-environment Kubernetes POC** using Kubernetes YAML manifests.

The POC covers:

1. Install Chocolatey
2. Install Flux CLI using Chocolatey
3. Install Azure CLI and required extensions
4. Create GitHub Personal Access Token (PAT)
5. Prepare the GitHub repository
6. Bootstrap FluxCD with GitHub
7. Deploy Kubernetes manifests using FluxCD

### 2. `flux-k8s-multi-env`

This branch contains a **multi-environment Kubernetes POC**.

It demonstrates:

* Multiple environments
* `default` namespace
* `prod` namespace
* Environment-specific Kubernetes YAML manifests
* FluxCD GitOps-based deployment

---

# FluxCD Installation on Kubernetes

## Prerequisites

The following tools are required:

* Kubernetes cluster
* `kubectl`
* Helm CLI
* Azure CLI — required if using AKS
* Flux CLI
* GitHub account
* GitHub repository
* GitHub Personal Access Token (PAT)

For AKS, make sure your local machine is authenticated to Azure and your `kubectl` context points to the correct AKS cluster.

---

# 1. Install Helm CLI

Verify Helm installation:

```cmd
helm version
```

Add the FluxCD Community Helm repository:

```cmd
helm repo add fluxcd-community https://fluxcd-community.github.io/helm-charts
```

Verify the repository:

```cmd
helm repo list
```

Update Helm repositories:

```cmd
helm repo update
```

---

# 2. Create Flux System Namespace

Create the `flux-system` namespace:

```cmd
kubectl create namespace flux-system
```

Verify:

```cmd
kubectl get namespace flux-system
```

> **Note:** If the namespace already exists, you can skip this step.

---

# 3. Install FluxCD Using Helm

Install FluxCD:

```cmd
helm install fluxcd fluxcd-community/flux2 -n flux-system
```

Check the Helm release:

```cmd
helm list -n flux-system
```

Check Helm release status:

```cmd
helm status fluxcd -n flux-system
```

Check FluxCD pods:

```cmd
kubectl get pods -n flux-system
```

If the installation requires an upgrade/reconciliation, run:

```cmd
helm upgrade fluxcd fluxcd-community/flux2 -n flux-system
```

Verify the pods again:

```cmd
kubectl get pods -n flux-system
```

---

# 4. Install Chocolatey

Chocolatey is used to install the Flux CLI on Windows.

Open **Command Prompt or PowerShell as Administrator** and install Chocolatey.

After installation, verify:

```cmd
choco --version
```

---

# 5. Install Flux CLI Using Chocolatey

Install Flux CLI:

```cmd
choco install flux
```

Verify the installation:

```cmd
flux --version
```

Check whether the Kubernetes cluster satisfies Flux prerequisites:

```cmd
flux check --pre
```

---

# 6. Install Azure CLI

If the Kubernetes cluster is running on Azure AKS, install Azure CLI.

Verify:

```cmd
az --version
```

Login to Azure:

```cmd
az login
```

Check the available subscriptions:

```cmd
az account list --output table
```

Select the required subscription:

```cmd
az account set --subscription "<SUBSCRIPTION_ID>"
```

---

# 7. Connect `kubectl` to AKS

Get AKS credentials:

```cmd
az aks get-credentials ^
  --resource-group "<RESOURCE_GROUP>" ^
  --name "<AKS_CLUSTER_NAME>" ^
  --overwrite-existing
```

Verify the current Kubernetes context:

```cmd
kubectl config current-context
```

Verify the AKS nodes:

```cmd
kubectl get nodes
```

---

# 8. Create GitHub Personal Access Token

FluxCD requires GitHub authentication to push the Flux configuration into the GitHub repository.

Go to:

**GitHub → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens**

Create a token with access to the repository:

```text
Repository:
    sravankumarpulle/flux-cd
```

Recommended permissions:

```text
Repository permissions

Administration    → Read and write
Contents          → Read and write
Metadata          → Read-only
```

> Keep the GitHub PAT secure. Do not commit the PAT to the Git repository or put it inside YAML files.

---

# 9. Configure GitHub Token

On Windows CMD:

```cmd
set GITHUB_TOKEN=github_pat_xxxxxxxxxxxxxxxxx
```

Verify that the environment variable is configured:

```cmd
echo %GITHUB_TOKEN%
```

> Do not share the actual token in GitHub, README files, screenshots, or chat messages.

---

# 10. Prepare GitHub Repository

Repository:

```text
https://github.com/sravankumarpulle/flux-cd
```

FluxCD will use the following directory:

```text
clusters/
└── my-cluster/
    └── flux-system/
```

The Flux bootstrap configuration will be created under:

```text
clusters/my-cluster/flux-system
```

---

# 11. Bootstrap FluxCD with GitHub

Bootstrap FluxCD into the Kubernetes cluster:

```cmd
flux bootstrap github ^
  --owner=sravankumarpulle ^
  --repository=flux-cd ^
  --branch=main ^
  --path=clusters/my-cluster/flux-system ^
  --personal
```

If your repository uses `master` instead of `main`, use:

```cmd
flux bootstrap github ^
  --owner=sravankumarpulle ^
  --repository=flux-cd ^
  --branch=master ^
  --path=clusters/my-cluster/flux-system ^
  --personal
```

The bootstrap process:

1. Connects to GitHub
2. Clones the repository
3. Generates FluxCD manifests
4. Commits the manifests to GitHub
5. Installs FluxCD components into `flux-system`
6. Creates the GitRepository configuration
7. Creates the Kustomization configuration
8. Configures FluxCD to continuously reconcile the cluster

---

# 12. Verify FluxCD Installation

Check FluxCD:

```cmd
flux check
```

Check all Flux resources:

```cmd
flux get all -A
```

Check Kustomizations:

```cmd
flux get kustomizations -A
```

Check GitRepository:

```cmd
flux get sources git -A
```

Check FluxCD pods:

```cmd
kubectl get pods -n flux-system
```

Check FluxCD resources:

```cmd
kubectl get all -n flux-system
```

---

# 13. Verify FluxCD GitHub Secret

Flux stores the Git authentication information in a Kubernetes Secret.

Verify that the Secret exists:

```cmd
kubectl get secret flux-system -n flux-system
```

Do **not** display or share the Secret contents.

---

# 14. FluxCD Components

After successful installation, the following Flux controllers are typically deployed:

```text
flux-system
│
├── source-controller
├── kustomize-controller
├── helm-controller
└── notification-controller
```

### Source Controller

Responsible for retrieving sources such as:

* Git repositories
* Helm repositories
* OCI repositories
* Buckets

### Kustomize Controller

Responsible for applying Kubernetes manifests and Kustomizations.

### Helm Controller

Responsible for managing Helm releases using FluxCD `HelmRelease` resources.

### Notification Controller

Responsible for handling events and notifications.

---

# 15. GitOps Workflow

The overall workflow is:

```text
Developer
    │
    │ git push
    ▼
GitHub Repository
    │
    │
    ▼
FluxCD Source Controller
    │
    ▼
FluxCD Kustomize / Helm Controller
    │
    ▼
Kubernetes / AKS
```

Once FluxCD is bootstrapped, Git becomes the source of truth for the Kubernetes configuration.

```text
GitHub
   │
   │ desired state
   ▼
FluxCD
   │
   │ reconciliation
   ▼
AKS Kubernetes Cluster
```

---

# 16. Important Note About Existing Flux Helm Installation

If FluxCD was already installed using Helm:

```cmd
helm install fluxcd fluxcd-community/flux2 -n flux-system
```

and you subsequently run:

```cmd
flux bootstrap github ...
```

Flux may detect the existing Helm installation and ask:

```text
Are you sure you want to override the Helm installation? Y/N:
```

Do not blindly override an existing Flux installation in a production cluster.

Before proceeding, check:

```cmd
helm list -n flux-system
```

```cmd
kubectl get pods -n flux-system
```

```cmd
flux check
```

If the cluster is only a fresh POC environment, the existing installation can be cleaned up and Flux can then be bootstrapped using GitHub.

---

# 17. FluxCD Repository Structure

Example repository structure:

```text
flux-cd/
│
├── clusters/
│   │
│   ├── my-cluster/
│   │   └── flux-system/
│   │       ├── gotk-components.yaml
│   │       ├── gotk-sync.yaml
│   │       └── kustomization.yaml
│   │
│   └── prod/
│       └── flux-system/
│
├── apps/
│   ├── default/
│   └── prod/
│
└── README.md
```

For the single-environment POC:

```text
clusters/
└── my-cluster/
    └── flux-system/
```

For the multi-environment POC:

```text
clusters/
├── dev/
│   └── flux-system/
│
└── prod/
    └── flux-system/
```

---

# 18. Branch Strategy

## `flux-k8s-single-env`

Single Kubernetes environment:

```text
flux-k8s-single-env
        │
        ▼
   my-cluster
        │
        ▼
 Kubernetes YAML
```

## `flux-k8s-multi-env`

Multiple environments:

```text
flux-k8s-multi-env
        │
        ├── default namespace
        │
        └── prod namespace
```

This branch demonstrates environment-specific GitOps configuration.

---

# 19. Useful FluxCD Commands

Check Flux version:

```cmd
flux --version
```

Check Flux prerequisites:

```cmd
flux check --pre
```

Check Flux installation:

```cmd
flux check
```

List all Flux resources:

```cmd
flux get all -A
```

List Kustomizations:

```cmd
flux get kustomizations -A
```

List Git sources:

```cmd
flux get sources git -A
```

List Helm releases:

```cmd
flux get helmreleases -A
```

Watch Flux reconciliation:

```cmd
flux get all -A --watch
```

Check Flux namespace:

```cmd
kubectl get all -n flux-system
```

Check Flux logs:

```cmd
kubectl logs -n flux-system deploy/source-controller
```

```cmd
kubectl logs -n flux-system deploy/kustomize-controller
```

```cmd
kubectl logs -n flux-system deploy/helm-controller
```

---

# 20. Final Verification

Run the following commands after bootstrap:

```cmd
kubectl get nodes
```

```cmd
kubectl get pods -n flux-system
```

```cmd
flux check
```

```cmd
flux get sources git -A
```

```cmd
flux get kustomizations -A
```

```cmd
flux get all -A
```

If all Flux resources show `Ready=True`, the FluxCD GitOps setup is successfully configured.
