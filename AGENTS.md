# Repository Guidelines

## Project Structure & Module Organization

This repository configures a Docker Compose media stack; it has no application source tree or automated test suite. Each deployed stack has its own `docker-compose.yml`:

- `jellyfin/`: Jellyfin, Gluetun VPN, qBittorrent, media managers, and Seerr.
- `immich/`: photo server, machine learning, Valkey, and PostgreSQL.
- `nginx/`: Nginx Proxy Manager and GoAccess.
- `navidrome/`: music server and music mounts.
- `portainer/`: container management.

`homepage/` currently has no Compose definition. Service directories also contain persistent configuration, caches, databases, and media; treat these as runtime data, not source assets.

## Build, Test, and Development Commands

Run commands from the relevant service directory so Compose resolves its local `.env` and relative mounts correctly. For example, run `cd jellyfin` first.

- `docker compose config --quiet`: validate configuration and variable interpolation without printing resolved secrets.
- `docker compose pull`: download configured container images.
- `docker compose up -d`: create or update the stack in the background.
- `docker compose ps`: inspect container status.
- `docker compose logs --tail=100 <service>`: inspect a specific service's recent logs.

Services use published images; there is no local build command. Starting a stack can affect running services.

## Coding Style & Naming Conventions

Use two-space YAML indentation and no tabs. Keep service names lowercase and descriptive, preserving existing names and container references. Use uppercase snake case for environment variables, such as `DATA_LOCATION` and `IMMICH_VERSION`. Preserve read-only mounts where configured. No formatter or linter configuration is present.

## Testing Guidelines

Validate every changed Compose file with `docker compose config --quiet`. There is no test framework or coverage requirement. After an authorized deployment, check container status, health checks where available, logs, and the affected web interface. Preserve `network_mode: "service:gluetun"` for services routed through the VPN.

## Commit & Pull Request Guidelines

Git history is unavailable in this checkout. Use concise, imperative commit subjects identifying the stack, for example `jellyfin: update Seerr image`. Describe affected services, configuration changes, validation results, and any downtime or migration steps. Link relevant issues; include screenshots for visible interface changes.

## Security & Configuration

Keep credentials in local environment files; never commit secrets, database files, or media. Replace placeholder admin credentials before deployment. Back up persistent data before upgrades, and avoid `docker compose down -v` unless volume deletion is explicitly intended.
