## What does this PR do?

<!-- Brief description of the change -->

## Type of change

- [ ] Bug fix (incorrect YAML, broken command, outdated API)
- [ ] New manifest example
- [ ] Documentation improvement
- [ ] CI / tooling change

## Checklist

- [ ] Manifests validated with `kubeconform -strict -summary ./manifests/`
- [ ] New manifests include inline comments explaining *why*, not just *what*
- [ ] Resource requests/limits set on all containers
- [ ] Liveness and readiness probes included where applicable
- [ ] Security context hardened (non-root, read-only FS, capabilities dropped)
- [ ] No `latest` image tags used

## Related issue

Closes #
