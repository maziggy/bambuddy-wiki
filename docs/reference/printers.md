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
| **X2D** | X2 | :material-check: | :material-check: | :material-check: |
| **H2D** | H2 | :material-check: | :material-check: | :material-check: |
| **H2D Pro** | H2 | :material-check: | :material-check: | :material-check: |
| **H2C** | H2 | :material-check: | :material-check: | :material-check: |
| **H2S** | H2 | :material-check: | :material-check: | :material-check: |
| **P1P** | P1 | :material-check: | Add-on | :material-check: |
| **P1S** | P1 | :material-check: | :material-check: | :material-check: |
| **P2S** | P2 | :material-check: | :material-check: | :material-check: |
| **A1** | A1 | :material-check: | :material-check: | AMS Lite |
| **A1 Mini** | A1 | :material-check: | :material-check: | :material-close: |
| **A2L** | A2 | :material-check: | :material-check: | AMS Lite |

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

### X2 Series

| Feature | X2D |
|---------|:---:|
| LAN Mode | :material-check: |
| Camera | :material-check: (RTSP, port 322) |
| AMS Support | AMS 2 Pro / AMS HT |
| Chamber Heating | :material-check: |
| Dual Nozzle | :material-check: |
| Steel Rod Gantry | :material-check: (hardened steel, shared with P2S) |
| Airduct Mode | :material-check: |
| Enclosure Door Sensor | :material-check: |

!!! note "X2D added in Bambuddy 0.2.3b4"
    X2D support was added in April 2026 after the printer's launch. It is treated
    as a dual-nozzle family alongside H2D for K-profile management and print-command
    formatting, and inherits the P2S wiki pages for steel-rod lubrication, belt
    tension, cold-pull maintenance, and PTFE tube replacement.

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
| Chamber Heating | :material-check: |
| Dual Nozzle | :material-close: |

### A1 Series

| Feature | A1 | A1 Mini |
|---------|:--:|:-------:|
| LAN Mode | :material-check: | :material-check: |
| Camera | :material-check: | :material-check: |
| AMS Support | AMS Lite | :material-close: |
| Chamber Heating | :material-close: | :material-close: |
| Dual Nozzle | :material-close: | :material-close: |

### A2 Series

| Feature | A2L |
|---------|:---:|
| LAN Mode | :material-check: |
| Camera | :material-check: (Chamber Image, port 6000) |
| AMS Support | AMS Lite |
| Chamber Heating | :material-close: |
| Dual Nozzle | :material-close: (single FDM extruder) |
| Linear Rails | :material-check: |
| Ethernet | :material-close: (Wi-Fi 2.4 GHz only) |
| Integrated Cutter / Plotter | :material-check: |

!!! note "A2L added in Bambuddy 0.2.5b1"
    A2L support shipped in June 2026. Internal model code `N9`, serial prefix `26A19`.
    The BambuStudio profile flag `use_double_extruder_default_texture: true` covers the
    two TOOL HEADS (FDM extruder + cutter/plotter), not dual filament extrusion —
    A2L is treated as single-nozzle for AMS routing, K-profile management, and
    print-command formatting. The cutter / plotter capability is not yet modelled
    in Bambuddy; FDM control works as on any A-series printer.

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

| Series | Protocol | Port |
|--------|:--------:|:----:|
| X1 | RTSP | 322 |
| X2 | RTSP | 322 |
| H2 | RTSP | 322 |
| P2 | RTSP | 322 |
| P1 | Chamber Image | 6000 |
| A1 | Chamber Image | 6000 |
| A2 | Chamber Image | 6000 |

### Cam Wall view

The Printers page has a **Cards / Cam wall** toggle next to the card-size selector. Cam Wall replaces the printer cards with a responsive grid of live camera tiles for at-a-glance monitoring across the whole farm.

To keep the host CPU and LAN bandwidth bounded — especially on Raspberry Pi installs — Cam Wall only streams tiles that are currently visible on screen, up to a per-user **Max live streams** cap. Tiles that are visible but over the cap fall back to periodic **snapshot polling** at a configurable interval. Tiles that scroll off-screen pause entirely and release their backend transcoder slot. Disconnected printers also render paused (with the offline indicator) and do not consume a live-stream slot — only working tiles count against the cap.

#### Settings

The gear icon above the grid exposes three per-user knobs:

| Setting | Default | Range / Options | Stored |
|---|:---:|:---:|---|
| Max live streams | 4 | 1 – 16 | per-user `localStorage` |
| Snapshot interval (s) | 8 | 2 – 60 | per-user `localStorage` |
| Status overlay | Full | Off / Compact / Full | per-user `localStorage` |

**Status overlay** controls how much of the printer's state is drawn on top of each tile:

- **Off** — camera image only.
- **Compact** — colour-coded state chip in the top-left corner (`Printing`, `Paused`, `Finished`, `Error`). Idle printers stay clean (no chip) so a wall of cold printers reads as quiet. A warning icon appears in the chip when the printer has active HMS errors.
- **Full** — Compact chip plus a bottom info strip on currently-printing or paused tiles: filename, progress percent, current/total layers, and remaining time. The numbers match what the printer card shows for the same printer at the same moment.

#### Clicking a tile

Click any tile to open it in the floating camera viewer or in a dedicated window, matching your existing **Settings → Camera view** preference. Opening the larger viewer while the cam-wall tile is still on screen no longer interrupts the tile — both viewers share the same backend MJPEG fan-out, and closing the larger viewer leaves the tile streaming. The cam-wall toggle requires the `camera:view` permission.

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
