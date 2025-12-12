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

### AMS to Spoolman

When filament is loaded in your AMS:

1. Bambuddy detects the filament
2. Matches with Spoolman inventory
3. Links the spool to the AMS slot

### Usage Tracking

When prints complete:

1. Bambuddy reports filament used
2. Spoolman updates spool quantity
3. Inventory stays accurate

### Unknown Filaments

When AMS has filament not in Spoolman:

1. Bambuddy detects unknown spool
2. Option to **Add to Spoolman**
3. Creates new spool entry

---

## :material-tray-full: AMS Slot Mapping

### Viewing Mappings

Each AMS slot shows:

- Linked Spoolman spool (if any)
- Material type
- Color
- Remaining quantity

### Manual Linking

If auto-detection fails:

1. Click the AMS slot
2. Select **Link to Spoolman**
3. Choose from available spools
4. Confirm linking

### Unlinking

Remove a link:

1. Click the linked AMS slot
2. Select **Unlink**
3. Slot becomes unlinked

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

Each archived print records:

- Spool used
- Grams consumed
- Material type

### Spoolman Integration

Usage syncs to Spoolman:

- Spool quantities update
- History recorded
- Low stock alerts

---

## :material-robot: Automatic Features

### Auto-Sync on Print Complete

After each print:

1. Calculate filament used
2. Update Spoolman quantity
3. Record usage in history

### Auto-Detect on AMS Change

When AMS filament changes:

1. Detect new configuration
2. Match with Spoolman
3. Update slot mappings

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
