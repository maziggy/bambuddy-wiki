---
title: System Info
description: View database statistics, system resources, and telemetry settings
---

# System Info

View system information, database statistics, and manage telemetry settings.

---

## :material-information: Overview

The System Info page shows:

- **Database statistics** - Archive counts, sizes
- **Resource usage** - Memory, storage
- **Application info** - Version, uptime
- **Telemetry settings** - Anonymous usage data

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

## :material-chart-bar: Telemetry

### Anonymous Usage Data

Bambuddy can collect anonymous usage statistics to help improve the application.

### What's Collected

| Data | Purpose |
|------|---------|
| **Feature usage** | Which features are popular |
| **Error counts** | Common issues |
| **Print counts** | Usage volume |
| **OS/Version** | Compatibility |

### What's NOT Collected

- Personal information
- Printer details
- IP addresses
- Print names or files
- Filament info
- Anything identifiable

### Opt-Out

Disable telemetry:

1. Go to **Settings** > **System Info**
2. Find **Anonymous Telemetry**
3. Toggle **OFF**
4. Click **Save**

!!! info "Telemetry Helps Development"
    Anonymous usage data helps prioritize features and identify issues. Consider leaving it enabled.

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
