---
title: Spoolman Integration
description: Sync filament inventory with Spoolman
---

# Spoolman Integration

Sync your AMS filament with [Spoolman](https://github.com/Donkie/Spoolman) for complete spool tracking and inventory management.

---

# Spoolman Integration — Native Inventory UI

Bambuddy ships a full native inventory UI for Spoolman-managed spools.
When Spoolman is enabled, the **Inventory** tab switches from the embedded
Spoolman iframe to a built-in interface that shares the exact same design
as the local inventory — filtering, editing, NFC writes, and AMS
deep-links all work identically regardless of which backend is active.

---

## :material-database-sync-outline: What is Spoolman?

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

1. Go to **Settings** > **Filament**
2. The **Spoolman** card is the first card on the page
3. Toggle **Enable Spoolman** on, then enter:

| Field | Description |
|-------|-------------|
| **URL** | Spoolman server URL (e.g., `http://192.168.1.50:7912`) |
| **API Key** | If Spoolman requires authentication |

4. Click **Test Connection**
5. Click **Save**

!!! note "Switching modes keeps your slot assignments"
    Each mode stores AMS slot assignments in its own place, and switching
    between them changes which one Bambuddy reads — it does not delete either.
    Turning Spoolman on to see what it does, then turning it back off, returns
    you to the built-in assignments you had.

    Only the assignments for the mode you are currently in are used, so after a
    switch you will need to link spools again in the new mode. Switching back
    restores what that mode had.

---

## :material-sync: Sync Features

### How Sync Works

!!! note "Bambu Lab Spools Only"
    Only official Bambu Lab spools with RFID are synced automatically. Third-party, refilled, or SpoolEase spools are skipped. Bambu Lab spools are identified by hardware RFID identifiers (`tray_uuid` and `tag_uid`), not by the filament preset type.

When AMS data changes:

1. Bambuddy detects the filament via RFID
2. Searches Spoolman for matching spool (by UUID)
3. If found: Updates remaining weight and location
4. If not found and **Auto-add unknown RFID spools** is on (default): Auto-creates new spool in Spoolman
5. If not found and the toggle is off: Bambuddy raises a confirmation modal in the running web UI instead — you decide whether to add it

### Auto-add unknown RFID spools

The toggle lives under **Settings → Filament → Filament Tracking** above the Spoolman card and applies to both Built-in Inventory and Spoolman modes. Default is **on** (legacy behaviour). Turn it **off** if you prefer to pre-register new spools in Spoolman before loading them, to avoid the matcher creating a duplicate when the AMS reads the tag.

With the toggle off:

- Loading an unknown spool no longer writes a Spoolman row silently
- A modal opens in the Bambuddy UI showing the printer / AMS slot / material / colour
- **Add to Inventory** creates the spool in Spoolman and binds it to the AMS slot in a single call
- Manual `Sync AMS` actions report the slot as skipped with the reason "Auto-add disabled; add to inventory manually"
- The modal does not re-pop on every MQTT push — only when you physically remove and re-insert a spool, or load a different unknown spool

### Sync Modes

- **Auto** - Syncs whenever AMS data changes (recommended)
- **Manual** - Only syncs when you click the Sync button

### Bulk actions

Spoolman-mode users get the same bulk actions documented in the Inventory page — select multiple spools in the Inventory table to bulk-edit fields, archive / restore, delete, or reset the "Total Consumed" counter in a single call. See [Inventory → Bulk actions](inventory.md#bulk-actions) for the full action set. The bulk-edit endpoint loops your selection through the same per-spool update path, so Spoolman's filament-linking, vendor resolution, and extra-dict behaviour are identical to single-spool edits.

If Spoolman is unreachable mid-batch, the backend collects per-spool errors and returns them alongside the success count. The frontend translates that into a partial-success warning toast (or a red "all failed" toast that preserves your selection so you can retry once Spoolman is back online) — bulk actions never silently drop rows.

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

#### How Bambuddy decides which spool a slot used

A sliced file numbers its filaments 1, 2, 3, 4. Which AMS tray each of those
numbers actually came from is a separate decision, made when the job is sent,
and it is frequently not left-to-right — you can map filament 1 to slot 3 in the
Print dialog of any slicer. Bambuddy works that mapping out in this order:

1. **The mapping Bambuddy sent**, for prints you start from Bambuddy itself —
   the Print button, the queue, a reprint. Exact, because Bambuddy chose it.
2. **The mapping the slicer sent**, captured from the print command when
   Bambu Studio or OrcaSlicer dispatched the job over your local network.
3. **The printer's own report.** Most models publish the running job's
   slot-to-tray assignment in their status. This is what covers a job sent from
   Bambu Studio through the Bambu cloud, where the command never passes through
   your network for Bambuddy to see.
4. **Colour matching**, for the models that publish no such report — the A1,
   A1 Mini, P1S and P2S. The sliced colour of each filament is matched against
   the loaded trays, and is only used when every filament matches exactly one.
5. **Position**, as a last resort: filament 1 to the first loaded tray, filament
   2 to the second, and so on.

Only the last of these can be wrong in a way you would notice, and it is also
the only one that assumes your AMS is loaded in slicer order. If a print
consistently deducts from the wrong spool, search the log for `slot_to_tray` —
the line names the mapping and which of the five produced it.

!!! note "The archive's filament changes when the print finishes"
    While a print runs, its archive shows the filaments it was *sliced* for.
    On completion, Bambuddy replaces them with the material and colour of the
    Spoolman spools it actually charged, so the archive reflects your curated
    inventory rather than the slicer's own values. A print whose filament
    changes colour the moment it finishes is telling you the mapping above
    resolved to a spool you did not expect.

#### Disable AMS Estimated Weight Sync

By default, Bambuddy syncs AMS weight estimates to Spoolman. If you prefer Spoolman's own usage-based tracking:

1. Go to **Settings** > **Filament** and open the **Spoolman** card
2. Enable **Disable AMS Estimated Weight Sync**
3. AMS weight estimates will no longer overwrite Spoolman quantities
4. New spools still use the AMS estimate as their initial weight

!!! tip "When to disable weight sync"
    Use this if you find AMS percentage-based estimates inaccurate. Spoolman's cumulative usage tracking (subtracting grams used per print) is often more precise.

#### Partial Usage for Failed Prints

When a print fails or is cancelled, filament was still consumed. Enable partial usage reporting:

1. Go to **Settings** > **Filament** and open the **Spoolman** card
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

### Custom fields Bambuddy registers

Spoolman rejects any write naming an extra field it doesn't know about, so Bambuddy registers the four it uses the first time it needs them:

| Key | Holds |
| --- | --- |
| `tag` | The spool's Bambu RFID UUID &mdash; what links an AMS tray to a Spoolman spool |
| `bambu_slicer_filament` | The Bambu filament preset ID (e.g. `GFA00`) |
| `bambu_slicer_filament_name` | The slicer preset's display name |
| `bambu_color_name` | The colour's human-readable name |

**You can rename these in Spoolman's own field editor, or give them a default, and Bambuddy will leave your changes alone.** It matches on the field's `key`, which never changes, rather than on the display name. Deleting one, though, means Bambuddy re-creates it on the next sync &mdash; it needs the field to exist in order to write to it.

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

## :material-currency-usd: Print cost

When a print finishes, each filament slot is charged to the spool that fed it
and priced from that spool, then the per-slot costs are added up. A
multi-material print is billed at each slot's own rate rather than at one rate
for the whole job.

The price comes from Spoolman, in this order:

1. **The spool's own `Price`**, if set. Use this when a particular purchase cost
   something other than the catalogue figure.
2. **The filament's `Price`**, otherwise.

The rate per gram is that price divided by the filament's **`Weight`** — the net
filament weight, not including the spool core — so a 750 g roll is priced as a
750 g roll.

!!! note "When a price is missing"
    Grams that no spool could price are charged at **Settings → Default
    Filament Cost**, which is a per-kilogram rate. That covers a spool with no
    price entered, a tray with no linked spool, and any filament the sliced file
    did not attribute to a slot. If nothing in the print could be priced from
    Spoolman at all, the cost recorded when the print was archived is left
    as it is.

!!! note "Reprints"
    The archive keeps the **first** run's cost, so a short failed reprint does
    not overwrite the figure from a successful one. Per-run costs are recorded
    separately in the print log.

    Recalculating costs (Archives → Recalculate Costs) does not overwrite a cost
    that came from Spoolman: the spool-to-slot resolution only exists while the
    print is being completed, so there is nothing better to rebuild it from
    afterwards.

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
    Enter costs in Spoolman for complete print cost calculations. See
    [Print cost](#print-cost) for how a price is chosen.
