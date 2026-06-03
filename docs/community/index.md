---
title: Community Add-ons
description: Third-party hardware and software add-ons built by the Bambuddy community
---

# Community Add-ons

Third-party projects that extend Bambuddy &mdash; built by the community, for the community. These are independent projects maintained outside the main Bambuddy repository, talking to Bambuddy through its public API.

!!! info "Want to build one?"
    Bambuddy exposes a documented [REST API](../reference/api.md) and supports [API keys](../features/api-keys.md) for headless integrations. If you build something cool, open a PR on the [wiki repo](https://github.com/maziggy/bambuddy-wiki) to add it here.

!!! warning "Independent projects"
    Community add-ons are maintained by their respective authors, not by the Bambuddy team. Issues and feature requests for an add-on belong on its own repository.

---

## :material-gesture-tap-button: Hardware

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-radiobox-marked: Bambutton](bambutton.md)
ESP32-C3 wireless button that lets you mark the build plate clear at the printer with a single press, so the next queued job can be dispatched automatically. Includes an LED ring that signals when a plate needs clearing.

**Author:** [Edward Chamberlain](https://github.com/EdwardChamberlain) &middot; [Repository](https://github.com/EdwardChamberlain/bambutton)
</div>

<div class="feature-card" markdown>
### [:material-nfc: ESPHome Bambu Spool Reader](https://github.com/bemble/esphome-bambuddy-reader)
ESPHome firmware for a Waveshare ESP32-S3 with PN532 NFC reader and 2.8" touchscreen. Reads Bambu Lab spool tags and syncs them with Bambuddy &mdash; looks up known spools by tag UID, displays remaining weight, and can register new spools straight from the device.

**Author:** [bemble](https://github.com/bemble) &middot; [Repository](https://github.com/bemble/esphome-bambuddy-reader)
</div>

<div class="feature-card" markdown>
### [:material-chip: ESP Virtual Printer](https://github.com/mengxyz/ESP_VP)
ESP-IDF firmware for an ESP32-S3 with PSRAM that presents an archive-only Bambu virtual printer on the LAN. Slicers discover it like a real printer and upload `.3mf` files over implicit FTPS; the ESP streams the bytes straight into Bambuddy as a chunked HTTP upload &mdash; no SD card, no local storage. Works with the bundled `buddy_recv.py` sidecar today.

**Author:** [mengxyz](https://github.com/mengxyz) &middot; [Repository](https://github.com/mengxyz/ESP_VP)
</div>

</div>

---

## :material-cellphone: Mobile Apps

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-android: BuddyDash](https://github.com/ChronosWing/BuddyDash)
Unofficial Android companion app for Bambuddy &mdash; a mobile-first dashboard for quick status checks, multi-printer overview, archives browsing, and smart-outlet control. The standout is NFC quick actions: stick a tag on the printer and tap it to clear the plate, toggle power (with idle-safety checks), or run a finish workflow &mdash; no app navigation needed. Built with Jetpack Compose, with layouts optimised for phones, foldables, and tablets. Connects to your existing Bambuddy server through the REST API; it's a companion, not a replacement.

!!! warning "Beta"
    The author flags BuddyDash as beta &mdash; expect rough edges and breaking changes. Track the repo for stable releases.

**Author:** [ChronosWing](https://github.com/ChronosWing) &middot; [Repository](https://github.com/ChronosWing/BuddyDash)
</div>

</div>

---

## :material-home-automation: Home Automation

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-toggle-switch-outline: Hubitat Driver](https://github.com/jc21/hubitat-bambuddy)
Hubitat Elevation driver that talks to Bambuddy over REST (and optionally MQTT). Lets you wire Bambu Lab printer state and commands &mdash; clear plate, light on/off, pause/resume/stop &mdash; into your home-automation rules and physical buttons.

**Author:** [jc21](https://github.com/jc21) &middot; [Repository](https://github.com/jc21/hubitat-bambuddy)
</div>

<div class="feature-card" markdown>
### [:material-home-assistant: Home Assistant Sidecar Add-ons](https://github.com/tzimpel/homeassistant-bambuddy-sidecar-apps)
Home Assistant Add-on repository that packages Bambuddy's optional sidecar containers as one-click HA add-ons. Includes the OrcaSlicer API sidecar for server-side slicing &mdash; add the repository URL in Home Assistant's add-on store and install without writing any Docker Compose.

**Author:** [Jan-Tobias Zimpel](https://github.com/tzimpel) &middot; [Repository](https://github.com/tzimpel/homeassistant-bambuddy-sidecar-apps)
</div>

<div class="feature-card" markdown>
### [:material-home-assistant: Home Assistant App (Bambuddy)](https://github.com/Spegeli/homeassistant-app-bambuddy)
Home Assistant Add-on repository that packages Bambuddy itself as a first-class HA App for Supervised / HA OS installations. Ships three flavours &mdash; stable, beta, and daily &mdash; with hourly update checks, so new Bambuddy releases appear directly in the HA App Store. Data is persisted in `addon_configs` across upgrades. Note: HA Ingress is not supported (SPA + service worker constraints).

**Author:** [Spegeli](https://github.com/Spegeli) &middot; [Repository](https://github.com/Spegeli/homeassistant-app-bambuddy)
</div>

<div class="feature-card" markdown>
### [:material-puzzle-outline: HACS Integration](https://github.com/Spegeli/hacs_bambuddy)
HACS custom integration that exposes a Bambuddy instance to Home Assistant. Adds each printer as a device with sensors (status, progress, layer, temperatures, fan speeds, HMS, diagnostics), a camera entity, the print-job cover image, pause / resume / stop / clear-plate buttons, a chamber-light switch, and a print-speed select. Also surfaces instance-level stats (total prints, filament used, disk usage).

!!! warning "Under active development"
    The author explicitly flags this integration as not yet production-ready &mdash; expect breaking changes and incomplete features. Track the repo for stable releases.

**Author:** [Spegeli](https://github.com/Spegeli) &middot; [Repository](https://github.com/Spegeli/hacs_bambuddy)
</div>

</div>

---

## :material-package-variant-closed: Filament &amp; Inventory

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-puzzle-outline: FilaMan Bambuddy Plugin](https://github.com/Fire-Devils/filaman-bambuddy-plugin)
FilaMan driver plugin that connects [FilaMan](https://www.filaman.app) to Bambuddy. Receives real-time AMS slot data via WebSocket, supports manual and automatic spool assignment, and optionally syncs FilaMan's spool inventory into Bambuddy.

**Author:** [FilaMan / Fire-Devils](https://github.com/Fire-Devils) &middot; [Repository](https://github.com/Fire-Devils/filaman-bambuddy-plugin)
</div>

<div class="feature-card" markdown>
### [:material-database-import-outline: CSV Spool Import](https://github.com/bsaunder/bambuddy_spoolimport)
Python toolkit for managing Bambuddy's filament inventory from the command line. Bulk-imports spools from a CSV file (mapping your existing spool IDs to Bambuddy catalog IDs), lists existing spools, and generates printable QR-code PDF labels for physical spool identification. Talks to Bambuddy through its public REST API with an API key &mdash; useful for migrating from other inventory tools.

**Author:** [bsaunder](https://github.com/bsaunder) &middot; [Repository](https://github.com/bsaunder/bambuddy_spoolimport)
</div>

<div class="feature-card" markdown>
### [:material-cash-sync: Spoolman Cost Sync](https://github.com/ojimpo/bambuddy-spoolman-cost-sync)
Python sidecar that copies per-spool prices from Spoolman into Bambuddy's local `cost_per_kg` field, so per-print cost calculations reflect what each spool actually cost. Read-only against Spoolman, writes only `cost_per_kg` on Bambuddy &mdash; matches RFID-linked spools by `tray_uuid` / `tag_uid` and runs on a configurable interval (default 10 min). Stdlib-only; ships as a Docker sidecar.

**Author:** [ojimpo](https://github.com/ojimpo) &middot; [Repository](https://github.com/ojimpo/bambuddy-spoolman-cost-sync)
</div>

<div class="feature-card" markdown>
### [:material-cellphone-nfc: Spool NFC PWA](https://github.com/Poltavtcev/spool-nfc-pwa)
Mobile Progressive Web App for reading and writing OpenTag3D NFC tags (NTAG213/215/216) on unofficial filament spools straight from your phone. Scan a tag to find or link a spool, browse the inventory with material filters and remaining-weight indicators, assign spools to AMS and external slots, and update weights &mdash; all over Bambuddy's REST API with an API key. Installs as a PWA on Android and desktop Chrome, and credentials stay in the browser only. NFC writing needs Chrome on Android with Web NFC; iOS can install the PWA but cannot use NFC.

**Author:** [Poltavtcev](https://github.com/Poltavtcev) &middot; [Repository](https://github.com/Poltavtcev/spool-nfc-pwa)
</div>

</div>

---

## :material-message-text-outline: Bots &amp; Notifications

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-discord: Discord Print Queue Bot](https://github.com/CrazyClone55/bambuddy-discord-bot)
Discord bot that turns a Forum channel into a print queue. Members submit `.gcode.3mf` files via `!print` in a thread; superusers queue immediately, others go through an approval reaction from a designated approver. Pings the submitter when their print starts and finishes.

**Author:** [CrazyClone55](https://github.com/CrazyClone55) &middot; [Repository](https://github.com/CrazyClone55/bambuddy-discord-bot)
</div>

<div class="feature-card" markdown>
### [:material-message-processing-outline: Signal Print Queue Bot](https://github.com/phieb/bambu-bot)
Signal bot that turns MakerWorld links into Bambuddy print jobs. DM the bot a link and it creates a persistent Signal group, then walks you through profile / plate(s) / per-filament AMS slot selection in numbered replies, re-slices for your target printer, and queues each plate through Bambuddy's REST API. Group commands give live progress with a camera snapshot (`!progress`), the current queue (`!queue`), and cancel-the-last-pending-job (`!cancel`); the bot pings the group when each queued print finishes or fails. Receives messages from an upstream dispatcher (the author's [signal-router](https://github.com/phieb/signal-router) or any forwarder that hits its `/claims` + `/receive` endpoints).

**Author:** [phieb](https://github.com/phieb) &middot; [Repository](https://github.com/phieb/bambu-bot)
</div>

<div class="feature-card" markdown>
### [:material-printer-pos: PrinterPrinter](https://github.com/FirstBuild/PrinterPrinter)
Local Python daemon that watches Bambuddy for print starts and auto-prints a job label on a Brother QL-820NWB. Polls Bambuddy's REST API on a configurable interval (default 5 s), detects new print events, persists them in SQLite, and renders a DK1202 (62&times;100 mm) label per job with filament mass and optional cost (price-per-gram &times; rounded grams). Admin endpoints expose `/health`, `/admin/printers`, `/admin/events`, and a reprint hook (`POST /admin/print-event/{id}`). Ships with an interactive Raspberry Pi installer for always-on deployments.

**Author:** [FirstBuild](https://github.com/FirstBuild) &middot; [Repository](https://github.com/FirstBuild/PrinterPrinter)
</div>

</div>

---

## :material-server-network: Deployment &amp; Automation

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-script-text-outline: Ansible Collection](https://github.com/nils-ost/ansible-collection-bambuddy)
Ansible collection for installing and configuring Bambuddy at scale. Includes modules for initial setup, login/token fetch, settings, printer management, and the virtual-printer feature &mdash; all callable via `delegate_to: localhost`.

**Author:** [nils-ost](https://github.com/nils-ost) &middot; [Repository](https://github.com/nils-ost/ansible-collection-bambuddy)
</div>

</div>

---

## :material-robot-outline: AI &amp; Integrations

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-cog-outline: Bambuddy MCP Server](bambuddy-mcp.md)
Model Context Protocol server that exposes Bambuddy's full REST API as tools for AI assistants like Claude Desktop and Claude Code. Talk to your printers in plain language &mdash; check status, queue prints, control lights, view camera snapshots.

**Author:** [MrMebelMan](https://github.com/MrMebelMan) &middot; [Repository](https://github.com/MrMebelMan/bambuddy-mcp)
</div>

</div>

---

## Contributing an add-on

If you've built something that integrates with Bambuddy &mdash; hardware, browser extensions, mobile shortcuts, scripts, dashboards, anything &mdash; we'd love to list it here.

To submit:

1. Make sure the project has a clear README and a license.
2. Open a PR against the [wiki repository](https://github.com/maziggy/bambuddy-wiki) adding a page under `docs/community/` and a card on this hub page.
3. Keep it interoperability-focused &mdash; add-ons should use Bambuddy's public API, not internal endpoints.

Questions? Drop into the [Discord](https://discord.gg/aFS3ZfScHM).
