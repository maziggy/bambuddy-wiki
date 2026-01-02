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
  "state": "printing",
  "progress": 45,
  "remaining_time": 3600,
  "current_layer": 120,
  "total_layers": 267,
  "temperatures": {
    "nozzle": 220,
    "bed": 60,
    "chamber": 35
  },
  "hms_status": "ok"
}
```

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

### Update Archive

```http
PATCH /archives/{id}
```

**Request:**
```json
{
  "name": "Updated Name",
  "notes": "Great print",
  "tags": ["functional", "gift"]
}
```

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

### Stream (MJPEG)

```http
GET /printers/{id}/camera/stream
```

Returns MJPEG stream. Query params:

| Parameter | Type | Description |
|-----------|------|-------------|
| `fps` | int | Frames per second (1-30) |

### Snapshot

```http
GET /printers/{id}/camera/snapshot
```

Returns single JPEG image.

### Stop Stream

```http
POST /printers/{id}/camera/stop
```

Terminates active streams for printer.

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

Except for file downloads (application/octet-stream) and images (image/jpeg).

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
