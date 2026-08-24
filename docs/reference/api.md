---
title: API Reference
description: Complete REST API documentation
---

# API Reference

Bambuddy provides a REST API for integration with external tools and automation.

---

## :material-key: Authentication

### API Key Authentication

Include your API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your-api-key" \
  http://localhost:8000/api/v1/printers
```

### Getting an API Key

1. Go to **Settings** > **API Keys**
2. Click **Create API Key**
3. Select permissions
4. Copy the key (shown only once)

See [API Keys & Webhooks](../features/api-keys.md) for details.

---

## :material-web: Interactive API Browser

Bambuddy includes a built-in API browser for exploring and testing endpoints without external tools.

### Accessing the API Browser

1. Go to **Settings** > **API Keys**
2. The API Browser appears in the right column

### Features

| Feature | Description |
|---------|-------------|
| **OpenAPI Integration** | Automatically loads all endpoints from the schema |
| **Grouped by Category** | Endpoints organized by printers, archives, settings, etc. |
| **Parameter Inputs** | Fill in path, query, and body parameters |
| **Auto-examples** | Request body pre-filled with schema examples |
| **Live Execution** | Execute requests and see real responses |
| **Response Display** | Formatted JSON with status code and timing |
| **Search** | Filter endpoints across all categories |

### Testing with API Keys

1. Paste your API key in the "API Key for Testing" input
2. The key is sent as `X-API-Key` header with each request
3. Test authenticated endpoints without external tools

!!! tip "Quick Setup"
    After creating a new API key, click "Use in API Browser" to automatically add it for testing.

---

## :material-link: Base URL

```
http://your-server:8000/api/v1
```

All endpoints are relative to this base URL.

---

## :material-printer-3d: Printers

### List Printers

```http
GET /printers
```

**Response:**
```json
[
  {
    "id": 1,
    "name": "Workshop X1C",
    "ip_address": "192.168.1.100",
    "serial_number": "01P00A000000001",
    "model": "X1 Carbon",
    "status": "idle"
  }
]
```

### Get Printer

```http
GET /printers/{id}
```

**Response:**
```json
{
  "id": 1,
  "name": "Workshop X1C",
  "ip_address": "192.168.1.100",
  "serial_number": "01P00A000000001",
  "model": "X1 Carbon",
  "status": "printing",
  "current_print": {
    "filename": "benchy.3mf",
    "progress": 45,
    "remaining_time": 3600
  }
}
```

### Get Printer Status

```http
GET /printers/{id}/status
```

**Response:**
```json
{
  "id": 1,
  "name": "X1C-Lab",
  "connected": true,
  "state": "RUNNING",
  "progress": 45,
  "remaining_time": 3600,
  "layer_num": 120,
  "total_layers": 267,
  "temperatures": {
    "nozzle": 220,
    "nozzle_target": 220,
    "bed": 60,
    "bed_target": 60,
    "chamber": 35
  },
  "hms_errors": [
    {
      "code": "0x8004",
      "attr": 50364420,
      "module": 3,
      "severity": 3,
      "actions": [],
      "job_id": "1234567890",
      "full_code": "03008004",
      "description": "Filament ran out. Please load new filament."
    }
  ],
  "awaiting_plate_clear": false
}
```

`layer_num` is the current layer; `total_layers` is the layer count of the running job. `temperatures` carries a `_target` companion for the heaters that have one, and omits `chamber` entirely on models without a chamber sensor. `state` is the firmware's own value (`IDLE`, `PREPARE`, `SLICING`, `RUNNING`, `PAUSE`, `FINISH`, `FAILED`), not a lowercased Bambuddy label.

`description` is the resolved text for the fault, so a client does not have to carry its own copy of the error catalogue to tell a user what happened. It is English only and is not localized. It is `null` whenever the catalogue does not cover the code, which is common for faults sourced from the printer's `hms[]` array. Treat `null` as "no text available", never as "no fault": the fault is fully reported either way, and `full_code` is what identifies it. The same field is on the `printer_status` [WebSocket](../features/monitoring.md) message.

`awaiting_plate_clear` is a Bambuddy-side gate, not printer telemetry. It goes `true` when a print reaches a terminal state and stays `true` until the plate is confirmed clear via [Clear Plate](#clear-plate); the queue will not dispatch the next job in the meantime. It survives restarts and Auto Off power cycles, so a printer that reports `IDLE` after a reboot can still be waiting. The same flag is pushed over the [WebSocket](../features/monitoring.md) `printer_status` message and over [MQTT](../features/mqtt.md) — including a dedicated retained topic, which is the better subscription for automations because it does not depend on the printer still being powered on.

### Refresh Printer Status

Request a full status update from the printer via MQTT pushall command. Useful for getting fresh AMS data after swapping spools.

```http
POST /printers/{id}/refresh-status
```

**Response:**
```json
{
  "status": "refresh_requested"
}
```

**Errors:**

- `404` - Printer not found
- `400` - Printer not connected

### Clear HMS Errors

Clear HMS/print errors on the printer. Sends a `clean_print_error` MQTT command and clears errors locally.

```http
POST /printers/{id}/hms/clear
```

**Response:**
```json
{
  "success": true,
  "message": "HMS errors cleared"
}
```

**Errors:**

- `404` - Printer not found
- `400` - Printer not connected
- `500` - Failed to clear HMS errors

**Permission:** `printers:control`

### Clear Plate

```http
POST /printers/{id}/clear-plate
```

Acknowledge that the build plate has been cleared after a finished/failed print. Sets a plate-cleared flag so the scheduler can start the next queued print.

**Response:**
```json
{
  "success": true,
  "message": "Plate cleared, next print will start shortly"
}
```

Acknowledgement is accepted whenever `awaiting_plate_clear` is `true`, whatever the printer currently reports — after an Auto Off power cycle it boots into `IDLE` with no memory of the finished print, and the gate still needs clearing. The reported state only matters as a fallback when the flag is not set.

The printer does **not** have to be online. Clearing the plate only mutates Bambuddy-side state — no command is sent to the printer — so it works on a machine [Auto Power Off](../features/smart-plugs.md) has switched off, which with that feature enabled is the normal end-of-print situation. The queue still waits for the printer to come back before dispatching; releasing the gate is what allows it to power the printer on again.

**Errors:**

- `404` - Printer not found
- `400` - Printer is not awaiting acknowledgement and is not in `FINISH`/`FAILED` state

**Permission:** `printers:clear_plate`

### Set Print Speed

Change the print speed preset during an active print.

```http
POST /printers/{id}/print-speed?mode=N
```

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|:--------:|-------------|
| `mode` | int | Yes | Speed preset: `1` (Silent 50%), `2` (Standard 100%), `3` (Sport 124%), `4` (Ludicrous 166%) |

**Response:**
```json
{
  "success": true,
  "message": "Print speed set to Standard"
}
```

**Errors:**

- `404` - Printer not found
- `400` - Printer not connected or no active print
- `422` - Invalid mode (must be 1-4)

**Permission:** `printers:control`

### Add Printer

```http
POST /printers
```

**Request:**
```json
{
  "name": "New Printer",
  "ip_address": "192.168.1.101",
  "access_code": "12345678",
  "serial_number": "01P00A000000002"
}
```

### Update Printer

```http
PATCH /printers/{id}
```

**Request:**
```json
{
  "name": "Updated Name"
}
```

### Delete Printer

```http
DELETE /printers/{id}
```

### Download Multiple Printer Files

API clients can request a disk-backed ZIP containing files from printer storage:

```http
POST /printers/{id}/files/download-zip
Content-Type: application/json

