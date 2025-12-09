# File Manager

Browse and manage files on your printer's internal storage.

## Overview

The File Manager allows you to:
- Browse files on printer's SD card
- View file details and thumbnails
- Delete unwanted files
- Free up storage space

---

## Accessing File Manager

1. Go to **Printers** page
2. Click on a connected printer
3. Select **File Manager** or click the folder icon

---

## File Browser

### Navigation
- **Folder tree**: Navigate directory structure
- **Breadcrumbs**: Quick path navigation
- **Back button**: Return to parent folder

### File List
Each file shows:
- **Thumbnail**: Preview image (for 3MF files)
- **Name**: Filename
- **Size**: File size
- **Date**: Last modified date
- **Type**: File type icon

### Sorting
Sort files by:
- Name (A-Z, Z-A)
- Date (newest, oldest)
- Size (largest, smallest)

---

## File Types

### 3MF Files
Print files with:
- Model geometry
- Slicer settings
- Thumbnails
- Metadata

### Timelapse Videos
Recorded print timelapses:
- MP4 format
- Stored in timelapse folder
- Can be downloaded or deleted

### Other Files
- Gcode files (legacy)
- Log files
- Cache files

---

## Managing Files

### Viewing Details
Click a file to see:
- Full filename
- File size
- Created/modified date
- Preview (if available)

### Deleting Files
1. Select file(s) to delete
2. Click **Delete**
3. Confirm deletion

> Deleted files cannot be recovered!

### Downloading Files
1. Click on a file
2. Select **Download**
3. File downloads to your computer

---

## Storage Information

### Space Usage
View storage status:
- Total capacity
- Used space
- Available space
- Usage percentage

### Low Storage Warning
Bambuddy warns when storage is low:
- Yellow warning at 80% full
- Red warning at 95% full

---

## Common Operations

### Freeing Up Space
To free storage:
1. Delete old timelapse videos
2. Remove failed print files
3. Clear cache files
4. Delete prints you won't reprint

### Finding Large Files
1. Sort by size (largest first)
2. Identify large files to remove
3. Timelapses are usually the largest

### Cleaning Up Timelapses
Timelapses consume significant storage:
1. Navigate to timelapse folder
2. Sort by date
3. Delete old timelapses you don't need

---

## FTP Connection

File Manager uses FTPS to connect:
- Port 990 (implicit FTPS)
- Encrypted connection
- Same credentials as printer connection

### Connection Issues
If File Manager won't connect:
- Verify printer is online
- Check FTP port (990) is accessible
- Ensure LAN Mode is enabled
- Try reconnecting the printer

---

## Limitations

### Read-Only Folders
Some folders are system-protected:
- Cannot delete system files
- Cannot modify firmware files

### File Upload
Currently not supported:
- Use Bambu Studio for uploads
- Or use the print queue feature

### Simultaneous Access
Only one FTP connection at a time:
- Close other FTP clients
- Bambu Studio may hold connection
- Wait and retry if busy

---

## Tips

- Regularly clean up timelapses to save space
- Download important files for backup
- Delete failed print files promptly
- Check storage before starting large prints
- Keep 20%+ storage free for new prints
