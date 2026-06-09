---
title: Print Log
description: Chronological table view of all print activity with filtering and search
---

# Print Log

The Print Log provides a chronological table of all print activity across your printers, giving you a quick at-a-glance history of every print.

---

## :material-clipboard-list: Overview

The Print Log is a dedicated view mode within the Archives page that focuses on print activity as a flat, filterable table. Unlike the standard archive views (grid, list, calendar), the Print Log shows one row per print event with key details at a glance.

### Accessing the Print Log

1. Go to the **Archives** page
2. Click the :material-clipboard-list: **Print Log** button in the header toolbar (next to the grid/list/calendar view toggles)

---

## :material-table: Table Columns

Each row in the Print Log represents a single print event:

| Column | Description |
|--------|-------------|
| **Date/Time** | When the print completed |
| **Print Name** | Name of the print (with thumbnail) |
| **Printer** | Which printer ran the job |
| **User** | Username who started the print (when authentication is enabled) |
| **Status** | Result badge (color-coded), with the failure-cause classification shown underneath when set |
| **Duration** | Total print time |
| **Filament** | Material type and color indicator |
| **Actions** | Per-row :material-pencil: edit and :material-delete: delete buttons (see [Per-row actions](#per-row-actions)) |

### Status Badges

| Status | Color | Description |
|--------|-------|-------------|
| **Completed** | Green | Print finished successfully |
| **Failed** | Red | Print encountered an error |
| **Stopped** | Orange | Print was manually stopped |
| **Cancelled** | Grey | Print was cancelled before finishing |
| **Skipped** | Purple | Print was skipped in the queue |

---

## :material-filter: Filtering

Narrow down log entries using the filter controls at the top of the table:

| Filter | Description |
|--------|-------------|
| **Search** | Free-text search across print names |
| **Printer** | Show entries from a specific printer |
| **User** | Show entries from a specific user |
| **Status** | Filter by result (completed, failed, stopped, cancelled, skipped) |
| **Date From** | Show entries from this date onward |
| **Date To** | Show entries up to this date |

### Combining Filters

All filters can be combined. For example, show only failed prints from "Workshop X1C" in the last week by selecting the printer, status, and date range together.

---

## :material-book-open-page-variant: Pagination

The Print Log supports configurable pagination for navigating large histories:

| Setting | Options |
|---------|---------|
| **Rows per page** | 10, 25, 50, or 100 |
| **Navigation** | Previous / Next page buttons |

The current page range and total entry count are displayed alongside the navigation controls.

---

## :material-cursor-pointer: Per-row actions

Every row in the Print Log has two action buttons on the right:

### :material-pencil: Edit (classification)

Click the pencil icon to classify the print run -- set or change the **failure cause** and **status** on a single entry.

| Field | Values |
|-------|--------|
| **Status** | Completed, Failed, Stopped, Cancelled, Skipped |
| **Failure cause** | Adhesion failure, Spaghetti / Detached, Layer shift, Clogged nozzle, Filament runout, Warping, Stringing, Under-extrusion, Power failure, User cancelled, Other |

The classification you pick here is what the **Failure Analysis** widget groups by, so updates flow into the dashboard charts immediately.

!!! tip "Works on orphan entries too"
    If a print failed before any archive was created (for example, a dispatch error or a manual log entry), there is no archive to edit through -- the Print Log row editor is the only way to classify those runs.

### :material-delete: Delete (single row)

Click the trash icon to remove a single entry from the log. The entry's filament, time, and cost contributions drop out of Quick Stats in the same response cycle. The matching archive (if any) is not affected -- the log row is independent of the archive.

!!! note "Permission gating"
    Both buttons honour the same ownership shape as archive edits. With `ARCHIVES_UPDATE_OWN` / `ARCHIVES_DELETE_OWN` you can only act on entries you created; `ARCHIVES_UPDATE_ALL` / `ARCHIVES_DELETE_ALL` see every entry.

---

## :material-delete-sweep: Clearing the Log

The **Clear** button removes all log entries from the database.

!!! warning "Permanent Action"
    Clearing the log permanently deletes all log entries. This action cannot be undone.

!!! note "Archives Are Not Affected"
    The Print Log is stored independently from your archives and print queue. Clearing the log only removes log entries -- your archived 3MF files, metadata, and queue history remain untouched.

---

## :material-database: How It Works

### Automatic Logging

Log entries are written automatically whenever a print completes. No configuration is required -- once archiving is active, print events are recorded in the log.

### Independent Storage

The Print Log uses its own database table (`print_log_entries`), separate from the archives table. This means:

- Log entries exist independently of archives
- Deleting an archive does not remove its log entry
- Clearing the log does not affect archives
- The log is lightweight and optimized for fast querying

---

## :material-lock: Permissions

When authentication is enabled, the Print Log respects the following permissions:

| Action | Required Permission |
|--------|-------------------|
| **View** the Print Log | `ARCHIVES_READ` |
| **Edit** classification on a row you own | `ARCHIVES_UPDATE_OWN` |
| **Edit** classification on any row | `ARCHIVES_UPDATE_ALL` |
| **Delete** a single row you own | `ARCHIVES_DELETE_OWN` |
| **Delete** any single row | `ARCHIVES_DELETE_ALL` |
| **Clear** all log entries | `ARCHIVES_DELETE_ALL` |

---

## :material-api: API Access

### List Log Entries

```http
GET /api/v1/print-log/
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search print names |
| `printer_id` | integer | Filter by printer ID |
| `created_by_username` | string | Filter by username |
| `status` | string | Filter by status (completed, failed, stopped, cancelled, skipped) |
| `date_from` | datetime | Start of date range |
| `date_to` | datetime | End of date range |
| `limit` | integer | Number of entries per page (default 25) |
| `offset` | integer | Pagination offset |

**Example:**

```bash
curl -H "X-API-Key: your-key" \
  "http://localhost:8000/api/v1/print-log/?status=failed&limit=10"
```

### Update a Single Entry

```http
PATCH /api/v1/print-log/{entry_id}
```

Edit the classification (failure cause and/or status) on one entry. Requires `ARCHIVES_UPDATE_OWN` for entries you created, or `ARCHIVES_UPDATE_ALL` for any entry.

**Request body:**

```json
{
  "failure_reason": "cloggedNozzle",
  "status": "failed"
}
```

Both fields are optional. To clear an existing classification send `"failure_reason": ""`.

| Field | Valid values |
|-------|--------------|
| `failure_reason` | `""` (clear), `adhesionFailure`, `spaghettiDetached`, `layerShift`, `cloggedNozzle`, `filamentRunout`, `warping`, `stringing`, `underExtrusion`, `powerFailure`, `userCancelled`, `other` |
| `status` | `completed`, `failed`, `stopped`, `cancelled`, `skipped` |

Unknown values return HTTP 400 -- the canonical vocabulary above is enforced server-side.

**Example:**

```bash
curl -X PATCH -H "X-API-Key: your-key" -H "Content-Type: application/json" \
  -d '{"failure_reason": "cloggedNozzle"}' \
  "http://localhost:8000/api/v1/print-log/42"
```

### Delete a Single Entry

```http
DELETE /api/v1/print-log/{entry_id}
```

Removes one entry from the log. The entry's filament, time, and cost contributions drop out of `/api/v1/archives/stats` on the same request cycle. The linked archive (if any) is untouched. Requires `ARCHIVES_DELETE_OWN` for entries you created, or `ARCHIVES_DELETE_ALL` for any entry.

**Example:**

```bash
curl -X DELETE -H "X-API-Key: your-key" \
  "http://localhost:8000/api/v1/print-log/42"
```

### Clear All Log Entries

```http
DELETE /api/v1/print-log/
```

Removes all log entries. Requires `ARCHIVES_DELETE_ALL` permission.

**Example:**

```bash
curl -X DELETE -H "X-API-Key: your-key" \
  "http://localhost:8000/api/v1/print-log/"
```

---

## :material-lightbulb: Tips

!!! tip "Quick Status Check"
    Use the status filter to quickly find all failed prints and investigate patterns.

!!! tip "Date Range Analysis"
    Set a date range to review a specific time period, such as "last week" or "this month."

!!! tip "Per-User Tracking"
    When authentication is enabled, filter by user to see each person's print activity.

!!! tip "Lightweight History"
    The Print Log loads faster than the full archive views since it only fetches tabular data without thumbnails and 3MF metadata.
