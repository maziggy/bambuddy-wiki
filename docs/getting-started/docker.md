---
title: Docker Installation
description: Deploy Bambuddy with Docker in one command
---

# Docker Installation

Docker is the easiest way to run Bambuddy. One command and you're done!

---

## :rocket: Quick Start

=== ":material-script: Install Script (Recommended)"

    Interactive script that prompts for configuration and sets everything up:

    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/docker-install.sh -o docker-install.sh && chmod +x docker-install.sh && ./docker-install.sh
    ```

    The script will:

    - Prompt for install path, port, bind address, timezone
    - Download docker-compose.yml (or clone repo if building from source)
    - Create .env file with your settings
    - Start the container

    !!! tip "Unattended Mode"
        For automation or CI, use the `--yes` flag to accept all defaults:
        ```bash
        curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/docker-install.sh | bash -s -- --yes
        ```

        Or customize with flags:
        ```bash
        curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/docker-install.sh | bash -s -- --path /srv/bambuddy --port 3000 --yes
        ```

=== ":material-download: Pre-built Image (Manual)"

    The fastest way - no building required:

    ```bash
    mkdir bambuddy && cd bambuddy
    curl -O https://raw.githubusercontent.com/maziggy/bambuddy/main/docker-compose.yml
    docker compose up -d
    ```

    !!! success "Multi-Architecture Support"
        Pre-built images are available for:

        - **linux/amd64** - Intel/AMD servers, desktops, most VPS
        - **linux/arm64** - Raspberry Pi 4/5, Apple Silicon Macs, AWS Graviton

        Docker automatically pulls the correct image for your system.

=== ":material-source-branch: Build from Source"

    If you want to build locally or modify the code:

    ```bash
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    docker compose up -d --build
    ```

Open `http://localhost:8000` in your browser (substitute your server's hostname or IP if Bambuddy isn't running on the same machine you're browsing from). :tada:

---

## :material-cog: Configuration

### docker-compose.yml

The default `docker-compose.yml` works out of the box:

```yaml
services:
  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    build: .
    # Usage:
    #   docker compose up -d          → pulls pre-built image
    #   docker compose up -d --build  → builds from source
    container_name: bambuddy
    network_mode: host  # Recommended for automatic printer discovery
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
    environment:
      - TZ=Europe/Berlin  # Your timezone
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
```

!!! info "Image vs Build"
    When both `image` and `build` are specified:

    - `docker compose up -d` pulls the pre-built image from `ghcr.io`
    - `docker compose up -d --build` builds locally from source

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TZ` | `UTC` | Your timezone (e.g., `America/New_York`) |
| `PORT` | `8000` | Port Bambuddy runs on (with host networking mode) |
| `DEBUG` | `false` | Enable debug logging |
| `LOG_LEVEL` | `INFO` | Log level: `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `HA_URL` | _(none)_ | Home Assistant URL for automatic integration (e.g., `http://192.168.1.100:8123`) |
| `HA_TOKEN` | _(none)_ | Home Assistant Long-Lived Access Token for automatic integration |
| `TRUSTED_FRAME_ORIGINS` | _(none)_ | Comma-separated origins permitted to embed Bambuddy via `<iframe>` (e.g., `http://homeassistant.local:8123`). Required for the HA Webpage dashboard panel. |
| `DATABASE_URL` | _(none)_ | External PostgreSQL connection string (e.g., `postgresql+asyncpg://user:pass@host:5432/bambuddy`). Uses built-in SQLite when not set. |

