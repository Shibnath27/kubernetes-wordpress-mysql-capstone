# Kubernetes WordPress + MySQL Capstone Project

> Ten days. Twelve concepts. One production-grade deployment.

---
## Overview

This capstone project deploys a full **WordPress + MySQL** stack on Kubernetes. It integrates the core concepts from **Days 1–9 of the Kubernetes-Practice** repository into a single, production-style application. All resources are organized within the `capstone` namespace and managed via the `manifests/` directory.

---

### Concepts Used

| # | Concept | Resource |
|---|---------|----------|
| 1 | Namespace | `capstone` |
| 2 | Secret | `mysql-secret` |
| 3 | ConfigMap | `wordpress-config` |
| 4 | StatefulSet | `mysql` |
| 5 | Headless Service | `mysql` (clusterIP: None) |
| 6 | PersistentVolumeClaim | `mysql-data-mysql-0` (via volumeClaimTemplate) |
| 7 | Deployment | `wordpress` |
| 8 | NodePort Service | `wordpress` (port 30080) |
| 9 | Resource Requests & Limits | Both MySQL and WordPress |
| 10 | Liveness & Readiness Probes | Both MySQL and WordPress |
| 11 | HorizontalPodAutoscaler | `wordpress-hpa` |
| 12 | Helm (Bonus) | `bitnami/wordpress` in `wp-helm` namespace |

---

## Architecture

![alt text](./docs/architecture-diagram.png)

---

## Kubernetes Concepts Used

- Namespace Isolation
- Secrets
- ConfigMaps
- StatefulSets
- Headless Services
- Deployments
- NodePort Services
- Persistent Volume Claims
- Resource Requests & Limits
- Liveness & Readiness Probes
- Horizontal Pod Autoscaler

---

## Project Structure

```bash
kubernetes-wordpress-mysql-capstone/
│
├── README.md
│
├── manifests/
│   ├── namespace.yaml
│   ├── mysql-secret.yaml
│   ├── mysql-headless-service.yaml
│   ├── mysql-statefulset.yaml
│   ├── wordpress-configmap.yaml
│   ├── wordpress-deployment.yaml
│   ├── wordpress-service.yaml
│   └── hpa.yaml
│
├── screenshots/
│   ├── wordpress-ui.png
│   ├── kubectl-get-all.png
│   └── hpa-output.png
│
└── docs/
    └── architecture-diagram.png

```

---

## Deployment

```bash
kubectl apply -f manifests/
```

---

## Validation

### Check Resources

```bash
kubectl get all -n capstone
```

### Access WordPress

```bash
kubectl port-forward svc/wordpress 8080:80 -n capstone
```

Open:

```bash
http://localhost:8080
```

---

## Key Features Demonstrated

### Self-Healing
Deleting WordPress/MySQL pods automatically recreates them.

### Persistent Storage
MySQL data survives pod restarts.

### Horizontal Scaling
HPA scales WordPress pods based on CPU utilization.

---
##  Bonus: Compare with Helm

```
# Add Bitnami repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Install in a separate namespace
helm install wp-helm bitnami/wordpress \
  --namespace wp-helm \
  --create-namespace \
  --set wordpressUsername=admin \
  --set wordpressPassword=adminPass123

# See what Helm created
kubectl get all -n wp-helm | wc -l
```

### Comparison

| Aspect | Manual (12 files) | Helm (`bitnami/wordpress`) |
|--------|-------------------|----------------------------|
| Resources created | ~8 manifests, you control every line | 20+ resources auto-generated |
| Control | Total — you see and own every field | Partial — override via `values.yaml` |
| Repeatability | Script or Kustomize required | `helm install` is one command |
| Upgrades | Manual `kubectl apply` per file | `helm upgrade` with rollback |
| Learning value | ⭐⭐⭐⭐⭐ — understand every concept | ⭐⭐ — abstracted away |
| Production speed | Slow first time, fast with templates | Fast from day one |

**Verdict:** Helm is faster for getting something running, but you pay with opacity. Understanding the manual approach (this capstone) is what makes you dangerous with Helm — you know what it's doing under the hood.

```bash
# Clean up Helm deployment
helm uninstall wp-helm -n wp-helm
kubectl delete namespace wp-helm
```

---

## Key Lessons Learned

1. StatefulSet vs Deployment — When to use which

StatefulSet: Databases, message brokers, anything needing stable identity + dedicated storage
Deployment: Stateless web apps, APIs — anything where pods are interchangeable

2. envFrom vs secretKeyRef — Mixing both intentionally
Using envFrom for the ConfigMap (all vars) and secretKeyRef for individual Secret keys is a deliberate security pattern: ConfigMaps can be read by anyone with namespace access, while Secrets (ideally) have tighter RBAC. Picking individual keys from a Secret also avoids leaking unrelated credentials.
3. Probes — Timing matters
Setting initialDelaySeconds too low causes crash loops. WordPress takes ~30s to connect to MySQL and initialize. The liveness probe fires later (60s) than the readiness probe (30s) so a slow-starting pod isn't killed before it has a chance to become ready.
4. Headless Services — The StatefulSet DNS trick
The DNS name pod-name.service-name.namespace.svc.cluster.local only works because of the Headless Service (clusterIP: None). Without it, you'd have to use a ClusterIP VIP which load-balances across all pods — not what you want for a primary database.
5. HPA stabilization windows — Preventing flapping
Setting a short scale-up window (30s) and long scale-down window (300s) is a classic pattern. Traffic spikes are real; rapid scale-up protects the site. Brief lulls between spikes shouldn't trigger scale-in — the 5-minute window absorbs that noise.
6. Namespace as blast radius control
kubectl delete namespace capstone cleanly removed 12 resources across 8 resource types in one command. This is the operational payoff of namespace-scoping everything.

---

## Screenshots

Add screenshots in `/screenshots`

---

## Author

**Shibnath Das**

DevOps / Cloud Infrastructure Engineer

---
