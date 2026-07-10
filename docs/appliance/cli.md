---
title: Command Line
description: The bambuddy-appliance command - upgrades, network aliases, Tailscale, and registration over SSH
---

# Command Line

Everything the [admin panel](admin-panel.md) does, the `bambuddy-appliance` command does too &mdash; the panel is a thin layer over this CLI, so there is exactly one code path that touches the appliance's state.

SSH is enabled by default:

```bash
ssh pi@bambuddy.local
```

Most subcommands need root. Prefix them with `sudo`.

---

## Status and restart

```bash
bambuddy-appliance status      # docker compose ps
bambuddy-appliance restart     # restart the Bambuddy stack
```

---

## Upgrades

```bash
# Health-checked, with automatic rollback if it doesn't come up
sudo bambuddy-appliance upgrade-bambuddy v0.2.5

# Debian packages. No rollback.
sudo bambuddy-appliance upgrade-os

# The current job's phase, as JSON
bambuddy-appliance upgrade-status
```

Only one upgrade may run at a time; a second one exits rather than racing the first. Jobs run detached, so an SSH disconnect mid-upgrade doesn't kill them &mdash; reconnect and poll `upgrade-status`.

See [Updates &amp; Backups](updates.md) for what each lane can and cannot undo.

---

## Network aliases

Extra addresses on the LAN interface, one per [virtual printer](../features/virtual-printer.md) that needs its own.

```bash
bambuddy-appliance net-aliases                        # list, as JSON
sudo bambuddy-appliance net-alias-add 192.168.1.50/24
sudo bambuddy-appliance net-alias-remove 192.168.1.50/24
```

Aliases are additive and persist across reboots. The primary DHCP address, the gateway, and the interface's own configuration are never modified &mdash; by design, so this command cannot strand the appliance off its network.

Addresses in reserved ranges are rejected before anything is applied.

---

## Tailscale

```bash
sudo bambuddy-appliance tailscale up       # prints a sign-in URL
bambuddy-appliance tailscale status        # tailnet IP, as JSON
sudo bambuddy-appliance tailscale down     # disconnect, stay logged in
sudo bambuddy-appliance tailscale logout   # forget the tailnet entirely
```

`tailscale up` prints a URL to open in a browser. Tailscale's own page handles first-time sign-up, so no console access or API key is needed. Once connected, `status` reports the `100.x` address to paste into a slicer.

---

## The admin password

```bash
sudo bambuddy-appliance set-admin-password
```

Reads the new password from standard input rather than an argument, so it never lands in `ps`, your shell history, or a process listing. On a terminal it prompts; in a script, pipe it:

```bash
printf '%s\n' "$NEW_PASSWORD" | sudo bambuddy-appliance set-admin-password
```

Only the hash is stored, in `/etc/bambuddy/admin-auth`. This is the same password the setup wizard sets, and it secures the admin panel, the console, and SSH.

---

## Registration

```bash
sudo bambuddy-appliance register
```

Claims the unit with the fleet registrar, or sends a heartbeat if it's already claimed, and applies the resulting [registration gate](registration.md) mode.

On a self-built image this command does nothing at all &mdash; there is no batch identifier, so it exits without contacting anything. A timer runs it in the background on reseller units; you should not normally need to call it by hand.

---

## Files it owns

| Path | What |
|---|---|
| `/etc/bambuddy/bambuddy.env` | Runtime environment for the Bambuddy container |
| `/etc/bambuddy/docker-compose.yml` | The pinned container tag |
| `/etc/bambuddy/docker-compose.override.yml` | Yours to write &mdash; e.g. bind-mounting extra file-manager roots |
| `/etc/bambuddy/admin-auth` | Admin panel password hash |
| `/etc/bambuddy/provisioning.json` | Reseller batch identifier, if any |
| `/var/lib/bambuddy/` | Bambuddy's database and uploads |

!!! warning "Don't hand-edit the compose file's image tag"
    `upgrade-bambuddy` rewrites it, and it is how the appliance knows what to roll back to.
