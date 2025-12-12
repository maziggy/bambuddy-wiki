---
title: Full-Text Search
description: Fast search across all archive metadata
---

# Full-Text Search

Bambuddy uses SQLite FTS5 (Full-Text Search) for lightning-fast search across all your archives.

---

## :material-magnify: Search Features

### What's Searchable

| Field | Example |
|-------|---------|
| **Print name** | "benchy" |
| **Filename** | "calibration_cube.3mf" |
| **Tags** | "functional" |
| **Notes** | "great layer adhesion" |
| **Designer** | "printables username" |
| **Filament type** | "PLA", "PETG", "ABS" |
| **Printer name** | "Workshop X1C" |

### Search Syntax

| Syntax | Description | Example |
|--------|-------------|---------|
| `word` | Simple search | `benchy` |
| `"phrase"` | Exact phrase | `"phone stand"` |
| `word*` | Prefix match | `ben*` matches "benchy" |
| `word1 word2` | Both words | `phone stand` |
| `word1 OR word2` | Either word | `benchy OR boat` |
| `-word` | Exclude word | `phone -case` |

---

## :material-lightning-bolt: Performance

FTS5 provides near-instant search:

| Archive Count | Search Time |
|:-------------:|:-----------:|
| 100 | < 1ms |
| 1,000 | < 5ms |
| 10,000 | < 20ms |
| 100,000 | < 100ms |

### Why FTS5?

- **Tokenized index** - Pre-processes text for fast lookup
- **Ranked results** - Most relevant first
- **Incremental updates** - Index updates automatically

---

## :material-keyboard: Quick Access

### Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| ++slash++ | Focus search box |
| ++escape++ | Clear search / close |
| ++enter++ | Execute search |
| ++arrow-down++ | Navigate results |

### Search Box Location

The search box is prominently displayed on the Archives page header.

---

## :material-filter: Combining Search with Filters

Search works alongside other filters:

1. Enter search term
2. Apply additional filters (printer, date, status)
3. Results show intersection

**Example**: Search "benchy" + filter by "Last 7 days" + filter by "Success"

---

## :material-history: Search History

Recent searches are remembered:

- Click the search box to see recent searches
- Click any previous search to reuse it
- Clear history in the search dropdown

---

## :material-cog: Index Management

The search index is automatically maintained:

### Automatic Updates

- New archives are indexed immediately
- Edits update the index
- Deletions remove from index

### Manual Rebuild

If search results seem incorrect:

1. Go to **Settings** > **System**
2. Click **Rebuild Search Index**
3. Wait for rebuild to complete

!!! note "When to Rebuild"
    Index rebuild is rarely needed. Only use if search results are clearly wrong or after database restoration.

---

## :material-api: API Search

Search via the REST API:

```bash
GET /api/v1/archives?search=benchy
```

Response includes matching archives with relevance ranking.

### API Parameters

| Parameter | Description |
|-----------|-------------|
| `search` | Search query string |
| `limit` | Max results (default: 50) |
| `offset` | Pagination offset |

---

## :material-lightbulb: Search Tips

!!! tip "Use Specific Terms"
    More specific searches yield better results. "voron stealthburner" is better than just "voron".

!!! tip "Quote Phrases"
    Use quotes for exact phrases: `"cable chain"` won't match "cable management chain".

!!! tip "Exclude with Minus"
    Exclude unwanted results: `case -phone` finds cases that aren't phone cases.

!!! tip "Prefix for Partial Matches"
    Use asterisk for partial matching: `bench*` matches "benchy", "benchmark", "bench".

!!! tip "Check Filters"
    If search returns nothing, check if filters are limiting results.
