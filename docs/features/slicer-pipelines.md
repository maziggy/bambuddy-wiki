---
title: Slicer Pipelines
description: Reusable preset bundles for one-click slicing and dispatch
---

# Slicer Pipelines

Save a printer + process + filament + bed-type combination once, then apply the whole bundle with a single click on the next file. Each pipeline can target a specific printer or a printer **class**, run multiple copies, and fan copies out across multiple printers in parallel.

---

## :material-pipe: What a Pipeline Is

A **pipeline** is a named, reusable bundle that captures everything the Slice dialog needs to slice a file the same way every time:

- **Printer profile** (e.g. `Bambu Lab H2D 0.4 nozzle`)
- **Process profile** (e.g. `0.20mm Standard @BBL H2D`)
- **One filament profile per AMS slot** (resolved against your imported / cloud / standard preset library)
- **Bed type** (Cool Plate / Engineering Plate / Textured PEI / …)
- **Target** — either a specific printer, or a printer class (e.g. *any X1C*)
- **Copies** — slice the file once, dispatch the resulting `.gcode.3mf` as N independent print queue items
- **Fanout strategy** — when N > 1, how to distribute the copies across printers in the class

Pipelines live under **Settings → Workflow → Slicer Pipelines** for editing, and on the **Print Queue → Pipelines** tab for the live runs dashboard.

---

## :material-content-save: Saving a Pipeline

The fastest way to make one is to open the Slice dialog for any file:

1. Pick your printer, process, filaments, and bed type as normal
2. Click **Save as pipeline** at the bottom of the dialog
3. Give the pipeline a name (e.g. `H2D — PETG-CF 0.4mm`)
4. Confirm

Your new pipeline appears under **Settings → Workflow → Slicer Pipelines** and is immediately available on every file's **Run with pipeline** menu.

You can also create one from scratch in Settings, but starting from a real Slice dialog avoids re-picking the same triplet manually.

---

## :material-play-circle: Running a Pipeline

Pipelines can be applied to any sliceable file:

- **File Manager** — the row's overflow menu (⋮) and the per-row action button both expose **Run with pipeline → \<name\>**
- **Archives** — the row menu offers **Reprint with pipeline → \<name\>** for any archive whose source 3MF is still in the library

Clicking a pipeline kicks off a **pipeline run** that:

1. Slices the file via the slicer sidecar using the pipeline's printer + process + filament + bed type
2. Creates one or more **print queue items** (one per copy) with the resulting `.gcode.3mf`
3. Hands dispatch to the existing queue scheduler

Each run shows up live on the **Print Queue → Pipelines** dashboard.

---

## :material-target: Targeting

Pipelines can be pinned in one of two ways:

### Specific printer

The run goes to *that printer's* queue. Useful when one printer is the only one wearing the right nozzle / bed / colour combination for this recipe.

### Printer class

The run goes to *any printer of this class* (e.g. *any H2D*). This lets one pipeline serve a fleet of identical machines — the queue scheduler picks the first available printer of the class. With **copies > 1** the run becomes a **fanout** (see below).

The Target picker on every pipeline card is grouped by **Specific printer** and **Printer class**; only printers and classes that at least one saved pipeline points at appear in the filter dropdowns so the menu stays short on installs with many printers.

---

## :material-content-copy: Multi-Copy & Fanout

A pipeline run can produce up to **N copies** of the sliced file as independent queue items. When targeting is set to **printer class**, those copies are distributed across the available printers in the class according to the chosen **fanout strategy**:

| Strategy | Behaviour |
|----------|-----------|
| **Spread** | Distribute copies as evenly as possible across all idle printers in the class — finishes the batch fastest. |
| **Single printer** | Send every copy to the same printer (the first idle one matching the class). Best when colour-change overhead between copies is what dominates wall-clock. |
| **First N** | Send one copy to each of the first N idle printers, in class order. |

The slice runs **once** per pipeline run — N copies share the same `.gcode.3mf` — so a 5-copy fanout doesn't pay the slice cost 5×.

---

## :material-view-dashboard: The Runs Dashboard

**Print Queue → Pipelines** lists every pipeline run across every pipeline. Each row shows:

- Run number, pipeline name, **status badge** (queued / slicing / dispatching / in-progress / completed / partial-failure / failed / cancelled), and the **target** chip (`Any X1C` or the specific printer name)
- Source filename
- Created-at timestamp, total copies, and a `completed/total` roll-up — failed copies are flagged in red
- Per-copy detail when expanded (each copy's status, assigned printer, and per-copy error message if any)

### Filters

Three filter dropdowns at the top of the dashboard:

- **Pipeline** — narrow to one specific pipeline's runs
- **Status** — narrow to runs in a single state (e.g. *partial failure*)
- **Target** — narrow to runs of a specific printer or printer class

Each dropdown shows only entries that actually exist in your data, so empty options never appear.

### In-flight actions

- **Cancel** — visible on `queued` / `slicing` / `dispatching` / `in_progress` runs; cancels every still-pending copy
- **Retry failed** — visible on `partial_failure` / `failed` runs; re-runs only the copies that didn't complete (the successful copies are not re-printed)

### Clearing terminal runs

The **Clear log** button (top-right of the filter row) opens a confirmation modal and then deletes every **terminal** run (`completed`, `failed`, `cancelled`, `partial_failure`). In-flight runs are **never** deleted — clearing during an active batch is safe.

---

## :material-shield-account: Permissions

Three distinct permissions gate the feature so authoring a recipe is a different trust dimension than spending filament with it:

| Permission | What it allows |
|------------|----------------|
| `pipelines:read` | View saved pipelines + view the runs dashboard |
| `pipelines:write` | Create, edit, delete pipelines; clear the runs log |
| `pipelines:run` | Kick off a pipeline run, cancel a running run, retry failed copies |

Default group membership:

| Group | `pipelines:read` | `pipelines:write` | `pipelines:run` |
|-------|:---:|:---:|:---:|
| Administrators | :material-check: | :material-check: | :material-check: |
| Operators | :material-check: | :material-check: | :material-check: |
| Viewers | :material-check: | — | — |

API keys cannot use any of the three — pipeline authoring and dispatch are admin-only operations that don't fit the API-key trust model.

!!! tip "Upgrading from an older version"
    On installs seeded before pipelines existed, the Administrators system group is auto-backfilled with the three permissions on the next startup. The Operators group is also auto-backfilled. Custom groups stay untouched — grant `pipelines:*` explicitly through **Settings → Users → Groups** if a custom role needs them.

---

## :material-file-tree: Where Things Live

| Path | Purpose |
|------|---------|
| **Settings → Workflow → Slicer Pipelines** | Manage the pipeline catalogue (create / edit / delete / inspect) |
| **Print Queue → Pipelines** | Live runs dashboard (filter / expand / cancel / retry / clear) |
| **File Manager → ⋮ → Run with pipeline** | Apply a pipeline to a library file |
| **Archives → ⋮ → Reprint with pipeline** | Re-run a pipeline against a previously-archived 3MF |
| **Slice dialog → Save as pipeline** | Capture the current Slice modal selection as a new pipeline |
