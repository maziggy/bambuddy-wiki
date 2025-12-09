# API Reference

Bambuddy provides a REST API for programmatic access to all features.

## Interactive Documentation

When Bambuddy is running, access interactive API docs:

- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

These provide:
- Complete endpoint documentation
- Request/response schemas
- Try-it-out functionality
- Authentication details

---

## Base URL

```
http://localhost:8000/api/v1
```

Replace `localhost:8000` with your server address.

---

## Authentication

Currently, Bambuddy does not require authentication for API access. The API is intended for local network use.

> **Note**: Future versions may add authentication. Keep your Bambuddy instance on a trusted network.

---

## Endpoints Overview

### Printers

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/printers` | List all printers |
| POST | `/printers` | Add a new printer |
| GET | `/printers/{id}` | Get printer details |
| PUT | `/printers/{id}` | Update printer |
| DELETE | `/printers/{id}` | Delete printer |
| POST | `/printers/{id}/connect` | Connect to printer |
| POST | `/printers/{id}/disconnect` | Disconnect from printer |

### Archives

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/archives` | List all archives |
| GET | `/archives/{id}` | Get archive details |
| DELETE | `/archives/{id}` | Delete archive |
| POST | `/archives/{id}/reprint` | Send to printer |
| GET | `/archives/{id}/download` | Download 3MF file |
| POST | `/archives/{id}/photos` | Upload photo |

### Print Queue

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/queue` | List all queue items |
| POST | `/queue` | Add to queue |
| GET | `/queue/{id}` | Get queue item |
| DELETE | `/queue/{id}` | Remove from queue |
| PUT | `/queue/{id}/reorder` | Change queue position |
| POST | `/queue/{id}/cancel` | Cancel queued print |

### Smart Plugs

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/smart-plugs` | List all smart plugs |
| POST | `/smart-plugs` | Add smart plug |
| GET | `/smart-plugs/{id}` | Get plug details |
| PUT | `/smart-plugs/{id}` | Update plug |
| DELETE | `/smart-plugs/{id}` | Delete plug |
| POST | `/smart-plugs/{id}/on` | Turn on |
| POST | `/smart-plugs/{id}/off` | Turn off |
| GET | `/smart-plugs/{id}/status` | Get current status |

### Notifications

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/notifications/providers` | List providers |
| POST | `/notifications/providers` | Add provider |
| PUT | `/notifications/providers/{id}` | Update provider |
| DELETE | `/notifications/providers/{id}` | Delete provider |
| POST | `/notifications/providers/{id}/test` | Send test |
| GET | `/notifications/templates` | Get templates |
| PUT | `/notifications/templates/{event}` | Update template |

### Settings

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/settings` | Get all settings |
| PUT | `/settings` | Update settings |
| GET | `/settings/filaments` | List filaments |
| POST | `/settings/filaments` | Add filament |

### Maintenance

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/maintenance/types` | List maintenance types |
| POST | `/maintenance/types` | Create type |
| GET | `/maintenance/status` | Get maintenance status |
| POST | `/maintenance/log` | Log maintenance |

### Cloud Profiles

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/cloud/profiles` | List profiles |
| POST | `/cloud/sync` | Sync with cloud |
| GET | `/cloud/profiles/{id}` | Get profile |
| POST | `/cloud/templates` | Create template |

### Updates

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/updates/check` | Check for updates |
| POST | `/updates/apply` | Apply update |
| GET | `/updates/status` | Get update status |

---

## WebSocket

Real-time updates via WebSocket:

```
ws://localhost:8000/api/v1/ws
```

### Connection
```javascript
const ws = new WebSocket('ws://localhost:8000/api/v1/ws');

ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Event:', data.type, data.payload);
};
```

### Event Types

| Event | Description |
|-------|-------------|
| `printer_state` | Printer status update |
| `print_progress` | Print progress update |
| `print_complete` | Print finished |
| `print_failed` | Print failed |
| `queue_update` | Queue changed |
| `notification` | System notification |

### Message Format
```json
{
  "type": "printer_state",
  "printer_id": 1,
  "payload": {
    "status": "printing",
    "progress": 45,
    "nozzle_temp": 210,
    "bed_temp": 60
  }
}
```

---

## Common Response Formats

### Success Response
```json
{
  "success": true,
  "data": { ... }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "PRINTER_NOT_FOUND",
    "message": "Printer with ID 123 not found"
  }
}
```

### List Response
```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "total": 100,
    "page": 1,
    "per_page": 20
  }
}
```

---

## Example Requests

### List Printers
```bash
curl http://localhost:8000/api/v1/printers
```

### Add Printer
```bash
curl -X POST http://localhost:8000/api/v1/printers \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Workshop X1C",
    "ip_address": "192.168.1.100",
    "access_code": "12345678",
    "serial_number": "00M00A000000000"
  }'
```

### Get Archives
```bash
curl "http://localhost:8000/api/v1/archives?limit=10&offset=0"
```

### Schedule Print
```bash
curl -X POST http://localhost:8000/api/v1/queue \
  -H "Content-Type: application/json" \
  -d '{
    "archive_id": 42,
    "printer_id": 1,
    "scheduled_time": "2024-01-15T14:00:00Z",
    "auto_power_off": true
  }'
```

### Control Smart Plug
```bash
# Turn on
curl -X POST http://localhost:8000/api/v1/smart-plugs/1/on

# Turn off
curl -X POST http://localhost:8000/api/v1/smart-plugs/1/off
```

### Send Test Notification
```bash
curl -X POST http://localhost:8000/api/v1/notifications/providers/1/test
```

---

## Rate Limiting

Currently no rate limiting is implemented. For high-frequency polling, consider using WebSocket instead.

---

## Error Codes

| Code | Description |
|------|-------------|
| `PRINTER_NOT_FOUND` | Printer ID doesn't exist |
| `PRINTER_OFFLINE` | Printer not connected |
| `ARCHIVE_NOT_FOUND` | Archive ID doesn't exist |
| `QUEUE_ITEM_NOT_FOUND` | Queue item doesn't exist |
| `PLUG_NOT_FOUND` | Smart plug doesn't exist |
| `PLUG_OFFLINE` | Smart plug not responding |
| `INVALID_REQUEST` | Request validation failed |
| `FTP_ERROR` | File transfer failed |
| `MQTT_ERROR` | Printer communication failed |

---

## SDK / Client Libraries

No official client libraries yet. The API is standard REST and can be consumed with:
- `fetch` / `axios` (JavaScript)
- `requests` / `httpx` (Python)
- `curl` (command line)
- Any HTTP client

---

## Webhooks (Outgoing)

Configure outgoing webhooks in notification settings. Bambuddy sends POST requests to your URL with JSON payloads for print events.

See [Push Notifications](Features-Notifications.md#webhook-generic) for details.
