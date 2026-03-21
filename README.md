# ☸️ k8s-mastery-guide

> A comprehensive, hands-on Kubernetes reference — from core concepts to production-grade patterns.

[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29+-326CE5?logo=kubernetes&logoColor=white)](https://kubernetes.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CI](https://github.com/joshivignesh/k8s-mastery-guide/actions/workflows/validate.yml/badge.svg)](https://github.com/joshivignesh/k8s-mastery-guide/actions)
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
│   ├── basics/
│   │   ├── pod.yaml                            # Annotated pod with security hardening
│   │   └── configmap-and-secret.yaml           # Config injection patterns
│   ├── deployments/
│   │   ├── deployment.yaml                     # Zero-downtime rolling deployment
│   │   ├── statefulset.yaml                    # PostgreSQL StatefulSet + headless Service
│   │   └── daemonset-cronjob.yaml              # Log shipper DaemonSet + backup CronJob
│   ├── networking/
│   │   └── service-ingress-networkpolicy.yaml  # Service, TLS Ingress, zero-trust NetworkPolicies
│   ├── storage/
│   │   └── storage.yaml                        # StorageClasses (AWS/GKE/Azure), PV, PVC patterns
│   ├── rbac/
│   │   └── rbac.yaml                           # ServiceAccount, Roles, ClusterRole, Bindings
│   └── monitoring/
│       ├── hpa-pdb-quota.yaml                  # HPA (v2), PDB, ResourceQuota, LimitRange
│       └── prometheus.yaml                     # ServiceMonitor, PrometheusRules, Grafana dashboard
├── helm/
│   └── demo-app/
│       ├── Chart.yaml
│       ├── values.yaml                         # Fully documented default values
│       ├── values/
│       │   ├── staging.yaml                    # Staging overrides
│       │   └── production.yaml                 # Production overrides
│       └── templates/
│           ├── _helpers.tpl                    # Reusable template helpers
│           ├── deployment.yaml
│           └── service-sa-hpa-pdb-ingress.yaml
├── docs/
│   ├── concepts.md        # Control plane, reconciliation loop, scheduling deep-dive
│   ├── patterns.md        # Production patterns & anti-patterns
│   ├── cheatsheet.md      # kubectl quick reference + aliases
│   └── troubleshooting.md # Systematic debugging guide (Pending, CrashLoop, OOMKill, RBAC...)
├── CONTRIBUTING.md
├── LICENSE
└── .github/
    ├── workflows/
    │   └── validate.yml   # CI: kubeconform + kube-score + yamllint
    ├── ISSUE_TEMPLATE/
    │   ├── bug_report.md
    │   └── feature_request.md
    └── PULL_REQUEST_TEMPLATE.md
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

- [Core Concepts](docs/concepts.md) — Control plane, reconciliation loop, scheduling, RBAC deep-dive
- [Production Patterns](docs/patterns.md) — Patterns that save you at 2am
- [Troubleshooting](docs/troubleshooting.md) — Systematic debugging: Pending, CrashLoop, OOMKill, networking, RBAC
- [Kubectl Cheatsheet](docs/cheatsheet.md) — Commands you'll use every day

## ⎈ Helm Chart

A production-ready Helm chart for  lives in [](helm/demo-app/):

```bash
# Install with defaults
helm install demo-app ./helm/demo-app --namespace default

# Install with environment-specific overrides
helm install demo-app ./helm/demo-app   -f helm/demo-app/values.yaml   -f helm/demo-app/values/production.yaml   --namespace production

# Preview rendered templates
helm template demo-app ./helm/demo-app -f helm/demo-app/values/staging.yaml
```

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
