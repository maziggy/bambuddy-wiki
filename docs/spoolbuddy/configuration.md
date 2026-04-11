---
title: Configuration
description: Connecting SpoolBuddy to your Bambuddy instance
---

# Configuration

This page covers the runtime settings used by the SpoolBuddy daemon and kiosk.

---

## :material-key: Backend Connection

SpoolBuddy daemon reads its connection settings from:

- `/opt/bambuddy/spoolbuddy/.env` (default install path)

Minimum configuration:

```bash
SPOOLBUDDY_BACKEND_URL=http://192.168.1.100:8000
SPOOLBUDDY_API_KEY=bb_xxx
```

Optional:

```bash
SPOOLBUDDY_DEVICE_ID=sb-yourfixedid
SPOOLBUDDY_HOSTNAME=spoolbuddy-pi
```

After changes:

```bash
sudo systemctl restart spoolbuddy
```

!!! tip "API key source"
    Create API keys in **Bambuddy → Settings → API Keys**. Select **full permissions** when creating the key.

---

## :material-monitor-dashboard: Kiosk Access and Auth

The kiosk runs at:

- `/spoolbuddy`

Installer-generated kiosk URL usually includes a token:

- `/spoolbuddy?token=bb_xxx`

On first load, Bambuddy stores that token and removes it from the visible URL.

---

## :material-nfc: NFC Behavior

SpoolBuddy supports:

- **Bambu RFID tags** (MIFARE Classic; reads tray/material metadata)
- **NTAG tags** (`NTAG213/215/216`) for writable custom tags

### Tag Writing

Tag writing is available in kiosk UI at:

- `/spoolbuddy/write-tag`

Current write format:

- OpenTag3D-compatible NDEF payload (`application/opentag3d`)

---

## :material-scale: Scale and Calibration

Scale settings are managed from kiosk **Settings**:

- `Scale` tab for tare/calibration
- `Display` tab for brightness + blank timeout

Calibration flow:

1. Remove all weight and run **Set Zero / Tare**
2. Place known reference weight
3. Run **Calibrate**

Daemon receives and applies calibration via heartbeat, then persists through Bambuddy.

!!! tip "Use a 1–2 kg known weight for calibration"
    For a 5 kg load cell, calibration is more stable with heavier reference weights (about 20–40% of full scale).

---

## :material-sync: Device Sync Model

The daemon:

- registers device capabilities (NFC, scale, backlight)
- sends heartbeat periodically
- pushes tag scans and scale readings
- receives pending commands (for example `tare`, `write_tag`)

If heartbeat stops, device is marked offline after timeout.

### AMS Behavior Clarification

SpoolBuddy does **not** physically switch spools.  
It configures AMS slots with filament/K-profile information and works with Bambuddy's AMS/inventory workflows.

---

## :material-cog: Managing Devices from Bambuddy

Every registered SpoolBuddy kiosk appears under **Bambuddy → Settings → SpoolBuddy**.

The tab shows:

- **Connection status** — Online/Offline pill, last-seen time, daemon uptime.
- **System details** — hostname, device ID, IP address, firmware version, CPU temperature, memory usage, disk usage, OS/kernel/architecture, system uptime.
- **Hardware health** — NFC reader and scale status flags from the last heartbeat.
- **Device count badge** — the tab header shows how many devices are registered and whether at least one is online (green bullet) or all are offline (gray bullet).

### Unregistering a device

Use the **Unregister** button on a device card to remove it from the database. A confirm dialog asks before deletion.

A device that is still online will **re-register itself** on its next heartbeat, so unregistering an active kiosk is not permanently destructive — it will reappear within a few seconds.

### Crash-duplicate cleanup

If a SpoolBuddy daemon crashes during first-time registration or loses its persistent `device_id`, it may register itself a second time under a new ID. The kiosk UI uses the first registered device, so the duplicate sits unused but still shows up in the device list.

When more than one device is registered, the SpoolBuddy tab displays a yellow warning banner. Unregister the stale duplicate to clean up — the surviving online device continues to operate normally.

---

## :material-update: Updating

For normal Pi updates, a full reinstall is usually not required.

Typical flow:

```bash
git pull
sudo systemctl restart spoolbuddy
```

If your local checkout is outdated, update it first, then run the same restart.

---

## :material-alert: Troubleshooting

### Device stays offline

- Verify `SPOOLBUDDY_BACKEND_URL` and `SPOOLBUDDY_API_KEY`.
- Check service logs:

```bash
sudo journalctl -u spoolbuddy -f
```

### Two SpoolBuddy devices registered

After a daemon crash the kiosk may register itself a second time, leaving a stale duplicate in Bambuddy's device list. The kiosk UI silently uses only the first-registered device, so the duplicate is harmless functionally — but it's easy to miss.

Fix: open **Settings → SpoolBuddy** in Bambuddy. A yellow banner appears whenever more than one device is registered. Click **Unregister** on the stale entry and confirm. If the surviving device is online it will continue running without interruption.

### NFC not detected

- Confirm PN5180 wiring and SPI enablement
- Confirm `dtoverlay=spi0-0cs`.
- Run:

```bash
sudo /opt/bambuddy/spoolbuddy/venv/bin/python /opt/bambuddy/spoolbuddy/scripts/pn5180_diag.py
```

### Scale shows no readings

- Confirm NAU7802 on I2C bus 1:

```bash
sudo i2cdetect -y 1
```

- Run:

```bash
sudo /opt/bambuddy/spoolbuddy/venv/bin/python /opt/bambuddy/spoolbuddy/scripts/scale_diag.py
```

### Scale calibration is inconsistent

- Ensure the RFID reader is not mechanically/electrically interfering with the scale platform.
- If needed, relocate RFID placement away from the weigh path and recalibrate.

### Brightness controls do nothing

- HDMI displays are dimmed by frontend CSS
- Physical backlight writes require a DSI backlight device and `video` group permissions

### Display goes blank when connecting touch

- This can indicate display-side power/ground issues (for example ground loop-like behavior).
- Test with a different Pi/power source and re-check cabling and adapter orientation.
