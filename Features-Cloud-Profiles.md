# Cloud Profiles

Access and manage your Bambu Cloud slicer presets with powerful template and comparison features.

## Overview

Cloud Profiles syncs with your Bambu Cloud account to provide:
- View and manage filament, printer, and process presets
- Template system for quick preset creation
- Side-by-side preset comparison with diff view
- Local storage of synced presets

---

## Connecting to Bambu Cloud

### Step 1: Get Your Credentials
You'll need your Bambu Cloud account credentials or access token.

### Step 2: Configure in Bambuddy
1. Go to **Settings** > **Cloud Profiles**
2. Enter your credentials
3. Click **Connect**
4. Presets will sync automatically

### Step 3: Verify Connection
- Status shows "Connected"
- Presets appear in the Cloud Profiles page

---

## Preset Types

### Filament Presets
Settings for specific filament types:
- Temperature profiles
- Cooling settings
- Retraction settings
- Flow calibration

### Printer Presets
Machine-specific settings:
- Build volume
- Speed limits
- Acceleration values
- Hardware capabilities

### Process Presets
Print quality and speed settings:
- Layer height
- Print speed
- Infill patterns
- Support settings

---

## Browsing Presets

### Filtering
- **By Type**: Filament, Printer, Process
- **By Source**: Official, User-created, Templates
- **Search**: Find by name

### Preset Cards
Each preset shows:
- Name and type
- Source (official/user)
- Base preset (if derived)
- Last modified date

### Preset Details
Click a preset to view:
- All settings and values
- Base preset reference
- Modification history

---

## Template System

Create reusable templates from any preset for quick preset creation.

### Creating a Template

1. Find a preset you want to use as a base
2. Click the template icon (or right-click > "Save as Template")
3. Give the template a name
4. Template is saved for future use

### Template Visibility

Control which templates appear in creation modals:
1. Go to template management
2. Toggle visibility for each template
3. Only visible templates show in dropdowns

### Using Templates

When creating a new preset:
1. Click "New Preset"
2. Select a template from the dropdown
3. Template settings are pre-filled
4. Modify as needed
5. Save the new preset

---

## Preset Comparison

Compare presets side-by-side to understand differences.

### Compare with Base

See what's changed from the original preset:
1. Open a preset that's derived from another
2. Click "Compare with Base"
3. View differences highlighted

### Compare Any Two Presets

Compare any presets of the same type:
1. Select first preset
2. Click "Compare"
3. Select second preset
4. View side-by-side diff

### Diff View Features

| Feature | Description |
|---------|-------------|
| **Added** | Settings only in new preset (green) |
| **Removed** | Settings only in base preset (red) |
| **Changed** | Different values (yellow) |
| **Search** | Filter diff by setting name |
| **Statistics** | Count of changes by type |

---

## Managing Presets

### Editing Presets
1. Open the preset
2. Click "Edit"
3. Modify settings
4. Save changes

> Note: Official presets cannot be edited. Create a copy first.

### Creating Presets
1. Click "New Preset"
2. Select preset type
3. Choose a base preset or template
4. Configure settings
5. Save with a unique name

### Deleting Presets
1. Right-click the preset
2. Select "Delete"
3. Confirm deletion

> Official presets cannot be deleted.

---

## Syncing

### Automatic Sync
Presets sync periodically with Bambu Cloud:
- New presets downloaded
- Changes uploaded
- Deleted presets removed

### Manual Sync
Force a sync:
1. Go to Cloud Profiles page
2. Click "Sync Now"
3. Wait for sync to complete

### Sync Status
- **Synced**: Up to date with cloud
- **Pending**: Local changes not yet uploaded
- **Conflict**: Different changes locally and in cloud

---

## Local Storage

Synced presets are stored locally:
- Available offline after initial sync
- Faster loading times
- Backup of your presets

### Storage Location
Presets stored in SQLite database alongside other Bambuddy data.

---

## Use Cases

### Standardizing Settings
1. Create a "Standard PLA" preset with your preferred settings
2. Save as template
3. Use template for all PLA prints
4. Consistent quality across prints

### Experimenting Safely
1. Copy an existing preset
2. Make experimental changes
3. Compare with original
4. Keep or discard based on results

### Team Sharing
1. Create optimized presets
2. Export or share via Bambu Cloud
3. Team members sync the same presets
4. Consistent settings across all printers

---

## Troubleshooting

### Connection Failed
- Verify credentials are correct
- Check internet connection
- Try logging out and back in to Bambu Cloud

### Presets Not Appearing
- Force a manual sync
- Check filter settings
- Verify presets exist in Bambu Cloud

### Sync Conflicts
- Local changes may conflict with cloud
- Review differences in comparison view
- Choose which version to keep

---

## Tips

- Use templates for consistent starting points
- Compare before and after making changes
- Keep official presets as reference
- Create project-specific presets
- Regular syncs ensure latest settings
