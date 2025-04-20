
```markdown
# 🚀 FluxCD + Helm POC: Multi-Namespace NGINX Deployment on AKS

This proof of concept demonstrates GitOps using **FluxCD** to deploy NGINX applications to an **AKS cluster**, using both:

- A **remote Helm chart** (Bitnami NGINX)
- A **locally created Helm chart** (`nginx-dev`)

FluxCD watches your GitHub repo and applies changes automatically.

---

## 🧰 Prerequisites

Before starting:

- ✅ AKS Cluster set up and running
- ✅ FluxCD installed (with GitHub bootstrapping)
- ✅ GitHub repo (e.g. `flux-helm-repos`)
- ✅ GitHub PAT token (with repo access)
- ✅ `flux`, `kubectl`, `helm`, and `git` CLI tools installed

---

## 🔧 Step 1: Bootstrap FluxCD with GitHub Repo

```bash
flux bootstrap github \
  --owner=<your-github-username> \
  --repository=flux-helm-repos \
  --branch=main \
  --path=flux-cd/clusters/my-cluster \
  --personal
```

> 🔄 This will push:
> - `gotk-components.yaml`
> - `gotk-sync.yaml`

---

## 📁 Project Folder Structure

```
flux-cd/
├── apps/
│   ├── nginx/                    # For nginx-local-env (prod)
│   │   ├── namespace.yaml
│   │   ├── nginx-helmrepo.yaml
│   │   ├── nginx-helmrelease.yaml
│   │   └── kustomization.yaml
│   └── nginx-dev/                # For dev namespace (local chart)
│       ├── namespace.yaml
│       ├── nginx-helmrelease.yaml
│       └── kustomization.yaml
├── charts/
│   └── nginx-dev/                # Created by `helm create nginx-dev`
├── clusters/
│   └── my-cluster/
│       ├── flux-system/
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       └── kustomization.yaml   # Main app reference
```

---

## 📦 Step 2: Deploy Bitnami NGINX via Remote Helm Chart

#### Create files under `apps/nginx/`:

**namespace.yaml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-local-env
```

**nginx-helmrepo.yaml**
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

**nginx-helmrelease.yaml**
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
      version: "17.0.2"  # check ArtifactHub for latest
  values:
    service:
      type: LoadBalancer
```

**kustomization.yaml**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - nginx-helmrepo.yaml
  - nginx-helmrelease.yaml
```

---

## 🧪 Step 3: Deploy Local NGINX Chart to `dev` Namespace

### 3.1 Create Helm Chart
```bash
cd flux-cd/charts
helm create nginx-dev
```

Modify chart as needed in:
- `values.yaml`
- `templates/`

---

### 3.2 Create Files in `apps/nginx-dev/`

**namespace.yaml**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

**nginx-helmrelease.yaml**
```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: nginx-dev
  namespace: dev
spec:
  interval: 1m
  releaseName: nginx-dev
  chart:
    spec:
      chart: ./charts/nginx-dev
      sourceRef:
        kind: GitRepository
        name: flux-system
        namespace: flux-system
      interval: 1m
  values:
    service:
      type: LoadBalancer
```

**kustomization.yaml**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - nginx-helmrelease.yaml
```

---

## 🗂 Step 4: Update Main Kustomization

📄 `clusters/my-cluster/kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - flux-system
  - ../../apps/nginx
  - ../../apps/nginx-dev
```

---

## 🚀 Step 5: Push Everything to GitHub

```bash
cd flux-cd
git add .
git commit -m "Add dev and prod NGINX deployments using FluxCD"
git push origin flux-helm-repos
```

> ✅ Flux will auto-sync from GitHub within 1 minute

---

## 🔍 Step 6: Verify Deployment

```bash
flux get kustomizations
flux get helmreleases -A
kubectl get all -n nginx-local-env
kubectl get all -n dev
```

Expected output:

```bash
NAMESPACE         NAME         REVISION   SUSPENDED   READY   MESSAGE
nginx-local-env   nginx        17.0.2     False       True    Helm upgrade succeeded
dev               nginx-dev    1.16.0     False       True    Helm upgrade succeeded
```

To get the LoadBalancer IP:

```bash
kubectl get svc -n dev
kubectl get svc -n nginx-local-env
```

---

## 📌 Helm Chart Reference

- Bitnami NGINX Chart:  
  🔗 https://artifacthub.io/packages/helm/bitnami/nginx

You can update `nginx-helmrelease.yaml` chart version and just push to GitHub — Flux will redeploy automatically!

---

## 🔁 Notes

- All sync intervals are `1m`
- Namespace must be explicitly created (Flux doesn’t do it unless included)
- Local charts are referenced using `chart: ./charts/nginx-dev` and `sourceRef` to the Git repo

---

## ✅ Commands Cheat Sheet

```bash
# GitOps
git add . && git commit -m "..." && git push origin flux-helm-repos

# Flux Status
flux get kustomizations
flux get helmreleases -A

# Kubernetes Resources
kubectl get all -n dev
kubectl get all -n nginx-local-env
```

---

## 🙌 Done!
