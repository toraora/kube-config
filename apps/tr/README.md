# tr (browser fingerprint collector)

Image: `ghcr.io/toraora/tr:latest`, published by that repo's `image` workflow on every push to
`main` (pulled with the shared `ghcr` Secret). Host: https://tr.tora.dev — TLS via the
namespaced `letsencrypt-dns` Issuer, so the `cloudflare-dns-token` Secret must be able to write
`_acme-challenge.tr.tora.dev` in the `tora.dev` zone.

Fastify collector on Postgres (`tr-db` StatefulSet, 10Gi RWO). Ingest is `POST /status`
(`/tr/collect` also accepted); the dashboard is `/admin?token=…` and the JSON APIs under
`/admin/api/*` take an `x-admin-token` header. Health: `GET /health` → `{"ok":true}`.
The collector creates its own schema on startup; there is no migration step.

Each property reverse-proxies `POST /status` from its own origin to this host and injects
`X-Tr-Site: <site>` (authoritative site id). `CLIENT_IP_HEADER=cf-connecting-ip` is set because
Cloudflare fronts the host.

## Secrets (created out-of-band)

    kubectl create secret generic tr-db -n toraora \
      --from-literal=password='<pg-password>' \
      --from-literal=url='postgres://tr:<pg-password>@tr-db:5432/tr'

    kubectl create secret generic tr-env -n toraora \
      --from-literal=ADMIN_TOKEN='<admin-token>' \
      --from-literal=SECRET='<root-secret>' \
      --from-literal=IP_HASH_SALT='<salt>'      # optional: store hashed IPs instead of raw

`SECRET` derives both the client `payloadKey` (`SECRET=… npx tr-mint --payload-key`) and the
honeypot token HMAC key, so rotating it invalidates existing `?k=` links.

## Redeploy latest

    kubectl rollout restart deploy/tr -n toraora
