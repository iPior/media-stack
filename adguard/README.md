# AdGuard Home

DNS filtering on the server's LAN and Tailscale addresses. DNS uses TCP/UDP port
53 on both interfaces. Setup and administration are exposed on Tailscale only.

## Set up a new server

Install and connect Tailscale first, then create the local environment file:

```bash
cd /opt/media-stack/adguard
cp .env.example .env
chmod 600 .env
```

Edit `ADGUARD_LAN_IP` and `ADGUARD_TAILSCALE_IP` to match addresses assigned to
that server. The example values match the original server and must be reviewed
on a new host. Ensure port 53 is available on those addresses before starting.

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
```

For first-time setup, open `http://<tailscale-ip>:3001`. In the setup wizard,
configure the web interface to listen on container port **80** and DNS on port
**53**, with listen addresses that accept container traffic. After setup, use
`http://<tailscale-ip>:8081` for administration. Configure LAN clients or your
router to use the LAN IP as their DNS server; configure Tailscale DNS separately
if desired.

## Configuration and backups

Git tracks the Compose file, environment template, and this guide. Local `.env`,
`conf/AdGuardHome.yaml`, and everything under `work/` remain excluded. AdGuard
writes its application settings to `conf/`; these may include credentials and
private network details. A clone starts a fresh instance requiring setup.

To preserve users, filters, upstream DNS choices, and other application settings,
keep a separate private backup of `conf/` and any required `work/` data. Stop
AdGuard before restoring its generated files so it cannot overwrite the restore,
and preserve their ownership and permissions. Review server-specific settings
before starting a restored instance.
