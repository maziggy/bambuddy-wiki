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

---

## :footprints: Next Steps

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-numeric-1-circle: Enable LAN Mode
Enable LAN Mode on your printer and note the access code.

[:material-arrow-right: See instructions](#enabling-lan-mode)
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

## Enabling LAN Mode

Bambuddy connects to your printer via **LAN Mode** - a local connection that works without internet.

!!! info "Why LAN Mode?"
    LAN Mode provides direct communication between Bambuddy and your printer over your local network. This means:

    - :material-check: **Works offline** - No internet required
    - :material-check: **Fast response** - Direct local connection
    - :material-check: **Your data stays local** - No cloud dependency

### Step 1: Enable Developer Mode (if required)

Some printer models require Developer Mode first:

1. On your printer's touchscreen, go to **Settings**
2. Navigate to **General** or **About**
3. Look for **Developer Mode** and enable it

!!! note
    Not all printers require this step. If you don't see Developer Mode, proceed to Step 2.

### Step 2: Enable LAN Mode

1. Go to **Settings** :material-arrow-right: **Network** :material-arrow-right: **LAN Mode**
2. Toggle **LAN Mode** to **ON**
3. Note down the **Access Code** displayed (8 characters)

!!! warning "Access Code Changes"
    The access code changes every time you toggle LAN Mode off and on. If you re-enable LAN Mode, you'll need to update the access code in Bambuddy.

### Step 3: Gather Printer Information

You'll need these details to add your printer:

| Information | Where to Find |
|------------|---------------|
| **IP Address** | Settings :material-arrow-right: Network |
| **Serial Number** | Settings :material-arrow-right: Device Info |
| **Access Code** | Shown when LAN Mode is enabled |

---

## :checkered_flag: What's Next?

<div class="quick-start" markdown>

[:material-printer-3d: **Add Your Printer**<br><small>Connect your first printer</small>](first-printer.md)

[:material-archive: **Print Archiving**<br><small>How automatic archiving works</small>](../features/archiving.md)

[:material-bell-ring: **Notifications**<br><small>Get alerts on your phone</small>](../features/notifications.md)

[:material-help-circle: **Troubleshooting**<br><small>Having issues?</small>](../reference/troubleshooting.md)

</div>
