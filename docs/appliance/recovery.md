---
title: Recovery
description: Factory reset, lockouts, and what to try when the Bambuddy Appliance won't come up
---

# Recovery

---

## Factory reset

Hold the recessed **reset button** on the case for **ten seconds**.

The appliance stops Bambuddy, wipes its user data, forgets the first-boot marker, and reboots into a fresh setup wizard &mdash; exactly as a new appliance would. This is the path of last resort, and it works when nothing else does: when the network is misconfigured, when the web interface is unreachable, when both ethernet and WiFi have failed.

| Erased | Kept |
|---|---|
| Bambuddy's database &mdash; printers, queue, history, user accounts | The operating system and Bambuddy itself |
| All uploads, archives, and timelapses | Partner branding |
| Session and encryption secrets | The appliance's warranty identity |
| Saved WiFi credentials | **The appliance password** |
| Hostname, timezone, locale | |
| SSH host keys (regenerated on next boot) | |

!!! warning "It does not reset the appliance password"
    A factory reset recovers a forgotten **Bambuddy** account, because those live in the database it erases. It does **not** clear the appliance password &mdash; the one covering the admin panel, the console, and SSH. Clearing it would leave the panel failing closed on `:8001` with no way back in, since the wizard's password step can be skipped.

    To change that password, use `sudo bambuddy-appliance set-admin-password` over SSH.

!!! danger "There is no undo, and no confirmation prompt"
    Ten seconds of button, and your print history is gone. Take a [backup](updates.md#backups) while things are working, not when they aren't.

The button cannot fire accidentally. It is ignored for the first thirty seconds after boot, so a stuck button can't put the appliance in a reset loop. A press shorter than ten seconds does nothing, and releasing early aborts cleanly.

!!! warning "No button, no reset"
    The button is the **only** way to trigger a factory reset. There is no web page, no URL, and no menu item for it anywhere in Bambuddy.

    If you flashed the image onto a bare Pi without a partner case, wire a momentary switch across **GPIO21** and **GND** (pins 40 and 39 on the header). Without one, your only recovery options are SSH or re-flashing the card.

---

## Lockouts

=== "Forgot the appliance password"

    Over SSH, if you can still get in:

    ```bash
    sudo bambuddy-appliance set-admin-password
    ```

    **The reset button will not help.** It does not clear this password. If SSH is also unreachable, the only way back is to re-flash the card.

=== "Admin panel says the password isn't set"

    The panel fails closed. Either the setup wizard's password step was skipped, or `/etc/bambuddy/admin-auth` was removed. Set one:

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
| Everything is unreachable | The reset button |
| Random slowdowns, printer disconnects | **Dashboard** &rarr; check the *since boot* undervoltage and throttling flags. A weak USB-C supply will set them. |

---

## Reporting a problem

Two support bundles exist, and a good bug report usually wants both:

- The **appliance** bundle, from the panel's **Diagnostics** tab. Host-side: service logs, container state, network configuration.
- **Bambuddy's** own bundle, from the **System** page inside Bambuddy. Application-side.

Both strip credentials before they're written, so they're safe to attach to a public issue.

File it at [github.com/maziggy/bambuddy/issues](https://github.com/maziggy/bambuddy/issues). Say which appliance image version you're on &mdash; the Dashboard shows it.