{
  "paths": [
    "/timelapse/video.mp4",
    "/ipcam/ipcam-record.20260812.mp4"
  ],
  "sizes": {
    "/timelapse/video.mp4": 773468,
    "/ipcam/ipcam-record.20260812.mp4": 250000000
  }
}
```

The response is a ZIP attachment. `sizes` is an optional map of FTP-reported byte sizes. Supplying it lets Bambuddy reject an oversized selection or insufficient free space before FTP transfer begins; existing clients that send only `paths`, relative paths, or duplicate paths remain supported, and actual downloaded bytes are always capped. Up to 1,000 paths and 10 GiB total can be requested. Files that cannot be downloaded are skipped; response headers report requested, downloaded, and failed counts, and an all-failed request preserves the historical empty-ZIP response.

`sizes` is all-or-nothing: send it for every path or omit it entirely. A map covering only some of the selection is rejected, as is a negative size, and the keys must match the `paths` strings exactly — if `paths` are relative, the `sizes` keys have to be relative too.

| Status | Meaning |
|--------|---------|
| `400` | Empty selection |
| `413` | Selection exceeds 1,000 paths or 10 GiB |
| `422` | `sizes` does not cover exactly the selected paths, or a size is negative |
| `504` | The 30-minute preparation deadline passed |
| `507` | The app data volume cannot safely stage the selection |

**Permission:** `printers:files`

The web UI uses an asynchronous browser-native variant so neither large source files nor the result have to be buffered into a JavaScript `Blob`, and the initiating HTTP request does not occupy a proxy connection for the whole FTP transfer:

1. `POST /printers/{id}/files/download-job` with normal authentication and `paths`, `sizes`, `filename`, and `as_zip`. The endpoint returns the job immediately. It validates more strictly than `download-zip` does: duplicate paths are rejected with `400`, and `as_zip: false` requires exactly one path, because a native download has no container to put a second file in.
2. Poll `GET /printers/{id}/files/download-jobs/{job_id}`. The body carries `job_id`, `printer_id`, `state`, `requested`, `successful`, `failed`, `token`, `filename`, and `message`, where `state` is one of `queued`, `preparing`, `ready`, `failed`, or `cancelled`. `DELETE` the same URL to cancel; the FTP worker cooperatively stops and removes partial staging.
3. When `state` is `ready`, the body's `token` fills in the native `GET /printers/{id}/files/dl/{token}/{filename}` URL. The token is short-lived, single-use, and bound to the printer ID. A spent or expired token answers `403` with a short text file rather than JSON, because the browser reaches this URL through a download click and saves whatever comes back.

All job and polling endpoints require `printers:files`, including the API key's optional `printer_ids` allowlist. Only the final `/dl/` URL bypasses the gateway middleware, and it validates its resource-bound token itself. Staging lives under the configured archive data volume and is eligible for cleanup after one hour; cleanup runs at startup, before preparation, and every 15 minutes. Job state and cancellation are published as files, so any app worker can report on or cancel a job that another one is running. Preparations do not wait on each other: free space is re-checked against the bytes actually written throughout every transfer, so two jobs that start together each stop on their own when the volume runs low.

---

## :material-archive: Archives

### List Archives

```http
GET /archives
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `printer_id` | int | Filter by printer |
| `status` | string | success, failed, stopped |
| `start_date` | date | Filter from date |
| `end_date` | date | Filter to date |
| `search` | string | Full-text search |
| `project_id` | int | Filter by project |
| `limit` | int | Max results (default: 50) |
| `offset` | int | Pagination offset |

