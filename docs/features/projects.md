---
title: Projects
description: Group related prints into projects with progress tracking
---

# Projects

Group related prints into projects to track multi-part builds, gift sets, or any collection of related prints.

![Projects Page](../assets/projects.png){ .screenshot }

---

## :material-folder-multiple: What Are Projects?

Projects let you:

- **Group prints** - Organize related archives together
- **Track progress** - See completion status
- **Set targets** - Define how many items needed
- **Track quantities** - Count items per print for batch printing
- **Bill of materials** - Track sourced parts (screws, electronics)
- **File attachments** - Store documentation and references
- **Cost tracking** - Budget and expense tracking
- **Color code** - Visual identification
- **Add notes** - Document the project

### Use Cases

| Project | Description |
|---------|-------------|
| **Voron Build** | All parts for a Voron printer |
| **Gift Set** | Prints for a birthday gift |
| **Home Organizers** | Kitchen and bathroom organizers |
| **Prototypes** | Iterations of a design |
| **Commission** | Client order tracking |

---

## :material-plus-circle: Creating a Project

1. Go to the **Projects** page
2. Click **New Project**
3. Fill in the details:

| Field | Description |
|-------|-------------|
| **Name** | Project name (required) |
| **Description** | What the project is for |
| **Color** | Badge color for identification |
| **Target Plates** | Number of print jobs needed |
| **Target Parts** | Total number of parts/objects needed |

4. Click **Create**

!!! tip "Plates vs Parts"
    - **Plates** = Number of print jobs (each time you click print)
    - **Parts** = Total objects printed (can be multiple per plate)

    Example: A Voron build might need 25 plates to print 150 parts total.

---

## :material-folder-arrow-down: Adding Archives to Projects

### From Archive Card

1. Right-click an archive card
2. Select **Add to Project**
3. Choose the project
4. Archive is linked

### From Archive Details

1. Open an archive
2. Click the **Project** dropdown
3. Select a project
4. Changes save automatically

### Bulk Assignment

Assign multiple archives at once using the multi-select toolbar:

1. Click **Select** button (or Shift+click / Ctrl+click archives)
2. Select the archives you want to assign
3. Click the **Project** button in the bottom toolbar
4. Choose a project from the list
5. All selected archives are assigned instantly

!!! tip "Remove from Project"
    The same modal has a "Remove from project" option to unassign archives in bulk.

---

## :material-progress-check: Progress Tracking

Track how close you are to completing a project with separate progress for plates and parts:

### Dual Progress Bars

When you set both target plates and target parts, you'll see two progress bars:

```
Plates Progress
[████████░░░░░░░░░░░░] 40%
2 of 5 print jobs

Parts Progress
[████████░░░░░░░░░░░░] 40%
10 of 25 parts
```

### Progress States

| State | Description |
|:-----:|-------------|
| :material-circle-outline: | Not started (0%) |
| :material-circle-half-full: | In progress |
| :material-check-circle:{ style="color: #4caf50" } | Complete (100%) |

### Plates Progress

Tracks the number of print jobs completed:

- **Plates Target: 5** with **2 prints done** = **40%**
- Based on archive count (not quantities)

### Parts Progress

Tracks the total number of parts/objects printed:

- **Parts Target: 25** with **10 parts printed** = **40%**
- Sum of all archive quantities

### Auto-Detection from 3MF

When a print is archived, BamBuddy automatically detects how many parts/objects were on the plate:

- Reads `slice_info.config` from the 3MF file
- Counts non-skipped objects
- Sets the archive quantity automatically

!!! example "Auto-Detection Example"
    You print a plate with 4 copies of a bracket:

    - BamBuddy detects 4 objects in the 3MF
    - Archive quantity is set to 4 automatically
    - Project parts count increases by 4

### Manual Quantity Override

You can still manually adjust the quantity:

1. Open the archive in edit mode
2. Set **Items Printed** to the correct number
3. Project progress updates automatically

### Updating Existing Archives

If you have existing archives with quantity=1, run the migration script:

```bash
# Preview changes
python scripts/update_archive_quantities.py --dry-run

# Apply changes
python scripts/update_archive_quantities.py
```

This re-parses all 3MF files to extract the correct object counts.

---

## :material-palette: Color Coding

Each project has a color-coded badge:

### Available Colors

- :material-circle:{ style="color: #f44336" } Red
- :material-circle:{ style="color: #ff9800" } Orange
- :material-circle:{ style="color: #ffeb3b" } Yellow
- :material-circle:{ style="color: #4caf50" } Green
- :material-circle:{ style="color: #2196f3" } Blue
- :material-circle:{ style="color: #9c27b0" } Purple
- :material-circle:{ style="color: #607d8b" } Gray

### Badge Display

Project badges appear on:

- Archive cards
- Archive list view
- Project overview

---

## :material-view-dashboard: Project Cards

Each project displays as a card with plates and parts stats:

```
┌────────────────────────────────────────┐
│  [Color] Voron Build          150/200  │
│                                parts   │
│  Building a Voron 2.4r2 printer       │
│                                        │
│  Plates [████████░░░░] 40%            │
│  20/50 print jobs                      │
│                                        │
│  Parts  [████████████░░░] 75%         │
│  150/200 parts                         │
│                                        │
│  📦 20 plates  🎯 150 parts            │
└────────────────────────────────────────┘
```

### Card Information

- **Color badge** - Visual identification
- **Name** - Project title
- **Progress badge** - Parts completed / target
- **Plates progress** - Print jobs completed
- **Parts progress** - Objects printed
- **Stats footer** - Quick view of plates and parts count

---

## :material-pencil: Editing Projects

1. Click the **edit** icon on a project card
2. Modify any field
3. Click **Save**

### Editable Fields

- Name
- Description
- Color
- Target Plates (print jobs)
- Target Parts (objects)

---

## :material-filter: Filtering by Project

Filter archives to show only those in a project:

1. Go to **Archives** page
2. Click the **Project** filter
3. Select a project
4. Only linked archives are shown

---

## :material-delete: Deleting Projects

Deleting a project:

1. Click the **delete** icon on a project card
2. Confirm deletion

!!! note "Archives Preserved"
    Deleting a project does NOT delete the archives. They remain in your archive but are no longer linked to a project.

---

## :material-export: Import & Export

Share projects between Bambuddy instances or backup your project data.

### Exporting a Single Project (ZIP)

Export a complete project bundle including all files from linked library folders:

1. Open the project detail page
2. Click the **Export** button
3. A ZIP file is downloaded containing:
   - `project.json` - Project settings, BOM items, and folder metadata
   - `files/` - All files from linked library folders

!!! tip "Complete Bundles"
    ZIP export includes the actual 3MF/gcode files from linked folders, making it easy to share complete project bundles.

### Exporting All Projects (JSON)

Export all projects as metadata only (no files):

1. Go to the **Projects** page
2. Click the **Export** button in the header
3. A JSON file is downloaded with all project metadata

!!! info "Metadata Only"
    Bulk export is JSON format without files. Use single-project ZIP export if you need the files.

### Importing Projects

Import projects from ZIP or JSON files:

1. Go to the **Projects** page
2. Click the **Import** button
3. Select a `.zip` or `.json` file

**ZIP Import:**

- Creates the project with all settings
- Creates linked library folders
- Extracts and stores all files from the ZIP
- Files are available in File Manager

**JSON Import:**

- Creates the project with all settings
- Creates empty linked folders (names only)
- Supports single project or batch import (array of projects)

!!! example "Use Cases"
    - **Share project templates** - Export a project setup for others to use
    - **Migrate instances** - Move projects between Bambuddy servers
    - **Backup projects** - Archive important project bundles with all files
    - **Collaborate** - Share complete project bundles with team members

---

## :material-archive-arrow-up: Project Archives View

Click a project card to see all its archives:

![Project Detail View](../assets/project-detail-1.png){ .screenshot }

- Grid view of all linked prints
- Same filtering and sorting as main Archives
- Quick access to add more prints

---

## :material-clipboard-list: Bill of Materials

Track non-printed parts needed for your project (screws, electronics, hardware):

### Adding BOM Items

1. Open a project
2. Scroll to **Bill of Materials**
3. Click **Add Item**
4. Fill in the details:

| Field | Description |
|-------|-------------|
| **Name** | Part name (e.g., "M3x8 SHCS") |
| **Quantity** | How many needed |
| **Unit Price** | Cost per item |
| **Sourcing URL** | Where to buy |
| **Remarks** | Additional notes |

### Tracking Acquisition

- Check the box when a part is acquired
- Progress bar shows acquired/total items
- Use **Hide done** to focus on remaining items

---

## :material-paperclip: File Attachments

Attach reference files to your project:

### Supported File Types

| Category | Extensions |
|----------|------------|
| **Images** | JPG, PNG, GIF, WebP, SVG |
| **Documents** | PDF, DOC, DOCX, TXT, MD |
| **3D Files** | STL, OBJ, 3MF, STEP, F3D, SCAD |
| **Archives** | ZIP, RAR, 7Z, TAR, GZ |
| **Scripts/Configs** | PY, SH, CFG, GCODE, INI |
| **Data** | JSON, XML, YAML |

### Uploading Files

1. Open a project
2. Scroll to **Attachments**
3. Click **Upload** or drag and drop
4. Files are stored with the project

---

## :material-currency-usd: Cost Tracking

Track project expenses:

### Cost Categories

- **Parts cost** - Sum of BOM item prices
- **Additional costs** - Manual entries for other expenses

### Currency

Uses your configured currency from Settings → General.

---

## :material-lightbulb: Project Ideas

!!! tip "Multi-Part Prints"
    Create a project for complex models that print in multiple pieces. Track which parts are done.

!!! tip "Gift Planning"
    Start a project before a birthday or holiday. Add prints as you complete them.

!!! tip "Printer Builds"
    Building a Voron, Trident, or other kit? Track every printed part.

!!! tip "Iterative Design"
    Group prototypes of the same design to see your progression.

!!! tip "Client Orders"
    Track prints for commission work or client requests.
