---
title: Spoolman Integration
description: Sync filament inventory with Spoolman
---

# Spoolman Integration

Sync your AMS filament with [Spoolman](https://github.com/Donkie/Spoolman) for complete spool tracking and inventory management.

---

## :material-spool: What is Spoolman?

Spoolman is a self-hosted filament inventory manager that:

- Tracks spool quantities
- Records filament usage
- Manages multiple materials and vendors
- Integrates with slicers

Bambuddy syncs your AMS slots with Spoolman for unified tracking.

---

## :material-link: Connection Setup

### Requirements

- Spoolman instance running and accessible
- Network access from Bambuddy to Spoolman

### Configuration

1. Go to **Settings** > **Integrations**
2. Find **Spoolman** section
3. Enter:

| Field | Description |
|-------|-------------|
| **URL** | Spoolman server URL (e.g., `http://192.168.1.50:7912`) |
| **API Key** | If Spoolman requires authentication |

4. Click **Test Connection**
5. Click **Save**

---

## :material-sync: Sync Features

### How Sync Works

!!! note "Bambu Lab Spools Only"
    Only official Bambu Lab spools with RFID are synced automatically. Third-party, refilled, or SpoolEase spools are skipped. Bambu Lab spools are identified by hardware RFID identifiers (`tray_uuid` and `tag_uid`), not by the filament preset type.

When AMS data changes:

1. Bambuddy detects the filament via RFID
2. Searches Spoolman for matching spool (by UUID)
3. If found: Updates remaining weight and location
4. If not found: Auto-creates new spool in Spoolman

### Sync Modes

- **Auto** - Syncs whenever AMS data changes (recommended)
- **Manual** - Only syncs when you click the Sync button

### Sync Results

After syncing, you'll see:

- **Synced count** - Number of spools successfully synced
- **Skipped spools** - List of spools that couldn't sync (with reasons)
- **Errors** - Any issues that occurred

Skipped spools show:

- Location (e.g., "AMS A1")
- Color swatch
- Reason (e.g., "Non-Bambu Lab spool")

### Usage Tracking

When prints complete, Bambuddy reports **per-filament** usage to Spoolman:

1. Bambuddy extracts per-filament usage data from the archived 3MF file (slicer estimates)
2. For partial prints, per-layer G-code analysis provides precise consumption up to the exact failure layer
3. On completion, each spool's usage is reported individually to Spoolman (multi-material support)
4. Spoolman updates spool quantities accordingly
5. If no 3MF data is available, AMS remain% delta is used as a fallback

#### Disable AMS Estimated Weight Sync

By default, Bambuddy syncs AMS weight estimates to Spoolman. If you prefer Spoolman's own usage-based tracking:

1. Go to **Settings** > **Spoolman**
2. Enable **Disable AMS Estimated Weight Sync**
3. AMS weight estimates will no longer overwrite Spoolman quantities
4. New spools still use the AMS estimate as their initial weight

!!! tip "When to disable weight sync"
    Use this if you find AMS percentage-based estimates inaccurate. Spoolman's cumulative usage tracking (subtracting grams used per print) is often more precise.

#### Partial Usage for Failed Prints

When a print fails or is cancelled, filament was still consumed. Enable partial usage reporting:

1. Go to **Settings** > **Spoolman**
2. Enable **Report Partial Usage for Failed Prints**
3. Bambuddy calculates filament used via per-layer G-code analysis up to the exact failure layer
4. Falls back to linear scaling (total estimate × progress%) if layer data is unavailable

---

## :material-tray-full: AMS Slot Mapping

### Viewing Mappings

Hover over any AMS slot to see:

- **Vendor** - Bambu Lab or Generic
- **Profile** - Filament type (e.g., "PLA Basic")
- **Color** - Color name and swatch
- **K Factor** - Pressure advance value
- **Fill Level** - Remaining percentage with visual bar
- **Spool ID** - Bambu Lab UUID (when Spoolman enabled)

#### Fill Level for AMS Lite / External Spools

AMS Lite units (e.g., A1 series) have no weight sensor and always report 0% fill level. When a spool is linked to Spoolman and has weight data, Bambuddy uses Spoolman's remaining weight instead:

- **AMS with weight sensor** - Uses AMS percentage directly (no change)
- **AMS Lite (reports 0%)** - Falls back to Spoolman: `(remaining_weight / filament_weight) × 100`
- **External spool** - Shows fill level from Spoolman if linked (otherwise shows "—")

When Spoolman data is used, the hover card displays "(Spoolman)" next to the fill percentage so you can distinguish the data source.

### Opening Linked Spools

For spools already linked to Spoolman:

1. **Hover** over any AMS slot with a linked spool
2. Click **Open in Spoolman** button
3. Opens the spool's page in Spoolman in a new tab
4. Edit spool details directly in Spoolman

!!! info "Quick Access"
    The "Open in Spoolman" button provides one-click access to edit spool details like vendor, cost, notes, or remaining weight directly in Spoolman.

### Manual Linking

Link existing Spoolman spools to your AMS:

1. **Hover** over any AMS slot with a Bambu Lab spool that's not yet linked
2. Click **Link to Spoolman** button
3. Select from list of unlinked Spoolman spools
4. Click **Link** to confirm

!!! info "When to use Link"
    Use this when you already have spools in Spoolman (e.g., from manual entry) and want to connect them to physical spools in your AMS. New Bambu Lab spools are auto-created on sync - no linking needed.

!!! note "Button States"
    - **Open in Spoolman** - Shows when spool is already linked (tag found in Spoolman)
    - **Link to Spoolman** (enabled) - Shows when spool is not linked and there are unlinked spools available
    - **Link to Spoolman** (disabled) - Shows when spool is not linked but no unlinked spools are available in Spoolman

### Spool UUID

Each Bambu Lab spool has a unique identifier (UUID):

- Visible in AMS hover card when Spoolman is enabled
- Click the copy button to copy full UUID
- Used internally to match spools between AMS and Spoolman

### Unlinking

Remove a link:

1. Open Spoolman interface
2. Find the spool
3. Clear the `extra.tag` field

---

## :material-plus-circle: Adding Spools

### From AMS

When unknown filament is detected:

1. Click **Add to Spoolman**
2. Enter spool details:
   - Material type
   - Color
   - Vendor
   - Initial weight
   - Cost per kg
3. Spool is created and linked

### From Spoolman

Add spools directly in Spoolman:

1. Open Spoolman interface
2. Add new spool
3. Spool appears in Bambuddy when loaded

---

## :material-database: Inventory View

View your complete inventory:

### In Bambuddy

- AMS slots with linked spools
- Quick view of what's loaded
- Remaining quantities

### In Spoolman

- Full spool database
- Usage history
- Cost tracking
- Vendor management

---

## :material-chart-line: Usage Statistics

Track filament consumption:

### Per-Print Usage

Each archived print records per-filament data:

- Each spool used (tracked individually for multi-material prints)
- Grams consumed per spool (from G-code extrusion analysis)
- Material type and slot mapping

### Spoolman Integration

Usage syncs to Spoolman:

- Spool quantities update
- History recorded
- Low stock alerts

---

## :material-robot: Automatic Features

### Auto-Sync on Print Complete

After each print:

1. Calculate per-filament usage from G-code layer data
2. Report each spool's consumption individually to Spoolman
3. Spoolman updates spool quantities and records usage history
4. For multi-material prints, each filament is tracked separately

### Auto-Detect on AMS Change

When AMS filament changes:

1. Detect new configuration
2. Match with Spoolman
3. Update slot mappings

### Auto-Clear Location on Removal

When spools are removed from AMS:

1. Bambuddy detects the empty slot
2. Finds spools with matching location (e.g., "Workshop X1C - AMS A Slot 1")
3. Clears the location field in Spoolman
4. Spool is now available for other printers

!!! info "Location Format"
    Spoolman locations use the format: `Printer Name - AMS X Slot Y`

---

## :material-alert: Low Stock Alerts

Get notified when spools run low:

### In Spoolman

Configure low stock threshold:

1. Set minimum quantity per spool
2. Spoolman alerts when below

### In Bambuddy

Notifications for low filament:

- Enable **Low Filament** event
- Get notified when AMS spool is low

[:material-arrow-right: Notification setup](notifications.md)

---

## :material-cog: Advanced Configuration

### Multiple Printers

Each printer's AMS syncs independently:

- Different spools per printer
- Separate usage tracking
- Unified inventory in Spoolman

### Bambu Lab Filaments

Bambu Lab filaments include RFID data:

- Material type auto-detected
- Color recognized
- Can match or create in Spoolman

---

## :material-help-circle: Troubleshooting

### Connection Failed

1. Verify Spoolman URL is correct
2. Check network connectivity
3. Ensure Spoolman is running
4. Check firewall rules

### Sync Not Working

1. Verify connection is configured
2. Check Spoolman logs
3. Restart Bambuddy if needed
4. Manually trigger sync

### Wrong Spool Linked

1. Unlink the incorrect spool
2. Manually link correct spool
3. Check RFID data matches

---

## :material-lightbulb: Tips

!!! tip "Initial Setup"
    Add your existing spools to Spoolman first, then configure Bambuddy integration.

!!! tip "Consistent Naming"
    Use consistent naming in Spoolman for easier matching.

!!! tip "Track Everything"
    Add all filaments to Spoolman, even partials, for accurate inventory.

!!! tip "Regular Check"
    Periodically verify AMS mappings match physical reality.

!!! tip "Cost Tracking"
    Enter costs in Spoolman for complete print cost calculations.