!!! info "Home Assistant Integration"
    When both `HA_URL` and `HA_TOKEN` are set, the Home Assistant integration is automatically enabled and configured. The URL and token fields become read-only in the UI. This is primarily used by the [Home Assistant add-on](https://github.com/hobbypunk90/homeassistant-addon-bambuddy/) for zero-configuration setup.

!!! tip "Embedding Bambuddy in Home Assistant's Webpage panel"
    By default, Bambuddy emits strict iframe-blocking headers (`X-Frame-Options: SAMEORIGIN` and CSP `frame-ancestors 'none'`) to protect against clickjacking on internet-exposed deployments. This blocks embedding Bambuddy inside Home Assistant's Webpage dashboard panel even on a trusted LAN, because HA on port 8123 and Bambuddy on port 8000 are different origins to the browser.

    To allow embedding from your HA instance, set:

    ```yaml
    environment:
      - TRUSTED_FRAME_ORIGINS=http://homeassistant.local:8123
    ```

    Replace the URL with your actual HA origin (`scheme://host[:port]` only — no paths, no wildcards). Multiple origins can be comma-separated. When set, `X-Frame-Options` is removed and the CSP `frame-ancestors` directive lists `'self'` plus your configured origins. Plain Docker / bare-metal deployments without this variable retain the strict default.

    **HTTPS Home Assistant (Nabu Casa, custom domain, anything with TLS in front):** browsers refuse to embed an HTTP iframe inside an HTTPS page (mixed-content block — Chrome and Firefox enforce this and the user can't override it for individual sites). If your HA instance is HTTPS, Bambuddy must also be reachable over HTTPS.

    The simplest recipe for HA users is the **[Nginx Proxy Manager addon](https://github.com/hassio-addons/addon-nginx-proxy-manager)** from the HA addon store. NPM does DNS-01 Let's Encrypt against your own domain, so no port-forwarding is required and Bambuddy never has to be exposed publicly:

    1. Install and start the Nginx Proxy Manager addon from the HA addon store.
    2. In NPM, add a **Proxy Host**: domain `bambuddy.<your-domain>`, scheme `http`, forward host `<bambuddy-host-on-LAN>`, forward port `8000`.
    3. Under **SSL**, request a Let's Encrypt cert via **DNS Challenge** for `bambuddy.<your-domain>` — NPM has dropdowns for Cloudflare, Hetzner, Route53, and many other DNS providers.
    4. Set `TRUSTED_FRAME_ORIGINS=https://homeassistant.<your-domain>` (or whatever your HA HTTPS origin is) on Bambuddy and restart.
    5. In HA, add a Webpage dashboard panel with `url: https://bambuddy.<your-domain>`.

    Both ends are HTTPS, the certificate is publicly trusted, and Bambuddy stays on the LAN — DNS-01 only needs API access to your DNS provider, not inbound connectivity. If you don't have a domain, any other reverse proxy works the same way (Caddy, Traefik, plain nginx outside the addon ecosystem) — the only requirement is that Bambuddy ends up behind a trusted HTTPS certificate that matches the URL you embed in the Webpage panel.

    **Path-prefixed reverse proxies** (e.g. `https://example.com/bambuddy/` via Traefik path prefix, nginx `location /bambuddy/`, Cloudflare Tunnel with path routing) are supported as of v0.2.4b2 — assets load correctly under any subpath. API calls still go to the host root, so the proxy must route `/api/v1/*` to the same Bambuddy upstream as `/bambuddy/*` (most users either reverse-proxy Bambuddy at a dedicated subdomain or expose `/api/v1/` at the host root alongside the subpath).

!!! warning "HA Ingress / addon-based subpath embedding is not supported"
    Home Assistant's add-on Ingress system serves the addon at a rotating per-session subpath (`/api/hassio_ingress/<token>/`). Even though the asset-path fix above lets the SPA boot under that prefix, the rest of the SPA still assumes a stable origin: API calls, React Router basename, PWA manifest scope, service-worker scope, and push-notification subscriptions are all anchored at the URL the SPA was first installed under. Making each of those subpath-aware would mean rewriting how the SPA bootstraps and would create new failure modes around PWA installs and deep-link reloads.

    The supported HA embedding path is the Webpage panel + `TRUSTED_FRAME_ORIGINS` flow above. Note that terminating TLS inside an HA addon container with a self-signed certificate is not a viable workaround: modern browsers (Chrome, Edge) block self-signed certificates on raw LAN IPs with no `thisisunsafe`-style override available. If you maintain an HA add-on that wraps Bambuddy, the practical path is to have the user front Bambuddy with the Nginx Proxy Manager addon (or any other reverse proxy that produces a publicly trusted certificate via DNS-01), then embed Bambuddy's HTTPS URL via Webpage panel.

!!! tip "External PostgreSQL Database"
    By default, Bambuddy uses a built-in SQLite database that requires zero configuration. For larger setups or when you prefer a dedicated database server, set `DATABASE_URL` to point to an external PostgreSQL instance:

    ```yaml
    environment:
      - DATABASE_URL=postgresql+asyncpg://bambuddy:yourpassword@db-host:5432/bambuddy
    ```

    Bambuddy will automatically create all tables on first startup. Backup/restore uses `pg_dump`/`pg_restore` instead of file copy.

### Custom Port

=== ":material-linux: Linux (Host Mode)"

    With `network_mode: host`, set the PORT environment variable:

    ```bash
    PORT=8080 docker compose up -d
    ```

    Or add it to your `docker-compose.yml`:

    ```yaml
    environment:
      - TZ=Europe/Berlin
      - PORT=8080
    ```

=== ":material-apple: macOS / :material-microsoft-windows: Windows"

    With port mapping, modify the ports section:

    ```yaml
    ports:
      - "8080:8000"  # Access on port 8080
    ```

!!! note "Linux Users: Permission Denied?"
    If you get "permission denied" errors, either prefix commands with `sudo` (e.g., `sudo docker compose up -d`) or [add your user to the docker group](https://docs.docker.com/engine/install/linux-postinstall/).

---

## :material-database: Data Persistence

Three volumes store your data:

| Volume | Purpose |
|--------|---------|
| `bambuddy.db` | SQLite database with all your print data |
| `archive/` | Archived 3MF files and thumbnails |
| `logs/` | Application logs |

!!! tip "Backup"
    To backup your data, simply copy these files/directories. See [Backup & Restore](../features/backup.md) for the built-in backup feature.

---

## :material-update: Updating

!!! info "In-App Updates Not Available"
    Docker installations cannot use the in-app update feature — upgrade from the command line.

!!! warning "Check your `image:` line first"
    If your `docker-compose.yml` pins a specific tag (e.g. `ghcr.io/maziggy/bambuddy:0.2.2.2`),
    `docker compose pull` will just re-fetch that same tag. Edit the line to
    `:latest` or the target version (e.g. `:0.2.3`) before pulling.

=== ":material-download: Pre-built Image"

    ```bash
    docker compose pull
    docker compose up -d
    ```

=== ":material-source-branch: Built from Source"

    ```bash
    cd bambuddy
    git pull
    docker compose build --pull
    docker compose up -d
    ```

    The `--pull` flag ensures you get the latest base image with security updates.

!!! tip "Stale `docker-compose.yml`?"
    Releases since 0.2.2 added `cap_add: NET_BIND_SERVICE`, extra virtual-printer
    ports for bridge mode (2024-2026), and an optional PostgreSQL block. If
    your compose file is older, bridge-mode users (macOS/Windows) may silently
    lose FTP and RTSP proxies. Compare yours against
    [the current file](https://github.com/maziggy/bambuddy/blob/main/docker-compose.yml)
    and merge by hand.

---

## :material-console: Useful Commands

### View Logs

```bash
# Follow logs
docker compose logs -f

# Last 100 lines
docker compose logs --tail 100
```

### Stop/Start

```bash
# Stop
docker compose down

# Start
docker compose up -d
```

### Rebuild (after changes)

```bash
docker compose up -d --build
```

### Shell Access

```bash
docker compose exec bambuddy /bin/bash
```

---

## :material-server: Advanced Setups

### Reverse Proxy (Nginx / Caddy)

See [Installation → Reverse Proxy](installation.md#reverse-proxy) — the Nginx and Caddy configs there work the same for Docker deployments. Just point the upstream at the Bambuddy container (`bambuddy:8000` on the same compose network, or `localhost:8000` if the proxy runs on the host).

### Traefik Labels

If you're using Traefik:

```yaml
services:
  bambuddy:
    build: .
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.bambuddy.rule=Host(`bambuddy.yourdomain.com`)"
      - "traefik.http.routers.bambuddy.entrypoints=websecure"
      - "traefik.http.routers.bambuddy.tls.certresolver=letsencrypt"
    # ... rest of config
```

### Network Mode Host

Host network mode is **required** for printer discovery and camera streaming:

```yaml
services:
  bambuddy:
    build: .
    network_mode: host
    # Note: ports mapping not needed with host mode
```

!!! warning "Required for Printer Discovery"
    Docker's default bridge networking cannot receive SSDP multicast packets needed for automatic printer discovery. You **must** use `network_mode: host` for discovery to work.

!!! note "When Using Host Mode"
    - Remove the `ports:` section (not needed with host mode)
    - Bambuddy will be accessible on port 8000 directly
    - All other features work the same

### macOS and Windows (Docker Desktop)

Docker Desktop on macOS and Windows runs containers inside a Linux VM, so `network_mode: host` connects to the VM's network, not your computer's network. This means:

- **Port 8000 won't be accessible** via localhost
- **Printer discovery won't work** (the container can't see your LAN)

**Solution:** Use port mapping instead of host mode:

```yaml
services:
  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    container_name: bambuddy
    # network_mode: host  # Doesn't work on macOS/Windows
    cap_add:
      - NET_BIND_SERVICE
    ports:
      - "${PORT:-8000}:8000"           # Use PORT=8080 docker compose up for custom port
      - "3000:3000"                    # Virtual printer bind/detect
      - "3002:3002"                    # Virtual printer bind/detect (alt port)
      - "990:990"                      # Virtual printer FTPS
      - "8883:8883"                    # Virtual printer MQTT
      - "6000:6000"                    # Virtual printer file transfer tunnel
      - "322:322"                      # Virtual printer RTSP camera (X1/H2/P2)
      - "2024-2026:2024-2026"          # Virtual printer proprietary ports (A1/P1S)
      - "50000-50100:50000-50100"      # Virtual printer FTP passive data
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
      # Share virtual printer certs with native installation
      - ./virtual_printer:/app/data/virtual_printer
    environment:
      - TZ=Europe/Berlin
      # Required for virtual printer FTP passive mode behind Docker NAT:
      # Set to your Docker host's LAN IP
      #- VIRTUAL_PRINTER_PASV_ADDRESS=192.168.1.100
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
```

!!! warning "Manual Printer Setup Required"
    On macOS/Windows, you must add printers manually by IP address. Automatic discovery is not available because Docker Desktop cannot access LAN multicast traffic.

### Printer Discovery in Docker

!!! note "Virtual Printer SSDP Discovery"
    SSDP discovery for the **virtual printer** (slicer discovering Bambuddy) also requires host networking or same-LAN connectivity. In Docker bridge mode (macOS/Windows), slicers must add the virtual printer manually by IP address.

When running in Docker with `network_mode: host`, Bambuddy uses **subnet scanning** instead of SSDP multicast for printer discovery:

1. Click **Add Printer** on the Printers page
2. Bambuddy detects it's running in Docker and shows a subnet input field
3. Enter your network range (e.g., `192.168.1.0/24`)
4. Click **Scan Network** - Bambuddy will probe each IP for Bambu printer ports (8883, 990)
5. Discovered printers appear in the list with their name and model

!!! tip "Finding Your Subnet"
    Your subnet is typically your IP address with the last number replaced by `0/24`. For example:

    - If your computer's IP is `192.168.1.50`, use `192.168.1.0/24`
    - If your computer's IP is `10.0.0.25`, use `10.0.0.0/24`

!!! info "How It Works"
    Subnet scanning checks each IP address in the range for open ports 8883 (MQTT) and 990 (FTPS). When both ports are open, it sends an SSDP query to get the printer's name, serial number, and model.

---

## :material-help-circle: Troubleshooting

### Container Won't Start

Check the logs:

```bash
docker compose logs bambuddy
```

Common issues:

- **Port in use**: Change the port mapping
- **Permission denied**: Check volume permissions

### Can't Connect to Printer

Ensure your Docker network can reach your printer:

```bash
# Test connectivity from inside container
docker compose exec bambuddy ping YOUR_PRINTER_IP
```

If using `bridge` network mode and having issues, try `network_mode: host`.

### Database Issues

If you need to reset the database:

```bash
docker compose down
rm bambuddy.db
docker compose up -d
```

!!! danger "Data Loss"
    This will delete all your print history and settings!

---

## :checkered_flag: Next Steps

<div class="quick-start" markdown>

[:material-printer-3d: **Add Your Printer**<br><small>Connect your first printer</small>](first-printer.md)

[:material-cellphone: **Mobile Access**<br><small>Access from your phone</small>](mobile.md)

[:material-help-circle: **Troubleshooting**<br><small>Having issues?</small>](../reference/troubleshooting.md)

</div>
