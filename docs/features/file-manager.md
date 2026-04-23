---
title: File Manager
description: Browse and manage your local library of print files
---

# File Manager

Browse, download, and manage files in your local Bambuddy library.

---

## :material-folder: Overview

The File Manager lets you:

- **Browse** files in your local library
- **Mount external folders** from NAS, USB, or network shares
- **Upload** files including ZIP archives
- **Download** files to your computer
- **Rename** files and folders
- **Delete** unwanted files
- **View** file details and metadata
- **Print directly** to any printer with full configuration
- **Add to Queue** sliced files for later printing
- **Link** folders to projects or archives

---

## :material-folder-open: Accessing File Manager

1. Click **File Manager** in the sidebar
2. Or navigate to `/file-manager` in the URL

---

## :material-file-tree: File Browser

### Library Structure

Your library contains uploaded and archived files:

- Folders for organizing files
- 3MF and sliced gcode files
- Linked folders connected to projects/archives

### File Information

Each file shows:

- Filename
- Size
- Modified date
- File type icon
- Print count (if printed before)
- Uploaded by (when authentication is enabled)

---

## :material-navigation: Navigation

### Browsing

- Click folders to enter
- Click ← to go back
- Click root to return home

### Folder sidebar preferences

Two small toggles in the folder sidebar header let you tailor how the tree is rendered. Both preferences are stored in your browser and applied on every subsequent page load.

| Toggle | What it does |
|---|---|
| **Wrap** | When off (default), long folder names are truncated with an ellipsis. When on, long names wrap across multiple lines so the full name stays visible. |
| **Collapse** | When off (default), the folder tree opens with every level expanded. When on, only the top-level folders are shown on load — click the chevron to expand a branch. Toggling the preference also immediately re-collapses or re-expands the current tree. |

!!! tip "When to enable Collapse"
    If your library has many nested folders, turning on **Collapse** keeps the sidebar compact — you only see the top-level folders and drill into a branch when you need it. Small, flat libraries won't notice a difference because the toggle only affects nested folders; top-level folders are always visible.

### Sorting

Sort files by:

- Name (A-Z, Z-A)
- Size (largest/smallest)
- Date (newest/oldest)

### Filtering

Filter by file type:

- All files
- 3MF only
- Videos only

---

## :material-download: Downloading Files

### Single File

1. Find the file
2. Click **Download**
3. File saves to your computer

### Multiple Files

1. Select files (checkbox)
2. Click **Download Selected**
3. Files download (may be zipped)

---

## :material-folder-zip: ZIP File Uploads

Upload ZIP archives to automatically extract their contents into your library.

### Uploading a ZIP File

1. Click the **Upload** button in the toolbar
2. Select a `.zip` file from your computer
3. The upload modal will detect it's a ZIP file
4. Choose whether to **preserve folder structure** from the ZIP
5. Click **Extract** to upload and extract

### Extraction Options

| Option | Description |
|--------|-------------|
| **Preserve folder structure from ZIP** | Maintains folder hierarchy from inside the ZIP |
| **Create folder from ZIP filename** | Creates a new folder named after the ZIP file (e.g., `MyProject.zip` → `MyProject/`) and extracts all files into it |

!!! tip "Combining Options"
    Both options can be used together. If you enable both, a folder is created from the ZIP filename, and the internal folder structure is preserved inside it.

### What Gets Extracted

- 3MF files with thumbnail and metadata extraction
- Gcode files with print time and filament detection
- Other supported file types

### Progress Indicator

During extraction:

- Progress shows number of files extracted
- Thumbnails and metadata are generated for each file
- Errors are reported if any files fail to extract

!!! tip "Large ZIP Files"
    For ZIP files with many files, extraction may take a moment. The progress indicator shows how many files have been processed.

!!! note "Nested ZIPs"
    ZIP files inside ZIP files are not automatically extracted—they are added as regular files.

---

## :material-cube-outline: STL Thumbnail Generation

Generate preview thumbnails for STL files to make them easier to identify in your library.

### Automatic Generation on Upload

When uploading STL files:

1. Click the **Upload** button
2. Select your STL file(s)
3. Check **Generate thumbnails for STL files** option
4. Click **Upload**

Thumbnails are generated automatically during the upload process.

### Generate for Existing STL Files

For STL files already in your library:

1. Click **Generate Thumbnails** button in the toolbar
2. Select which files to process:
    - **All missing** - Only STL files without thumbnails
    - **Selected files** - Only checked files
    - **Entire folder** - All STL files in current folder
3. Click **Generate**
4. Thumbnails appear as they're created

### Single File Generation

Generate a thumbnail for one file:

1. Find the STL file
2. Click the three-dot menu (:material-dots-vertical:)
3. Select **Generate Thumbnail**
4. The thumbnail updates automatically when done

### ZIP Extraction with Thumbnails

When extracting ZIP files containing STL files:

1. Upload a ZIP file
2. Check **Generate thumbnails for STL files**
3. Thumbnails are created for all STL files in the archive

### Technical Details

