# Troubleshooting

Solutions to common issues with Bambuddy.

## Printer Connection Issues

### Printer Won't Connect

**Check LAN Mode**
1. On printer: **Settings** > **Network** > **LAN Mode**
2. Ensure LAN Mode is **enabled**
3. Note the **Access Code** (changes when toggled)

**Verify Credentials**
- **IP Address**: Correct local IP (not cloud address)
- **Access Code**: 8-character code from LAN Mode screen
- **Serial Number**: Found in **Settings** > **Device Info**

**Network Issues**
- Printer and Bambuddy must be on same network
- Check printer can be pinged: `ping <printer-ip>`
- Ensure ports 8883 (MQTT) and 990 (FTP) aren't blocked

**Other Applications**
- Only one MQTT connection allowed per printer
- Close Bambu Studio or Bambu Handy
- Disconnect other monitoring tools

### "Connection Refused" Errors

**Printer in Sleep Mode**
- Wake the printer and try again
- Sleep mode disconnects MQTT

**Another Connection Active**
- Close Bambu Studio
- Close Bambu Handy
- Wait 30 seconds and retry

**Printer Restart**
- Power cycle the printer
- Wait for full boot
- Try connecting again

### Intermittent Disconnections

**Network Stability**
- Check WiFi signal strength at printer
- Consider wired connection if available
- Reduce network congestion

**Printer Firmware**
- Update to latest firmware
- Some versions have MQTT bugs

**Bambuddy Logs**
- Check `logs/bambuddy.log` for errors
- Enable DEBUG mode for more detail

---

## Archiving Issues

### Prints Not Archiving Automatically

**Check Connection**
- Printer must show green (connected)
- Verify real-time updates are working

**Print Completion**
- Only completed prints are archived
- Cancelled prints may not archive
- Check print actually finished

**FTP Access**
- Verify FTP connectivity
- Check logs for FTP errors

**View Logs**
```bash
tail -f logs/bambuddy.log
```
Look for archiving-related errors.

### 3MF Download Fails

**FTP Connection**
- Port 990 must be accessible
- Check firewall settings
- Verify printer responds to FTP

**File Still in Use**
- Wait for print to fully complete
- Printer may still be processing

**Storage Full**
- Check Bambuddy server disk space
- Check `archive/` directory size

### Missing Thumbnails

**3MF Content**
- Some 3MF files lack thumbnails
- Manually sliced files may not have previews

**Extraction Issue**
- Check logs for thumbnail errors
- Re-archive the print if needed

---

## Timelapse Issues

### Timelapse Not Attaching Automatically

**Printer Clock Issue** (Most Common)
When printers run in LAN-only mode, they can't sync time via NTP. The internal clock drifts, causing timelapse matching to fail.

**Symptoms:**
- "No matching timelapse found"
- Files exist on printer but don't attach
- Printer shows wrong date/time

**Workaround - Manual Selection:**
1. Right-click archive > **Scan for Timelapse**
2. If no auto-match, dialog shows available files
3. Select the correct timelapse
4. Click to attach

**Permanent Fix:**
1. Temporarily connect printer to internet
2. Wait for NTP time sync
3. Return to LAN-only mode
4. Clock remains accurate until power cycle

### Timelapse Recording Not Working

**Enable in Printer Settings**
1. On printer: **Settings** > **Camera**
2. Enable **Timelapse Recording**

**Storage Space**
- Timelapses need SD card space
- Clear old files if full

---

## Frontend Issues

### Frontend Not Loading

**Build the Frontend**
```bash
cd frontend
npm install
npm run build
```

**Verify Static Files**
- Check `/static` folder exists
- Contains `index.html` and `/assets`

**Clear Browser Cache**
- Hard refresh: Ctrl+Shift+R (Cmd+Shift+R on Mac)
- Try incognito/private window

### Blank Page or Errors

**Browser Console**
- Open DevTools (F12)
- Check Console for errors
- Look for failed network requests

**API Connection**
- Backend must be running
- Check backend logs for errors
- Verify port 8000 is accessible

