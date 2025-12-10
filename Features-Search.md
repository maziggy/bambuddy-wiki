# Full-Text Search

Bambuddy includes powerful full-text search capabilities powered by SQLite FTS5, allowing you to quickly find any print in your archive.

## Overview

The search feature indexes multiple fields from your archives:
- Print name
- Filename
- Tags
- Notes
- Designer
- Filament type

## Using Search

### Basic Search

1. Navigate to the **Archives** page
2. Type your search term in the search box
3. Results update automatically as you type

### Search Tips

- **Partial matches** — Searching "ben" will find "benchy"
- **Multiple words** — All words must match (AND logic)
- **Case insensitive** — "BENCHY" finds "benchy"

## Searchable Fields

| Field | Example |
|-------|---------|
| Print Name | "Voron Skirt" |
| Filename | "cable_chain.3mf" |
| Tags | "functional, replacement" |
| Notes | "printed for client" |
| Designer | "VoronDesign" |
| Filament Type | "PLA, PETG" |

## Combining with Filters

Search works alongside other filters:
- **Printer filter** — Search within prints from a specific printer
- **Material filter** — Search within prints using specific filament
- **Color filter** — Search within prints using specific colors
- **Favorites** — Search within favorite prints only
- **Tags** — Search within a specific tag

## Technical Details

Bambuddy uses SQLite FTS5 (Full-Text Search 5) for efficient searching:
- Indexes are automatically maintained via database triggers
- Searches are near-instantaneous even with thousands of archives
- Ranking ensures most relevant results appear first

## Tips

- Use specific terms for better results
- Add tags to your prints for easier searching later
- Include designer names when uploading prints from MakerWorld
- Use notes to add searchable context about prints
