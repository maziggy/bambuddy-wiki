---
title: Build Plate Check
description: Check that the build plate is empty before a print starts — with OpenCV or an AI vision backend
---

# Build Plate Check

Before a print starts, Bambuddy can check the camera to confirm the build plate is actually empty, and pause the job if it isn't. Two detection backends are available:

- **OpenCV** (default) — compares a live camera frame against reference photos of the empty plate, captured during calibration, and flags anything that differs beyond a threshold.
- **AI** (opt-in) — sends one snapshot to a vision-language model over an OpenAI-compatible API and uses the model's own judgment of whether the plate is empty. No calibration, no reference photos.

The AI backend is an *alternative*, not a replacement — OpenCV remains the default, and nothing changes for existing installs until you switch.

!!! info "When the check runs"
    Once at print start (only for printers with the check enabled), when you run a manual check from the printer card, and when you press **Test connection** in Settings. Never during a print and never on a timer — during-print monitoring is [AI Failure Detection](failure-detection.md), a separate feature.

## Setup

Settings → **Build Plate Check** tab:

| Field | What to put there |
|---|---|
| Detection backend | `OpenCV` (default) or `AI` — the global choice |
| AI backend URL | An OpenAI-compatible `/v1` endpoint — a local Ollama server, or a hosted API |
| Model name | The model to request, e.g. `qwen2.5vl:7b` for local Ollama |
| API key (optional) | Leave empty for a local server that doesn't require one |

Use **Test connection** to check whether the configured endpoint and model return a valid verdict for a synthetic gray frame. It does not send a real printer photo, and a successful test does not guarantee real-camera results or prove hosted-provider compatibility.

The same tab also shows:

- **Monitored printers** — which printers run the check before a print (the per-printer enable toggle, same one as on the printer card), and which backend each printer uses: the global default, or a per-printer override.
- **Status** — the effective configuration at a glance: global backend, endpoint and model, and the resolved backend for every printer.

## Per-printer backend

Mixed fleet? Each printer can override the global backend choice — from the **Monitored printers** list, or from the printer card's plate-check dialog (**Backend for this printer**: Global default / OpenCV / AI). Printers without an override follow the global setting.

## The decision panel

After a manual check, the plate-check dialog shows a **decision panel**: which backend produced the verdict, the verdict itself, its confidence, and the evidence — the pixel-difference percentage for OpenCV, or the model's stated reasoning for AI. AI confidence is the model's estimate of confidence in its empty/not-empty verdict; it is not a pixel-difference score. When the AI backend is active, the calibration references and region-of-interest editor shown below it apply to the OpenCV backend only; the AI backend evaluates the full camera frame.

## Worked example — local Ollama (privacy-preserving)

- **AI backend URL**: `http://<your-ollama-host>:11434/v1`
- **Model name**: `qwen2.5vl:7b`
- **API key**: leave empty

This keeps every snapshot on your own network — nothing leaves your LAN. Pull the model first (`ollama pull qwen2.5vl:7b`) and allow time for it to load on first use; response times vary by hardware and model.

## Hosted OpenAI compatibility (untested)

The AI backend sends requests using the OpenAI `chat/completions` format, so a hosted OpenAI endpoint may be configurable in principle. **Hosted OpenAI has not been tested against a live service**; no API key was available for verification. Automated tests use mocked HTTP responses to check request construction and response handling. Those tests do not establish that OpenAI, or another hosted provider, works end to end. The only live service exercised so far is local Ollama.

For experimentation, the endpoint URL would be `https://api.openai.com/v1`, with a vision-capable model name and an API key accepted by that service. Treat this as an unverified configuration; hosted compatibility is theoretical until tested against a live provider. Other services that describe themselves as OpenAI-compatible may differ in accepted request options.

