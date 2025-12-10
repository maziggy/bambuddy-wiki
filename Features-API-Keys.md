# API Keys & Webhooks

Bambuddy provides API key authentication for external integrations, allowing you to build automations and connect third-party services.

## Overview

- **Secure authentication** — API keys with granular permissions
- **Webhook endpoints** — Control queue, printers, and query status
- **Easy management** — Create, view, and delete keys from Settings

## Creating an API Key

1. Navigate to **Settings** → **API Keys** tab
2. Click **Create API Key**
3. Enter a descriptive name (e.g., "Home Assistant", "n8n Automation")
4. Select permissions:
   - **Read Status** — Query printer and queue status
   - **Manage Queue** — Add, remove, and reorder queue items
   - **Control Printer** — Start, stop, pause, cancel prints
5. Click **Create**
6. **Important:** Copy the API key immediately — it won't be shown again!

## API Key Security

- Keys are hashed before storage (only the prefix is visible)
- Keys cannot be retrieved after creation
- Delete and recreate if a key is compromised
- Use descriptive names to track key usage

## Using API Keys

Include the API key in the `X-API-Key` header:

```bash
curl -H "X-API-Key: your-api-key-here" \
  http://your-bambuddy:8000/api/v1/webhook/status
```

## Available Endpoints

### Status Endpoints (requires `can_read_status`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/webhook/status` | GET | Get all printer statuses |
| `/api/v1/webhook/status/{printer_id}` | GET | Get specific printer status |
| `/api/v1/webhook/queue` | GET | Get print queue |

### Queue Endpoints (requires `can_manage_queue`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/webhook/queue/add` | POST | Add item to queue |
| `/api/v1/webhook/queue/{id}` | DELETE | Remove item from queue |

### Printer Control (requires `can_control_printer`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/webhook/printer/{id}/start` | POST | Start next queued print |
| `/api/v1/webhook/printer/{id}/stop` | POST | Stop current print |
| `/api/v1/webhook/printer/{id}/pause` | POST | Pause current print |
| `/api/v1/webhook/printer/{id}/resume` | POST | Resume paused print |

## Example: Home Assistant Integration

```yaml
rest_command:
  bambuddy_start_print:
    url: "http://bambuddy:8000/api/v1/webhook/printer/{{ printer_id }}/start"
    method: POST
    headers:
      X-API-Key: "your-api-key-here"

  bambuddy_get_status:
    url: "http://bambuddy:8000/api/v1/webhook/status"
    method: GET
    headers:
      X-API-Key: "your-api-key-here"
```

## Example: n8n Webhook

1. Create an HTTP Request node
2. Set method to GET/POST as needed
3. Add header: `X-API-Key` with your key value
4. Set URL to your Bambuddy webhook endpoint

## Permissions Reference

| Permission | Allows |
|------------|--------|
| `can_read_status` | Query printer status, queue status, archive info |
| `can_manage_queue` | Add/remove queue items, reorder queue |
| `can_control_printer` | Start, stop, pause, resume, cancel prints |

## Tips

- Create separate keys for different integrations
- Use minimal permissions needed for each integration
- Regularly audit and rotate keys
- Delete unused keys promptly
