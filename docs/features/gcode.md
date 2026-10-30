# G-code Reference (Macro Context)

Whitelisted G-code is forwarded directly to the printer via MQTT. Use
`GET /api/v1/macros/gcode-whitelist` for the authoritative list of allowed commands.

Key safety rules enforced automatically:

- **Homing and heating** commands are blocked while a print is running
- Any G-code not in the whitelist is rejected and logged as an error

!!! warning "**Important**"
    The folowing list of commands is mostly **untested, unverified and incomplete!** Always run them under superivision. Some commands migth be printer-specific, wrong, or not in use anymore.

---

## Movement

| Command | Syntax | Description |
|---------|--------|-------------|
| `G0` | `G0 X<mm> Y<mm> Z<mm> F<mm/s>` | Rapid travel move (no extrusion) |
| `G1` | `G1 X Y Z E<mm> F<mm/s>` | Controlled linear move with optional extrusion |
| `G2` | `G2 X Y I<offset> J<offset> F E` | Clockwise arc move |
| `G3` | `G3 X Y I J F E` | Counter-clockwise arc move |
| `G380` | see below | **Bambu:** guarded Z move — S2=up, S3=down. Must follow `G91`. |

### G380 — Guarded Z Move

Likely a variant of G38 ("Probe Target"), customized for Bambu. Used when the nozzle rubs the build plate during cleaning and during early-startup Z positioning.

| S | Description |
|---|-------------|
| `S2` | Move Z up (guarded) |
| `S3` | Move Z down (guarded, gentle) |

Example from startup ("avoid end stop" section):

```gcode
G91
G380 S2 Z30 F1200
G380 S3 Z-20 F1200
G1 Z5 F1200
G90
```

---

## Homing & Positioning

| Command | Syntax | Description |
|---------|--------|-------------|
| `G28` | `G28 [X] [Y] [Z] [Z P0 T<C>]` | Home axes. Always home XY before Z to avoid collisions. |
| `G29` | `G29 A1 X<mm> Y<mm> I<w> J<h>` | Bed mesh calibration (does **not** follow standard Marlin G29) |
| `G29.1` | `G29.1 Z<mm>` | Set Z offset |
| `G29.2` | `G29.2 S0/S1` | Disable (S0) / enable (S1) ABL / bed mesh recording |
| `G39.4` | `G39.4` | Quick build plate type detection — flag-guarded; disable ABL first with `G29.2 S0` |
| `G90` | — | Absolute positioning mode |
| `G91` | — | Relative positioning mode (XY ignored via API) |
| `G92` | `G92 E0` | Set current position without moving (e.g. reset extruder) |

### G28 — Homing notes

`G28 Z P0 T300` homes Z with low precision while permitting a 300 C nozzle temperature.

### G29 — Bed mesh notes

Bambu does not follow Marlin's G29 manual. Known usage:

- `G29.2 S0` — turn off ABL
- `G29.2 S1` — likely enables recording of height when probe detects collision and/or loads saved calibration
- `G29 A1 X-1.4 Y82.7 I68.0 J98.7` — probable bed probing; usually guarded by `g29_before_print_flag`

### G39.4 — Build plate detection notes

Fires a canned command to detect the build plate type. Usually flag-guarded (`build_plate_detect_flag`) and followed by `M400`. ABL should be disabled first with `G29.2 S0`.

---

## Temperature

| Command | Syntax | Description |
|---------|--------|-------------|
| `M104` | `M104 S<C>` | Set hotend temperature (non-blocking) |
| `M109` | `M109 S<C>` | Set hotend temperature and **wait** (also updates screen to show "Heating") |
| `M140` | `M140 S<C>` | Set bed temperature (non-blocking) |
| `M190` | `M190 S<C>` | Set bed temperature and **wait** |

---

## Fan Control

| Command | Syntax | Description |
|---------|--------|-------------|
| `M106` | `M106 P<fan> S<0-255>` | Set fan speed. **P is required.** P1=part, P2=aux, P3=chamber. |
| `M107` | `M107 P<fan>` | Turn off fan (prefer `M106 P<n> S0` for reliability) |

---

## Motor Control

