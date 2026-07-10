---
title: Quick Start
description: Power on the Bambuddy Appliance, run the setup wizard, and reach Bambuddy
---

# Quick Start

From a sealed box to a running Bambuddy in about twenty minutes.

---

## Before you begin

Two of the things you need live on your **printer**, not on the appliance. Getting them ready first makes the rest quick.

- [x] An **SD card inserted** in the printer. Without one, Bambuddy cannot transfer files or start prints.
- [x] **LAN Only Mode** switched on, from the printer's touchscreen.
- [x] **Developer Mode** switched on. It appears once LAN Only Mode is on. Without it Bambuddy can watch your printer but not control it.
- [x] The printer's **access code** (eight characters, shown when Developer Mode is enabled), its **IP address**, and its **serial number**.

See [Enabling Developer Mode](../getting-started/index.md#enabling-developer-mode) for the full walkthrough.

!!! warning "The access code changes"
    Every time you toggle LAN Only Mode or Developer Mode off and on again, the printer generates a new access code. If Bambuddy stops connecting weeks later, this is usually why.

---

## 1. Power on

Connect the supplied 27 W supply. The first boot takes a couple of minutes while the pre-baked Bambuddy container is loaded from the card.

Then pick whichever applies:

=== ":material-ethernet: With an ethernet cable"

    The appliance takes a DHCP lease and serves the setup wizard straight away. Open:

    ```
    http://bambuddy.local:8000
    ```

    If that name doesn't resolve, find the appliance's IP in your router's client list and browse to `http://<ip>:8000`.

=== ":material-wifi: Without ethernet"

    The appliance raises its own WiFi access point:

    | | |
    |---|---|
    | Network | `Bambuddy-Setup` |
    | Setup page | `http://bambuddy.local:8000` |

    Join it from a phone or laptop. Most devices open the setup page by themselves through the captive-portal prompt. If yours doesn't, browse to the address above.

!!! info "Why the wizard waits"
    Before it lets you continue, the wizard waits for the clock to synchronise. A Raspberry Pi has no battery-backed clock, and an unsynchronised clock breaks every HTTPS connection the appliance needs to make &mdash; including the one that fetches your printers' certificates. This usually takes a few seconds.

---

## 2. The setup wizard

The wizard shows one screen at a time. On ethernet, the WiFi screen is skipped &mdash; you're already connected.

| Screen | What it asks |
|---|---|
| **Welcome** | Nothing. Continue. |
| **Connect to your WiFi** | Pick your network and enter its password. **Skipped on ethernet.** |
| **Name your appliance** | A device name (used as its `.local` hostname), your time zone, and the language for Bambuddy. All changeable later. |
| **Set a password** | See below. Don't skip it. |
| **Almost there** | The hand-off. Read the next section before pressing the button. |

---

## 3. Set the password

!!! warning "The password step is optional, and skipping it leaves the box open"
    The appliance ships with the password printed on the sticker underneath it. That single password unlocks **three** things:

    - the admin panel on port `8001`, as user `admin`
    - the local console
    - **SSH on port 22, which is enabled by default and reachable from your entire LAN**

    Setting a password in the wizard replaces it in all three places. Do it before the appliance is reachable from any network you don't fully control.

You can change it later at any time:

```bash
sudo bambuddy-appliance set-admin-password
```

---

## 4. The network switch

This is the only confusing part of the whole process, and it only happens if you set up over WiFi. Read it *before* you press the button.

=== ":material-wifi: If you set up over WiFi"

    1. Press **Apply and switch networks**. The appliance shows its progress: loading Bambuddy, joining your WiFi, starting up.
    2. **The `Bambuddy-Setup` network disappears.** This is supposed to happen &mdash; the appliance has stopped being a hotspot and joined your network instead.
    3. **Your phone or laptop goes quiet. Reconnect it to your normal WiFi.** It will not do this by itself, and nothing is broken.
    4. Open the address the wizard showed you, usually `http://bambuddy.local:8000`. Give it thirty to sixty seconds.

=== ":material-ethernet: If you set up over ethernet"

    Press **Start Bambuddy**, wait thirty to sixty seconds, and open the address shown. Your network never changes, so nothing disconnects.

!!! tip "Mistyped the WiFi password?"
    The appliance notices it can't join, and brings the `Bambuddy-Setup` network back up. Rejoin it &mdash; the wizard is waiting exactly where you left it, and you can try again. It retries this indefinitely rather than stranding you.

---

## 5. Set up Bambuddy

The first time Bambuddy opens it asks how you want to sign in. Create an account with a username and password, or choose to run without authentication on a trusted home network. You can change your mind later.

!!! info "Two different doors"
    This account belongs to **Bambuddy**. It is separate from the appliance password you set in the wizard, which covers the admin panel, the console, and SSH.

---

## 6. Add your printer

Open the **Printers** page and press **Add Printer**. You need the four things from [Before you begin](#before-you-begin):

| Field | Where it comes from |
|---|---|
| **Name** | Whatever you like &mdash; `Workshop X1C` |
| **IP address** | The printer's screen, or your router |
| **Access code** | Eight characters, from Developer Mode |
| **Serial number** | The printer's screen |

Save. The indicator turns green once connected, and temperatures begin updating on their own. If it doesn't, it is almost always the access code &mdash; check it hasn't changed &mdash; or Developer Mode being off.

The full walkthrough is in [First Printer Setup](../getting-started/first-printer.md).

---

## Where everything lives

| Address | What |
|---|---|
| `http://bambuddy.local:8000` | Bambuddy &mdash; printers, queue, history |
| `http://bambuddy.local:8001` | Appliance admin panel (user `admin`) |
| `ssh pi@bambuddy.local` | Shell, for recovery |

Port `8000` is served by a small proxy sitting in front of Bambuddy. It passes the API, websockets, and camera streams straight through, so Home Assistant, your slicer, and the live view all behave exactly as they would against a plain Bambuddy install.

---

## 5. Add a printer

From here you're in ordinary Bambuddy. Follow [First Printer Setup](../getting-started/first-printer.md).

---

## Next

- [Admin Panel](admin-panel.md) &mdash; what the box itself can tell you
- [Updates &amp; Backups](updates.md) &mdash; before you need them, not after
- [Recovery](recovery.md) &mdash; the reset button, and when to reach for it
