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
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/docker-install.sh | bash
    ```

    The script will:

    - Prompt for install path, port, bind address, timezone
    - Download docker-compose.yml (or clone repo if building from source)
    - Create .env file with your settings
    - Start the container

    !!! tip "Unattended Mode"
        For automation, use the `--yes` flag:
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

Open [http://localhost:8000](http://localhost:8000) in your browser. :tada:

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
    network_mode: host  # Required for printer discovery
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
    Docker installations cannot use the in-app update feature. When an update is available, Bambuddy will show the commands below in Settings → Updates.

=== ":material-download: Pre-built Image"

    Simply pull the latest image:

    ```bash
    docker compose pull
    docker compose up -d
    ```

    Or as a one-liner:

    ```bash
    docker compose pull && docker compose up -d
    ```

=== ":material-source-branch: Built from Source"

    Pull changes and rebuild:

    ```bash
    cd bambuddy
    git pull
    docker compose build --pull
    docker compose up -d
    ```

    The `--pull` flag ensures you get the latest base image with security updates.

    One-liner:

    ```bash
    cd bambuddy && git pull && docker compose build --pull && docker compose up -d
    ```

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

### Reverse Proxy (Nginx)

To run Bambuddy behind a reverse proxy with HTTPS:

```nginx
server {
    listen 443 ssl http2;
    server_name bambuddy.yourdomain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_read_timeout 86400;
    }
}
```

!!! warning "WebSocket Support"
    Make sure your reverse proxy supports WebSocket connections - this is required for real-time printer updates.

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
    ports:
      - "${PORT:-8000}:8000"           # Use PORT=8080 docker compose up for custom port
      - "8883:8883"                    # Virtual printer MQTT (proxy mode)
      - "9990:9990"                    # Virtual printer FTP control
      - "50000-50100:50000-50100"      # Virtual printer FTP passive data
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
      - bambuddy_vprinter:/app/virtual_printer
    environment:
      - TZ=Europe/Berlin
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
  bambuddy_vprinter:
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
