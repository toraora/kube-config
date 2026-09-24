One directory per app. Minimal template:

```
apps/<name>/
  kustomization.yaml   # resources: [deployment.yaml, service.yaml, ingress.yaml]
  deployment.yaml
  service.yaml
  ingress.yaml
```

Then add `- apps/<name>` to the root `kustomization.yaml`.
