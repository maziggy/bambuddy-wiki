---
title: Bambuddy Appliance
description: The Bambuddy Appliance - a Raspberry Pi 5 image with Bambuddy preinstalled, a guided setup wizard, and its own admin panel
---

# Bambuddy Appliance

The Bambuddy Appliance is a Raspberry Pi 5 image with Bambuddy already installed and configured. You write it to a memory card, walk through a setup wizard in your browser, and start adding printers. There is no Docker to install, no compose file to edit, and no cloud account to create.

You supply the hardware. The appliance is sold as an **image download plus an annual subscription**, from October 2026 &mdash; nothing is shipped, and the Raspberry Pi is yours rather than rented.

It runs the same Bambuddy you would install yourself &mdash; same features, same AGPL licence, same local-only operation. What the appliance adds is everything *around* Bambuddy: first-boot bring-up, a captive-portal WiFi setup, an admin panel for the box itself, health-checked container upgrades with automatic rollback, and an A/B operating system that can fall back to the copy that worked.

---

## Who it's for

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-package-variant-closed: You want it to just work
Flash one card, answer four questions, and skip the self-hosting entirely. Updates are tested together and undo themselves, and support comes from the person who builds it.
</div>

<div class="feature-card" markdown>
### :material-hammer-wrench: You already self-host
You probably don't need this. A Docker install gives you the same Bambuddy. The appliance is about the box, not the software.
</div>

</div>

!!! info "Bambuddy stays Bambuddy"
    The application on the appliance is the ordinary AGPL-3.0 Bambuddy container from `ghcr.io/maziggy/bambuddy`. You can inspect it, replace it, or run it somewhere else. The appliance wrapper &mdash; the wizard, the admin panel, the CLI &mdash; is separately licensed, and it never restricts what the AGPL grants you.

---

## What you need to buy

Four parts, all of them stocked by every Raspberry Pi dealer.

| | | |
|---|---|---|
| **Computer** | Raspberry Pi **5**, 4 GB | 8 GB for a large fleet |
| **Power** | The official 27 W USB-C supply | Undervoltage shows up as dropped printers, not as an error |
| **Cooling** | The official Active Cooler | The box runs continuously; a throttled Pi looks like slow software |
| **Storage** | microSD, **32 GB minimum** | 64 GB recommended, endurance-rated if you can |

!!! warning "A Raspberry Pi 4 will not work"
    The appliance keeps two copies of its operating system and switches between them, so an update that fails can fall back to the one that worked. That layout needs the Pi 5's bootloader. On a Pi 4 the card may simply never start, with nothing on screen to explain why.

!!! info "Why 32 GB is a floor, not a suggestion"
    The two operating-system copies and their boot partitions claim about 21 GB before any of your data is stored, so a 16 GB card cannot hold the image at all. A 64 GB card leaves roughly 38 GB for prints, models and history.

---

## What the appliance adds

- **Guided first boot.** No display or keyboard needed. Plug in ethernet, or join the appliance's own `Bambuddy-Setup` WiFi network and the setup page opens by itself.
- **Its own admin panel** on port `8001`, separate from Bambuddy, so you can still fix the box when Bambuddy is down.
- **Health-checked upgrades.** The Bambuddy container upgrade pulls, starts, and waits for a health check &mdash; and rolls back automatically if the new version doesn't come up.
- **Secondary IP aliases**, one per virtual printer that needs its own address on your LAN.
- **Tailscale**, ready to sign in from the panel, so you can reach the box and its virtual printers from anywhere without opening a router port.
- **A password of its own.** Every card generates a unique appliance password on its first boot, before SSH is allowed to accept a connection.
- **A factory reset** from the panel or over SSH, returning you to a fresh setup wizard.

---

## Start here

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### [:material-rocket-launch: Quick Start](quick-start.md)
Power on, run the wizard, reach Bambuddy. Fifteen minutes.
</div>

<div class="feature-card" markdown>
### [:material-view-dashboard: Admin Panel](admin-panel.md)
The four tabs: Dashboard, Network, Updates, Diagnostics.
</div>

<div class="feature-card" markdown>
### [:material-console: Command Line](cli.md)
`bambuddy-appliance` &mdash; everything the panel does, over SSH.
</div>

<div class="feature-card" markdown>
### [:material-update: Updates &amp; Backups](updates.md)
Two upgrade lanes, what a backup covers, and what it doesn't.
</div>

<div class="feature-card" markdown>
### [:material-shield-check: Registration](registration.md)
What a partner-built unit sends home, and why a downloaded one sends nothing.
</div>

<div class="feature-card" markdown>
### [:material-lifebuoy: Recovery](recovery.md)
Factory reset, lockouts, and what to try when it won't come up.
</div>

</div>

---

## How to get one

The appliance goes on sale in **October 2026** as an image download plus an annual subscription: Personal at &euro;79/year for non-commercial use, Business at &euro;249/year for commercial use with a named support channel and an agreed response time. One subscription covers one appliance, whatever number of printers you point it at.

If the subscription lapses, **the appliance keeps working exactly as it is.** What stops is updates and support, not the software you already have.

[Leave your email on bambuddy.cool](https://bambuddy.cool/appliance.html) to hear when it is available, and read the [printed quick start](https://bambuddy.cool/assets/downloads/bambuddy-appliance-quickstart.pdf) first &mdash; it is the full setup guide, and it costs nothing to find out whether this is something you want to do.

If you would rather build your own, everything Bambuddy needs is in the [Docker installation guide](../getting-started/docker.md). The appliance carries no exclusive features and never will.
