---
title: Firmware Updates
description: Check and upload firmware updates for LAN-only printers
---

# Firmware Updates

Check and upload firmware updates for printers operating in LAN-only mode.

---

## :material-update: Overview

Bambu Lab printers typically receive firmware updates through Bambu Cloud. For printers operating in **LAN-only mode**, this feature provides an alternative way to update firmware:

- **Check** for available firmware updates
- **Download** firmware from Bambu Lab servers
- **Upload** directly to printer's SD card via FTP
- **Trigger** update from printer's touchscreen

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for firmware updates. The firmware file is uploaded to the SD card via FTP.

!!! info "LAN-Only Mode"
    This feature is designed for printers not connected to Bambu Cloud. If your printer is cloud-connected, use the standard OTA update method.

---

## :material-bell-badge: Update Notifications

### Printer Card Badge

When a firmware update is available, an orange **Update** badge appears on the printer card:

- Badge shows download icon + "Update" text
- Hover for version info (current → latest)
- Click to open the firmware update modal

### Automatic Checking

Firmware versions are checked automatically:

- On printer connection
- Cached for 5 minutes to reduce API calls
- Compares installed version against Bambu Lab's latest

### Disabling Firmware Checks

You can disable firmware update checks in **Settings → General → Updates**:

- Toggle **"Check printer firmware"** off
- When disabled, no firmware check requests are made to Bambu Lab servers
- Useful for users who prefer to manage firmware manually or have network restrictions
- The update badge will not appear on printer cards when disabled

---

## :material-download: Updating Firmware

### Step 1: Open Update Modal

Click the orange **Update** badge on the printer card to open the firmware update modal.

### Step 2: Review Version Info

The modal shows:

| Field | Description |
|-------|-------------|
| **Current** | Installed firmware version |
| **Latest** | Available firmware version |
| **Release Notes** | Click to expand and view changes |

### Step 3: Check Prerequisites

Before uploading, the system checks:

- :material-sd: SD card is inserted
- :material-database: Sufficient free space (~400MB recommended)
- :material-wifi: Printer is connected

### Step 4: Upload Firmware

1. Click **Upload Firmware**
2. Progress bar shows real-time upload status
3. Wait for upload to complete (typically 2-5 minutes for ~300MB)

### Step 5: Trigger Update on Printer

After upload completes, go to the printer's touchscreen:

1. Navigate to **Settings**
2. Select **Firmware**
3. Choose **Update from SD card**
4. Wait for update to complete (10-20 minutes)

!!! warning "Do Not Power Off"
    Never power off the printer during a firmware update. Wait for the process to complete fully.

---

## :material-cached: Firmware Cache

Downloaded firmware files are cached locally:

- **Location**: `firmware/` directory
- **Naming**: `{MODEL}_{VERSION}.bin`
- **Reuse**: Cached files are reused for subsequent uploads

This speeds up updates when:

- Updating multiple printers of the same model
- Re-uploading after a failed update
- Uploading to a new printer

---

## :material-printer-3d: Supported Printers

All Bambu Lab printer models are supported:

| Series | Models |
|--------|--------|
| **X1** | X1C, X1, X1E |
| **P1** | P1S, P1P, P2S |
| **A1** | A1, A1 Mini |
| **H2** | H2D, H2S |

---

## :material-help-circle: Troubleshooting

### No Update Badge

- Printer may already be on latest firmware
- Check printer is connected (online status)
- Firmware check may still be loading

### Upload Fails

- Ensure SD card is inserted and has space
- Check printer is not currently printing
- Verify network connectivity

### Upload Stuck

- Large files (300MB+) take several minutes
- Progress shows real bytes transferred
- 10-minute timeout prevents indefinite hangs

### Printer Doesn't See Update

- Firmware file must be in SD card root
- File must have correct `.bin` extension
- Try removing and reinserting SD card

---

## :material-frequently-asked-questions: FAQ

??? question "Can I update cloud-connected printers this way?"
    Yes, but it's unnecessary. Cloud-connected printers can receive OTA updates directly. This feature is primarily for LAN-only mode users.

??? question "Is the firmware from official sources?"
    Yes. Firmware is downloaded directly from Bambu Lab's official CDN servers.

??? question "Will this void my warranty?"
    No. Using official Bambu Lab firmware through SD card update is a supported method.

??? question "Can I downgrade firmware?"
    The system only shows newer versions as updates. Downgrading requires manually obtaining older firmware files.

??? question "What if the update fails?"
    The printer typically reverts to the previous firmware if an update fails. You can retry the upload process.
