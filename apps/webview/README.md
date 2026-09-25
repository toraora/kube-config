# webview (JHMT 2027 problem-writing webview)

Image: `ghcr.io/atomicgrader/webview:latest` (pulled with the shared `ghcr` Secret).
Host: https://jhmt.poketwo.io — TLS via the cluster `letsencrypt` ClusterIssuer,
DNS auto-created by external-dns (poketwo.io).

The entrypoint clones `GIT_REPO_URL` into an emptyDir on every fresh pod, so
nothing needs persistent storage.

## Out-of-band Secret `webview`

Created manually, never committed:

- `WEBVIEW_PASSWORD` — shared login password
- `SECRET_KEY` — session cookie signing key (`openssl rand -hex 32`)
- `GITHUB_APP_ID`, `GITHUB_APP_INSTALLATION_ID`, `GITHUB_APP_PRIVATE_KEY` — GitHub App
  with Contents: read/write installed on jhmt-pw/jhmt-2027
- `DISCORD_CLIENT_SECRET`, `DISCORD_BOT_TOKEN` — optional, together with the Discord
  IDs in `configmap.yaml`

## Redeploy latest

    kubectl rollout restart deploy/webview -n toraora
