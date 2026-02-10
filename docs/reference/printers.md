---
title: Supported Printers
description: Compatibility information for Bambu Lab printer models
---

# Supported Printers

Bambuddy supports Bambu Lab 3D printers with Developer Mode capability.

---

## :material-check-circle: Supported Printers

All Bambu Lab printer models are supported:

| Model | Series | LAN Mode | Camera | AMS |
|-------|--------|:--------:|:------:|:---:|
| **X1** | X1 | :material-check: | :material-check: | :material-check: |
| **X1 Carbon** | X1 | :material-check: | :material-check: | :material-check: |
| **X1E** | X1 | :material-check: | :material-check: | :material-check: |
| **H2D** | H2 | :material-check: | :material-check: | :material-check: |
| **H2D Pro** | H2 | :material-check: | :material-check: | :material-check: |
| **H2C** | H2 | :material-check: | :material-check: | :material-check: |
| **H2S** | H2 | :material-check: | :material-check: | :material-check: |
| **P1P** | P1 | :material-check: | Add-on | :material-check: |
| **P1S** | P1 | :material-check: | :material-check: | :material-check: |
| **P2S** | P2 | :material-check: | :material-check: | :material-check: |
| **A1** | A1 | :material-check: | :material-check: | AMS Lite |
| **A1 Mini** | A1 | :material-check: | :material-check: | :material-close: |

---

## :material-printer-3d: Feature Comparison by Series

### X1 Series

| Feature | X1 | X1 Carbon | X1E |
|---------|:--:|:---------:|:---:|
| LAN Mode | :material-check: | :material-check: | :material-check: |
| Camera | :material-check: | :material-check: | :material-check: |
| AMS Support | 4 units | 4 units | 4 units |
| Chamber Heating | :material-check: | :material-check: | :material-check: |
| Dual Nozzle | :material-close: | :material-close: | :material-close: |

### H2 Series

| Feature | H2D | H2D Pro | H2C | H2S |
|---------|:---:|:-------:|:---:|:---:|
| LAN Mode | :material-check: | :material-check: | :material-check: | :material-check: |
| Camera | :material-check: | :material-check: | :material-check: | :material-check: |
| AMS Support | 4 units | 4 units | 4 units | 4 units |
| Chamber Heating | :material-check: | :material-check: | :material-check: | :material-check: |
| Dual Nozzle | :material-check: | :material-check: | :material-close: | :material-close: |
| Nozzle Rack | :material-close: | :material-close: | 6-slot | :material-close: |

### P1 Series

| Feature | P1P | P1S |
|---------|:---:|:---:|
| LAN Mode | :material-check: | :material-check: |
| Camera | Add-on | :material-check: |
| AMS Support | 4 units | 4 units |
| Chamber Heating | :material-close: | :material-close: |
| Dual Nozzle | :material-close: | :material-close: |

### P2 Series

| Feature | P2S |
|---------|:---:|
| LAN Mode | :material-check: |
| Camera | :material-check: |
| AMS Support | 4 units |
| Chamber Heating | :material-close: |
| Dual Nozzle | :material-close: |

### A1 Series

| Feature | A1 | A1 Mini |
|---------|:--:|:-------:|
| LAN Mode | :material-check: | :material-check: |
| Camera | :material-check: | :material-check: |
| AMS Support | AMS Lite | :material-close: |
| Chamber Heating | :material-close: | :material-close: |
| Dual Nozzle | :material-close: | :material-close: |

---

## :material-lan: Developer Mode Requirements

### Why Developer Mode?

Since the January 2025 firmware update, **Developer Mode is required** for full printer control:

- **LAN Only Mode (without Developer Mode)**: Read-only access - monitoring only
- **Developer Mode**: Full access - control, file transfers, print scheduling

### Enabling Developer Mode

All supported printers require Developer Mode for full functionality:

1. Go to printer **Settings > Network > LAN Only Mode**
2. Enable **LAN Only Mode**
3. Enable **Developer Mode** (appears after LAN Only Mode is enabled)
4. Note the **Access Code** (8 characters)
5. Access code changes when these modes are toggled

---

## :material-video: Camera Support

### RTSP Streaming

Bambuddy uses RTSP to access printer cameras:

- Requires ffmpeg installed
- Converts to MJPEG for browser viewing
- Port varies by printer model

### Camera Ports

| Series | RTSP Port |
|--------|:---------:|
| X1 | 6000 |
| H2 | 6000 |
| P1 | 6000 |
| A1 | 6000 |

---

## :material-tray-full: AMS Support

### Standard AMS

- Up to 4 units per printer
- 4 slots per unit (16 total)
- Humidity monitoring
- RFID filament detection

### AMS-HT (High Temperature)

- Active drying capability
- Temperature monitoring
- For hygroscopic filaments

### AMS Lite (A1)

- Simplified AMS for A1
- 4 slots
- No active drying

---

## :material-help: Feedback & Issues

If you encounter any issues with your printer model:

1. Check the [Troubleshooting Guide](troubleshooting.md)
2. Search [existing issues](https://github.com/maziggy/bambuddy/issues) for your problem
3. If not found, [open a new issue](https://github.com/maziggy/bambuddy/issues/new) with:
    - Printer model and firmware version
    - What works and what doesn't
    - Any error messages or logs

---

## :material-close-circle: Known Issues

### By Model

| Model | Issue | Workaround |
|-------|-------|------------|
| - | - | - |

*No known model-specific issues currently reported*

### By Feature

| Feature | Issue | Affected Models |
|---------|-------|-----------------|
| - | - | - |

*No known feature-specific issues currently reported*

---

## :material-update: Firmware Compatibility

Bambuddy is tested with recent firmware versions. If you encounter issues:

1. Check if firmware is up to date
2. Try updating firmware
3. Report the firmware version with any issues

### Minimum Firmware

Specific minimum versions aren't defined yet. Modern firmware (2024+) should work.
