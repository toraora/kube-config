# kube-config

Declarative configuration for the `toraora` namespace on the `prod` cluster.

## Layout

```
base/          # namespace-wide resources (quotas, network policies, shared secrets refs)
apps/<name>/   # one Kustomize directory per app (Deployment, Service, Ingress, Certificate, ...)
kustomization.yaml  # root: aggregates base + apps
```

## Workflow

1. Edit or add manifests under `apps/<name>/` and register the directory in the root `kustomization.yaml`.
2. `kubectl diff -k .` to review, then `kubectl apply -k . --prune -l app.kubernetes.io/managed-by=kube-config`.
3. Commit and push so `main` always reflects what is running.

Deploys are run manually (by Devin) from a checkout of `main`; there is no CI.

## Access

The service account `toraora-admin` is namespace-scoped to `toraora`. It can manage workloads,
services, ingresses, cert-manager `Certificate`/`Issuer`, RabbitMQ CRs, PVCs, and secrets, but
cannot read cluster-scoped resources (nodes, ingressclasses, clusterissuers, storageclasses).

## Local

```
export KUBECONFIG=~/.kube/toraora-admin.kubeconfig
kubectl kustomize .
kubectl diff -k .
kubectl apply -k .
```

Never commit plaintext secrets. Reference them by name and create them out-of-band
(`kubectl create secret ...`) or via a sealed/external-secrets mechanism.
