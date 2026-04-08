---
title: Usage Guide
description: Day-to-day guide to using the SpoolBuddy touch interface
---

# :material-book-open: SpoolBuddy Usage Guide

This page covers day-to-day use of the SpoolBuddy interface — scanning spools, managing AMS slots, writing NFC tags, browsing inventory, calibrating the scale, and adjusting device settings.

The SpoolBuddy UI runs at `/spoolbuddy` on your Bambuddy backend:

```
http://<your-bambuddy-ip>:<port>/spoolbuddy
```

---

## :material-navigation: Navigation

The interface has a fixed top bar and bottom navigation visible on every screen.

### Top Bar

| Element | Position | Description |
|---------|----------|-------------|
| **SpoolBuddy logo** | Left | Branding |
| **Printer selector** | Center | Dropdown to pick the active printer. Auto-selects the first online printer. Shows "No printers online" when none are connected. |
| **Connection indicator** | Right | WiFi-style signal bars (white = connected, red = offline) and a green LED dot when the backend is reachable. |
| **Clock** | Far right | Live clock, updated every second. Respects your configured time format. |

### Bottom Navigation

Five tabs along the bottom of the screen:

| Tab | Icon | Route | Purpose |
|-----|------|-------|---------|
| **Dashboard** | Home | `/spoolbuddy` | Live spool scanning and device status |
| **AMS** | Grid | `/spoolbuddy/ams` | AMS slot overview and management |
| **Write Tag** | NFC | `/spoolbuddy/write-tag` | Write NFC tags to spools |
| **Inventory** | Box | `/spoolbuddy/inventory` | Browse and search your spool catalog |
| **Settings** | Gear | `/spoolbuddy/settings` | Device config, display, scale, updates, system info |

The active tab is highlighted in green.

---

## :material-home-analytics: Dashboard

The Dashboard is the main screen and where you spend most of your time. It has two columns: device status on the left and the current spool card on the right.

### Stats Bar

A compact row at the top showing:

- **Spools** — total spools in your inventory
- **Materials** — number of distinct material types
- **Brands** — number of distinct brands

### Device Panel (Left Column)

**Device Card** — shows live status of your SpoolBuddy hardware:

- Connection status (green dot = online, red = disconnected)
- Live scale weight in grams (or "—" when no reading)
- NFC status: "Tag detected" (green) or "No tag" (gray)

**Printer Badges** — one badge per configured printer showing its name and online/offline status (green/gray dot).

### Current Spool (Right Column)

This area changes based on what's happening:

#### Idle — Ready to Scan

When no tag is on the reader, you see an animated spool icon that cycles through colors with the message:

> **Ready to scan**
> Place a spool on the scale to identify it.
> NFC tag will be read automatically.

#### Known Spool Detected

When you place a tagged spool that matches your inventory, the **Spool Info Card** appears:

- **Spool color** and **fill percentage badge** (green >50%, yellow 20–50%, red <20%)
- **Color name**, **brand**, and **material** (with subtype if set)
- **Remaining filament** — large display showing remaining grams out of label weight
- **Fill bar** — color-coded horizontal bar
- **Details grid**:
    - Gross weight (from the scale)
    - Core weight (empty spool weight, default 250 g)
    - Spool size (label weight)
    - Scale weight — shows a green checkmark if the scale matches the expected weight (±50 g), or a yellow alert with a sync button if there is a mismatch
    - Tag ID (last 8 characters)

**Buttons:**

| Button | Action |
|--------|--------|
| **Sync Weight** | Saves the current scale reading to the spool record. Shows "Synced!" for 3 seconds on success. |
| **Assign to AMS** | Opens the AMS assignment modal (see below). |
| **Close** (X) | Dismisses the card. The same tag won't re-trigger until removed and placed again. |

#### Unknown Tag Detected

When you place a tag that doesn't match any spool in your inventory:

- **"New Tag Detected"** header
- Tag UID displayed in monospace
- Weight on the scale and estimated filament remaining

**Buttons:**

| Button | Action |
|--------|--------|
| **Link to Spool** | Opens a modal to link this tag to an existing untagged spool in your inventory. Only appears if you have untagged spools. |
| **Add to Inventory** | Creates a basic PLA spool entry with this tag. A hint recommends creating the spool in the Bambuddy web interface instead for a more complete record. |
| **Close** (X) | Dismisses the card. |