**Response:**
```json
{
  "total": 1234,
  "archives": [
    {
      "id": 1,
      "name": "Benchy",
      "filename": "benchy.3mf",
      "printer_id": 1,
      "printer_name": "Workshop X1C",
      "created_at": "2024-01-15T14:30:00Z",
      "duration": 8100,
      "status": "success",
      "filament_used": 45.2,
      "filament_type": "PLA"
    }
  ]
}
```

### Get Archive

```http
GET /archives/{id}
```

### Find Archive Videos

```http
GET /archives/{id}/printer-media
```

Returns an attached timelapse, matching printer-side timelapse, and IP-camera chunks whose timestamps overlap the print window. Directory inspection is read-only; no printer file is downloaded until a separate download request is made. Callers without `printers:files` still receive an attached local timelapse, plus a `printer_files_forbidden` warning, while printer discovery is skipped.

A printer-side timelapse is only looked for when nothing is attached to the archive yet, so `local_timelapse` and a `"kind": "timelapse"` entry in `remote_files` do not appear together. When a copy is attached, `local_timelapse` is `{"name": ..., "size": ...}`; otherwise it is `null`.

```json
{
  "archive_id": 42,
  "printer_id": 1,
  "local_timelapse": null,
  "remote_files": [
    {
      "name": "video_2026-08-12_14-38-46.mp4",
      "path": "/timelapse/video_2026-08-12_14-38-46.mp4",
      "size": 773468,
      "mtime": "2026-08-12T14:40:12",
      "kind": "timelapse"
    }
  ],
  "warnings": []
}
```

