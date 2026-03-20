# Contributing to k8s-mastery-guide

Thanks for taking the time to contribute! This guide aims to be a high-quality, practical Kubernetes reference — contributions that improve clarity, correctness, or coverage are very welcome.

---

## Ways to Contribute

- **Fix a bug** — incorrect YAML, broken command, outdated API version
- **Improve docs** — clearer explanations, better examples, fixing typos
- **Add examples** — new manifest patterns with thorough annotations
- **Add a topic** — something missing from the learning path

---

## Getting Started

### 1. Fork and clone

```bash
git clone https://github.com/YOUR_USERNAME/k8s-mastery-guide.git
cd k8s-mastery-guide
```

### 2. Create a branch

```bash
git checkout -b feat/add-ingress-example
# or
git checkout -b fix/hpa-api-version
```

### 3. Make your changes

See the style guide below before writing new content.

### 4. Validate manifests locally

```bash
# Install kubeconform
brew install kubeconform  # macOS
# or download from https://github.com/yannh/kubeconform/releases

# Validate
kubeconform -strict -summary ./manifests/
```

### 5. Open a pull request

Write a clear PR description explaining what changed and why.

---

## Style Guide

### Manifests

- **Annotate everything** — explain *why*, not just *what*. A comment saying `replicas: 3` explains nothing. A comment explaining why 3 is chosen for HA is valuable.
- **Production-grade defaults** — resource requests/limits, probes, security context should be included unless the example is specifically about something else.
- **Consistent structure** — follow the pattern of existing files: comments at the top explaining the concept, then the YAML.
- **Use `---` separators** — multi-resource files should separate each resource with `---` and a comment explaining the next resource.
- **Pin versions** — never use `latest` image tags in examples.

### Docs

- **Practical over theoretical** — lead with the "so what" before the "what".
- **Include failure modes** — what goes wrong and how to debug it is as valuable as the happy path.
- **Use tables** for comparisons, reference information, and quick lookups.
- **Keep prose tight** — say it once, say it clearly.

---

## Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add Istio traffic management examples
fix: correct HPA apiVersion to autoscaling/v2
docs: expand RBAC section with debugging tips
chore: update kubeconform version in CI
```

---

## Code of Conduct

Be kind and constructive. Focus feedback on the contribution, not the contributor.
