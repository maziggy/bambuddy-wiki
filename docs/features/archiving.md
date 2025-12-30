---
title: Print Archiving
description: Automatic 3MF archiving with metadata extraction
---

# Print Archiving

Bambuddy automatically archives every completed print with full metadata, 3D previews, and duplicate detection.

![Archives Page](../assets/archives.png){ .screenshot }

---

## :material-archive: How Archiving Works

When a print completes:

```mermaid
graph LR
    A[Print Completes] --> B[Download 3MF]
    B --> C[Extract Metadata]
    C --> D[Generate Thumbnail]
    D --> E[Check Duplicates]
    E --> F[Save to Archive]
```

### What Gets Archived

| Data | Description |
|------|-------------|
| **3MF File** | Complete print file from printer |
| **Thumbnail** | Preview image from slicer |
| **Metadata** | Print settings, layers, filament, etc. |
| **Print result** | Success, failed, or stopped |
| **Duration** | Actual print time |
| **Filament used** | Grams consumed |
| **Camera snapshot** | Photo at completion (if enabled) |

---

## :material-cube-scan: 3D Model Preview

View your models directly in the browser:

### Three.js Viewer

- **Rotate** - Click and drag
- **Zoom** - Scroll wheel
- **Pan** - Right-click and drag
- **Reset** - Double-click

### Viewer Features

- Wireframe mode
- Solid/transparent modes
- Color from slicer
- Multi-part support

---

## :material-content-duplicate: Duplicate Detection

Bambuddy automatically detects duplicate prints:

### How It Works

1. Extracts file hash from 3MF
2. Compares with existing archives
3. Marks potential duplicates
4. You decide what to keep

### Duplicate Indicators

- Badge showing "Duplicate"
- Link to original print
- Side-by-side comparison available

### Managing Duplicates

- **Keep both** - Sometimes you intentionally reprint
- **Delete duplicate** - Remove the newer one
- **Merge** - Combine statistics from both

---

## :material-card-text: Archive Cards

Each archive displays as a card with key information:

```
┌────────────────────────────────────────┐
│  [Thumbnail]                           │
│                                        │
│  Benchy.3mf                           │
│  Workshop X1C • 2h 15m                │
│  ✓ Success • 45g PLA                  │
│                                        │
│  [Tags] [Project Badge]               │
└────────────────────────────────────────┘
```

### Card Information

- **Thumbnail** - Visual preview
- **Filename** - Original file name
- **Printer** - Which printer completed it
- **Duration** - How long it took
- **Result** - Success, failed, or stopped
- **Filament** - Material and weight used
- **Tags** - Custom labels
- **Project** - Assigned project badge

---

## :material-mouse: Context Menu Actions

Right-click (or long-press on mobile) for quick actions:

| Action | Description |
|--------|-------------|
| :material-printer-3d: **Re-print** | Send to any connected printer |
| :material-compare: **Compare** | Side-by-side with other archives |
| :material-folder-move: **Add to Project** | Assign to a project |
| :material-tag: **Edit Tags** | Add or remove tags |
| :material-pencil: **Edit Details** | Modify name, notes, etc. |
| :material-download: **Download 3MF** | Get the original file |
| :material-delete: **Delete** | Remove from archive |

---

## :material-printer-3d: Re-print with AMS Preview

When re-printing an archive, Bambuddy shows a filament comparison:

![Re-print AMS Mapping](../assets/reprint_ams_mapping.png){ .screenshot }

### What It Shows

| Required (from 3MF) | → | Loaded (in AMS) | Status |
|---------------------|---|-----------------|--------|
| PLA Red (25g) | → | PLA Red (AMS-A Slot 1) | ✓ Match |
| PETG Black (10g) | → | PETG White (AMS-B Slot 2) | ⚠ Color mismatch |
| PLA Blue (5g) | → | TPU (AMS-A Slot 3) | ⚠ Type mismatch |

### Status Indicators

| Icon | Color | Meaning |
|------|-------|---------|
| ✓ | Green | Type and color both match (exact or similar) |
| ⚠ | Yellow | Same type, different color |
| ⚠ | Orange | Different filament type or not loaded |

### Features

- **AMS Slot Labels** - Shows which AMS unit and slot contains the matched filament (e.g., "AMS-B Slot 3")
- **Fuzzy Color Matching** - Colors are matched within a tolerance, so slight hex variations between 3MF and printer data still show as a match
- **Re-read Button** - Refresh AMS status from the printer if you've swapped spools since the modal opened

