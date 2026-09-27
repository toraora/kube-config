# the-commons (card game: relay + browser client)

Image: `ghcr.io/casa-de-dos-gatos/the-commons:latest`, published by that repo's `image`
workflow on every push to `main` (pulled with the shared `ghcr` Secret — the token behind it
must be able to read casa-de-dos-gatos packages, since the repo is private).
Host: https://commons.sfbanorcalarml.org — TLS via the namespaced `letsencrypt-dns` Issuer.

Single FastAPI process serving the static client and the `/ws` relay. Rooms are in-memory,
so `replicas: 1` with `Recreate`; every rollout drops in-progress games. No secrets, no
storage. Health: `GET /healthz` → `ok rooms=N`.

## Redeploy latest

    kubectl rollout restart deploy/the-commons -n toraora
