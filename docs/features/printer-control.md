---
title: Printer Control
description: Print from printer cards, view printer info, control print speed, chamber, fans, and AI detection settings
---

# Printer Control

Bambuddy provides control over various printer settings and features directly from the web interface.

---

## :material-printer: Print from Printer Card

Start prints directly from any printer card — no need to navigate to File Manager first.

### Print Button

A green **Print** button appears on each printer card when the printer is idle:

1. Click the **Print** button at the bottom of the printer card
2. A file upload modal opens — select a `.gcode` or `.gcode.3mf` file
3. The file is automatically uploaded to your library
4. The **Print Modal** opens with the printer pre-selected
5. Configure filament mapping and print options, then click **Print**

The printer selector is hidden since the target printer is already known.

### Drag & Drop

Drag a sliced file directly onto any printer card to start printing:

1. Drag a `.gcode` or `.gcode.3mf` file from your computer onto a printer card
2. A green **"Drop to print"** overlay appears (or red **"Printer busy"** if unavailable)
3. Drop the file — it uploads to your library automatically
4. The **Print Modal** opens with the printer pre-selected
5. Configure and print

!!! tip "Accepted File Types"
    Only sliced files (`.gcode` and `.gcode.3mf`) are accepted for drag-and-drop printing. Other file types will show an error. Use the File Manager to upload non-printable files.

### Printer Compatibility Check

Both flows automatically check if the file was sliced for a compatible printer model. If the file's `sliced_for_model` metadata doesn't match the target printer, you'll see an error and the file is removed from the library.

!!! note "Permission Required"
    Requires the **Printer Control** permission (`printers:control`).

---

## :material-information: Printer Information

View detailed printer information from the three-dot menu on any printer card.

Click **Printer Information** to see:

| Field | Description |
|-------|-------------|
| Model | Printer model name |
| Status | Connection status (online/offline) |
| State | Current state (idle, printing, paused, etc.) |
| IP Address | Network address (copyable) |
| Serial Number | Hardware serial (copyable) |
| WiFi Signal | Signal strength with quality indicator |
| Firmware | Current firmware version |
| Developer Mode | Whether LAN developer mode is enabled |
| Nozzle Count | Number of nozzles |
| SD Card | Whether an SD card is inserted |
| Auto-Archive | Whether auto-archiving is enabled |
| Print Hours | Total accumulated print hours |
| Location | Configured printer location |
| Added | Date the printer was added |

---

## :material-stop: Print Job Controls

Control active print jobs directly from the printer card:

### Stop Print

Cancel the current print job:

1. Locate the **Stop** button (:material-stop:) on the printer card during printing
2. Click to stop the print
3. Confirm in the modal dialog
4. Print is cancelled and printer returns to idle

!!! warning "Permanent Action"
    Stopping a print cannot be undone. The print must be restarted from the beginning.

### Pause / Resume

Temporarily pause and resume printing:

| Button | State | Action |
|--------|-------|--------|
| :material-pause: Pause | Printing | Pauses the print, filament retracts |
| :material-play: Resume | Paused | Resumes printing from where it stopped |

**To pause/resume:**

1. Click the **Pause** button during an active print
2. Confirm in the modal dialog
3. Printer pauses and displays "Paused" status
4. Click **Resume** to continue printing

!!! tip "When to Pause"
    Use pause to inspect the print, remove debris, or insert objects for encapsulation.

### Clear HMS Errors

Dismiss stale HMS errors directly from the HMS error modal:

1. Click the **HMS indicator** on the printer card to open the error modal
2. Review the listed errors
3. Click **Clear Errors** to dismiss them

The clear command sends `clean_print_error` via MQTT and immediately removes errors from the UI. This is useful after print cancellation or transient events that leave `print_error` values behind.

!!! note "Permission Required"
    Requires the **Printer Control** permission (`printers:control`).

### Clear Plate

When a print finishes or fails and there are queued prints waiting, a "Clear Plate & Start Next" button appears on the printer card. Clicking it confirms that the build plate has been cleared, allowing the queue scheduler to start the next print.

!!! note "Permission Required"
    Requires the **Clear Plate** permission (`printers:clear_plate`). This is a separate, more granular permission than `printers:control`, allowing admins to grant plate-clearing ability without full printer control access.

### Confirmation Modals

