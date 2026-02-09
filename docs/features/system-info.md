---
title: System Info
description: View database statistics and system resources
---

# System Info

View system information and database statistics.

---

## :material-information: Overview

The System Info page shows:

- **Database statistics** - Archive counts, sizes
- **Resource usage** - Memory, storage
- **Application info** - Version, uptime

---

## :material-database: Database Statistics

### Archive Statistics

| Metric | Description |
|--------|-------------|
| **Total Archives** | Number of archived prints |
| **Total Size** | Database file size |
| **3MF Files** | Number of stored 3MF files |
| **Thumbnails** | Number of thumbnail images |
| **Photos** | Number of attached photos |

### Print Statistics

| Metric | Description |
|--------|-------------|
| **Successful Prints** | Completed prints |
| **Failed Prints** | Print failures |
| **Total Print Time** | Cumulative hours |
| **Total Filament** | Cumulative kg |

### Database Health

| Check | Status |
|-------|--------|
| **Integrity** | Database file intact |
| **FTS Index** | Full-text search working |
| **Connections** | Active database connections |

---

## :material-memory: Resource Usage

### Memory

| Metric | Value |
|--------|-------|
| **RAM Used** | Current usage |
| **RAM Available** | Free memory |
| **Process Memory** | Bambuddy process |

### Storage

| Metric | Value |
|--------|-------|
| **Database Size** | SQLite file size |
| **Archive Folder** | 3MF and thumbnail storage |
| **Log Files** | Log directory size |
| **Total Used** | All Bambuddy data |

### Disk Space

| Location | Free Space |
|----------|------------|
| **Data Directory** | Available storage |
| **Archive Directory** | Available storage |

---

## :material-application: Application Info

### Version

| Info | Value |
|------|-------|
| **Version** | Current Bambuddy version |
| **Python** | Python version |
| **Database** | SQLite version |

### Runtime

| Info | Value |
|------|-------|
| **Uptime** | Time since start |
| **Started** | Start timestamp |
| **Restarts** | Number of restarts |

### Environment

| Info | Value |
|------|-------|
| **OS** | Operating system |
| **Architecture** | CPU architecture |
| **Docker** | Running in Docker? |

---

## :material-tools: Maintenance Actions

### Rebuild FTS Index

If search isn't working correctly:

1. Click **Rebuild Search Index**
2. Wait for completion
3. Search should work now

### Clear Cache

Remove temporary files:

1. Click **Clear Cache**
2. Confirm action
3. Cache is cleared

### Vacuum Database

Optimize database size:

1. Click **Vacuum Database**
2. Wait for completion
3. May reduce file size

!!! note "When to Vacuum"
    Vacuum after deleting many archives to reclaim disk space.

---

## :material-bug: Debug Information

### Log Files

Access application logs:

| Log | Content |
|-----|---------|
| **Application** | General operations |
| **MQTT** | Printer communication |
| **Database** | Database operations |

### Log Level

Configure logging detail:

| Level | Detail |
|-------|--------|
| ERROR | Only errors |
| WARNING | Errors + warnings |
| INFO | Normal operations |
| DEBUG | Everything (verbose) |

---

## :material-text-box-search: Log Viewer

View and filter application logs in real-time directly from the System Information page.

### Opening the Log Viewer

1. Go to **System Information** page
2. Find **Support & Troubleshooting** section
3. Click on **Application Logs** to expand the viewer

### Controls

| Button | Action |
|:------:|--------|
| **Start** | Begin live streaming (auto-refresh every 2 seconds) |
| **Stop** | Pause live streaming |
| :material-trash-can: Clear | Clear the log file |
| :material-refresh: Refresh | Manual one-time refresh |

### Filtering Options

| Filter | Description |
|--------|-------------|
| **Level** | Filter by log level: All, DEBUG, INFO, WARNING, ERROR |
| **Search** | Text search across messages and logger names |
| **Auto-scroll** | Automatically scroll to newest entries |

### Log Entry Details

Each log entry shows:

- **Timestamp** - Time the log was written
- **Level** - Log severity (color-coded)
- **Logger** - Component that generated the log
- **Message** - Log content

Click on entries with multi-line content (like stack traces) to expand them.

### Log Levels

| Level | Color | Description |
|-------|-------|-------------|
| DEBUG | Gray | Detailed debugging information |
| INFO | Blue | Normal operational messages |
| WARNING | Yellow | Warning conditions |
| ERROR | Red | Error conditions |

!!! tip "Debug Level Logs"
    DEBUG level logs are only captured when debug logging is enabled. Use the debug toggle above the log viewer to enable detailed logging.

---

## :material-lifebuoy: Support Bundle

Generate a support bundle for issue reporting. The bundle contains debug logs and system information to help diagnose problems.

### Enable Debug Logging

Before generating a support bundle:

1. Go to **System Information** page
2. Find **Support & Troubleshooting** section
3. Click **Enable** to turn on debug logging
4. A banner appears across all pages showing debug mode is active with a live timer
5. Reproduce the issue you want to report
6. Click **Download** to get the support bundle

### What's Collected

| Data | Description |
|------|-------------|
| **App version** | Current Bambuddy version |
| **OS info** | Platform, architecture, Python version |
| **Database stats** | Archive/printer/filament counts (numbers only) |
| **Printer diagnostics** | Model, firmware version, connectivity status, AMS unit/tray counts, WiFi signal, HMS error count (no names, IPs, or serials) |
| **Integration status** | Spoolman reachability, MQTT relay connection, Home Assistant enabled, notification provider types |
| **Network interfaces** | Interface names and subnets only (e.g., `192.168.1.0/24`) — no host IPs |
| **Python packages** | Versions of key dependencies (FastAPI, Pydantic, etc.) |
| **Database health** | SQLite journal mode, integrity check, DB/WAL file sizes |
| **Docker environment** | Container memory limit, network mode hint (only when running in Docker) |
| **WebSocket connections** | Number of active browser connections |
| **Log file** | Log file size |
| **Settings** | Non-sensitive settings (themes, formats) |
| **Debug logs** | Sanitized application logs |

### What's NOT Collected

| Data | Reason |
|------|--------|
| **Printer names/IPs/serials** | Privacy |
| **Access codes/passwords** | Security |
| **Email addresses** | Filtered from settings and logs |
| **API keys/tokens** | Security |
| **Webhook URLs** | May contain sensitive info |
| **Your hostname/username** | Privacy |

!!! info "Privacy First"
    IP addresses in logs are replaced with `[IP]` and email addresses with `[EMAIL]`. Paths are sanitized to hide usernames.

### Using the Bundle

1. Download the ZIP file
2. Attach it to your [GitHub issue](https://github.com/maziggy/bambuddy/issues)
3. Describe the problem you encountered
4. Remember to disable debug logging when done

!!! warning "Disable Debug Logging"
    Debug logging increases log verbosity and file size. Remember to disable it after collecting the bundle.

---

## :material-api: API Endpoint

Get system info via API:

```bash
GET /api/v1/system/info
```

Response includes:

```json
{
  "version": "0.1.5b6",
  "uptime": 86400,
  "database": {
    "archives": 1234,
    "size_mb": 45.6
  },
  "resources": {
    "memory_mb": 256,
    "disk_free_gb": 100
  }
}
```

---

## :material-lightbulb: Tips

!!! tip "Regular Checks"
    Check system info periodically to monitor storage and performance.

!!! tip "Before Backups"
    Review database size before backing up to plan storage.

!!! tip "Troubleshooting"
    Check uptime and logs when investigating issues.

!!! tip "Clean Up"
    Use vacuum and cache clear to maintain performance.
