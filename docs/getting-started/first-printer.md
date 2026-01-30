---
title: First Printer Setup
description: Connect your first Bambu Lab printer to Bambuddy
---

# First Printer Setup

This guide walks you through adding your first printer to Bambuddy.

---

## :material-list-status: Prerequisites

Before adding a printer, ensure:

- [x] Bambuddy is running ([Installation](installation.md) or [Docker](docker.md))
- [x] Your printer is powered on and connected to your network
- [x] **SD card is inserted** in the printer (required for file transfers)
- [x] Developer Mode is enabled ([see guide](index.md#enabling-developer-mode))
- [x] You have the IP address, serial number, and access code

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for Bambuddy to work properly. The SD card is required for file transfers, print uploads, and archiving. Without an SD card, prints cannot be started from Bambuddy and files cannot be transferred to/from the printer.

---

## :material-printer-3d-nozzle: Adding Your Printer

### Step 1: Open the Add Printer Dialog

1. Open Bambuddy in your browser
2. Go to the **Printers** page
3. Click the **:material-plus: Add Printer** button

### Step 2: Enter Printer Details

Fill in the required information:

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | A friendly name for your printer | `Workshop X1C` |
| **IP Address** | Your printer's local IP address | `192.168.1.100` |
| **Access Code** | 8-character code from Developer Mode | `12345678` |
| **Serial Number** | Your printer's serial number | `01P00A000000001` |

!!! tip "Finding Serial Number"
    The serial number format varies by model:

    - **X1 Series**: Starts with `01S` or `01P`
    - **P1 Series**: Starts with `01P`
    - **A1 Series**: Starts with `01A`
    - **H2 Series**: Starts with `03W`

### Step 3: Save and Connect

1. Click **Save**
2. Bambuddy will attempt to connect to your printer
3. Wait for the connection indicator to turn green

---

## :material-check-circle: Verifying Connection

A successfully connected printer shows:

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-circle:{ style="color: #4caf50" } Green Indicator
Connection is active and healthy
</div>

<div class="feature-card" markdown>
### :material-thermometer: Live Temperatures
Nozzle, bed, and chamber readings update in real-time
</div>

<div class="feature-card" markdown>
### :material-check-decagram: HMS Status
Health Management System shows "OK" (green)
</div>

</div>

---

## :material-layers: Understanding the Printer Card

Once connected, your printer card displays:

```
┌─────────────────────────────────────────────────────────────┐
│  [●] Workshop X1C                              [📷] [⚙️]   │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  🌡️ Nozzle: 220°C    🛏️ Bed: 60°C    🏠 Chamber: 35°C     │
│                                                             │
│  ████████████████████░░░░░░░░░░░░░░░░░░░  45%              │
│  Layer 120 of 267  •  1h 23m remaining  •  45g used        │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  AMS                                                 │   │
│  │  [🔵] PLA Blue   [🔴] PLA Red   [⚪] Empty   [⚪]    │   │
│  │  💧 45%  🌡️ 28°C                                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
│  HMS: ✅ OK                                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Card Elements

| Element | Description |
|---------|-------------|
| **Connection indicator** | Green = connected, Red = disconnected |
| **Camera button** | Open live camera feed |
| **Settings button** | Edit printer settings |
| **Temperatures** | Live nozzle, bed, and chamber temps |
| **Progress bar** | Current print progress (if printing) |
| **Layer info** | Current layer / total layers |
| **Time remaining** | Estimated completion time |
| **Filament used** | Grams consumed this print |
| **AMS section** | Filament slots, humidity, and temperature |
| **HMS status** | Health status indicator |

---

## :material-camera: Camera Setup

Bambuddy supports live camera streaming:

1. Click the :material-camera: camera icon on the printer card
2. A new window opens with the live feed
3. Toggle between **Live** stream and **Snapshot** mode

!!! note "Camera Requirements"
    - Camera must be enabled in your printer settings
    - Requires `ffmpeg` installed on the Bambuddy server for MJPEG streaming
    - Works over Developer Mode connection

---

## :material-alert-circle: Connection Issues?

If your printer won't connect:

### Check These First

1. **Is Developer Mode enabled?** Re-check that both LAN Only Mode and Developer Mode are enabled
2. **Correct IP address?** Verify in printer network settings
3. **Access code fresh?** Codes change when Developer Mode is toggled
4. **Same network?** Bambuddy server and printer must be on the same LAN
5. **Firewall rules?** Ensure ports 8883 (MQTT) and 990 (FTPS) are open

### Status Indicators

| Indicator | Meaning |
|-----------|---------|
| :material-circle:{ style="color: #4caf50" } Green | Connected and communicating |
| :material-circle:{ style="color: #ff9800" } Yellow | Connecting or reconnecting |
| :material-circle:{ style="color: #f44336" } Red | Disconnected or error |

### Still Having Issues?

See the full [Troubleshooting Guide](../reference/troubleshooting.md#printer-connection-issues).

---

## :material-printer-3d: Adding More Printers

Repeat the process to add additional printers. Bambuddy supports unlimited printers - manage your entire print farm from one interface!

---

## :material-archive: What Happens Next?

Once your printer is connected, Bambuddy automatically:

1. **Monitors print status** in real-time
2. **Archives completed prints** with full metadata
3. **Captures camera snapshots** on print completion (if enabled)
4. **Tracks statistics** for your dashboard

Start a print from Bambu Studio, Handy, or the printer itself - Bambuddy will detect it automatically!

---

## :checkered_flag: Next Steps

<div class="quick-start" markdown>

[:material-archive: **Print Archiving**<br><small>How archiving works</small>](../features/archiving.md)

[:material-bell-ring: **Notifications**<br><small>Get alerts on your phone</small>](../features/notifications.md)

[:material-chart-line: **Statistics**<br><small>Track your prints</small>](../features/statistics.md)

[:material-clock-outline: **Print Queue**<br><small>Schedule prints</small>](../features/print-queue.md)

</div>
