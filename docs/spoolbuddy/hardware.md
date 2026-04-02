---
title: Hardware Requirements
description: What you need to build a SpoolBuddy
---

# Hardware Requirements

---

## :material-check-all: Required Components

| Component | Recommendation | Notes |
|----------|----------------|-------|
| **Raspberry Pi** | Pi 4/Pi 5 recommended | Must run Raspberry Pi OS (Debian-based). Other models may work but are less validated. |
| **Display** | LAFVIN 7\" HDMI Touch 1024x600 | Kiosk is optimized for touch interaction. This kit includes Raspberry Pi adapters (angled HDMI/micro USB style adapters). |
| **NFC reader** | PN5180 module (SPI) | Current daemon driver targets PN5180 (`SPI`, manual CS). |
| **Scale ADC** | NAU7802 module (I2C) | Uses I2C bus 1 (`/dev/i2c-1`, address `0x2A`). |
| **Load cell** | Compatible load cell for your platform | Required for live spool weight readings. |
| **Power supply** | Stable Pi PSU for your model | Undervoltage causes unstable kiosk/device behavior. |
| **Angled USB-C connector** | 90° angled USB-C connector | Plugs into the Pi's USB-C power port. The cable exits through the cutout on the back of the case. |

---

## :material-memory: NFC Tags

SpoolBuddy can interact with:

- **Bambu Lab RFID spools** (MIFARE Classic)
- **NTAG213/215/216** stickers/cards for writable custom tags

---

## :material-connection: Wiring Reference

Full pin-by-pin wiring tables and ASCII diagrams are documented on:

- [Wiring Diagrams](wiring-diagrams.md)

!!! warning "Dual voltage requirement"
    The PN5180 board requires **both** `5V` and `3.3V` connections. Do **not** feed 5V into the 3V3 pin — this will damage the module.

!!! info "Manual CS"
    SpoolBuddy uses **manual chip-select on GPIO23** (`dtoverlay=spi0-0cs`) to satisfy PN5180 timing requirements.

---

## :material-wrench: Practical Build Notes

- **Module side (PN5180, NAU7802):** Solder wires directly to the pads — easy and reliable.
- **Pi GPIO side:** Use crimped 2.54 mm connectors (HARWIN M20 or similar). Dupont connectors work but can cause intermittent errors — verify contact with a multimeter. Soldering to GPIO pins is possible but risky; a solder bridge can permanently damage the board.
- Keep NFC wiring short and clean to reduce signal issues.
- Ensure Pi user/service has access to `gpio`, `spi`, `i2c`, and `video` groups.
- If you use DSI backlight control, `video` group access is required for brightness writes.


