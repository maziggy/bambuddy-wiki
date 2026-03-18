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

## :material-fan: Fan Controls

Monitor and adjust cooling fans directly from the printer card.

### Real-Time Fan Status

The Controls section displays compact fan badges showing real-time speeds:

| Fan | Icon | Color | Description |
|-----|:----:|:-----:|-------------|
| **Part Cooling** | :material-fan: | Cyan | Cools printed layers (0-100%) |
| **Auxiliary** | :material-weather-windy: | Blue | Controls chamber airflow (0-100%) |
| **Chamber** | :material-air-filter: | Green | Exhausts hot air (0-100%) |

Badges are always visible with dynamic coloring:

- **Active** (colored icon + text): Fan is running
- **Inactive** (gray icon + text): Fan is off, shows 0%

### Part Cooling Fan

- View current speed (%)
- Adjust during print if needed
- Auto-controlled by slicer normally

### Auxiliary Fan

- Controls airflow in the chamber
- Helps with certain materials
- Can be adjusted manually

### Chamber Fan

- Exhausts hot air from enclosure
- Important for high-temp printing
- Auto-controlled based on chamber temp

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