`warnings` distinguishes the reasons a listing came back short, so a caller can tell an unreachable printer from one that simply has no footage:

| Warning | Meaning |
|---------|---------|
| `printer_files_forbidden` | The caller lacks `printers:files`, so only the attached copy was considered |
| `printer_missing` | The archive names a printer that no longer exists |
| `timelapse_unavailable` | No timelapse directory could be read — the printer is off, unreachable, or in the FTPS handshake cool-off |
| `ipcam_unavailable` | `/ipcam` could not be read, for the same set of reasons |

An archive with no printer or no recorded start time returns empty lists and no warning, because there is nothing to look for rather than something that failed.

**Permissions:** `archives:read_all` or ownership through `archives:read_own`; `printers:files` is additionally required for the printer-side portion of the response

### Download an Attached Timelapse

```http
POST /archives/{id}/media-download-token
```

Mints a single-use token bound to this archive's attached timelapse and returns it with the file's name:

```json
{
  "token": "0e2a...",
  "filename": "video_2026-08-12_14-38-46.mp4"
}
```

Then fetch the file itself, which needs no other credential:

```http
GET /archives/{id}/media/dl/{token}/{filename}
```

The token expires after five minutes and is consumed on first use. Because a browser reaches this URL through a download click and saves whatever comes back, a spent token, a missing attachment, or a file gone from disk answers with a short text file explaining the failure rather than a JSON error body.

**Permissions:** `archives:read_all` or ownership through `archives:read_own`. Printer access is not required, and neither is `camera:view` — the attached copy is an archive asset. `404` if this archive has no attached timelapse.

Printer-side files found by the endpoint above are fetched through the printer download job endpoints instead, which require `printers:files`.

### Update Archive

```http
PATCH /archives/{id}
```

**Request:**
```json
{
  "name": "Updated Name",
  "notes": "Great print",
  "tags": ["functional", "gift"],
  "filament_used_grams": 46.16
}
```

`filament_used_grams` is accepted between 0 and 100000 and is written to the archive's most recent run as well, so the filament totals on the Projects page and in the Prometheus metrics — which sum the runs, not the archives — agree with the card. A run that measured its own weight through spool tracking keeps that measurement; only a run with no figure, or one that inherited the archive's, is updated. Nothing is deducted from Spoolman or from internal inventory. It exists for a print that archived without its 3MF, where nothing else can supply a figure: the rescan endpoint reads the figure out of the 3MF, and such an archive has no file to read. On an archive that does have its 3MF, a rescan overwrites a hand-typed figure with the sliced one.

### Delete Archive

```http
DELETE /archives/{id}
```

### Download 3MF

```http
GET /archives/{id}/3mf
```

Returns the 3MF file as download.

### Export Archives

```http
GET /archives/export
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `format` | string | csv or xlsx |
| (others) | | Same filters as list |

---

## :material-folder-multiple: Projects

### List Projects

```http
GET /projects
```

### Get Project

```http
GET /projects/{id}
```

### Create Project

```http
POST /projects
```

**Request:**
```json
{
  "name": "Voron Build",
  "description": "Building a Voron 2.4",
  "color": "#4caf50",
  "target_count": 100
}
```

### Update Project

```http
PATCH /projects/{id}
```

### Delete Project

```http
DELETE /projects/{id}
```

---

## :material-file-cad: Library Files

### Export Mesh

```http
GET /library/files/{file_id}/mesh
```

Returns the file's geometry as a binary STL, so a client can show a 3D preview
without having to parse the original container itself. Accepts `.3mf`, `.stl`,
`.obj` and `.ply`.

Requires the same library read permission as downloading the file.

**Response:**

```http
Content-Type: model/stl
Cache-Control: private, max-age=86400
```

| Code | Meaning |
|------|---------|
| 200 | Binary STL |
| 400 | Not a file type a mesh can be exported from |
| 404 | File not found |
| 422 | The file could not be read, or holds no geometry |

Very dense models are simplified towards a 50,000-vertex budget first — enough
detail to orbit a model, without sending tens of megabytes to a phone.

!!! note "Sliced files are refused"
    A `.gcode.3mf` is rejected with a `400`, even though it ends in `.3mf`. It
    holds toolpaths rather than a plain model, so the G-code preview is the right
    way to look at one.

---

## :material-printer-3d-nozzle: Print Queue

### Get Queue

```http
GET /queue
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `printer_id` | int | Filter by printer |
| `status` | string | pending, printing, completed |

