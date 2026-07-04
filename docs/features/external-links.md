---
title: Sidebar Customization
description: Customize sidebar pages, external links, ordering, visibility, and default navigation
---

# Sidebar Customization

Customize the Bambuddy sidebar by showing or hiding built-in pages, reordering navigation items, and adding custom links to external tools and resources.

---

## :material-page-layout-sidebar-left: Overview

Sidebar customization lets you:

- Keep your most-used Bambuddy pages near the top
- Hide built-in pages you do not use often
- Add one-click links to Spoolman, OctoPrint, Grafana, and other tools
- Set a default sidebar layout for other users

---

## :material-view-dashboard: Built-in Pages

Built-in Bambuddy pages can be shown, hidden, and reordered from the sidebar settings.

1. Go to **Settings** > **General**
2. Open the **Sidebar** pane
3. Toggle pages on or off
4. Drag visible pages into your preferred order

Hidden pages are removed from the sidebar navigation. Use this to keep the sidebar focused on the pages you use day to day.

Hidden pages remain accessible by direct URL and from any in-app links that point to them. **Settings** cannot be hidden, so you can always return to the sidebar settings and show pages again.

---

## :material-link: External Links

External links add custom sidebar entries for tools and resources outside Bambuddy.

### Adding a Link

1. Go to **Settings** > **External Links**
2. Click **Add Link**
3. Fill in the details:

| Field | Description |
|-------|-------------|
| **Name** | Display name (e.g., "Spoolman") |
| **URL** | Full URL including protocol |
| **Icon** | Select an icon |
| **Open in New Tab** | Open link in new tab |

4. Click **Save**

### Editing

1. Click the **edit** icon on a link
2. Modify any field
3. Click **Save**

### Deleting

1. Click the **delete** icon
2. Confirm deletion

### Icons

Choose from Material Design icons:

| Icon | Name | Good For |
|:----:|------|----------|
| :material-cylinder: | Spool | Spoolman, filament |
| :material-printer-3d: | Printer | OctoPrint, Mainsail |
| :material-chart-line: | Chart | Analytics, Grafana |
| :material-cog: | Cog | Settings, config |
| :material-github: | GitHub | Repositories |
| :material-file-document: | Document | Documentation |
| :material-home: | Home | Dashboards |
| :material-link: | Link | Generic links |

Additional icons available in the icon picker.

---

## :material-format-list-bulleted: Ordering and Visibility

### Reordering

Built-in pages and external links can be reordered together from the sidebar settings.

1. Go to **Settings** > **General**
2. Open the **Sidebar** pane
3. Drag an item to a new position
4. Release to save the new order

### Visibility

Use sidebar visibility controls to decide what appears in navigation:

- Built-in pages can be shown or hidden from the sidebar settings
- **Settings** is always visible and cannot be hidden
- Hidden built-in pages stay available by direct URL and from in-app links
- External links cannot be hidden; delete an external link to remove it from the sidebar

### External Link Appearance

External links display with:

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
| Icon | :material-database-sync-outline: |
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

## :material-sort: Admin Default Sidebar Order

Admins can set a default sidebar navigation layout that applies to users who have not customized their own sidebar. The default captures the saved order for built-in pages and external links, plus the visibility state for built-in pages.

### Setting a Default

1. Arrange sidebar items in your preferred order
2. Show or hide the built-in pages you want included
3. Go to **Settings** > **General** > **Sidebar Order**
4. Toggle **Set Default** on

The current sidebar layout is saved as the default for all users.

### How It Works

| Scenario | Behavior |
|----------|----------|
| **New user** | Automatically gets the admin-set sidebar layout |
| **User hasn't customized** | Gets the admin-set layout on next page load |
| **User has customized** | Keeps their own layout (admin default is not re-applied) |
| **Admin toggles off** | Default is cleared; existing user layouts are unchanged |

!!! note "Permission Required"
    The Set Default toggle is only visible to users with **Settings: Update** permission (`settings:update`).

!!! tip "One-Time Application"
    The default layout is applied once per user. If you update the default, users who already received a previous default will not be affected. To re-apply it, users can click **Reset** on their sidebar order first.

---

## :material-keyboard: Keyboard Shortcuts

Visible sidebar entries can be accessed via keyboard:

- Sidebar items are assigned number shortcuts
- Press the assigned number to navigate

!!! note "Shortcut Assignment"
    Shortcut assignment follows the current sidebar layout. Reordering items or hiding built-in pages updates the number shortcuts automatically.

---

## :material-backup-restore: Backup Integration

Sidebar customization is included in backups:

- External links are exported with database backups
- Sidebar order and built-in page visibility settings are restored when importing a backup
- No manual re-entry needed after restore

[:material-arrow-right: Backup & Restore](backup.md)

---

## :material-lightbulb: Tips

!!! tip "Related Tools"
    Add links to tools you use alongside Bambuddy for quick switching.

!!! tip "Open in New Tab"
    Enable "New Tab" to keep Bambuddy open while visiting links.

!!! tip "Organize by Usage"
    Put most-used pages and links at the top via drag-and-drop.

!!! tip "Keep It Clean"
    Hide rarely-used pages and links from the sidebar to reduce clutter.
