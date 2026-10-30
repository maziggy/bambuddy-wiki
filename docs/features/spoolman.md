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

**Sync** and **Sync all** unassign a slot that reads empty, except while its printer is printing (a runout reads empty too) and when the AMS reports a spool in it anyway (a spool without filament data, for example).

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
2. For partial prints, the per-layer G-code gives each filament's share fed up to the failure layer, and that share of the slicer estimate is reported
3. On completion, each spool's usage is reported individually to Spoolman (multi-material support)
4. Spoolman updates spool quantities accordingly
5. If no 3MF data is available, AMS remain% delta is used as a fallback

#### When usage cannot be tracked at all

Both routes above can be unavailable at once, and the result is a spool whose
remaining amount never moves:

- **There is no 3MF to read.** A file sent to an H2-series or P2S printer from
  Bambu Studio is kept on the printer's internal eMMC, which FTPS does not
  serve, so the print archives without one. Bambuddy probes external storage for
  a copy by name before giving up, and often finds it, but not always.
- **And the slot reports no remaining percentage.** The delta fallback needs a
  reading at the *start* of the print. A spool without an RFID tag — including
  anything in an AMS HT — reports `-1` until somebody sets a remaining amount
  on the printer, so there is no starting figure to subtract from.

Bambuddy logs the slot it could not charge rather than passing over it in
silence. To get tracking back, either start the print through Bambuddy or
OrcaSlicer (both upload to external storage, so the 3MF is readable), or set a
remaining amount for the slot on the printer's own screen so the AMS begins
reporting a percentage for it.

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

#### A spool replaced during a print

