 **POC on deploying NGINX using HelmRelease and exposing via Kong Ingress Controller using FluxCD**:
---

## 🚀 POC: Deploy `nginx-dev` App and Expose via Kong Ingress (FluxCD GitOps)

This proof-of-concept demonstrates how to deploy an NGINX application using FluxCD with HelmRelease and expose it publicly via **Kong Ingress Controller**.

---

### ✅ Prerequisites

- AKS Cluster with FluxCD installed and configured.
- Kong Ingress Controller is deployed in the `kong` namespace via FluxCD HelmRelease.
- You have a working Flux GitOps repository with the following layout.

---

### 📁 Folder Structure

```
flux-cd/
├── apps/
│   ├── nginx-dev/
│   │   ├── nginx-helmrelease.yaml       # HelmRelease for nginx-dev
│   │   ├── kong-ingress.yaml      # Ingress for exposing via Kong
│   │   └── namespace.yaml         # Namespace manifest
├── clusters/
│   └── my-cluster/
│       └── kustomization.yaml     # Cluster-level kustomization
```

---

### 📄 Step 1: `namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

---

### 📄 Step 2: `nginx-helmrelease.yaml`

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
    image:
      repository: nginx
      tag: "1.25.3"
    replicaCount: 1
    service:
      type: ClusterIP
      port: 80
    ingress:
      enabled: true
      ingressClassName: kong
      annotations:
        konghq.com/strip-path: "true"
      hosts:
        - host: kong-econ-test.southindia.cloudapp.azure.com
          paths:
            - path: /
              pathType: Prefix
```

---

### 📄 Step 3: `kong-ingress.yaml`

> This is optional if your Helm chart manages the ingress well. But if needed separately, you can apply this too.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-dev-kong
  namespace: dev
  annotations:
    konghq.com/strip-path: "true"
    konghq.com/override: "nginx-dev-override"
spec:
  ingressClassName: kong
  rules:
    - host: kong-econ-test.southindia.cloudapp.azure.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-dev
                port:
                  number: 80
```

---

### 📄 Step 4: `kustomization.yaml` in `apps/nginx-dev`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - nginx-helmrelease.yaml
  - kong-ingress.yaml
```

---

### 📄 Step 5: Register App with Main Flux Kustomization

In `clusters/my-cluster/kustomization.yaml`:

```yaml
resources:
  - ../../apps/nginx-dev
```

---

### 📦 Commit and Push to Git

```bash
cd flux-cd
git add .
git commit -m "Add nginx-dev app with Kong ingress"
git push origin main
```

---

### 🔍 Verify Deployment

After 1–2 minutes, check:

```bash
# Check Helm release status
flux get helmreleases -n dev

# Check ingress resource
kubectl get ingress -n dev

# Check service and pods
kubectl get svc -n dev
kubectl get pods -n dev

# Check Kong controller
kubectl get all -n kong
```

---

### 🌐 Access App

Visit:

```
http://kong-econ-test.southindia.cloudapp.azure.com
```

You should see the default **nginx welcome page**.

---

Let me know if you'd like to add TLS (Let’s Encrypt), rate limiting, custom Kong plugins, or basic auth!
