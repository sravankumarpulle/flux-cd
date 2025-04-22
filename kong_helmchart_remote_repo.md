Let's deploy **Kong Ingress Controller** on AKS using **Helm charts with FluxCD GitOps** — clean, declarative, and easy to manage! 💥
---
## 🎯 Goal

Use **Kong's official Helm chart** to deploy **Kong Ingress Controller** on AKS using **FluxCD GitOps**, via a HelmRelease pointing to the official Helm repo.

---

## 📁 Final Folder Structure

```bash
flux-cd/
├── apps/
│   └── kong/
│       ├── namespace.yaml
│       ├── helmrepo.yaml
│       ├── helmrelease.yaml
│       └── kustomization.yaml
├── clusters/
│   └── my-cluster/
│       ├── flux-system/
│       │   └── gotk-sync.yaml
│       └── kustomization.yaml  # <-- Update this to include apps/kong
```

---

## 🔧 Step-by-Step YAML Setup

### 1️⃣ `apps/kong/namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kong
```

---

### 2️⃣ `apps/kong/helmrepo.yaml`

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: kong
  namespace: flux-system
spec:
  interval: 1m
  url: https://charts.konghq.com
```

---

### 3️⃣ `apps/kong/helmrelease.yaml`

```yaml
# Type Loadbalancer Automatically Public ip address created In Azure Portal
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: kong
  namespace: kong
spec:
  interval: 1m
  releaseName: kong
  chart:
    spec:
      chart: kong
      version: "2.37.0"  # 🔍 Check latest: https://artifacthub.io/packages/helm/kong/kong
      sourceRef:
        kind: HelmRepository
        name: kong
        namespace: flux-system
  values:
    ingressController:
      enabled: true
    proxy:
      type: LoadBalancer

# Loadbalancer  type Automatically Private ip address created inside Azure Vnet Configuration below
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: kong
  namespace: kong
spec:
  interval: 1m
  releaseName: kong
  chart:
    spec:
      chart: kong
      version: "2.37.0"  # 🔍 Check latest: https://artifacthub.io/packages/helm/kong/kong
      sourceRef:
        kind: HelmRepository
        name: kong
        namespace: flux-system
  values:
    ingressController:
      enabled: true

    proxy:
      type: LoadBalancer
      annotations:
        # 👉 Azure-specific annotation for internal LB
        service.beta.kubernetes.io/azure-load-balancer-internal: "true"
      http:
        enabled: true
        servicePort: 80
      tls:
        enabled: true
        servicePort: 443


```

> ☝️ You can modify `proxy.type` to `ClusterIP` or `NodePort` if not using a public LB.

---

### 4️⃣ `apps/kong/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - helmrepo.yaml
  - helmrelease.yaml
```

---

### 5️⃣ Update `clusters/my-cluster/kustomization.yaml`

Add the Kong deployment to the main Flux sync:

```yaml
resources:
  - flux-system
  - ../../apps/nginx
  - ../../apps/nginx-dev
  - ../../apps/kong         # 👈 Add this line
```

---

## 🚀 Deploy via Git

```bash
cd flux-cd
git add .
git commit -m "Deploy Kong Ingress Controller via Helm & FluxCD"
git push origin flux-helm-repos
```

---

## ✅ Verify Deployment

After 1–2 minutes:

```bash
flux get helmreleases -n kong
kubectl get all -n kong
kubectl get svc -n kong
```

You should see:

- Kong controller pods running
- `kong-proxy` with a LoadBalancer IP exposed

---

## 📚 Useful Links

- Helm Chart: https://github.com/Kong/charts
- ArtifactHub Kong: https://artifacthub.io/packages/helm/kong/kong
- Kong Docs: https://docs.konghq.com/kubernetes-ingress-controller/

---
