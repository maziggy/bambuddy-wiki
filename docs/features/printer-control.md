---
title: Printer Control
description: Control chamber, speed, fans, and AI detection settings
---

# Printer Control

Bambuddy provides control over various printer settings and features directly from the web interface.

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

## :material-ruler: Automated Calibration

Trigger calibration routines from Bambuddy:

### Bed Leveling

Automatic bed mesh calibration:

1. Open printer settings
2. Click **Bed Level Calibration**
3. Printer runs the calibration sequence
4. Results stored in printer

### Vibration Compensation

Input shaping calibration:

1. Open printer settings
2. Click **Vibration Calibration**
3. Printer runs vibration test
4. Compensation values updated

!!! note "Calibration Time"
    Calibration routines take several minutes to complete. Ensure the printer is idle.

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

## :material-console: Sending Commands

Advanced users can send raw GCODE commands:

1. Open printer settings
2. Find **Send Command** section
3. Enter GCODE command
4. Click Send

!!! danger "Use with Caution"
    Incorrect GCODE can damage your printer or cause injuries. Only use if you understand the commands.

---

## :material-lightbulb: Tips

!!! tip "Silent Night Printing"
    Use Silent mode for overnight prints to minimize noise.

!!! tip "Chamber Management"
    For ABS/ASA, keep the chamber warm. Open doors for PLA to prevent heat creep.

!!! tip "AI Detection Value"
    AI detection has saved countless prints. The small time cost is worth the protection.

!!! tip "Regular Calibration"
    Run bed leveling after major moves or every few weeks for best first layers.
