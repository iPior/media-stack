# Homepage

A dashboard for the existing media stack, available on port **2999**. Its service
links use the server hostname or LAN IP from a local environment file.

## Start

```bash
cd /opt/media-stack/homepage
cp .env.example .env
chmod 600 .env
```

Edit `.env` before starting. For a server at `192.168.1.50`, use:

```dotenv
HOMEPAGE_ALLOWED_HOSTS=192.168.1.50:2999,localhost:2999,127.0.0.1:2999
HOMEPAGE_VAR_SERVER_HOST=192.168.1.50
```

Match `PUID` and `PGID` to the owner of `config/` (check with `id -u`, `id -g`, and
`ls -ld config`). The default is `1000:1000`, like the existing media services.
Then start the dashboard:

```bash
docker compose config --quiet
docker compose up -d
docker compose ps
docker compose logs --tail=100 homepage
```

Open `http://192.168.1.50:2999`, substituting your server's address. If you change
the host port, update `HOMEPAGE_ALLOWED_HOSTS` to include that port too. Apply `.env`
changes with `docker compose up -d`.

## Customize

- `config/services.yaml`: grouped links to the existing service web interfaces.
  Replace individual `href` values if you use separate proxy domains.
- `config/settings.yaml`: title, colors, and layout.
- `config/widgets.yaml`: clock and search.
- `config/bookmarks.yaml`: optional extra links.

These files are tracked in Git. The local `.env`, generated logs, and other runtime
files are ignored. Use the dashboard refresh button after editing YAML. API
widgets can be added later; store their secrets in `.env` as `HOMEPAGE_VAR_*`
variables and pass them through Compose rather than writing keys into YAML.

## Nginx Proxy Manager

For a proxy hostname, forward to this server's LAN IP on port `2999` using HTTP.
Add the hostname (for example `homepage.example.com`) to `HOMEPAGE_ALLOWED_HOSTS`.
Enable TLS and configure access control in the proxy before exposing the dashboard
outside your trusted network. The allowed-hosts setting is not authentication.

The initial dashboard uses service links without mounting the Docker socket or
requiring API credentials. It does not display container status or application
statistics yet.

Reference: [Docker installation](https://gethomepage.dev/installation/docker/),
[allowed hosts](https://gethomepage.dev/installation/#homepage_allowed_hosts), and
[services configuration](https://gethomepage.dev/configs/services/).
