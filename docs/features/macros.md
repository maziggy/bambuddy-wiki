# Macros

Macros let you automate printer actions — drying filament, sending notifications, managing spool assignments, controlling prints — using a simple scripting syntax powered by Jinja2 templates.

---

Navigate to **Macros** in the sidebar (Code icon). The page has two sections:

- **Files** — the `.cfg` files on disk that contain your macro definitions
- **Macros** — the individual macros parsed from those files

---

## File format

Macros are stored in `.cfg` files in `data/macros/`. Each file can contain multiple macros. Create a new file via the **New File** button (plus icon) in the UI, or drop a `.cfg` file directly into `data/macros/`.

An example:

```ini
[macro preheat_bed]
description: Heat bed and wait
trigger: manual
printer: My X1C

M140 S65
WAIT_FOR_TEMP --target=65 --tolerance=2 --sensor=bed
NOTIFY --message="Bed ready"

[macro filament_check]
trigger: schedule
cron: 0 9 * * *

{% if vars.last_spool != assignments[0].spool_id %}
SET_VAR --key=last_spool --value="{{ assignments[0].spool_id }}"
NOTIFY --message="Spool changed on AMS slot 0"
{% endif %}
```

### Config headers

Each `[macro name]` block can start with optional config lines before the script body:

| Key | Values | Description |
|---|---|---|
| `description` | any text | Shown in the macro list |
| `trigger` | `manual`, `webhook`, `schedule` | How the macro is invoked |
| `cron` | cron expression | Required when `trigger: schedule` (e.g. `0 8 * * *` = 8am daily) |
| `printer` | printer name | Which printer to target by default |

---

## Triggers

| Type | How it runs |
|---|---|
| **Manual** | Click **Run** in the Macros page or `POST /api/v1/macros/{id}/run` |
| **Schedule** | Runs automatically on a cron schedule (checked every 60s) |
| **Webhook** | `POST /api/v1/webhook/macro/{id}/run` with your API key |

---

## Jinja2 templating

Scripts are rendered as Jinja2 templates before execution. You can use conditionals, loops, and variable interpolation.
The full documentation is avialable on Jinja [official docs](https://jinja.palletsprojects.com/en/stable/templates/#).

### Available context variables

| Variable | Type | What it contains |
|---|---|---|
| `printer` | dict | Live printer state: `printer.nozzle_temp`, `printer.bed_temp`, `printer.state`, `printer.model` |
| `ams` | list | AMS unit list, each with a `tray` list containing slot data |
| `queue` | int | Number of pending items in the print queue |
| `vars` | dict | Persistent variables set with `SET_VAR` |
| `assignments` | list | Current spool assignments: `ams_id`, `tray_id`, `spool_id`, `material`, `color`, `brand` |

**Example:**
```jinja
{% if printer.nozzle_temp > 180 %}
NOTIFY --message="Nozzle still hot: {{ printer.nozzle_temp }}C"
{% endif %}
```

---

## System commands

System commands are written one per line, using `--key=value` flags.

### Printer control

> These commands cannot run inside G-code embeds.

```
PRINTER_PAUSE
PRINTER_RESUME
PRINTER_STOP
```

```
AMS_DRYING --ams=0 --temp=65 --duration=4
```

- `--ams`: AMS unit index (0–3)
- `--temp`: drying temperature in °C (20–90)
- `--duration`: duration in hours (1–12)

```
WAIT_FOR_TEMP --target=60 --tolerance=2 --max_wait=300 --sensor=nozzle
```

- `--sensor`: which heater to read — `nozzle` (default), `bed`, or `chamber`
- `--target`: target nozzle temperature in °C (0–350)
- `--tolerance`: acceptable delta in °C (default 2)
- `--max_wait`: timeout in seconds (default 300)

```
CLEAR_HMS_ERRORS
```

```
PRINT_QUEUE_ADD --file_id=42 --plate=1
```

### Notifications & timing

```
NOTIFY --message="Print complete"
```

```
WAIT --seconds=30
```

### Spool assignments

```
ASSIGN_SPOOL --spool_id=3 --ams=0 --tray=1
UNASSIGN_SPOOL --ams=0 --tray=1
```

### Persistent variables

Variables persist across macro runs and restarts.

```
SET_VAR --key=my_count --value=1
SET_VAR --key=label --value="PLA" --scope=macro --ttl=3600
DELETE_VAR --key=my_count
```

- `--scope`: `global` (default, shared across all macros) or `macro` (isolated to this macro)
- `--ttl`: auto-expiry in seconds (omit for permanent)

Use stored values in templates via `{{ vars.my_count }}`.

---

## G-code commands

Whitelisted G-code is forwarded directly to the printer via MQTT. Key safety rules:

- **Homing and heating** commands are blocked while a print is running
- Any G-code not in the whitelist is rejected and logged as an error

See the full [G-code command reference](gcode.md) for all allowed commands, syntax, and Bambu-specific caveats.

---

## Calling another macro

From within a script, call another macro by name:

```jinja
{{ run_macro("preheat_bed") }}
```

The called macro runs inline and its output is appended to the parent run log. Circular calls (A → B → A) are detected and blocked.

---

## Terminal

The **Terminal** icon on the Printer page and printer card lets you execute individual lines immediately. Each terminal has its own execution session, and is opened in a new browser window. 

- Type a raw G-code line (e.g. `G28`) to dispatch it directly
- Type `run: macro_name` to invoke a full macro by name
- Type `@` to target a sepcific printer, or `@all` to target all the machines

The terminal reuses  current session — no separate login required. 

---

## Run history

Each macro execution creates a **MacroRun** record with:

- Status: `pending` → `running` → `success` or `error`
- Full execution log with timestamps
- Which printer was targeted and what triggered the run

Run history is shown in the **Runs** tab of each macro. Active runs can be cancelled via the **Cancel** button.

---

## Permissions

| Action | Administrators | Operators | Viewers |
|---|---|---|---|
| View macros & run history | yes | yes | no |
| Run a macro | yes | yes | no |
| Create / edit / delete macros | yes | no | no |

---

## Tips

- **Test in the terminal first** before putting commands in a scheduled macro.
- **Use `--scope=macro`** for variables that should not leak between different macros.
- **Use `--ttl`** on `SET_VAR` for state that should reset automatically (e.g. a cooldown flag).
- **Duplicate block names** in the same `.cfg` file will cause a parse error shown in the Files list — only the first block is used.
- The `NOTIFY` command is a no-op if no notification providers are configured — it will not error.
