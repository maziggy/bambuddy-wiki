---
title: MakerWorld Integration
description: Import and print MakerWorld models directly from Bambuddy — paste a URL, pick a plate, print. No Bambu Handy app needed.
---

# MakerWorld Integration

Paste any MakerWorld model URL into Bambuddy and import or print the 3MF
directly — without leaving Bambuddy for the Bambu Handy app.

This was the single reason most users kept Handy installed after moving
their printers to LAN-only mode. With this integration, you can finally
uninstall it.

---

## :material-earth: What it does

- **Paste a URL** — any `https://makerworld.com/en/models/…` link
  (locale prefix, slug, and `#profileId-…` fragment all accepted).
- **Preview the model** — title, summary, creator, license, download
  count, and the full plate list with per-plate filament counts.
- **Import to Library** — downloads the 3MF via MakerWorld's official
  per-instance download API. The file shows up in your File Manager
  tagged with its MakerWorld source URL.
- **Print Now** — imports (or reuses an existing import) and opens
  Bambuddy's existing plate picker + AMS mapping dialog, then dispatches
  the print like any other library file.
- **"Already in library" detection** — re-pasting a URL you've already
  imported (even a different variant of the same model URL) is a no-op;
  the existing library entry is reused.

---

## :material-login: Prerequisites

- A **Bambu Cloud account** linked to Bambuddy under **Settings → Bambu
  Cloud**. The same token you use for firmware checks and slicer
  settings is reused for MakerWorld downloads — no separate login.
- The **`makerworld:view`** permission (browse) and/or
  **`makerworld:import`** permission (import/print). The default
  Administrators and Operators groups have both. Viewers get
  `makerworld:view` only.

Without a Bambu Cloud token you can still paste a URL and read the
model's public metadata, but downloads require MakerWorld authentication.

---

## :material-cursor-default-click: How to use it

1. Open **MakerWorld** from the sidebar (between *File Manager* and
   *Notifications*).
2. Paste a MakerWorld URL into the input field and click **Resolve**.
3. Review the model details — plates list with filament count, AMS
   requirement, and individual plate download counts.
4. For each plate, you can:
   - Click **Import** to save the 3MF to your library.
   - Click **Print Now** to import + open the standard print dialog.

---

## :material-alert-circle-outline: Limitations

- **Browse / search is not available.** MakerWorld's public search
  endpoint returns empty results for server-originated requests
  (requires browser-specific session state we can't reproduce from a
  backend). Discovery happens outside Bambuddy — Reddit, YouTube,
  shared links — and the URL-paste flow catches you at the moment you
  actually want to print.
- **Anti-bot protection on HTML pages.** If MakerWorld changes their
  internal API endpoints, the fallback `__NEXT_DATA__` parse-from-HTML
  strategy used by some reverse-engineering projects won't work because
  Cloudflare blocks those requests. Bambuddy will surface a clear error
  and you can open the model in a browser and download manually.
- **Download URLs are short-lived.** MakerWorld returns a signed CDN
  URL valid for ~5 minutes. Bambuddy fetches the 3MF immediately and
  stores it locally; the signed URL is never cached.

---

## :material-shield-check: Privacy & compliance

- Bambuddy is **not affiliated with or endorsed by** MakerWorld or Bambu
  Lab.
- The integration only uses community-documented endpoints
  (`/api/v1/design-service/*`) that Bambu Studio itself already uses.
- No credentials are scraped or forwarded — the existing Bambu Cloud
  token stored in your Bambuddy account is the only thing sent.
- Thumbnails are hot-linked from MakerWorld's CDN (no proxying).

---

## :material-file-code: Developer reference

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/v1/makerworld/status` | GET | `makerworld:view` | Report token presence |
| `/api/v1/makerworld/resolve` | POST | `makerworld:view` | Resolve URL → design + plate list |
| `/api/v1/makerworld/import` | POST | `makerworld:import` | Download a plate's 3MF into the library |

Backend code: `backend/app/services/makerworld.py`,
`backend/app/api/routes/makerworld.py`.
