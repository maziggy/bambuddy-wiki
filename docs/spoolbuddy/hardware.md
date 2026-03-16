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
| **Scale ADC** | NAU7802 module (I2C) | Uses I2C bus 0 (`/dev/i2c-0`, address `0x2A`). |
| **Load cell** | Compatible load cell for your platform | Required for live spool weight readings. |
| **Power supply** | Stable Pi PSU for your model | Undervoltage causes unstable kiosk/device behavior. |

---

## :material-memory: NFC Tags

SpoolBuddy can interact with:

- **Bambu Lab RFID spools** (MIFARE Classic)
- **NTAG213/215/216** stickers/cards for writable custom tags

---

## :material-connection: Wiring Reference

Full pin-by-pin wiring tables and ASCII diagrams are documented on:

- [Wiring Diagrams](wiring-diagrams.md)

!!! warning "PN5180 power pins"
    The PN5180 board uses both `3V3` and `5V` pins for proper operation.  
    Do **not** feed 5V into 3V3.

!!! info "Manual CS"
    SpoolBuddy uses **manual chip-select on GPIO23** (`dtoverlay=spi0-0cs`) to satisfy PN5180 timing requirements.

!!! info "SparkFun Qwiic scale AVDD bridge"
    SparkFun NAU7802 boards require a `3V3` to `AVDD` bridge.
    See [Wiring Diagrams](wiring-diagrams.md) for the full instruction.

---

## :material-wrench: Practical Build Notes

- Soldering is recommended for best signal quality and long-term stability.
- Crimped/plugged connections can work, but should be treated as test/experimental until validated in your build.
- Keep NFC wiring short and clean to reduce signal issues.
- Ensure Pi user/service has access to `gpio`, `spi`, `i2c`, and `video` groups.
- If you use DSI backlight control, `video` group access is required for brightness writes.

---

## :material-flask-outline: Hardware Validation

After wiring, these checks should pass:

```bash
# SPI devices present
ls /dev/spidev0.*

# I2C bus 0 present
ls /dev/i2c-0

# NAU7802 visible on bus 0 (0x2A)
sudo i2cdetect -y 0
```

Then run SpoolBuddy diagnostics from your install path:

```bash
sudo /opt/bambuddy/spoolbuddy/venv/bin/python /opt/bambuddy/spoolbuddy/scripts/pn5180_diag.py
sudo /opt/bambuddy/spoolbuddy/venv/bin/python /opt/bambuddy/spoolbuddy/scripts/scale_diag.py
```
