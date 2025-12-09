# Docker Installation

Docker is the recommended way to run Bambuddy. It handles all dependencies automatically and makes updates simple.

## Requirements

- Docker 20.10+
- Docker Compose v2+
- Bambu Lab printer with LAN Mode enabled
- Same network as your printer

## Quick Start

```bash
# Clone the repository
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy

# Start Bambuddy
docker compose up -d
```

Access Bambuddy at **http://localhost:8000**

## Configuration

### Change Port

Edit `docker-compose.yml` to use a different port:

```yaml
services:
  bambuddy:
    ports:
      - "3000:8000"  # Access on port 3000
```

### Set Timezone

```yaml
services:
  bambuddy:
    environment:
      - TZ=America/New_York  # Change to your timezone
```

### Custom Data Location

By default, data is stored in Docker volumes. To use a local directory:

```yaml
services:
  bambuddy:
    volumes:
      - ./data:/app/data
      - ./logs:/app/logs
```

## Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f

# Restart
docker compose restart

# Update to latest version
git pull
docker compose build
docker compose up -d

# Remove everything (including data)
docker compose down -v
```

## Building Manually

If you prefer to build without Docker Compose:

```bash
# Build the image
docker build -t bambuddy .

# Run the container
docker run -d \
  --name bambuddy \
  -p 8000:8000 \
  -v bambuddy_data:/app/data \
  -v bambuddy_logs:/app/logs \
  --restart unless-stopped \
  bambuddy
```

## Remote Access

Bambuddy is accessible from other devices on your network at:

```
http://<your-server-ip>:8000
```

Make sure:
1. Port 8000 (or your custom port) is open in your firewall
2. Your server and printer are on the same network

### Firewall (UFW)

```bash
sudo ufw allow 8000
```

### Firewall (firewalld)

```bash
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload
```

## Troubleshooting

### Build Fails with npm Permission Error

If you see `EACCES` errors during build, disable BuildKit:

```bash
DOCKER_BUILDKIT=0 docker compose build
```

### Container Exits Immediately

Check the logs:

```bash
docker compose logs bambuddy
```

### Can't Connect to Printer

Ensure the container can reach your printer:

```bash
docker exec bambuddy ping <printer-ip>
```

If using Docker Desktop on macOS/Windows, the container uses a different network. Make sure your printer is reachable from the host.

### Data Not Persisting

Check that volumes are mounted:

```bash
docker compose exec bambuddy ls -la /app/data
```

## Resource Usage

Typical resource consumption:

- **Memory**: ~150-300 MB
- **CPU**: Minimal (spikes during 3MF processing)
- **Disk**: Depends on archive size

---

[← Back to Installation](Installation.md) | [Getting Started →](Getting-Started.md)
