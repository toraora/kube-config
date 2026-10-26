# ag-demo — Atomic Grader demo at https://demo.atomicgrader.com

Separate copy of Atomic Grader (same `ghcr.io/atomicgrader/{otter,walrus}:latest`
images as prod) with `DJANGO_DEMO_GOLDEN_DIR=/assets/golden`, which mounts the
`/demo` page: an org admin can reset their org to any golden fixture under that
dir. Own Postgres, RabbitMQ, assets volume and secrets; nothing is shared with prod.

## Secrets (out-of-band)

```sh
kubectl -n toraora create secret generic ag-demo-db \
  --from-literal=password="$(openssl rand -hex 24)"
kubectl -n toraora create secret generic ag-demo-otter \
  --from-literal=DJANGO_SECRET_KEY="$(openssl rand -hex 50)" \
  --from-literal=DJANGO_JWT_SIGNING_KEY="$(openssl rand -hex 50)" \
  --from-literal=GEMINI_API_KEY=...        # uploads on the demo bill this key
```

`ag-demo-rabbitmq-default-user` is created by the RabbitMQ operator; `ghcr` is the
shared pull secret.

## Golden fixtures

Fixtures are `<name>/{fixture.json,media/}` produced by `manage.py demo_snapshot`
(or `otter/app/scripts/pull_prod_fixture.sh`). Copy them onto the assets volume:

```sh
POD=$(kubectl -n toraora get pod -l app=ag-demo-otter -o name | head -1)
kubectl -n toraora exec -i $POD -- sh -c 'mkdir -p /assets/golden && tar xzf - -C /assets/golden' < bmmt-2026-team.tar.gz
```

They show up on `/demo` immediately. First-time setup: create a superuser
(`kubectl exec ... -- python manage.py createsuperuser`), create an org in `/admin`
and add users to it; org admins then reset from `/demo`.
