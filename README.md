# Media Stack

Docker Compose deployment configuration for a personal media server. Each directory
is an independent Compose project; there is no root Compose file.

| Directory | Services | Main interface |
| --- | --- | --- |
| `jellyfin/` | Jellyfin, Gluetun, qBittorrent, Radarr, Sonarr, Lidarr, Bazarr, Prowlarr, FlareSolverr, Seerr | Jellyfin: 8096; Seerr: 5055 |
| `immich/` | Immich server, machine learning, Valkey, PostgreSQL | 2283 |
| `nginx/` | Nginx Proxy Manager, GoAccess | Admin: 81; GoAccess: 7880 |
| `navidrome/` | Navidrome | 4533 |
| `portainer/` | Portainer Enterprise | 9443 |
| `homepage/` | Homepage dashboard | 2999 |
| `adguard/` | AdGuard Home DNS filtering | Tailscale setup: 3001; admin: 8081; DNS: 53 |

## What Git preserves

The `.gitignore` allowlist tracks Compose files, sanitized environment templates,
documentation, and the standalone Nginx site configuration in
`nginx/etc/nginx/sites-available/jellyfin`. That site file is a reference and is not
mounted by the current Compose definition; adapt its hostname and certificate paths
if installing it separately.

Local `.env` files, databases, downloaded media, photo libraries, certificates,
caches, and application state are excluded. Settings entered through application
interfaces (including proxy hosts, indexers, users, and library configuration) live
in that excluded state. A fresh clone recreates the containers, but those settings
must be configured again or restored from a separate backup.

## Folder Structure

Start each desired stack from its own directory:

```bash
cd jellyfin
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs --tail=100 gluetun
```

Repeat for `immich`, `nginx`, `navidrome`, `portainer`, and `homepage` as needed. See
[Homepage setup](homepage/README.md) for dashboard configuration. Verify VPN
connectivity and service health, then complete application setup in their web
interfaces. Do not start stacks on the existing server merely to initialize Git.

For DNS filtering, follow [AdGuard setup](adguard/README.md). Its interface-bound
ports require the configured LAN and Tailscale addresses to exist on the server.
