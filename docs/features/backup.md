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
- **Git backup** - Automatic cloud backup of profiles and settings to a Git provider

---

## :material-source-repository-multiple: Git Profile Backup

Automatically back up your K-profiles, cloud profiles, app settings, spool inventory, and print archive history to a Git repository.

Bambuddy supports multiple Git providers. Choose one below and follow the setup steps.

### Setup

#### :material-github: GitHub

1. **Create a GitHub repository** (can be private)
2. **Generate a Personal Access Token (PAT)**:
    - Go to [GitHub Personal Access Tokens](https://github.com/settings/tokens){ target="_blank" rel="noopener" } (`https://github.com/settings/tokens`)
    - Click **Generate new token**, then **Generate new token (classic)**
	- Choose your expiration timeframe (`No expiration` is recommended)
	- Under **Select scopes**, check `repo` (required for repository access and commits)
3. **Configure in Bambuddy**:
    - Go to **Settings** → **Backup & Restore**
    - Enter your repository URL (e.g., `https://github.com/username/bambuddy-backup`)
    - Enter your PAT (from Step 2)
    - Click **Test Connection** to verify

!!! note "GitHub Authentication"
    Instead of *Classic Tokens*, you can use the *Fine-grained tokens*. Be sure to choose read access for `Metadata`. It will automatically include `Read and Write access to code` upon creation.

#### :material-gitlab: GitLab

1. **Create a GitLab repository** (can be private)
2. **Generate a Personal Access Token (PAT)**:
    - Go to [GitLab Personal Access Tokens](https://gitlab.com/-/user_settings/personal_access_tokens){ target="_blank" rel="noopener" } (`https://gitlab.com/-/user_settings/personal_access_tokens`)
	- Under **Personal access tokens**,  click **Generate token**, then **Legacy token**
    - Under scopes, check the `api` scope (required for repository access and commits)
3. **Configure in Bambuddy**:
    - Go to **Settings** → **Backup & Restore**
    - Select **GitLab** from the provider dropdown
    - Enter your repository URL (e.g., `https://gitlab.com/username/bambuddy-backup`)
	- If you are hosting locally without HTTPS, be sure to check the `Allow insecure HTTP` checkbox
    - Enter your PAT (from Step 2)
    - Click **Test Connection** to verify

!!! note "GitLab Authentication"
    Project Access Tokens may also be used. Be sure to give your token the `api` and `write_repository` scopes, otherwise you'll run into access failures.

#### :material-git: Gitea

1. **Create a new repository** (can be private)
2. **Generate a Personal Access Token (PAT)**:
    - Go to **Settings** → **Applications**
    - Under **Manage Access Tokens**, expand **Generate New Token**
    - Enter a name for the token
    - Under **Repository and organization access**, choose `All (public, private, and limited)`
    - Under **Repository permissions**, set **repository** to `Read and write`
    - Click **Generate token**
3. **Configure in Bambuddy**:
    - Go to **Settings** → **Backup & Restore**
    - Select **Gitea** from the provider dropdown
    - Enter your repository URL (e.g., `https://example.com/username/bambuddy-backup`)
	- If you are hosting locally without HTTPS, be sure to check the `Allow insecure HTTP` checkbox
    - Enter your PAT (from Step 2)
	- Be sure to specify the right **Branch** (main/master/etc)
    - Click **Test Connection** to verify

!!! info "Tested Versions"
    Bambuddy has been tested and verified against **Gitea 1.24.x, 1.25.x, and 1.26.x**. Other versions may work but are not officially supported.

#### :material-git: Forgejo

1. **Create a new repository** (can be private)
2. **Generate a Personal Access Token (PAT)**:
    - Go to **Settings** → **Applications**
    - Click **New access token**
    - Enter a name for the token
    - Under **Repository and organization access**, choose either:
        - `All` to allow access to all repositories
        - `Specific` and select your Bambuddy backup repository
    - Under **Repository permissions**, set **repository** to `Read and write`
    - Click **Generate Token**
3. **Configure in Bambuddy**:
    - Go to **Settings** → **Backup & Restore**
    - Select **Forgejo** from the provider dropdown
    - Enter your repository URL (e.g., `https://example.com/username/bambuddy-backup`)
	- If you are hosting locally without HTTPS, be sure to check the `Allow insecure HTTP` checkbox
    - Enter your PAT (from Step 2)
    - Click **Test Connection** to verify

!!! info "Tested Versions"
    Bambuddy has been tested and verified against **Forgejo 11.x LTS and 15.x LTS**. Other versions may work but are not officially supported.

### What's Backed Up

| Data | Description | Default |
|------|-------------|---------|
| K-profiles | Per-printer pressure advance profiles (organized by serial number) | On |
| Cloud profiles | Filament, printer, and process profiles from Bambu Cloud | On |
| App settings | Application configuration | Off |
| Spool inventory | Filament spools, usage history, and cost tracking | Off |
| Print archives | Print history metadata — filament, temperatures, times, costs, energy (no gcode/3MF files) | Off |

!!! warning "Bambu Cloud Login Required"
    In order to backup your *Cloud profiles* and your *K-profiles*, you must be logged into Bambu Cloud. Login via **Profiles** → **Cloud Profiles**.

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
├── settings/
│   └── app_settings.json
├── spools/
│   ├── inventory.json
│   └── usage_history.json
└── archives/
    └── print_history.json
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

## :material-clock-outline: Scheduled Local Backups

Automate local backups on a recurring schedule so you never forget to protect your data.

### Overview

The **Scheduled Backups** card in **Settings** > **Backup** lets you configure automatic local backups that run in the background. Each backup produces the same complete ZIP file as a manual backup (database + all data directories), stored on disk where you can easily copy them to a NAS or external drive.

Works with both SQLite and PostgreSQL installations.

### Configuration

| Setting | Options | Default |
|---------|---------|---------|
| Enable | On / Off | Off |
| Frequency | Hourly, Daily, Weekly | Daily |
| Time of day | Time picker (for Daily and Weekly) | 00:00 |
| Retention count | 1--100 | 5 |
| Output path | Any local or network path | `DATA_DIR/backups/` |

!!! info "Retention"
    When a new backup completes, Bambuddy automatically deletes the oldest backups beyond the retention count. For example, with retention set to 5, only the 5 most recent backups are kept.

### Managing Backups

Each backup in the list supports three actions:

- **Download** -- download the ZIP file to your browser
- **Restore** -- restore directly from the backup (same as manual restore)
- **Delete** -- remove the backup file from disk

Click **Run Now** to trigger an immediate backup outside the schedule.

### Docker Setup

Docker users should mount the backup output directory as a volume so backups are persisted outside the container. Add a bind mount to your `docker-compose.yml`:

```yaml
services:
  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    container_name: bambuddy
    network_mode: host
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
      - /path/to/nas/bambuddy-backups:/app/data/backups  # Scheduled backup output
    environment:
      - TZ=Europe/Berlin
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
```

Or with `docker run`:

```bash
docker run -d \
  --network host \
  -v bambuddy_data:/app/data \
  -v bambuddy_logs:/app/logs \
  -v /path/to/nas/bambuddy-backups:/app/data/backups \
  -e TZ=Europe/Berlin \
  --name bambuddy \
  --restart unless-stopped \
  ghcr.io/maziggy/bambuddy:latest
```

!!! tip "NAS or Network Share"
    Point the bind mount at a NAS share, Samba mount, or NFS path for automatic off-site backups without any extra scripts.

### Non-Docker Setup

For bare-metal or virtual machine installs, set the **Output path** in the UI to any directory your Bambuddy process can write to -- a local folder, a mounted network drive, or an external USB drive.

### API Endpoints

For automation or monitoring, the scheduled backup system exposes a REST API:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/local-backup/status` | Current schedule config and next run time |
| `POST` | `/local-backup/run` | Trigger an immediate backup |
| `GET` | `/local-backup/backups` | List all backup files |
| `GET` | `/local-backup/backups/{filename}/download` | Download a backup file |
| `POST` | `/local-backup/backups/{filename}/restore` | Restore from a backup file |
| `DELETE` | `/local-backup/backups/{filename}` | Delete a backup file |

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
├── bambuddy.db              # Database (portable SQLite format, works on both SQLite and PostgreSQL installs)
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
- **Different database backends**: Restore a SQLite backup into a PostgreSQL install (and vice versa)

The backup system always exports data in portable SQLite format, regardless of which database backend you use. When restoring into PostgreSQL, Bambuddy automatically converts data types (booleans, datetimes) and handles foreign key constraints.

---

## :material-database-export: Manual Database Access

=== ":material-database: SQLite (Default)"

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

=== ":material-elephant: PostgreSQL"

    ### Connection

    Connect to your PostgreSQL database using the same `DATABASE_URL` from your configuration:

    ```bash
    psql "postgresql://user:pass@host:5432/bambuddy"
    ```

    ### Direct Backup

    ```bash
    pg_dump -Fc "postgresql://user:pass@host:5432/bambuddy" > bambuddy_backup_$(date +%Y%m%d).dump
    ```

    ### Restore

    ```bash
    pg_restore --clean --if-exists -d "postgresql://user:pass@host:5432/bambuddy" bambuddy_backup.dump
    ```

    !!! tip "Bambuddy's Built-in Backup is Easier"
        The Settings > Backup page creates portable backups that work across both SQLite and PostgreSQL. Use manual `pg_dump` only if you need PostgreSQL-specific features like point-in-time recovery.

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
| Daily | Active printing (use [Scheduled Local Backups](#scheduled-local-backups)) |
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
