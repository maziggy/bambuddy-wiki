---
title: File Manager
description: Browse and manage files on your printer's storage
---

# File Manager

Browse, download, and manage files on your Bambu Lab printer's internal storage.

---

## :material-folder: Overview

The File Manager lets you:

- **Browse** files on printer storage
- **Download** files to your computer
- **Delete** unwanted files
- **View** file details and metadata

---

## :material-folder-open: Accessing File Manager

1. Go to the **Printers** page
2. Click the settings icon on a printer card
3. Select **File Manager**

Or:

1. Go to printer details
2. Click **File Manager** tab

---

## :material-file-tree: File Browser

### Storage Locations

Bambu Lab printers have internal storage:

| Location | Contents |
|----------|----------|
| `/` | Root directory |
| `/timelapse` | Timelapse videos |
| `/model` | Print files (3MF) |
| `/cache` | Temporary files |

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

## :material-movie: Timelapse Videos

### Viewing Timelapses

1. Navigate to `/timelapse`
2. Click a video to preview
3. Or download to view locally

### Timelapse Format

- Format: MP4
- Resolution: Varies by printer
- Contains print progression

### Managing Timelapses

Timelapses can consume significant storage:

- Download favorites
- Delete old timelapses
- Check storage usage

---

## :material-harddisk: Storage Usage

### Checking Space

View storage usage:

- Total capacity
- Used space
- Free space

### Managing Space

When storage is full:

1. Download important files
2. Delete old files
3. Clear cache folder
4. Remove old timelapses

---

## :material-file-upload: File Upload

!!! note "Upload Not Supported"
    Direct file upload isn't currently supported. Use Bambu Studio or Handy to send files to your printer.

---

## :material-refresh: Refreshing

### Manual Refresh

Click **Refresh** to reload the file list.

### Auto-Refresh

File list updates when:

- You navigate directories
- After delete operations
- After downloads

---

## :material-printer: Per-Printer Files

Each printer has its own storage:

- Files are not shared between printers
- Select printer to view its files
- Downloads are printer-specific

---

## :material-shield-alert: Safety

### Before Deleting

- Ensure files aren't needed
- Download backups first
- Don't delete system files

### System Files

Some files are important for printer operation:

- Don't delete unknown system folders
- Focus on `/timelapse` and `/model`

---

## :material-api: API Access

Access files programmatically:

```bash
# List files
GET /api/v1/printers/{id}/files

# Get file
GET /api/v1/printers/{id}/files/{path}

# Delete file
DELETE /api/v1/printers/{id}/files/{path}
```

See [API Reference](../reference/api.md) for details.

---

## :material-lightbulb: Tips

!!! tip "Regular Cleanup"
    Clear old files periodically to maintain free space.

!!! tip "Archive Timelapses"
    Download timelapses of successful prints before deleting.

!!! tip "Check Before Prints"
    Verify enough free space before starting long prints.

!!! tip "Organize Locally"
    Keep downloaded files organized on your computer.
