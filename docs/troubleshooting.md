# Troubleshooting Guide

A systematic approach to diagnosing the most common Kubernetes problems.

**Golden rule**: always start with `kubectl describe` and `kubectl logs`. 90% of answers are in the Events section.

---

## Quick Diagnostic Tree

```
Pod not running?
├── Pending        → Can't be scheduled (resources, affinity, PVC)
├── CrashLoopBackOff → App crashes on startup
├── ImagePullBackOff → Can't pull container image
├── OOMKilled      → Out of memory
├── Error          → Exited with non-zero code
└── Terminating    → Stuck finalizer or eviction

Service not routing?
├── No endpoints   → Selector mismatch
├── Connection refused → App not listening on expected port
└── Timeout        → NetworkPolicy blocking traffic

Ingress not working?
├── 404            → Ingress rule path mismatch
├── 502/503        → Backend Service or Pod unhealthy
└── TLS error      → cert-manager issue
```

---

## Pod Problems

### `Pending`

The scheduler cannot find a suitable node.

```bash
kubectl describe pod <pod-name> -n <namespace>
# Look at Events section at the bottom
```

**Common causes and fixes:**

| Event message | Cause | Fix |
|---|---|---|
| `Insufficient cpu/memory` | No node has enough resources | Scale node group, reduce requests, or check for resource hogs (`kubectl top nodes`) |
| `didn't match Pod's node affinity` | nodeSelector or affinity doesn't match any node | Check node labels: `kubectl get nodes --show-labels` |
| `node(s) had taint ... that the pod didn't tolerate` | Node is tainted | Add toleration to pod, or remove taint from node |
| `pod has unbound PersistentVolumeClaims` | PVC not bound | See PVC section below |
| `0/3 nodes are available` | Combination of above | Read the full message — it lists each reason |

```bash
# Check node capacity vs allocated
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Check what's consuming resources
kubectl top pods -A --sort-by=cpu
kubectl top pods -A --sort-by=memory
```

---

### `CrashLoopBackOff`

The container starts then crashes immediately, repeatedly.

```bash
# Get logs from the crashed container (--previous = last run)
kubectl logs <pod> --previous -n <namespace>

# If multiple containers
kubectl logs <pod> -c <container-name> --previous

# Describe to see restart count and last exit code
kubectl describe pod <pod>
# Look for: Last State, Exit Code, Reason
```

**Common causes:**

- **Exit code 1** — application error (check logs)
- **Exit code 137** — OOMKilled (increase memory limit)
- **Exit code 143** — SIGTERM not handled (fix graceful shutdown)
- **Exit code 2** — misuse of shell command (check entrypoint)
- **Missing config/secret** — app crashes because env var or mounted file is missing

```bash
# Check if secrets/configmaps exist
kubectl get secret <secret-name> -n <namespace>
kubectl get configmap <cm-name> -n <namespace>

# Decode a secret value to verify it's correct
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 --decode
```

**Aggressive liveness probe** — probe fires before app is ready and keeps restarting it:
```yaml
# Fix: use startupProbe to give the app time to start
startupProbe:
  failureThreshold: 30   # 30 * 10s = 5 minutes
  periodSeconds: 10
```

---

### `ImagePullBackOff` / `ErrImagePull`

Kubernetes can't pull the container image.

```bash
kubectl describe pod <pod> | grep -A 5 "Events"
```

**Common causes:**

```bash
# 1. Wrong image name or tag — typo in manifest
# Fix: verify the image exists
docker pull your-registry/your-app:tag

# 2. Private registry — missing imagePullSecret
kubectl create secret docker-registry registry-creds \
  --docker-server=your-registry.io \
  --docker-username=user \
  --docker-password=pass

# Then reference in pod spec:
# imagePullSecrets:
#   - name: registry-creds

# 3. Rate limiting (Docker Hub) — add authenticated pull secret
# 4. Registry unreachable from cluster — check network/firewall
```

---

### `OOMKilled`

Container exceeded its memory limit and was killed by the kernel.

```bash
kubectl describe pod <pod> | grep -A 3 "Last State"
# You'll see: Reason: OOMKilled
```

**Fixes:**

```bash
# Option 1: Increase memory limit (short-term)
kubectl set resources deployment/<name> --limits=memory=512Mi

# Option 2: Find the memory leak (long-term)
# Check memory trend in Grafana or:
kubectl top pods -n <namespace> --sort-by=memory

# Option 3: Check if app has unbounded caches / connection pools
# Profile with: kubectl exec -it <pod> -- jmap -histo <pid>  (Java)
```

---

### Pod stuck in `Terminating`

```bash
# Check for finalizers blocking deletion
kubectl get pod <pod> -o jsonpath='{.metadata.finalizers}'

# Force delete (last resort — may cause data inconsistency)
kubectl delete pod <pod> --grace-period=0 --force

# If a namespace is stuck terminating
kubectl get namespace <ns> -o json | \
  python3 -c "import sys,json; d=json.load(sys.stdin); d['spec']['finalizers']=[]; print(json.dumps(d))" | \
  kubectl replace --raw /api/v1/namespaces/<ns>/finalize -f -
```

