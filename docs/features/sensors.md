---
title: Sensors
description: Bind read-only Home Assistant sensors to printers or storage locations for alerts, notifications, and live readings
---

Bambuddy can bind read-only Home Assistant sensors — a door contact, a
thermometer, a humidity/battery sensor — to either a **printer** or a
**storage location**. Both work the same way: pick an entity, optionally set
an alert condition, and the reading shows up where it's relevant, with
alerts and notifications on top.

![Settings — Sensors overview](/assets/settings-sensors-overview.png){ .screenshot }

---

## :material-printer: Printer Sensors

Sensors are things you read: an enclosure door contact, a chamber
thermometer, a smoke detector. Bind them to a printer and their state
appears on that printer's card.

Sensors use the same Home Assistant URL and token as [Smart Plugs](smart-plugs.md), so if
plugs already work there is nothing else to set up. If Home Assistant is not
connected yet, the sensor dialog says so and the entity list stays empty until
it is.

### Adding a Sensor

1. Go to **Settings** > **Sensors**
2. Under **Home Assistant Sensors (Printers)**, click **Add Sensor**
3. Pick the printer
4. Pick the entity — the list offers every `binary_sensor` plus the `sensor`
   entities that carry a reading
5. Give it a display name (prefilled from Home Assistant's friendly name)
6. Optionally set an alert condition
7. Save

**Supported entities:**

| Domain | Example | Shown as |
|--------|---------|----------|
| `binary_sensor` | `binary_sensor.enclosure_door` | Open / Closed |
| `binary_sensor` | `binary_sensor.workshop_smoke` | Detected / Clear |
| `sensor` | `sensor.enclosure_temp` | `41.2 °C` |
| `sensor` | `sensor.workshop_humidity` | `48 %` |

The wording follows Home Assistant's own device class, so a door reads
"Open" rather than "On". Entities with no device class fall back to On / Off.

### Alert Conditions

An alert condition is what "needs attention" means for that sensor:

- **Binary sensors** — alert when the sensor is on, or when it is off. A door
  contact alerts when open; a "fan running" contact alerts when it stops.
- **Numeric sensors** — alert above a value, below a value, or both.

A sensor with no alert condition is display-only, which is the default.

When a sensor is alerting, its pill on the printer card turns red, and two
further options become available.

### Notify on Alert

Sends a notification the moment the sensor enters its alert state. Enable
**Sensor Alert** on the notification provider in **Settings** >
**Notifications** as well — the per-sensor switch decides *which* sensors
speak, the provider decides *where*.

The notification fires on the transition into the alert state, not repeatedly
while it lasts, and a sensor that drops off the network and comes back while
still alerting does not re-announce itself.

### Hold Prints While Alerting

Holds queued jobs for that printer while the sensor is alerting — the reason
this feature exists, for the enclosure door you meant to shut before starting
a print from your phone.

!!! info "A hold, never a failure"
    Held jobs stay in the queue with a reason you can read on the Queue page
    ("Waiting on Enclosure Door") and start by themselves as soon as the sensor
    clears. Nothing is cancelled.

For a job queued as "Any \<model\>", a held printer is simply passed over: the
job runs on a sibling printer whose sensors are clear rather than waiting.

!!! warning "Home Assistant must be reachable"
    A sensor Bambuddy cannot read holds nothing and alerts on nothing. If Home
    Assistant is down, the queue keeps running as though the interlock were not
    configured — a printer farm that stops because an unrelated service went
    offline would be worse than the problem being guarded against.

### Polling

Sensors are read every 15 seconds, and the printer cards serve that cached
reading, so the cost to Home Assistant does not grow with the number of
browser tabs you have open.

---

## :material-map-marker: Storage Location Sensors

Bind a Home Assistant temperature, humidity, or battery sensor to a storage
location — a drybox, a bin, a shelf — and its reading shows up on the
filament card and in the inventory table for every spool stored there. This
works the same way as [Printer Sensors](#printer-sensors) above, just scoped
to a storage location instead of a printer.

![Filament card with a location sensor footer](/assets/inventory-location-sensors-card.png){ .screenshot }

The inventory table gains sortable Temperature, Humidity, and Battery
columns, colorized the same way as the filament card:

![Inventory table with Temperature, Humidity, and Battery columns](/assets/inventory-location-sensors-table.png){ .screenshot }

### Adding a Sensor

1. Go to **Settings** > **Sensors**
2. Under **Home Assistant Sensors (Storage Locations)**, click **Add Sensor**
3. Pick the storage location
4. Pick the entity — the list offers `sensor` entities carrying a
   temperature, humidity, or battery reading
5. Optionally set an alert threshold and enable **Notify on alert**
6. Leave **Show on filament card** on to surface the reading on spool cards
   at that location; turn it off to keep the binding for alerts only
7. Save

**Supported entities:**

| Domain | Example | Shown as |
|--------|---------|----------|
| `sensor` | `sensor.drybox_1_temperature` | `24.7 °C` |
| `sensor` | `sensor.drybox_1_humidity` | `43.6 %` |
| `sensor` | `sensor.drybox_1_battery` | `78.1 %` |

The entity's device class determines which category it's treated as
(temperature, humidity, or battery) and which unit is shown.

!!! info "One sensor per category per location"
    Each storage location can hold at most one temperature, one humidity,
    and one battery sensor. Binding a second sensor of the same category
    prompts to replace the existing one.

### Auto-Adding Related Sensors

Many drybox sensors report temperature, humidity, and battery as separate
entities on the same device, following a shared `entity_id` prefix — for
example `sensor.drybox_1_temperature`, `sensor.drybox_1_humidity`, and
`sensor.drybox_1_battery`. When you bind the **first** sensor for a location
and Bambuddy finds matching siblings on the same device, it offers to bind
those too — each with its own checkbox, checked by default.

### Sensor Options

![Location Sensor Options dialog](/assets/inventory-location-sensors-options.png){ .screenshot .centered style="max-width: 420px" }

Click the gear icon next to **Add Sensor** to open **Location Sensor
Options**:

| Setting | Description | Default |
|---|---|---|
| Temperature above / below | Default alert thresholds applied to newly auto-added temperature sensors | 30 °C / 20 °C |
| Humidity above / below | Default alert thresholds for humidity | 30% / 10% |
| Battery below | Default low-battery threshold (batteries have no "above" threshold) | 10% |
| Update interval | How often Bambuddy polls Home Assistant and refreshes values on screen | 120s (minimum 60s) |
| Colorize sensor values | Color a reading against its threshold — one color above, one below, one within range | On |

!!! tip "Bulk-apply thresholds"
    The **Reset** button in this dialog applies the values currently on
    screen to every existing location sensor of that category — useful
    after tightening a threshold across several dryboxes at once. This can
    take a moment with many sensors bound.

### Notifications

Alerts use the same notification providers as other Bambuddy alerts
(**Settings** > **Notifications**) with the **Home Assistant Sensor Alert**
event enabled. A notification fires on the transition into the alert state
only, not repeatedly while it lasts, and not immediately on restart for a
sensor that was already alerting before Bambuddy started up.

### Polling

Sensors are read on the interval configured above (120 seconds by default),
and the poller runs once immediately on startup so a restart doesn't leave
the reading blank until the first cycle finishes.

Storage locations don't change nearly as fast as a running print, so there's
no need to poll as often as printer sensors do — a drybox's temperature and
humidity drift slowly, which is why the default interval here is far longer
than the fixed 15-second cadence used for printer sensors.