### How It Works

1. Click **Re-print** on an archive
2. Select target printer
3. Review filament comparison (click **Re-read** if you've changed spools)
4. Proceed or cancel based on results

!!! tip "File Type Badge"
    Archive cards show a **GCODE** (green) or **SOURCE** (orange) badge. Only GCODE files have AMS mapping data - SOURCE files are slicer project files without embedded print settings.

---

## :material-image-multiple: Photo Attachments

Add photos to your archives:

### Camera Snapshot

Automatic camera capture on print completion:

1. Go to **Settings** > **General**
2. Enable **Capture snapshot on print complete**
3. Photos are automatically added to archives

### Manual Photos

Upload photos of your finished prints:

1. Open an archive
2. Click **Add Photo**
3. Upload from your device
4. Photos are stored with the archive

!!! tip "Failure Documentation"
    Add photos of failed prints to help analyze what went wrong.

---

## :material-movie-edit: Timelapse Editor

Edit your timelapse videos directly in Bambuddy:

![Timelapse Editor](../assets/edit_timelapse.png){ .screenshot }

### Opening the Editor

1. Open an archive with a timelapse
2. Click the timelapse to view it
3. Click **Edit** in the viewer header

### Editor Features

| Feature | Description |
|---------|-------------|
| **Trim** | Set start and end points with visual timeline |
| **Speed** | Adjust playback from 0.25x to 4x |
| **Music** | Add audio overlay with volume control |
| **Preview** | Preview changes before saving |

### Timeline Controls

- **Thumbnail strip** - Visual preview of video frames
- **Trim handles** - Drag to set start/end points
- **Playhead** - Shows current position
- **Play/Pause** - Preview trimmed section

### Adding Music

1. Click the **Music** section
2. Upload an audio file (MP3, WAV, M4A, AAC, OGG)
3. Adjust volume with the slider
4. Preview synced with video playback

### Saving Changes

Click **Save** to process the video. The original timelapse will be replaced with the edited version.

!!! note "Processing Time"
    Video processing uses FFmpeg on the server. Longer videos may take a few moments to process.

---

## :material-download: Source 3MF Upload

Upload the original 3MF file for prints started outside Bambuddy:

1. Open an archive
2. Click **Upload 3MF**
3. Select the source file
4. 3MF is stored with the archive

This enables 3D preview and re-printing even for imported archives.

---

## :material-tag: Tags

Organize archives with custom tags:

### Creating Tags

1. Open any archive
2. Click **Edit Tags**
3. Type a new tag name
4. Press Enter to create

### Using Tags

- Filter archives by tag
- Combine multiple tags
- Color-coded in the interface

### Tag Examples

- `functional` - Useful prints
- `decoration` - Decorative items
- `gift` - Prints for others
- `prototype` - Test iterations
- `failed` - Print failures

---

## :material-note-text: Notes

Add notes to archives:

- Design changes
- Print settings tweaks
- Quality observations
- Reference links

Notes are searchable via full-text search.

---

## :material-account: Designer Attribution

Credit the model designer:

1. Open an archive
2. Click **Edit Details**
3. Add **Designer** name
4. Optionally add **Designer URL**

Great for tracking models from Printables, Thingiverse, etc.

---

## :material-filter: Filtering Archives

Find archives quickly:

### Filter Options

| Filter | Description |
|--------|-------------|
| **Printer** | Show only from specific printer |
| **Status** | Success, failed, or stopped |
| **Date range** | Prints within timeframe |
| **Project** | Assigned to project |
| **Tags** | Has specific tags |
| **Material** | Filament type used |

### Combining Filters

Stack multiple filters to narrow results. For example: "PLA prints from Workshop X1C in the last week."

---

## :material-sort: Sorting

Sort archives by:

- **Date** (newest/oldest)
- **Name** (A-Z/Z-A)
- **Duration** (shortest/longest)
- **Printer** (grouped)

---

## :material-lightbulb: Tips

!!! tip "Batch Operations"
    Select multiple archives with Shift+click or Ctrl+click for batch actions.

!!! tip "Quick Search"
    Press ++slash++ to jump to the search box from anywhere.

!!! tip "Project Organization"
    Use projects to group related prints (like "Voron Build" or "Gift Set").

!!! tip "Tag Consistently"
    Develop a consistent tagging system for easy filtering later.
