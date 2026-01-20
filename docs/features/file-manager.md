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
- **Download** files to your computer
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

---

## :material-navigation: Navigation

### Browsing

- Click folders to enter
- Click ← to go back
- Click root to return home

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

## :material-printer: Print Directly

Print files directly from File Manager with full configuration options.

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
    For multi-plate 3MF files (exported as "All sliced file" from the slicer), you'll see a plate selection grid with thumbnails. Select the plate you want to print.

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

### Single File

1. Find the file
2. Click the **delete** icon
3. Confirm deletion

### Multiple Files

1. Select files (checkbox)
2. Click **Delete Selected**
3. Confirm deletion

!!! warning "No Recovery"
    Deleted files cannot be recovered. Download important files first.

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

# Add to queue
POST /api/v1/library/files/add-to-queue

# Delete file
DELETE /api/v1/library/files/{id}
```

See [API Reference](../reference/api.md) for details.

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