---

## Service & Networking Problems

### Service has no endpoints

```bash
# Check endpoints (should list pod IPs)
kubectl get endpoints <service-name> -n <namespace>

# If NONE — selector mismatch
# Get the service selector
kubectl get svc <service-name> -o jsonpath='{.spec.selector}'

# Get pod labels
kubectl get pods -n <namespace> --show-labels

# They must match exactly
```

### Can't connect to Service from another pod

```bash
# Shell into a debug pod
kubectl run debug --image=nicolaka/netshoot --rm -it --restart=Never -- bash

# Test DNS resolution
nslookup <service>.<namespace>.svc.cluster.local

# Test TCP connectivity
curl http://<service>.<namespace>.svc.cluster.local:<port>/healthz

# If DNS works but connection fails — check NetworkPolicy
kubectl get networkpolicies -n <namespace>
kubectl describe networkpolicy <name>
```

### NetworkPolicy blocking traffic

```bash
# Temporarily remove all NetworkPolicies to isolate the issue
# (don't do in production — use a test namespace)
kubectl delete networkpolicies --all -n <test-namespace>

# Then re-apply one at a time to identify the culprit
```

---

## Ingress Problems

### 404 Not Found

The Ingress controller can't match the request to a rule.

```bash
# Check ingress rules
kubectl describe ingress <name> -n <namespace>

# Verify the backend service exists
kubectl get svc <backend-service> -n <namespace>

# Check ingress controller logs
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx --tail=50
```

### 502 Bad Gateway

The Ingress controller can reach the Service but gets an error from the pod.

```bash
# Check pod health
kubectl get pods -n <namespace>
kubectl logs <pod> -n <namespace>

# Check if readiness probe is passing
kubectl describe pod <pod> | grep -A 10 "Readiness"
```

### TLS / Certificate issues

```bash
# Check cert-manager certificate status
kubectl get certificate -n <namespace>
kubectl describe certificate <name> -n <namespace>

# Check if the TLS secret was created
kubectl get secret <tls-secret-name> -n <namespace>

# Check cert-manager logs for ACME challenges
kubectl logs -n cert-manager -l app=cert-manager --tail=50
```

---

## Storage Problems

### PVC stuck in `Pending`

```bash
kubectl describe pvc <pvc-name> -n <namespace>
# Look at Events

# Check StorageClass exists
kubectl get storageclass

# Check if provisioner pod is running (e.g., EBS CSI driver)
kubectl get pods -n kube-system | grep csi
```

**Common causes:**

| Reason | Fix |
|---|---|
| `no persistent volumes available for this claim` | Create a PV (static provisioning) or fix StorageClass |
| `storageclass not found` | Check StorageClass name spelling |
| `WaitForFirstConsumer` binding | PVC binds when pod is scheduled — this is normal |
| CSI driver not installed | Install the cloud provider CSI driver |

---

## RBAC Problems

```bash
# Test if a service account has permission
kubectl auth can-i list pods \
  --as system:serviceaccount:<namespace>:<serviceaccount-name> \
  -n <namespace>

# List all permissions for a service account
kubectl auth can-i --list \
  --as system:serviceaccount:<namespace>:<serviceaccount-name>

# Check which roles are bound to a service account
kubectl get rolebindings,clusterrolebindings -A \
  -o jsonpath='{range .items[?(@.subjects[0].name=="<serviceaccount>")]}{.metadata.name}{"\n"}{end}'
```

---

## Cluster-Level Issues

### Node NotReady

```bash
kubectl describe node <node-name>
# Look at Conditions section

# Check kubelet on the node (requires SSH to node)
systemctl status kubelet
journalctl -u kubelet -n 50

# Common causes: disk pressure, memory pressure, network issues
# Check node conditions:
kubectl get nodes -o wide
kubectl describe node <name> | grep -A 5 "Conditions"
```

### etcd issues (control plane)

```bash
# Check etcd health (if you have access to control plane)
kubectl -n kube-system exec -it etcd-<node> -- \
  etcdctl endpoint health \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
  --key=/etc/kubernetes/pki/etcd/healthcheck-client.key
```

---

## Useful One-Liners

```bash
# All pods not in Running state across all namespaces
kubectl get pods -A --field-selector=status.phase!=Running

# Events sorted by timestamp (last events first)
kubectl get events -A --sort-by='.lastTimestamp' | tail -30

# Pods on a specific node
kubectl get pods -A --field-selector spec.nodeName=<node-name>

# Find which pods are consuming the most CPU/memory
kubectl top pods -A --sort-by=cpu | head -20

# Describe all failing pods
kubectl get pods -A | grep -v Running | grep -v Completed | \
  awk 'NR>1 {print $1, $2}' | \
  xargs -n2 sh -c 'kubectl describe pod $2 -n $1' _

# Watch pod restarts in real time
watch -n 2 'kubectl get pods -A | sort -k5 -rn | head -20'

# Get all resources in a namespace (useful for auditing)
kubectl api-resources --verbs=list --namespaced -o name | \
  xargs -I{} kubectl get {} -n <namespace> --ignore-not-found
```