**Test connection** uses a synthetic frame to check the endpoint and model. It starts with `max_tokens` and strict `json_schema` output, then may probe alternatives if the endpoint rejects that shape: `max_completion_tokens` instead of `max_tokens`, and `json_object` instead of `json_schema` (up to three requests). A successful `json_object` fallback is usable but marked **degraded** in AI health status because its output is less constrained. The discovered request shape is cached only in memory for the running Bambuddy process. After a restart, run **Test connection** again if the endpoint needs a fallback shape; otherwise requests use the default shape. A successful probe only confirms the tested request was accepted; it does not verify live hosted-provider behavior.

## Privacy

**Snapshots only leave your local network if you configure a non-local endpoint, and only because you explicitly typed that endpoint in.** With a local Ollama URL (loopback or an RFC-1918 address), every snapshot stays on your LAN. If you configure a public URL, Bambuddy warns you in Settings before you save — but it will not stop you. There is no default hosted endpoint, and no snapshot is sent anywhere until you configure a URL and select `AI` as a backend.

## Two things worth stating plainly

1. **The per-printer "Build Plate Check" toggle still controls whether the check runs at all.** The backend choice (global, or per-printer override) selects *which* engine runs the check — it doesn't turn the check on or off. Enable the check for each printer you want covered, then choose the backend.
2. **The AI backend needs no calibration.** No reference photos, no region-of-interest, no "not yet calibrated" state — but the configured endpoint and model still need to be reachable and usable. Any calibration references and ROI you've set up are used by the OpenCV backend only.

## Which printers benefit

Bambu's H2-series (H2/H2D/H2C), X2D, and P2S already run Bambu's own native foreign-object detection — this feature doesn't add anything for those models. It's most useful on printers **without** native detection: **X1, X1C, P1P, P1S, and A1**.

## Fail-open behavior

The AI backend fails **open** on errors: a check that cannot produce a verdict is **unavailable**, not a finding that the plate is empty. At print start, an unavailable result allows printing to proceed, but the UI shows a distinct no-verdict state rather than a green empty verdict. The Settings status also exposes the latest per-printer health outcome. When a printer transitions to unavailable, Bambuddy sends a printer-error notification if configured, rate-limited per printer. Successful checks report the model's verdict; a successful check using `json_object` fallback is marked degraded. This differs from OpenCV, which reports a pixel-difference measurement.

| Situation | What you'll see | What happens |
|---|---|---|
| Base URL or model name left empty | "AI backend not configured" | Fails open, no network call attempted |
| Request times out (e.g. cold local model) | "request timed out" | Fails open |
| Can't reach the endpoint | "connection failed" | Fails open |
| Endpoint returns an HTTP error (401, 500, …) | "AI backend returned an error" | Fails open |
| Response isn't valid JSON or lacks a valid verdict | Manual check: one parse/schema retry, then unavailable. Print-start check: no parse retry. | Fails open; unavailable is distinct from an empty verdict |
| Camera frame is corrupt/truncated | "camera frame could not be processed" | Fails open as unavailable |
| Database error reading settings | "AI backend unavailable" | Fails open as unavailable |

Messages are short and generic on purpose — they never include your configured server address, even in an error. Full detail goes to the server log.

## Request time limits

Manual checks and **Test connection** use a 60-second default per-request timeout. Test connection may issue up to two fallback probes; the first request can use that default, while subsequent discovery retries have a shorter per-request cap. Print-start checks use a separate 20-second aggregate budget for AI analysis, including frame processing and the request, and do not spend extra time on a parse/schema retry. The print-start AI analysis runs asynchronously after printer selection and the light-settle stage, so a slow backend does not hold up that path beyond its budget. Actual response time depends on the configured endpoint and model; no general latency is guaranteed.

## Troubleshooting

- **"AI backend not configured"** — base URL or model name is empty. Fill both in.
- **"connection failed"** — the URL is wrong, or the server isn't reachable from where Bambuddy runs (firewall, VLAN isolation).
- **"AI backend returned an error" (401)** — API key missing or wrong for an endpoint that requires one.
- **"AI backend returned an error" (404 or similar)** — the model name doesn't exist on that endpoint; check spelling and that the model is pulled/deployed.
- **Test button times out** — check that the endpoint is reachable and the model is available, then retry; response time depends on the endpoint and model.
