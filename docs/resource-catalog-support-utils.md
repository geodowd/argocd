# resource-catalog-support-utils App Notes

This document explains what is in `apps/resource-catalog-support-utils/` and what gets created in Kubernetes.

## Folder Structure

```text
apps/resource-catalog-support-utils/
  base/
    kustomization.yaml
    namespace.yaml
    deployment.yaml
    service.yaml
    ingress.yaml
  envs/
    test/kustomization.yaml
    staging/kustomization.yaml
    prod/kustomization.yaml
```

## Base Resources

### `base/kustomization.yaml`

- Declares namespace: `resource-catalog-support-utils`.
- Includes namespace, deployment, service, and ingress manifests.

### `base/namespace.yaml`

- Creates namespace `resource-catalog-support-utils`.
- Adds annotation `linkerd.io/inject: enabled` for Linkerd sidecar injection.

### `base/deployment.yaml`

- Deploys one replica of `public.ecr.aws/eodh/resource-catalog-support-utils:main`.
- Exposes container port `8000` (named `http`).
- Uses readiness/liveness checks on `/health`.
- Defines CPU and memory requests/limits.

### `base/service.yaml`

- Creates an internal Kubernetes Service.
- Selects pods labeled `app: resource-catalog-support-utils`.
- Exposes service port `8000` and targets named container port `http`.

### `base/ingress.yaml`

- Uses ingress class `nginx`.
- Routes path `/api/resource-catalog-support-utils` to the service.
- Rewrites the request path before forwarding to the backend.
- Includes NGINX annotations/snippets for headers and CORS behavior.
- Uses host placeholder `${[.vars.platform.domain]}`.

## Environment Overlays

Each environment (`test`, `staging`, `prod`):

- References `../../base`.
- Overrides the image tag via `images`.
- Currently all three overlays use tag `main`.

## What Argo CD Creates on Sync

When Argo CD points at one of the env paths and syncs, it renders and applies:

1. Namespace
2. Deployment
3. Service
4. Ingress

Traffic flow is generally:

`Ingress -> Service -> Pod container (port 8000)`

## Important Note About Host Placeholder

`${[.vars.platform.domain]}` is not standard Kustomize variable syntax by itself.

This means one of the following must exist in your platform setup:

- Argo CD plugin or custom render step
- Separate template preprocessing
- Another substitution mechanism before apply

If no substitution step runs, ingress host configuration may not render as intended.
