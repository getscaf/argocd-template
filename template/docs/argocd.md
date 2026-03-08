# ArgoCD Guide

This project uses ArgoCD and GitOps to manage Kubernetes environments.

## Goal

The goal is to keep manifests and deployment automation easy to reason about:

- ArgoCD platform and app-of-apps config under `k8s/argocd/*`
- Project-specific manifests under `k8s/{{ copier__project_slug }}/*`
- Kustomize overlays for `sandbox`, `staging`, and `prod`

## What ArgoCD Is

ArgoCD is a Kubernetes GitOps controller. It watches your Git repository and
syncs cluster state to match the manifests defined in Git.

In this project, ArgoCD is bootstrapped once and then manages ongoing sync for
platform components plus your project workload.

## Repository Layout

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
  {{ copier__project_slug }}/
    base/
    overlays/{sandbox,staging,prod}
    templates/
```

## Bootstrap ArgoCD

After cluster infrastructure is provisioned:

1. Change to bootstrap directory:

```bash
cd k8s/argocd/bootstrap
```

2. Select environment:

```bash
export ENV=sandbox
```

3. Run bootstrap:

```bash
task argocd:bootstrap
```

For detailed, step-by-step bootstrap guidance, see:

- `k8s/argocd/bootstrap/README.md`

## Local Commands

Initialize tools:

```bash
{% if copier__task_runner == "make" %}
make init
{% elif copier__task_runner == "just" %}
just init
{% else %}
task init
{% endif %}
```

Run checks:

```bash
{% if copier__task_runner == "make" %}
make check
{% elif copier__task_runner == "just" %}
just check
{% else %}
task check
{% endif %}
```

## Branch / Environment Defaults

- `sandbox` and `staging` track `develop`
- `prod` tracks `main`
