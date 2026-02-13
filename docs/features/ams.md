---
title: AMS & Humidity Monitoring
description: Monitor AMS and AMS-HT filament systems
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

### Configure AMS Slot

Manually configure AMS slots for third-party or generic filaments:

1. Hover over an AMS slot on the printer card
2. Click the menu button (:material-dots-vertical:) that appears
3. Select **Configure Slot**
4. Choose a filament preset from your Bambu Studio cloud presets
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

### Multi-AMS Support

Bambuddy supports multiple AMS units per printer:

- Up to 4 AMS units (16 total slots)
- Each unit displayed with its slots
- Visual indication of active slot during printing

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
    Different filaments have different humidity sensitivities. PLA is more tolerant than Nylon or PETG.

---

## :material-thermometer: Temperature Monitoring

For AMS-HT (High Temperature) units, temperature is also tracked:

| Reading | Description |
|---------|-------------|
| **Current temp** | Live temperature inside the unit |
| **Target temp** | Configured drying temperature |
| **Status** | Heating, holding, or idle |

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
| Active drying | :material-close: | :material-check: |
| High-temp filaments | :material-close: | :material-check: |

### AMS-HT Temperature Control

AMS-HT units can actively dry filament:

- Set target temperature in printer settings
- Monitor heating status in Bambuddy
- Track temperature over time

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

On dual-nozzle printers (H2D, H2D Pro), each AMS unit is physically connected to either the left or right nozzle. When a 3MF file assigns filaments to specific nozzles, Bambuddy constrains filament matching to only AMS trays connected to the correct nozzle.

**How it works:**

1. The 3MF file contains `filament_nozzle_map` and `physical_extruder_map` in `project_settings.config`, mapping each filament slot to a target nozzle (0 = right, 1 = left)
2. The printer reports `ams_extruder_map` via MQTT, indicating which AMS unit feeds which nozzle
3. When matching filaments to AMS slots, only trays on the correct nozzle are considered
4. If no trays exist on the target nozzle, matching falls back to the full tray list

This applies to:

- **Print scheduler** — automatic filament matching for queued prints
- **Reprint modal** — filament mapping when reprinting from archives
- **Queue modal** — filament mapping when adding to queue
- **Multi-printer selection** — per-printer mapping for print farms

The filament mapping UI shows **L** (left) and **R** (right) badges next to each filament requirement for dual-nozzle prints.

!!! note "Single-Nozzle Printers"
    On single-nozzle printers (X1C, P1S, A1, etc.), nozzle filtering is not applied. All AMS trays are available for matching as before.

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
