---
title: Docker Installation
description: Deploy Bambuddy with Docker in one command
---

# Docker Installation

Docker is the easiest way to run Bambuddy. One command and you're done!

---

## :rocket: Quick Start

```bash
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy
docker compose up -d
```

Open [http://localhost:8000](http://localhost:8000) in your browser. :tada:

---

## :material-cog: Configuration

### docker-compose.yml

The default `docker-compose.yml` works out of the box:

```yaml
services:
  bambuddy:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - ./bambuddy.db:/app/bambuddy.db
      - ./archive:/app/archive
      - ./logs:/app/logs
    restart: unless-stopped
    environment:
      - TZ=Europe/Berlin  # Your timezone
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `TZ` | `UTC` | Your timezone (e.g., `America/New_York`) |
| `DEBUG` | `false` | Enable debug logging |
| `LOG_LEVEL` | `INFO` | Log level: `DEBUG`, `INFO`, `WARNING`, `ERROR` |

### Custom Port

To run on a different port, modify the ports mapping:

```yaml
ports:
  - "3000:8000"  # Access on port 3000
```

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

### Pull Latest Changes

```bash
cd bambuddy
git pull origin main
docker compose build
docker compose up -d
```

### One-Liner Update

```bash
cd bambuddy && git pull && docker compose up -d --build
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

### Printer Discovery in Docker

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
