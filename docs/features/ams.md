---
title: AMS, Humidity & Drying
description: Monitor AMS and AMS-HT filament systems, manual and automatic remote drying, ambient drying, configurable presets
---

# AMS & Humidity Monitoring

Bambuddy provides comprehensive monitoring for your AMS (Automatic Material System) and AMS-HT (High Temperature) units.

---

## :material-tray-full: AMS Slot Status

Each AMS slot displays:

| Information | Description |
|-------------|-------------|
| **Filament color** | Visual color swatch |
| **Material type** | PLA, PETG, ABS, ASA, etc. |
| **Remaining** | Estimated filament left |
| **Active** | Currently feeding indicator |

### RFID Re-read

Refresh filament information for individual AMS slots:

1. Hover over an AMS slot on the printer card
2. Click the menu button (:material-dots-vertical:) that appears
3. Select **Re-read RFID**
4. A loading indicator appears while the printer reads the RFID tag
5. Filament information updates automatically when complete

!!! note "Availability"
    The re-read menu is hidden when the printer is busy (printing). Wait until the printer is idle to re-read RFID data.

!!! tip "When to Re-read"
    Use this feature when you've swapped a spool but the AMS hasn't automatically detected the change, or if filament information seems incorrect.

### Load / Unload Filament

Drive Load and Unload directly from any AMS slot or external spool — no need to walk to the touchscreen:

1. Hover over an AMS slot or external spool on the printer card
2. Click the menu button (:material-dots-vertical:) that appears
3. Select **Load** to feed that tray, or **Unload** to retract whatever is currently loaded

!!! note "Availability"
    The Load / Unload menu is hidden while the printer is `RUNNING`. Wait until the printer is idle.

!!! info "Dual-extruder H2D"
    On the H2D with two external spools, **Ext-L** (left) and **Ext-R** (right) each drive their own nozzle. Loading from Ext-R uses the right nozzle's actual current temperature for the load command (matching BambuStudio's behaviour); a sensible 215 °C fallback is used if the right nozzle reports as cold or unknown.

!!! warning "Permission"
    Load / Unload requires the `printers:control` permission — the same scope used to start, stop, pause, and resume prints.

### Configure AMS Slot

Manually configure AMS slots for third-party or generic filaments. This tells the printer which filament profile to use for a specific slot (temperatures, flow rate, pressure advance).

