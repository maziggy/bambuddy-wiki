# Bambuddy Wiki

Welcome to the Bambuddy documentation! Bambuddy is a self-hosted print archive and management system for Bambu Lab 3D printers.

## Quick Links

| Getting Started | Features | Reference |
|-----------------|----------|-----------|
| [Installation](Installation.md) | [Print Archiving](Features-Print-Archiving.md) | [API Documentation](API-Reference.md) |
| [Getting Started](Getting-Started.md) | [Real-time Monitoring](Features-Monitoring.md) | [Troubleshooting](Troubleshooting.md) |
| [Adding Your First Printer](Getting-Started.md#adding-your-first-printer) | [Print Queue & Scheduling](Features-Print-Queue.md) | [Environment Variables](Installation.md#environment-variables) |

## Feature Documentation

### Core Features
- **[Print Archiving](Features-Print-Archiving.md)** - Automatic 3MF archiving with metadata extraction
- **[Real-time Monitoring](Features-Monitoring.md)** - Live printer status, temperatures, and progress
- **[Statistics Dashboard](Features-Statistics.md)** - Print analytics, success rates, and cost tracking

### Automation
- **[Print Queue & Scheduling](Features-Print-Queue.md)** - Schedule prints with smart plug automation
- **[Smart Plug Integration](Features-Smart-Plugs.md)** - Tasmota-based power control and monitoring
- **[Push Notifications](Features-Notifications.md)** - Multi-provider alerts (WhatsApp, Telegram, Discord, etc.)

### Integrations
- **[Spoolman Integration](Features-Spoolman.md)** - Filament inventory sync
- **[Cloud Profiles](Features-Cloud-Profiles.md)** - Bambu Cloud slicer preset management
- **[K-Profiles](Features-K-Profiles.md)** - Pressure advance settings management

### Maintenance & Management
- **[Maintenance Tracker](Features-Maintenance.md)** - Schedule and track printer maintenance
- **[File Manager](Features-File-Manager.md)** - Browse and manage printer SD card files
- **[External Links](Features-External-Links.md)** - Add custom sidebar links to external tools

## Supported Printers

| Series | Models | Status |
|--------|--------|--------|
| H2 Series | H2C, H2D, H2S | Tested (H2D) |
| X1 Series | X1, X1 Carbon | Tested (X1C) |
| P1 Series | P1P, P1S | Needs Testing |
| A1 Series | A1, A1 Mini | Needs Testing |

> **Testers Needed!** If you have a printer model that needs testing, please help by reporting your experience in [GitHub Issues](https://github.com/maziggy/bambuddy/issues).

## Requirements

- **Python 3.10+** (3.11 or 3.12 recommended)
- **Node.js 18+** (only for building frontend from source)
- Bambu Lab printer with **LAN Mode** enabled
- Printer and server on the same local network

## Need Help?

- Check the [Troubleshooting](Troubleshooting.md) guide
- Search [existing issues](https://github.com/maziggy/bambuddy/issues)
- Open a [new issue](https://github.com/maziggy/bambuddy/issues/new) if you're stuck
