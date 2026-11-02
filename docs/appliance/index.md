---
title: Bambuddy Appliance
description: The Bambuddy Appliance - a Raspberry Pi 5 image with Bambuddy preinstalled, a guided setup wizard, and its own admin panel
---

# Bambuddy Appliance

The Bambuddy Appliance is a Raspberry Pi 5 image with Bambuddy already installed and configured. You write it to a memory card, walk through a setup wizard in your browser, and start adding printers. There is no Docker to install, no compose file to edit, and no cloud account to create.

You supply the hardware. The appliance is sold as an **image download you pay for once** &mdash; nothing is shipped, and the Raspberry Pi is yours rather than rented.

It runs the same Bambuddy you would install yourself &mdash; same features, same AGPL licence, same local-only operation. What the appliance adds is everything *around* Bambuddy: first-boot bring-up, a captive-portal WiFi setup, an admin panel for the box itself, health-checked container upgrades with automatic rollback, and an A/B operating system that can fall back to the copy that worked.

---

## Who it's for

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-package-variant-closed: You want it to just work
Flash one card, answer five questions, and skip the self-hosting entirely. Updates are tested together and undo themselves, and support comes from the person who builds it.
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

Four parts, all of them stocked by every Raspberry Pi dealer &mdash; plus a case you print yourself.

| | | |
|---|---|---|
| **Computer** | Raspberry Pi **5**, 4 GB | 8 GB for a large fleet &mdash; RAM only; the database does not depend on it |
| **Power** | The official 27 W USB-C supply | Undervoltage shows up as dropped printers, not as an error |
| **Cooling** | The official Active Cooler | The box runs continuously; a throttled Pi looks like slow software |
| **Storage** | microSD, **64 GB minimum** | Endurance-rated if you can; the appliance refuses to start on anything smaller |
| **Case** | Print it yourself &mdash; [download the 3MF](https://bambuddy.cool/assets/downloads/bambuddy-appliance-case.3mf) | The Pi is held in it by 4 &times; M2.5 &times; 5 screws and 4 &times; M2.5 nuts |

!!! warning "A Raspberry Pi 4 will not work"
    The appliance keeps two copies of its operating system and switches between them, so an update that fails can fall back to the one that worked. That layout needs the Pi 5's bootloader. On a Pi 4 the card may simply never start, with nothing on screen to explain why.

!!! info "Why 64 GB is a floor, not a suggestion"
    The two operating-system copies and their boot partitions claim about 21 GB before any of your data is stored. A 64 GB card leaves roughly 38 GB for prints, models and history; a 32 GB one is full before you print anything, and the [hardware check](admin-panel.md#the-hardware-check) refuses to start Bambuddy on it.

---

## What the appliance adds

- **Guided first boot.** No display or keyboard needed. Plug in ethernet, or join the appliance's own `Bambuddy-Setup` WiFi network and the setup page opens by itself.
- **Its own admin panel** on port `8001`, separate from Bambuddy, so you can still fix the box when Bambuddy is down.
- **PostgreSQL, on every unit.** No database to choose and none to migrate to later: see [below](#the-database).
- **Health-checked upgrades.** The Bambuddy container upgrade pulls, starts, and waits for a health check &mdash; and rolls back automatically if the new version doesn't come up.
- **Secondary IP aliases**, one per virtual printer that needs its own address on your LAN.
- **Tailscale**, ready to sign in from the panel, so you can reach the box and its virtual printers from anywhere without opening a router port.
- **A password of its own.** Every card generates a unique appliance password on its first boot, before SSH is allowed to accept a connection.
- **A factory reset** from the panel or over SSH, returning you to a fresh setup wizard.

---

## The database

The appliance runs **PostgreSQL**, on every unit, whatever the size of the farm. You do not choose it, set it up or migrate to it.

A self-hosted Bambuddy defaults to SQLite and [moves to PostgreSQL above roughly ten printers](../reference/farm-sizing.md#move-the-database-to-postgresql), because SQLite allows one writer at a time and a farm's dispatch-and-completion peaks turn that into `database is locked`. That failure arrives mid-print, months after setup. An appliance exists to take that decision away, so it ships the engine that scales rather than the one that is smaller.

It runs as a second container beside Bambuddy, its credentials are generated per machine on first boot, and its data lives on the data partition with everything else &mdash; so an image update leaves it alone and a [factory reset](recovery.md) erases it. Both containers are shown on the [admin panel's Dashboard](admin-panel.md#the-containers), and both can be tailed from Diagnostics.

!!! info "Pointing it at your own PostgreSQL"
    Set `DATABASE_URL` in `/etc/bambuddy/bambuddy.env`. That file is read after the generated one, so your value wins. Nothing else changes.

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
The licence key, what the appliance sends home, and what a refusal means.
</div>

<div class="feature-card" markdown>
### [:material-lifebuoy: Recovery](recovery.md)
Factory reset, lockouts, and what to try when it won't come up.
</div>

</div>

---

## How to get one

The appliance is sold at [get.bambuddy.cool](https://get.bambuddy.cool/#pricing) as an image download, in three editions:

| Edition | Price | Updates | Support |
|---|---|---|---|
| **Personal** | &euro;79 once | 12 months | 12 months, through GitHub issues |
| **Lifetime** | &euro;199 once | for as long as the appliance is developed | 12 months, through GitHub issues |
| **Business** | &euro;249/year | while subscribed | while subscribed, by email, handled first, with a stated response time |

Personal and Lifetime are for non-commercial use; commercial use needs Business. Support for Personal and Lifetime runs through the [public issue tracker](https://github.com/maziggy/bambuddy/issues), so never post your licence key there; questions about your key, an order or a refund go to support@bambuddy.cool for every edition. One purchase covers one appliance, whatever number of printers you point it at.

Your licence key arrives by email after the purchase. The setup wizard asks for it, the appliance sends it when it registers, and that is what opens the update channel. Entering it is optional: skip the screen and the box runs exactly the same, it simply receives no appliance updates until you add the key in the panel.

When the updates end, **the appliance keeps working exactly as it is.** What stops is updates and support, not the software you already have. A Personal appliance can be upgraded to Lifetime later: write to support@bambuddy.cool.

Before you buy, read the [printed quick start](https://bambuddy.cool/assets/downloads/bambuddy-appliance-quickstart.pdf) first &mdash; it is the full setup guide, and it costs nothing to find out whether this is something you want to do.

If you would rather build your own, everything Bambuddy needs is in the [Docker installation guide](../getting-started/docker.md). The appliance carries no exclusive features and never will.
