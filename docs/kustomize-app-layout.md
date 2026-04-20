# Kustomize App Layout

This repo uses a Kustomize layout pattern to define Kubernetes apps with shared base manifests and environment-specific overlays.

## Why Use This Layout

- Avoid duplicating full YAML sets per environment.
- Keep test/staging/prod consistent.
- Apply small, explicit overrides where needed (for example image tags).

## Typical Structure

```text
apps/<app-name>/
  base/
    kustomization.yaml
    deployment.yaml
    service.yaml
    ingress.yaml
    namespace.yaml
  envs/
    test/kustomization.yaml
    staging/kustomization.yaml
    prod/kustomization.yaml
```

## How It Works

1. `base/` contains reusable core resources.
2. Each `envs/<env>/kustomization.yaml` references `../../base`.
3. Environment overlays apply targeted differences (for example image tags).
4. Argo CD points to one env path and renders/applies the final manifests.

## Argo CD Behavior

When an Argo CD Application targets an overlay path containing `kustomization.yaml`:

- Argo CD runs Kustomize render for that path.
- The rendered resources are compared to live cluster state.
- Sync applies the rendered manifests.

## Useful Commands

Render an environment locally:

```bash
kustomize build apps/<app-name>/envs/test
```

Render via kubectl:

```bash
kubectl kustomize apps/<app-name>/envs/test
```

## Common Gotchas

- Base resources should stay generic; env-specific values belong in overlays.
- Ensure namespaces exist or are included in manifests.
- Non-standard template placeholders require an extra preprocessing/plugin step.
