# MQTT Publishing

Bambuddy can publish events to an external MQTT broker, enabling integration with home automation platforms like **Home Assistant**, **Node-RED**, and other MQTT-based systems.

## Configuration

Navigate to **Settings → Network → MQTT Publishing** to configure the connection.

### Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable MQTT** | Toggle MQTT publishing on/off | Off |
| **Broker Hostname** | MQTT broker address (IP or hostname) | - |
| **Port** | Broker port | 1883 (8883 with TLS) |
| **Username** | Authentication username (optional) | - |
| **Password** | Authentication password (optional) | - |
| **Topic Prefix** | Prefix for all published topics | `bambuddy` |
| **Use TLS** | Enable TLS/SSL encryption | Off |

!!! tip "Port Auto-Population"
    When you toggle TLS on/off, the port field automatically updates:

    - TLS enabled → Port 8883
    - TLS disabled → Port 1883

## Published Topics

All topics are prefixed with your configured topic prefix (default: `bambuddy`).

### Status

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/status` | Online/offline status | Yes |

### Printer Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/printers/{serial}/status` | Real-time printer state (throttled to 1/sec) | Yes |
| `bambuddy/printers/{serial}/print/started` | Print job started | No |
| `bambuddy/printers/{serial}/print/completed` | Print job completed successfully | No |
| `bambuddy/printers/{serial}/print/failed` | Print job failed | No |
| `bambuddy/printers/{serial}/ams/changed` | AMS filament changed | No |

### Queue Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/queue/job_added` | Job added to print queue | No |
| `bambuddy/queue/job_started` | Queued job started printing | No |
| `bambuddy/queue/job_completed` | Queued job completed | No |

### Maintenance Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/maintenance/alert` | Maintenance due alert | No |
| `bambuddy/maintenance/reset` | Maintenance counter reset | No |

### Smart Plug Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/smart_plugs/on` | Smart plug turned on | No |
| `bambuddy/smart_plugs/off` | Smart plug turned off | No |
| `bambuddy/smart_plugs/energy` | Energy consumption update | No |

### Archive Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/archive/created` | New archive created | No |
| `bambuddy/archive/updated` | Archive updated | No |

## Payload Format

All payloads are JSON objects. Here's an example printer status payload:

```json
{
  "printer_id": 1,
  "printer_name": "X1C-1",
  "printer_serial": "00M09C411500579",
  "timestamp": "2026-01-13T12:00:00.000000",
  "connected": true,
  "state": "PRINTING",
  "progress": 45.5,
  "remaining_time": 3600,
  "layer_num": 150,
  "total_layers": 300,
  "current_print": "benchy.3mf",
  "subtask_name": "Benchy",
  "temperatures": {
    "bed": 60.0,
    "bed_target": 60.0,
    "nozzle": 220.0,
    "nozzle_target": 220.0,
    "chamber": 35.0
  },
  "wifi_signal": -55,
  "chamber_light": true,
  "speed_level": 2,
  "cooling_fan_speed": 100,
  "big_fan1_speed": 50,
  "big_fan2_speed": 50
}
```

## Home Assistant Integration

### MQTT Discovery

Bambuddy does not currently support Home Assistant MQTT discovery. You'll need to configure sensors manually.

### Example Sensor Configuration

Add to your `configuration.yaml`:

```yaml
mqtt:
  sensor:
    - name: "X1C Print Progress"
      state_topic: "bambuddy/printers/00M09C411500579/status"
      value_template: "{{ value_json.progress }}"
      unit_of_measurement: "%"

    - name: "X1C State"
      state_topic: "bambuddy/printers/00M09C411500579/status"
      value_template: "{{ value_json.state }}"

    - name: "X1C Bed Temperature"
      state_topic: "bambuddy/printers/00M09C411500579/status"
      value_template: "{{ value_json.temperatures.bed }}"
      unit_of_measurement: "°C"
      device_class: temperature
```

### Node-RED Example

Use an **MQTT In** node subscribed to `bambuddy/#` to receive all events, then route them based on topic:

```
[MQTT In] → [Switch (by topic)] → [Function/Dashboard]
```

## TLS/SSL Configuration

Bambuddy supports TLS connections with self-signed certificates:

- Certificates are **not verified** (allows self-signed certs)
- Connection is still encrypted
- Suitable for home networks

For production environments, use properly signed certificates from your broker.

## Troubleshooting

### Connection Status

The connection status is shown in the Settings page:

- **Green dot** = Connected
- **Red dot** = Disconnected or error

### Common Issues

| Issue | Solution |
|-------|----------|
| "Not authorized" | Check username/password |
| Connection refused | Verify broker address and port |
| TLS errors | Ensure broker supports TLS on the configured port |
| No messages received | Check your subscriber's topic filter matches |

### Testing with Mosquitto

```bash
# Subscribe to all Bambuddy topics
mosquitto_sub -h your-broker -t "bambuddy/#" -v

# With authentication
mosquitto_sub -h your-broker -u username -P password -t "bambuddy/#" -v

# With TLS
mosquitto_sub -h your-broker -p 8883 --cafile ca.crt -t "bambuddy/#" -v
```
