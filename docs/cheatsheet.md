# kubectl Cheatsheet

Quick reference for the commands you'll use every day.

---

## Context & Cluster

```bash
kubectl config get-contexts                    # List all contexts
kubectl config use-context my-cluster          # Switch context
kubectl config set-context --current --namespace=staging  # Set default namespace
kubectl cluster-info                           # Show cluster endpoints
kubectl get nodes -o wide                      # Node details + IPs
```

---

## Pods

```bash
kubectl get pods -n <namespace> -o wide        # List pods with node/IP
kubectl get pods --all-namespaces              # All namespaces
kubectl describe pod <pod> -n <namespace>      # Full pod details + events
kubectl logs <pod> -f --tail=100               # Stream last 100 lines
kubectl logs <pod> -c <container>              # Specific container logs
kubectl logs <pod> --previous                  # Logs from crashed container
kubectl exec -it <pod> -- /bin/sh              # Shell into pod
kubectl exec -it <pod> -c <container> -- bash  # Shell into specific container
kubectl delete pod <pod> --grace-period=0      # Force delete stuck pod
```

---

## Deployments

```bash
kubectl get deployments -n <namespace>
kubectl rollout status deployment/<name>       # Watch rollout progress
kubectl rollout history deployment/<name>      # Show revision history
kubectl rollout undo deployment/<name>         # Rollback to previous revision
kubectl rollout undo deployment/<name> --to-revision=3  # Rollback to specific
kubectl scale deployment/<name> --replicas=5  # Manual scale
kubectl set image deployment/<name> app=image:tag  # Update image (triggers rollout)
```

---

## Debugging

```bash
# Run a debug pod in the cluster
kubectl run debug --image=nicolaka/netshoot --rm -it --restart=Never -- bash

# Port-forward to a pod or service
kubectl port-forward pod/<pod> 8080:8080
kubectl port-forward svc/<service> 8080:80

# Copy files to/from pod
kubectl cp <pod>:/var/log/app.log ./app.log
kubectl cp ./config.yaml <pod>:/etc/app/config.yaml

# Top (requires metrics-server)
kubectl top pods -n <namespace> --sort-by=cpu
kubectl top nodes

# Events (sorted by time)
kubectl get events --sort-by='.lastTimestamp' -n <namespace>

# Check RBAC permissions
kubectl auth can-i list pods --as system:serviceaccount:default:demo-app-sa
kubectl auth can-i --list --as system:serviceaccount:default:demo-app-sa
```

---

## Resource Inspection

```bash
# Get resource YAML / JSON
kubectl get pod <pod> -o yaml
kubectl get deployment <name> -o jsonpath='{.spec.replicas}'

# Diff what would change before applying
kubectl diff -f manifests/

# Dry-run (validate without applying)
kubectl apply -f manifests/ --dry-run=client
kubectl apply -f manifests/ --dry-run=server  # Server-side: more accurate

# Resource usage
kubectl describe node <node> | grep -A5 "Allocated resources"
```

---

## Namespaces & Secrets

```bash
kubectl create namespace staging
kubectl get secrets -n <namespace>
kubectl get secret <name> -o jsonpath='{.data.password}' | base64 --decode
```

---

## Useful Aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get services'
alias kgd='kubectl get deployments'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias klog='kubectl logs -f'
alias kexec='kubectl exec -it'

# Quickly switch namespace
kns() { kubectl config set-context --current --namespace="$1"; }

# Watch pods in real-time
alias kwatch='kubectl get pods -w'
```
