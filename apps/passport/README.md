# passport

Image: `ghcr.io/toraora/passport:latest` (built by the passport repo's GitHub Actions).

Out-of-band secrets (never committed):

- `ghcr` — `kubectl create secret docker-registry ghcr --docker-server=ghcr.io --docker-username=<user> --docker-password=<PAT read:packages>`
- `passport-db` — keys `password`, `url` (`postgres://passport:<password>@passport-db:5432/passport`)
- `passport-env` — `DISCORD_CLIENT_ID`, `DISCORD_CLIENT_SECRET`, `DISCORD_GUILD_ID`, `DISCORD_BOT_TOKEN`,
  `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `ADMIN_EMAILS`, `ADMIN_DISCORD_IDS`

Redeploy latest image: `kubectl rollout restart deploy/passport -n toraora`.
