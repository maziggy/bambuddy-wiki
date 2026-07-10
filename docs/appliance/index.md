---
title: Bambuddy Appliance
description: The ready-to-run Bambuddy box - a Raspberry Pi 5 appliance with Bambuddy preinstalled, a guided setup wizard, and its own admin panel
---

# Bambuddy Appliance

The Bambuddy Appliance is a small Raspberry Pi 5 box that arrives with Bambuddy already installed. You plug it in, walk through a five-step setup wizard in your browser, and start adding printers. There is no Docker to install, no compose file to edit, and no cloud account to create.

It runs the same Bambuddy you would install yourself &mdash; same features, same AGPL licence, same local-only operation. What the appliance adds is everything *around* Bambuddy: first-boot bring-up, a captive-portal WiFi setup, an admin panel for the box itself, health-checked upgrades with automatic rollback, and a hardware factory-reset button.

---

## Who it's for

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-package-variant-closed: You want it to just work
Buy it from an authorised reseller, plug it in, and skip the self-hosting entirely. Support comes from the people who built it.
</div>

<div class="feature-card" markdown>
### :material-hammer-wrench: You already self-host
You probably don't need this. A Docker install gives you the same Bambuddy. The appliance is about the box, not the software.
</div>

</div>

!!! info "Bambuddy stays Bambuddy"
    The application on the appliance is the ordinary AGPL-3.0 Bambuddy container from `ghcr.io/maziggy/bambuddy`. You can inspect it, replace it, or run it somewhere else. The appliance wrapper &mdash; the wizard, the admin panel, the CLI &mdash; is separately licensed, and it never restricts what the AGPL grants you.

---

## What's in the box

| | |
|---|---|
| **Computer** | Raspberry Pi 5, 4 GB |
| **Power** | 27 W USB-C supply |
| **Cooling** | Active Cooler |
| **Storage** | microSD with the appliance image |
| **Reset** | Recessed button on the case |

That's the supported baseline. The image runs on lesser hardware, but a 4 GB Pi 5 with active cooling is what gets tested and supported.

---

## What the appliance adds

- **Guided first boot.** No display or keyboard needed. Plug in ethernet, or join the appliance's own `Bambuddy-Setup` WiFi network and the setup page opens by itself.
- **Its own admin panel** on port `8001`, separate from Bambuddy, so you can still fix the box when Bambuddy is down.
- **Health-checked upgrades.** The Bambuddy container upgrade pulls, starts, and waits for a health check &mdash; and rolls back automatically if the new version doesn't come up.
- **Secondary IP aliases**, one per virtual printer that needs its own address on your LAN.
- **Tailscale**, ready to sign in from the panel, so you can reach the box and its virtual printers from anywhere without opening a router port.
- **A real reset button.** Ten seconds, and you're back to a fresh setup wizard.

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
What a reseller unit sends home, and what a self-built one doesn't.
</div>

<div class="feature-card" markdown>
### [:material-lifebuoy: Recovery](recovery.md)
Factory reset, lockouts, and what to try when it won't come up.
</div>

</div>

---

## How to get one

The appliance is sold through authorised resellers. It is not currently available as a direct download.

If you want to build your own, everything Bambuddy needs is in the [Docker installation guide](../getting-started/docker.md) &mdash; the appliance carries no exclusive features.