### Add to Queue

```http
POST /queue
```

**Request:**
```json
{
  "archive_id": 123,
  "printer_id": 1,
  "scheduled_time": "2024-01-15T18:00:00Z"
}
```

### Remove from Queue

```http
DELETE /queue/{id}
```

### Reorder Queue

```http
POST /queue/reorder
```

**Request:**
```json
{
  "order": [3, 1, 2, 4]
}
```

### List Batches

```http
GET /queue/batches
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | `active`, `completed`, or `cancelled` |

Batches with neither queue items nor per-plate targets are omitted &mdash; see
[Batch Orders](../features/print-queue.md#batch-orders). Fetching one by id
(`GET /queue/batches/{batch_id}`) returns it regardless.

### Create Batch or Order

```http
POST /queue/batches
```

Without `plates` this creates a plain grouping: pass `item_ids` to group existing
pending items, or omit them and pass the returned `id` as `batch_id` on later
`POST /queue` calls. With `plates` it becomes an **order** that records how many
runs of each plate are wanted, so a failed run still counts as owed.

**Request:**
```json
{
  "name": "Bracket run",
  "library_file_id": 42,
  "plates": [
    { "plate_id": 1, "quantity_target": 1 },
    { "plate_id": 2, "quantity_target": 2 },
    { "plate_id": 3, "quantity_target": 3 }
  ],
  "due_date": "2026-09-01T12:00:00Z",
  "notes": "Rush job"
}
```

`plate_id` is `null` for a single-plate file. A `quantity_target` of `0` is
allowed &mdash; a plate that is not required yet keeps its row so the target can
be raised later &mdash; but an order in which every target is `0` is rejected.

### Update an Order

```http
PATCH /queue/batches/{batch_id}
```

Every field is optional. Supplying `plates` **replaces** the whole target set, so
a plate left out of the list has its target row removed. Lowering a target below
what has already been dispatched is allowed and simply leaves nothing owed;
queued items are never cancelled implicitly.

**Request:**
```json
{
  "name": "Bracket run (revised)",
  "plates": [{ "plate_id": 1, "quantity_target": 5 }]
}
```

### Dispatch Remaining Runs

```http
POST /queue/batches/{batch_id}/dispatch
```

Creates queue items for the runs the order still owes. Each is copied from the
most recent item for that plate, inheriting its printer or model target, AMS
mapping, filament overrides and print options, and is appended to the end of the
relevant printer's queue.

**Request:**
```json
{
  "plate_id": 2,
  "only_plate": true,
  "limit": 3
}
```

| Field | Type | Description |
|-------|------|-------------|
| `plate_id` | int \| null | Which plate to dispatch. Only read when `only_plate` is true |
| `only_plate` | bool | Restrict to the single plate named above. Default `false` &mdash; every plate with work outstanding |
| `limit` | int | Cap on items created across all plates. Omit to queue everything owed |

Returns `400` when a plate owes runs but has never been queued, since there is no
existing item to copy settings from, and when the batch has been cancelled.

### Ungroup a Batch

```http
POST /queue/batches/{batch_id}/ungroup
```

Clears `batch_id` from every member the caller owns. The batch row is deleted
once no members remain.

### Cancel a Batch

```http
DELETE /queue/batches/{batch_id}
```

Cancels the batch's pending items and marks the batch cancelled. Items that have
already run are untouched.

---

## :material-calendar-clock: Scheduled Drying Sessions

### Get Scheduled Drying Sessions

```http
GET /scheduled-dryings
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `printer_id` | int | Filter by printer |

Returns sessions that are `pending`, `running`, or `failed`, earliest start time first, with sessions that have no start time ahead of the rest. Completed and cancelled sessions are not returned.

**Permission:** `printers:read`

### Create Scheduled Drying Session

```http
POST /scheduled-dryings
```