#### Device Offline

When SpoolBuddy is not connected to the backend:

> **Device Offline**
> Connect the SpoolBuddy display to scan spools.

### Link Tag to Spool Modal

Opened from the "Link to Spool" button on unknown tags:

1. The tag UID is shown at the top.
2. A scrollable list of all untagged spools appears, each showing its color, name, brand, material, and estimated remaining weight.
3. Tap a spool to select it (green highlight).
4. Press **Link Tag** to confirm, or **Cancel** to close.

### Assign to AMS Modal

Opened from the "Assign to AMS" button on known spools. This is a full-screen overlay showing all AMS units on the selected printer:

- **Regular AMS units** (4-slot) are shown in a grid with color previews, fill levels, temperature, and humidity readings.
- **Single-slot units** (AMS-HT, external) appear in a separate row below.
- Tap a slot to assign the spool.

**Material mismatch warning** — if the spool's material doesn't match what's configured on the slot, a confirmation dialog appears explaining the mismatch before proceeding.

Status messages appear at the top: blue while configuring, green on success ("Assigned!"), red on error.

---

## :material-tray-full: AMS Management

The AMS page shows all AMS units connected to the currently selected printer.

### States

| State | Display |
|-------|---------|
| No printer selected | "No printer selected — Select a printer from the top bar" |
| Printer disconnected | "Printer disconnected" |
| No AMS detected | "No AMS detected — Connect an AMS to see filament slots" |

### AMS Unit Cards

Each regular AMS unit (A, B, C, D) shows a card with 4 slots. Each slot displays:

- **Spool color** indicator
- **Active dot** (green) if that slot is currently printing
- **Slot name** (e.g., "Slot 1")
- **Material type** or "Empty"
- **Temperature** — color-coded thermometer icon (green/amber/red)
- **Humidity** — color-coded water drop icon (green/amber/red)
- **Fill level bar** — color-coded (green >15%, amber 10–15%, red <10%)
- **Nozzle badge** ("L" or "R") on dual-nozzle printers

Single-slot items (AMS-HT, External) appear in a horizontal row below the regular units.

### Slot Action Picker

Tap any slot to open the action picker. It shows the current slot contents and offers these actions:

| Action | Icon | Description |
|--------|------|-------------|
| **Configure** | Settings (blue) | Set the filament preset, K-profile, and color for this slot. |
| **Assign** / **Unassign** | Package (green) / Unlink (amber) | Track an inventory spool in this slot, or remove the current assignment. |
| **Link** / **Unlink** | Link (green) / Unlink (amber) | Link a Spoolman spool to this slot. Only appears when Spoolman integration is enabled. |

Each action opens its own configuration modal.

### Fill Level Priority

Fill levels are resolved in this order:

1. Spoolman fill level (if Spoolman integration is enabled)
2. Inventory assignment fill level (if a spool is assigned)
3. AMS remaining percentage (from the printer firmware)

---

## :material-nfc: Write Tag

The Write Tag page lets you write spool data to NFC tags. It has a two-column layout: spool selection on the left, NFC status panel on the right.

### Tabs

Three tabs across the top control which spools are shown:

| Tab | Shows | Use Case |
|-----|-------|----------|
| **Existing Spool** | Spools without a tag assigned | Writing a tag for a spool already in your inventory |
| **New Spool** | A form to create a new spool | Creating a brand-new spool and writing its tag in one step |
| **Replace Tag** | Spools that already have a tag | Re-writing or replacing a damaged/lost tag |

Switching tabs clears the current selection and any write status.

### Existing Spool / Replace Tag

- **Search bar** at the top filters by material, color, brand, and subtype.
- Each spool in the list shows its color dot, brand, material, color name, remaining weight, and (on Replace tab) the current tag UID.
- Tap a spool to select it (checkmark appears).

### New Spool

Two view modes toggled by **Simple** / **Full** buttons:

**Simple view:**

- Material dropdown (PLA, PETG, ABS, ASA, TPU, PA, PC, PVA, HIPS)
- Color name input
- Color picker
- Brand input
- Weight input
- **Create Spool** button

**Full view:**

