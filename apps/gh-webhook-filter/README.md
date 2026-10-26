# gh-webhook-filter (GitHub webhook interceptor)

Image: `ghcr.io/hummus-2/gh-webhook-filter:latest`, published by that repo's `image`
workflow on every push to `main` (pulled with the shared `ghcr` Secret; the repo is private).
Host: https://hooks.berkeley.mt — TLS via the `letsencrypt` ClusterIssuer (same as the other
`*.berkeley.mt` hosts).

Point the GitHub org/app webhook at `https://hooks.berkeley.mt/` instead of the real
consumer. Deliveries for repos in `BLOCK_REPOS` (currently `berkeleymt/bmt-2026`) are
answered `200 dropped` and never forwarded; everything else is proxied byte-for-byte
(headers included, so the consumer can still verify `X-Hub-Signature-256`) to
`UPSTREAM_URL`, and the consumer's response is relayed back to GitHub. If GitHub is pointed
at a sub-path, it is appended to `UPSTREAM_URL`. Stateless, no storage.
Health: `GET /healthz` → `ok`.

## Secrets (created out-of-band)

    kubectl create secret generic gh-webhook-filter -n toraora \
      --from-literal=UPSTREAM_URL='https://<where the webhooks used to go>' \
      --from-literal=WEBHOOK_SECRET='<github webhook secret, optional>'

`WEBHOOK_SECRET` is optional (drop the flag to skip signature checks). DNS:
`hooks.berkeley.mt` → the cluster ingress, like the other `*.berkeley.mt` records.

## Redeploy latest

    kubectl rollout restart deploy/gh-webhook-filter -n toraora
