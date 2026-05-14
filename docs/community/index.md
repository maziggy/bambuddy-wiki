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

## :material-home-automation: Home Automation

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-toggle-switch-outline: Hubitat Driver](https://github.com/jc21/hubitat-bambuddy)
Hubitat Elevation driver that talks to Bambuddy over REST (and optionally MQTT). Lets you wire Bambu Lab printer state and commands &mdash; clear plate, light on/off, pause/resume/stop &mdash; into your home-automation rules and physical buttons.

**Author:** [jc21](https://github.com/jc21) &middot; [Repository](https://github.com/jc21/hubitat-bambuddy)
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
Python script that bulk-imports filament spools into Bambuddy from a CSV file. Maps your existing spool IDs to Bambuddy catalog IDs and pushes them through the public REST API &mdash; useful for migrating from other inventory tools.

**Author:** [bsaunder](https://github.com/bsaunder) &middot; [Repository](https://github.com/bsaunder/bambuddy_spoolimport)
</div>

<div class="feature-card" markdown>
### [:material-cash-sync: Spoolman Cost Sync](https://github.com/ojimpo/bambuddy-spoolman-cost-sync)
Python sidecar that copies per-spool prices from Spoolman into Bambuddy's local `cost_per_kg` field, so per-print cost calculations reflect what each spool actually cost. Read-only against Spoolman, writes only `cost_per_kg` on Bambuddy &mdash; matches RFID-linked spools by `tray_uuid` / `tag_uid` and runs on a configurable interval (default 10 min). Stdlib-only; ships as a Docker sidecar.

**Author:** [ojimpo](https://github.com/ojimpo) &middot; [Repository](https://github.com/ojimpo/bambuddy-spoolman-cost-sync)
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

Questions? Drop into the [Discord](https://discord.gg/aFS3ZfScHM) or [forum](https://forum.bambuddy.cool).