All print control actions require confirmation to prevent accidental clicks:

- **Stop**: "Are you sure you want to stop this print?"
- **Pause**: "Are you sure you want to pause this print?"
- **Resume**: "Are you sure you want to resume this print?"

Toast notifications provide feedback after each action.

---

## :material-checkbox-multiple-marked: Bulk Actions

Select multiple printer cards and apply actions to all of them at once — ideal for print farms and multi-printer setups.

### Entering Selection Mode

1. Click the **Select** button (:material-checkbox-marked:) in the printer page header toolbar
2. Printer cards now show checkboxes — click cards to select/deselect
3. A floating toolbar appears at the bottom of the screen

### Selection Shortcuts

The floating toolbar provides quick ways to select printers:

| Button | Description |
|--------|-------------|
| **Select All** | Select all visible printers |
| **Select by State** | Dropdown to select all printers in a specific state (Printing, Paused, Finished, Idle, Error, Offline) |
| **Select by Location** | Dropdown to select all printers at a specific location (only visible when printers have locations assigned) |

When viewing printers grouped by location, each location header also shows a "Select All" link to select that group.

### Available Bulk Actions

| Action | Description | Confirmation |
|--------|-------------|:------------:|
| **Stop** | Cancel active prints on selected printers | :material-check: |
| **Pause** | Pause active prints on selected printers | :material-check: |
| **Resume** | Resume paused prints on selected printers | — |
| **Clear Notifications** | Clear HMS errors on selected printers | — |
| **Clear Bed** | Clear print bed on selected printers (may trigger queued jobs) | :material-check: |

Each action button is **smart-enabled** — it only activates when at least one selected printer is in the appropriate state. For example, "Stop" is only clickable when a selected printer is printing or paused.

### Permissions

- Stop, Pause, Resume, Clear Notifications require the **Printer Control** permission (`printers:control`)
- Clear Bed requires the **Clear Plate** permission (`printers:clear_plate`)

### Exiting Selection Mode

- Press **Escape** on your keyboard
- Click the **X** button on the floating toolbar

---

## :material-skip-forward: Skip Objects

Skip individual objects during a print without stopping the entire job.

### How It Works

When a print starts, Bambuddy extracts object information from the 3MF file. You can then skip any object that's failing or that you no longer need.

### Using Skip Objects

1. Click the **Skip** button (top-right of printer card) during printing
2. A modal appears showing:
   - **Preview image** with object ID markers overlaid in a grid
   - **Object list** with large ID badges and names
   - **Info banner** explaining to match IDs with your printer's display
3. Click **Skip** next to any object you want to exclude
4. The object will be marked as skipped and won't be printed

### Skip Objects Modal

| Element | Description |
|---------|-------------|
| Preview image | Shows print with ID markers overlaid |
| ID badges | Large numbered badges matching printer display |
| Object list | Names with skip buttons |
| Skipped count | Badge on button shows how many skipped |

### Object Display

| Status | Appearance |
|--------|------------|
| Active | Green ID badge, Skip button visible |
| Skipped | Red ID badge, strikethrough text, "Skipped" badge |

### Layer Requirement

!!! warning "Wait for Layer 2"
    Skip is only available after the first layer is printed. This is a printer limitation - the printer needs to establish object positions before allowing skips.

    A warning message appears in the modal when on layer 0 or 1.

### Requirements

For skip objects to work properly:

1. Enable **Exclude Objects** in your slicer (Bambu Studio / OrcaSlicer)
   - Go to **Other** → **Exclude objects**
   - This ensures each object gets a unique ID
2. Objects must be properly separated in the slicer
3. Clone/array objects may share IDs unless exclude is enabled
4. Print must have 2 or more objects

!!! tip "Matching Object IDs"
    The printer's display shows object IDs on the build plate visualization. Match these numbers with the IDs shown in Bambuddy's modal to identify which object is which.

!!! tip "When to Skip"
    Use skip objects when:

    - An object is failing but others are printing fine
    - You want to cancel part of a multi-object print
    - A support structure or test piece is no longer needed

!!! warning "Cannot Undo"
    Skipping an object is permanent for the current print. The object cannot be un-skipped once marked.

---

## :material-thermometer: Chamber Control

For enclosed printers (X1 series, H2 series), control the chamber environment:

### Chamber Temperature

