---
title: Home
description: Bambuddy - Self-hosted print archive and management system for Bambu Lab 3D printers
hide:
  - navigation
  - toc
---

<style>
.md-typeset h1 { display: none; }
</style>

<div class="hero" markdown>

<div markdown>

# Your Prints.<br>Your Data.<br>Your Control.

**Bambuddy** is a self-hosted print archive and management system for Bambu Lab 3D printers. Monitor your print farm in real-time, archive every print automatically, and take control of your 3D printing workflow.

<div class="stats-row" markdown>
  <span class="stat-badge" markdown>:material-printer-3d: Multi-Printer</span>
  <span class="stat-badge" markdown>:material-cloud-off-outline: Works Offline</span>
  <span class="stat-badge" markdown>:material-open-source-initiative: Open Source</span>
</div>

[Get Started :material-arrow-right:](getting-started/index.md){ .btn .btn-primary }
[View on GitHub :material-github:](https://github.com/maziggy/bambuddy){ .btn .btn-secondary }

</div>

<div markdown>
![Bambuddy Dashboard](assets/hero-screenshot.png){ .screenshot }
</div>

</div>

---

## :rocket: Quick Start

<div class="quick-start" markdown>

[:material-download: **Installation**<br><small>Get up and running in minutes</small>](getting-started/installation.md)

[:material-docker: **Docker**<br><small>One-command deployment</small>](getting-started/docker.md)

[:material-printer-3d: **Add Printer**<br><small>Connect your first printer</small>](getting-started/first-printer.md)

[:material-cellphone: **Mobile**<br><small>Access from anywhere</small>](getting-started/mobile.md)

</div>

---

## :sparkles: Features

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-archive: Print Archive
Automatic 3MF archiving with metadata extraction, 3D model preview, duplicate detection, and full-text search. Never lose a print file again.
</div>

<div class="feature-card" markdown>
### :material-monitor-dashboard: Real-time Monitoring
Live printer status via WebSocket, MJPEG camera streaming, HMS error tracking, and AMS humidity/temperature monitoring with historical charts.
</div>

<div class="feature-card" markdown>
### :material-chart-line: Statistics & Analytics
Customizable drag-and-drop dashboard with success rates, filament usage trends, cost tracking, time accuracy analysis, and failure correlation.
</div>

<div class="feature-card" markdown>
### :material-clock-outline: Scheduling & Automation
Print queue with drag-and-drop ordering, scheduled prints, smart plug integration (Tasmota, Home Assistant), auto power-on/off, and energy consumption tracking.
</div>

<div class="feature-card" markdown>
### :material-bell-ring: Push Notifications
Multi-provider alerts via WhatsApp, Telegram, Discord, Email, Pushover, ntfy, and custom webhooks. Quiet hours and daily digest support.
</div>

<div class="feature-card" markdown>
### :material-puzzle: Integrations
Spoolman filament sync, Bambu Cloud profiles, K-profiles (pressure advance), external links, and a REST API with API key authentication.
</div>

</div>

[Explore All Features :material-arrow-right:](features/index.md){ .md-button }

---

## :printer: Supported Printers

| Series | Models | Status |
|--------|--------|:------:|
| **H2 Series** | H2D, H2S | :material-check-circle:{ .text-green } Tested |
| **X1 Series** | X1, X1 Carbon | :material-check-circle:{ .text-green } Tested |
| **P1 Series** | P1P, P1S | :material-help-circle:{ .text-yellow } Needs Testing |
| **A1 Series** | A1, A1 Mini | :material-help-circle:{ .text-yellow } Needs Testing |

!!! tip "Testers Wanted!"
    Help make Bambuddy work great with all Bambu Lab printers! Please [report your experience](https://github.com/maziggy/bambuddy/issues).

---

## :wrench: Tech Stack

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### Backend
:material-language-python: Python
:material-api: FastAPI
:material-database: SQLAlchemy + SQLite
</div>

<div class="feature-card" markdown>
### Frontend
:material-react: React
:material-language-typescript: TypeScript
:material-tailwind: Tailwind CSS
</div>

<div class="feature-card" markdown>
### Communication
:material-transit-connection-variant: MQTT over TLS
:material-folder-network: FTPS
:material-web: WebSocket
</div>

</div>

---

## :people_holding_hands: Community

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-bug: Found a Bug?
[Open an Issue](https://github.com/maziggy/bambuddy/issues/new)
</div>

<div class="feature-card" markdown>
### :material-lightbulb: Feature Request?
[Start a Discussion](https://github.com/maziggy/bambuddy/discussions)
</div>

<div class="feature-card" markdown>
### :material-code-tags: Want to Contribute?
[Read Contributing Guide](https://github.com/maziggy/bambuddy/blob/main/CONTRIBUTING.md)
</div>

</div>

---

<div style="text-align: center; margin-top: 3rem;" markdown>
<span style="opacity: 0.6;">Made with :heart: for the 3D printing community</span>
<br><br>
![Wiki Visitors](https://api.visitorbadge.io/api/visitors?path=http%3A%2F%2Fwiki.bambuddy.cool&labelColor=%23555555&countColor=%2379C83D&label=visitors&style=flat-square)
</div>