- Two sub-tabs: **Filament** and **PA Profile**
- Filament tab: material, brand, color, spool weight, weight used, cost per kg, slicer preset, notes, and bulk quantity
- PA Profile tab: select calibration profiles per printer
- **Quick Add** toggle hides the PA Profile tab for faster creation

After creating the spool, a confirmation card appears: "Spool created! Ready to write."

### NFC Status Panel (Right Column)

The panel shows the current state of the tag writing process:

| State | Display | Actions |
|-------|---------|---------|
| **Device offline** | "SpoolBuddy is offline" | — |
| **No spool selected** | "Select a spool, then place a blank NTAG on the reader" | — |
| **Spool selected, no tag** | "Place an NTAG on the reader" (pulsing border) | — |
| **Spool selected, tag detected** | "Tag detected — ready to write" with tag UID | **Write Tag** (or **Replace Tag** on Replace tab) |
| **Writing** | "Writing tag..." with animated rings | **Cancel** |
| **Success** | Green checkmark with success message and spool info | Auto-clears after 5 seconds |
| **Error** | Red X with error message | **Try Again** |

On the **Replace Tag** tab, if the selected spool already has a tag, an additional **Untag Selected Spool** button appears to remove the existing tag without writing a new one. A yellow warning explains that the old tag will be unlinked.

---

## :material-package-variant: Inventory

The Inventory page lets you browse your full spool catalog.

!!! tip "Spoolman mode"
    If Spoolman integration is enabled, this page shows the Spoolman interface in an embedded frame instead of the native inventory view.

### Search and Filtering

- **Search bar** — real-time search across material, subtype, brand, color name, and notes.
- **Filter pills** — horizontal scrollable row:
    - **All Spools** — shows total count
    - **In AMS** — shows count of spools currently assigned to AMS slots
    - **Per-material pills** — one pill per unique material type (auto-generated)

Tap a pill to toggle that filter. Active pills are highlighted.

### Spool Grid

Spools are displayed in an auto-fill card grid. Each card shows:

- Circular spool icon with the spool's color
- Material name (with subtype if set)
- Color dot and color name
- Fill bar (color-coded green/yellow/red)
- Remaining weight in grams and percentage
- **AMS badge** if assigned (e.g., "AMS A1", "AMS HT A")

### Spool Detail Modal

Tap any spool card to open a detail modal (bottom-sheet style) showing:

- Large spool visual with color
- Material, brand, and color name in the header
- **Remaining weight** with a fill bar
- **AMS location** (if assigned)
- **Detail grid** (2 columns):
    - Label weight
    - Used weight
    - Core weight
    - Gross weight
    - Nozzle temperature range
    - Cost per kg
    - Last scale weight
    - Tag ID (monospace)
    - Slicer filament preset
    - PA K-Profiles (with K-values, if any)
    - Notes (if present)

---

## :material-cog: Settings

The Settings page has five tabs across the top.

### Device Tab

**NFC Reader** card (left column):

- Reader type (e.g., "PN532")
- Connection method (e.g., "Serial /dev/ttyUSB0")
- Status indicator: green "Ready", red "Error", or gray "N/A"

**Device Info** card (right column):

- Hostname
- IP address
- Uptime (formatted as "Xh Ym")
- Device ID (monospace, truncated)

**Backend & Auth Config** card:

- Backend URL input field
- API Token input field (password masked)
- **Save** button — saves configuration to the device
- Status message (green on success, red on error)

**Diagnostics** — three buttons:

| Button | Color | What It Does |
|--------|-------|--------------|
| **NFC Diagnostic** | Blue | Tests NFC reader connectivity and reports results |
| **Scale Diagnostic** | Yellow | Tests scale/ADC hardware and reports results |
| **Read Tag** | Green | Reads raw tag data and displays contents |

Each opens a modal showing live terminal-style output, an exit code, and a **Run Again** button.

### Display Tab

**Brightness** card:

- Slider from 0–100%
- Live percentage display
- Changes are applied immediately (software brightness filter)
- Shows "No DSI backlight detected" if no hardware backlight is available

**Screen Blank Timeout** card:

- Description: "Screen turns off after inactivity. Touch to wake."
- Six preset buttons: **Off**, **1m**, **2m**, **5m**, **10m**, **30m**
- Active selection highlighted in green
- Touch anywhere on the screen resets the blank timer

### Scale Tab

**Idle view:**

