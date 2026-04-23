---
title: MakerWorld Integration
description: Import and print MakerWorld models directly from Bambuddy — paste a URL, pick a plate, slice and print. No Bambu Handy app, no browser extension, no cookie paste.
---

# MakerWorld Integration

Paste any MakerWorld model URL into Bambuddy and save or slice the 3MF
directly — without leaving Bambuddy for the Bambu Handy app, a browser
extension, or a cookie-paste escape hatch.

This was the single reason most users kept Handy installed after moving
their printers to LAN-only mode. With this integration, you can finally
uninstall it.

---

## :material-earth: What it does

- **Paste a URL** — any `https://makerworld.com/en/models/…` link
  (locale prefix, slug, and `#profileId-…` fragment all accepted).
- **Preview the model** — title, summary, creator, license, download
  count, full plate list with per-plate filament counts, AMS
  requirement, and per-plate image gallery with keyboard-navigable
  lightbox.
- **Save** — downloads the 3MF and files it in a dedicated
  **MakerWorld** folder in File Manager (auto-created on first use; you
  can override per-import via the folder picker).
- **Save & Slice in Bambu Studio / OrcaSlicer** — same as Save, plus
  opens the file in your preferred slicer (Settings → Slicer
  Integration). MakerWorld plates are **unsliced source files**, so
  they can't be sent straight to the printer — the slicer is the right
  next step.
- **Import all plates** — sequentially imports every plate of a
  multi-plate model in one click. Progress is shown as "Importing
  2/5 · Downloading · 12s".
- **Recent imports sidebar** — the last 10 MakerWorld imports with
  one-click jump to File Manager, slicer, or back to MakerWorld.
- **Per-plate delete** — removes an imported plate from your library
  (file + row) via the standard Bambuddy confirm modal. The plate
  can be re-imported from MakerWorld any time.
- **Duplicate detection** — each plate is stored with a canonical
  source URL (`…/models/{id}#profileId-{pid}`), so re-importing the
  same plate is a no-op and Bambuddy surfaces "Already in library" on
  the row.

---

## :material-login: Prerequisites

- A **Bambu Cloud account** linked to Bambuddy under **Settings → Bambu
  Cloud**. The same bearer you use for firmware checks and slicer
  settings is reused for MakerWorld downloads — no separate login, no
  companion extension, no cookie paste.
- The **`makerworld:view`** permission (browse / resolve / see recent
  imports) and/or **`makerworld:import`** permission (Save / Save &
  Slice / Delete). The default Administrators and Operators groups
  have both. Viewers get `makerworld:view` only.

Without a Bambu Cloud token you can still paste a URL and read the
model's public metadata, but downloads require MakerWorld
authentication — a clear sign-in banner appears on the page when the
token is missing.

Your Bambu Cloud access tokens are valid for about 90 days. If
downloads suddenly start returning "Please log in to download models"
after months of working, sign out and back into Bambu Cloud in
Settings to refresh the token.

---

## :material-cursor-default-click: How to use it

1. Open **MakerWorld** from the sidebar (between *File Manager* and
   *Notifications*).
2. Paste a MakerWorld URL into the input field and click **Resolve**.
3. Review the model details — cover image, plate list with filament
   count, AMS requirement, per-plate images (click the cover for a
   lightbox).
4. (Optional) Change the target folder in the **Import to file
   manager** dropdown; default is the auto-created *MakerWorld* folder.
5. For each plate, choose:
   - **Save** — file goes to the library, visible in File Manager.
   - **Save & Slice in Bambu Studio / OrcaSlicer** — same, plus the
     slicer opens with the file loaded.
   - Or click **Import all** to import every plate sequentially.
6. After import, each plate row gets an inline action strip with
   **View in File Manager**, **Open in Bambu Studio**, **Open in
   OrcaSlicer**, and a trash button to delete it again.

---

## :material-alert-circle-outline: Limitations

- **Browse / search is not available.** MakerWorld's public search
  endpoint returns empty results for server-originated requests
  (requires browser-specific session state we can't reproduce from a
  backend). Discovery happens outside Bambuddy — Reddit, YouTube,
  shared links — and the URL-paste flow catches you at the moment you
  actually want to print.
- **Unsliced source files only.** MakerWorld plates are project files;
  Bambuddy can save them to your library but can't send them to a
  printer directly. That's why the per-plate action is **Save & Slice
  in your slicer**, not **Print Now** — the slicer is the correct tool
  for that step.
