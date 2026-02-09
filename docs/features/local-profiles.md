---
title: Local Profiles
description: Import OrcaSlicer presets without Bambu Cloud
---

# Local Profiles

Import and manage slicer presets from OrcaSlicer without needing a Bambu Cloud account.

---

## :material-harddisk: Overview

Local Profiles lets you:

- **Import** OrcaSlicer preset exports (filament, process, and printer profiles)
- **Resolve** OrcaSlicer inheritance chains automatically
- **Browse** presets in a searchable 3-column layout
- **Configure** AMS slots using imported filament presets
- **Delete** presets you no longer need

This is ideal for users who use OrcaSlicer without Bambu Cloud and want to manage their presets through Bambuddy.

---

## :material-file-import: Supported File Types

| Format | Extension | Description |
|--------|-----------|-------------|
| OrcaSlicer Filament | `.orca_filament` | Single filament preset bundle |
| OrcaSlicer Config Bundle | `.bbscfg` | Full OrcaSlicer config bundle (filament, process, printer) |
| OrcaSlicer Filament Bundle | `.bbsflmt` | Filament preset bundle |
| ZIP Archive | `.zip` | ZIP containing OrcaSlicer JSON configs |
| JSON | `.json` | Single OrcaSlicer preset file |

---

## :material-upload: Importing Presets

### Drag and Drop

1. Go to **Profiles** > **Local Profiles** tab
2. Drag your export file onto the import zone
3. Wait for the import to complete
4. Imported presets appear in their respective columns

### File Picker

1. Click the import zone to open a file picker
2. Select one or more files to import
3. Multiple files can be imported at once

### Import Results

After import, you'll see toast notifications:

- **Green**: Successfully imported presets with count
- **Orange**: Skipped presets (duplicates already in database)
- **Red**: Errors during import

---

## :material-file-tree: Profile Types

Imported presets are automatically classified into three categories:

| Type | Description | Examples |
|------|-------------|---------|
| **Filament** | Material presets with temperature, flow, and color settings | `Overture PLA Matte @BBL X1C`, `eSUN PETG @Bambu Lab H2D` |
| **Process** | Print quality presets with layer height, speed, and infill settings | `0.20mm Standard @BBL X1C`, `0.08mm Extra Fine @BBL H2D` |
| **Printer** | Machine configuration presets | `Bambu Lab X1 Carbon 0.4 nozzle` |

Profile type is detected using multiple heuristics:

1. Explicit `type` field in the JSON
2. ZIP directory path (`filament/`, `process/`, `machine/`)
3. Settings ID keys (`print_settings_id`, `filament_settings_id`, `printer_settings_id`)
4. Content-based keys (e.g., `layer_height` for process, `filament_type` for filament)
5. Name patterns (e.g., `0.20mm` indicates process)

---

## :material-link-variant: Inheritance Resolution

Many OrcaSlicer presets use inheritance — they specify an `inherits` field pointing to a base Bambu profile and only override specific settings. For example, a custom PLA preset might inherit from `Bambu PLA Basic @BBL X1C` and only change the temperature.

During import, Bambuddy:

1. Detects the `inherits` field in the preset
2. Fetches the base profile from the [OrcaSlicer GitHub repository](https://github.com/SoftFever/OrcaSlicer/tree/main/resources/profiles/BBL)
3. Recursively resolves the full inheritance chain (up to 10 levels deep)
4. Merges parent and child settings (child keys override parent)
5. Stores the fully resolved preset in the database
6. Caches base profiles locally with a 7-day TTL

!!! tip "Offline Import"
    Presets without inheritance work fine without internet access. Base profile cache persists across imports — after the first import resolves a base profile, subsequent imports reuse the cache.

!!! note "GitHub Unreachable"
    If GitHub is unreachable during import, presets with `inherits` will be stored with their override-only data. The preset will still work but may have fewer fields populated.

---

## :material-format-list-bulleted: Preset Details

Click on any preset card to expand its details:

| Field | Description |
|-------|-------------|
| **Type** | Material type (PLA, PETG, ABS, etc.) |
| **Vendor** | Filament manufacturer |
| **Nozzle Temp** | Min-max nozzle temperature range |
| **Cost** | Filament cost per unit |
| **Density** | Filament density (g/cm3) |
| **Pressure Advance** | PA/K-factor value |
| **Printers** | Compatible printer models |
| **Inherits** | Base profile name (hidden if same as preset name) |
| **Source** | Import source (e.g., "orcaslicer") |

Filament presets show a color dot:

- **Full opacity**: Explicit color from the preset
- **50% opacity**: Color inferred from material type
- **25% opacity**: No color information available

---

## :material-water-percent: AMS Slot Configuration

Imported local filament presets automatically appear in the AMS slot configuration modal alongside cloud presets:

- Local presets are shown with a green **Local** badge
- Local presets appear first in the list, followed by cloud presets
- When configuring a slot with a local preset, Bambuddy uses the extracted nozzle temperature, material type, and color data

---

## :material-api: API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/local-presets/` | List all presets grouped by type |
| `GET` | `/api/v1/local-presets/{id}` | Get full preset detail with settings JSON |
| `POST` | `/api/v1/local-presets/import` | Import presets from uploaded file |
| `POST` | `/api/v1/local-presets/` | Manually create a preset |
| `DELETE` | `/api/v1/local-presets/{id}` | Delete a preset |
| `GET` | `/api/v1/local-presets/base-cache/status` | Check base profile cache status |
| `POST` | `/api/v1/local-presets/base-cache/refresh` | Force refresh cached base profiles |
| `POST` | `/api/v1/local-presets/reclassify` | Re-evaluate preset types for all presets |

All endpoints respect authentication permissions when auth is enabled (`SETTINGS_READ` for reads, `SETTINGS_UPDATE` for writes).

---

## :material-frequently-asked-questions: FAQ

**Q: Do I need a Bambu Cloud account?**
No. Local Profiles work entirely without Bambu Cloud. The only external request is fetching base profiles from OrcaSlicer's public GitHub repository during import.

**Q: Can I use both Cloud Profiles and Local Profiles?**
Yes. Both appear in the AMS slot configuration, with local presets shown first.

**Q: What happens if I import the same file twice?**
Duplicate presets (matched by name) are skipped. You'll see an orange toast showing the skip count.

**Q: How do I export presets from OrcaSlicer?**
In OrcaSlicer, go to **File** > **Export** > **Export Preset Bundle** and choose your format (.bbscfg, .bbsflmt, or .zip).
