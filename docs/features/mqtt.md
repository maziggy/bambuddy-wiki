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

!!! note "Saved password"
    Once saved, the password is never shown again: the field shows `********` instead. Leave it as is to keep the saved value, clear it to remove it, or enter a new one. If you change the broker, port or TLS setting, enter the password again; the settings are not saved until you do.

!!! tip "Port Auto-Population"
    When you toggle TLS on/off, the port field automatically updates:

    - TLS enabled → Port 8883
    - TLS disabled → Port 1883

## Published Topics

All topics are prefixed with your configured topic prefix (default: `bambuddy`).

### Status

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/status` | Online/offline status. If Bambuddy's connection to the broker drops without a clean disconnect (a crash, a killed container), the broker publishes `offline` for it (a Last Will) | Yes |

### Printer Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/printers/{serial}/status` | Real-time printer state (throttled to 1/sec) | Yes |
| `bambuddy/printers/{serial}/print/started` | Print job started | No |
| `bambuddy/printers/{serial}/print/completed` | Print job completed successfully | No |
| `bambuddy/printers/{serial}/print/failed` | Print job failed | No |
| `bambuddy/printers/{serial}/run/started` | A print's run opened at its start: names the run and its archive (see [Run Events](#run-events)) | No |
| `bambuddy/printers/{serial}/run/completed` | A print's run as its end wrote it: outcome and figures | No |
| `bambuddy/printers/{serial}/run/discarded` | A run dropped without an end (its print ended unseen, or its archive was deleted) | No |
| `bambuddy/printers/{serial}/ams/changed` | AMS filament changed | No |
| `bambuddy/printers/{serial}/plate_clear` | Plate-clear gate raised or released | Yes |
| `bambuddy/printers/{serial}/offline` | Printer has been disconnected for a minute | No |
| `bambuddy/printers/{serial}/online` | Printer reconnected after an `offline` event | No |
| `bambuddy/printers/{serial}/filament/low` | The spool in an AMS slot crossed its low-stock threshold | No |

### Queue Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/queue/job_added` | Job added to print queue, from any source; one message per job | No |
| `bambuddy/queue/job_started` | Queued job started printing | No |
| `bambuddy/queue/job_completed` | Queued job completed | No |

### Maintenance Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/maintenance/alert` | Maintenance due or nearly due, after a print; `current_value` is the hours (or days) since it was last done, `threshold` its interval, `unit` says `hours` or `days` | No |
| `bambuddy/maintenance/reset` | Maintenance counter reset | No |

### Smart Plug Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/smart_plugs/on` | A plug was switched on, by hand or automatically (auto-on, schedule, queue); also sent when it already was on | No |
| `bambuddy/smart_plugs/off` | A plug was switched off, by hand or automatically (auto-off, schedule, failure detection); also sent when it already was off | No |
| `bambuddy/smart_plugs/energy` | Power and energy of each enabled plug, once a minute | No |

### Archive Events

| Topic | Description | Retained |
|-------|-------------|----------|
| `bambuddy/archive/created` | New archive created | No |
| `bambuddy/archive/updated` | Archive updated | No |

### Run Events

An archive is a file; each print of it is a run. The print and archive events above go out before the print's run exists, so they don't name it. `run/started` and `run/completed` do, once it does:

```json
{
  "run_id": 456,
  "archive_id": 123,
  "print_name": "Benchy",
  "printer_id": 1,
  "printer_name": "X1C-1",
  "plate_id": 2,
  "status": "completed",
  "started_at": "2026-10-31T09:00:00+00:00",
  "completed_at": "2026-10-31T10:12:00+00:00",
  "duration_seconds": 4320,
  "duration_unmeasured": false,
  "failure_reason": null,
  "filament_used_grams": 12.5,
  "cost": 0.38,
  "energy_kwh": null,
  "timestamp": "2026-10-31T10:12:01.000000+00:00"
}
```

`run/started` carries the fields up to `started_at` (its `status` is `printing`); `run/completed` adds the rest. Times are UTC. `run/started` is sent once per print: a restart that resumes a print does not send it again.

`run/completed` is sent when the print's end reaches Bambuddy, and also when a queued print is stopped while its printer is offline (as `cancelled`). Should that printer report the real end later, a second `run/completed` for the same `run_id` replaces the first. `duration_unmeasured` is `true` for an end only found at a reconnect: its duration is then the estimate, or `0` when there is none. `energy_kwh` is filled in a moment later, after the plug's reading, and is not in this message yet.

`run/discarded` (the `run/started` fields) says a run is gone without an end: a new print started on the printer while the run was still open, or its archive was deleted while it printed. Its `run_id` no longer exists after it.

