```markdown
# 🔧 POC for Deploying Bitnami NGINX Helm Chart via FluxCD in AKS

This POC demonstrates how to deploy the NGINX Helm chart (maintained by Bitnami) into an AKS cluster using FluxCD. All resources are managed through GitOps by storing manifests in a GitHub repository.

---

This guide includes:

- Clear structure ✅  
- Prerequisites ✅  
- Step-by-step POC ✅  
- Directory tree ✅  
- Helm chart customization instructions ✅  
- Explanation of `kustomization.yaml` files ✅  

---

## ✅ Prerequisites

1. ✅ A GitHub **Personal Access Token (PAT)** with `repo` scope  
2. ✅ An **AKS cluster**  
3. ✅ **Flux CLI** installed and configured  
4. ✅ FluxCD **installed** in the AKS cluster  
5. ✅ Your GitHub repo is ready (e.g. `flux-cd`) and `flux-helm-repos` branch exists  

---

## 🚀 Step-by-Step Guide

### ✅ Step 1: Bootstrap FluxCD to GitHub and AKS

```bash
flux bootstrap github \
  --owner=your-github-username \
  --repository=flux-cd \
  --branch=flux-helm-repos \
  --path=clusters/my-cluster \
  --personal
```

➡️ This command installs Flux into your AKS cluster and syncs the GitHub repo.

### ✅ After Bootstrap, the following files are auto-generated:

```
flux-cd/
└── clusters/
    └── my-cluster/
        └── flux-system/
            ├── gotk-components.yaml
            └── gotk-sync.yaml
            ---- kustomization.yaml
```

---

### ✅ Step 2: Create Directory for NGINX Helm Deployment

```bash
mkdir -p flux-cd/apps/nginx
```

---

### ✅ Step 3: Add HelmRepository and HelmRelease

#### `nginx-helmrepo.yaml`

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: bitnami
  namespace: flux-system
spec:
  url: https://charts.bitnami.com/bitnami
  interval: 10m
```

#### `nginx-helmrelease.yaml`

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx
  namespace: nginx-local-env
spec:
  interval: 1m
  releaseName: nginx
  chart:
    spec:
      chart: nginx
      sourceRef:
        kind: HelmRepository
        name: bitnami
        namespace: flux-system
      version: "17.0.2"  # Changeable - see https://artifacthub.io/packages/helm/bitnami/nginx
  values:
    service:
      type: LoadBalancer
```

---

### ✅ Step 4: Add Namespace Manifest

#### `namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-local-env
```

---

### ✅ Step 5: Create `apps/nginx/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - nginx-helmrepo.yaml
  - nginx-helmrelease.yaml
```

---

### ✅ Step 6: Update Cluster Kustomization

#### `clusters/my-cluster/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - flux-system
  - ../../apps/nginx
```

---

### ✅ Step 7: Commit and Push Changes to GitHub

```bash
cd flux-cd
git add .
git commit -m "Add NGINX deployment using Bitnami Helm chart via FluxCD"
git push origin flux-helm-repos
```

⏱️ **Wait 1 minute** — Flux will automatically pick up changes.

---

## ✅ Step 8: Verify Deployment

```bash
flux get kustomizations
flux get helmreleases -A
kubectl get all -n nginx-local-env
kubectl get svc -n nginx-local-env
```

### ✅ Example Output:
```
NAMESPACE       NAME    REVISION        SUSPENDED   READY   MESSAGE
nginx-local-env nginx   17.0.2          False       True    Helm upgrade succeeded...
```

---

## 🔁 How to Change Helm Chart Version

1. Go to 👉 https://artifacthub.io/packages/helm/bitnami/nginx  
2. Copy the desired version (e.g. `17.3.5`)
3. Edit `nginx-helmrelease.yaml` → update `spec.chart.version`
4. Push changes to GitHub

Flux will **automatically redeploy** with the new version!

---

## 🗂️ Directory Structure (Final)

```
flux-cd/
├── apps/
│   └── nginx/
│       ├── namespace.yaml
│       ├── nginx-helmrepo.yaml
│       ├── nginx-helmrelease.yaml
│       └── kustomization.yaml
├── clusters/
│   └── my-cluster/
│       ├── flux-system/
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       └── kustomization.yaml
```

---

## 🧠 Short Difference Between the 3 Kustomization Files

| File Path                                       | Purpose                                                                 |
|------------------------------------------------|-------------------------------------------------------------------------|
| `clusters/my-cluster/kustomization.yaml`       | Main entry point for Flux. Includes app and flux-system directories.   |
| `clusters/my-cluster/flux-system/kustomization.yaml` | Manages core Flux components (`gotk-*.yaml`)                            |
| `apps/nginx/kustomization.yaml`                | Defines how to deploy NGINX using HelmRelease and HelmRepository       |

---

## ✅ Done!
