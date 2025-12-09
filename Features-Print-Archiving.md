# Print Archiving

Bambuddy automatically archives every print with full metadata extraction, creating a searchable history of all your 3D prints.

## How It Works

When a print completes on any connected printer:

1. **Detection**: Bambuddy detects the print completion via MQTT
2. **Download**: The 3MF file is downloaded from the printer via FTPS
3. **Extraction**: Metadata is extracted from the 3MF (layers, filament, settings)
4. **Storage**: Everything is saved to the `archive/` directory
5. **Display**: The print appears in your Archives with a thumbnail

## What Gets Archived

### Files
- **3MF File**: Complete print file (can be re-printed later)
- **Thumbnail**: Preview image from the slicer
- **Timelapse**: Video if enabled on printer (auto-attached or manual)
- **Finish Photo**: Automatic camera capture on completion

### Metadata
| Field | Description |
|-------|-------------|
| Print Time | Estimated vs actual duration |
| Filament | Material type, color, weight used |
| Layers | Total count and layer height |
| Temperatures | Nozzle and bed temperatures |
| Printer | Which printer completed the job |
| Result | Success, failed, or stopped |
| Colors | Multi-color prints show color swatches |

---

## Archive Features

### 3D Model Preview
Click any archive to open an interactive Three.js viewer:
- Rotate, zoom, and pan the model
- View from different angles
- See the actual geometry that was printed

### Duplicate Detection
Bambuddy automatically detects when you've printed the same model before:
- **SHA256 hash** matches exact file content
- **Purple badge** indicates duplicates
- **"Duplicates" filter** shows all duplicate prints
- View duplicate history in archive details

### Print Time Accuracy
Compare estimated vs actual print times:
- **Green badge**: Accurate (within 5%)
- **Blue badge**: Faster than estimated
- **Orange badge**: Slower than estimated
- Per-printer accuracy statistics available

### Photo Attachments
Document your prints with photos:
- **Automatic finish photo**: Camera captures when print completes
- **Manual uploads**: Add your own photos
- Multiple photos per archive supported

### Failure Analysis
When a print fails, document what went wrong:
- Add failure notes
- Attach photos of the failure
- Track failure patterns over time

---

## Managing Archives

### Filtering & Search
- **By Printer**: Show prints from specific printers
- **By Date**: Filter by time range
- **By Status**: Success, failed, stopped
- **By Collection**: Custom collections you create
- **Search**: Find by filename

### Context Menu Actions
Right-click any archive card for:
- **Re-print**: Send to any connected printer
- **Schedule**: Add to print queue
- **Project Page**: View/edit MakerWorld metadata
- **Scan for Timelapse**: Find matching timelapse
- **Delete**: Remove from archive

### Collections
Organize archives into custom collections:
1. Right-click an archive
2. Select "Add to Collection"
3. Create or select a collection
4. Filter archives by collection

---

## Project Page Editor

3MF files from MakerWorld contain embedded project pages with:
- Model title and description
- Designer information
- License details
- Preview images

### Viewing Project Pages
1. Right-click an archive
2. Select "Project Page"
3. View all embedded information

### Editing Project Pages
1. Open the project page
2. Click "Edit"
3. Modify title, description, or other fields
4. Click "Save" (changes are saved to the 3MF file)

---

## Re-printing Archives

Send any archived print back to a printer:

1. Right-click the archive
2. Select "Re-print" or "Schedule"
3. Choose a target printer
4. Optionally set a scheduled time
5. Confirm to start the print

The 3MF file is uploaded via FTP and the print starts automatically.

---

## Storage

Archives are stored in the `archive/` directory:
```
archive/
├── {printer_id}/
│   ├── {archive_id}/
│   │   ├── model.3mf
│   │   ├── thumbnail.png
│   │   ├── timelapse.mp4 (if available)
│   │   └── photos/
│   │       └── finish.jpg
```

The SQLite database (`bambuddy.db`) stores metadata and indexes.

---

## Tips

- **Enable timelapse** on your printer to automatically capture print videos
- **Enable camera capture** in settings for automatic finish photos
- Use **collections** to organize prints by project or client
- Check **duplicate detection** before reprinting to see previous results