| Command | Syntax | Description |
|---------|--------|-------------|
| `M17` | `M17 [X Y Z E<amps>]` | Set stepper current (amps). Required before `M1006` sound and before `T` tool changes. |
| `M17` | `M17 S` | Enable all steppers (explicit enable, no current change) |
| `M17` | `M17 R` | Restore default stepper currents (X=1.2A, Y=1.2A, Z=0.75A) |
| `M18` | `M18` | Disable all steppers |
| `M18` | `M18 [X] [Y] [Z] [E]` | Disable steppers for specific axes only |
| `M84` | `M84` | Disable steppers (Marlin alias for M18) |

---

## Extrusion & Flow

| Command | Syntax | Description |
|---------|--------|-------------|
| `M82` | — | Extruder absolute mode |
| `M83` | — | Extruder relative mode (overrides G90 for E axis only) |
| `M900` | `M900 K<factor> L<limit>` | Pressure advance. K=0 disables. |
| `M220` | `M220 S<pct>` / `M220 K<mult>` | Feed rate override |
| `M302` | `M302 S70 P0/P1` | Cold extrusion protection toggle |

### M221 — Flow Rate & Soft Endstop Control (dual-use)

`M221` does two unrelated things depending on which parameters are passed:

**Flow rate override** (standard):

| Syntax | Description |
|--------|-------------|
| `M221 S100` | Reset flow rate to 100% |
| `M221 S<pct>` | Set flow rate to any percentage |

**Soft endstop control** (Bambu-specific):

| Syntax | Description |
|--------|-------------|
| `M221 S` | Push (save) current soft endstop state |
| `M221 R` | Pop (restore) previously saved soft endstop state |
| `M221 X0 Y0 Z0` | Disable soft endstops on all axes |
| `M221 Z0` | Disable Z soft endstop only |
| `M221 X1 Y1 Z1` | Re-enable soft endstops on all axes |

> `M221 S` with no value = save state. `M221 S100` with a number = set flow rate. These are different despite sharing the `S` param. Prefer `M211` for endstop control.

---

## Timing & Flow Control

| Command | Syntax | Description |
|---------|--------|-------------|
| `M400` | `M400 [S<s>] [P<ms>] [U1]` | Wait for moves to finish. `S#` sleeps ~# seconds, `P300` sleeps ~300 ms. `U1` pauses until user taps Resume. |
| `G4` | `G4 S<s>` / `G4 P<ms>` | Dwell (delay). Max 90 s per call — chain for longer waits. |

---

## AMS & Filament

### T — Tool / Filament Change

| Syntax | Description |
|--------|-------------|
| `T0`–`T3` | Load AMS tray by index (zero-indexed, matching GUI filament list) |
| `T255` | Unload filament / AMS retract (used at end of print) |
| `T1000` | Switch to local/nozzle tool (used twice during startup) |
| `T1100` | Switch to scanning space (**X1 only**) |

> **Important:** `T` orchestrates AMS motor, extruder, and sensors (retries load 3x, then errors). If a bare `M17` hasn't run since steppers were last reset, `T` will freeze — the printer may appear stuck and requires a power cycle. Always ensure motors are enabled before issuing `T`.

### AMS commands

| Command | Syntax | Description |
|---------|--------|-------------|
| `M620` | `M620 M` | Enable AMS remap mode (run once before first M620 command) |
| `M620` | `M620 S<n>A` | Start AMS branch — skip until matching `M621 S<n>A` if AMS disabled. Used for filament change during print. |
| `M620` | `M620 S<n>` | Start AMS branch — skip until matching `M621 S<n>` if AMS disabled. Used for final unload at end of print. `n=255` for unload. |
| `M620` | `M620 C<n>` | Calibrate AMS by AMS index |
| `M620` | `M620 R<n>` | Refresh AMS by tray index |
| `M620` | `M620 P<n>` | Select AMS tray by tray index |
| `M620.1` | `M620.1 E F<speed> T<C>` | Set extrusion parameters used by `T` during load/unload. Used before and after `T`. |
| `M620.1` | `M620.1 X<n> Y<n> F<n> P<0/1/2>` | Travel path for filament change (X1 + AMS) |
| `M620.3` | `M620.3 W0/W1` | Tangle detection toggle (W1=on) |
| `M620.10` | `M620.10 A0 F<speed>` | Run just before `T` during filament changes (not initial load/unload). Part of long-retraction feature. |
| `M620.10` | `M620.10 A1 F<speed> L<flush_len> H<nozzle_dia> T<max_temp>` | Run shortly after `T` during filament change. |
| `M620.11` | `M620.11 S1 I<prev_ext> E-<dist> F1200` | Run before `T` if current filament has long retraction enabled. |
| `M620.11` | `M620.11 S1 I<prev_ext> E<dist> F<speed>` | Run after `T` if old filament had long retraction enabled. |
| `M620.11` | `M620.11 S0` | Run when filament being unloaded does not have long retraction. |
| `M621` | `M621 S<n>A` | End of `M620 S<n>A` branch |
| `M621` | `M621 S<n>` | End of `M620 S<n>` branch |
| `M412` | `M412 S0/S1` | Filament runout detection |
| `G392` | `G392 S0/S1` | Clog detection toggle (behavior uncertain — seen commented as both enable and disable at different points) |

