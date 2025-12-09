# Spoolman Integration

Sync your AMS filament data with [Spoolman](https://github.com/Donkie/Spoolman) for comprehensive filament inventory management.

## Overview

When enabled, Bambuddy syncs AMS filament data with your Spoolman server:
- Track remaining filament across all spools
- Automatic usage deduction during prints
- Auto-create spools and filaments in Spoolman
- Support for multiple printers and AMS units

---

## Prerequisites

- A running Spoolman server (self-hosted or Docker)
- Bambu Lab spools with original RFID tags in your AMS
- Printer connected to Bambuddy

---

## Setting Up Spoolman

### Step 1: Enable Integration
1. Go to **Settings** > scroll to **Spoolman Integration**
2. Enable the **Enable Spoolman** toggle

### Step 2: Configure Connection
1. Enter your Spoolman server URL:
   - Example: `http://192.168.1.100:7912`
   - Include protocol (http/https) and port
2. Click **Save**

### Step 3: Connect
1. Click **Connect** to establish the connection
2. Status should change to "Connected"
3. If connection fails, verify URL and network access

---

## Sync Modes

### Automatic Sync
AMS data syncs automatically when changes are detected:
- Filament loaded/unloaded
- Filament usage during prints
- AMS slot changes

Enable automatic sync for hands-off operation.

### Manual Only
Sync only when you explicitly request it:
- Click the **Sync** button
- Select specific printer or "All Printers"
- Results show how many trays were synced

---

## How Syncing Works

### Tray UUID Matching
Bambuddy matches AMS spools to Spoolman using the **tray UUID**:
- Unique 32-character identifier
- Assigned by Bambu Lab to each original spool
- Consistent across different printers

### What Gets Synced

| Data | Direction | Description |
|------|-----------|-------------|
| Remaining weight | AMS → Spoolman | Current filament amount |
| Usage during print | AMS → Spoolman | Deducted from inventory |
| Filament type | AMS → Spoolman | PLA, PETG, ABS, etc. |
| Color | AMS → Spoolman | Spool color |

### Auto-Creation

When a Bambu Lab spool is detected that doesn't exist in Spoolman:

1. **Vendor**: "Bambu Lab" vendor created if needed
2. **Filament**: Filament type created (matched by material/color)
3. **Spool**: New spool created with:
   - Tray UUID as identifier
   - Material type and color
   - Comment noting auto-creation

---

## Supported Spools

### Bambu Lab Original Spools ✓
- Have valid RFID tags with tray UUIDs
- Full sync support
- Automatic matching

### Third-Party Spools ✗
These are gracefully skipped (no errors):
- SpoolEase refilled spools
- Other refilled spools
- Generic filament without RFID
- Spools with invalid/missing UUIDs

> Third-party spools don't cause sync errors—they're simply skipped.

---

## Manual Sync

When using manual sync mode:

1. Go to **Settings** > **Spoolman Integration**
2. Select a printer from the dropdown (or "All Printers")
3. Click **Sync**
4. Results display:
   - Number of trays synced
   - Any skipped spools
   - Error messages if applicable

---

## Viewing Synced Data

### In Spoolman
After syncing, check your Spoolman interface:
- New spools appear in inventory
- Remaining amounts are updated
- Usage history is recorded

### In Bambuddy
AMS data displayed on printer cards shows:
- Slot colors and materials
- Remaining filament percentages
- Sync status indicators

---

## Troubleshooting

### Connection Issues

**"Connection refused" or timeout**
- Verify Spoolman URL is correct
- Check server is running: `http://<spoolman-ip>:7912/api/v1/info`
- Ensure no firewall blocking port 7912
- Verify same network or proper routing

**"Invalid URL"**
- Include protocol: `http://` or `https://`
- Include port: `:7912`
- No trailing slash

### Sync Issues

**"No spools synced"**
- Verify AMS has spools loaded
- Check spools have valid RFID (original Bambu Lab)
- Ensure printer is connected

**"Third-party spools skipped"**
- This is normal behavior
- SpoolEase and refilled spools don't have valid UUIDs
- No action needed

**"Spool not found in Spoolman"**
- Auto-creation should handle this
- Check Spoolman logs for errors
- Verify Spoolman API is accessible

### Data Mismatch

**Spoolman shows wrong amount**
- Manual edits in Spoolman override synced data
- Re-sync to update from AMS
- AMS sensor may have calibration issues

---

## Best Practices

### Initial Setup
1. Do a full manual sync first
2. Verify all spools appear in Spoolman
3. Then enable automatic sync

### Filament Changes
- Sync after loading new spools
- Spoolman updates automatically during prints
- Check periodically for accuracy

### Multiple Printers
- Each printer's AMS syncs independently
- Same spool moved between printers is tracked by UUID
- Use "All Printers" sync for comprehensive update

---

## API Endpoints

Bambuddy uses these Spoolman API endpoints:

| Endpoint | Purpose |
|----------|---------|
| `/api/v1/info` | Connection test |
| `/api/v1/spool` | List/create spools |
| `/api/v1/filament` | List/create filaments |
| `/api/v1/vendor` | List/create vendors |
| `/api/v1/spool/{id}/use` | Record usage |

---

## Tips

- Use automatic sync for seamless tracking
- Check Spoolman periodically for accuracy
- Original Bambu Lab spools work best
- Third-party spools need manual tracking in Spoolman
- Export Spoolman data for backup
