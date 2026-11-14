---
title: BIQU Panda Breath
description: Show and control a BIQU Panda Breath chamber heater on the card of the printer it is bound to
---

# BIQU Panda Breath

The BIQU Panda Breath is a chamber heater and air filter. Bambuddy shows the chamber temperature it reads on the printer card, and lets you switch it on or off, change its mode and set its target temperature from there.

Bambuddy talks to the Panda Breath through an MQTT broker, the same one it uses for [MQTT Publishing](mqtt.md). The Panda Breath must be bound to its printer in its own web interface: that is how Bambuddy knows which printer card to show it on.

## Before you start

- **Firmware 1.0.4 or later** on the Panda Breath. Earlier versions cannot connect to an MQTT broker. Firmware is available from [BIQU](https://github.com/bigtreetech/Panda_Breath).
- **An MQTT broker** that both Bambuddy and the Panda Breath can reach, for example Mosquitto.
- **The Panda Breath bound to its printer** in the Panda Breath web interface. An unbound device is listed in Bambuddy but not shown on any printer card.

## Set it up

1. In the Panda Breath web interface, open the **Home Assistant** page. Enter the broker address, port and, if your broker needs them, username and password. Click **Bind Home Assistant**.
2. In Bambuddy, open **Settings → Network → MQTT Publishing**. Turn on MQTT and enter the same broker.
3. In the same card, turn on **BIQU Panda Breath**.

After a few seconds, each Panda Breath that reports to the broker appears under the switch, with the printer it is bound to. A red dot means the device is offline. A Panda Breath bound to a printer you have no access to is listed without the printer's name.

## On the printer card

A **Panda Breath** tile appears next to the printer's temperatures. It shows the chamber temperature the Panda Breath reads and, while it is on, its target. The Panda Breath keeps running when the printer is switched off, so the tile stays on the card then too. Click the tile to:

- turn it on or off,
- choose a mode: **Auto** (the Panda Breath follows the print), **Always on** or **Drying**,
- set a target temperature from 0 to 60 °C.

Changes need the **Control printers** permission for that printer. The tile is read-only without it, and while the Panda Breath is offline. If Bambuddy loses the connection to the broker, the tile shows the Panda Breath as offline until it is back.

Filament drying presets and timers are set in the Panda Breath web interface. Bambuddy shows the remaining drying time in the tile's tooltip.

## Troubleshooting

- **No device under the switch:** check that the Panda Breath runs firmware 1.0.4 or later and is bound to Home Assistant with the same broker as Bambuddy. Its messages are published under `panda_breath/<device>/state`, so `mosquitto_sub -h <broker> -t 'panda_breath/#' -v` shows whether it reaches the broker.
- **Listed as "not bound to a printer":** bind the Panda Breath to its printer in the Panda Breath web interface.
- **Listed as "bound to a printer not shown here":** the serial number the Panda Breath reports matches no printer in Bambuddy that you have access to.
