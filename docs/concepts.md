# Core Kubernetes Concepts

A practical explanation of how Kubernetes actually works — not just what the objects are, but why they exist and how they interact.

---

## The Big Picture

Kubernetes is a **container orchestration platform**. You describe the desired state of your system ("I want 3 replicas of this container"), and Kubernetes continuously reconciles actual state toward desired state.

This reconciliation loop is the heart of everything in Kubernetes.

```
You declare desired state
       ↓
API Server stores it in etcd
       ↓
Controllers watch for drift
       ↓
Controllers act to reconcile
       ↓
Actual state matches desired state
```

---

## Cluster Architecture

### Control Plane (the brain)

| Component | Role |
|-----------|------|
| **API Server** (`kube-apiserver`) | The single entry point. Every `kubectl` command hits this. Validates and persists objects to etcd. |
| **etcd** | Distributed key-value store. The only stateful component. Losing etcd = losing the cluster state. Back it up. |
| **Scheduler** (`kube-scheduler`) | Watches for unscheduled pods and assigns them to nodes based on resources, affinity, taints/tolerations. |
| **Controller Manager** (`kube-controller-manager`) | Runs all the built-in controllers (Deployment, ReplicaSet, Node, etc.) in one process. |
| **Cloud Controller Manager** | Integrates with cloud providers (AWS, GCP, Azure) for load balancers, node lifecycle, storage. |

### Worker Nodes (the muscle)

| Component | Role |
|-----------|------|
| **kubelet** | Agent on every node. Gets pod specs from the API server, talks to the container runtime to start/stop containers. |
| **kube-proxy** | Maintains iptables/IPVS rules for Service routing. Runs on every node. |
| **Container Runtime** | Actually runs containers. Common: containerd, CRI-O. Docker was deprecated as runtime in K8s 1.24. |

---

## The Object Model

Every Kubernetes resource is an object with the same structure:

```yaml
apiVersion: apps/v1      # API group + version
kind: Deployment         # Object type
metadata:
  name: my-app           # Unique name within namespace
  namespace: default     # Scope
  labels: {}             # Key-value tags (used by selectors)
  annotations: {}        # Metadata not used for selection
spec:                    # Desired state — YOU define this
  ...
status:                  # Actual state — Kubernetes writes this
  ...
```

You only write `spec`. Kubernetes populates `status`.

---

## Pods

A Pod is the smallest deployable unit — a group of one or more containers that share:
- **Network namespace** — same IP address, can communicate via `localhost`
- **Storage volumes** — can share files between containers in the pod
- **Lifecycle** — always scheduled together on the same node

### Why not just one container per pod?

The multi-container pattern exists for sidecar use cases:
- **Sidecar**: Enhances the main container (log shipper, metrics exporter)
- **Init container**: Runs once before the main container (DB migration, config setup)
- **Ambassador**: Proxies traffic for the main container (Envoy, Istio sidecar)

### Pod lifecycle

```
Pending → Running → Succeeded / Failed
                ↓
            (restart if restartPolicy allows)
```

Pods are **ephemeral** — they are never "moved". When a node fails, pods are terminated and new pods are created on another node. This is why you use Deployments, not bare Pods.

---

## Workload Controllers

Controllers wrap pods and add management logic.

### Deployment → ReplicaSet → Pod

```
Deployment (you manage this)
  └── ReplicaSet v1 (Deployment creates this)
        ├── Pod 1
        ├── Pod 2
        └── Pod 3
```

When you update a Deployment, it creates a new ReplicaSet and gradually scales it up while scaling the old one down. Old ReplicaSets are kept for rollback (`revisionHistoryLimit`).

**Never edit a ReplicaSet directly** — the Deployment controller will undo your changes.

### StatefulSet

Like a Deployment, but pods have:
- **Stable names**: `app-0`, `app-1`, `app-2`
- **Stable DNS**: `app-0.app.namespace.svc.cluster.local`
- **Stable storage**: each pod gets its own PVC from `volumeClaimTemplates`
- **Ordered operations**: creates in order `0→1→2`, deletes in reverse `2→1→0`

### DaemonSet

Ensures exactly one pod per node. When you add a node, the DaemonSet pod is automatically scheduled on it.

### Job / CronJob

- **Job**: Run a pod to completion (not forever). Retry on failure.
- **CronJob**: Create a Job on a schedule.

---

## Networking

### Services

Pods come and go — their IPs change. A Service provides a **stable virtual IP** (ClusterIP) that load-balances across matching pods.

```
Client → Service (stable IP) → kube-proxy (iptables/IPVS) → Pod A
                                                           → Pod B
                                                           → Pod C
```

