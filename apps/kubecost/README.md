# Kubecost Argo CD Application

This folder contains the Argo CD `Application` manifest to install Kubecost using Argo CD multi-source.

## Files

- `application.yaml`: Argo CD multi-source app definition:
  - Helm chart source: `https://kubecost.github.io/cost-analyzer/` (`cost-analyzer` `2.9.6`)
  - Git values source: this repo (`base/values.yaml`)
- `namespace.yaml`: Optional explicit namespace manifest for `kubecost`.

## Apply

Apply both manifests:

```bash
kubectl apply -f apps/kubecost/namespace.yaml
kubectl apply -f apps/kubecost/federated-store-secret.yaml
kubectl apply -f apps/kubecost/application.yaml
```
