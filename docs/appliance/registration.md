---
title: Registration
description: What a Bambuddy Appliance sends home, what the licence key is for, why a self-built one sends nothing, and what happens when the registrar says no
---

# Registration

Registration is how an appliance is matched to its licence and offered updates. It is **not** a licence check on the software: Bambuddy is AGPL-3.0 and is never gated, whatever the registrar says about your box.

There are three kinds of unit, and they behave differently:

| Unit | Registers? | What authorises it |
|---|---|---|
| **Bought as a download** | Yes | The licence key from your purchase email |
| **Built by a reseller** | Yes | The batch identifier baked into their image |
| **Built by you from source** | **Never** | Nothing &mdash; it carries no batch and no key, so it contacts nothing |

A self-built appliance not contacting anything is not a setting you have to find and switch off. It is what "no batch identifier and no key" means in the code, and you can confirm it in a file (see [Checking for yourself](#checking-for-yourself)).

---

## The licence key

The key arrives by email after the purchase, twenty characters in four groups. The [setup wizard](quick-start.md#your-licence-key) asks for it, and the appliance sends it every time it registers or heartbeats.

Every customer downloads the **same image**, so the batch identifier inside it is worth nothing as authorisation &mdash; anyone who ever got hold of the file would have it. The purchase is what authorises, and the key is what carries the purchase.

!!! tip "Entering it is optional, at first boot and afterwards"
    Skip the screen and the appliance runs exactly as it would otherwise; it simply receives no appliance updates. The admin panel then shows an **Enter a key** notice until one is entered, and the key rides along with the next heartbeat. Nothing has to be reset or re-flashed.

    ```bash
    sudo bambuddy-appliance license XXXXX-XXXXX-XXXXX-XXXXX
    ```

    Setting it this way asks the registrar immediately, so a mistyped key tells you now rather than on a timer you cannot see.

---

## What is sent

| Field | What it is |
|---|---|
| `device_uuid` | An identifier derived from the board's serial number, so the same board is always the same device |
| `batch_id` | Which image the unit was flashed from |
| `license_key` | Your licence key, when one is set |
| `model` | The hardware model string, e.g. `Raspberry Pi 5 Model B` |
| `appliance_version` | Which version of the appliance layer is running |

**No printer data. No print history. No user accounts. No credentials.** Nothing about what you print, when, or with what.

The registrar records the connecting IP address as a salted hash, never in the clear, and uses it only to notice when one identity turns up from many places at once.

The unit claims once on first boot, then heartbeats roughly once a day. In between, an hourly timer asks one question &mdash; am I still entitled? &mdash; which writes nothing on the registrar and exists so that a revoked, renewed or upgraded unit finds out within the hour rather than at its next heartbeat.

!!! info "It never blocks anything"
    If the registrar is unreachable, the attempt is logged and retried later. Registration never blocks boot, and the appliance is fully usable whether or not it ever succeeds.

---

## What registration buys

One thing: **appliance updates**. The archive that serves the appliance package answers only a registered unit whose licence is active ([Updates](updates.md#upgrading-the-appliance-layer)).

Two things it does **not** touch:

- **Bambuddy itself.** It is never intercepted, gated or watermarked. The API, websockets and the camera stream are untouched, and the container is the ordinary AGPL one from `ghcr.io/maziggy/bambuddy`.
- **Debian security updates.** The archive's package index is open to everyone, so `apt` keeps working on a unit that is not entitled. Revocation stops the appliance layer, not the operating system.

A unit the registrar has **flagged or revoked** gets one more consequence: the appliance's own admin panel goes read-only behind a notice explaining why. That panel is the proprietary part, and it is the only part that is ever gated.

---

## When the registrar says no

A claim is refused when the batch is unknown, when the batch needs a key and none was sent, when the key is not recognised, when the licence's updates have ended or it was withdrawn, or when a reseller's batch has already used every unit it was cut for.

The reason is the registrar's own sentence, and the appliance keeps it:

```bash
sudo bambuddy-appliance license          # shows the key state and the last refusal
journalctl -u bambuddy-register.service -b
```

The admin panel shows the same sentence on its Dashboard, with an **Enter a key** button beside it &mdash; except after a re-flash, where a key is not what is missing and no button appears. A unit that has a key but has not reached the registrar yet reads **Waiting for the registrar**: that is a first boot without a network, and it resolves itself on the next tick. Retrying on its own will not fix a refusal &mdash; something has to change first, usually the key.

---

## Checking for yourself

```bash
cat /etc/bambuddy/provisioning.json
```

An empty `batch_id` is what a self-built appliance looks like. With no key either, the unit never contacts the registrar at all:

```json
{ "batch_id": "", "registrar_url": "https://appliance.bambuddy.cool" }
```

The rest of the state lives in `/var/lib/bambuddy/registrar/`, on the data partition:

| File | What it holds |
|---|---|
| `device-uuid` | This unit's identity |
| `token` | Its credential. Also the password apt uses for the archive |
| `license-key` | The licence key, `0600` |
| `entitlement`, `entitled-until`, `tier` | The registrar's last word on the licence: its state, the date updates run until, and the edition |
| `last-status` | `active`, `flagged` or `revoked` |
| `refused` | Why the last claim was turned away, when it was |
| `refused-kind` | `refused` (a key is what is missing) or `reclaim` (a re-flashed card) |

```bash
systemctl status bambuddy-register.timer          # is it scheduled?
journalctl -u bambuddy-register.service -b        # what did it do?
```

---

## Re-flashing a registered unit

A unit's identity comes from the board's serial number, so a re-flashed card comes back as **the same device** &mdash; but with no token, because the token lived on the card.

**With a licence key, it fixes itself.** The key is the proof of purchase, it is your secret, and it is not readable off the board &mdash; so a unit that comes back presenting the key it activated with re-claims on its next tick, with nobody pressing anything.

Without a key &mdash; a reseller unit, where the board serial is the only identifier &mdash; the registrar will not hand over a replacement, because a serial is readable by anyone holding the board. The unit is refused, and says so, until an operator opens a one-shot re-claim window for it. It then mints and stores a fresh token by itself; there is nothing for you to copy anywhere.

!!! info "Re-claiming restores a credential, not an entitlement"
    A revoked unit that re-claims is still revoked. And re-flashing never affects Bambuddy's own data, which the [backup](updates.md#backups) covers separately.