1. Hover over an AMS slot on the printer card
2. Click the menu button (:material-dots-vertical:) that appears
3. Select **Configure Slot**
4. Choose a filament preset (from Bambu Cloud, local OrcaSlicer imports, or the built-in Bambu filament catalog — see [preset sources](inventory.md#where-presets-come-from))
5. Select a matching K profile (pressure advance calibration)
6. Optionally set a custom color using the color picker
7. Click **Configure Slot** to apply

**Color Picker Features:**

- 8 basic colors shown by default (White, Black, Red, Blue, Green, Yellow, Orange, Gray)
- Click **+** to expand 24 additional colors
- Enter custom hex codes or color names (e.g., "brown", "FF8800")
- Live preview of selected color

!!! tip "User Presets"
    User presets that inherit from Bambu presets (e.g., "# Overture Matte PLA @BBL H2D") are fully supported. Bambuddy automatically derives the correct filament ID from the preset's base configuration.

#### Printer Model Filtering

Filament presets in the Configure AMS Slot modal are automatically filtered by the printer model. This means you only see presets that are compatible with the printer you are configuring:

- **Cloud presets** are matched by the model suffix in the preset name (e.g., `@BBL X1C`, `@BBL P1S`, `@BBL H2D`). Generic presets without a model suffix are always included.
- **Local presets** are filtered by their `compatible_printers` field to show only those that list the current printer model.
- The **currently-configured preset** for the slot is always shown in the list, regardless of model filter, so you can see what is already set even if the preset would otherwise be filtered out.

This filtering reduces clutter and prevents accidentally selecting a preset intended for a different printer.

The picker also filters by the **nozzle diameter actually installed** on the printer, not a fixed 0.4 mm. If you have a 0.6 mm nozzle fitted, you'll see the 0.6 mm profiles for your trays; on a dual-nozzle H2D each AMS shows the profiles for the nozzle that feeds it. Configuring a slot with a profile that doesn't match the installed nozzle is what causes the printer to reject a print with *"Failed to get AMS mapping table"* — see [Troubleshooting](../reference/troubleshooting.md#nozzle-size-mismatch).

#### Pre-Population for Configured Slots

When you open the Configure AMS Slot modal for a slot that already has a configuration, Bambuddy pre-populates the form fields so you can review or adjust the current settings without starting from scratch:

- **Filament preset** — The previously-configured preset is pre-selected. Bambuddy resolves this from the saved preset mapping or by matching the slot's `tray_info_idx` to the corresponding preset.
- **Color** — The color picker is pre-populated with the slot's current filament color.
- **K-profile** — The active pressure advance profile is pre-selected by matching the slot's `cali_idx` to the available K-profile entries.
- **Auto-scroll** — The preset list automatically scrolls to the selected item so it is visible without manual scrolling.

!!! tip "Quick Adjustments"
    Pre-population makes it easy to tweak a single setting (for example, updating just the color or K-profile) without re-selecting every field from scratch.

### Multi-AMS Support

Bambuddy supports multiple AMS units per printer:

- Up to 4 AMS units (16 total slots)
- Each unit displayed with its slots
- Visual indication of active slot during printing

#### External Spool Holders

Printers without an AMS (or with filament loaded outside the AMS) use an external spool holder. The H2D is a dual-nozzle printer with two external spool positions — **Ext-L** (left) and **Ext-R** (right) — each feeding its respective nozzle. Bambuddy displays both external slots and supports configuring, color-setting, and K-profile assignment for each one independently.

---

## :material-autorenew: AMS Filament Backup

Bambuddy reads and controls the printer's per-printer **AMS Filament Backup** setting (BambuStudio's "Auto switch filament" / `auto_switch_filament`). When enabled, the printer automatically switches to a matching spool in another slot once one runs out, so a long print can roll across multiple spools of the same filament profile and colour.

### Status badge

The Filaments section header on each printer card carries a small badge:

| Badge | State | Meaning |
|:-----:|-------|---------|
| :material-autorenew:{ style="color: #2196f3" } | **ON** (blue circular arrow) | Auto-switch is enabled. The printer will roll over to a matching peer spool when one empties. |
| :material-autorenew:{ style="color: #6b7280" } | **OFF** (dim) | Auto-switch is disabled. The print pauses when a spool runs out. |
| **?** | Unknown | The printer didn't advertise the state (A1 / A1 Mini family — they emit no `cfg` field in push_status). Click disabled. |

Click the badge to open the **AMS Filament Backup modal**.

### Backup modal

A BambuStudio Auto Refill-style view of every backup pair currently formed on the printer.

- **Toggle row** at the top with the current ON / OFF / unsupported state and the same setting wired to the firmware via MQTT.
- **One ring per backup pair**, modelled on BambuStudio's "Auto Refill" widget. The ring is filled with the actual filament colour; the material name and rotation count (`N× ↻`) sit in the centre; member slot labels (e.g. `A·1`, `B·3`) sit on contrast-aware pills distributed around the colour band. Lone slots (no peer) are deliberately not listed — if a slot doesn't appear in any ring, it has no backup.
- **R / L badges** on each ring only render on dual-extruder printers (H2D / H2C / X2D) when the extruder map carries two distinct values; single-nozzle printers or printers with routing data not yet reported don't see the badges.
- **Esc**, click-outside, or the close button dismisses the modal.

### Pairing rule

Two slots pair when they share **the same Bambu filament preset ID** (`tray_info_idx`, e.g. `GFA00`) AND **the same colour** (alpha-normalised — `1A1A1AFF` matches `1A1A1A`). This mirrors the firmware's own switch decision.

- Three PETG HF spools in different colours **don't** back each other up. The firmware would correctly swap PETG HF but the print would change colour mid-run.
- User-tagged spools without a Bambu preset **never** pair with anything else. The firmware backup logic relies on the preset, and pairing on cosmetic name match alone risks treating two visually-identical but materially-different spools as backups.
- On dual-extruder printers, pairing is scoped per extruder side. Two same-`(preset, colour)` slots on opposite extruders aren't peers — the firmware can't cross.

!!! info "Why the badge sits in the section header, not on each AMS"
    The setting is **per-printer**, not per-AMS. Bambuddy decodes it from bit 18 of the printer's top-level `print.cfg` hex string, which is a single value covering every AMS attached to that printer.

### Backup-aware "insufficient filament" check

The pre-dispatch deficit check that gates "Print Now" / "Send" buttons (and the queue dispatcher) is **backup-aware**.

When AMS Filament Backup is **ON**, the deficit check pools remaining grams across same-`(preset, colour)` spools on the printer (per extruder side on dual-nozzle) before declaring a shortfall. So a 200 g print routed to a slot with 10 g remaining no longer gets blocked when a peer slot holds the same filament with 500 g spare — the firmware will swap automatically.

When AMS Filament Backup is **OFF**, the check falls back to strict per-slot accounting — there's no peer to roll to, so an empty slot would actually stop the print.

This works both in internal-inventory mode (via the local Spool table) and Spoolman mode (via Spoolman's `filament.id` + `color_hex`).

### Interaction with "Prefer Lowest Remaining Filament"

The **Prefer Lowest Remaining Filament** setting (Settings → Filament) tells the AMS auto-mapper to pick the spool with the least remaining weight first — useful for finishing off near-empty spools so they don't accumulate. This only makes sense when **AMS Filament Backup** is **ON**, because without backup the printer will stop on the first empty spool, defeating the purpose.

When AMS Filament Backup is **OFF**, Bambuddy automatically suppresses the prefer-lowest sort (both backend dispatcher and the PrintModal mapping UI honour this) so it won't reach for a near-empty spool the printer can't roll off of. The setting itself isn't hidden — flip Backup back on and it takes effect again immediately.

With Backup **ON**, prefer-lowest picks the lowest-remaining slot **within the same `(preset, colour)` group** — so when the picked slot empties, the firmware swaps to the same-colour peer that holds more material. If only one same-`(preset, colour)` slot exists, prefer-lowest can't pick anything riskier than that one, and the backup-aware deficit check still catches the case where it's short.

---

## :material-water-percent: Humidity Monitoring

Track humidity levels inside your AMS units:

### Current Reading

The humidity percentage is displayed on the printer card:

| Level | Status | Action |
|:-----:|--------|--------|
| < 20% | :material-check-circle:{ style="color: #4caf50" } Excellent | None needed |
| 20-40% | :material-check-circle:{ style="color: #8bc34a" } Good | None needed |
| 40-60% | :material-alert:{ style="color: #ff9800" } Fair | Consider drying |
| > 60% | :material-alert-circle:{ style="color: #f44336" } High | Replace desiccant |

### Configurable Thresholds

Set custom warning thresholds in Settings:

1. Go to **Settings** > **General**
2. Find **AMS Humidity Threshold**
3. Set your preferred warning level
4. Save changes

!!! tip "Filament Sensitivity"
    Different filaments have different humidity sensitivities. PLA is more tolerant than Nylon or PETG. If you run multi-material AMS units (e.g. one dedicated to PLA, another to Nylon), set a **[per-filament threshold](#per-filament-humidity-threshold)** below instead of one global value — auto-drying and the humidity alarm will trigger at the right level for each material.

---

## :material-thermometer: Temperature Monitoring

For AMS-HT (High Temperature) units, temperature is also tracked:

| Reading | Description |
|---------|-------------|
| **Current temp** | Live temperature inside the unit |
| **Target temp** | Configured drying temperature |
| **Status** | Heating, holding, or idle |

---

## :material-fire: Remote AMS Drying

Control AMS drying directly from Bambuddy — no need to use the printer's touchscreen. Start, monitor, and stop drying sessions for AMS 2 Pro and AMS-HT units.

!!! warning "P1P / P1S: drying is screen-only"
    The P1 series does not accept drying commands over the network. Bambu's own [P1 manual](https://wiki.bambulab.com/en/p1/manual/screen-operation) says it plainly: *"P1S connected AMS drying functions may only be controlled from the P1S screen."* The firmware acknowledges the command and then ignores it, so no app — Bambuddy, Studio, or Handy — can start or stop a cycle on a P1.

    Bambuddy still shows the drying button on a P1, greyed out with a tooltip saying why, and still displays a cycle you started at the printer, including its countdown. Auto-drying (queue and ambient) skips P1 printers.

### Supported Hardware

Remote drying requires an AMS unit with built-in heating:

| AMS Type | Module Type | Max Temp | Supported |
|----------|:-----------:|:--------:|:---------:|
| AMS 2 Pro | n3f | 65°C | :material-check: |
| AMS-HT | n3s | 85°C | :material-check: |
| AMS (original) | ams | — | :material-close: |

### Printer Firmware Requirements

Not all printers support remote drying commands. The following minimum firmware versions are required:

| Printer Model | Min Firmware | Notes |
|---------------|:------------:|-------|
| X1 / X1C | 01.09.00.00 | |
| P2S | 01.02.00.00 | |
| H2D | 01.02.30.00 | |
| H2C | 01.02.00.00 | |
| H2S | 01.02.00.00 | |
| H2D Pro | Any | No version gate |
| X1E | Any | No version gate |
| P1P / P1S | — | :material-close: Screen-only — see the warning above |
| A1, A1 Mini | — | :material-close: No drying-capable AMS |

!!! note "Unknown Models"
    For printers not listed above (future models), Bambuddy allows the drying command. If the printer's firmware doesn't support it, the command fails gracefully with no side effects.

### Power Supply Requirements

AMS 2 Pro and AMS-HT units require an external power supply (PSU) to run the drying heater. Without it, the AMS can only monitor humidity — it cannot actively dry.

The printer firmware reports power constraints via the `dry_sf_reason` field per AMS unit. Bambuddy reads these automatically:

| Reason | Code | Description |
|--------|:----:|-------------|
| Insufficient Power | 1 | Too many AMS units drying simultaneously — disconnect other units or connect a PSU |
| Need Plugin Power | 8 | No external PSU connected — plug in the AMS power adapter |
| Task Occupied | 0 | Printer is busy with another operation |
| AMS Busy | 2 | AMS is performing another operation |
| Consumable at Outlet | 3 | Filament detected at the AMS outlet |
| Initiating | 4 | Drying is already starting up |
| Not Supported in 2D Mode | 5 | Cannot dry in current mode |
| Already Drying | 6 | Drying session already active |
| Upgrading | 7 | Firmware update in progress |

When a power constraint is detected, the :material-fire: drying button is **disabled** and shows a "Power required" tooltip. This applies to manual drying, queue auto-drying, and ambient drying — the scheduler skips AMS units with active `dry_sf_reason` entries.

!!! warning "PSU Not Connected"
    If you see the drying button greyed out with a "Power required" tooltip, connect the external power adapter to your AMS unit. This is the most common reason drying cannot start.

### HMS Error Codes (AMS Power)

When the AMS encounters a power-related issue, the printer reports it as an HMS (Health Management System) error. These appear in Bambuddy's HMS error panel:

**AMS 2 Pro Errors:**

| HMS Code | Description |
|----------|-------------|
| `07XX_9200_0002_0003` | Heater fan 1 can't start — PSU not connected |
| `07XX_9300_0002_0003` | Heater fan 2 can't start — PSU not connected |
| `07XX_9800_0002_0001` | PSU voltage too low |
| `07XX_9800_0002_0002` | PSU voltage too high |

**AMS-HT Errors:**

| HMS Code | Description |
|----------|-------------|
| `18XX_2500_0002_0001` | Using printer power instead of dedicated adapter — connect the AMS-HT power adapter |
| `18XX_9200_0002_0003` | Heater fan 1 can't start — PSU not connected |
| `18XX_9300_0002_0003` | Heater fan 2 can't start — PSU not connected |
| `18XX_9800_0002_0001` | PSU voltage too low |
| `18XX_9800_0002_0002` | PSU voltage too high |

!!! note "HMS Code Format"
    `XX` represents the AMS unit index (`00`–`07` for units A–H). For example, `0700_9200_0002_0003` is unit A, `0701_9200_0002_0003` is unit B.

### Starting a Drying Session

1. Find the AMS 2 Pro or AMS-HT card on the Printers page
2. Click the :material-fire: flame icon in the AMS card header
3. In the drying popover:
      - **Select filament type** — Choose from PLA, PETG, TPU, ABS, ASA, PA, PC, or PVA
      - **Temperature** — Auto-set from BambuStudio official presets; adjust manually with the slider or input field
      - **Duration** — Auto-set from presets (1–24 hours); adjust as needed
      - **Rotate spool** — Optionally enable spool rotation during drying for more even heat distribution. Off by default. The toggle is automatically **disabled** (greyed out, with a tooltip) when any tray in the targeted AMS has filament threaded into the feed tube — the whole AMS rotates as one mechanism, so a single loaded slot mechanically locks the entire unit. The submission also clamps the value off in that state to prevent a stale toggle from leaking through
4. Click **Start**

!!! tip "Filament Presets"
    Temperature and duration defaults come from BambuStudio's official filament profiles. You can customize them in **Settings** > **AMS Display Thresholds** > **Drying Presets**. These presets are shared between manual drying, queue auto-drying, and ambient drying.

### Monitoring Drying Progress

When drying is active, a status bar appears between the AMS header and slot grid:

- **Time remaining** — Countdown in hours and minutes (e.g., "3h 42m" or "42m")
- The flame icon in the header turns amber to indicate active drying

### Stopping a Drying Session

- Click the **×** button on the drying status bar, or
- Click the flame icon (when drying is active, it acts as a stop button)

### Permissions

| Action | Required Permission |
|--------|:------------------:|
| View drying status | No permission needed |
| Start / Stop drying | `printers:control` |

!!! warning "Drying During Prints"
    The AMS can dry filament while the printer is idle or printing. However, drying during a print may affect the AMS temperature readings and humidity levels.

---

## :material-fire-circle: Queue Auto-Drying

Automatically dry AMS filament between scheduled prints. When a printer is idle and humidity exceeds the configured threshold, Bambuddy starts a drying session to keep filament in optimal condition before the next print begins.

### How It Works

1. The scheduler checks idle printers that have **scheduled** queue items
2. For each AMS unit, it reads the current humidity level
3. If humidity exceeds the **Fair (orange)** threshold from Settings, drying is triggered
4. The drying temperature and duration are determined by the loaded filament types using the configured [drying presets](#configurable-drying-presets)
5. Drying runs for the preset **duration** — the printer stops it automatically when the cycle completes
6. When the next scheduled print is ready to start, any remaining drying is stopped and the print begins

### Conservative Temperature Selection

When an AMS unit contains **mixed filament types** (e.g., PLA and PETG in the same unit), the scheduler uses conservative parameters:

- **Temperature** — The **lowest** temperature across all loaded filaments (to avoid overheating sensitive materials)
- **Duration** — The **longest** duration across all loaded filaments

### Per-Filament Humidity Threshold

A single global humidity threshold is a poor fit for multi-material print farms — Nylon wants to stay under 20%, PLA is happy at 60%, ASA somewhere in between. Bambuddy lets you set a different **trigger threshold per filament type**, in addition to the conservative drying temp/duration above.

Configure overrides in **Settings** > **Workflow** > **Auto-Drying** in the table directly below the **Drying Presets** table:

| Filament | Threshold |
|----------|-----------|
| Default (unknown types) | 60 % |
| PLA | 60 % |
| PETG | 50 % |
| Nylon (PA) | 20 % |
| ASA | 30 % |
| ... | ... |

- **Empty / unset** — the editor pre-fills from your global **AMS Humidity Threshold (Fair)** value, so the upgrade is silent. You only need to touch the rows for materials you want to treat differently.
- **Range** — 5 % to 95 %.
- **Clearing a row** (delete the value, click away) resets that row to use the **Default** row.

#### Mixed-AMS Resolution: Most-Restrictive Wins

When one AMS unit holds different filament types, Bambuddy picks the **lowest** threshold across the loaded spools. A unit with PLA at 60 % and Nylon at 20 % triggers drying when humidity exceeds 20 % — the Nylon's level — protecting the most sensitive material in the unit.

This matches the conservative-params strategy used for drying temperature and duration above. If you want a dedicated humidity profile per material, dedicate one AMS unit per filament type.

#### What the Override Drives

The per-filament threshold is read by **both**:

1. The **auto-drying scheduler** — decides when to start, stop, and skip drying per AMS.
2. The hourly **humidity alarm** notifier — fires `on_ams_humidity_high` / `on_ams_ht_humidity_high` (see [Notifications](notifications.md)).

Both consumers go through the same resolver, so they can never disagree on whether a given AMS is "too humid."

!!! note "What this does NOT change"
    - The **AMS humidity badge color** on the printer card stays based on the single global **Fair** threshold from Settings > General. Badge color is per-AMS-unit, and there's no clean way to color a mixed AMS based on its most sensitive material — the override applies only to the scheduler and the alarm, which act on per-AMS firmware commands.
    - The **drying temperature and duration** for each filament are still set in **Drying Presets** above (separate setting). Humidity = *when* to dry. Temp + duration = *how* to dry.

### Enabling Auto-Drying

1. Go to **Settings** > **AMS Display Thresholds**
2. Set the **Fair (orange) ≤** humidity threshold — this is the trigger point for auto-drying
3. Scroll to **Queue Auto-Drying**
4. Enable **Enable auto-drying**
5. Optionally enable **Wait for drying to complete** (blocking mode)

### Blocking vs Non-Blocking Mode

| Mode | Behavior |
|------|----------|
| **Non-blocking** (default) | Prints take priority. Drying stops when a print is ready to start. |
| **Blocking** | Queue waits until the drying session finishes before starting the next print. |

!!! tip "Which mode should I use?"
    **Non-blocking** is recommended for most users — it ensures prints start on time while filling idle gaps with drying. Use **blocking** only if filament dryness is critical for print quality (e.g., Nylon, PC) and you don't mind delayed print starts.

### When Auto-Drying Stops

Auto-drying is stopped automatically when:

- **The drying cycle completes** — drying runs for the preset duration and the printer stops it when the cycle finishes
- A scheduled print is ready to start (non-blocking mode)
- The queue item's schedule is removed or changed to "Queue Only"
- All scheduled items are removed from the queue
- Auto-drying is disabled in Settings

!!! note "The threshold only starts drying"
    The **Fair (orange)** humidity threshold controls when drying **starts** — a cycle begins when humidity exceeds it. Once a cycle is running, Bambuddy lets it run for its full preset duration rather than stopping on a mid-cycle humidity reading. Relative humidity drops sharply in the heated air of an active dryer, so a mid-cycle reading isn't a reliable measure of how dry the filament actually is.

### Requirements

- At least one **scheduled** queue item (items in "Queue Only" mode do not trigger auto-drying)
- AMS 2 Pro or AMS-HT unit (original AMS does not support drying)
- Supported printer firmware (see [firmware requirements](#printer-firmware-requirements) above)
- Humidity above the Fair threshold

!!! note "No Scheduled Prints?"
    If you want drying to run on idle printers regardless of whether prints are scheduled, see [Ambient Drying](#ambient-drying) below.

---

## :material-fire-alert: Ambient Drying

Automatically dry filament on any idle printer whenever AMS humidity exceeds the configured threshold — no scheduled prints required. While [queue auto-drying](#queue-auto-drying) only activates when prints are scheduled, ambient drying keeps filament dry on all idle printers at all times.

### How It Works

1. The scheduler continuously monitors all idle printers
2. For each AMS unit, it reads the current humidity level
3. If humidity exceeds the **Fair (orange)** threshold from Settings, drying is triggered
4. The drying temperature and duration are determined by the loaded filament types using the configured [drying presets](#configurable-drying-presets)
5. Drying runs for the preset **duration** — the printer stops it automatically when the cycle completes

Unlike queue auto-drying, ambient drying does not require any scheduled queue items. It runs whenever a printer is idle and its AMS humidity is above the threshold.

### Enabling Ambient Drying

1. Go to **Settings** > **Print Queue**
2. Find **Ambient Drying**
3. Enable **Enable ambient drying**

### Using Both Modes Together

Ambient drying and queue auto-drying can be enabled simultaneously. They complement each other:

| Mode | Triggers when | Stops when |
|------|---------------|------------|
| **Queue auto-drying** | Printer is idle with scheduled prints pending | Drying cycle completes, or next print is ready to start |
| **Ambient drying** | Printer is idle (no scheduled prints required) | Drying cycle completes |

When both are enabled and a printer has scheduled prints, queue auto-drying takes precedence (since it is aware of print scheduling and blocking/non-blocking behavior). On printers with no scheduled prints, ambient drying takes over.

### Requirements

- AMS 2 Pro or AMS-HT unit (original AMS does not support drying)
- Supported printer firmware (see [firmware requirements](#printer-firmware-requirements) above)
- Humidity above the Fair threshold
- No active power constraints on the AMS unit (see [power supply requirements](#power-supply-requirements))

!!! tip "Print Farm Use Case"
    Ambient drying is particularly useful for print farms where printers may sit idle for extended periods. Rather than letting humidity build up, Bambuddy keeps filament dry on every idle printer automatically.

---

## :material-fire-truck: Continue Drying While Printing

Bambu shipped an "AMS Print While Drying" firmware feature on selected printers that lets the AMS keep running its drying cycle **concurrently** with an active print. With this feature enabled in Bambuddy, the existing auto-drying scheduler can also evaluate printers that are mid-print — drying does not stop the instant a print starts.

**Off by default.** Opt-in toggle in **Settings** > **Workflow** > **Continue drying while printing**.

### Firmware Requirements

These minimum firmware versions are verified per Bambu wiki release notes (release-note phrasing: "printing while filament is drying" / "Print While Drying"):

| Printer Model | Min Firmware |
|---------------|:------------:|
| H2D | 01.03.00.00 |
| H2D Pro | 01.02.00.00 |
| H2C | 01.02.00.00 |
| H2S | 01.02.00.00 |
| X2D | 01.01.00.00 |
| X1C | 01.11.02.00 |
| P2S | 01.02.00.00 |
| A2L | 01.01.00.00 |

**Not supported** (intentionally excluded — wiki release notes never mention "Print While Drying" for these models):

- P1P / P1S
- A1, A1 Mini
- X1 (non-C), X1E

On an unsupported printer the toggle has no effect — the firmware reports `dry_sf_reason=[0]` (`TaskOccupied`) any time a print is running, so any attempt to start drying mid-print is rejected at the printer.

### How It Works

1. The auto-drying scheduler evaluates **all** active printers, not just idle ones
2. For each printer, it checks model and firmware against the matrix above
3. If the printer is mid-print **and** supports concurrent drying **and** AMS humidity exceeds the threshold, drying starts (or continues) using the conservative per-AMS preset
4. The mid-print drying temperature is automatically **capped at `max(40, preset_temp - 5)`** &mdash; Bambu's own release notes warn "lower drying temperature during printing" and "drying temperature must not exceed the filament's softening temperature". The 5 &deg;C offset protects spools inside the hot enclosure during an active print
5. Drying runs for the preset duration, just like the idle path — the printer stops it when the cycle completes
6. The firmware's per-AMS `dry_sf_reason` field continues to arbitrate — if hardware rejects the command for any reason (busy AMS, power constraint, filament at outlet), the scheduler honours that decision per AMS

### Enabling

1. Go to **Settings** > **Print Queue**
2. Find **Continue drying while printing**
3. Enable the toggle

The toggle is independent of [queue auto-drying](#queue-auto-drying) and [ambient drying](#ambient-drying) — you can mix and match. With all three enabled, drying runs in idle gaps **and** during prints on capable hardware, each cycle running for its configured preset duration.

### Safety

- **Temperature cap** — `max(40, preset_temp - 5)` &deg;C, applied automatically. The user-configured idle preset is not touched; the cap only applies to the mid-print code path
- **Firmware is the ultimate arbiter** — Bambuddy never sends a drying command that contradicts the printer's reported state. The matrix is a UI affordance; if Bambu silently ships the feature to a new model, Bambuddy will surface the toggle there too on the next release
- **Default off** — existing installs see no behaviour change until the toggle is enabled

### Requirements

- Supported printer firmware (matrix above)
- AMS 2 Pro or AMS-HT unit (original AMS does not support drying)
- Humidity above the configured threshold
- No active blocking constraints on the AMS unit (see [power supply requirements](#power-supply-requirements))

---

## :material-tune: Configurable Drying Presets

Customize the drying temperature and duration for each filament type. These presets are used by both [manual drying](#starting-a-drying-session), [queue auto-drying](#queue-auto-drying), and [ambient drying](#ambient-drying).

### Default Presets

Defaults are based on BambuStudio's official filament drying profiles:

| Filament | AMS 2 Pro Temp | AMS-HT Temp | AMS 2 Pro Duration | AMS-HT Duration |
|----------|:--------------:|:-----------:|:-------------------:|:---------------:|
| PLA | 45°C | 45°C | 12h | 12h |
| PETG | 65°C | 65°C | 12h | 12h |
| TPU | 65°C | 75°C | 12h | 18h |
| ABS | 65°C | 80°C | 12h | 8h |
| ASA | 65°C | 80°C | 12h | 8h |
| PA | 65°C | 85°C | 12h | 12h |
| PC | 65°C | 80°C | 12h | 8h |
| PVA | 65°C | 85°C | 12h | 18h |

### Editing Presets

1. Go to **Settings** > **AMS Display Thresholds**
2. Find the **Drying Presets** table
3. Edit temperature (°C) and duration (hours) for each filament type
4. Changes auto-save

!!! note "AMS 2 Pro Temperature Limit"
    AMS 2 Pro (n3f) has a hardware maximum of 65°C. AMS-HT (n3s) supports up to 85°C.

---

## :material-power-plug-off: Auto Off After Drying (Smart Plug)

If the printer (or its AMS) is on a [smart plug](smart-plugs.md), Bambuddy can cut power automatically once a drying cycle completes. The trigger works for all three drying paths — [manual drying](#starting-a-drying-session), [queue auto-drying](#queue-auto-drying), and [ambient drying](#ambient-drying) — because Bambuddy detects completion from the printer's MQTT state (`dry_time` falling from a positive value to `0`), not from how the cycle was started.

### Enabling

1. Open **Settings** > **Smart Plugs** (or expand the plug card on the dashboard)
2. On the plug that powers the printer and AMS, enable **Auto Off After Drying**
3. Set the **Drying delay (minutes)** — default **10 minutes**, gives the AMS chamber time to cool before power is cut

### How It Differs From Print-Finish Auto-Off

| | Print-Finish Auto-Off | Auto Off After Drying |
|---|---|---|
| **Trigger** | Print completes successfully | AMS `dry_time` reaches 0 |
| **Default delay** | 5 minutes | 10 minutes (chamber stays warmer) |
| **Cooldown mode** | Time or temperature (hotend) | Time only |
| **Per-plug toggle** | `auto_off` | `auto_off_after_drying` |

Both toggles are independent — you can enable either, both, or neither on the same plug.

!!! note "Per-AMS routing"
    The trigger is plug-vs-printer-level: if your printer has multiple AMS units on one plug, the auto-off fires whenever **any** of them finishes a cycle. Per-AMS targeting (a separate plug for the AMS, or different plugs for AMS 0 vs AMS 1 on dual-AMS printers) is not currently supported.

[:material-arrow-right: Full Smart Plug documentation](smart-plugs.md#auto-off-after-ams-drying)

---

## :material-chart-line: Historical Charts

Click on the humidity or temperature indicator to view historical data:

### Time Ranges

- **6 hours** - Recent trends
- **24 hours** - Daily pattern
- **48 hours** - Extended view
- **7 days** - Weekly overview

### Chart Features

- **Min/Max/Avg** statistics
- **Threshold reference lines**
- **Interactive tooltips**
- **Zoom and pan**

---

## :material-database: Data Retention

AMS data is stored for historical analysis:

### Default Retention

- **30 days** of humidity/temperature data
- Configurable in Settings

### Configuring Retention

1. Go to **Settings** > **General**
2. Find **AMS Data Retention**
3. Set number of days (1-365)
4. Older data is automatically purged

!!! warning "Storage Impact"
    Longer retention periods increase database size. Consider your available storage.

---

## :material-bell-ring: AMS Notifications

Get notified about AMS conditions:

### Available Alerts

| Event | Description |
|-------|-------------|
| **High Humidity** | When humidity exceeds threshold |
| **Low Filament** | When filament is running low |
| **AMS Error** | When AMS encounters issues |

### Setting Up

1. Go to **Settings** > **Notifications**
2. Add or edit a provider
3. Enable AMS-related events
4. Save changes

[:material-arrow-right: Full notifications guide](notifications.md)

---

## :material-fire: AMS-HT vs Standard AMS

| Feature | Standard AMS | AMS-HT |
|---------|-------------|--------|
| Humidity monitoring | :material-check: | :material-check: |
| Temperature monitoring | :material-close: | :material-check: |
| [Remote drying](#remote-ams-drying) | :material-close: | :material-check: |
| High-temp filaments | :material-close: | :material-check: |

### AMS-HT Temperature Control

AMS-HT units can actively dry filament. See [Remote AMS Drying](#remote-ams-drying) for setup and usage.

---

## :material-label: Custom AMS Labels

Assign friendly names to your AMS units to easily identify them in multi-AMS setups.

### AMS Info Card

Hover over any AMS label (e.g. "AMS-A") on the Printers page to see a popover with:

- **Serial Number** — The hardware serial from MQTT
- **Firmware Version** — Parsed from the printer's `get_version` response
- **Friendly Name** — An editable text field for your custom label

### Setting a Custom Label

1. Hover over the AMS label on the Printers page
2. Type a name in the "AMS Name" field (e.g. "Silk Colours", "Workshop AMS")
3. Click **Save** or press **Enter**
4. To remove a label, clear the field and save, or click **Clear**

!!! note "Permission Required"
    Editing AMS labels requires the `printers:update` permission when authentication is enabled.

### Label Persistence

Labels are stored by AMS serial number, so they **persist when the AMS is moved to a different printer**. If the AMS serial is not available (older firmware), a fallback key based on printer ID and AMS position is used.

Custom labels also appear in the **Inventory page** location column alongside the slot identifier, making it easy to find spools across a print farm.

### Slot Numbers

Each filament color circle now displays a 1-based slot number with automatically inverted text contrast — black text on light filaments, white text on dark filaments.

---

## :material-lan: AMS Discovery

Bambuddy automatically discovers connected AMS units:

- Detected during printer connection
- Updates when AMS configuration changes
- No manual configuration needed

### Dual-Nozzle Wiring

For H2D and other dual-nozzle printers, Bambuddy displays the AMS wiring configuration:

- Which AMS units feed which nozzle
- Visual diagram of connections
- Helps plan multi-material prints

### Nozzle-Aware Filament Mapping

On dual-nozzle printers (H2D, H2D Pro), each AMS unit is physically connected to either the left or right nozzle. The 3MF file's nozzle assignment is used to *pre-fill* the right slot per filament; you can still **manually override** with a tray on the other extruder when you've intentionally loaded the required filament there.

**Auto-match (pre-fill behaviour):**

1. The 3MF file contains `filament_nozzle_map` and `physical_extruder_map` in `project_settings.config`, mapping each filament slot to a target nozzle (0 = right, 1 = left)
2. The printer reports `ams_extruder_map` via MQTT, indicating which AMS unit feeds which nozzle
3. Auto-match prefers trays on the slicer-assigned nozzle when matching by type + colour
4. If no matching tray exists on the target nozzle, auto-match falls back to the full tray list

**Manual cross-extruder picks:**

The per-filament slot dropdown in the Print modal shows **every loaded AMS slot**, regardless of which extruder it feeds. The **L** / **R** badge stays next to each filament as a *hint to the slicer's intent*, but it does not hide slots on the other extruder. If you've loaded the required filament into an AMS on the "wrong" side on purpose (a common H2D workflow when one AMS is busy with another colour set), you can pick it.

The printer's firmware decides at start-print whether the resulting `ams_mapping` is physically valid; if a cross-extruder combination really can't be served, the firmware reports the error normally.

This applies to:

- **Print scheduler** — automatic filament matching for queued prints (still nozzle-preferring; never picks cross-extruder on its own)
- **Print modal** — filament mapping for ASAP, Queue, Schedule, and Manual Start dispatch options
- **Multi-printer selection** — per-printer mapping for print farms

!!! note "Single-Nozzle Printers"
    On single-nozzle printers (X1C, P1S, A1, etc.), nozzle filtering is not applied. All AMS trays are available for matching as before.

### Filament Track Switch (FTS) Support

The **Filament Track Switch** is an external accessory for dual-nozzle printers that sits between an AMS and the printer's extruders, dynamically routing any AMS slot to either nozzle. With the FTS installed, the AMS is no longer wired to a single extruder — it can feed either one through the track switch.

When Bambuddy detects the FTS (via `print.device.fila_switch` in the printer's MQTT push), the per-nozzle filter is automatically suppressed in the print modal:

- **Without FTS** — each AMS unit feeds a fixed nozzle, so the dropdown only shows trays on the matching nozzle (preventing the "position of left hotend is abnormal" failure that comes from cross-nozzle assignment).
- **With FTS installed** — every loaded AMS slot is selectable for any nozzle's filament, since the FTS handles the routing on the fly.

**Routing badges:** slots currently fed into a track display an `[L]` or `[R]` badge in the dropdown, indicating which extruder the FTS is currently routing them to. Idle slots (not currently in any track) show no badge — they can be routed on demand.

!!! note "Detection is automatic"
    There's no setting to toggle. The FTS is detected from MQTT in real time, so plugging in or removing the accessory updates the dropdown behaviour on the next status push.

---

## :material-sync: Spoolman Integration

Sync AMS slots with Spoolman for complete filament tracking:

1. Configure Spoolman integration in Settings
2. AMS slots sync automatically
3. Track usage across your inventory

[:material-arrow-right: Spoolman integration guide](spoolman.md)

---

## :material-lightbulb: Tips

!!! tip "Desiccant Maintenance"
    When humidity consistently stays high, it's time to replace or regenerate your desiccant packets.

!!! tip "Filament Storage"
    For filaments not in the AMS, store in vacuum bags with desiccant to maintain low humidity.

!!! tip "Historical Analysis"
    Review humidity charts to understand how your environment affects filament condition over time.

!!! tip "AMS-HT for Hygroscopic Filaments"
    Consider AMS-HT for moisture-sensitive materials like Nylon, PC, and PETG.

!!! tip "Auto-Drying Between Prints"
    Enable queue auto-drying to keep filament dry during long print queues, or enable ambient drying to keep filament dry on all idle printers — even when no prints are scheduled. Both use the Fair humidity threshold as the trigger point.
