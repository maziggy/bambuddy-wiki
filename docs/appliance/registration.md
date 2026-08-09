---
title: Registration
description: What a partner-built Bambuddy Appliance sends home, why a downloaded or self-built one sends nothing, and how the registration notice works
---

# Registration

**Most appliances never register.** The image sold as a download and any image you build yourself both carry an empty batch identifier, and a unit with no batch never contacts anything. That is not a setting you have to find and switch off &mdash; it is the default state, and this page is here mainly so you can confirm it.

Registration exists for **partner batches**: units built by a partner with a batch identifier baked in, so the unit can be identified for warranty and offered a curated update feed.

---

## What is sent

A partner unit sends exactly four things, once on first boot and then roughly once a day:

| Field | What it is |
|---|---|
| `device_uuid` | A random identifier the unit generates for itself |
| `batch_id` | Which partner batch the unit came from |
| `model` | The hardware model string, e.g. `Raspberry Pi 5 Model B` |
| `appliance_version` | Which appliance image is running |

A downloaded or self-built unit sends **none** of it, because the check that decides whether to register at all is `batch_id` being non-empty.

**No printer data. No print history. No user accounts. No credentials.** Nothing about what you print, when, or with what.

The registrar records the connecting IP address as a salted hash, never in the clear, and uses it only to notice when one identity appears from many places at once.

!!! info "It never blocks anything"
    If the registrar is unreachable, the attempt is logged and retried later. Registration never blocks boot, and the appliance is fully usable whether or not it ever succeeds.

---

## Checking for yourself

```bash
cat /etc/bambuddy/provisioning.json
```

An empty `batch_id` means the unit will never contact the registrar &mdash; this is what a downloaded or self-built appliance looks like:

```json
{ "batch_id": "", "registrar_url": "https://appliance.bambuddy.cool" }
```

To turn a partner unit into a non-registering one, blank that field. You will also lose the curated update feed and the warranty identity that goes with it.

```bash
systemctl status bambuddy-register.timer          # is it scheduled?
journalctl -u bambuddy-register.service -b        # what did it do?
```

---

## The registration notice

Bambuddy on the appliance sits behind a small proxy. When a **partner** unit has not registered, that proxy shows a notice on top-level page loads. A downloaded or self-built unit never sees it.

| State | What you see |
|---|---|
| Downloaded, self-built, or registered | Nothing. The proxy is transparent. |
| Partner unit, first 14 days | Nothing. There's a grace period. |
| Partner unit, unregistered after 14 days | A notice you can dismiss |
| Unit reported as non-genuine | A notice you cannot dismiss |

!!! success "Bambuddy is never locked"
    The notice only ever intercepts top-level page navigations in a browser. The API, websockets, and the camera stream always pass straight through, so Home Assistant, your slicer, and any integration keep working regardless.

    Bambuddy is AGPL-3.0 software. The notice belongs to the appliance wrapper, not to Bambuddy, and it never restricts a right the AGPL grants you. You can pull the same container and run it anywhere, including on the appliance itself.

---

## Re-flashing a partner unit

!!! warning "A re-flash creates a new, unregistered unit"
    A partner unit's identity &mdash; its `device_uuid` and its token &mdash; lives on the SD card. Flashing a generic image over it does not carry that identity across. The appliance comes back as an unregistered unit, its old registration is orphaned, and the notice will eventually appear.

    If you need to re-flash a partner appliance, contact whoever sold it to you first. This does not apply to a downloaded appliance, which has no registration to lose.

Backing up Bambuddy does not help here: the [backup](updates.md#backups) covers Bambuddy's data, not the appliance's identity.