### M622 / M623 — Conditional blocks

| Command | Syntax | Description |
|---------|--------|-------------|
| `M622` | `M622 J1` | If last `M1002 judge_flag` was **true**, continue; else skip to `M623` |
| `M622` | `M622 J0` | If last `M1002 judge_flag` was **false**, continue; else skip to `M623` |
| `M622.1` | `M622.1 S0` | Run just before dynamic extrusion (`M9833`) |
| `M622.1` | `M622.1 S1` | Turns on whatever `M622.1 S0` disables (usually already on by default) |
| `M623` | `M623` | End of `M622` conditional block |

---

## Display & Sound

| Command | Syntax | Description |
|---------|--------|-------------|
| `M73` | `M73 P<pct> R<min>` | Update print progress on display |
| `M73.2` | `M73.2 R<mult>` | Reset left-time magnitude (e.g. `M73.2 R1.0` at print start) |
| `M1006` | `M1006 S1` then note params, `M1006 W` | Play sound via stepper coils. Requires `M17` first; `M1006 W` blocks until done. No-op if tones are disabled in printer config. |

### M1002 — LCD / Flag Control

| Syntax | Description |
|--------|-------------|
| `M1002 gcode_claim_action : 0` | Clear LCD status message |
| `M1002 gcode_claim_action : 1` | Display "Auto bed levelling" |
| `M1002 gcode_claim_action : 2` | Display "Heatbed preheating" |
| `M1002 gcode_claim_action : 3` | Display "Sweeping XY mech mode" |
| `M1002 gcode_claim_action : 4` | Display "Changing filament" |
| `M1002 gcode_claim_action : 5` | Display "M400 pause" |
| `M1002 gcode_claim_action : 6` | Display "Paused due to filament runout" |
| `M1002 gcode_claim_action : 7` | Display "Heating hotend" |
| `M1002 gcode_claim_action : 8` | Display "Calibrating extrusion" |
| `M1002 gcode_claim_action : 9` | Display "Scanning bed surface" |
| `M1002 gcode_claim_action : 10` | Display "Inspecting first layer" |
| `M1002 gcode_claim_action : 11` | Display "Identifying build plate type" |
| `M1002 gcode_claim_action : 12` | Display "Calibrating Micro Lidar" |
| `M1002 gcode_claim_action : 13` | Display "Homing toolhead" |
| `M1002 gcode_claim_action : 14` | Display "Cleaning nozzle tip" |
| `M1002 gcode_claim_action : 15` | Display "Checking extruder temperature" |
| `M1002 gcode_claim_action : 16` | Display "Paused by the user" |
| `M1002 gcode_claim_action : 17` | Display "Paused — front cover fell off" |
| `M1002 gcode_claim_action : 18` | Display "Calibrating the micro lidar" |
| `M1002 gcode_claim_action : 19` | Display "Calibrating extruder flow" |
| `M1002 gcode_claim_action : 20` | Display "Paused — nozzle temperature malfunction" |
| `M1002 gcode_claim_action : 21` | Display "Paused — heat bed temperature malfunction" |
| `M1002 set_filament_type:<type>` | Update filament type on display (e.g. `PLA`, `PETG`, `TPU`, `UNKNOWN`) |
| `M1002 judge_flag <flag>` | Evaluate flag for following `M622` branch |
| `M1002 set_gcode_claim_speed_level` | Update speed profile on LCD (default 5) |

