---
title: Installation
description: Install Bambuddy on your system
---

# Installation

> Get Bambuddy running in about a minute. One command, one browser tab — you're done.

---

## :material-rocket: Quick Install (Recommended)

The easiest way to install Bambuddy. Interactive scripts that handle everything for you.

=== ":material-linux: Linux"

    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh -o install.sh && chmod +x install.sh && ./install.sh
    ```

=== ":material-apple: macOS"

    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh -o install.sh && chmod +x install.sh && ./install.sh
    ```

The script will:

- Prompt for install path, port, bind address, timezone, and more
- Detect your package manager and install dependencies
- Set up Python virtual environment
- Build the frontend (Node.js 22)
- Create a systemd/launchd service
- Start Bambuddy automatically

!!! info "Supported Systems"
    - **Debian/Ubuntu** (apt)
    - **RHEL/Fedora/CentOS** (dnf/yum)
    - **Arch Linux** (pacman)
    - **openSUSE** (zypper)
    - **macOS** (Homebrew)

!!! tip "Unattended Mode"
    For automation or CI, use the `--yes` flag to accept all defaults:
    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh | bash -s -- --yes
    ```

    Or customize with flags:
    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh | bash -s -- --path /srv/bambuddy --port 3000 --yes
    ```

---

## :material-check-all: Requirements

Before you begin, make sure you have:

