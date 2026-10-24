# tr (browser fingerprint collector)

Image: `ghcr.io/hummus-2/tr:latest`, published by that repo's `image` workflow on every push to
`main` (pulled with the shared `ghcr` Secret). Host: https://tr.tora.dev — TLS via the
namespaced `letsencrypt-dns` Issuer, so the `cloudflare-dns-token` Secret must be able to write
`_acme-challenge.tr.tora.dev` in the `tora.dev` zone.

Fastify collector on Postgres (`tr-db` StatefulSet, 10Gi RWO). Ingest is `POST /status`
(`/tr/collect` also accepted); the dashboard is `/admin` (sign in with `ADMIN_TOKEN`; httpOnly cookie session) and the JSON APIs under
`/admin/api/*` take an `x-admin-token` header. Health: `GET /health` → `{"ok":true}`.
The collector creates its own schema on startup; there is no migration step.

Each property reverse-proxies `POST /status` from its own origin to this host and injects
`X-Tr-Site: <site>` (authoritative site id) plus `X-Tr-Client-Ip: <client ip>`;
`CLIENT_IP_HEADER=x-tr-client-ip` reads that (Cloudflare's `cf-connecting-ip` here would only be
the proxying property's address).

## Secrets (created out-of-band)

    kubectl create secret generic tr-db -n toraora \
      --from-literal=password='<pg-password>' \
      --from-literal=url='postgres://tr:<pg-password>@tr-db:5432/tr'

    kubectl create secret generic tr-env -n toraora \
      --from-literal=ADMIN_TOKEN='<admin-token>' \
      --from-literal=SECRET='<root-secret>'

IPs are stored raw (no `IP_HASH_SALT`) so cross-property matching can use them directly.

`SECRET` derives both the client `payloadKey` (`SECRET=… npx tr-mint --payload-key`) and the
honeypot token HMAC key, so rotating it invalidates existing `?k=` links.

## Redeploy latest

    kubectl rollout restart deploy/tr -n toraora
