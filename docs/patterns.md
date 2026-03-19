# Production Patterns & Anti-Patterns

Hard-won lessons from running Kubernetes in production.

---

## ✅ Patterns

### 1. Always Set Resource Requests and Limits

Without requests, the scheduler can't make good placement decisions. Without limits, one noisy pod can OOM-kill its neighbors.

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

**QoS classes** (affects eviction order during resource pressure):
| Class | Condition | Evicted |
|-------|-----------|---------|
| Guaranteed | requests == limits | Last |
| Burstable | requests < limits | Middle |
| BestEffort | No requests/limits | First |

---

### 2. Use Startup + Liveness + Readiness Probes Together

Each probe has a distinct purpose:

| Probe | Triggers | Use for |
|-------|----------|---------|
| `startupProbe` | Container restart | Slow-starting apps (JVM, ML models) |
| `livenessProbe` | Container restart | Deadlock / hung process detection |
| `readinessProbe` | Remove from Service | "Not ready for traffic yet" |

**Never use a liveness probe for startup delay** — it creates restart loops.

---

### 3. Spread Replicas Across Nodes and Zones

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app: demo-app
```

Without this, your 3 replicas can land on the same node and all go down together.

---

### 4. Use PodDisruptionBudgets

Protect against cluster maintenance taking down all your pods:

```yaml
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: demo-app
```

---

### 5. Use Dedicated ServiceAccounts

Never use the `default` ServiceAccount. Create one per workload, and disable auto-mounting unless the pod actually needs API access:

```yaml
automountServiceAccountToken: false
```

---

### 6. Pin Image Tags

```yaml
# ❌ Never do this in production
image: nginx:latest

# ✅ Pin to a specific version
image: nginx:1.25.3-alpine

# ✅ Even better: pin to digest for immutability
image: nginx@sha256:abc123...
```

---

## ❌ Anti-Patterns

### Running as Root

```yaml
# ❌ Default — runs as root
spec:
  containers:
    - name: app
      image: myapp:1.0

# ✅ Hardened
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
    - name: app
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop: ["ALL"]
```

---

### Storing Secrets in ConfigMaps

ConfigMaps are not encrypted. Never store passwords, API keys, or tokens in them.

Use:
- Kubernetes Secrets (base + encryption at rest via KMS)
- [sealed-secrets](https://github.com/bitnami-labs/sealed-secrets) for GitOps
- [external-secrets-operator](https://external-secrets.io/) for Vault / AWS SSM / GCP Secret Manager

---

### Ignoring `terminationGracePeriodSeconds`

When a pod is deleted, Kubernetes sends `SIGTERM` and waits `terminationGracePeriodSeconds` before force-killing. If your app doesn't handle `SIGTERM`, in-flight requests get dropped.

```yaml
spec:
  terminationGracePeriodSeconds: 60  # Give app time to drain

# And in your app: handle SIGTERM to stop accepting new requests,
# finish existing ones, then exit cleanly.
```

---

### No Namespace Quotas

In a shared cluster, a rogue deployment can consume all cluster CPU/memory. Always apply `ResourceQuota` and `LimitRange` to every namespace.

---

### Forgetting to Handle Node Evictions

If you have `replicas: 1` and no PDB, a node drain during a cluster upgrade takes your service down. Even for non-critical apps, run at least 2 replicas and set a PDB.
