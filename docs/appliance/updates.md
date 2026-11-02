---
title: Updates & Backups
description: How the Bambuddy Appliance upgrades itself, what a backup covers, and what it leaves behind
---

# Updates &amp; Backups

The appliance is five layers, and each updates differently.

| Layer | How it updates | Reversible? |
|---|---|---|
| **Bambuddy** (the container) | In place, from the Updates tab | Yes &mdash; automatic rollback |
| **PostgreSQL** (the second container) | With the image; the major is pinned | N/A |
| **The appliance layer** (wizard, panel, CLI, units) | In place, from the Updates tab | Yes &mdash; automatic rollback |
| **Operating system** (Debian packages) | In place, from the Updates tab | No |
| **Kernel, firmware, partition layout** | Re-flash | N/A |

Nothing installs itself. Every lane is a button you press, with one exception: a freshly flashed card brings its appliance layer up to date once, on the first boot that has a network, so a card written from an older image does not start out behind.

---

## Upgrading Bambuddy

This is the one you'll do most, and it's the safe one.

The appliance pulls the new container, repins the compose file, starts it, and waits for Bambuddy's health check to pass. If it doesn't, the previous tag goes back and gets restarted. Your database and uploads are never touched &mdash; only the image changes.

From the panel: **Updates** &rarr; **Check for updates** &rarr; **Upgrade**.

From the shell:

```bash
sudo bambuddy-appliance upgrade-bambuddy v0.2.5
```

If the pull itself fails &mdash; no network, bad tag &mdash; nothing changes at all. The compose file still points at the version you were running.

---

## Upgrading the appliance layer

The wizard, the admin panel, the CLI, the systemd units and the gate configuration &mdash; everything around Bambuddy, and where nearly every fix lands.

From the panel: **Updates** &rarr; **Check for updates** &rarr; **Update appliance**. From the shell:

```bash
sudo bambuddy-appliance upgrade-check-appliance   # is there one? (JSON)
sudo bambuddy-appliance upgrade-appliance
```

It is a Debian package from Bambuddy's own signed archive, installed in place &mdash; no reboot, no card. The lane health-checks Bambuddy afterwards and puts the previous version back if it does not come up, the same as the Bambuddy lane.

!!! info "This lane needs an active licence"
    The archive hands the package only to a registered appliance that is entitled to it. An expired or revoked unit keeps running and keeps getting Debian security updates; it stops getting the appliance layer. See [Registration](registration.md).

!!! success "The signature is what is trusted, not the server"
    The archive is generated and signed away from the machine that serves it, and the appliance verifies it against a key baked into the image. Someone who took over the archive host could delete or corrupt packages, but not forge an update.

---

## Upgrading the OS

!!! danger "There is no rollback"
    `apt full-upgrade` cannot be undone. The appliance refuses to start one with less than 1 GB free and warns when the clock isn't synchronised, but past that point you are trusting Debian. **Take a backup first.**

From the panel: **Updates** &rarr; **Update OS packages**. It will tell you if a reboot is required.

---

## Backups

Bambuddy backs itself up from **Settings** &rarr; **Backup &amp; Restore**. You get one zip containing the database and every data directory, and it restores onto either SQLite or PostgreSQL &mdash; the export is portable between them.

Back up before an OS upgrade, before re-flashing, and on a schedule. Keep the file **somewhere other than the appliance**; a backup on the card you are about to overwrite is not a backup.

See [Backup &amp; Restore](../features/backup.md) for the full picture, including scheduled backups.

The backups Bambuddy saves on the appliance itself &mdash; **Scheduled Backups**, on the schedule or with **Run Now** &mdash; can also be downloaded from the [admin panel](admin-panel.md#bambuddy-backup), under **Dashboard** &rarr; **Bambuddy Backup**. That works when Bambuddy is down, which is when you need it most.

### What the backup does not contain

The backup covers *Bambuddy*. It does not cover the *appliance*:

- the admin panel password
- the Tailscale login
- your WiFi credentials
- any secondary IP aliases

Restore a backup onto a freshly flashed card and you get all your printers, queue, and print history back &mdash; and you redo the appliance setup by hand. Budget ten minutes for that, and don't discover it at the worst possible moment.

---

## Re-flashing the image

The image carries the operating system, the appliance layer and the partition layout. The version of the appliance layer you are running is shown on the Dashboard.

Very little requires a re-flash. Bambuddy, the appliance layer and Debian all update in place. In practice you re-flash for two reasons: the SD card died, or a change below the appliance layer &mdash; the kernel, the firmware, the partition table &mdash; which no in-place lane can deliver.

!!! warning "Re-flashing erases the card"
    Everything on it. The appliance's data lives on that card and nowhere else.

    1. Back up Bambuddy, and copy the file off the appliance.
    2. Flash the new image.
    3. Run the setup wizard again.
    4. Restore the backup.

!!! info "A re-flashed unit has to be let back in"
    The identity comes from the board, so the appliance comes back as the same device &mdash; but its credential lived on the card, and the registrar will not replace one on the strength of a serial number. An operator opens a one-shot re-claim window, and the unit takes a fresh credential by itself. See [Registration](registration.md#re-flashing-a-registered-unit).

    Keep your licence key: the wizard asks for it again.

---

## What upgrades cannot break

No in-place lane touches `/var/lib/bambuddy/`, where the database and uploads live. An upgrade that fails leaves your data exactly as it was &mdash; that is why the Bambuddy lane can roll back at all.

The one thing that *does* destroy data is re-flashing, and the [factory reset](recovery.md). Both are deliberate, and both start with a backup.
