---
title: Export
description: Export archives and statistics to CSV or Excel
---

# Export

Export your print data for analysis, reporting, or backup purposes.

---

## :material-file-export: Export Options

### Export Formats

| Format | Extension | Best For |
|--------|-----------|----------|
| **CSV** | .csv | Spreadsheets, data analysis, importing |
| **Excel** | .xlsx | Reports, formatting, charts |

---

## :material-archive: Archive Export

Export your print archive data:

### How to Export

1. Go to **Archives** page
2. Apply any desired filters
3. Click **Export** button
4. Choose format (CSV or Excel)
5. Download begins

### Exported Fields

| Field | Description |
|-------|-------------|
| Name | Print name |
| Filename | Original 3MF filename |
| Printer | Which printer completed it |
| Date | Completion date/time |
| Duration | Print time |
| Result | Success/Failed/Stopped |
| Filament Type | Material used |
| Filament Used | Grams consumed |
| Filament Cost | Cost of material |
| Energy | Wh consumed (if tracked) |
| Energy Cost | Electricity cost |
| Total Cost | Filament + energy |
| Tags | Assigned tags |
| Project | Assigned project |
| Designer | Model designer |
| Notes | Archive notes |

### Filter-Aware Export

Export respects your current filters:

- **Printer filter** - Only selected printer
- **Date range** - Only within range
- **Status filter** - Only selected status
- **Project filter** - Only in project
- **Search query** - Only matching results

---

## :material-chart-bar: Statistics Export

Export statistics data:

### How to Export

1. Go to **Statistics** page
2. Set your time range and filters
3. Click **Export** button
4. Choose format
5. Download begins

### Exported Data

| Data | Description |
|------|-------------|
| Summary stats | Total prints, success rate, etc. |
| Per-printer stats | Breakdown by printer |
| Material usage | Filament consumption |
| Cost breakdown | By printer, material |
| Time analysis | Duration statistics |
| Daily/weekly counts | Activity over time |

---

## :material-cog: Export Settings

### Date Format

Dates are exported in ISO 8601 format:

```
2024-01-15T14:30:00Z
```

### Number Format

- Decimals use period (.)
- No thousands separator
- Suitable for most spreadsheets

### Text Encoding

- UTF-8 encoding
- Handles international characters
- Compatible with all modern software

---

## :material-microsoft-excel: Working with Excel

### Opening CSV in Excel

1. Open Excel
2. Go to **Data** > **From Text/CSV**
3. Select the downloaded file
4. Choose **UTF-8** encoding
5. Click **Load**

!!! warning "Direct Open May Fail"
    Double-clicking CSV files may cause encoding issues. Use the import method above.

### Creating Charts

1. Select your data
2. Go to **Insert** > **Charts**
3. Choose chart type
4. Customize as needed

---

## :material-google: Working with Google Sheets

### Importing CSV

1. Open Google Sheets
2. Go to **File** > **Import**
3. Upload the CSV file
4. Choose **Replace spreadsheet** or **Insert new sheet**
5. Click **Import data**

### Direct Link (Advanced)

If you host Bambuddy publicly, you can link directly to export URLs.

---

## :material-code-json: API Export

Export via the REST API:

### Archive Export

```bash
GET /api/v1/archives/export?format=csv
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| `format` | csv or xlsx |
| `printer_id` | Filter by printer |
| `status` | Filter by status |
| `start_date` | Filter start |
| `end_date` | Filter end |
| `search` | Search query |

### Example

```bash
curl -H "X-API-Key: your-key" \
  "http://localhost:8000/api/v1/archives/export?format=csv&status=success" \
  -o archives.csv
```

---

## :material-calendar-export: Scheduled Exports

For regular exports, set up a cron job:

```bash
# Weekly export every Sunday at midnight
0 0 * * 0 curl -H "X-API-Key: your-key" \
  "http://localhost:8000/api/v1/archives/export?format=csv" \
  -o "/backup/archives_$(date +\%Y\%m\%d).csv"
```

---

## :material-backup-restore: Backup Use

Export can serve as a partial backup:

### What Export Captures

- :material-check: Print metadata
- :material-check: Statistics
- :material-check: Tags and notes

### What Export Doesn't Capture

- :material-close: 3MF files
- :material-close: Thumbnails
- :material-close: Settings
- :material-close: Notification configs

For full backups, use the [Backup & Restore](backup.md) feature.

---

## :material-lightbulb: Tips

!!! tip "Regular Exports"
    Export monthly for record keeping, even if you have database backups.

!!! tip "Filter First"
    Apply filters before exporting to get exactly the data you need.

!!! tip "Version Control"
    Include dates in export filenames to track versions.

!!! tip "External Analysis"
    Export to CSV for analysis in Python, R, or other tools.

!!! tip "Reporting"
    Use Excel format for formatted reports with charts and branding.
