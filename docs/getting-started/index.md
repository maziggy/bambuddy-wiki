---
title: Getting Started
description: Get up and running with Bambuddy in minutes
---

# Getting Started

Welcome to Bambuddy! This guide will help you get your print archive up and running quickly.

---

## :rocket: Quick Install

=== ":material-docker: Docker (Recommended)"

    ```bash
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    docker compose up -d
    ```

    Open [http://localhost:8000](http://localhost:8000) in your browser.

    [:material-arrow-right: Full Docker Guide](docker.md)

=== ":material-language-python: Python"

    ```bash
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    python3 -m venv venv
    source venv/bin/activate
    pip install -r requirements.txt
    uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
    ```

    Open [http://localhost:8000](http://localhost:8000) in your browser.

    [:material-arrow-right: Full Installation Guide](installation.md)

=== ":material-microsoft-windows: Windows"

    ```cmd
    git clone https://github.com/maziggy/bambuddy.git
    cd bambuddy
    ./start_bambuddy.bat
    ```
    This batch file can be run very simply. It automatically builds the necessary components and launches bambuddy in your browser.

    * If you need to rebuild for any reason, use the `update` option. To clear the environment, use the `reset` option.

    * If you want to use the video functionality, you must install FFmpeg manually.
---

## :footprints: Next Steps

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-numeric-1-circle: Enable Developer Mode
Enable Developer Mode on your printer and note the access code.

[:material-arrow-right: See instructions](#enabling-developer-mode)
</div>

<div class="feature-card" markdown>
### :material-numeric-2-circle: Add Your Printer
Enter your printer's IP, access code, and serial number.

[:material-arrow-right: Add first printer](first-printer.md)
</div>

<div class="feature-card" markdown>
### :material-numeric-3-circle: Start Printing!
Bambuddy automatically archives every print.

[:material-arrow-right: Explore features](../features/index.md)
</div>

</div>

---

## Enabling Developer Mode

Bambuddy connects to your printer via **Developer Mode** - a local connection that provides full control without internet.

!!! info "Why Developer Mode?"
    Developer Mode provides direct communication between Bambuddy and your printer over your local network. This means:

    - :material-check: **Works offline** - No internet required
    - :material-check: **Full control** - Start/stop prints, upload files, control lights
    - :material-check: **Your data stays local** - No cloud dependency

!!! warning "Developer Mode vs LAN Only Mode"
    Since the January 2025 firmware update, standard LAN Only Mode (without Developer Mode) only provides **read-only** access. You can monitor your printer, but you cannot control it. **Developer Mode is required** for full functionality with Bambuddy.

### Step 1: Enable LAN Only Mode

1. On your printer's touchscreen, go to **Settings**
2. Navigate to **Network** or **WLAN**
3. Toggle **LAN Only Mode** to **ON**

### Step 2: Enable Developer Mode

1. After enabling LAN Only Mode, a **Developer Mode** option will appear
2. Toggle **Developer Mode** to **ON**
3. Note down the **Access Code** displayed (8 characters)

!!! warning "Access Code Changes"
    The access code changes every time you toggle these modes off and on. If you re-enable Developer Mode, you'll need to update the access code in Bambuddy.

### Step 3: Insert SD Card

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for Bambuddy to work properly. The SD card is required for:

    - :material-check: File transfers to/from the printer
    - :material-check: Starting prints from Bambuddy
    - :material-check: Archiving completed prints

    Without an SD card, Bambuddy cannot transfer files or start prints on your printer.

### Step 4: Gather Printer Information

You'll need these details to add your printer:

| Information | Where to Find |
|------------|---------------|
| **IP Address** | Settings :material-arrow-right: Network |
| **Serial Number** | Settings :material-arrow-right: Device Info |
| **Access Code** | Shown when Developer Mode is enabled |

---

## :checkered_flag: What's Next?

<div class="quick-start" markdown>

[:material-printer-3d: **Add Your Printer**<br><small>Connect your first printer</small>](first-printer.md)

[:material-archive: **Print Archiving**<br><small>How automatic archiving works</small>](../features/archiving.md)

[:material-bell-ring: **Notifications**<br><small>Get alerts on your phone</small>](../features/notifications.md)

[:material-help-circle: **Troubleshooting**<br><small>Having issues?</small>](../reference/troubleshooting.md)

</div>
