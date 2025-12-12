---
title: External Links
description: Add custom sidebar links to external tools
---

# External Links

Add custom links to external tools and resources in your Bambuddy sidebar navigation.

---

## :material-link: Overview

External links let you:

- Quick access to related tools
- One-click to Spoolman, OctoPrint, etc.
- Customize your workflow
- Keep everything accessible

---

## :material-plus-circle: Adding a Link

1. Go to **Settings** > **External Links**
2. Click **Add Link**
3. Fill in the details:

| Field | Description |
|-------|-------------|
| **Name** | Display name (e.g., "Spoolman") |
| **URL** | Full URL including protocol |
| **Icon** | Select an icon |
| **Show in Sidebar** | Toggle sidebar visibility |
| **Open in New Tab** | Open link in new tab |

4. Click **Save**

---

## :material-format-list-bulleted: Managing Links

### Reordering

Drag links to change their order in the sidebar:

1. Hover over a link
2. Grab the drag handle
3. Drag to new position
4. Release

### Editing

1. Click the **edit** icon on a link
2. Modify any field
3. Click **Save**

### Deleting

1. Click the **delete** icon
2. Confirm deletion

---

## :material-palette: Icons

### Available Icons

Choose from Material Design icons:

| Icon | Name | Good For |
|:----:|------|----------|
| :material-spool: | Spool | Spoolman, filament |
| :material-printer-3d: | Printer | OctoPrint, Mainsail |
| :material-chart-line: | Chart | Analytics, Grafana |
| :material-cog: | Cog | Settings, config |
| :material-github: | GitHub | Repositories |
| :material-file-document: | Document | Documentation |
| :material-home: | Home | Dashboards |
| :material-link: | Link | Generic links |

### Custom Icons

Additional icons available in the icon picker.

---

## :material-sidebar: Sidebar Display

### Visibility

Toggle **Show in Sidebar** to control visibility:

- :material-check: Shows in sidebar navigation
- :material-close: Hidden (accessible from settings)

### Position

Links appear in the sidebar below the main navigation items.

### Appearance

Links display with:

- Icon (if selected)
- Name
- External indicator (if opens new tab)

---

## :material-lightbulb: Use Cases

### Spoolman

Link to your filament inventory:

| Field | Value |
|-------|-------|
| Name | Spoolman |
| URL | `http://192.168.1.50:7912` |
| Icon | :material-spool: |
| Sidebar | Yes |
| New Tab | Yes |

### OctoPrint / Mainsail

Link to printer web interfaces:

| Field | Value |
|-------|-------|
| Name | Mainsail |
| URL | `http://192.168.1.100` |
| Icon | :material-printer-3d: |
| Sidebar | Yes |
| New Tab | Yes |

### Grafana Dashboard

Link to monitoring dashboards:

| Field | Value |
|-------|-------|
| Name | Grafana |
| URL | `http://grafana:3000/d/printers` |
| Icon | :material-chart-line: |
| Sidebar | Yes |
| New Tab | Yes |

### Documentation

Link to manuals or wikis:

| Field | Value |
|-------|-------|
| Name | Docs |
| URL | `https://github.com/maziggy/bambuddy-wiki` |
| Icon | :material-file-document: |
| Sidebar | Yes |
| New Tab | Yes |

### Printables / Thingiverse

Quick access to model sites:

| Field | Value |
|-------|-------|
| Name | Printables |
| URL | `https://www.printables.com` |
| Icon | :material-cube: |
| Sidebar | Yes |
| New Tab | Yes |

---

## :material-keyboard: Keyboard Shortcuts

External links can be accessed via keyboard:

- Links are assigned numbers after main navigation
- Press the assigned number to navigate

!!! note "Shortcut Assignment"
    External link shortcuts start after built-in page shortcuts.

---

## :material-backup-restore: Backup Integration

External links are included in backups:

- Exported with database backup
- Restored when importing backup
- No manual re-entry needed

[:material-arrow-right: Backup & Restore](backup.md)

---

## :material-lightbulb: Tips

!!! tip "Related Tools"
    Add links to tools you use alongside Bambuddy for quick switching.

!!! tip "Open in New Tab"
    Enable "New Tab" to keep Bambuddy open while visiting links.

!!! tip "Organize by Usage"
    Put most-used links at the top via drag-and-drop.

!!! tip "Keep It Clean"
    Hide rarely-used links from sidebar to reduce clutter.
