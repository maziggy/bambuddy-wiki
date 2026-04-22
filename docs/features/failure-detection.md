---
title: AI Failure Detection
description: Detect print failures automatically using a self-hosted Obico ML API
---

# AI Failure Detection

Bambuddy can periodically check your prints against a **self-hosted** [Obico](https://github.com/TheSpaghettiDetective/obico-server) `ml_api` container and automatically act when a failure is detected (spaghetti, layer shift, loose debris, etc.). No Obico account, no cloud calls, no WebSocket — Bambuddy just hands the ML API the printer's snapshot URL and reads back a score.

!!! info "Self-hosted only"
    This integration talks to your own Obico ML API over HTTP on your local network. Bambuddy does not connect to obico.io or any third-party service.

---

## How It Works

1. While a print is running, Bambuddy hands the ML API your camera snapshot URL every *N* seconds (default 10s).
2. The ML API fetches the snapshot itself and returns a list of detected defects with confidence scores.
3. Bambuddy smooths scores over time (Obico's own EWM + rolling-mean math) so a single noisy frame cannot trigger an action.
4. When the smoothed score crosses the **High** threshold, Bambuddy runs your configured action **once** for this print.

The smoothing uses a 30-frame warmup, an exponentially weighted moving average with `alpha = 2/13`, a short rolling mean (~5 min at 10 s/frame), and a long rolling-mean baseline (~20 h). This is the same approach Obico's own detector uses.

---

## Setting Up the Obico ML API

You only need the `ml_api` container from Obico's stack. The web app, Django site, and printer registration are **not required**.

### 1. Clone the Obico server

```bash
git clone -b release https://github.com/TheSpaghettiDetective/obico-server.git
cd obico-server
```

### 2. Expose port 3333 (ml_api)

Edit `docker-compose.yml` and add a `ports` mapping on the `ml_api` service:

```yaml
ml_api:
  ports:
    - "3333:3333"
```

### 3. Start the stack

```bash
docker compose up -d ml_api
```

The first start downloads the YOLO model (~100 MB) and allocates ~4 GB RAM.

### 4. Verify

```bash
curl http://<obico-host>:3333/hc/
# → "ok"
```

---

## Configuring Bambuddy

Go to **Settings → Failure Detection**.

### Required

- **Enable toggle** — turns the detection service on.
- **Obico ML API URL** — base URL to your ML API, e.g. `http://192.168.1.10:3333`. Click **Test** to ping `/hc/`.
- **External URL** (set in **Settings → Network**) — the URL the ML API will use to fetch Bambuddy snapshots. This must be reachable from the ML API container, *not* from your browser. Usually the IP of your Bambuddy host.

### Tuning

- **Sensitivity** — Low / Medium / High. Scales the confidence thresholds:
    - **Low** is less eager to trigger (fewer false positives, may miss early failures)
    - **Medium** is the default (Obico's original thresholds)
    - **High** triggers earlier but with more false positives
- **Poll interval** — how often each active print is checked (5–120 seconds). 10 s is the Obico default.
- **Action on detected failure** — what Bambuddy should do when the score crosses the high threshold:
    - **Notify only** — send a printer-error notification through your existing notification providers
    - **Pause print** — notify + send an MQTT pause command
    - **Pause and cut power** — pause + turn off any smart plug linked to that printer
- **Monitored printers** — by default all connected printers are watched. Uncheck "Monitor all connected printers" to pick a specific subset.

### Status card

The right column shows:

- Whether the background service is running
- The currently active thresholds (after sensitivity scaling)
- Each active print's live classification (*safe / warning / failure*), smoothed score, and frame count
- Recent detection history (timestamp, printer, class, score)

---

## Requirements & Gotchas

- **The ML API container must be able to reach your Bambuddy host.** It fetches snapshots by URL. If they're on the same Docker network or LAN, use Bambuddy's LAN IP. `localhost` only works if both run on the same host.
- **The External URL setting must be set.** Without it, Bambuddy can't tell the ML API where to fetch snapshots.
- **The image URL must be publicly accessible.** The image URL at `/api/v1/obico/cached-frame/{image_id}` must be publicly accessible without authentication if using a reverse proxy in front of Bambuddy with authentication. The URL is already publicly accessible if using Bambuddy default authentcation.
- **The printer camera must be enabled and reachable** — same requirement as Bambuddy's own camera page.
- **The action fires exactly once per print.** After a detected failure, subsequent frames won't re-trigger until a new print starts.
- **Calibration prints are automatically skipped** by Bambuddy's detection loop — the service only runs while a print is in the `RUNNING` state.
- **Disk / RAM** — the ML model needs roughly 4 GB RAM on the Obico host. CPU use scales with how many printers you monitor and how short the poll interval is.

---

## Troubleshooting

**"external_url not set — ML API cannot reach snapshot endpoint"**
: Go to **Settings → Network** and set the External URL to a hostname or IP the ML API container can reach.

**Test button returns an error**
: Check the ML API is running (`docker compose ps ml_api`) and that port 3333 is exposed. Try `curl` from the Bambuddy host: `curl http://<obico-host>:3333/hc/`.

**Service is running but no detections appear**
: No news is good news — entries are only written to the history when a detection is returned or the classification leaves "safe". Check the Status card to confirm the service is actively running and a print is in progress.

**False positives on normal prints**
: Lower the Sensitivity setting (High → Medium → Low).

**Actions didn't fire on an obvious failure**
: Raise the Sensitivity. Remember there's a 30-frame warmup at the start of each print (5 min at 10 s/frame), during which the service is deliberately quiet.

---

## License & Attribution

Obico's ML model and detection algorithms are licensed under AGPL-3.0, the same license as Bambuddy. Bambuddy does **not** vendor or link any Obico code — it only calls the ML API over HTTP.
