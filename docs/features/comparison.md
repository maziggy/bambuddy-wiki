---
title: Archive Comparison
description: Compare print settings side-by-side
---

# Archive Comparison

Compare 2-5 archives side-by-side to understand setting differences and optimize your prints.

---

## :material-compare: How It Works

Archive comparison extracts settings from 3MF files and displays them in a table with differences highlighted.

### Starting a Comparison

**Method 1: Context Menu**

1. Right-click an archive
2. Select **Compare**
3. Select up to 4 more archives
4. Click **Compare**

**Method 2: Multi-Select**

1. Select 2-5 archives (Ctrl+click or Shift+click)
2. Right-click the selection
3. Choose **Compare Selected**

---

## :material-table: Comparison View

Archives are displayed in columns with settings in rows:

| Setting | Benchy v1 | Benchy v2 | Benchy v3 |
|---------|:---------:|:---------:|:---------:|
| Layer Height | 0.20mm | **0.16mm** | 0.20mm |
| Infill | 15% | 15% | **20%** |
| Speed | 100mm/s | **80mm/s** | 100mm/s |
| Material | PLA | PLA | **PETG** |

### Highlighting

- **Bold** values indicate differences from other archives
- Makes it easy to spot what changed between prints

---

## :material-cog: Compared Settings

### Print Settings

| Setting | Description |
|---------|-------------|
| Layer height | First layer and subsequent layers |
| Wall loops | Number of perimeter walls |
| Infill pattern | Grid, gyroid, honeycomb, etc. |
| Infill percentage | Fill density |
| Top/bottom layers | Solid layer count |

### Speed Settings

| Setting | Description |
|---------|-------------|
| Print speed | Overall movement speed |
| Travel speed | Non-printing moves |
| First layer speed | Initial layer speed |
| Infill speed | Internal fill speed |

### Temperature Settings

| Setting | Description |
|---------|-------------|
| Nozzle temp | Hotend temperature |
| Bed temp | First layer and subsequent |
| Chamber temp | Enclosure temperature |

### Material Settings

| Setting | Description |
|---------|-------------|
| Filament type | PLA, PETG, ABS, etc. |
| Filament brand | Manufacturer |
| Color | Filament color |

### Support Settings

| Setting | Description |
|---------|-------------|
| Support enabled | Yes/no |
| Support type | Normal, tree, etc. |
| Support pattern | Grid, lines, etc. |
| Support interface | Interface layers |

---

## :material-chart-bar: Results Comparison

Beyond settings, compare print results:

| Metric | Benchy v1 | Benchy v2 | Benchy v3 |
|--------|:---------:|:---------:|:---------:|
| Duration | 2h 15m | **2h 45m** | 2h 10m |
| Filament | 45g | 42g | **48g** |
| Result | Success | Success | **Failed** |

This helps correlate settings with outcomes.

---

## :material-magnify: Difference Filtering

Focus on what matters:

### Show Only Differences

Toggle to hide settings that are identical across all archives.

### Group by Category

Organize settings by type:

- Print settings
- Speed settings
- Temperature settings
- Material settings
- Support settings

---

## :material-content-copy: Copying Settings

Found better settings? Apply them to future prints:

1. Note the settings from the comparison
2. Update your slicer profile
3. Or use the K-profiles feature for pressure advance settings

---

## :material-file-export: Export Comparison

Export the comparison as:

- **CSV** - For spreadsheets
- **Markdown** - For documentation

---

## :material-history: Comparison History

Recent comparisons are saved:

- Access from the comparison page
- Re-run previous comparisons quickly
- No need to re-select archives

---

## :material-lightbulb: Use Cases

!!! tip "Troubleshooting Failures"
    Compare a successful print with a failed one to identify what changed.

!!! tip "Optimizing Quality"
    Compare prints with different layer heights to see the quality/time tradeoff.

!!! tip "Material Comparison"
    Same model in different materials - compare settings and results.

!!! tip "Speed Tuning"
    Compare prints at different speeds to find the sweet spot.

!!! tip "Profile Validation"
    Compare prints from different slicer profiles to ensure consistency.

---

## :material-alert: Limitations

- Maximum 5 archives per comparison
- Requires 3MF file (not all settings available for imported archives)
- Some slicer-specific settings may not be parsed