- View current chamber temperature
- Set target temperature (if supported)
- Monitor heating/cooling status

### Chamber Light

Toggle the internal chamber LED directly from the printer card.

**Location:** Light button next to the camera button at the bottom of the printer card.

| State | Appearance |
|:-----:|------------|
| On | Yellow background, filled bulb icon with rays |
| Off | Gray background, outline bulb icon |

**To toggle:**

1. Click the light button at the bottom of the printer card
2. Light state changes immediately (optimistic update)
3. Toast notification confirms the change

!!! info "H2D Dual Lights"
    On H2D printers with dual chamber lights, both lights are controlled together.

---

## :material-speedometer: Print Speed Control

Change print speed presets on-the-fly during active prints without leaving the printer card.

### Speed Badge

A compact badge with a :material-gauge: gauge icon appears in the controls row, next to the fan status badges. It displays the current speed percentage at a glance.

| State | Appearance |
|:-----:|------------|
| **Active print** | Colored badge showing current speed % — click to open speed menu |
| **No active print** | Dimmed/disabled badge — visible but not interactive |

### Speed Presets

Click the speed badge to open a dropdown menu with four presets:

| Preset | Speed | Mode | Description |
|--------|:-----:|:----:|-------------|
| :material-tortoise: **Silent** | 50% | 1 | Quieter operation, reduced vibration |
| :material-printer-3d: **Standard** | 100% | 2 | Default slicer speed |
| :material-run-fast: **Sport** | 124% | 3 | Faster printing with good quality |
| :material-lightning-bolt: **Ludicrous** | 166% | 4 | Maximum speed |

### How to Change Speed

1. Locate the **speed badge** (:material-gauge: with percentage) in the controls row during an active print
2. Click the badge to open the speed dropdown
3. Select a preset — the command is sent immediately via MQTT
4. The badge updates to reflect the new speed

!!! tip "When to Adjust Speed"
    - Use **Silent** for night prints or noise-sensitive environments
    - Use **Sport** or **Ludicrous** for simple geometry where quality impact is minimal
    - Drop to **Standard** for detailed sections or overhangs

!!! note "Permission Required"
    Requires the **Printer Control** permission (`printers:control`).

---

## :material-snowflake: Airduct Mode (P2S / X2D / H2*)

Switch the chamber airduct between **Cooling** and **Heating** modes from the printer card. Available only on P2S, X2D, H2D, H2C, and H2S — printers without an active airduct don't show the badge.

### Airduct Badge

A compact badge appears in the controls row, immediately to the left of the print speed badge.

| Mode | Icon | Color | When to Use |
|------|:----:|:-----:|-------------|
| **Cooling** | :material-snowflake: | sky blue | PLA / PETG / TPU — filters and cools chamber air |
| **Heating** | :material-fire: | orange | ABS / ASA / PC / PA — circulates and heats chamber air, closes top exhaust flap |

Click the badge to open a dropdown and pick the mode. The command is sent via MQTT (`set_airduct`) and the badge updates immediately on confirmation.

---

## :material-arrow-up-down: Move Build Plate (Z-Jog)

Move the build plate up or down directly from the printer card — useful for inspecting the plate through the camera after a print finishes, or for small manual positioning.

### Bed-Jog Badge

A compact **Bed** badge appears in the controls row, between the print-speed badge and the Stop / Pause buttons. Click it to open a small popover containing:

- **↑ / ↓** buttons that move the plate by the selected step
- A **step selector** — `1 / 10 / 50 mm`

The badge is automatically disabled while a print is running.

### Not-Homed Warning (Studio-style)

After a print completes, the Z axis is usually no longer referenced. The first time you click up/down in a session, Bambuddy shows a warning modal matching the Bambu Studio / printer-touchscreen flow:

| Action | What it does |
|--------|--------------|
| **Home Z** | Sends `G28 Z` and dismisses the dialog — click the jog again once homing completes |
| **Move anyway** | Bypasses soft endstops (`M211 S0`) for this move, performs the jog, then re-enables endstops (`M211 S1`). The "already warned" flag is remembered for the rest of the browser session, so subsequent jogs go through without the dialog |
| **Cancel** | Closes the dialog, no command is sent |

!!! warning "Bypassing soft endstops"
    The **Move anyway** option disables axis soft limits for a single move. Keep the step small (1 – 10 mm) until the plate is in a safe position — the printer firmware will still refuse moves that would physically crash the gantry, but it's on you to make sure the commanded distance is reasonable.