Use `run_id` with `GET /archives/{archive_id}?run_id=` for everything else about the print.

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
  "big_fan2_speed": 50,
  "door_open": false,
  "active_extruder": 0,
  "tray_now": 0,
  "ams": [
    {
      "id": 0,
      "humidity": 4,
      "temp": 25.0,
      "tray": [
        {"id": 0, "tray_type": "PLA", "tray_color": "FF0000FF", "tray_sub_brands": "PLA Basic", "remain": 80}
      ]
    }
  ],
  "vt_tray": [],
  "hms_errors": [],
  "awaiting_plate_clear": false
}
```

### Door, Extruder, Filament and Errors

The status carries what an automation otherwise had to poll `GET /api/v1/printers/{id}/status` for ([#3401](https://github.com/maziggy/bambuddy/issues/3401)). Each field has the shape it has there:

| Field | Meaning |
|-------|---------|
| `door_open` | The enclosure door is open. Only printers with a door sensor report it; on the others it stays `false` |
| `active_extruder` | The nozzle that is printing on a dual-nozzle printer: `0` right, `1` left |
| `tray_now` | The tray feeding the print, numbered as Bambu Studio does: `ams_id * 4 + slot` on a regular AMS, the unit's `id` (128 and up) on an AMS-HT, the external spool's `id` (254, or 255 for the second one on a dual-nozzle printer). `255` means none on a printer with one external spool |
| `ams` | Every AMS unit with its trays: `tray_type` (the material), `tray_color` (RGBA hex), `tray_sub_brands`, `remain` and more. `null` without an AMS |
| `vt_tray` | The external spool(s), in the same tray shape |
| `hms_errors` | The active printer errors, each with `code`, `severity` and `description`. Empty when there are none |

For the material being printed, find the tray `tray_now` names and read its `tray_type`: on a regular AMS the unit in `ams` whose `id` is `tray_now // 4`, and in it the tray whose `id` is `tray_now % 4`; on an AMS-HT the unit whose `id` is `tray_now` (it has one tray); for an external spool the `vt_tray` entry with that `id`. The H3C numbers its AMS 3H and nozzle rack differently.

!!! note "Dropped printers are published at once"
    The status is throttled to one message a second per printer, except that a change of `connected` goes out straight away. The retained status of a printer that dropped says `"connected": false` even if a regular update was published a moment before.

### Plate Clear

`awaiting_plate_clear` is a Bambuddy-side flag, not printer telemetry: the printer itself only ever reports `RUNNING`, `PAUSE`, `FAILED`, `FINISH` and `IDLE`. It goes `true` when a print reaches a terminal state and stays `true` until someone confirms the build plate is free — the queue will not dispatch the next job in the meantime. See [Print Queue](print-queue.md) for the gate itself.

Because the status topic only refreshes when the printer pushes telemetry — which stops entirely once [Auto Off](smart-plugs.md) powers the printer down — every transition is also published on its own **retained** topic:

```json
{
  "printer_id": 1,
  "printer_name": "X1C-1",
  "printer_serial": "00M09C411500579",
  "awaiting": true,
  "timestamp": "2026-01-13T12:00:00.000000"
}
```

Subscribe to `bambuddy/printers/+/plate_clear` and you get the current state of every printer the moment you connect, then one message per change. Acknowledge from your own automation with `POST /api/v1/printers/{id}/clear-plate` (see the [API reference](../reference/api.md)).

!!! tip "Notification too"
    The same transition can raise a push notification — enable **Plate Clear Required** on a notification provider. It is off by default because it fires after every print, at the same moment as the print-complete notification. See [Notifications](notifications.md).

### Printer Offline and Online

`offline` is sent when a printer has stayed disconnected for a minute, so a short network blip sends nothing. `online` is sent when that printer connects again, and only after an `offline` went out, so an `online` never arrives on its own. If the broker is unreachable when the printer comes back, `online` follows once the relay has reconnected. Both carry the printer:

```json
{
  "printer_id": 1,
  "printer_name": "X1C-1",
  "printer_serial": "00M09C411500579",
  "timestamp": "2026-01-13T12:00:00.000000"
}
```

### Low Filament

Sent once when the spool assigned to an AMS slot drops below its low-stock threshold (the spool's own, or the global one, 20% by default; with Spoolman always the global one), at the same moment as the **Low Filament** notification. It is sent again only after the slot has gone back above the threshold, for example with a new spool, or after the relay was switched off and on again, which sends every slot that is low at that point. If the relay is only disconnected at that moment, the event follows once it reconnects.

```json
{
  "printer_id": 1,
  "printer_name": "X1C-1",
  "printer_serial": "00M09C411500579",
  "slot": "A1",
  "remaining_percent": 15,
  "color": "Jade White",
  "spool_id": 42,
  "timestamp": "2026-01-13T12:00:00.000000"
}
```

`color` is the spool's colour name, or `null` when it has none. `spool_id` is Bambuddy's spool ID, or Spoolman's when Spoolman manages the inventory. Only slots with an assigned spool are checked.

### Smart Plug Energy

Every enabled plug's readings are published once a minute while the relay is connected. A value the plug doesn't report is `null`. A plug that gives no reading at all is left out (a Home Assistant plug without a power value counts as that), and so is an MQTT plug that has stopped sending messages. An MQTT plug has a single energy value, the one from its energy topic, which is given as both `energy_today_kwh` and `energy_total_kwh`.

```json
{
  "plug_id": 3,
  "plug_name": "X1C plug",
  "power_watts": 182.4,
  "energy_today_kwh": 1.27,
  "energy_total_kwh": 315.8,
  "timestamp": "2026-01-13T12:00:00.000000"
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
