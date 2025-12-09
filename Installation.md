# Installation

This guide covers all methods for installing Bambuddy on your system.

## Quick Install (Linux/macOS)

```bash
# Clone the repository
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install Python dependencies
pip install -r requirements.txt

# Start the server
uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```

Open http://localhost:8000 in your browser.

---

## Detailed Installation

### Step 1: Install Prerequisites

#### macOS
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Python and Node.js
brew install python@3.12 node
```

#### Ubuntu/Debian
```bash
sudo apt update
sudo apt install python3 python3-venv python3-pip nodejs npm git
```

#### Windows
1. Download and install [Python 3.12](https://www.python.org/downloads/) (check "Add to PATH")
2. Download and install [Node.js LTS](https://nodejs.org/)
3. Download and install [Git](https://git-scm.com/download/win)

### Step 2: Clone the Repository

```bash
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy
```

### Step 3: Set Up Python Environment

#### Linux/macOS
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

#### Windows (PowerShell)
```powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt
```

#### Windows (Command Prompt)
```cmd
python -m venv venv
venv\Scripts\activate.bat
pip install --upgrade pip
pip install -r requirements.txt
```

### Step 4: Build Frontend (Optional)

The repository includes pre-built frontend files in `/static`. To build from source:

```bash
cd frontend
npm install
npm run build
cd ..
```

### Step 5: Run the Application

```bash
uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```

Open http://localhost:8000 in your browser.

---

## Running as a Service (Linux)

Create a systemd service for automatic startup:

```bash
sudo nano /etc/systemd/system/bambuddy.service
```

Add the following content (adjust paths):

```ini
[Unit]
Description=Bambuddy Print Archive
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME/bambuddy
Environment="PATH=/home/YOUR_USERNAME/bambuddy/venv/bin"
ExecStart=/home/YOUR_USERNAME/bambuddy/venv/bin/uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable bambuddy
sudo systemctl start bambuddy

# Check status
sudo systemctl status bambuddy

# View logs
sudo journalctl -u bambuddy -f
```

---

## Docker Installation (Recommended)

Docker is the easiest way to run Bambuddy. See the full [Docker Guide](Docker.md) for detailed instructions.

```bash
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy
docker compose up -d
```

Open http://localhost:8000 in your browser.

---

## Environment Variables

Configure Bambuddy using environment variables or a `.env` file:

```bash
cp .env.example .env
```

| Variable | Default | Description |
|----------|---------|-------------|
| `DEBUG` | `false` | Enable debug mode (verbose logging, SQL queries) |
| `LOG_LEVEL` | `INFO` | Log level: `DEBUG`, `INFO`, `WARNING`, `ERROR` |
| `LOG_TO_FILE` | `true` | Write logs to `logs/bambuddy.log` |

### Production Settings (default)
- INFO level logging
- SQLAlchemy and HTTP library noise suppressed
- Logs written to `logs/bambuddy.log` (5MB rotating, 3 backups)

### Development Settings (`DEBUG=true`)
- DEBUG level logging (verbose)
- All SQL queries logged
- Useful for troubleshooting printer connections

Example `.env` for development:
```bash
DEBUG=true
LOG_TO_FILE=true
```

---

## Updating Bambuddy

### Manual Update
```bash
cd bambuddy
git pull origin main

# Activate virtual environment
source venv/bin/activate  # Linux/macOS
# or: .\venv\Scripts\Activate.ps1  # Windows PowerShell

# Update dependencies
pip install -r requirements.txt

# Rebuild frontend (if needed)
cd frontend
npm install
npm run build
cd ..

# Restart the application
```

### Auto Updates
Bambuddy includes automatic update checking. Go to **Settings** to check for updates and apply them with one click.

---

## Network Requirements

| Port | Protocol | Purpose |
|------|----------|---------|
| 8000 | HTTP | Bambuddy web interface |
| 8883 | MQTT/TLS | Printer communication |
| 990 | FTPS | File transfers |

Ensure your firewall allows these connections between Bambuddy and your printers.

---

## Next Steps

Once installed, proceed to [Getting Started](Getting-Started.md) to add your first printer.
