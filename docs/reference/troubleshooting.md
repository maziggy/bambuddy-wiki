---
title: Troubleshooting
description: Common issues and solutions
---

# Troubleshooting

Solutions for common issues with Bambuddy.

---

## :material-printer-3d: Printer Connection Issues

### Run the Connection Diagnostic first

Before working through the steps below, let Bambuddy check for you. The
built-in **Connection Diagnostic** runs the same checks a maintainer would:
port reachability (MQTT 8883, FTPS 990, RTSPS 322), LAN Developer Mode,
Docker network mode, printer/host subnet match, printer credentials, and
the printer-side "Store sent files on external storage" toggle (install [step 4](../getting-started/index.md#step-4-enable-store-sent-files-on-external-storage)).
Each result comes with a plain-language fix.

Open it from any of:

- The **printer card** — a *Run diagnostic* button appears when a printer is offline.
- The **Add Printer** dialog — *Run diagnostic* tests the address and credentials before you save.
- **System page → Connection Diagnostic** — run it for any configured printer on demand.

Most "won't connect" and "won't print" problems are identified here in a few seconds.

---

### Printer Won't Connect

**Symptoms:** Printer shows as disconnected, red indicator

**Solutions:**

1. **Verify Developer Mode is enabled**
   - Check printer: Settings > Network > LAN Only Mode (must be ON)
   - Then enable Developer Mode (appears after LAN Only Mode)
   - Toggle off and on to get a fresh access code

2. **Check IP address**
   - Verify IP in printer network settings
   - Ensure IP hasn't changed (use static IP or DHCP reservation)

3. **Verify access code**
   - Access code changes when Developer Mode is toggled
   - Copy the code exactly (case-sensitive)
   - If the printer actively refused the code, the log says so:
     `MQTT connection refused by the printer: Not authorized`. Run the
     **Connection Diagnostic** — the *Printer credentials* check names the
     cause when the printer told us, and hedges when all Bambuddy knows is
     that there is no session (a rebooting printer, or one already at its
     limit of simultaneous connections, looks the same from outside).

4. **Check network connectivity**
   ```bash
   ping YOUR_PRINTER_IP
   ```

5. **Verify ports are accessible**
   - MQTT: Port 8883
   - FTPS: Port 990

   ```bash
   telnet YOUR_PRINTER_IP 8883
   ```

6. **Check firewall rules**
   - Ensure Bambuddy server can reach printer
   - Check both server and network firewalls

---

### Connection Drops Frequently

**Symptoms:** Printer connects but disconnects intermittently

**Solutions:**

1. **Check WiFi signal strength**
   - View signal strength on printer card
   - Move printer closer to router if weak

2. **Network congestion**
   - Too many devices on network
   - Try a dedicated network/VLAN

3. **Router issues**
   - Restart router
   - Check for firmware updates
   - Disable "smart" features that may interfere

4. **Check Bambuddy logs**
   ```bash
   tail -f logs/bambuddy.log
   ```

5. **Enable FTP retry**
   - Go to Settings > General > FTP Retry
   - Enable retry with 3 attempts and 2 second delay
   - Increase connection timeout (default 30s) for slow WiFi
   - Helps with intermittent connection issues during file transfers

!!! info "Printers that drop off and never come back"
    From 1.2.6 Bambuddy watches for this. A printer that had a working
    connection, has been silent for five minutes, and still answers on its MQTT
    port has its session rebuilt automatically — the log records how long it was
    gone and what the last connection error was. Printers that are switched off
    are left alone, so powering a farm down overnight does not cause reconnect
    churn. On earlier versions a session that died without recovering could stay
    down until something else prompted a reconnect.

---

### A1/A1 Mini FTP Issues

**Symptoms:** File transfers fail with "read operation timed out" on A1 or A1 Mini printers

**Background:**

A1 and A1 Mini printers have different FTP/SSL behavior than X1C/P1S printers. They have issues with SSL encryption on the FTP data channel, causing transfers to hang or timeout waiting for completion responses. Bambuddy v0.1.6+ automatically detects A1/A1 Mini printers and skips SSL on the data channel while keeping the control channel encrypted via implicit FTPS.

**Solutions:**

1. **Update to latest Bambuddy version**
   - Version 0.1.6+ automatically handles A1 SSL compatibility
   - No manual configuration required

2. **Increase FTP timeout for weak WiFi**
   - Go to Settings > General > FTP Retry
   - Increase connection timeout from default 30s to 60-120s
   - A1 printers often have weaker WiFi signal than X1C/P1S

3. **Improve WiFi signal**
   - A1 printers are known to have weak WiFi reception
   - Check signal strength on printer card (-30 to -50 dBm = excellent, -50 to -70 dBm = fair, below -70 dBm = weak)
   - Move printer closer to router or add a WiFi extender

4. **Enable FTP retry**
   - Go to Settings > General > FTP Retry
   - Enable retry with 3-5 attempts and 2-3 second delay
   - Combined with longer timeout, helps recover from transient failures

---

### Wrong Printer Status

**Symptoms:** Status shows idle when printing, or vice versa

**Solutions:**

1. **Wait for sync**
   - Status updates every few seconds
   - Refresh the page

2. **Check MQTT debug**
   - Enable MQTT debug logging
   - Verify messages are being received

3. **Restart connection**
   - Delete printer from Bambuddy
   - Re-add with correct details

### The Printer Accepts Every Job and Starts None of Them

**Symptoms:** The 3MF uploads fine, the queue item goes to *Printing*, and the
printer stays idle. After a few minutes the job fails. Temperature and filament
controls report success and the printer ignores them. Printing the same file
straight from Bambu Studio or OrcaSlicer works.

**Cause:** the printer rejected the command because it could not verify it —
HMS **0500-0500-0001-0007**, *MQTT command verification failed*. Bambu Lab
firmware from `01.08.03.00beta` / `01.08.05.00` authenticates control commands,
and **Developer Mode** is the documented way to turn that off for LAN control.

This is confusing to diagnose because the printer keeps answering *queries*
normally. Status, AMS contents, RFID tags and the camera all keep working, so
the connection looks perfectly healthy — only commands are being dropped.

**Fix:**

1. On the printer: **Settings > Network**, turn on **LAN Only Mode**, then
   enable **Developer Mode** (it appears once LAN Only Mode is on).
2. **Restart the printer.** The setting does not take effect until it reboots.
3. Re-check the access code — it changes when the mode is toggled.
4. Start the job again.

!!! note "Bambuddy 1.2.6 and newer"
    Bambuddy now shows this error on the printer card with the fix attached, and
    fails the queue item immediately rather than re-uploading the file twice
    more. On older versions nothing surfaced it — the queue item failed with a
    message about the SD card, and the Connection Diagnostic reported Developer
    Mode as *passing*, because the probe it uses cannot tell this firmware's
    silent refusal apart from a normal answer.

---

### Printer Looks Like It's Trying to Re-Start the Last Print After Power-On

**Symptoms:** Every time you power the printer on, the touchscreen shows the last printed file's name, and it looks as though Bambuddy or the printer is about to start that print again.

**This is the printer's firmware, not Bambuddy.** The Bambu Lab firmware (A1, A1 mini, P1, X1, H2D) keeps the **last** `gcode_file` and `subtask_name` in its MQTT status payload after a print finishes. Those fields persist across power cycles and are echoed in every status push, even when the printer is sitting idle. The touchscreen displays them too.

Bambuddy only acts on a real **state transition** (`IDLE → PREPARE / RUNNING`). It never sends a print command just because the last filename is still visible. You can confirm this in your support bundle log — a queued auto-start always shows up as a `print_scheduler` "Queue check" → "Starting queue item N" → `PRINT COMMAND` line. If you don't see those lines around the power-on time, Bambuddy isn't starting anything.

**What to check on the printer:**

1. **Touchscreen prompt** — if a "Resume last print?" or "Start print?" dialog is open, dismiss it on the printer itself.
2. **HMS errors** — codes like `0500_C010` (MicroSD card read/write exception) can leave the printer in a state where the touchscreen still shows a print as "ready". Reseat or replace the MicroSD card; if the error persists, contact Bambu Lab Support.
3. **Saved task on the printer** — the printer may keep the file in its own queue/history. Clear it from the printer's storage UI directly.

If you have **already** deleted the queue item in Bambuddy and removed the file from the printer's backup/cache folder and it still happens, the issue is with the printer firmware or hardware, not with Bambuddy.

---

## :material-sd: SD Card Issues

### Prints Won't Start / File Transfer Fails

**Symptoms:** Cannot start prints from Bambuddy, file transfers fail, or "No SD card" errors

**Solutions:**

1. **Ensure SD card is inserted**
   - An SD card is **required** for Bambuddy to work with your printer
   - Check that the SD card is properly inserted in your printer
   - The printer should recognize the SD card in its file browser

2. **Check SD card health**
   - Try a different SD card if transfers fail frequently
   - Format the SD card using the printer's built-in format function
   - Use a high-quality SD card (Class 10 or better recommended)

3. **SD card full**
   - Check available space on the SD card
   - Delete old files from the printer's storage
   - Use Bambuddy's File Manager to clean up files

!!! warning "SD Card Required"
    Bambuddy requires an SD card in your printer for:

    - Starting prints from Bambuddy or the print queue
    - Transferring files to/from the printer
    - Archiving completed prints (downloading 3MF files)
    - Firmware updates for LAN-only printers

---

## :material-archive: Archiving Issues

### Prints Not Being Archived

**Symptoms:** Prints complete but don't appear in archives

**Solutions:**

1. **Check SD card is inserted**
   - An SD card is required for archiving to work
   - Verify the SD card is properly inserted and recognized

2. **Check printer connection**
   - Must be connected during and after print
   - Verify green connection indicator

3. **Check FTP access**
   - Port 990 must be accessible
   - Try downloading a file manually

4. **Check disk space**
   - Ensure enough space for 3MF files
   - Clear old files if needed

5. **Enable FTP retry for weak WiFi**
   - P1S, X1C, A1, and other printers often have weak WiFi
   - Go to Settings > General > FTP Retry
   - Enable retry with 3-5 attempts and 2-3 second delay
   - Increase connection timeout to 60-120s for A1/A1 Mini
   - This helps when FTP transfers fail intermittently

6. **Check logs for errors**
   ```bash
   grep -i "archive\|error" logs/bambuddy.log
   ```

---

### Archive Card Has Only a Name

**Symptoms:** The print is archived with its name and timing, but there is no thumbnail, no filament total, no layer count, and the Reprint button is greyed out. Other printers on the same Bambuddy install archive normally.

**Background:**

Bambuddy reads a print's 3MF and its cover over FTPS on port 990. On every Bambu model that port serves **external storage only** — the SD card or USB stick. It is not a view of the printer's whole filesystem.

On H2-series and P2S, **Bambu Studio** puts the sliced file on the printer's **internal storage** instead, uploading over a separate service on port 6000. When it does, there is no file on FTPS to fetch, at any path.

What decides it is the slicer, not the printer and not your settings. Measured on an H2C, an H2D and an X1C — same Bambu Studio, same model, one minute apart, all three reporting "Store sent files on external storage" as **on**:

| Printer | Where Bambu Studio put it | Archive |
| --- | --- | --- |
| H2C | internal storage | name and timing only |
| H2D | internal storage | name and timing only |
| X1C | external storage | complete |

The same H2C and H2D sliced in **OrcaSlicer** put the file on the card and archived in full, and turning "Store sent files on external storage" *off* made no difference to that — OrcaSlicer always uploads over FTPS. So the toggle does not control this on H2-series, in either direction.

Bambu Studio's **Send** dialog does offer a storage picker — **Cache** (the printer's internal memory) or **External** — and choosing External puts the file on the card where Bambuddy can read it. Its **Print** button offers no such choice (tested on an H2D) and goes to Cache every time. So staying in Bambu Studio means sending first and starting the print as a second step. [BambuStudio#10481](https://github.com/bambulab/BambuStudio/issues/10481) tracks the default.

You can tell which storage a print used from the log:

```bash
grep '"url"' logs/bambuddy.log
```

`"url": "ftp://<name>"` means the card, and the archive will be complete. `"url": "brtc://emmc/<name>"` means internal storage, and it will not be.

**Solutions:**

All the routes below need a card or stick in the printer. On X1 and P1 series, where Bambu Studio already uses external storage, only the first point applies.

1. **Insert a card or stick**
      - Check Settings > Printers > Connection Diagnostic: `Store sent files on external storage` reports `no_media` when the slot is empty
      - The setting itself lives in the printer's own Print Settings on current firmware, and in Bambu Studio / OrcaSlicer's Device tab on older versions. It is worth having on, but on H2-series and P2S it will not change where Bambu Studio sends the file

2. **Start the print from Bambuddy instead of the slicer**
      - Bambuddy uploads over FTPS itself, so the file lands on external storage and the archive is complete, on every model

3. **Slice in OrcaSlicer**
      - OrcaSlicer always uploads over FTPS, so its prints archive in full on H2-series too
      - It will refuse to send with an empty slot — "storage needs to be inserted before printing via lan" — rather than silently falling back to internal storage

4. **Send from Bambu Studio with External picked, then start the print**
      - The picker is in the **Send** dialog only; **Print** always uses Cache
      - Two steps rather than one, but it keeps you in Bambu Studio

5. **Accept the partial archive**
      - Name, timing, status, and the finish photo still work; only the slicer-derived data is missing

!!! info "Files already on internal storage cannot be recovered"

    Bambuddy can list what is on a printer's internal storage over the port-6000 tunnel, but the firmware refuses to serve those files back, so a print already sent there by Bambu Studio cannot be archived after the fact. [Issue #2762](https://github.com/maziggy/bambuddy/issues/2762) tracks uploading over that tunnel, which would remove the card requirement from route 2 — it would not change what Bambu Studio does.

---

### Wrong Timelapse Attached to Archive

**Symptoms:** After a print, the archive shows a timelapse from a previous print

**Background:**

In LAN-only mode (no cloud connection), Bambu printers don't sync their clock via NTP, so file modification times are unreliable. Bambuddy v0.1.9b+ uses a snapshot-diff approach instead of relying on mtime: it records which timelapse files exist before the print completes, then detects the new file that appears after encoding. If no new file is detected after retries, it falls back to matching the print name against filenames.

**Solutions:**

1. **Update to latest Bambuddy version**
   - Version 0.1.9b+ fixes the timelapse detection logic
   - No manual configuration required

2. **Manual scan if auto-detection missed**
   - Open the archive
   - Click **Scan for Timelapse** button
   - This uses print-name matching and timestamp proximity

3. **Check printer storage**
   - Ensure the SD card has enough free space
   - Old timelapse files may fill up the timelapse directory

---

### Calibration Prints Being Archived

**Symptoms:** Calibration prints (flow rate, vibration compensation, bed leveling) appear in archives

**Background:**

Bambu printers announce their own calibration routines over MQTT through the
same print-start event a real print uses, so Bambuddy has to recognise them by
name. Two shapes exist and both are filtered:

| Routine | How the printer reports it | Filtered since |
|---------|---------------------------|----------------|
| Bed levelling, vibration compensation | An internal gcode path under `/usr/`, e.g. `/usr/etc/print/auto_cali_for_user.gcode` | v0.1.9b |
| Auto pressure-advance (K profile) line, run before a print when flow dynamics calibration is on | The subtask name `auto_pa_line_calib_mode`, with no path at all | v1.2.6 |

Filtered runs create no archive and send no print-started or print-completed
notification.

The match is exact, so a file you have deliberately named after a calibration -
`auto_pa_line_calib_mode_v2.3mf`, or your own calibration cube - is still
archived as the print it is.

**Solutions:**

1. **Update to latest Bambuddy version**
   - v1.2.6+ filters both routines above

2. **Delete unwanted calibration archives**
   - Search the archives for `auto_cali` or `auto_pa_line` to find rows created before the upgrade
   - Select and delete any you don't want

---

### Duplicate Archives

**Symptoms:** Same print appears multiple times

**Solutions:**

1. **Enable duplicate detection**
   - Go to Settings > General
   - Enable duplicate detection

2. **Clean up duplicates**
   - Filter archives by name
   - Delete unwanted duplicates

---

### Missing Thumbnails

**Symptoms:** Archive cards show placeholder instead of thumbnail

**Solutions:**

1. **Check 3MF file**
   - Some files don't include thumbnails
   - Re-slice with thumbnail enabled

2. **Regenerate thumbnails**
   - Bambuddy can regenerate from 3MF
   - Check Settings > Maintenance

---

### Wrong Plate Thumbnail on Printer Card During a Multi-Plate Print

**Symptoms:** The printer card shows a thumbnail from a different plate than the one currently printing.

**Cause:** Some firmware versions (e.g. P1S 01.10.00.00) only put the bare `.3mf` filename in the MQTT `gcode_file` field, omitting the `Metadata/plate_N.gcode` path. Without the plate path the cover route used to default to plate 1's thumbnail.

**Resolution:** Fixed in 0.2.4b2. Bambuddy now records the plate it dispatched at publish time and prefers that record over parsing the gcode path. For prints started outside Bambuddy (e.g. Bambu Studio direct), the cover route additionally scans the downloaded 3MF for a unique `Metadata/plate_*.gcode` to identify the active plate. If you still see the wrong thumbnail on 0.2.4b2 or later, ensure the print was dispatched from Bambuddy and capture a support package — the resolved plate is logged at INFO level (`Cover: resolved plate N`).

---

### Cover Thumbnail Stuck Loading During an Active Print

**Symptoms:** Printer card shows a loading state instead of a thumbnail; logs show `FTP connection timed out` to the printer.

**Cause:** Bambu printers run a single-socket FTP server that prioritises the active print. While a print is running, concurrent FTP reads (for the cover thumbnail) sometimes time out — especially on large 3MF files (50 MB+) where the read window is tight. An aborted upload to the printer can also wedge the FTP session until the printer is restarted.

**Resolution:** As of 0.2.4b2, Bambuddy registers the local archive 3MF in the cover-cache at dispatch time, so the cover route reads the file straight from disk rather than refetching it over FTP. This applies to any print dispatched **from Bambuddy** (archive card, file manager, queue). If you see this on a Studio-direct print:

- Wait — the cover route caches successful downloads, so once it makes it through once subsequent loads are instant.
- If FTP repeatedly times out, restart the printer to clear any stuck FTP session from a previous aborted upload.

---

## :material-camera: Camera Issues

### Camera Won't Stream

**Symptoms:** Camera page shows error or black screen

**Solutions:**

1. **Check ffmpeg installation**
   ```bash
   ffmpeg -version
   ```

   Install if missing:
   ```bash
   # Ubuntu/Debian
   sudo apt install ffmpeg

   # macOS
   brew install ffmpeg
   ```

2. **Verify camera is enabled**
   - Check printer settings
   - Camera must be enabled in printer settings

3. **Check connection**
   - Camera requires active printer connection
   - Verify printer shows as connected

4. **Try snapshot mode**
   - Click "Snapshot" instead of "Live"
   - May work when streaming doesn't

5. **Check for a second connection to the camera**

    Most Bambu printers accept **exactly one** camera connection at a time —
    the chamber-image socket on port 6000 (A1 / P1), or the RTSP socket on
    port 322 (X1 / H2 / P2). Bambuddy therefore opens one connection per
    printer and fans it out to every viewer (camera page, cam-wall tile,
    embedded viewer, popup) rather than one per viewer.

    If a second connection ever overlaps, the printer keeps feeding the first
    one and the newcomer receives nothing — the page goes black and stays
    black until the printer's TCP keepalive reaps the orphan, which takes
    **around 20 minutes**. A black stream that recovers on its own after
    roughly that long is the signature.

    While the screen is black, check how many connections the printer has:

    ```bash
    # A1 / P1 (chamber image)
    ss -tn | grep :6000

    # X1 / H2 / P2 (RTSP)
    ss -tn | grep :322
    ```

    More than one line means something is holding an extra socket open —
    another tool talking to the same printer (Bambu Studio, Home Assistant,
    a second Bambuddy instance) is the usual cause. Close it and reload the
    camera page.

---

### Stream Freezes

**Symptoms:** Video starts but freezes

**Solutions:**

1. **Network bandwidth**
   - Lower FPS setting
   - Check network congestion

2. **Check ffmpeg processes**
   ```bash
   ps aux | grep ffmpeg
   ```

   Kill orphaned processes:
   ```bash
   killall ffmpeg
   ```

3. **Refresh the stream**
   - Click refresh button
   - Close and reopen camera page

---

## :material-bell: Notification Issues

### Notifications Not Sending

**Symptoms:** No notifications received

**Solutions:**

1. **Test the provider**
   - Go to Settings > Notifications
   - Click "Send Test"
   - Check for errors

2. **Verify configuration**
   - Double-check API keys/tokens
   - Verify phone numbers include country code

3. **Check quiet hours**
   - Notifications suppressed during quiet hours
   - Verify current time vs quiet hours setting

4. **Check event triggers**
   - Ensure desired events are enabled
   - Check printer filter settings
   - **Obico spaghetti detections**: enable the dedicated **AI Failure Detection** toggle, not "Printer Error" — AI alerts moved to their own event (see [Failure Detection](../features/failure-detection.md))

---

### Notification Variables Empty

**Symptoms:** Messages show "Unknown" or blank for variables

**Solutions:**

1. **Update to latest version**
   - Variable handling improved in recent versions

2. **Check print data**
   - Some variables require specific print data
   - May show "Unknown" for incomplete prints

---

## :material-database: Database Issues

### Database Errors

**Symptoms:** 500 errors, data not saving

**Solutions:**

1. **Check disk space**
   ```bash
   df -h
   ```

2. **Verify database integrity**
   ```bash
   sqlite3 bambuddy.db "PRAGMA integrity_check;"
   ```

3. **Restore from backup**
   - If corrupted, restore from backup
   - See Backup & Restore guide

### "Database is locked" Errors

**Symptoms:** Intermittent "database is locked" errors, especially with multiple printers

**Background:**

Bambuddy v0.2.0b+ uses SQLite WAL (Write-Ahead Logging) mode, which significantly reduces lock contention by allowing simultaneous reads during writes. WAL mode is automatically enabled on startup along with a 5-second busy timeout.

**Solutions:**

1. **Update to latest Bambuddy version**
   - WAL mode is automatically enabled on startup — no configuration needed

2. **Check for WAL files**
   - SQLite WAL mode creates `bambuddy.db-wal` and `bambuddy.db-shm` files next to the database
   - These are normal runtime files and should not be deleted while Bambuddy is running
   - They are cleaned up automatically when the database is closed properly

3. **Docker volume mounts**
   - Ensure the data directory volume has sufficient write permissions
   - WAL files must be on the same filesystem as the database

---

### Search Not Working

**Symptoms:** Search returns no results or wrong results

**Solutions:**

1. **Rebuild FTS index**
   - Go to Settings > System Info
   - Click "Rebuild Search Index"

2. **Check search syntax**
   - Use quotes for exact phrases
   - Check for typos

---

## :material-power-plug: Smart Plug Issues

### Tasmota Plug Not Responding

**Symptoms:** Can't control Tasmota smart plug

**Solutions:**

1. **Verify IP address**
   - Check plug still has same IP
   - Use static IP or DHCP reservation

2. **Test Tasmota interface**
   - Open `http://PLUG_IP` in browser
   - Verify Tasmota web interface loads

3. **Check network access**
   ```bash
   curl "http://PLUG_IP/cm?cmnd=Status%200"
   ```

---

### REST/Webhook Plug Not Responding

**Symptoms:** Can't control REST/Webhook smart plug

**Solutions:**

1. **Test the URL with curl**
   ```bash
   curl -X POST "http://your-device:8080/api/endpoint" -d "ON"
   ```

2. **Verify network access**
   - Ensure the target service is reachable from Bambuddy's network

3. **Check headers**
   - Custom headers must be valid JSON (e.g., `{"Authorization": "Bearer token"}`)

4. **Check HTTP method**
   - Ensure the method (GET/POST/PUT/PATCH) matches what the target API expects

5. **Check the target service's logs**
   - Look for authentication errors or invalid request formats

---

### Auto Power-Off Not Working

**Symptoms:** Printer doesn't turn off after print

**Solutions:**

1. **Check feature is enabled**
   - Settings > Smart Plugs > Auto Power Off

2. **Verify cooldown settings**
   - Check temperature threshold
   - Check cooldown time

3. **Check for queued prints**
   - Won't power off if more prints queued

---

## :material-printer-3d-nozzle: Virtual Printer Issues

### Slicer shows no AMS / no filament slots / no temperatures

If your VP **has a target printer configured**, the slicer should show live AMS slots, FTS / dual-extruder routing, k-profiles, temperatures, and the camera stream — same view as a direct slicer connection. If the panel is empty, check:

1. **Target printer set?** Settings → Virtual Printer → check the VP has a target printer selected. The mirror only runs when this is set.
2. **Target printer connected?** Bambuddy must be online with the real printer (Settings → Printers shows green status). The bridge has nothing to mirror if Bambuddy itself isn't talking to the printer.
3. **Restart Bambuddy** after setting the target — the bridge attaches at VP startup.

If your VP has **no target printer set**, the slicer sees synthetic stub state by design — the VP is a pure file receiver. Set filaments manually, hit Send, then dispatch to a real printer from Bambuddy's UI later. To enable live mirror, set a target printer in Settings → Virtual Printer.

See the [Virtual Printer guide](../features/virtual-printer.md#live-target-printer-mirror) for the full explanation.

### Slicer camera shows "LAN connection failed"

Camera streaming requires the VP's **access code to match the target printer's** LAN access code. The slicer authenticates the camera RTSPS stream with the access code stored in its profile, and that auth happens against the real printer. Fix:

1. Settings → Virtual Printer → set the VP's access code equal to the target printer's LAN access code (Settings → Printers shows the printer's code)
2. Re-add the VP in Bambu Studio / Orca Slicer so it picks up the new code

MQTT and FTP work either way — only the camera path needs the access codes to match.

### Slicer can't find the Virtual Printer

See [Slicer Can't Find or Connect to Virtual Printer](../features/virtual-printer.md#slicer-cant-find-or-connect-to-virtual-printer) in the Virtual Printer guide — covers SSDP, bind ports (3000/3002), TLS certificate, and cross-subnet setups.

### Print uploads fail at ~10% ("Failed to send") in Docker bridge mode

**Symptoms:** The VP is added manually by the Docker host's IP. Detection, MQTT, status, and camera all work, but sending a print job fails around 10% with **"Failed to send"**. A tcpdump shows the bind/detect exchange (port 3002) succeed and then the session close — with no connection ever attempted to FTPS port 990 or the passive-data ports.

**Cause:** This is a fundamental limitation of Docker's **default bridge** (docker0 NAT), not a Bambuddy bug. The upload is the one flow where the printer advertises its own IP to the slicer as the FTP target. Inside a default-bridge container the only address that exists is the NAT IP (e.g. `172.17.0.10`), so that's what gets advertised — and your LAN client has no route to it, so the FTP connection never leaves the machine. The container cannot discover the host's real LAN IP on its own, and `VIRTUAL_PRINTER_PASV_ADDRESS` only overrides the FTP passive-data address, not the advertised identity (and the slicer never reaches the FTP stage to use it).

**Resolution:** Give the Virtual Printer a LAN-routable identity:

- **`network_mode: host`** (recommended, Linux) — the container uses the host's LAN IP directly.
- **macvlan** — the container gets its own real LAN IP, so it stays reachable from your other Docker services (behind Caddy/Authentik, etc.) while also being routable for uploads.

See [Docker (macOS / Windows)](../features/virtual-printer.md#docker-macos-windows) in the Virtual Printer guide for the bridge-mode warning and the host/macvlan setups.

---

### Uploads to the VP arrive corrupt / the real printer can't start the job

If files sent to a Virtual Printer are archived and queued but the physical printer then fails to parse or start them — and the received `.gcode.3mf` is smaller than what the slicer exported (often ending at an exact multiple of 4096 bytes) — your server is likely running under **uvloop**, whose SSL layer can silently drop the tail of an upload on slow storage (e.g. a microSD / eMMC on an SBC).

**Fix:** launch uvicorn with `--loop asyncio`. The Docker image already does this; native installs must add it to the run command / service unit:

```bash
uvicorn backend.app.main:app --host 0.0.0.0 --port 8000 --loop asyncio
```

Re-run the installer or edit your systemd unit / launchd plist / NSSM service to include `--loop asyncio`, then restart Bambuddy. Bambuddy 0.2.5b2+ also validates every received `.3mf` is a complete ZIP before accepting it — a truncated upload is now rejected with an FTP `426` (an immediate send error in the slicer) instead of being archived and forwarded to the printer.

---

## :material-refresh-auto: Print Queue Issues

### Next queued print auto-started without "Clear Plate" confirmation

**Symptoms:** You have Auto Off enabled and a queue of prints. When one print finishes, the smart plug cuts power; the plug immediately re-powers the printer because the next job is queued; the next print then starts without showing the **Clear Plate & Start Next** prompt.

**Background:**

Before Bambuddy v0.2.3b4, the plate-clear gate lived only in memory and was tied to the printer's reported state (`FINISH` / `FAILED`). When the plug re-powered the printer, it booted fresh into `IDLE` with no memory of the previous finish, so the scheduler's idle check saw a normal idle printer and dispatched the next job immediately — bypassing the confirmation (#961).

**Solutions:**

1. **Update to Bambuddy v0.2.3b4 or later** — the gate is now persisted to the database and keyed off an explicit "awaiting acknowledgment" flag instead of the reported state. The prompt survives Auto Off power cycles, Bambuddy restarts, and the printer booting back into `IDLE`.

2. **If the prompt still doesn't appear after the update**, confirm **Require plate-clear confirmation** is enabled in **Settings → Workflow → Queue & Dispatch**.

3. **Dismissing a stale prompt** — if Bambuddy thinks the plate is still uncleared but you've already removed the print (e.g. after a crash), just tap **Clear Plate & Start Next**. The scheduler will dispatch the next job if one is queued; if the queue is empty the flag is simply cleared.

---

### Print fails with "Failed to get AMS mapping table" or a nozzle-size error { #nozzle-size-mismatch }

**Symptoms:** You slice a file in Bambuddy and send it to the printer, and the printer immediately rejects it with an HMS error such as **"Failed to get AMS mapping table; please select 'Resume' to retry"** (code `0700_8012`) or **"The nozzle diameter in the sliced file is not consistent with the current nozzle setting"** (code `0500_4038`). A related giveaway: when configuring an AMS slot, the filament-profile picker only offers **0.4 mm** profiles even though a different nozzle (e.g. 0.6 mm) is installed.

**Background:**

These errors mean the file was sliced for a different nozzle diameter than the one physically installed on the printer. Two things contributed before Bambuddy v0.2.5b2:

- The **Configure AMS Slot** picker assumed a 0.4 mm nozzle regardless of the hardware, so it filtered out the 0.6 mm (or 0.2 / 0.8 mm) filament profiles you needed — you couldn't set your trays to the profile that matched the slice (#1899).
- Bambuddy did not check the sliced nozzle size against the installed nozzle before dispatch, so a mismatch only surfaced later as the printer's cryptic HMS error.

**Solutions:**

1. **Update to Bambuddy v0.2.5b2 or later.** The AMS Slot picker now reads the nozzle actually installed on the printer (per-nozzle on dual-nozzle H2D) and offers the matching profiles. Bambuddy also validates the sliced nozzle size before sending the job and, on a mismatch, fails the queue item with a clear message — *"File sliced for a 0.6mm nozzle, but the printer has 0.4mm installed"* — instead of letting the printer throw the HMS error.

2. **Make the nozzle sizes agree.** Either re-slice the file for the nozzle that's installed, or install the nozzle the file was sliced for. Bambuddy filters filament and process profiles by the selected printer preset's nozzle size, so pick a printer preset whose name matches your installed nozzle (e.g. `… 0.6 nozzle`) when slicing.

3. **Confirm the printer reports the right nozzle.** The printer card shows the detected nozzle diameter (e.g. "• 0.6mm") next to the model in expanded view. If it's wrong or missing after a nozzle swap, the printer firmware may not have re-detected it yet — power-cycle or re-home the printer so it re-reads the installed nozzle, then reconnect in Bambuddy.

---

## :material-folder-lock: Backup Issues

### Backups fail with "Read-only file system" { #backup-read-only-filesystem }

**Symptoms:** Scheduled or manual backups to a NAS share, USB drive or any path outside the Bambuddy install fail with `[Errno 30] Read-only file system`. Your own shell can write to that directory, the permissions look correct, and existing backups in the folder still show up in the list.

**Cause:** This is not a permission problem -- errno 30 is `EROFS`, a permission problem would be errno 13. On a systemd install, Bambuddy runs with `ProtectSystem=strict`, which makes every directory outside the install, data and log directories read-only **for the service**, no matter what its owner and mode say. Reads are unaffected, which is why the backup list still works.

**Solution (systemd):** allow the path in a drop-in, which survives reinstalls and upgrades.

```bash
sudo systemctl edit bambuddy
```

```ini
[Service]
ReadWritePaths=/mnt/nasbackup
```

```bash
sudo systemctl restart bambuddy
systemctl show bambuddy -p ReadWritePaths   # confirm
```

**Solution (Docker):** bind-mount the path into the container -- see [Backup & Restore](../features/backup.md#docker-setup).

Bambuddy now checks the output directory when you save it and reports exactly this, with the command to fix it. If the share itself is genuinely mounted read-only (a stale CIFS/NFS mount after a network blip does this), remount it -- `findmnt -no OPTIONS /mnt/nasbackup` will show `ro`.

---

## :material-cog: General Issues

### Bambuddy Won't Start

**Solutions:**

1. **Check Python version**
   ```bash
   python3 --version  # Need 3.10+
   ```

2. **Check dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Check port availability**
   ```bash
   lsof -i :8000
   ```

4. **Check logs**
   ```bash
   cat logs/bambuddy.log
   ```

---

### Slow Performance

**Solutions:**

1. **Check system resources**
   ```bash
   htop
   ```

2. **Vacuum database**
   - Settings > System Info > Vacuum Database

3. **Clear old logs**
   - Logs rotate automatically
   - Delete very old log files if needed

4. **Check for runaway processes**
   ```bash
   ps aux | grep -E "ffmpeg|python"
   ```

---

## :material-heart-pulse: System Health Checks

Bambuddy scans its own recent log against a catalog of known issues and surfaces
what it finds on the **System page → System Health** section and inside the
in-app bug reporter. Each finding links here. The checks below cover the issues
that catalog can detect — most are setup problems you can fix yourself.

### Printer rejected the access code { #wrong-access-code }

Bambuddy's file-transfer login was refused by the printer. The access code is
wrong, or it changed — the printer generates a **new** access code every time
LAN Developer Mode is toggled.

Re-copy the access code from the printer screen (**Settings → LAN**) and update
it under the printer's settings in Bambuddy. See also [Printer Won't Connect](#printer-wont-connect).

### File-transfer connection timed out { #ftps-port-990-blocked }

Bambuddy could not reach the printer's file-transfer port (**FTPS 990**). The
port is blocked by a firewall, or the printer is powered off or on a different
subnet.

Make sure nothing blocks port 990 between Bambuddy and the printer, and that
both are on the same network. See also [Prints Won't Start / File Transfer Fails](#prints-wont-start-file-transfer-fails).

### Secure file-transfer handshake failed { #ftps-tls-failure }

Bambuddy opened a connection to the printer's file-transfer port (**FTPS 990**)
and the printer answered with something that is not TLS. The log entry looks
like this:

```
FTP SSL error connecting to 192.168.1.50: [SSL: WRONG_VERSION_NUMBER] wrong version number
```

**Restart the printer.** In every case seen so far this is the printer's own
file service getting stuck: the port still accepts connections -- so the
Connection Diagnostic's port check and any firewall test look fine -- but
nothing behind it speaks TLS any more. It is not caused by the printer model,
the firmware version, or a Bambuddy setting. The same model and firmware works
normally on other installs, and the affected printers worked for days before
and after the fault. Power-cycling the printer clears it.

While a printer is in this state, anything that reads a file from it fails:

- print archives arrive with only a filename -- no filament totals, no layer
  count, no MakerWorld link
- cover thumbnails do not load
- timelapse scanning finds nothing

Bambuddy pauses file transfers to that printer for five minutes after a failed
handshake rather than retrying every candidate path, so the log shows a handful
of these entries rather than thousands. Printing itself is unaffected -- the
control connection (MQTT 8883) is a separate service.

If a restart does not help, check that no firewall or TLS-inspecting proxy sits
between Bambuddy and the printer, and see [A1/A1 Mini FTP Issues](#a1a1-mini-ftp-issues)
if the printer is an A1 or A1 Mini.

### Printer connection keeps dropping { #mqtt-connection-unstable }

The control connection (**MQTT 8883**) repeatedly disconnects and reconnects.
This is usually a weak Wi-Fi path to the printer or a partially blocked port.

Check the Wi-Fi signal at the printer, prefer a wired connection, and make sure
port 8883 is reliably reachable. See also [Connection Drops Frequently](#connection-drops-frequently).

### Camera stream unreachable { #camera-rtsps-port-322 }

The live camera could not be reached on **RTSPS 322**. The port is blocked, or
the camera or LAN liveview is disabled on the printer. This does not affect
printing.

Enable the camera and LAN liveview on the printer and make sure port 322 is not
blocked. See also [Camera Won't Stream](#camera-wont-stream).

### Bambu Cloud is asking for a CAPTCHA { #bambu-cloud-captcha }

Signing in to Bambu Cloud fails with a message about confirming you are not a
robot, and no CAPTCHA is shown anywhere. There is nothing to click because there
is nowhere to click it: Bambu's anti-abuse layer is challenging your network,
and answering a CAPTCHA needs a real browser.

Bambu answers the sign-in with `HTTP 418` and a challenge body:

```
{"captchaId": "...", "error": "We need you to confirm you are not a robot"}
```

**Your email and password are not the problem, and neither is your Bambuddy
install.** The challenge is keyed to your public IP address, so it applies to
every account and every app behind that address, and it normally clears by
itself within a few hours. Repeated sign-in attempts extend it -- Bambuddy stops
sending sign-in requests for five minutes after seeing a challenge for exactly
that reason, so a retry inside that window is refused locally without touching
Bambu.

What makes it more likely:

- a VPN, Tailscale exit node, or mobile/CGNAT connection that shares one public
  address with many users
- a hosting provider or datacenter IP range
- a burst of Bambu Cloud or MakerWorld requests from the same address

To sign in while it lasts, use an access token instead. On the **Profiles →
Cloud Profiles** login form, choose **Use access token instead** and paste a
token from a browser session — the token path does not go through the
challenged sign-in endpoint. The same challenge also blocks MakerWorld imports;
use **Open on MakerWorld** and import the 3MF manually until it clears.

### Database write contention { #database-is-locked }

The SQLite database is hitting `database is locked` errors under load — common
when running several printers at once.

Switch Bambuddy to an external PostgreSQL database. See ["Database is locked" Errors](#database-is-locked-errors)
and the [PostgreSQL guide](../features/postgresql.md).

---

## :material-help-circle: Getting More Help

### Information to Gather

When reporting issues, include:

1. **Bambuddy version** - Settings > About
2. **Printer model** - X1C, P1S, etc.
3. **Operating system** - Linux, macOS, Windows
4. **Installation method** - Docker, manual
5. **Steps to reproduce** - What you did
6. **Error messages** - Exact text
7. **Logs** - Relevant log entries

### Where to Get Help

- [GitHub Issues](https://github.com/maziggy/bambuddy/issues)
- [GitHub Discussions](https://github.com/maziggy/bambuddy/discussions)

### Debugging Mode

Enable debug logging for more details:

```bash
# In .env file
DEBUG=true

# Or environment variable
DEBUG=true uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```
