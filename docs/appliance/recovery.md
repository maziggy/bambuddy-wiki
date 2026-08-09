---
title: Recovery
description: Factory reset, lockouts, and what to try when the Bambuddy Appliance won't come up
---

# Recovery

---

## Factory reset

Two ways in, both ending in the same wipe.

=== ":material-view-dashboard: Admin panel"

    **Dashboard &rsaquo; Factory reset**, on the panel at `:8001`. The normal path. It asks twice, because there is no undo.

=== ":material-console: Over SSH"

    ```bash
    sudo bambuddy-appliance factory-reset
    ```

    Prompts for a typed `YES` unless you pass `--yes`. This is the path when the panel itself is unreachable.

The appliance stops Bambuddy, wipes its user data, forgets the first-boot marker, and reboots into a fresh setup wizard &mdash; exactly as a newly flashed card would.

| Erased | Kept |
|---|---|
| Bambuddy's database &mdash; printers, queue, history, user accounts | The operating system and Bambuddy itself |
| All uploads, archives, and timelapses | Partner branding |
| Session and encryption secrets | The registration identity, on a partner unit |
| Saved WiFi credentials | |
| Hostname, timezone, locale | |
| **The appliance password** | |
| SSH host keys (regenerated on next boot) | |

!!! success "It resets the appliance password too"
    A factory reset recovers a forgotten **Bambuddy** account, because those live in the database it erases &mdash; and it now also recovers a forgotten **appliance** password, the one covering the admin panel, the console, and SSH.

    This changed. The password used to survive a reset, because clearing it would have left the panel failing closed on `:8001` with no way back in while the wizard's password step was skippable. That step is mandatory now, so the wizard the reset reboots into is guaranteed to set a new one.

    Between the reboot and the end of setup the panel answers `503`. That is the wizard's window, and finishing the wizard is what reopens it. The new appliance password is written to `bambuddy-password.txt` on the boot partition, as on a freshly flashed card.

!!! danger "There is no undo, and no confirmation prompt beyond the two asks"
    Your print history is gone. Take a [backup](updates.md#backups) while things are working, not when they aren't.

!!! info "There is no reset button, and no reset page in Bambuddy"
    Earlier appliances documented a recessed GPIO button on the case. It was never part of a shipped unit and the handler has been removed &mdash; the two paths above are the only ones. There is also no factory-reset URL anywhere inside Bambuddy itself; the reset belongs to the appliance panel on `:8001`, not to Bambuddy on `:8000`.

---

## Lockouts

=== "Forgot the appliance password"

    If you can still get in over SSH, change it in place and keep your data:

    ```bash
    sudo bambuddy-appliance set-admin-password
    ```

    If SSH is unreachable too, a **factory reset** clears it and the wizard asks for a new one &mdash; at the cost of your Bambuddy data. Restore a [backup](updates.md#backups) afterwards.

    If the card has never finished its setup wizard, the password it generated for itself is in `bambuddy-password.txt` on the boot partition. Power the appliance down, put the card in any computer, and read it.

=== "Admin panel says the password isn't set"

    The panel fails closed when `/etc/bambuddy/admin-auth` is missing. That is the expected state between a factory reset and the end of the setup wizard &mdash; finish the wizard on `:8000` and the panel comes back.

    Outside that window, set one directly:

    ```bash
    sudo bambuddy-appliance set-admin-password
    ```

=== "Forgot the Bambuddy password"

    That's Bambuddy's own account, not the appliance's. See [Authentication](../features/authentication.md).

---

## When it won't come up

| Symptom | Try |
|---|---|
| `bambuddy.local` doesn't resolve | Find the IP in your router's client list and use `http://<ip>:8000` |
| Wizard won't advance past Welcome | It's waiting for the clock. Give it a minute with a working uplink. |
| Setup WiFi network never appears | Wait two minutes for first boot. If ethernet is plugged in, the appliance uses it and never raises the access point. |
| Bambuddy is down, panel still works | **Diagnostics** &rarr; Bambuddy log. Then `sudo bambuddy-appliance restart`. |
| Panel is down too | SSH in and check `systemctl status bambuddy-admin` |
| Everything is unreachable | SSH in and `sudo bambuddy-appliance factory-reset`, or re-flash the card |
| Random slowdowns, printer disconnects | **Dashboard** &rarr; check the *since boot* undervoltage and throttling flags. A weak USB-C supply will set them. |

---

## Reporting a problem

Two support bundles exist, and a good bug report usually wants both:

- The **appliance** bundle, from the panel's **Diagnostics** tab. Host-side: service logs, container state, network configuration.
- **Bambuddy's** own bundle, from the **System** page inside Bambuddy. Application-side.

Both strip credentials before they're written, so they're safe to attach to a public issue.

File it at [github.com/maziggy/bambuddy/issues](https://github.com/maziggy/bambuddy/issues). Say which appliance image version you're on &mdash; the Dashboard shows it.