- **Download URLs are short-lived.** MakerWorld returns a signed CDN
  URL valid for ~5 minutes. Bambuddy fetches the 3MF immediately and
  stores it locally; the signed URL is never cached.
- **Bambu Cloud token has ~90 day lifetime.** If imports start
  returning "Please log in to download models" after a long period of
  working, re-authenticate under Settings → Bambu Cloud.

---

## :material-shield-check: Privacy, security, and compliance

- Bambuddy is **not affiliated with or endorsed by** MakerWorld or
  Bambu Lab.
- The integration only uses community-documented endpoints
  (`api.bambulab.com/v1/design-service/*` for metadata,
  `api.bambulab.com/v1/iot-service/api/user/profile/{pid}` for the
  download URL). Credit to Pr0zak/YASTL#51 for publishing the
  iot-service endpoint shape.
- Thumbnails and MakerWorld CDN images are **proxied through Bambuddy**
  (`/api/v1/makerworld/thumbnail`) so the user's IP address is never
  exposed to MakerWorld's CDN on page render. The proxy enforces a
  host allowlist and does not follow redirects.
- The MakerWorld description HTML is sanitised with **DOMPurify**
  before rendering — user-authored content can't inject scripts, event
  handlers, or javascript: URLs.
- The Bambu Cloud bearer is used **only** against `api.bambulab.com`;
  it is never forwarded to the MakerWorld CDN or to S3 presigned
  fetches.
- No credentials are scraped or forwarded beyond the existing Bambu
  Cloud token already stored in your Bambuddy account.
- Filenames from MakerWorld responses are sanitised (`os.path.basename`
  strip) before persistence so a malicious response cannot surface
  path-traversal strings into the UI. On-disk storage uses UUID
  filenames regardless.

---

## :material-file-code: Developer reference

### Endpoints

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/v1/makerworld/status` | GET | `makerworld:view` | Report Bambu Cloud token presence and regional host |
| `/api/v1/makerworld/resolve` | POST | `makerworld:view` | Resolve URL → design + plate list + already-imported profile IDs |
| `/api/v1/makerworld/import` | POST | `makerworld:import` | Download a specific plate (`profile_id`) into the library |
| `/api/v1/makerworld/recent-imports` | GET | `makerworld:view` | Last N MakerWorld library files (default 10, clamped [1, 50]) |
| `/api/v1/makerworld/thumbnail` | GET | public | Proxy the MakerWorld / public-cdn CDN for `<img>` rendering (host-allowlisted, no redirects) |

### Upstream flow

The actually-working upstream pattern (undocumented by Bambu; reverse
engineered from Pr0zak/YASTL#51):

1. `GET https://api.bambulab.com/v1/design-service/design/{designId}` —
   public metadata. Returns `{id, modelId, title, coverUrl, instances[], …}`.
   The `modelId` field is the alphanumeric identifier
   (e.g. `US2bb73b106683e5`) — **different from** the integer
   `designId` from the URL.
2. `GET https://api.bambulab.com/v1/iot-service/api/user/profile/{profileId}?model_id={modelId}`
   with `Authorization: Bearer {cloud_token}`. Returns
   `{url, name}` where `url` is a 5-minute-TTL presigned S3 URL
   (`s3.<region>.amazonaws.com/...?at=…&exp=…&key=…`).
3. Fetch the presigned URL **without following redirects** and without
   re-encoding the query string — S3 signatures are computed over the
   exact query bytes, so httpx / any normalising client breaks them
   with `SignatureDoesNotMatch`. Bambuddy uses `urllib.request` with a
   no-op `HTTPRedirectHandler` for this step.

The older `makerworld.com/api/v1/design-service/instance/{id}/f3mf`
path that some reverse-engineering projects document is cookie-gated
at Cloudflare and returns a generic "Please log in to download models"
regardless of bearer. The `api.bambulab.com` path does not go through
that gate.

### Code

- `backend/app/services/makerworld.py` — API client + download logic +
  thumbnail proxy helpers.
- `backend/app/api/routes/makerworld.py` — FastAPI routes.
- `backend/app/schemas/makerworld.py` — Pydantic request/response
  models.
- `frontend/src/pages/MakerworldPage.tsx` — entire UI (URL paste,
  preview, plates, image gallery, recent imports sidebar, confirm
  modal, in-flight progress labels).
