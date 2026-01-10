---
title: Troubleshooting
description: Common issues and solutions
---

# Troubleshooting

Solutions for common issues with Bambuddy.

---

## :material-printer-3d: Printer Connection Issues

### Printer Won't Connect

**Symptoms:** Printer shows as disconnected, red indicator

**Solutions:**

1. **Verify LAN Mode is enabled**
   - Check printer: Settings > Network > LAN Mode
   - Toggle off and on to get a fresh access code

2. **Check IP address**
   - Verify IP in printer network settings
   - Ensure IP hasn't changed (use static IP or DHCP reservation)

3. **Verify access code**
   - Access code changes when LAN Mode is toggled
   - Copy the code exactly (case-sensitive)

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

---

## :material-archive: Archiving Issues

### Prints Not Being Archived

**Symptoms:** Prints complete but don't appear in archives

**Solutions:**

1. **Check printer connection**
   - Must be connected during and after print
   - Verify green connection indicator

2. **Check FTP access**
   - Port 990 must be accessible
   - Try downloading a file manually

3. **Check disk space**
   - Ensure enough space for 3MF files
   - Clear old files if needed

4. **Enable FTP retry for weak WiFi**
   - P1S, X1C, A1, and other printers often have weak WiFi
   - Go to Settings > General > FTP Retry
   - Enable retry with 3-5 attempts and 2-3 second delay
   - Increase connection timeout to 60-120s for A1/A1 Mini
   - This helps when FTP transfers fail intermittently

5. **Check logs for errors**
   ```bash
   grep -i "archive\|error" logs/bambuddy.log
   ```

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
   - Camera must be enabled in LAN Mode settings

3. **Check connection**
   - Camera requires active printer connection
   - Verify printer shows as connected

4. **Try snapshot mode**
   - Click "Snapshot" instead of "Live"
   - May work when streaming doesn't

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

### Plug Not Responding

**Symptoms:** Can't control smart plug

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
