# Kubernetes Mastery Guide 🚀

A structured, hands-on learning path through **Kubernetes** — from core concepts to production-grade patterns. Built for developers who want to go from zero to confidently deploying and operating containerised applications at scale.

## 📚 What's Covered

### 🟢 Beginner
- Kubernetes architecture — control plane, nodes, etcd
- Pods, ReplicaSets, Deployments
- Services: ClusterIP, NodePort, LoadBalancer
- ConfigMaps and Secrets
- kubectl essentials

### 🟡 Intermediate
- Namespaces and RBAC
- Persistent Volumes and Persistent Volume Claims
- Liveness and Readiness probes
- Horizontal Pod Autoscaling (HPA)
- Rolling updates and rollbacks
- Ingress controllers and TLS termination

### 🔴 Advanced
- Helm charts — packaging and deploying applications
- StatefulSets for databases
- DaemonSets for node-level workloads
- Custom Resource Definitions (CRDs)
- Kubernetes Operators
- Multi-cluster strategies
- GitOps with ArgoCD / Flux

## 🛠 Prerequisites

```bash
# Install kubectl
brew install kubectl          # macOS
choco install kubernetes-cli  # Windows

# Install minikube for local clusters
brew install minikube

# Start local cluster
minikube start

# Verify
kubectl get nodes
```

## 🗂 Repository Structure

```
k8s-mastery-guide/
├── 01-basics/
│   ├── pod.yaml
│   ├── deployment.yaml
│   └── service.yaml
├── 02-intermediate/
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── hpa.yaml
│   └── ingress.yaml
├── 03-advanced/
│   ├── helm/
│   ├── statefulset.yaml
│   ├── crd.yaml
│   └── argocd/
└── labs/
    └── real-world-scenarios/
```

## ⚡ Quick Start — Deploy a Sample App

```bash
# Deploy nginx
kubectl apply -f 01-basics/deployment.yaml

# Expose it
kubectl apply -f 01-basics/service.yaml

# Check status
kubectl get pods
kubectl get svc

# Access via minikube
minikube service nginx-service
```

## 🏗 CI/CD + DevOps Integration

| Tool | Purpose |
|---|---|
| **Docker** | Container image builds |
| **GitHub Actions** | CI pipeline — build, test, push to registry |
| **Helm** | Kubernetes package management |
| **ArgoCD** | GitOps continuous delivery to K8s |
| **Prometheus + Grafana** | Cluster monitoring and dashboards |

### GitHub Actions Pipeline Example

```yaml
name: Build & Deploy to K8s
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Push to registry
        run: docker push myregistry/myapp:${{ github.sha }}
      - name: Deploy to K8s
        run: kubectl set image deployment/myapp myapp=myregistry/myapp:${{ github.sha }}
```

## 📖 Key Concepts Reference

| Concept | Description |
|---|---|
| Pod | Smallest deployable unit — one or more containers |
| Deployment | Manages replica count and rolling updates |
| Service | Stable network endpoint to reach Pods |
| Ingress | HTTP routing + TLS at cluster edge |
| HPA | Auto-scales pods based on CPU/memory |
| Helm | Kubernetes package manager (like apt for K8s) |
| ArgoCD | Declarative GitOps CD for Kubernetes |

## 🔗 Author

**Vignesh Joshi** — Full Stack .NET Developer | DevOps Enthusiast  
[github.com/joshivignesh](https://github.com/joshivignesh)