### Permissions

Both the jog and home actions require `printers:control`. With authentication enabled, anonymous and read-only users see the buttons greyed out.

### Under the Hood

| Endpoint | G-code Sent |
|----------|-------------|
| `POST /printers/{id}/bed-jog?distance=N` | `G91 / G1 ZN F600 / G90` |
| `POST /printers/{id}/bed-jog?distance=N&force=true` | `M211 S0 / G91 / G1 ZN F600 / G90 / M211 S1` |
| `POST /printers/{id}/home-axes?axes=z` | `G28 Z` |
| `POST /printers/{id}/home-axes?axes=xy` | `G28 X Y` |
| `POST /printers/{id}/home-axes?axes=all` | `G28` |

The `distance` parameter is validated server-side to be non-zero and within ±200 mm.

---

## :material-door: SD Card &amp; Door Status Badges

The top status row of every printer card now shows two compact icon-only badges so you can see drive and enclosure state at a glance.

| Badge | Green | Red / Yellow | Notes |
|-------|:-----:|:------------:|-------|
| :material-sd: **SD Card** | inserted | red when missing | All printers |
| :material-door-closed: **Door** | closed | yellow when open | X1 / P1S / P2S / X2D / H2 series only |

Door state is detected from the right MQTT field per printer family (X1: `home_flag` bit 23, others: `stat` bit 23) and pushed live via WebSocket — no waiting for the next status poll.

---

## :material-refresh: Force Refresh

The printer card kebab menu has a **Force Refresh** entry that triggers an MQTT `pushall` request, asking the printer to re-broadcast its full status. Useful when a value looks stale and you don't want to fully reconnect (which would tear down the existing MQTT/FTP session).

---

## :material-fan: Fan Status

Monitor cooling fan speeds in real time directly from the printer card.

### Real-Time Fan Status

The Controls section displays compact fan badges showing real-time speeds:

| Fan | Icon | Color | Description |
|-----|:----:|:-----:|-------------|
| **Part Cooling** | :material-fan: | Cyan | Cools printed layers (0-100%) |
| **Auxiliary** | :material-weather-windy: | Blue | Chamber airflow (0-100%) |
| **Chamber** | :material-air-filter: | Green | Exhausts hot air from enclosure (0-100%) |

Badges are always visible with dynamic coloring:

- **Active** (colored icon + text): Fan is running, shows current speed %
- **Inactive** (gray icon + text): Fan is off, shows 0%

!!! info "Display Only"
    Fan badges show the current speed reported by the printer. Fan speeds are controlled by the slicer profile and printer firmware — they cannot be adjusted through BambuBuddy.

---

## :material-nozzle: Dual Nozzle Support

For dual-nozzle printers (H2D, etc.):

### Nozzle Status

View both nozzles:

- Current temperature
- Target temperature
- Heating status

### Nozzle Selection

See which nozzle is active during printing.

---

## :material-power: Power Controls

Control printer power state:

### Power Switch

If a smart plug is configured:

- **Power On** - Turn printer on
- **Power Off** - Turn printer off
- **Status** - View current power state

[:material-arrow-right: Smart plug setup](smart-plugs.md)

### Auto Power Off

Configure automatic shutdown:

1. Enable **Auto Power Off** in settings
2. Set **Cooldown Temperature** threshold
3. Set **Cooldown Time** delay
4. Printer powers off after print completes and cools

---

## :material-printer-settings: Print Settings Override

During an active print, you can adjust:

| Setting | Adjustable? |
|---------|:-----------:|
| Print speed preset | :material-check: |
| Part cooling fan | :material-check: |
| Chamber light | :material-check: |
| Pause/Resume | :material-check: |
| Cancel | :material-check: |

!!! warning "Limited Changes"
    Not all settings can be changed mid-print. Temperature and layer settings are locked.

---

## :material-lightbulb: Tips

!!! tip "Chamber Management"
    For ABS/ASA, keep the chamber warm. Open doors for PLA to prevent heat creep.

!!! tip "AI Detection Value"
    AI detection has saved countless prints. The small time cost is worth the protection.

!!! tip "Regular Calibration"
    Run bed leveling from the printer's touchscreen after major moves or every few weeks for best first layers.
