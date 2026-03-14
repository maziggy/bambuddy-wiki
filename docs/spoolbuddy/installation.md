---
title: Installation
description: Setting up SpoolBuddy on your Raspberry Pi
---

# Installation

---

## :material-check-all: Prerequisites

Before running the installer:

- Raspberry Pi with Raspberry Pi OS (Debian-based), internet access, and `sudo`.
- Hardware wired (or ready to wire): PN5180 NFC + NAU7802 scale + display
- Existing Bambuddy server URL + API key (for SpoolBuddy-only mode), or plan to run full mode on the Pi

---

## :material-rocket: Recommended Install Method

Use the install script from the Bambuddy repository:

```bash
curl -fsSL https://raw.githubusercontent.com/maziggy/bambuddy/main/install/install.sh -o install.sh
chmod +x install.sh
sudo ./install.sh
```

The installer configures:

- SPI + I2C and required boot overlays
- System packages and Python venv
- SpoolBuddy daemon service (`spoolbuddy.service`)
- Touch kiosk environment (labwc + Chromium autostart + splash setup)
- Optional local Bambuddy service (full mode)

---

## :material-form-select: Installation Modes

For mode selection details and tradeoffs, see [Deployment Modes](deployment-modes.md).

### Unattended Examples

```bash
# SpoolBuddy-only mode
sudo ./install.sh --mode spoolbuddy --bambuddy-url http://192.168.1.100:8000 --api-key bb_xxx --yes

# Full local mode
sudo ./install.sh --mode full --port 8000 --yes
```

---

## :material-restart: First Boot After Install

A reboot is required after installation for:

- SPI/I2C and boot config changes
- kiosk auto-login/session startup
- splash/initramfs changes

```bash
sudo reboot
```

If you installed full mode, create an API key in **Bambuddy → Settings → [API Keys](../features/api-keys.md)**, then update `/opt/bambuddy/spoolbuddy/.env` with `SPOOLBUDDY_API_KEY=bb_xxx` and restart the service (`sudo systemctl restart spoolbuddy`).

---

## :material-application-cog: Service Management

```bash
# SpoolBuddy service status
sudo systemctl status spoolbuddy

# Follow SpoolBuddy logs
sudo journalctl -u spoolbuddy -f
```

If you installed full mode:

```bash
sudo systemctl status bambuddy
sudo journalctl -u bambuddy -f
```

---

## :material-clipboard-check: Verify the Setup

1. Open Bambuddy in a browser.
2. Open the kiosk route on the Pi: `/spoolbuddy`.
3. Confirm the device appears online in SpoolBuddy.
4. Place a known tag on the reader and confirm detection.
5. Check live scale value changes.

If hardware appears offline, run the diagnostics from [Hardware Requirements](hardware.md).

---

## :material-alert-circle: Common Pitfalls

- Wrong raw script URL path (use `.../bambuddy/main/install/install.sh`)
- Missing API key in SpoolBuddy-only mode
- I2C bus 0 not enabled (`dtparam=i2c_vc=on`)
- SPI CS overlay missing (`dtoverlay=spi0-0cs`)
- NFC/scale permissions not applied because reboot was skipped
