---
title: Backup & Restore
description: Database backup and restore
---

# Backup & Restore

Protect your print history with database backups and restore when needed.

---

## :material-backup-restore: Overview

Bambuddy's backup system:

- **Full database backup** - All your data in one file
- **User settings included** - Preferences preserved
- **Archives included** - Print history saved
- **ZIP format** - Includes 3MF files and thumbnails when selected

---

## :material-cloud-download: Creating a Backup

### Manual Backup

1. Go to **Settings** > **Backup**
2. Click **Create Backup**
3. Wait for backup to complete
4. Download the backup file

### Backup Contents

| Data | Included |
|------|:--------:|
| Print archives | :material-check: |
| Printer configs | :material-check: |
| User settings | :material-check: |
| Notification configs | :material-check: |
| Smart plug configs | :material-check: |
| K-profiles | :material-check: |
| External links | :material-check: |
| Projects | :material-check: |
| Maintenance records | :material-check: |
| API keys | :material-check: |

### What's NOT Included

| Data | Reason |
|------|--------|
| 3MF files | Too large (download separately) |
| Thumbnails | Regenerated from archives |
| Logs | Not needed for restore |
| Cache | Temporary data |

---

## :material-folder-zip: Backup File

### Format

Backup files are SQLite database exports:

```
bambuddy_backup_2024-01-15_143022.db
```

### File Size

Typical sizes:

- 100 prints: ~5-10 MB
- 1,000 prints: ~50-100 MB
- 10,000 prints: ~500 MB - 1 GB

Size depends on metadata stored per print.

---

## :material-cloud-upload: Restoring from Backup

### Full Restore

Restore everything from a backup:

1. Go to **Settings** > **Backup**
2. Click **Restore from Backup**
3. Select your backup file
4. Click **Full Restore**
5. Confirm (this overwrites current data)
6. Wait for restore to complete
7. Restart Bambuddy

!!! warning "Full Restore Overwrites"
    A full restore replaces all current data. Create a backup first!

---

## :material-database-export: Manual Database Access

### Location

The database is stored at:

```
/path/to/bambuddy/bambuddy.db
```

### Direct Backup

Copy the database file directly:

```bash
cp bambuddy.db bambuddy_backup_$(date +%Y%m%d).db
```

!!! warning "Stop Bambuddy First"
    Stop Bambuddy before copying to ensure consistency.

### SQLite Tools

Use SQLite tools for advanced operations:

```bash
# Dump to SQL
sqlite3 bambuddy.db .dump > backup.sql

# Restore from SQL
sqlite3 new.db < backup.sql
```

---

## :material-folder-download: Downloading Archives

3MF files aren't in database backups. Download separately:

### Individual Download

1. Open an archive
2. Click **Download 3MF**

### Bulk Export

1. Go to **Archives**
2. Click **Export**
3. Select **Include 3MF files**
4. Download the archive

---

## :material-restore: Recovery Scenarios

### Lost Database

If database is corrupted or deleted:

1. Stop Bambuddy
2. Remove corrupted `bambuddy.db`
3. Start Bambuddy (creates fresh database)
4. Go to Settings > Backup
5. Restore from your backup

### New Installation

Moving to a new server:

1. Install Bambuddy on new server
2. Copy backup file to new server
3. Go to Settings > Backup
4. Full restore from backup

### Data Migration

Moving between versions:

1. Create backup on old version
2. Upgrade Bambuddy
3. If needed, restore from backup

---

## :material-shield-check: Best Practices

### Recommended Backup Frequency

| Frequency | Good For |
|-----------|----------|
| Daily | Active printing (set a reminder) |
| Weekly | Regular use |
| Monthly | Light use |

### Storage

- Keep backups off the Bambuddy server
- Use cloud storage (Dropbox, Google Drive, etc.)
- Keep multiple versions

### Testing

- Periodically test restoring
- Verify backup integrity
- Document your backup process

---

## :material-lightbulb: Tips

!!! tip "Before Major Changes"
    Always backup before upgrading or making big configuration changes.

!!! tip "Off-Site Storage"
    Store at least one backup off-site (cloud or another location).

!!! tip "Regular Testing"
    Test your restore process periodically to ensure backups work.

!!! tip "Version in Filename"
    Include date and version in backup filenames for easy identification.

!!! tip "3MF Backup"
    For complete backup, also download your 3MF files separately.
