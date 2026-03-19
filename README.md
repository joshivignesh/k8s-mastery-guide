# ☸️ k8s-mastery-guide

> A comprehensive, hands-on Kubernetes reference — from core concepts to production-grade patterns.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29+-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/YOUR_USERNAME/k8s-mastery-guide/actions/workflows/validate.yml/badge.svg)](https://github.com/YOUR_USERNAME/k8s-mastery-guide/actions)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](CONTRIBUTING.md)

---

## 📌 What This Is

This repo is a structured, production-aware Kubernetes guide built for engineers who want to go beyond `kubectl apply` and understand **how and why** things work.

Every manifest is annotated. Every pattern is explained. Real-world gotchas included.

---

## 📂 Repo Structure

```
k8s-mastery-guide/
├── manifests/
│   ├── basics/          # Pods, ConfigMaps, Secrets, Namespaces
│   ├── deployments/     # Deployments, StatefulSets, DaemonSets, Jobs
│   ├── networking/      # Services, Ingress, NetworkPolicies
│   ├── storage/         # PVs, PVCs, StorageClasses
│   ├── rbac/            # Roles, ClusterRoles, ServiceAccounts
│   └── monitoring/      # Prometheus, HPA, resource limits
├── docs/
│   ├── concepts.md      # Core K8s concepts explained
│   ├── patterns.md      # Production patterns & anti-patterns
│   └── cheatsheet.md    # Quick reference commands
└── .github/
    └── workflows/       # CI: YAML validation, kubeconform
```

---

## 🗺️ Learning Path

| Level | Topics | Manifests |
|-------|--------|-----------|
| 🟢 **Beginner** | Pods, Namespaces, ConfigMaps, Secrets | `manifests/basics/` |
| 🟡 **Intermediate** | Deployments, Services, Ingress, RBAC | `manifests/deployments/`, `manifests/networking/` |
| 🔴 **Advanced** | StatefulSets, NetworkPolicies, HPA, Storage | `manifests/storage/`, `manifests/monitoring/` |

---

## 🚀 Quick Start

### Prerequisites
- [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.29+
- A local cluster: [minikube](https://minikube.sigs.k8s.io/), [kind](https://kind.sigs.k8s.io/), or [k3d](https://k3d.io/)
- (Optional) [kubeconform](https://github.com/yannh/kubeconform) for manifest validation

### Run the examples

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/k8s-mastery-guide.git
cd k8s-mastery-guide

# Start a local cluster (minikube)
minikube start

# Apply a basic pod
kubectl apply -f manifests/basics/pod.yaml

# Watch it come up
kubectl get pods -w

# Apply a full deployment with service
kubectl apply -f manifests/deployments/deployment.yaml
kubectl apply -f manifests/networking/service.yaml
```

---

## 🔑 Key Topics Covered

### 1. Core Primitives
- **Pods** — lifecycle, multi-container patterns, init containers
- **ConfigMaps & Secrets** — environment injection, volume mounts
- **Namespaces** — resource isolation and quota management

### 2. Workload Controllers
- **Deployments** — rolling updates, rollbacks, strategy tuning
- **StatefulSets** — ordered pod management, stable network identities
- **DaemonSets** — node-level agents (logging, monitoring)
- **Jobs & CronJobs** — batch processing, retry logic

### 3. Networking
- **Services** — ClusterIP, NodePort, LoadBalancer
- **Ingress** — host/path routing, TLS termination
- **NetworkPolicies** — zero-trust pod-to-pod communication

### 4. Storage
- **PersistentVolumes & Claims** — dynamic provisioning
- **StorageClasses** — cloud provider examples (EBS, GCE, Azure Disk)

### 5. Security & RBAC
- **Roles & ClusterRoles** — least-privilege access
- **ServiceAccounts** — workload identity
- **Pod Security** — securityContext best practices

### 6. Reliability
- **HorizontalPodAutoscaler** — CPU/memory-based scaling
- **Resource Requests & Limits** — QoS classes explained
- **Liveness & Readiness Probes** — health check patterns
- **PodDisruptionBudgets** — safe rolling upgrades

---

## 📖 Docs

- [Core Concepts](docs/concepts.md) — What is a Pod, Node, Control Plane, etcd?
- [Production Patterns](docs/patterns.md) — Patterns that save you at 2am
- [Kubectl Cheatsheet](docs/cheatsheet.md) — Commands you'll use every day

---

## ✅ CI / Validation

All manifests are automatically validated on every push using [kubeconform](https://github.com/yannh/kubeconform):

```bash
# Run locally
kubeconform -strict -summary ./manifests/
```

See [`.github/workflows/validate.yml`](.github/workflows/validate.yml) for the full pipeline.

---

## 🤝 Contributing

Found a bug? Want to add an example? See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## 📜 License

MIT — use freely, contribute back when you can.
