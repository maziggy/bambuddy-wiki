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
