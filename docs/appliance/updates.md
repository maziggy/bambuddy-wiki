---
title: Updates & Backups
description: How the Bambuddy Appliance upgrades itself, what a backup covers, and what it leaves behind
---

# Updates &amp; Backups

The appliance is three layers, and each updates differently.

| Layer | How it updates | Reversible? |
|---|---|---|
| **Bambuddy** (the container) | In place, from the Updates tab | Yes &mdash; automatic rollback |
| **Operating system** (Debian packages) | In place, from the Updates tab | No |
| **Appliance image** (the card itself) | Re-flash | N/A |

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

## Upgrading the OS

!!! danger "There is no rollback"
    `apt full-upgrade` cannot be undone. The appliance refuses to start one with less than 1 GB free and warns when the clock isn't synchronised, but past that point you are trusting Debian. **Take a backup first.**

From the panel: **Updates** &rarr; **Update OS packages**. It will tell you if a reboot is required.

---

## Backups

Bambuddy backs itself up from **Settings** &rarr; **Backup &amp; Restore**. You get one zip containing the database and every data directory, and it restores onto either SQLite or PostgreSQL &mdash; the export is portable between them.

Back up before an OS upgrade, before re-flashing, and on a schedule. Keep the file **somewhere other than the appliance**; a backup on the card you are about to overwrite is not a backup.

See [Backup &amp; Restore](../features/backup.md) for the full picture, including scheduled backups.

### What the backup does not contain

The backup covers *Bambuddy*. It does not cover the *appliance*:

- the admin panel password
- the Tailscale login
- your WiFi credentials
- any secondary IP aliases

Restore a backup onto a freshly flashed card and you get all your printers, queue, and print history back &mdash; and you redo the appliance setup by hand. Budget ten minutes for that, and don't discover it at the worst possible moment.

---

## Re-flashing the image

The appliance image carries the operating system, the setup wizard, the admin panel, and the CLI. It is versioned separately from Bambuddy, and the version you flashed is shown on the Dashboard.

Almost nothing requires a re-flash. The container lane covers Bambuddy, and the OS lane covers Debian. In practice you re-flash for two reasons: the SD card died, or a new image carries a change the two in-place lanes cannot deliver.

!!! warning "Re-flashing erases the card"
    Everything on it. The appliance's data lives on that card and nowhere else.

    1. Back up Bambuddy, and copy the file off the appliance.
    2. Flash the new image.
    3. Run the setup wizard again.
    4. Restore the backup.

!!! info "Reseller units lose their identity on a re-flash"
    A reseller appliance's registration identity lives on the card. Flashing a generic image over it produces an unregistered unit. See [Registration](registration.md) before you re-flash one.

---

## What upgrades cannot break

Neither in-place lane touches `/var/lib/bambuddy/`, where the database and uploads live. An upgrade that fails leaves your data exactly as it was &mdash; that is why the Bambuddy lane can roll back at all.

The one thing that *does* destroy data is re-flashing, and the [factory reset](recovery.md). Both are deliberate, and both start with a backup.
