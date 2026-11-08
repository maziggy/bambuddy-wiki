---
title: WLED
description: Show each printer's status on a WLED light strip by switching presets saved in WLED
---

# WLED

Bambuddy can use a WLED controller for each printer and activate existing WLED presets when the printer state changes. Colours, effects, and segments are configured in WLED itself. Bambuddy only maps printer states to saved presets and activates them.

## Before you start

- Make sure the WLED controller is reachable over the network from the environment running Bambuddy.
- Create and save the desired colours and effects as presets in WLED before configuring Bambuddy.
- Use a WLED URL with an IP address or a reachable hostname. A `.local` hostname requires working mDNS resolution in Bambuddy's runtime environment; it may not resolve inside Docker.
- The URL must start with `http://` or `https://` and must not contain a username or password.
- Configure the integration separately for each printer.

## Enable WLED for a printer

1. Open **Settings → WLED**.
2. Select the printer.
3. Turn on **Enable WLED integration**.
4. Enter the **WLED URL**.
5. Click **Test connection** to check that Bambuddy can reach the controller.
6. Click **Load presets** to retrieve the existing presets from WLED.
7. Assign the desired printer states to those presets.
8. Click **Save**.

Not every state needs a preset. Select **Disabled — no preset** to leave a state unmapped. When an unmapped state occurs, Bambuddy uses the next state that applies and has a preset (see the priority order below). For example, without a **Filament problem** preset a filament runout shows the **Paused** preset. If no applicable state has a preset, the strip keeps its current preset.

## Printer state mappings

| State | When it applies |
|-------|-----------------|
| Idle | The printer is connected and reports `IDLE`, with no higher-priority condition. |
| Preparing | The printer reports `PREPARE` or `SLICING`. |
| Printing | The printer reports `RUNNING` or `PRINTING`. |
| Paused | The printer reports `PAUSE` for any reason other than a filament runout, for example a pause by the user. |
| Finished | The printer reports `FINISH`. |
| Failed / Error | The printer reports `FAILED`. |
| Awaiting plate clear | **Require plate-clear confirmation** is enabled (**Settings → Workflow → Queue & Dispatch**) and the printer waits for the plate to be confirmed clear, while it reports `IDLE`, `FINISH`, or `FAILED`. With the setting off, a finished print shows **Finished** and a failed one **Failed / Error**. |
| Filament problem | The printer paused because the filament ran out. |
| HMS error | The printer reports a fault that the printer card shows as an error. Notices such as "the top cover is open" do not count. |
| Offline | The printer is not connected to Bambuddy, was disconnected, or is set inactive. |

Special conditions can override the normal printer state. In priority order, Bambuddy checks Offline, Filament problem, HMS error, and Awaiting plate clear before using the normal state mapping.

## Finished timeout

Set **Finished timeout (seconds)** to automatically activate the configured **Idle** preset after the printer has remained in the effective **Finished** state for that many seconds.

- Set the value to **0** to disable the timeout.
- An **Idle** preset must be configured for the timeout to work.
- If the printer state changes before the timeout expires, the pending timeout is cancelled; it does not force the Idle preset over the new state.

## When WLED is unreachable

A WLED controller that is switched off or not reachable never holds up the printer. Bambuddy retries the current preset every 30 seconds until WLED answers, and a change of printer state is sent at once. The log gets one warning when WLED stops answering and one line when it answers again.

## Shared WLED strips

Several printers can share one WLED controller and LED strip. Enter the same controller URL for each printer, then configure the presets in WLED:

1. Create one segment per printer.
2. Set the desired colour or effect for that printer's segment.
3. Check only that segment and save a preset with **Checked segments only** enabled.
4. Repeat for the desired states, then map each printer to its corresponding presets in Bambuddy.

Bambuddy activates the saved preset. If the preset contains only the intended segment, a status change for one printer does not overwrite the other printers' segments. Keep shared controller settings, such as global brightness, out of these presets if they would affect the other printers. Bambuddy does not create or manage the segments.

## WLED preset limit

WLED supports up to **250 presets** per controller. Bambuddy offers the ten mappable states listed above. If every state has its own unique preset, each printer uses **10 presets**, giving an approximate maximum of **250 / 10 = 25 printers** based on preset slots alone.

Fewer mapped states or shared presets use fewer slots. Other presets on the controller also consume slots, and a shared-strip setup must fit within the controller's supported segment count. Bambuddy does not remove or bypass WLED's preset limit.

## Permissions

Changing WLED configuration requires **`printers:update`**. With advanced authentication and printer groups, this permission is scoped to the printer: users can only change WLED settings for printers they are allowed to update.
