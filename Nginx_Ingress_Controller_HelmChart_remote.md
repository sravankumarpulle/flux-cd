**FluxCD GitOps NGINX Ingress Controller** setup using a **remote Helm chart repo** on **Azure AKS**.

---

## 📘 Deploy NGINX Ingress Controller with FluxCD on AKS

This PoC demonstrates deploying the **NGINX Ingress Controller** using **FluxCD GitOps** and the **official Helm chart** from a **remote Helm repository** on an **Azure Kubernetes Service (AKS)** cluster.

---

### ✅ What’s Included

- Namespace: `nginx-ingress`
- Remote Helm chart: [ingress-nginx](https://kubernetes.github.io/ingress-nginx)
- HelmRelease managed by FluxCD
- LoadBalancer service for Azure public IP exposure
- GitOps-driven deployment using branch: `flux-helm-repos`
- Helm chart version `4.10.0` is used — you can check other versions on [ArtifactHub](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)

---

## 📁 Folder Structure

```
flux-cd/
├── apps/
│   └── ingress-nginx/
│       ├── namespace.yaml
│       ├── helmrepo.yaml
│       ├── helmrelease.yaml
│       └── kustomization.yaml
├── clusters/
│   └── my-cluster/
│       ├── flux-system/
│       └── kustomization.yaml
```

---

## 🧩 Step-by-Step Setup

### 1️⃣ Create Namespace

📄 `apps/ingress-nginx/namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-ingress
```

---

### 2️⃣ Create HelmRepository

📄 `apps/ingress-nginx/helmrepo.yaml`

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: HelmRepository
metadata:
  name: ingress-nginx
  namespace: flux-system
spec:
  url: https://kubernetes.github.io/ingress-nginx
  interval: 10m
```

---

### 3️⃣ Create HelmRelease

📄 `apps/ingress-nginx/helmrelease.yaml`

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: ingress-nginx
  namespace: nginx-ingress
spec:
  interval: 5m
  releaseName: ingress-nginx
  chart:
    spec:
      chart: ingress-nginx
      version: "4.10.0"  # ✅ Check for latest on ArtifactHub
      sourceRef:
        kind: HelmRepository
        name: ingress-nginx
        namespace: flux-system
  values:
    controller:
      replicaCount: 2
      service:
        type: LoadBalancer
```

---

### 4️⃣ Create App-Level Kustomization

📄 `apps/ingress-nginx/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - helmrepo.yaml
  - helmrelease.yaml
```

---

### 5️⃣ Update Cluster-Level Kustomization

📄 `clusters/my-cluster/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - flux-system
  - ../../apps/nginx
  - ../../apps/nginx-dev
  - ../../apps/ingress-nginx  # 👈 Include this line
```

---

## 🚀 Apply Changes

```bash
cd flux-cd
git add .
git commit -m "Add ingress-nginx via remote Helm chart using FluxCD"
git push origin flux-helm-repos
```

Flux will sync changes and deploy the ingress controller within a minute or two.

---

## ✅ Verify Deployment

Run the following commands:

```bash
flux get kustomizations
flux get helmreleases -n nginx-ingress
kubectl get all -n nginx-ingress
kubectl get svc -n nginx-ingress
```

Expected Output:

```
NAME                            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)                      AGE
ingress-nginx-controller        LoadBalancer   10.0.123.45     20.123.45.67     80:30876/TCP,443:30977/TCP   1m
```

---

## 🧠 Notes

- Helm chart version `4.10.0` is used — you can check other versions on [ArtifactHub](https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx)
- Ensure your Azure AKS cluster has permissions to create LoadBalancer resources
- Use `kubectl describe` to debug any pod or service issues

---

Let me know if you'd like to:

- Add **TLS support** for HTTPS ingress
- Deploy **sample apps** behind the ingress
- Add **ingress objects** via GitOps
- Convert this into a **GitHub template repo**

Want this zipped into a starter PoC package? I can do that too.
