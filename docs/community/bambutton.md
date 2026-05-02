---
title: Bambutton
description: ESP32-C3 wireless plate-clear button for Bambuddy
---

# Bambutton

A physical plate-clear button for Bambuddy. Bambutton turns a small ESP32-C3 board into a dedicated wireless control for your printer: the LED ring signals when a plate needs clearing, and a single press marks the plate as clear in Bambuddy so the next queued job can be dispatched automatically.

It gives each printer a simple shop-floor control that's quick to see and quick to press &mdash; no need to open the Bambuddy interface every time a print finishes.

**Author:** [Edward Chamberlain](https://github.com/EdwardChamberlain)
**Repository:** [github.com/EdwardChamberlain/bambutton](https://github.com/EdwardChamberlain/bambutton)
**Status:** Independent community project

---

## What you need

- An ESP32-C3 board (the README bundles a tested MicroPython firmware image for the `ESP32_GENERIC_C3` profile).
- An LED and a momentary push button.
- A data-capable USB cable for flashing.
- A Bambuddy [API key](../features/api-keys.md) and the printer ID you want the button to control.

---

## Setup

Two supported routes &mdash; pick whichever fits your comfort level.

### Setup Assistant GUI (recommended)

The packaged setup assistant walks you through everything: selecting the firmware `.bin`, picking the connected board, entering Bambuddy connection details, fetching printers from your Bambuddy instance, choosing one by friendly name, setting the LED and button pins plus Wi-Fi credentials, and finally flashing the firmware and pushing settings.

Release builds for Windows and macOS are published on the [bambutton releases page](https://github.com/EdwardChamberlain/bambutton/releases) so end users don't need Python, `mpremote`, or `esptool` installed locally.

### Manual flashing (advanced)

For development or debugging, the project exposes the underlying Python scripts and a `micro/config.json` file that can be edited directly before pushing to the board with `mpremote`. See the [project README](https://github.com/EdwardChamberlain/bambutton#manual-flashing) for the full command sequence.

---

## How it talks to Bambuddy

Bambutton uses Bambuddy's documented [REST API](../reference/api.md) authenticated via an [API key](../features/api-keys.md). The board polls the configured printer's state on a configurable interval, drives the LED to indicate when the plate needs clearing, and sends a "plate cleared" request when the button is pressed. No internal Bambuddy endpoints are used &mdash; everything goes through public API routes, so the integration stays stable across Bambuddy releases.

---

## Support

Issues, feature requests, and questions about the bambutton hardware and firmware belong on the [project's GitHub repository](https://github.com/EdwardChamberlain/bambutton/issues). The Bambuddy team doesn't maintain bambutton and can't ship fixes for it.

For Bambuddy-side questions (API key setup, finding the printer ID, etc.), the [Discord](https://discord.gg/aFS3ZfScHM) and [forum](https://forum.bambuddy.cool) are the right places to ask.
