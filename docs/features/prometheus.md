# Prometheus Metrics

Bambuddy can expose printer telemetry in Prometheus format, enabling integration with **Grafana**, **Prometheus**, and other monitoring systems.

## Configuration

Navigate to **Settings → Network → Prometheus Metrics** to enable the endpoint.

### Settings

| Setting | Description | Default |
|---------|-------------|---------|
| **Enable Metrics Endpoint** | Toggle metrics endpoint on/off | Off |
| **Bearer Token** | Optional authentication token | Empty (no auth) |

## Endpoint

```
GET /api/v1/metrics
```

Returns metrics in [Prometheus text exposition format](https://prometheus.io/docs/instrumenting/exposition_formats/).

### Authentication

If a bearer token is configured, requests must include the `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" http://bambuddy:8000/api/v1/metrics
```

Without a token configured, the endpoint is publicly accessible (when enabled).

---

## Available Metrics

### Printer Connection

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_printer_connected` | gauge | Connection status (1=connected, 0=disconnected) |
| `bambuddy_printer_state` | gauge | Printer state (0=unknown, 1=idle, 2=running, 3=pause, 4=finish, 5=failed) |

### Print Progress

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_print_progress` | gauge | Current print progress (0-100%) |
| `bambuddy_print_remaining_seconds` | gauge | Estimated remaining time in seconds |
| `bambuddy_print_layer_current` | gauge | Current layer number |
| `bambuddy_print_layer_total` | gauge | Total layers in current print |

### Temperatures

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_bed_temp_celsius` | gauge | Current bed temperature |
| `bambuddy_bed_target_celsius` | gauge | Target bed temperature |
| `bambuddy_nozzle_temp_celsius` | gauge | Current nozzle temperature (label: nozzle=0/1) |
| `bambuddy_nozzle_target_celsius` | gauge | Target nozzle temperature |
| `bambuddy_chamber_temp_celsius` | gauge | Chamber temperature (X1/H2/P2 series only) |

### Fans

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_fan_speed_percent` | gauge | Fan speed (label: fan=part/aux/chamber) |

### Network

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_wifi_signal_dbm` | gauge | WiFi signal strength in dBm |

### Statistics (Counters)

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_prints_total` | counter | Total prints by result (label: result=completed/failed/etc) |
| `bambuddy_printer_prints_total` | counter | Total prints per printer |
| `bambuddy_filament_used_grams` | counter | Total filament used in grams |
| `bambuddy_print_time_seconds` | counter | Total print time in seconds |

### Queue

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_queue_pending` | gauge | Number of pending queue items |
| `bambuddy_queue_printing` | gauge | Number of currently printing queue items |

### System

| Metric | Type | Description |
|--------|------|-------------|
| `bambuddy_printers_connected` | gauge | Total connected printers |
| `bambuddy_printers_total` | gauge | Total configured printers |

---

## Labels

Most metrics include labels for filtering:

| Label | Description |
|-------|-------------|
| `printer_id` | Numeric printer ID |
| `printer_name` | Human-readable printer name |
| `serial` | Printer serial number |
| `model` | Printer model (X1C, H2D, etc.) |
| `nozzle` | Nozzle index (0 or 1) for dual-nozzle printers |
| `fan` | Fan type (part, aux, chamber) |
| `result` | Print result (completed, failed, archived, etc.) |

---

## Prometheus Configuration

Add Bambuddy to your `prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'bambuddy'
    scrape_interval: 15s
    metrics_path: '/api/v1/metrics'
    static_configs:
      - targets: ['bambuddy-host:8000']
    # If using bearer token:
    bearer_token: 'YOUR_TOKEN'
```

For Docker networks, use the container name or `host.docker.internal`:

```yaml
static_configs:
  - targets: ['host.docker.internal:8000']
```

---

## Grafana Dashboard

### Example Queries

**Printer temperatures:**
```promql
bambuddy_bed_temp_celsius{printer_name="X1C-1"}
bambuddy_nozzle_temp_celsius{printer_name="X1C-1", nozzle="0"}
```

**Print progress:**
```promql
bambuddy_print_progress
```

**Success rate (last 24h):**
```promql
increase(bambuddy_prints_total{result="completed"}[24h]) /
(increase(bambuddy_prints_total{result="completed"}[24h]) + increase(bambuddy_prints_total{result="failed"}[24h]))
```

**Filament usage rate:**
```promql
rate(bambuddy_filament_used_grams[1h]) * 3600
```

### Sample Dashboard Panels

1. **Stat Panel**: Printers online (`bambuddy_printers_connected`)
2. **Gauge Panel**: Print progress per printer
3. **Time Series**: Temperature graphs (bed, nozzle, chamber)
4. **Time Series**: WiFi signal strength over time
5. **Bar Chart**: Prints by result (`bambuddy_prints_total`)
6. **Table**: Printer status overview

---

## Example Output

```
# HELP bambuddy_printer_connected Printer connection status (1=connected, 0=disconnected)
# TYPE bambuddy_printer_connected gauge
bambuddy_printer_connected{printer_id="1",printer_name="X1C-1",serial="00M09C411500579",model="X1C"} 1

# HELP bambuddy_printer_state Printer state
# TYPE bambuddy_printer_state gauge
bambuddy_printer_state{printer_id="1",printer_name="X1C-1",serial="00M09C411500579"} 2

# HELP bambuddy_bed_temp_celsius Current bed temperature
# TYPE bambuddy_bed_temp_celsius gauge
bambuddy_bed_temp_celsius{printer_id="1",printer_name="X1C-1",serial="00M09C411500579"} 60.0

# HELP bambuddy_nozzle_temp_celsius Current nozzle temperature
# TYPE bambuddy_nozzle_temp_celsius gauge
bambuddy_nozzle_temp_celsius{printer_id="1",printer_name="X1C-1",serial="00M09C411500579",nozzle="0"} 220.0

# HELP bambuddy_prints_total Total number of prints by result
# TYPE bambuddy_prints_total counter
bambuddy_prints_total{result="completed"} 342
bambuddy_prints_total{result="failed"} 18

# HELP bambuddy_filament_used_grams Total filament used in grams
# TYPE bambuddy_filament_used_grams counter
bambuddy_filament_used_grams 2042.0

# HELP bambuddy_printers_connected Number of connected printers
# TYPE bambuddy_printers_connected gauge
bambuddy_printers_connected 2
```

---

## Troubleshooting

### Endpoint Returns 404

Prometheus metrics are disabled. Enable in **Settings → Network → Prometheus Metrics**.

### Endpoint Returns 401

A bearer token is configured. Include the `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" http://bambuddy:8000/api/v1/metrics
```

### No Printer Metrics

If printer-specific metrics are missing:

- Verify printers are connected (check Bambuddy dashboard)
- Check that printers are marked as active

### Prometheus Shows "Down"

- Verify Bambuddy is reachable from Prometheus
- Check firewall rules allow port 8000
- Verify the bearer token matches (if configured)
