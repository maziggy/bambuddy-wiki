---
title: Printer Control
description: Control chamber, speed, fans, and AI detection settings
---

# Printer Control

Bambuddy provides control over various printer settings and features directly from the web interface.

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

Toggle the internal chamber LED:

| State | Description |
|:-----:|-------------|
| :material-lightbulb-on: On | Light is active |
| :material-lightbulb-off: Off | Light is off |

Control via the printer card or detailed view.

---

## :material-speedometer: Speed Profiles

Adjust print speed profiles on the fly:

### Available Profiles

| Profile | Description | Use Case |
|---------|-------------|----------|
| **Silent** | Slowest, quietest | Night printing |
| **Standard** | Balanced speed/noise | Default |
| **Sport** | Faster printing | When speed matters |
| **Ludicrous** | Maximum speed | Speed printing |

### Changing Speed

1. Click the speed indicator on the printer card
2. Select a new profile
3. Change applies immediately to current print

!!! warning "Quality Impact"
    Higher speeds may affect print quality. Use Sport/Ludicrous for non-critical prints.

---

## :material-fan: Fan Controls

Monitor and adjust cooling fans:

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

## :material-robot: AI Detection Modules

Bambu Lab printers include AI-powered detection features:

### Spaghetti Detection

Detects when prints fail and create "spaghetti" tangles:

| Setting | Description |
|---------|-------------|
| **Enabled** | AI monitors for failures |
| **Disabled** | No automatic detection |
| **Pause on detect** | Pauses print when detected |

### First Layer Detection

Monitors first layer adhesion:

| Setting | Description |
|---------|-------------|
| **Enabled** | AI checks first layer |
| **Disabled** | No first layer monitoring |

### Configuring AI Detection

1. Go to printer settings
2. Find **AI Detection** section
3. Toggle features on/off
4. Save changes

!!! tip "Recommended"
    Keep AI detection enabled - it can save failed prints and prevent wasted filament.

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
| Speed profile | :material-check: |
| Part cooling fan | :material-check: |
| Chamber light | :material-check: |
| Pause/Resume | :material-check: |
| Cancel | :material-check: |

!!! warning "Limited Changes"
    Not all settings can be changed mid-print. Temperature and layer settings are locked.

---

## :material-lightbulb: Tips

!!! tip "Silent Night Printing"
    Use Silent mode for overnight prints to minimize noise.

!!! tip "Chamber Management"
    For ABS/ASA, keep the chamber warm. Open doors for PLA to prevent heat creep.

!!! tip "AI Detection Value"
    AI detection has saved countless prints. The small time cost is worth the protection.

!!! tip "Regular Calibration"
    Run bed leveling from the printer's touchscreen after major moves or every few weeks for best first layers.
