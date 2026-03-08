# ArgoCD Template

This repository is a Copier template that scaffolds Kubernetes GitOps projects
using ArgoCD and Kustomize overlays.

## Goal Of The Template

The generated project is structured so teams can:

- Keep Kubernetes manifests under a single `k8s/` root
- Manage platform and ArgoCD config under `k8s/argocd/*`
- Manage app/workload manifests under `k8s/<project_slug>/*`
- Layer `sandbox`, `staging`, and `prod` cleanly with Kustomize overlays

## What ArgoCD Is

ArgoCD is a GitOps controller for Kubernetes. It watches Git and reconciles
cluster state to match declared manifests.

In generated projects, ArgoCD uses an app-of-apps model and deploys from:

- `k8s/argocd/overlays/<env>/apps`
- `k8s/<project_slug>/overlays/<env>`

## Rendered Project Structure

```text
k8s/
  argocd/
    base/
    overlays/{sandbox,staging,prod}
    bootstrap/
      Taskfile.yml
      argocd.yaml
      root-app.template.yaml
      env/{sandbox,staging,prod}.env
  <project_slug>/
    base/
    overlays/{sandbox,staging,prod}
    templates/
docs/
scripts/
```

## Use This Template

```bash
# From a local clone
copier copy . /path/to/new-project --trust

# Or from a remote repository URL
copier copy <template-repo-url> /path/to/new-project --trust
```

## In The Generated Project

- ArgoCD-specific onboarding is in `docs/argocd.md`
- Upgrade guidance is in `docs/upgrading.md`
- Semantic release setup (optional) is in `docs/semantic-release-github.md`

## Template Validation

Run local render checks with:

```bash

task test-template-render

```

This repository also includes `.github/workflows/template-correctness.yaml`, which runs the same check on every push/PR (for GitHub CI).
