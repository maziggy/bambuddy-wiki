---
title: AI Failure Detection
description: Detect print failures with Obico or OctoEverywhere AI failure detection
---

<span id="ai-failure-detection"></span>

Bambuddy can check camera snapshots while a print is running and notify you, pause the print, or pause and cut power when a possible failure is detected.

Choose a **Provider** in **Settings → Failure Detection**:

| Provider | Where analysis runs | What you need |
|----------|---------------------|---------------|
| [Obico](#obico) | Your own ML API container on your local network | An Obico ML API URL and a reachable Bambuddy External URL |
| [OctoEverywhere](#octoeverywhere) | OctoEverywhere's private cloud | A free Gadget API key and outbound internet access |

---

## OctoEverywhere

[OctoEverywhere](https://octoeverywhere.com/) provides cloud AI failure detection through its Gadget API. Bambuddy captures snapshots from the printer's configured built-in or external camera and uploads them over HTTPS. Gadget follows each print over time and returns print quality and warning/pause suggestions; Bambuddy handles notifications and printer actions.

### Privacy

OctoEverywhere has a strict privacy policy for the Gadget AI APIs that states all images are deleted immediately after the processing request is complete. OctoEverywhere does not store images or use them to train its AI models. [Read the full Privacy Policy here.](https://octoeverywhere.com/privacy?source=footer#gadget-developer-api)

### Setup

1. Open the [Gadget API account page](https://octoeverywhere.com/gadgetapi), sign in or create an account, and copy your API key.
2. Go to **Settings → Failure Detection** and select **OctoEverywhere** under **Provider**.
3. Turn on **OctoEverywhere AI Detection**, paste the key into **Gadget API key**, and click **Test**.
4. Confirm the key verification succeeds. Connected printers are monitored while actively printing. Clear **Monitor all connected printers** to choose individual printers.
5. In **Settings → Notifications**, enable **AI Failure Detection** on each notification provider that should receive alerts. Check that its printer filters include the monitored printers, and use the provider's test to verify delivery. See [Notifications → Printer Events](notifications.md#printer-events).

### Detection Settings

| Setting | Behavior |
|---------|----------|
| **Confidence** | **Lowest**, **Low**, **Medium** (default), **High**, or **Highest**. Lower confidence reports possible failures sooner, with more potential false positives. Higher confidence requires more certainty before warning or pausing. |
| **Inspection interval** | **20 seconds** by default. Choose an interval from **5 to 30 seconds** with the slider. |
| **Action on detected failure** | **Notify only** (default), **Pause print**, or **Pause and cut power**. Pause actions run when Gadget suggests pausing; cutting power also turns off enabled smart plugs linked to the printer. |
| **Monitored Printers** | **Monitor all connected printers** is selected by default. Clear it to select a subset; selecting no printers monitors none. |

A warning sends at most one notification per monitored print session. If Gadget later suggests pausing, **Pause print** or **Pause and cut power** runs once and sends a separate notification. With **Notify only**, the printer continues printing and no second notification is sent after the warning.

### Inspection Timing

The selected interval is subject to the server's minimum interval. When Gadget suggests faster inspections, Bambuddy temporarily uses its recommended interval if it is shorter than your selection, while still respecting the minimum. Camera capture, processing time, rate limits, and error retry delays can make checks take longer than the selected interval. The selected interval never overrides the server's minimum or retry timing.

### Status and Print Quality

The **Status** and **Recent Detections** cards show monitored prints and their results. The **Monitoring** field distinguishes waiting for results, successful checks, and problems that need attention. The printer card's AI badge opens the detection details.

| Badge | Meaning |
|-------|---------|
| **Starting** | Waiting for the first usable result. |
| **Safe** | The latest successful check did not suggest a warning or pause. |
| **Warning** | Gadget suggested warning about a possible issue. |
| **Failure** | Gadget suggested pausing the print. |
| **Not checking** | A camera, connection, or API problem prevented a result. Open the details for the reason. |

**Print quality** ranges from **1/10 to 10/10**, with higher values indicating better quality. It is not a failure probability. Warnings and actions follow Gadget's suggestions, not the quality score alone.

### Troubleshooting OctoEverywhere

**Key rejected or account unavailable**
: Check the saved Gadget API key and the status on the [Gadget API account page](https://octoeverywhere.com/gadgetapi). For a disabled key, [contact OctoEverywhere support](https://octoeverywhere.com/support). For an IP restriction, use the original key associated with that public IP or contact support if it is unavailable. After resolving access, click **Test** to resume inspections.

**Usage limit reached**
: When the API returns `OE_FREE_USAGE_LIMIT_REACHED`, Bambuddy shows **Usage limit reached.** with a **Set up billing to continue** link. Wait for the next monthly allowance, or optionally set up billing and turn off **Free Usage Only** on the [Gadget API account page](https://octoeverywhere.com/gadgetapi). If billing is already configured, only **Free Usage Only** needs changing. After the allowance renews or the account settings are updated, click **Test** to resume inspections.

**IP restriction remains after changing account settings**
: Changing billing settings or creating another key does not remove an IP restriction. Use the original Gadget API key associated with that public IP, or [contact OctoEverywhere support](https://octoeverywhere.com/support) if the key is unavailable or the IP is shared with another account. Click **Test** after resolving access.

**Camera or connection failure**
: Confirm the printer is connected, actively printing, and selected for monitoring. Check that its configured camera works in Bambuddy and that the Bambuddy host has DNS and outbound HTTPS access to OctoEverywhere. Temporary connection and service failures retry automatically; repeated failures delay subsequent retries.

**Checks slower than the selected interval**
: Camera capture, processing time, the server's minimum interval, rate limits, and error retry delays can lengthen the interval. See [Inspection Timing](#inspection-timing).

**False alarms or late alerts**
: Raise **Confidence** if healthy prints trigger warnings; lower it to report possible failures sooner. This is the opposite direction to Obico's **Sensitivity** control.

**Detection appears but no notification arrives**
: Enable **AI Failure Detection** on a working notification provider and check its printer filters. The settings page reports missing notification coverage; the provider's own test checks delivery.

!!! warning "Account errors stop monitoring"
    Key, account, IP restriction, and usage-limit errors stop inspections across all monitored printers until access is restored and **Test** succeeds with the configured key, or a replacement key is saved. These errors do not pause the printers.

**Test** verifies access by creating a context; it does not upload an image or verify the remaining inspection allowance. A successful test resumes inspections, but the next inspection can stop monitoring again if the allowance is still exhausted. Creating another context or switching processing URLs does not reset the allowance.

A successful **Test** with the same key preserves existing monitoring contexts. Saving a replacement key creates new contexts while preserving notification/action tracking and the current inspection timing.

#### Inspection Debug Logs

Each inspection logs its start and whether it reuses a context. Successful results include the printer ID, frame count, verdict, print quality, warning/pause suggestions, faster-inspection flag, server minimum, and effective interval until the next check. Failed attempts log a safe error message, recognized error code, and retry delay when another attempt is scheduled; canceled or discarded inspections are also recorded. Scheduled polls that are not yet due do not produce inspection logs. These diagnostics omit API keys, context IDs/URLs, image data, and raw API response bodies.

### API Reference

See [OctoEverywhere API Reference](../reference/api.md#octoeverywhere-ai-failure-detection) for settings keys, defaults, status and test endpoints, required permissions, and notification coverage fields.

---

## Obico

The [Obico](https://github.com/TheSpaghettiDetective/obico-server) integration uses a **self-hosted** `ml_api` container. No Obico account is required. Snapshots are analyzed on your own network; this provider does not connect to obico.io.

### How It Works

1. While a print is running, Bambuddy hands the ML API your camera snapshot URL every *N* seconds (default 10s).
2. The ML API fetches the snapshot itself and returns a list of detected defects with confidence scores.
3. Bambuddy smooths scores over time (Obico's own EWM + rolling-mean math) so a single noisy frame cannot trigger an action.
4. When the smoothed score crosses the **High** threshold, Bambuddy runs your configured action **once** for this print.

The smoothing uses a 30-frame warmup, an exponentially weighted moving average with `alpha = 2/13`, a short rolling mean (~5 min at 10 s/frame), and a long rolling-mean baseline (~20 h). This is the same approach Obico's own detector uses.

---

### Setting Up the Obico ML API

You only need the `ml_api` container from Obico's stack. The web app, Django site, and printer registration are **not required**.

#### 1. Clone the Obico server

```bash
git clone -b release https://github.com/TheSpaghettiDetective/obico-server.git
cd obico-server
```

#### 2. Expose port 3333 (ml_api)

Edit `docker-compose.yml` and add a `ports` mapping on the `ml_api` service:

```yaml
ml_api:
  ports:
    - "3333:3333"
```

#### 3. Start the stack

```bash
docker compose up -d ml_api
```

The first start downloads the YOLO model (~100 MB) and allocates ~4 GB RAM.

#### 4. Verify

```bash
curl http://<obico-host>:3333/hc/
# → "ok"
```

#### 5. Optional: protect it with a token

The ml_api container reads an `ML_API_TOKEN` environment variable. Set it and the container answers `401` to any detection request that doesn't carry that token, which keeps anything else on your network from using your inference server:

```yaml
ml_api:
  environment:
    - ML_API_TOKEN=<a long random string>
```

Put the same value in Bambuddy's **ML API Token** field below. Leave both unset if you don't need it.

!!! warning "The health endpoint is not protected"
    `/hc/` answers `ok` whether or not the token is right — only the detection endpoint is gated. So a `curl` of `/hc/` proves nothing about your token. Bambuddy's **Test** button probes both, and tells you specifically when the token is rejected.

---

### Configuring Bambuddy

Go to **Settings → Failure Detection** and select **Obico** under **Provider**.

#### Required

- **Enable toggle** — turns the detection service on.
- **Obico ML API URL** — base URL to your ML API, e.g. `http://192.168.1.10:3333`. Click **Test** to check reachability and the token.
- **ML API Token** — only needed if your ml_api container runs with `ML_API_TOKEN` set. It must match that value exactly. Leave empty otherwise.
- **Bambuddy address** — set **Bambuddy Internal URL** below, or **External URL** in **Settings → Network**. The chosen address must be reachable from the ML API container.

### Optional: Bambuddy Internal URL

**Bambuddy Internal URL** lets the ML API fetch snapshots through a different Bambuddy address. Leave it empty to use **External URL**. Other integrations continue to use External URL.

For example, when both containers share a Docker network:

| Setting | Example | Connection |
|---|---|---|
| **Obico ML API URL** | `http://obico-ml:3333` | Bambuddy calls the ML API. |
| **Bambuddy Internal URL** | `http://bambuddy:8000` | The ML API fetches snapshots from Bambuddy. |
| **External URL** (Settings → Network) | `https://bambuddy.example.com` | Other integrations use the public address. |

Use your actual Docker service names and Bambuddy's container port. You can also use a LAN address, such as `http://192.168.1.20:8000`. Include `http://` or `https://` and any required port. `localhost` inside the ML container points to that container, not Bambuddy.

The field saves automatically. An address without `http://` or `https://` is marked in red and not saved until it has one. Changes take effect on the next detection cycle without restarting Bambuddy. Clear it to return to External URL.

The **Test** button checks the ML API and token. It does not check whether the ML API can fetch a snapshot; check detection during a print to confirm that connection.

#### Tuning

- **Sensitivity** — Low / Medium / High. Scales the confidence thresholds:
    - **Low** is less eager to trigger (fewer false positives, may miss early failures)
    - **Medium** is the default (Obico's original thresholds)
    - **High** triggers earlier but with more false positives
- **Poll interval** — how often each active print is checked (5–120 seconds). 10 s is the Obico default.
- **Action on detected failure** — what Bambuddy should do when the score crosses the high threshold:
    - **Notify only** — fires the **AI Failure Detection** notification event through your existing notification providers
    - **Pause print** — notify + send an MQTT pause command
    - **Pause and cut power** — pause + turn off any smart plug linked to that printer
- **Monitored printers** — by default all connected printers are watched. Uncheck "Monitor all connected printers" to pick a specific subset.

!!! note "Enabling notifications for detected failures"
    Detections fire the dedicated **AI Failure Detection** event, not the general "Printer Error" event. Edit each notification provider you want to receive spaghetti alerts on (Telegram, Discord, ntfy, etc.) and turn on the **AI Failure Detection** toggle in the Printer Status section. See [Notifications → Printer Events](notifications.md#printer-events). Existing providers that had "Printer Error" enabled will continue to receive HMS hardware errors unchanged; they will not automatically receive AI alerts until the new toggle is enabled.

#### Status card

The right column shows:

- Whether the background service is running
- The currently active thresholds (after sensitivity scaling)
- Each active print's live classification (*safe / warning / failure*), smoothed score, and frame count
- Recent detection history (timestamp, printer, class, score)

#### Printer card badge

When failure detection is enabled, every monitored printer's card on the **Printers** page shows an AI badge next to the HMS indicator, so you can watch detection track your print without leaving the Printers screen:

| Badge | Meaning |
|---|---|
| gray **Idle** | No print is being watched on this printer. |
| gray **Starting** | A monitored print has begun; waiting for the first result. |
| green **Safe** | An inference came back and found nothing. |
| amber **Warning** | The smoothed score is between the Low and High thresholds. |
| red **Failure** | The smoothed score has crossed the High threshold. |
| amber **Not checking** | The last check produced no result at all — a rejected token, an unreachable ML API, a snapshot that could not be captured, or a missing snapshot address. **This print is not being watched.** Hover, or click, for the reason. Detection resumes on its own once the cause is fixed. |

Hover for the current smoothed score; click to open a modal with the live status, score, frames analyzed, and the reason for a **Not checking** badge, plus a shortcut to **Settings → Failure Detection** for the full history. Score and frame count are omitted while the badge reads **Not checking** — there is no measurement behind them. Printers excluded from monitoring (and setups with detection disabled) show no badge.

---

### Requirements & Gotchas

- **The ML API container must be able to reach the configured Bambuddy address.** Use a Docker service name on a shared network or a reachable LAN address.
- **Set Bambuddy Internal URL or External URL.** Without either, Bambuddy cannot tell the ML API where to fetch snapshots.
- **Snapshots must be accessible to the ML API without additional proxy authentication.** The cached-image endpoint uses short-lived, single-use URLs and does not require a Bambuddy login. Bambuddy Internal URL can avoid an authenticated public reverse proxy; the snapshots do not need to be exposed to the internet.
- **The printer camera must be enabled and reachable** — same requirement as Bambuddy's own camera page.
- **The action fires exactly once per print.** After a detected failure, subsequent frames won't re-trigger until a new print starts.
- **Calibration prints are automatically skipped** by Bambuddy's detection loop — the service only runs while a print is in the `RUNNING` state.
- **Disk / RAM** — the ML model needs roughly 4 GB RAM on the Obico host. CPU use scales with how many printers you monitor and how short the poll interval is.

---

### Troubleshooting

**"No address set for the ML API to fetch snapshots from"**
: Set **Bambuddy Internal URL** in **Settings → Failure Detection**, or **External URL** in **Settings → Network**, to an address the ML API container can reach.

**The Test button succeeds, but detection reports a snapshot error**
: The ML API may be reachable from Bambuddy while it cannot fetch images in the other direction. Check that the snapshot address resolves and is reachable from the ML API container. If the public address requires proxy authentication, set Bambuddy Internal URL to a reachable address.

**Test button returns an error**
: Check the ML API is running (`docker compose ps ml_api`) and that port 3333 is exposed. Try `curl` from the Bambuddy host: `curl http://<obico-host>:3333/hc/`.

**"The ML API is reachable but rejected the token"**
: The container runs with `ML_API_TOKEN` set and Bambuddy's **ML API Token** doesn't match it. Copy the value from your `docker-compose.yml` (or `docker compose exec ml_api env | grep ML_API_TOKEN`) into the field, or remove `ML_API_TOKEN` from the container and clear the field.

**Detection never runs, but the Test button says everything is fine**
: On versions before 1.2.6 this was the signature of a token-protected ML API: the test only pinged the ungated `/hc/` endpoint while every detection call was rejected with `401`. Set the **ML API Token**, or update — the test now checks the token too, and the status card names the problem outright.

**Service is running but no detections appear**
: No news is good news — entries are only written to the history when a detection is returned or the classification leaves "safe". Check the printer card badge: green **Safe** with a climbing frame count means checks are landing and finding nothing. Amber **Not checking** means they are not landing, and names the cause.

**The ML API container's log shows no requests from Bambuddy**
: A request rejected for a bad token is turned away by the ML API's auth layer before its request log ever sees it, so a token problem looks exactly like "Bambuddy never called". The same is true of successful checks, which log nothing at all. Trust the printer card badge over the container log: **Not checking** names the real cause, and a climbing frame count means the calls are getting through. Bambuddy's own log records every rejected call.

**False positives on normal prints**
: Lower the Sensitivity setting (High → Medium → Low).

**Actions didn't fire on an obvious failure**
: Raise the Sensitivity. Remember there's a 30-frame warmup at the start of each print (5 min at 10 s/frame), during which the service is deliberately quiet.

**Detection fires in the History card but no Telegram / Discord / ntfy notification arrives**
: The notification is dispatched on the dedicated **AI Failure Detection** event, not the general "Printer Error" event. Open **Settings → Notifications**, edit each provider you want spaghetti alerts on, and turn on the **AI Failure Detection** toggle in the Printer Status section. Providers that only had "Printer Error" enabled won't receive AI alerts.

---

### License & Attribution

Obico's ML model and detection algorithms are licensed under AGPL-3.0, the same license as Bambuddy. Bambuddy does **not** vendor or link any Obico code — it only calls the ML API over HTTP.