- Live weight display with stability indicator (green dot = stable, amber = fluctuating)
- Current tare offset value
- Current calibration factor
- Last calibrated timestamp
- Two buttons: **Tare** and **Calibrate**

**Tare** — quick zero adjustment. Press when the scale is empty to reset the zero point.

**Calibrate** — opens a two-step calibration wizard:

**Step 1 — Set Zero:**

1. Remove all items from the scale.
2. Press **Set Zero** to record the empty reference point.

**Step 2 — Known Weight:**

1. Place an object of known weight on the scale.
2. Enter the known weight using the on-screen numpad (0–9, decimal point, backspace).
3. Press **Calibrate** to apply.

A live weight bar shows the current reading throughout the process. Press **Cancel** at any point to abort.

### Updates Tab

**Version & Status** card:

- Current version number (or "Waiting for daemon...")
- Status line showing check results
- Action buttons:

| Button | When Shown | Action |
|--------|-----------|--------|
| **Check for Updates** | No update available | Queries the daemon for new versions |
| **Force Update** | No update available | Forces a re-install of the current version |
| **Apply Update** | Update available | Downloads and installs the update. Disabled if device is offline. |

**SSH Setup** card (collapsible):

- Expand to reveal the device's SSH public key in a pre-formatted box.
- **Copy** button copies the key to clipboard (shows "Copied!" briefly).
- Use this key to authorize the SpoolBuddy device for passwordless access if needed.

### System Tab

Read-only system information displayed when stats are available:

| Card | Fields |
|------|--------|
| **CPU** | Cores, load average (1m/5m/15m), temperature (color-coded: green <65°C, amber 65–80°C, red ≥80°C) |
| **Memory** | Usage bar with percentage, used/total in MB, free in MB |
| **Disk** | Usage bar with percentage, used/total in GB |
| **OS** | Distribution name, kernel version, architecture |
| **Runtime** | Python version, system uptime |

Shows "Waiting for system stats..." while data is loading.

---

## :material-gesture-swipe-horizontal: Gestures and Shortcuts

| Gesture | Action |
|---------|--------|
| **Swipe left/right** | Switch between online printers (anywhere in the UI) |
| **Swipe down from top** | Open the Quick Menu (power controls and system commands) |
| **Touch anywhere** | Reset the screen blank timer |
| **Escape key** | Close open modals |

---

## :material-menu: Quick Menu

Swipe down from the top edge of the screen to open the Quick Menu — a slide-down overlay panel for printer power control and system management.

### Printer Power

If any of your printers have smart plugs configured (Tasmota, Home Assistant, or REST), they appear in the **Printer Power** section:

- Each printer is shown with its name, plug name, and current state (**ON** / **OFF**).
- Tap a printer row to toggle its smart plug on or off.
- The plug state updates in real-time after toggling.

!!! note
    Only printers with an enabled, non-MQTT smart plug appear here. MQTT plugs are monitor-only and cannot be toggled. Configure smart plugs in **Settings > Smart Plugs** on the main Bambuddy web interface.

### System Controls

Four system control buttons are available:

| Button | Action | Confirmation |
|--------|--------|--------------|
| **Restart Daemon** | Restarts the SpoolBuddy daemon service. NFC and scale will be temporarily unavailable while the daemon reinitializes. | Yes |
| **Restart Browser** | Restarts the kiosk browser (labwc + Chromium). The display goes blank briefly during the restart. | Yes |
| **Reboot** | Reboots the Raspberry Pi. The device will be offline for 30–60 seconds. | Yes |
| **Shutdown** | Shuts down the Raspberry Pi. You will need physical access to power it back on. | Yes |

All four actions require a confirmation dialog before executing. The **Shutdown** and **Reboot** buttons are highlighted in red and amber respectively to indicate their destructive nature.

!!! warning
    System controls are disabled when the SpoolBuddy device is offline. The commands are queued via the backend and executed on the next daemon heartbeat (within ~10 seconds).

To close the Quick Menu, tap the backdrop area behind the panel.

---

## :material-link: Related Pages

- [Wiring Diagrams](wiring-diagrams.md) — Electrical connections for all hardware
- [Installation](installation.md) — Software setup and first boot
- [Configuration](configuration.md) — Environment variables and `.env` settings
- [Assembly](assembly.md) — Physical case assembly guide
