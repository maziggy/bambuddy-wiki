---
title: Getting Started
description: Get up and running with Bambuddy in minutes
---

# Getting Started

> One command. Two minutes. Every print archived automatically.

---

## :rocket: Quick Install

=== ":material-docker: Docker (Recommended)"

    Interactive script — handles everything for you:

    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/docker-install.sh -o docker-install.sh && chmod +x docker-install.sh && ./docker-install.sh
    ```

    Open `http://localhost:8000` in your browser (substitute your server's hostname or IP if Bambuddy isn't on the same machine you're browsing from).

    [:material-arrow-right: Full Docker Guide](docker.md)

=== ":material-language-python: Native (Linux / macOS)"

    Interactive script — installs Python venv, builds the frontend, sets up a systemd/launchd service:

    ```bash
    curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh -o install.sh && chmod +x install.sh && ./install.sh
    ```

    Open `http://localhost:8000` in your browser (substitute your server's hostname or IP if Bambuddy isn't on the same machine you're browsing from).

    [:material-arrow-right: Full Installation Guide](installation.md)

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

### Step 4: Enable "Store sent files on external storage"

Bambuddy needs the slicer to leave a copy of the `.gcode.3mf` on the printer's SD card so it can pull thumbnails and slicer metadata back into the archive. There are two places this setting can live, depending on your slicer + firmware combination:

=== "In the slicer (older Bambu Studio / OrcaSlicer)"

    1. Open **Bambu Studio** or **OrcaSlicer**
    2. Go to the **Device** tab for your printer
    3. Enable **"Store sent files on external storage"**

=== "On the printer (newer firmware, e.g. P2S 01.02.00.00 + Bambu Studio 2.6+)"

    Recent firmware/slicer releases moved the toggle out of the slicer onto the printer itself. If the option is missing in Bambu Studio's Device tab, set it on the printer:

    1. On the printer, open **Settings :material-arrow-right: Print Settings** (or **Print Options**, depending on firmware)
    2. Enable the equivalent **"Store sent files on external storage"** option

    Reported on a P2S in [#1170](https://github.com/maziggy/bambuddy/issues/1170).

!!! info "Why is this needed?"
    Bambuddy extracts thumbnails and 3D model previews from the 3MF files on the SD card. Without this setting, every print falls back to a no-3MF archive: no thumbnail, no source 3MF, no slicer metadata.

### Step 5: Gather Printer Information

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