The Service selects pods using `selector.matchLabels`. Any pod matching those labels gets added to the Service's **Endpoints** object.

### DNS

Every Service gets a DNS name: `<service>.<namespace>.svc.cluster.local`

Pods within the same namespace can use just `<service>`. Cross-namespace: `<service>.<namespace>`.

### Ingress

Services only route within the cluster. Ingress exposes HTTP(S) routes from outside the cluster to Services inside.

```
Internet → LoadBalancer (cloud) → Ingress Controller → Ingress rules → Service → Pods
```

The Ingress Controller (nginx, traefik, etc.) reads Ingress objects and configures its proxy rules accordingly.

---

## Storage

### The abstraction layers

```
Pod spec (volumeMount)
  └── Volume definition
        └── PersistentVolumeClaim (what the app requests)
              └── PersistentVolume (actual storage)
                    └── StorageClass (how to provision it)
                          └── Cloud provider / NFS / local disk
```

This abstraction means your pod spec doesn't need to know whether storage comes from AWS EBS, GCP PD, or NFS — it just requests a PVC.

### Volume types (quick reference)

| Type | Use case | Survives pod restart? | Survives pod delete? |
|------|----------|----------------------|---------------------|
| `emptyDir` | Temp scratch space, sharing between containers in pod | ✅ | ❌ |
| `hostPath` | Access node filesystem (logging agents) | ✅ | ❌ |
| `configMap` / `secret` | Inject config/secrets as files | ✅ | ❌ |
| `persistentVolumeClaim` | Durable application data | ✅ | ✅ |

---

## RBAC

Kubernetes RBAC answers: **"Can subject X perform action Y on resource Z?"**

```
Who?             What?          Where?
ServiceAccount → Role/ClusterRole (verbs + resources) → Namespace/Cluster
       ↑                 ↑
 RoleBinding ─────── binds them
```

### Verbs reference

| Verb | HTTP | Notes |
|------|------|-------|
| `get` | GET (single) | Read one resource |
| `list` | GET (collection) | Read all of a type |
| `watch` | GET + watch param | Stream change events |
| `create` | POST | Create new resource |
| `update` | PUT | Full replace |
| `patch` | PATCH | Partial update |
| `delete` | DELETE | Remove resource |

Minimum read access: `["get", "list", "watch"]`

---

## Scheduling

### How the scheduler picks a node

1. **Filtering** — eliminate nodes that don't meet hard requirements
   - Resource requests (not enough CPU/memory)
   - Node selectors / affinity rules
   - Taints the pod doesn't tolerate
   - Pod already running there (anti-affinity)

2. **Scoring** — rank remaining nodes
   - Prefer nodes with least resource usage
   - Prefer nodes already pulling the image
   - Spread pods across nodes/zones (topology)

### Taints & Tolerations

**Taints** repel pods from nodes. **Tolerations** allow pods to be scheduled despite taints.

```yaml
# Node taint (set by admin or cloud provider)
# kubectl taint nodes node1 dedicated=gpu:NoSchedule

# Pod toleration
tolerations:
  - key: dedicated
    operator: Equal
    value: gpu
    effect: NoSchedule
```

Common built-in taints:
- `node.kubernetes.io/not-ready` — node is unhealthy
- `node.kubernetes.io/unreachable` — node is unreachable
- `node-role.kubernetes.io/control-plane` — control plane node

---

## The Reconciliation Loop (deeper)

Every controller in Kubernetes follows the same pattern:

```
Watch API Server for objects of type X
  ↓
On change event:
  1. Get desired state from object spec
  2. Get actual state (observe the world)
  3. Compute diff
  4. Take action to reduce diff
  5. Update object status
  ↓
Repeat forever
```

This is why Kubernetes is **eventually consistent** and **self-healing**. If you manually delete a pod that belongs to a ReplicaSet, the ReplicaSet controller immediately notices the actual count dropped below desired, and creates a replacement.

---

## Common Pitfalls

**Pods in `Pending` state** → scheduler can't find a node
- Check: `kubectl describe pod <n>` → Events section
- Common causes: insufficient CPU/memory, node selector mismatch, PVC not bound

**Pods in `CrashLoopBackOff`** → container keeps crashing
- Check: `kubectl logs <pod> --previous` (logs from crashed container)
- Common causes: bad config, missing secret, liveness probe too aggressive

**Service not routing traffic** → selector mismatch
- Check: `kubectl get endpoints <service>` — should list pod IPs
- If empty: your pod labels don't match the Service selector

**Image pull errors** → `ErrImagePull` or `ImagePullBackOff`
- Check: image name/tag is correct, registry credentials exist
- For private registries: need an `imagePullSecret` on the pod