When a spool runs out and you load a new one in the **same** slot (an
external spool, or an AMS slot without a backup), assign the new spool to the
slot while the print is still running. Bambuddy notes the layer the printer was
on, and when the print ends it splits the filament between the two spools: the
old one is charged up to that layer, the new one from there, weighed by the
filament the G-code feeds in each part. A slot that had no spool when the print
started is charged to the spool you assign, for the whole print.

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
- **AMS Lite (reports 0%)** - Falls back to Spoolman: `(remaining weight / spool size) × 100`, with the spool size taken as described in [Spool Size (Label Weight)](#spool-size-label-weight)
- **External spool** - Shows fill level from Spoolman if linked (otherwise shows "—")

When Spoolman data is used, the hover card displays "(Spoolman)" next to the fill percentage so you can distinguish the data source.

#### Spool Size (Label Weight)

Spoolman stores how much filament a full spool holds in two places: the spool's own **Initial Weight**, and the filament's **Weight** as the catalogue value. One filament can have spools of different sizes, so Bambuddy uses the spool's **Initial Weight** and falls back to the filament's **Weight** only when the spool has none.

This is the **Label Weight** in the spool form. It is used for the fill level, for **Sync Weights from AMS** (the AMS percentage is a percentage of this spool), and for print cost. Changing it in the spool form changes only that spool; the filament and any other spools of it are left alone.

The **Cost per kg** in the spool form is stored in Spoolman as the spool's **Price**, which Spoolman treats as the price of the whole spool. Bambuddy converts between the two using the spool's size, so 25 per kg on a 250 g spool is stored as a price of 6.25.

#### Empty Spool Weight (Tare)

When you weigh a spool in Bambuddy or on a SpoolBuddy scale, the remaining weight is the scale reading minus the empty spool weight. Bambuddy takes that weight from Spoolman in the same order Spoolman does:

1. The spool's own **Spool Weight**
2. The filament's **Spool Weight**
3. The vendor's **Empty Spool Weight**
4. 250 g, if none of the three is set

The **Empty Spool Weight** shown in the spool form comes from the same order. Setting it once on the vendor covers every spool of that vendor that has no weight of its own.

### Opening Linked Spools

For spools already linked to Spoolman:

1. **Hover** over any AMS slot with a linked spool
2. Click **Open in Spoolman** button
3. Opens the spool's page in Spoolman in a new tab
4. Edit spool details directly in Spoolman

!!! info "Quick Access"
    The "Open in Spoolman" button provides one-click access to edit spool details like vendor, cost, notes, or remaining weight directly in Spoolman.

### Manual Linking

To put an existing Spoolman spool in an AMS slot from the printer card, hover over the slot and click **Assign Spool**. That assigns the spool to the slot; it does not write the spool's tag into Spoolman.

To link the spool's RFID tag itself (so the spool is recognised by its tag wherever it is loaded), use SpoolBuddy's [AMS page](../spoolbuddy/usage.md):

1. Tap the slot with the Bambu Lab spool that is not yet linked
2. Tap **Assign Spool** (*Link a Spoolman spool to this slot*)
3. Pick the Spoolman spool from the list of spools without a tag

!!! info "When to use Link"
    Use this when you already have spools in Spoolman (e.g., from manual entry) and want to connect them to physical spools in your AMS. New Bambu Lab spools are auto-created on sync - no linking needed.

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

On Spoolman 0.27 or later, also remove the spool's tags in Spoolman's own tag list, or the spool is still found by them (see [Native tags](#native-tags-spoolman-027)). Unlinking in Bambuddy, or **Clear RFID Tag** on the spool, removes both for you.

### Custom fields Bambuddy registers

Spoolman rejects any write naming an extra field it doesn't know about, so Bambuddy registers the four it uses the first time it needs them:

| Key | Holds |
| --- | --- |
| `tag` | The spool's Bambu RFID UUID &mdash; what links an AMS tray to a Spoolman spool |
| `bambu_slicer_filament` | The Bambu filament preset ID (e.g. `GFA00`) |
| `bambu_slicer_filament_name` | The slicer preset's display name |
| `bambu_color_name` | The colour's human-readable name |

**You can rename these in Spoolman's own field editor, or give them a default, and Bambuddy will leave your changes alone.** It matches on the field's `key`, which never changes, rather than on the display name. Deleting one, though, means Bambuddy re-creates it on the next sync &mdash; it needs the field to exist in order to write to it.

On Spoolman 0.27 or later, the identifiers in `tag` are also kept as native tags (next section). `tag` keeps being written all the same, so readers that only know `extra.tag` and a downgrade to an older Spoolman lose nothing.

### :material-nfc-variant: Native tags (Spoolman 0.27+) { #native-tags-spoolman-027 }

Spoolman 0.27 links NFC tags to spools itself: a spool can carry several tags, a tag belongs to exactly one spool, and Spoolman can say which spool a tag belongs to without anyone loading the whole inventory. When Bambuddy finds a Spoolman that supports this, it uses those tags next to `extra.tag`. On an older Spoolman nothing changes.

What that gives you:

- **Every side of a Bambu spool finds it.** A Bambu spool has a tray UUID and two RFID chips, one per side. `extra.tag` holds one identifier; Spoolman's tag list holds all three.
- **Tags linked elsewhere count.** A tag you link in Spoolman, or with another reader on the same Spoolman, is found by Bambuddy and SpoolBuddy too, and the other way round.
- **Faster scans.** A SpoolBuddy scan asks Spoolman for the one tag instead of loading every spool.

What Bambuddy does with them:

| When | What happens |
|------|--------------|
| SpoolBuddy scans a tag | The tray UUID is looked up first, then the chip UID. Each is first asked of Spoolman with one query, and `extra.tag` is searched as before only if that finds nothing. The spool that is found gets what the scan read added to its tags. |
| The AMS reads a spool | Same as a scan: the spool collects its tray UUID and the UID of the chip facing the reader. Turning a spool around adds the other chip. |
| You link a tag | The tag is added to the spool's tags; tags it already had stay. |
| You unlink, or use **Clear RFID Tag** | The spool's tags are removed together with `extra.tag`. |
| SpoolBuddy writes a tag | The written tag is linked to the spool and taken from any other spool that held it, since it was just rewritten for this one. |

!!! note "Chip UIDs from the AMS"
    The AMS reports a Bambu chip's 4-byte UID padded to 8 bytes: a chip any reader sees as `D3E68F32` arrives as `D3E68F3200000100`. Bambuddy stores the chip's own 4 bytes, so the tag matches what SpoolBuddy and other readers read. **Copy tags** does the same with a padded UID it finds in `extra.tag`.

#### Copying your existing tags

Spools linked before you upgraded Spoolman have their tag only in `extra.tag`. They are found as before, and each one moves over by itself the first time it is scanned or read by the AMS. To copy them all at once (archived spools are left out):

1. Go to **Settings** > **Filament**. With Spoolman connected and on 0.27 or later, the **Spoolman native tags** section appears.
2. Click **Check tags**. This is a dry run and changes nothing:

    | Line | Meaning |
    |------|---------|
    | **To copy** | Spools whose `extra.tag` is not a native tag yet |
    | **Already in Spoolman** | Spools that already carry it |
    | **AMS slot IDs** | Placeholders Bambuddy uses for a slot without RFID. They name a slot, not a spool, and stay in `extra.tag` only |
    | **Conflicts** | The tag already belongs to another active spool, a filament or a location, listed by spool number |

3. Click **Copy tags** to copy what the check listed under **To copy**.

![Native tags check](../assets/spoolman_native_tags_check.png){ .screenshot .centered }

`extra.tag` is left as it is.

#### When a tag belongs to another spool

Spoolman lets a tag belong to one spool, filament or location only, and that includes archived spools.

- **An archived spool** gives the tag up. It was archived because it is used up or was replaced, so the tag belongs to the active spool now. Bambuddy moves it there, and the log says `Native tag … moved from archived spool …`.
- **Another active spool, a filament or a location** keeps it. Bambuddy does not move it by guessing (the one exception is a tag SpoolBuddy has just written, see above):
    - linking such a tag answers with the spool that holds it, as linking an already used tag always did
    - a scan or AMS read still finds the right spool, but does not add that tag, and logs `Native tag … belongs to …` once. The AMS sync tries again after 10 minutes, not on every AMS update; scanning the spool on a reader tries again at once
    - **Check tags** lists it under **Conflicts**

To settle a conflict, remove the tag from the wrong spool in Spoolman. Scanning the spool adds it where it belongs right away; otherwise the AMS sync adds it within 10 minutes.

For a Bambu spool, the tray UUID decides: both chips of a spool carry it, so a scan finds the spool by its UUID even if one of its chips was linked to another spool by mistake.

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

### Profile defaults

[Profile defaults](inventory.md#profile-defaults-for-new-spools) work the same in Spoolman mode: a spool added from Bambuddy, or by the AMS sync, starts with its product's presets and K profiles. The default's own preset goes into the spool's `bambu_slicer_filament` fields, except on a spool the AMS sync adds from a tag that names its preset, which keeps the tag's. The presets per model and the K profiles stay in Bambuddy, as they do for any Spoolman spool.

A spool added in Spoolman's own interface never passes through Bambuddy, so it gets no default when it is created. **Settings → Filament → Profile defaults → Apply to existing spools** fills it in afterwards.

---

## :material-database: Inventory View

View your complete inventory:

### In Bambuddy

- AMS slots with linked spools
- Quick view of what's loaded
- Remaining quantities
- Per-print usage history of each spool, shown in its spool dialog and used by the stock forecast

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
3. Spoolman updates spool quantities, and Bambuddy records the print in the spool's usage history (Spoolman itself keeps only the totals)
4. For multi-material prints, each filament is tracked separately

!!! warning "Spoolman must be reachable when a print ends"
    A print's usage is sent to Spoolman once, when the print finishes. If Spoolman cannot be reached at that moment (restarting, offline, network down), that print's usage is not sent later: the Bambuddy log records it as "Not reachable for usage reporting", and the spool's weight has to be corrected in Spoolman by hand. Keep Spoolman running while prints are in progress.

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

Bambuddy sends the alert; Spoolman supplies the remaining weight.

1. Assign your Spoolman spools to AMS slots in Bambuddy.
2. Set the low-stock threshold on the **Inventory** page (default 20 %).
3. Enable the **Low Filament** event on a notification provider.

When an assigned spool drops below the threshold you get one notification, such as "X2D: Slot A1 at 15%". It can fire again once the spool goes back above the threshold, for example when you load a fresh one.

!!! note
    In Spoolman mode only the global threshold applies. The per-spool override is a field of Bambuddy's own inventory and has no counterpart in Spoolman.

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

### A Tag Is Not Added to a Spool

On Spoolman 0.27 or later, the log says `Native tag … belongs to …` when a tag is already linked to another active spool, a filament or a location. Remove it from that spool in Spoolman; see [When a tag belongs to another spool](#when-a-tag-belongs-to-another-spool).

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

The rate per gram is that price divided by the net filament weight it was paid
for, not including the spool core. The spool's own price is divided by the
spool's **`Initial Weight`** (or the filament's **`Weight`** if the spool has
none), so a 250 g spool is priced as a 250 g spool. The filament's price is
divided by the filament's **`Weight`**.

!!! note "While the print is running"
    When a print starts, Bambuddy already prices it the same way from the
    sliced file's per-slot estimate and the spools in the mapped trays. The
    archive card shows that figure with a `~` until the print finishes and the
    cost of what was actually used replaces it. A print whose slots can't be
    matched to trays at the start (no mapping from the slicer or the queue, and
    no colour match) keeps the default-rate figure until then.

!!! note "When a price is missing"
    Grams that no spool could price are charged at **Settings → Default
    Filament Cost**, which is a per-kilogram rate. That covers a spool with no
    price entered, a tray with no linked spool, and any filament the sliced file
    did not attribute to a slot. If nothing in the print could be priced from
    Spoolman at all, the cost recorded when the print was archived is left
    as it is.

!!! note "Reprints"
    The archive keeps the cost of one full print: the **first** run's, if it
    completed, else the estimate it carried when that run started. A short failed run, first
    or later, does not overwrite it. Per-run costs, partial ones included, are
    recorded separately in the print log.

    **Recalculate Costs** (in the Statistics dashboard header) prices each
    recorded Spoolman charge at the spool's current price in Spoolman, and the
    archive and its print in the Print Log follow. A charge to a spool gone
    from Spoolman keeps its figure, and a print whose spools have no price
    keeps its cost. A print from before
    Spoolman charges were recorded has none to rebuild its cost from: the
    spool-to-slot resolution only exists while the print is being completed,
    so its cost is not overwritten.

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