**Request:**
```json
{
  "printer_id": 1,
  "ams_id": 2,
  "temp": 45,
  "duration_hours": 12,
  "filament": "PLA",
  "rotate_tray": true,
  "start_after": "2026-07-26T18:00:00Z"
}
```

`start_after` is the earliest start, and it is optional: omit it (or send `null`) and the session runs as soon as the printer is idle and the AMS is ready. The drying popover has no equivalent — its **Now** option starts drying immediately through `POST /printers/{id}/drying/start` instead — so this is an API-only way to say "next time the printer is free".

**Permission:** `printers:control`

### Cancel or Dismiss Scheduled Drying Session

```http
DELETE /scheduled-dryings/{id}
```

Cancels a `pending` or `running` session and responds with `{"status": "cancelled"}`. A running session is also sent a stop command. On a `failed` session the record is deleted instead, and the response is `{"status": "dismissed"}`.

**Permission:** `printers:control`

---

## :material-cube-scan: Spool Inventory

### List Spools

```http
GET /inventory/spools
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `include_archived` | bool | Include archived spools (default `false`) |

### Get Spool

```http
GET /inventory/spools/{id}
```

### Find Spool by Tag

Look up a single spool by its NFC tag identifiers without listing the whole inventory. This is intended for NFC inventory integrations that scan a Bambu Lab spool tag and need to check whether it already exists before creating or updating it.

```http
GET /inventory/spools/by-tag
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `tray_uuid` | string | Bambu Lab spool UUID from the tag — the same value the AMS reports over MQTT |
| `tag_uid` | string | RFID tag UID |
| `include_archived` | bool | Include archived spools (default `false`) |

At least one of `tray_uuid` or `tag_uid` must be supplied. Values are normalised (case-insensitive, non-hex separators ignored). `tray_uuid` is matched first and `tag_uid` is used as a fallback. Returns the single matching spool.

**Response:**
```json
{
  "id": 42,
  "material": "PLA",
  "brand": "Bambu",
  "color_name": "Red",
  "tray_uuid": "AABBCCDDEEFF0011AABBCCDDEEFF0011",
  "tag_uid": "04A1B2C3"
}
```

**Errors:**

- `400` - Neither `tray_uuid` nor `tag_uid` was provided
- `404` - No matching spool found

!!! note "Required scope"
    This endpoint accepts **inventory read** *or* **inventory update** access — for API keys, either the **Read Status** scope *or* the **Manage Inventory** scope. This lets a key that can already create, update, and delete spools look one up to dedupe an NFC scan. (Listing spools and fetching a spool by id still require **Read Status**.)

---

## :material-chart-line: Statistics

### Get Statistics

```http
GET /statistics
```

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `printer_id` | int | Filter by printer |
| `start_date` | date | Period start |
| `end_date` | date | Period end |

**Response:**
```json
{
  "total_prints": 1234,
  "successful_prints": 1100,
  "failed_prints": 100,
  "stopped_prints": 34,
  "success_rate": 89.14,
  "total_print_time": 360000,
  "total_filament_used": 15000.5,
  "total_cost": 350.00
}
```

### Export Statistics

```http
GET /statistics/export
```

---

## :material-camera: Camera

