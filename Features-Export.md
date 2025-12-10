# CSV/Excel Export

Export your print archives and statistics for analysis in spreadsheet applications, backup purposes, or reporting.

## Overview

- **Multiple formats** — CSV and Excel (.xlsx) support
- **Filtered exports** — Export respects current filters
- **Statistics export** — Export dashboard data
- **Archives export** — Export print history

## Exporting Archives

### From Archives Page

1. Navigate to **Archives**
2. Apply any desired filters (printer, material, tags, etc.)
3. Click the **Export** button
4. Choose format:
   - **CSV** — Universal format, smaller file size
   - **Excel** — Formatted spreadsheet with headers

### Exported Fields

| Field | Description |
|-------|-------------|
| ID | Archive unique identifier |
| Print Name | Name of the print |
| Filename | Original file name |
| Printer | Printer used |
| Status | completed/failed |
| Print Time | Estimated time |
| Actual Time | Real print duration |
| Filament Used | Grams of filament |
| Filament Type | PLA, PETG, etc. |
| Tags | Comma-separated tags |
| Notes | User notes |
| Designer | Model designer |
| Created | Archive date |
| Project | Assigned project name |

## Exporting Statistics

### From Dashboard

1. Navigate to **Dashboard**
2. Click **Export Stats**
3. Choose format (CSV or Excel)

### Exported Statistics

- Print totals by status
- Filament usage by type
- Print time totals
- Success/failure rates
- Cost calculations
- Printer utilization

## Filtered Exports

Exports respect active filters:

| Filter | Effect on Export |
|--------|------------------|
| Printer | Only archives from selected printer |
| Material | Only archives using selected material |
| Color | Only archives with selected colors |
| Tags | Only archives with selected tag |
| Favorites | Only favorite archives |
| Date Range | Only archives within range |
| Search | Only archives matching search term |

## Excel Features

Excel exports include:
- Formatted headers
- Auto-fitted column widths
- Data type formatting (dates, numbers)
- Ready for pivot tables and charts

## Use Cases

### Backup
Export all archives periodically as a backup of your print history metadata.

### Reporting
Generate monthly reports of print activity, costs, and success rates.

### Analysis
Import into Excel or Google Sheets for custom charts and analysis.

### Sharing
Share print history with clients or team members.

## Tips

- Export regularly for backup purposes
- Use filters to create focused reports
- Excel format is best for further analysis
- CSV format is best for importing to other systems
