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

<div class="feature-card" markdown>
### [:material-nfc-search-variant: ESPoolBuddy](https://github.com/CSchlipp/espoolbuddy)
ESPHome firmware that turns cheap ESP32-S3 boards into a [SpoolBuddy](../spoolbuddy/index.md)-style NFC spool station &mdash; no Raspberry Pi needed. The **Console** (WT32-SC01 Plus, 3.5" touchscreen + PN532) registers with Bambuddy over the HTTP API, heartbeats, polls AMS and printer state, and reports NFC scans; scan a tag and load the spool into the AMS within 60 seconds and it is assigned automatically. Tag *writing* is triggered from the Bambuddy dashboard. An optional **Scale** (any ESP32-S3 + HX711 load cell + PN532) never talks to Bambuddy itself &mdash; it finds the Console over mDNS and pushes weights through it, so everything arrives under one device ID. Deep-sleep and battery operation supported.

**Author:** [CSchlipp](https://github.com/CSchlipp) &middot; [Repository](https://github.com/CSchlipp/espoolbuddy)
</div>

<div class="feature-card" markdown>
### [:material-barcode-scan: NFC Handheld Case](https://makerworld.com/en/models/3043887)
Printable barcode-scanner-style enclosure that turns a SpoolBuddy console, an ESPoolBuddy console, or a SpoolEase console into a **handheld** NFC scanner &mdash; one-handed operation, everything in thumb reach, so you can walk the shelf and scan spools instead of carrying them to a fixed station. Two build options: a cable inlet for a permanently wired USB-C setup, or a battery compartment plus a pogo-pin connector for cordless use with magnetic charging (Adafruit PowerBoost 1000C + 18650). Assembly guide covers the heat-inserts, the PN532 on the back cover, and the optional speaker.

**Designer:** Chris ([CSchlipp](https://github.com/CSchlipp)) &middot; [MakerWorld model](https://makerworld.com/en/models/3043887)
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

<div class="feature-card" markdown>
### [:material-watch: Bambuddy Mobile](https://codeberg.org/DoYouHost/bambuddy-mobile)
Flutter companion app for **Android phones and Wear OS watches**. Live status, progress, temperatures and ETA; pause / resume / stop and queue management; filament inventory with QR scanning; maintenance reminders and hardware alerts; home-screen widgets. The watch app runs standalone or relays through the phone when it cannot reach the server itself. Talks only to your own Bambuddy instance over its REST and WebSocket APIs &mdash; never to Bambu's cloud &mdash; and a demo mode lets you look around without a server. AGPL-3.0, same as Bambuddy.

!!! warning "Early access"
    Published on Google Play as *testing / early access*; phone and watch APKs are also on Codeberg. Expect rough edges.

**Author:** [MorganMLGman](https://codeberg.org/MorganMLGman) &middot; [Repository](https://codeberg.org/DoYouHost/bambuddy-mobile)
</div>

</div>

---

## :material-monitor: Desktop

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-apple: BambuddyTray](https://github.com/bcsutar/BambuddyTray)
Native macOS **menu-bar** companion. Keeps a status badge (&check; healthy, ! attention, &times; error) in the menu bar with live progress and remaining time, and optionally raises macOS notifications when a print changes state or the server goes unreachable &mdash; so you can watch a print without a browser tab open and without Bambu Handy or any cloud account. Points at your Bambuddy instance with an API key, ships a settings UI and a LaunchAgent to start at login, and is distributed as a signed and notarised `.dmg`.

**Author:** [bcsutar](https://github.com/bcsutar) &middot; [Repository](https://github.com/bcsutar/BambuddyTray)
</div>

</div>

---

## :material-google-chrome: Browser Extensions

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-google-chrome: MakerWorld Import Extension](https://github.com/wolfrage76/Bambuddy-Extension)
Chrome (MV3) extension that sends any MakerWorld print profile to your Bambuddy instance in one click, matching the built-in *Import from MakerWorld* feature. Resolves models via Bambuddy's own resolve endpoint (the URL fragment pre-selects the profile when the page contains `#profileId-XXXXX`), flags profiles already in your library with a ✓ badge, and pulls thumbnails through Bambuddy's proxy so your IP never touches MakerWorld's CDN directly. Configure with your Bambuddy base URL + an API key that has **Manage Library** + **Allow cloud access**; a *Test Connection* helper runs `/health` + `/system/info` + `/makerworld/status` to catch misconfiguration before you save.

**Author:** [wolfrage76](https://github.com/wolfrage76) &middot; [Repository](https://github.com/wolfrage76/Bambuddy-Extension)
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

<div class="feature-card" markdown>
### [:material-nfc-variant: BambuMan](https://github.com/bambuman/BambuMan.App)
Android (built-in NFC) and Windows (PCSC reader like ACR122U) app that reads Bambu Lab spool NFC tags and imports them straight into your Bambuddy inventory. Matches each tag against SpoolmanDB so vendor, material, color, weight, and the corresponding Bambuddy slicer preset are filled in automatically; subsequent scans update the existing spool by `tray_uuid` instead of duplicating. Backend is selectable between Bambuddy and Spoolman in settings, so the spool the AMS reports over MQTT lines up with the one BambuMan imported. Distributed on Google Play and F-Droid; the project also runs its own public NFC tag library at [bambuman.ee](https://bambuman.ee).

**Author:** [bambuman](https://github.com/bambuman) &middot; [Repository](https://github.com/bambuman/BambuMan.App)
</div>

<div class="feature-card" markdown>
### [:material-nfc-tap: SpoolTap](https://github.com/dmuth23/spooltap)
NFC filament tracking orchestrated through Home Assistant. Stick an NTAG215 tag on each AMS slot and each spool, scan with the HA Companion app on Android, and SpoolTap pushes the matching custom filament profile to the printer through Bambuddy &mdash; resolves the spool's profile from Spoolman, applies it to the scanned slot via Bambuddy's API (`setting_id` codes), and pairs with `ha-bambulab` for usage telemetry back into Spoolman. The glue lives entirely in HA automations &mdash; no bespoke server.

**Author:** [dmuth23](https://github.com/dmuth23) &middot; [Repository](https://github.com/dmuth23/spooltap)
</div>

<div class="feature-card" markdown>
### [:material-barcode-scan: Filament Box Scanner](https://github.com/hibikipr/filament_to_bambuddy)
Mobile web app (PWA) for adding third-party spools to Bambuddy's inventory by scanning the box barcode or photographing the label. Lookup goes: local per-barcode cache (anything you've confirmed before) → the [Open Filament Database](https://openfilamentdatabase.org) (a spool-barcode-keyed catalogue of brand / material / colour / weight / print temps) → blank form with a paste-the-title auto-fill helper; the label-OCR path uses on-device text recognition so it works over plain HTTP (unlike the live camera which needs HTTPS). Everything you confirm gets remembered per barcode so the same product auto-fills instantly next time. Talks to Bambuddy's inventory API with a key that has **Manage Inventory**. Ships a multi-arch Docker image on GHCR and installs as a PWA on Android and iOS.

**Author:** [Victor Manuel / hibikipr](https://github.com/hibikipr) &middot; [Repository](https://github.com/hibikipr/filament_to_bambuddy)
</div>

<div class="feature-card" markdown>
### [:material-console-line: bambuddy-cli](https://github.com/mailletf/bambuddy-cli)
Terminal companion for assigning AMS slots and editing print-archive metadata without opening the web UI. Closes the AMS Lite gap &mdash; AMS Lite cannot read RFID, so third-party spools show up as occupied-but-unknown; the CLI calls Bambuddy's assignment API directly to map any inventory spool to any unit/slot. Also patches archived prints (cost, notes, tags, status, failure reason, external URL) from one-shot commands or an interactive picker. Talks to Bambuddy over the REST API with host + printer ID from `.env` or flags; shells out to `curl` to dodge a macOS IPv6 socket quirk.

**Author:** [mailletf](https://github.com/mailletf) &middot; [Repository](https://github.com/mailletf/bambuddy-cli)
</div>

</div>

---

## :material-folder-multiple-outline: Model Libraries

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-folder-sync-outline: Manyfold Sync](https://github.com/hibikipr/bambuddy_to_manyfold)
Python tool that syncs Bambuddy's print archives **and** file-manager library into [Manyfold](https://manyfold.app), the self-hosted 3D model manager. Bambuddy's folder hierarchy recreates as nested Manyfold collections, MakerWorld metadata (description, tags, cover image) enriches each model, and multiple profiles of the same design can be grouped into one Manyfold model. Uploads use Manyfold's resumable (Tus) API, which needs an OAuth application with **client_credentials** + the `upload` scope (personal access tokens can't carry it — the script probes for it on startup and stops early with instructions if missing). A local state file skips already-synced items on re-runs and `--force` recovers items whose upload background job failed. Ships CLI, Tkinter desktop GUI, and web GUI variants; `--cleanup-empty` deletes zero-file Manyfold models left over from failed uploads (needs the `delete` scope too).

**Author:** [Victor Manuel / hibikipr](https://github.com/hibikipr) &middot; [Repository](https://github.com/hibikipr/bambuddy_to_manyfold)
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

<div class="feature-card" markdown>
### [:material-bell-ring-outline: Bambark](https://github.com/rogernolan/Bambark)
Small Go service that receives Bambuddy's [webhook notifications](../features/notifications.md) and forwards them to [Bark](https://bark.day.app), the open-source iOS push app &mdash; so print-finished and print-failed events land on an iPhone without an account anywhere. Accepts the webhook as POST JSON or GET query parameters, authenticates callers with a bearer token, exposes `/healthz`, and ships systemd units for both root and rootless deployment. Intended to sit on your LAN behind your own reverse proxy: it does not terminate TLS itself.

**Author:** [Roger Nolan](https://github.com/rogernolan) &middot; [Repository](https://github.com/rogernolan/Bambark)
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

<div class="feature-card" markdown>
### [:material-kubernetes: Helm Chart](https://github.com/npelikan/bambuddy-helm)
Helm chart for running Bambuddy on Kubernetes. Pins the `ghcr.io/maziggy/bambuddy` image via the chart's `appVersion`, defaults to SQLite on a persistent volume with optional external PostgreSQL via `database.externalDatabase`, and persists `/app/data` and `/app/logs` on separate PVCs. Supports `hostNetwork` for SSDP printer discovery and grants `NET_BIND_SERVICE` so the virtual-printer ports (322, 990, FTP passive range) bind as a non-root user. Single-instance by design &mdash; uses the `Recreate` strategy so the RWO data volume is released cleanly between rollouts.

**Author:** [npelikan](https://github.com/npelikan) &middot; [Repository](https://github.com/npelikan/bambuddy-helm)
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

<div class="feature-card" markdown>
### [:material-shield-key-outline: Print Concierge](https://github.com/dpdev69/print-concierge)
Python MCP / CLI safety layer that sits between agent clients (Claude, Codex, Hermes, OpenClaw) and a Bambuddy server. Exposes a curated tool set for searching the archive, discovering models via 3DSEARCH, importing from MakerWorld / Printables / Thingiverse, slicing with explicit presets, hash-verifying outputs, and assembling deterministic print plans &mdash; but deliberately does **not** expose raw queue / start / pause / cancel. The only sensitive action is a scoped `queue_print_request(request_id)` that submits a pre-approved, plan-hash-bound request, with audit logging and Bambuddy's manual-start as the default. Talks to Bambuddy over the REST API with a least-privilege API key (`BAMBUDDY_BASE_URL` + `BAMBUDDY_API_KEY`). Ships a sandbox backend mode for no-hardware demos.

**Author:** [dpdev69](https://github.com/dpdev69) &middot; [Repository](https://github.com/dpdev69/print-concierge)
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