### WebSocket Not Connecting

**Check Backend**
- Backend must be running
- WebSocket endpoint: `/api/v1/ws`

**Firewall/Proxy**
- WebSocket needs persistent connection
- Some proxies block WebSocket
- Check network configuration

---

## Database Issues

### Database Errors

**Backup and Reset**
```bash
# Backup current database
mv bambuddy.db bambuddy.db.backup

# Restart Bambuddy - creates new database
uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```

**Migration Issues**
- New versions may change schema
- Usually handled automatically
- Check logs for migration errors

### Data Not Persisting

**File Permissions**
- Bambuddy needs write access to:
  - `bambuddy.db`
  - `archive/` directory
  - `logs/` directory

**Disk Space**
- Ensure adequate free space
- SQLite needs space for transactions

---

## Smart Plug Issues

### Plug Not Responding

**Network**
1. Verify IP address is correct
2. Check plug is on same network
3. Test directly: `http://<plug-ip>/cm?cmnd=Power`

**Tasmota Web Interface**
- Access `http://<plug-ip>`
- Verify Tasmota is running
- Check for firmware updates

**Authentication**
- If Tasmota has password, configure in Bambuddy
- Check credentials are correct

### Auto Power-Off Not Working

**Configuration**
1. Plug must be linked to printer
2. Automation must be enabled
3. Auto Off must be enabled

**Temperature Mode**
- Printer must stay connected
- Bambuddy reads nozzle temp
- Check temp threshold setting

**Time Mode**
- Verify delay value is set
- Check logs for power-off attempts

---

## Scheduled Print Issues

### Print Not Starting

**Printer Not Ready**
- Printer must be idle
- No active prints
- Printer must connect after power-on

**Smart Plug**
- If using auto power-on, plug must work
- Verify plug automation is configured

**Queue Status**
- Check queue page for errors
- Look for failed status

### "Failed to Start" Error

**Common Causes**
- Printer not ready or connected
- FTP upload failed
- HMS errors preventing print

**Check:**
1. Printer HMS status (no errors)
2. Printer is idle and connected
3. FTP port 990 accessible
4. File exists in archive

---

## Notification Issues

### Not Receiving Notifications

**Provider Configuration**
- Verify credentials are correct
- Use **Send Test** to verify
- Check provider-specific requirements

**Event Triggers**
- Ensure desired events are enabled
- Check per-printer filtering

**Quiet Hours**
- Notifications suppressed during quiet hours
- Verify quiet hours settings

### Test Works But Events Don't

**Event Not Triggering**
- Verify event type is enabled
- Check printer filter settings
- Look for errors in logs

**Daily Digest**
- If enabled, events are batched
- Wait for digest time
- Or disable digest for immediate notifications

---

## Performance Issues

### Slow Interface

**Browser**
- Clear cache and cookies
- Try different browser
- Disable browser extensions

**Backend**
- Check server resources
- Review logs for errors
- Consider DEBUG=false for production

### High Memory Usage

**Large Archives**
- Many archives increase memory
- Consider archiving to external storage
- Clean up old/unwanted archives

**Multiple Printers**
- Each printer uses MQTT connection
- Normal for multi-printer setups

---

## Viewing Logs

### Log File Location
```bash
# Default location
logs/bambuddy.log

# View live
tail -f logs/bambuddy.log

# Search for errors
grep -i error logs/bambuddy.log
```

### Enable Debug Logging
```bash
DEBUG=true uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```

### Systemd Service Logs
```bash
sudo journalctl -u bambuddy -f
```

---

## Getting Help

### Before Asking for Help
1. Check this troubleshooting guide
2. Review logs for error messages
3. Search [existing issues](https://github.com/maziggy/bambuddy/issues)
4. Enable DEBUG mode and reproduce the issue

### Reporting Issues
When opening an issue, include:
- Bambuddy version
- Printer model
- Operating system
- Relevant log snippets
- Steps to reproduce

[Open a new issue](https://github.com/maziggy/bambuddy/issues/new)
