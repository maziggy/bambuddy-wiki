---
title: Installation
description: Install Bambuddy on your system
---

# Installation

This guide covers all methods for installing Bambuddy on your system.

---

## :material-check-all: Requirements

Before you begin, ensure you have:

| Requirement | Details |
|------------|---------|
| **Python** | 3.10+ (3.11 or 3.12 recommended) |
| **Network** | Same LAN as your Bambu Lab printer |
| **Printer** | Developer Mode enabled ([see guide](index.md#enabling-developer-mode)) |
| **SD Card** | Inserted in the printer (required for file transfers) |

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for Bambuddy to function properly. File transfers, print uploads, and archiving all require the SD card.

!!! tip "Docker Alternative"
    If you prefer containers, check out the [Docker installation guide](docker.md) - it's even simpler!

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

=== ":material-microsoft-windows: Windows (Portable)"

    The easiest way to run Bambuddy on Windows - **no installation required**:

    ```batch
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy/install
    start_bambuddy.bat
    ```

    Or simply double-click `start_bambuddy.bat` after cloning.

    !!! success "What happens"
        On first run, the script automatically:

        - Downloads Python 3.13 and Node.js 22 (portable, no system changes)
        - Verifies downloads with SHA256 checksums
        - Creates a virtual environment and installs dependencies
        - Builds the frontend
        - Opens your browser to http://localhost:8000

    All files are stored in the `.portable\` folder - nothing is installed system-wide.

    **Commands:**

    | Command | Description |
    |---------|-------------|
    | `start_bambuddy.bat` | Launch Bambuddy |
    | `start_bambuddy.bat update` | Update dependencies and rebuild frontend |
    | `start_bambuddy.bat reset` | Clean everything and start fresh |
    | `set PORT=9000 & start_bambuddy.bat` | Run on a custom port |

    !!! tip "Requirements"
        - Windows 10 version 1803 or later (includes `curl` and `tar`)
        - Supports both x64 (Intel/AMD) and ARM64 Windows

=== ":material-microsoft-windows: Windows (Manual)"

    If you prefer to manage Python yourself:

    1. Download and install [Python 3.12](https://www.python.org/downloads/)

        !!! warning "Important"
            Check **"Add Python to PATH"** during installation!

    2. Open PowerShell and run:

    ```powershell
    # Clone and setup
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    python -m venv venv
    .\venv\Scripts\Activate.ps1
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

!!! tip "Accessing from Other Devices"
    To access Bambuddy from other devices on your network, use your server's IP address instead of `localhost`. For example: `http://192.168.1.100:8000`

---

## :material-update: Updating

### Manual Update

```bash
cd bambuddy
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
```

Then restart the service:

```bash
sudo systemctl restart bambuddy  # Linux
```

### Auto Updates

Bambuddy includes automatic update checking. Go to **Settings** to check for updates and apply them with one click.

!!! info "What Gets Updated"
    The auto-update feature:

    - :material-check: Downloads the latest release
    - :material-check: Updates Python dependencies
    - :material-check: Restarts the application
    - :material-close: Does NOT affect your database or settings

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
