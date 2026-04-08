# Kubernetes WordPress + MySQL Capstone Project

Production-style multi-tier Kubernetes deployment of WordPress and MySQL demonstrating core Kubernetes orchestration concepts.

---

## Architecture

Client
↓
NodePort Service
↓
WordPress Deployment (2 Replicas)
↓
MySQL StatefulSet
↓
Persistent Volume Claim

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

## Architecture

![alt text](docs/architecture-diagram.png.png)
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

## Screenshots

Add screenshots in `/screenshots`

---

## Author

**Shibnath Das**

DevOps / Cloud Infrastructure Engineer

---
