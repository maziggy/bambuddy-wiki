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
- **GitHub backup** - Automatic cloud backup of profiles and settings

---

## :material-github: GitHub Profile Backup

Automatically backup your K-profiles, cloud profiles, and app settings to a GitHub repository.

### Setup

1. **Create a GitHub repository** (can be private)
2. **Generate a Personal Access Token (PAT)**:
    - Go to GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
    - Generate a new token with `repo` scope
3. **Configure in Bambuddy**:
    - Go to **Settings** → **Backup & Restore**
    - Enter your repository URL (e.g., `https://github.com/username/bambuddy-backup`)
    - Enter your PAT
    - Click **Test Connection** to verify

!!! info "Bambu Cloud Login Required"
    GitHub backup requires you to be logged into Bambu Cloud to access your cloud profiles and K-profiles. Login via **Profiles** → **Cloud Profiles**.

### What's Backed Up

| Data | Description |
|------|-------------|
| K-profiles | Per-printer pressure advance profiles (organized by serial number) |
| Cloud profiles | Filament, printer, and process profiles from Bambu Cloud |
| App settings | Application configuration (when enabled) |

### Schedule Options

| Schedule | Description |
|----------|-------------|
| Hourly | Backup runs every hour |
| Daily | Backup runs once per day (midnight) |
| Weekly | Backup runs once per week (Sunday midnight) |

### Manual Backup

Click **Backup Now** to trigger an immediate backup.

### Repository Structure

```
repo/
├── backup_metadata.json
├── kprofiles/
│   └── {serial_number}/
│       ├── 0.2.json
│       ├── 0.4.json
│       └── ...
├── cloud_profiles/
│   ├── filament.json
│   ├── printer.json
│   └── process.json
└── settings/
    └── app_settings.json
```

### Backup History

View backup history in the **Backup History** section:

- Status (success, failed, skipped)
- Trigger (manual or scheduled)
- Files changed
- Commit SHA (linked to GitHub)

!!! tip "Skip Unchanged"
    Bambuddy only creates a commit when data has actually changed, avoiding unnecessary commits.

---

## :material-cloud-download: Creating a Local Backup

### Manual Backup

1. Go to **Settings** > **Backup & Restore**
2. Click **Download Backup**
3. Wait for backup to complete (progress indicator shown)
4. ZIP file downloads automatically

!!! warning "Don't Navigate Away"
    During backup/restore, stay on the page. The UI blocks navigation to prevent data corruption.

### Backup Contents

The backup is a complete ZIP file containing:

| Data | Included |
|------|:--------:|
| Database (all tables) | :material-check: |
| Print archives (3MF files) | :material-check: |
| Archive thumbnails | :material-check: |
| Timelapse videos | :material-check: |
| Library files | :material-check: |
| Library thumbnails | :material-check: |
| Project files | :material-check: |
| Printer icons | :material-check: |
| Plate calibration data | :material-check: |
| Virtual printer uploads | :material-check: |

!!! success "Complete by Definition"
    The new backup system copies the entire database file and all data directories. No data can be missed because everything is included automatically.

---

## :material-folder-zip: Backup File

### Format

Backup files are ZIP archives:

```
bambuddy-backup-20260201-143022.zip
```

### ZIP Structure

```
bambuddy-backup-YYYYMMDD-HHMMSS.zip
├── bambuddy.db              # Complete SQLite database
├── archive/                 # All archive data
│   ├── <archive_id>/        # Individual archive folders
│   │   ├── *.3mf            # Archived print files
│   │   ├── thumbnail.png    # Thumbnails
│   │   ├── timelapse.mp4    # Timelapses
│   │   ├── source.3mf       # Original source 3MF
│   │   ├── *.f3d            # Fusion 360 files
│   │   └── photos/*.jpg     # Photo attachments
│   └── library/             # File Manager
│       ├── files/           # Uploaded files
│       └── thumbnails/      # Generated thumbnails
├── virtual_printer/         # Pending uploads
├── projects/                # Project files
├── icons/                   # Custom printer icons
└── plate_calibration/       # Plate detection references
```

### File Size

Typical sizes depend on your archive content:

- Small archive (100 prints, no timelapses): ~100-500 MB
- Medium archive (500 prints, some timelapses): ~1-5 GB
- Large archive (1000+ prints, full timelapses): ~10+ GB

!!! tip "Large Backups"
    If you have many timelapse videos, backups can be large. Consider periodic cleanup of old timelapses.

---

## :material-cloud-upload: Restoring from Backup

### Full Restore

Restore everything from a backup ZIP:

1. Go to **Settings** > **Backup & Restore**
2. Click **Restore Backup**
3. Select your backup ZIP file
4. Wait for restore to complete (progress indicator shown)
5. **Restart Bambuddy** when prompted

!!! warning "Full Restore Overwrites"
    A full restore replaces all current data including the database and all files. Create a backup first!

!!! info "Restart Required"
    After restore, you must restart Bambuddy for changes to take effect. The database connection is replaced during restore.

### Portable Backups

Backups are fully portable between installations:

- **Different servers**: Move from one machine to another
- **Different paths**: Works even if data directory changed
- **Different Docker volumes**: Migrate between container setups

The backup system stores relative paths internally, so files work correctly regardless of where Bambuddy is installed.

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