| Requirement | Details |
|------------|---------|
| **Python** | 3.10+ (3.11 or 3.12 recommended) — only needed for manual/native install; Docker doesn't need Python on the host |
| **Network** | Same LAN as your Bambu Lab printer |
| **Printer** | Developer Mode enabled ([see guide](index.md#enabling-developer-mode)) |
| **SD Card** | Inserted in the printer (required for file transfers) |

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for Bambuddy to function properly. File transfers, print uploads, and archiving all require the SD card.

!!! tip "Docker Alternative"
    If you prefer containers, check out the [Docker installation guide](docker.md) - it's even simpler!

---

## :material-download: Manual Install

Prefer to do it yourself? Follow these steps.

=== ":material-apple: macOS"

    ```bash
    # Install prerequisites (if needed)
    brew install python@3.12

    # Clone and setup
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt

    # Run
    uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
    ```

=== ":material-ubuntu: Ubuntu/Debian"

    ```bash
    # Install prerequisites
    sudo apt update
    sudo apt install python3 python3-venv python3-pip git

    # Clone and setup
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt

    # Run
    uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
    ```

Open [http://localhost:8000](http://localhost:8000) in your browser. :tada:

---

## :material-cog: Running as a Service

For production use, run Bambuddy as a system service that starts automatically.

=== ":material-linux: systemd (Linux)"

    Create the service file:

    ```bash
    sudo nano /etc/systemd/system/bambuddy.service
    ```

    Add this content (adjust paths for your system):

    ```ini
    [Unit]
    Description=BamBuddy Print Archive
    After=network.target

    [Service]
    Type=simple
    User=YOUR_USERNAME
    Group=YOUR_USERNAME
    WorkingDirectory=/home/YOUR_USERNAME/bambuddy
    Environment="PATH=/home/YOUR_USERNAME/bambuddy/venv/bin"

    # Kill any zombie ffmpeg processes before starting/after stopping
    # The - prefix ignores errors if no ffmpeg processes exist
    ExecStartPre=-/usr/bin/pkill -9 ffmpeg
    ExecStopPost=-/usr/bin/pkill -9 ffmpeg

    # Ensure directories exist and have correct permissions
    # The + prefix runs the command as root even though User= is set
    ExecStartPre=+/bin/mkdir -p /home/YOUR_USERNAME/bambuddy/logs
    ExecStartPre=+/bin/mkdir -p /home/YOUR_USERNAME/bambuddy/archive
    ExecStartPre=+/bin/chown -R YOUR_USERNAME:YOUR_USERNAME /home/YOUR_USERNAME/bambuddy/logs
    ExecStartPre=+/bin/chown -R YOUR_USERNAME:YOUR_USERNAME /home/YOUR_USERNAME/bambuddy/archive

    ExecStart=/home/YOUR_USERNAME/bambuddy/venv/bin/uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
    Restart=always
    RestartSec=10

    [Install]
    WantedBy=multi-user.target
    ```

    !!! tip "Why kill ffmpeg?"
        BamBuddy uses ffmpeg for camera streaming. Sometimes ffmpeg processes can become orphaned when the service restarts. The `ExecStartPre` and `ExecStopPost` commands ensure clean restarts.

    Enable and start:

    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable bambuddy
    sudo systemctl start bambuddy
    ```

    Check status:

    ```bash
    sudo systemctl status bambuddy
    ```

    View logs:

    ```bash
    sudo journalctl -u bambuddy -f
    ```

=== ":material-apple: launchd (macOS)"

    Create the plist file:

    ```bash
    nano ~/Library/LaunchAgents/com.bambuddy.plist
    ```

    Add this content:

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>Label</key>
        <string>com.bambuddy</string>
        <key>ProgramArguments</key>
        <array>
            <string>/Users/YOUR_USERNAME/bambuddy/venv/bin/uvicorn</string>
            <string>backend.app.main:app</string>
            <string>--host</string>
            <string>0.0.0.0</string>
            <string>--port</string>
            <string>8000</string>
        </array>
        <key>WorkingDirectory</key>
        <string>/Users/YOUR_USERNAME/bambuddy</string>
        <key>RunAtLoad</key>
        <true/>
        <key>KeepAlive</key>
        <true/>
    </dict>
    </plist>
    ```

    Load and start:

    ```bash
    launchctl load ~/Library/LaunchAgents/com.bambuddy.plist
    ```

---

## :material-tune: Configuration

Configure Bambuddy using environment variables or a `.env` file:

```bash
cp .env.example .env
nano .env
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `DEBUG` | `false` | Enable debug mode (verbose logging) |
| `LOG_LEVEL` | `INFO` | Log level: `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `LOG_TO_FILE` | `true` | Write logs to `logs/bambuddy.log` |

### Production Settings (default)

- INFO level logging
- SQLAlchemy and HTTP library noise suppressed
- Logs written to `logs/bambuddy.log` (5MB rotating, 3 backups)

### Development Settings

For debugging, create a `.env` file:

```bash
DEBUG=true
LOG_TO_FILE=true
```

This enables:

- DEBUG level logging (very verbose)
- All SQL queries logged
- Useful for troubleshooting printer connections

---

## :material-network: Network Requirements

Ensure your firewall allows these connections:

| Port | Protocol | Direction | Purpose |
|------|----------|-----------|---------|
| 8000 | HTTP | Inbound | Bambuddy web interface |
| 8883 | MQTT/TLS | Outbound | Printer communication |
| 990 | FTPS | Outbound | File transfers from printer |
| 2024-2026 | TCP | Outbound | Proprietary slicer ports (A1/P1S) |

!!! tip "Accessing from Other Devices"
    To access Bambuddy from other devices on your network, use your server's IP address instead of `localhost`. For example: `http://192.168.1.100:8000`

---

## :material-update: Updating

!!! warning "One-time note for 0.2.2.x → 0.2.3"
    The in-app **Update** button does not reliably perform this specific
    migration. Do this one upgrade from the command line using the paths
    below. Once you're on 0.2.3, the in-app Update button works normally
    again for all future releases.

### Recommended: `update.sh`

```bash
sudo /opt/bambuddy/install/update.sh
```

`update.sh` stops the service, snapshots the database via the built-in backup
API, fast-forwards to `origin/main`, installs Python dependencies, rebuilds
the frontend, and restarts the service. It rolls back automatically if any
step fails.

### Manual update

If you prefer to run the steps yourself:

```bash
cd /opt/bambuddy
sudo systemctl stop bambuddy
sudo -u bambuddy git fetch origin
sudo -u bambuddy git reset --hard origin/main
sudo -u bambuddy venv/bin/pip install -r requirements.txt
sudo systemctl start bambuddy
```

Replace `/opt/bambuddy` with your install directory if different. Database
schema migrations run automatically on startup.

!!! tip "Installed from a GitHub ZIP download?"
    Those installs have no `.git` directory and can't use either path above.
    See [UPDATING.md](https://github.com/maziggy/bambuddy/blob/main/UPDATING.md#installed-from-a-github-zip-or-tarball-download)
    for the backup-and-reinstall procedure.

---

## :material-folder-cog: Build Frontend from Source

The repository includes pre-built frontend files. To build from source:

```bash
# Install Node.js 18+ first
cd frontend
npm install
npm run build
cd ..
```

---

## :checkered_flag: Next Steps

Now that Bambuddy is installed:

<div class="quick-start" markdown>

[:material-printer-3d: **Add Your Printer**<br><small>Connect your first printer</small>](first-printer.md)

[:material-docker: **Try Docker Instead**<br><small>Even simpler setup</small>](docker.md)

[:material-help-circle: **Troubleshooting**<br><small>Installation issues?</small>](../reference/troubleshooting.md)

</div>
