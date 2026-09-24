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
2. Open a PR — CI runs `kubectl kustomize` + `kubectl diff` against the cluster.
3. Merge to `main` — CI runs `kubectl apply -k . --prune -l app.kubernetes.io/managed-by=kube-config`.

## Access

The service account `toraora-admin` is namespace-scoped to `toraora`. It can manage workloads,
services, ingresses, cert-manager `Certificate`/`Issuer`, RabbitMQ CRs, PVCs, and secrets, but
cannot read cluster-scoped resources (nodes, ingressclasses, clusterissuers, storageclasses).

CI authenticates with the `KUBECONFIG_B64` repository secret (base64 of the kubeconfig).

## Local

```
export KUBECONFIG=~/.kube/toraora-admin.kubeconfig
kubectl kustomize .
kubectl diff -k .
kubectl apply -k .
```

Never commit plaintext secrets. Reference them by name and create them out-of-band
(`kubectl create secret ...`) or via a sealed/external-secrets mechanism.