An `<img>` or `<video>` tag cannot send an `Authorization` header, so the stream
and snapshot endpoints also accept a token in the query string. Pass either a
60-minute browser token (`POST /printers/camera/stream-token`) or a long-lived
camera token — see [Long-Lived Camera Tokens](../features/camera.md#long-lived-camera-tokens).

### Stream (MJPEG)

```http
GET /printers/{id}/camera/stream
```

Returns MJPEG stream. Query params:

| Parameter | Type | Description |
|-----------|------|-------------|
| `fps` | int | Frames per second (1-30) |
| `token` | string | Camera token, when auth is enabled |

### Snapshot

```http
GET /printers/{id}/camera/snapshot
```

Returns single JPEG image. Accepts the same `token` query param.

### Stop Stream

```http
POST /printers/{id}/camera/stop
```

Terminates active streams for printer.

---

## :material-monitor-dashboard: Cam Wall

### Wall Feed

```http
GET /camwall/printers
```

Every printer plus the handful of status fields a [Cam Wall](printers.md#cam-wall-view)
tile draws. One call for the whole wall — a kiosk display polls this on a fixed
interval with no WebSocket to invalidate it.

| Parameter | Type | Description |
|-----------|------|-------------|
| `token` | string | A **Cam Wall**-scoped camera token, when auth is enabled |

Authenticated **only** by a `camwall`-scoped token. A `camera_stream` token is
refused here — it was minted to hand out video, not to enumerate a fleet by
name.

```json
[
  {
    "id": 1,
    "name": "X1C-Lab",
    "camera_rotation": 0,
    "connected": true,
    "state": "RUNNING",
    "progress": 42.0,
    "remaining_time": 33,
    "layer_num": 120,
    "total_layers": 300,
    "hms_errors": []
  }
]
```

That list is the entire payload. It deliberately carries no `serial_number`, no
`ip_address`, no `access_code`, and no print filename: the token travels in a URL
displayed on a screen, so the feed behind it must not disclose more than the
camera picture already does.

---

## :material-video-box: Streaming Overlay

### Overlay Status

```http
GET /printers/{printer_id}/overlay-status
```

Everything the [streaming overlay](../features/camera.md#streaming-overlay-for-obs)
draws for one printer — name, camera rotation, live print state, and the one
display setting it reads. A token-authenticated sibling of the printer status
endpoint, so an OBS browser source with no login session can back the overlay.

| Parameter | Type | Description |
|-----------|------|-------------|
| `token` | string | A **Streaming Overlay**-scoped camera token, when auth is enabled |

Authenticated **only** by an `overlay`-scoped token. A `camwall` or
`camera_stream` token is refused here — unlike the Cam Wall feed this names the
file being printed, so it sits behind its own scope.

```json
{
  "id": 1,
  "name": "X1C-Lab",
  "camera_rotation": 0,
  "connected": true,
  "state": "RUNNING",
  "current_print": "Benchy.gcode.3mf",
  "gcode_file": "Metadata/plate_1.gcode",
  "progress": 42.0,
  "remaining_time": 33,
  "layer_num": 120,
  "total_layers": 300,
  "stg_cur_name": null,
  "time_format": "system"
}
```

That object is the entire payload. Like the Cam Wall feed it carries no
`serial_number`, `ip_address`, or `access_code` — but it *does* carry the print
filename, which is why the overlay scope is distinct from `camwall`.

---

## :material-cog: System

### System Info

```http
GET /system/info
```

**Response:**
```json
{
  "version": "0.1.5b6",
  "uptime": 86400,
  "database": {
    "archives": 1234,
    "size_mb": 45.6
  }
}
```

### Health Check

```http
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "database": "connected",
  "mqtt": "connected"
}
```

---

## :material-alert-circle: Error Responses

### Error Format

```json
{
  "detail": "Error message",
  "code": "ERROR_CODE"
}
```

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 400 | Bad Request |
| 401 | Unauthorized (no/invalid API key) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 422 | Unprocessable (well-formed request, unusable content) |
| 429 | Rate Limited |
| 500 | Server Error |

---

## :material-speedometer: Rate Limits

| Endpoint Type | Limit |
|--------------|-------|
| Read | 100/minute |
| Write | 30/minute |
| Control | 10/minute |

### Rate Limit Headers

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705331400
```

---

## :material-code-json: Content Types

### Request

```http
Content-Type: application/json
```

### Response

```http
Content-Type: application/json
```

Except for file downloads (application/octet-stream), images (image/jpeg) and
mesh exports (model/stl).

---

## :material-webhook: Webhooks

Bambuddy can send webhooks for events. Configure in Settings > Notifications.

### Webhook Payload

```json
{
  "event": "print_complete",
  "timestamp": "2024-01-15T14:30:00Z",
  "data": {
    "printer_id": 1,
    "printer_name": "Workshop X1C",
    "archive_id": 123,
    "filename": "benchy.3mf",
    "duration": 8100,
    "status": "success"
  }
}
```

### Event Types

| Event | Trigger |
|-------|---------|
| `print_started` | Print begins |
| `print_progress` | Progress milestone |
| `print_complete` | Print finishes |
| `print_failed` | Print fails |
| `printer_offline` | Connection lost |
| `printer_error` | HMS error |
