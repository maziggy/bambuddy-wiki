---
title: Deployment Modes
description: Choose between full local install or SpoolBuddy-only companion mode
---

# Deployment Modes

SpoolBuddy supports two recommended deployment models.

---

## :material-lan-connect: Mode A - Companion Only (Preferred for Lower-RAM Pi)

Run only SpoolBuddy on the Pi connected to the display; connect to remote Bambuddy server.

### When to choose this

- You already run Bambuddy on a NAS/server/mini-PC.
- You want kiosk hardware separated from core server.
- Your Pi is 2GB class and should not run full stack.

### Install command

```bash
sudo ./install.sh --mode spoolbuddy --bambuddy-url http://192.168.1.100:8000 --api-key bb_xxx --yes
```

### Notes

- Pi only runs kiosk + daemon + hardware interfaces.
- Remote Bambuddy handles API/database/web tasks.

---

## :material-server: Mode B - Full Local (Bambuddy + SpoolBuddy on the Same Pi)

Run both services on one Raspberry Pi:

- Bambuddy backend/frontend
- SpoolBuddy daemon
- local kiosk display session

### When to choose this

- You want an all-in-one box.
- Your Pi has enough resources for both workloads (4GB+ recommended).
- You prefer simpler network topology.

### Install command

```bash
sudo ./install.sh --mode full --port 8000 --yes
```

### Notes

- After first boot, create an API key in **Bambuddy -> Settings -> API Keys**, then update `/opt/bambuddy/spoolbuddy/.env` with `SPOOLBUDDY_API_KEY=bb_xxx` and restart the service (`sudo systemctl restart spoolbuddy`).
- Good for single-printer or compact setups.

---

## :material-lightbulb: Recommendation

For a full compatibility matrix and per-model guidance, see [Supported Pis](supported-pis.md).

In short: 2GB Pi -> Companion Only. 4GB+ Pi -> either mode depending on your preference for all-in-one vs. split architecture.