Common `judge_flag` names:

| Flag | Meaning |
|------|---------|
| `build_plate_detect_flag` | Build plate detection is enabled |
| `g29_before_print_flag` | Automatic bed leveling before print is enabled |
| `extrude_cali_flag` | Flow dynamics calibration is enabled |
| `filament_need_cali_flag` | Filament needs calibration |
| `timelapse_record_flag` | Timelapse is enabled |
| `g39_3rd_layer_detect_flag` | 3rd layer detection is enabled |

---

## Limits & Calibration

| Command | Syntax | Description |
|---------|--------|-------------|
| `M201` | `M201 Z<mm/s2>` | Max acceleration (Z axis) |
| `M204` | `M204 S<mm/s2>` | Default acceleration |
| `M204.2` | `M204.2 K<mult>` | Acceleration multiplier (default 1.0) |
| `M205` | `M205 X Y Z E<mm/s>` | Jerk limits per axis |
| `M211` | `M211 [S] [X0/X1] [Y0/Y1] [Z0/Z1] [R]` | Soft endstop enable/disable. S=save, R=restore. |
| `M500` | `M500` | Save settings to EEPROM |
| `M975` | `M975 S0/S1` | Vibration compensation toggle (S1=on; may need repeated calls — something can reset it) |
| `M982` | `M982 Q P V D L T I` | Motor noise cancellation (full parameter set, partially documented) |
| `M982.2` | `M982.2 C0/C1` | Motor cog noise reduction toggle (C0=off, C1=on) |
| `M982.4` | `M982.4 S<val> V<val>` | Motor noise cancellation parameter setter (partially documented) |
| `M1003` | `M1003 S0/S1` | Power loss recovery toggle (S0=disable, S1=enable) |
| `M1005` | `M1005 X Y` / `M1005 I<rad>` / `M1005 P0/P1` | Skew calibration — set from diagonals, direct radians, or enable/disable |
| `M290` | `M290 X<mm> Y<mm>` | XY dimensional compensation offset |
| `M290.2` | `M290.2 X<mm> Y<mm>` | XY compensation (variant — distinction from M290 unclear) |

---

## LEDs, Camera & Detection

| Command | Syntax | Description |
|---------|--------|-------------|
| `M960` | `M960 S4 P0/P1` | Nozzle LED on (P1) / off (P0) |
| `M960` | `M960 S5 P0/P1` | Toolhead logo LED on (P1) / off (P0) |
| `M960` | `M960 S1 P0/P1` | Horizontal laser on/off |
| `M960` | `M960 S2 P0/P1` | Vertical laser on/off |
| `M971` | `M971 S<mode> P<exposure>` | Capture image (e.g. for timelapse) to `/userdata/log/` |
| `M972` | `M972 S5 P0` | Measure Xcam clarity |
| `M973` | `M973 S1` | Nozzle cam autoexpose |
| `M973` | `M973 S2 P16000` | Perform scan |
| `M973` | `M973 S3 P1` | Start camera stream / nozzle cam on |
| `M973` | `M973 S4` | Disable scanner / nozzle cam off |
| `M973` | `M973 S6 P0` | Auto-expose for horizontal laser |
| `M973` | `M973 S6 P1` | Auto-expose for vertical laser |
| `M976` | `M976 S1 P1` | First layer scan |
| `M976` | `M976 S2 P1` | Hot bed scan before print |
| `M976` | `M976 S3 P2` | Register void printing detection |
| `M981` | `M981 S0/S1 P20000` | Spaghetti detector toggle (S0=off, S1=on) |
| `M991` | `M991 S0 P<layer>` | Notify layer change |
| `M991` | `M991 S0 P-1` | End smooth timelapse at safe position |

---

## Internal / State Commands

| Command | Syntax | Description |
|---------|--------|-------------|
| `M630` | `M630 S0 P0` | Reset machine internal state (used early in "reset machine status" section) |
| `M1007` | `M1007 S0` | Unknown — run after clog detect disable (e.g. during filament change) |
| `M1007` | `M1007 S1` | Unknown — run before printing, after filament change and after initialization |
| `M9833.2` | `M9833.2` | Unknown — follows clog detect disable at startup; likely sets internal noise params |

---
