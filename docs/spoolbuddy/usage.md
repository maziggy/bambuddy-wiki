# :material-book-open: SpoolBuddy Usage Guide

This page covers day-to-day use of the SpoolBuddy interface — reading tags, managing AMS slots, writing tags, calibrating the scale, and adjusting settings.

The SpoolBuddy UI runs at `/spoolbuddy` on your Bambuddy backend:

```
http://<your-bambuddy-ip>:<port>/spoolbuddy
```

---

!!! warning "Work in progress"
    This usage guide is incomplete and we are looking for contributors to help document the SpoolBuddy UI in detail. If you use SpoolBuddy and would like to help, please open a PR or reach out on [Discord](https://discord.gg/aFS3ZfScHM).

---

## :material-navigation: Navigation

The bottom navigation bar has four tabs available from anywhere in the UI:

| Tab | Icon | Purpose |
|-----|------|---------|
| **Dashboard** | Home | Live spool reading and device status |
| **AMS** | Grid | AMS slot overview and slot management |
| **Write** | WiFi | Write NFC tags for spools |
| **Settings** | Gear | Device config, scale, display, updates |

---

## :material-home-analytics: Dashboard

The Dashboard is the main screen. It shows:

- **Stats bar** (top) — total spools, materials, and brands in your Bambuddy inventory
- **Device panel** (left) — live status of the device, scale, and NFC reader
- **Printer status** — connected printers (e.g. A1)
- **Current Spool** (right) — shows spool info when a tag is detected

<!-- TODO: Add screenshot of dashboard with spool detected, document spool info panel fields -->

---

<!-- TODO: Document remaining sections:
  - AMS slot management (assigning, configuring, unassigning slots)
  - Write tab (Existing Spool, New Spool, Replace Tag workflows)
  - Settings: Device, Display, Scale (tare + calibration), Updates tabs
-->

## :material-link: Related Pages

- [Wiring Diagrams](wiring-diagrams.md) — Electrical connections for all hardware
- [Installation](installation.md) — Software setup and first boot
- [Configuration](configuration.md) — Environment variables and `.env` settings
- [Assembly](assembly.md) — Physical case assembly guide
