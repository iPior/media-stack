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

## Publish this existing server's configuration

For an empty GitHub repository, run these commands in a server shell with write
access to `.git`:

```bash
cd /opt/media-stack
git init -b main
git remote add origin https://github.com/iPior/media-stack.git
git add .
git diff --cached --stat
git diff --cached
git commit -m "Initialize media stack deployment configuration"
git push -u origin main
```

Review the staged diff before committing. Use your GitHub-supported authentication
when prompted. If the remote already contains commits, fetch and integrate its
history before pushing; do not force-push over existing work.

## Set up a new server

Install Git, Docker Engine, and the Docker Compose plugin. The Jellyfin stack
requires Linux with `/dev/net/tun` available for Gluetun. Portainer uses the local
Docker socket and its Enterprise image requires appropriate licensing.

```bash
git clone https://github.com/iPior/media-stack.git
cd media-stack
cp jellyfin/.env.example jellyfin/.env
cp immich/.env.example immich/.env
cp homepage/.env.example homepage/.env
cp adguard/.env.example adguard/.env
chmod 600 jellyfin/.env immich/.env homepage/.env adguard/.env
```

Edit the `.env` files before starting: set VPN credentials, a unique database
password, storage paths, timezone, Homepage's server hostname/allowed hosts,
and AdGuard's LAN/Tailscale interface addresses.
The root `.env.example` is optional; its
Cloudflare setting is not referenced by the current Compose files.

Prepare storage directories and permissions. Jellyfin and Navidrome run as
UID/GID `1000:1000`; their configuration and media mounts must be accessible to
that identity. Navidrome reads music from `navidrome/mixes/`. Gluetun currently
mounts the absolute host directory `/gluten`; provision that directory on the new
host or deliberately change the mount. Immich creates its library and database
under the paths specified in its environment file.

Review host ports and firewall access. Nginx Proxy Manager uses host networking;
its `ports` mappings do not control exposure in that mode. Replace its placeholder
admin credentials in `nginx/docker-compose.yml` before first startup. Review image
tags before pulling: several track `latest`, and Immich currently tracks `v3`.

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

## Save and deploy configuration changes

Validate each changed stack with `docker compose config --quiet`. Review the
staged diff for credentials before committing:

```bash
git add .
git diff --cached --stat
git diff --cached
git commit -m "jellyfin: update deployment configuration"
git push origin main
```

On another server, run `git pull --ff-only`, then validate and run
`docker compose up -d` in each affected directory. Run `docker compose pull` when
you intend to update image contents; check release and migration requirements
first. Existing local `.env` files are preserved; manually add any new variables
introduced by updated templates.

New deployment files must be explicitly added to `.gitignore`'s allowlist. Never
use `git add -f` to include runtime state or secrets. Keep independent backups of
important data and application settings before upgrades; Git is not that backup.