| Feature | Details |
|---------|---------|
| **Rendering** | 3D isometric view using trimesh and matplotlib |
| **Color** | Bambu green (#00AE42) model on dark background |
| **Format** | PNG with transparent-compatible background |
| **Size** | Optimized for thumbnail display |

!!! tip "Large STL Files"
    Very complex STL files (100k+ vertices) may take longer to process. The generator handles these gracefully.

!!! note "Supported Formats"
    Both ASCII and binary STL formats are supported.

---

## :material-printer: Print Directly

Print files directly from File Manager with full configuration options.

!!! warning "SD Card Required"
    An SD card must be inserted in your printer for printing and file transfers to work. The file is transferred to the printer's SD card before the print starts.

### Starting a Print

1. Find a sliced file (`.gcode` or `.gcode.3mf`)
2. Click the **printer icon** or right-click for context menu
3. Select **Print**
4. The print modal opens with:
   - **Printer selection** - Choose one or more printers
   - **Plate selection** - For multi-plate 3MF files, select which plate to print
   - **Filament mapping** - Map required filaments to loaded AMS slots
   - **Print options** - Bed levelling, flow calibration, timelapse, etc.
5. Click **Print** to start immediately

### Multi-Printer Printing

Select multiple printers to send the same file to all of them at once—ideal for print farms.

!!! tip "Plate Selection"
    For multi-plate 3MF files (exported as "All sliced file" from the slicer), you'll see a plate selection grid with thumbnails. Select the plate you want to print. When adding to queue, you can select multiple plates at once using checkboxes.

---

## :material-playlist-plus: Add to Queue

Queue sliced files for later printing without creating archives upfront.

### How It Works

When you add a file to the queue:

1. The queue item references the library file directly
2. **No archive is created** until the print actually starts
3. This keeps your Archives clean—only files that were actually printed appear there

### Single File

1. Find a sliced file (`.gcode` or `.gcode.3mf`)
2. Click the **printer icon** or right-click for context menu
3. Select **Add to Queue**
4. Configure:
   - **Printer** - Select target printer(s)
   - **Plate** - For multi-plate files
   - **Filament mapping** - AMS slot configuration
   - **Schedule** - ASAP, scheduled time, or manual start
   - **Print options** - All print settings
5. Click **Add to Queue**

### Multiple Files

1. Select multiple sliced files (checkbox)
2. Click **Add to Queue** in the toolbar
3. Choose a printer
4. All files are queued

!!! tip "Sliced Files Only"
    Only sliced files can be printed or queued. Look for `.gcode` or `.gcode.3mf` extensions, or files with the "sliced" badge.

!!! info "Deferred Archive Creation"
    Archives are created automatically when the print starts, not when you add to queue. This means files that are queued but never printed won't clutter your Archives.

---

## :material-delete: Deleting Files

Deleted files move to the **Trash**. They stay there for a configurable retention window (default **30 days**) before a background sweeper permanently removes them from disk, which gives you an undo window for accidental deletions.

### Single File

1. Find the file
2. Click the **delete** icon
3. Confirm deletion — the file moves to the Trash

### Multiple Files

1. Select files (checkbox)
2. Click **Delete Selected**
3. Confirm deletion — all selected managed files move to the Trash

!!! note "External files bypass Trash"
    Files in external / linked folders skip the trash entirely because their bytes live outside Bambuddy's control and can't be restored. Deleting an external file still just removes Bambuddy's DB record; the file on disk is untouched.

### Restoring or permanently removing trashed files

Open the **Trash** (button in the File Manager header) to see files you've deleted. Regular users see their own trashed files; admins see everyone's.

- **Restore** — moves the file back to its original folder
- **Delete now** — permanently removes the file from disk immediately, bypassing the retention window
- **Empty trash** — hard-deletes every trashed file in your scope

Admins can also change how long trashed files live on the Trash page itself (1–365 days, default 30).

---

## :material-broom: Purge Old Files (admin)

For libraries that have grown into gigabytes, admins get a bulk **Purge old** action in the File Manager header. Pick an age threshold (e.g. "files not printed in 90 days"), see a live preview of how many files would move and how much disk that frees, then confirm — all matching files move to the Trash in one shot, where they sit for the retention window before being permanently deleted.

!!! tip "Use the checkbox"
    The modal includes an **"Include files that have never been printed"** checkbox (on by default). Turn it off to limit the purge to files you've actually printed before — never-printed files fall back to their upload date for age.

External (linked) files are never eligible for purge. The purge respects the standard two-stage delete flow — nothing is destroyed immediately.

The **"Purge old"** button only appears for users holding the new `library:purge` permission, which ships enabled by default on the built-in *Administrators* role. To grant it to an Operator role, add `library:purge` in Settings → Users → Groups.

---

## :material-pencil: Renaming Files & Folders

Rename files and folders directly in the File Manager.

### Renaming a File

**Grid View:**

1. Hover over the file card
2. Click the three-dot menu (:material-dots-vertical:)
3. Select **Rename**
4. Enter the new name
5. Click **Rename** to save

**List View:**

1. Find the file in the list
2. Click the pencil icon (:material-pencil:) in the actions column
3. Enter the new name
4. Click **Rename** to save

### Renaming a Folder

1. Hover over the folder in the sidebar
2. Click the three-dot menu (:material-dots-vertical:)
3. Select **Rename**
4. Enter the new name
5. Click **Rename** to save

!!! note "Filename Restrictions"
    Filenames cannot contain path separators (`/` or `\`). The rename will fail if these characters are included.

---

## :material-folder-network: External Folder Mounting

Mount host directories (NAS shares, USB drives, network storage) into the File Manager without copying files.

### Setting Up an External Folder

**Step 1: Bind-mount the directory into Docker**

Add the host directory as a volume in your `docker-compose.yml`:

```yaml
services:
  bambuddy:
    volumes:
      - /mnt/nas/3d-prints:/external/prints:ro
```

Restart the container after changing volumes.

**Step 2: Link the folder in Bambuddy**

1. Open **File Manager**
2. Click **Link External** in the toolbar
3. Enter a display name (e.g., "NAS Prints")
4. Enter the container path (e.g., `/external/prints`)
5. Choose options:
    - **Read Only** (default) — prevents uploads and deletions
    - **Show hidden files** — includes dotfiles in scan
6. Click **Link Folder**

The folder is automatically scanned and files appear immediately.

### Scanning & Refreshing

External folders are indexed on creation. To pick up new or removed files:

1. Click on the external folder in the sidebar
2. Click the **Scan** button in the info bar
3. New files are added, deleted files are removed from the index

!!! info "Files Are Not Copied"
    Bambuddy indexes external files into its database but reads them directly from the original path. No disk space is used for file copies. Thumbnails for 3MF, STL, and gcode files are generated and stored locally.

### Read-Only Protection

When **Read Only** is enabled (default):

- Uploads to the folder are blocked (403)
- Moving files into the folder is blocked
- ZIP extraction to the folder is blocked
- Files can still be downloaded, printed, and queued

### Deleting External Folders

When you delete an external folder from Bambuddy:

- The database index is removed
- Generated thumbnails are cleaned up
- **The actual files on disk are never deleted**

!!! tip "Docker Volume Permissions"
    Use `:ro` in your Docker volume mount for an extra layer of read-only protection at the filesystem level.

!!! tip "Supported File Types"
    External folder scanning discovers: `.3mf`, `.gcode`, `.stl`, `.obj`, `.step`, `.stp`, and image files (`.png`, `.jpg`, `.gif`, `.webp`, `.svg`).

---

## :material-link: Linking Folders

Link folders to projects or archives for organization:

### Creating Links

1. Hover over an unlinked folder
2. Click the **link icon** that appears
3. Choose to link to a **Project** or **Archive**
4. Select the target from the dropdown
5. Folder shows a colored badge when linked

### Managing Links

- Click the badge on a linked folder to change/remove the link
- Or use the context menu (right-click)

### Link Benefits

- Quick navigation to related projects
- Visual organization with color-coded badges
- Group related files together

---

## :material-refresh: Refreshing

### Manual Refresh

Click **Refresh** to reload the file list.

### Auto-Refresh

File list updates when:

- You navigate directories
- After delete operations
- After adding files to queue

---

## :material-shield-alert: Safety

### Before Deleting

- Ensure files aren't needed for queued prints
- Download backups of important files first
- Deleted files cannot be recovered

---

## :material-api: API Access

Access library files programmatically:

```bash
# List files
GET /api/v1/library/files

# Get file details
GET /api/v1/library/files/{id}

# Upload file
POST /api/v1/library/files/upload

# Extract ZIP file
POST /api/v1/library/files/extract-zip

# Add to queue
POST /api/v1/library/files/add-to-queue

# Delete file
DELETE /api/v1/library/files/{id}

# Create external folder
POST /api/v1/library/folders/external

# Scan external folder
POST /api/v1/library/folders/{id}/scan
```

See [API Reference](../reference/api.md) for details.

---

## :material-cellphone: Mobile & PWA Support

The File Manager is optimized for touch devices and the PWA (Progressive Web App).

### Touch-Friendly Interface

- **Action buttons** are always visible on mobile (no hover required)
- **Selection checkboxes** appear on all file cards for easy multi-select
- **Context menus** accessible via the three-dot button on each card
- **Responsive grid** adjusts columns based on screen size

### PWA Tips

- Add Bambuddy to your home screen for a native app experience
- File Manager works offline for browsing cached files
- Swipe gestures work naturally on touch devices

---

## :material-lightbulb: Tips

!!! tip "Print or Queue"
    Use **Print** for immediate printing with full options, or **Add to Queue** to schedule prints for later.

!!! tip "Multi-Printer Support"
    Select multiple printers to send the same file to your entire print farm at once.

!!! tip "Organize with Links"
    Link folders to projects to keep related files grouped together.

!!! tip "Multi-Select"
    Select multiple files to queue or delete them all at once.

!!! tip "File Badges"
    Look for "sliced" badges to identify files ready for printing.

!!! tip "Rename from Context Menu"
    Right-click any file or folder to access the rename option along with other actions.
