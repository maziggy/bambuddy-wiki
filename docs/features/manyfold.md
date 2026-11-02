---
title: Manyfold Integration
description: Browse and search your self-hosted Manyfold library from Bambuddy and import its 3MF, STL and STEP files to slice and print them.
---

# Manyfold Integration

[Manyfold](https://manyfold.app) is a self-hosted library for 3D models.
Bambuddy can browse and search it and import single files into the
Bambuddy library, from where you slice and print them like any other file.

Nothing is copied ahead of time: Bambuddy only downloads a file when you
import it, and never changes anything in Manyfold.

---

## :material-link-variant: Connecting Manyfold

Bambuddy signs in to Manyfold with an OAuth application.

1. In Manyfold, open **Settings → API** and create a new application.
   Give it the scopes **`public`** and **`read`**. Bambuddy needs nothing
   more.
2. Note the application's **client ID** and **client secret**.
3. In Bambuddy, open **Model Sources** in the sidebar and pick the
   **Manyfold** tab.
4. Enter Manyfold's URL (for example `http://192.168.1.10:3214`, or
   `https://example.com/manyfold` behind a reverse proxy), the client ID and
   the client secret.
5. Click **Test connection**. Bambuddy signs in and shows how many models it
   can see. Then click **Save**.

!!! info "What Bambuddy can see"
    Bambuddy sees the models that the application's **owner** can see in
    Manyfold. To limit what Bambuddy shows, create the application with a
    Manyfold user whose access covers just those models.

To change the connection later, click **Connection** at the top of the
Manyfold tab. Leave the secret field empty to keep the stored secret.
**Disconnect** makes Bambuddy forget the URL, client ID and secret. Files you
imported stay in your library.

The client secret is never sent back to the browser and is left out of
[GitHub backups](backup.md). After restoring one, enter it again.

---

## :material-magnify: Browsing and importing

- **Search** takes Manyfold's own search syntax. The page size is the
  application owner's Manyfold setting.
- **Previews** come from Manyfold. A 3D file shows a picture only when
  Manyfold rendered one: turn on **3D Renders** in Manyfold under
  **Settings → File Derivatives**. Otherwise the card shows a placeholder.
- **Open a model** to see its description, tags, licence, a link to it in
  Manyfold, and its files.
- **Import** downloads a **3MF**, **STL** or **STEP** file into the library.
  The default destination is a **Manyfold** folder, created on first use;
  **Import to** picks another folder. Other file types are listed but
  can't be imported, because Bambuddy can't slice or print them.
- An imported file shows **In library**, with **Show in library** and
  **Slice**. Importing it again doesn't download it twice. A file you
  deleted from the library can be imported again. Only copies you can
  open in the File Manager count: if you see only your own files, a file a
  colleague imported is imported again as yours.
- Files are limited to 200 MB.

### Importing several files

- **In a model:** tick files, or tick **Select all**, then click **Import
  selected**. **Import all** imports every printable file that isn't in the
  library yet.
- **From the grid:** tick models with the box in each card's corner, or
  click **Select page**. The ticks stay while you page and search. **Import
  their files** imports every printable file of those models that isn't in
  the library yet, into the folder chosen next to it. **Clear selection**
  removes all ticks.

Files import one after another. A file that fails doesn't stop the rest;
at the end a message says how many were imported, were already in the
library, or failed, and names the ones that failed.

---

## :material-shield-account: Permissions

| Permission | Allows |
|---|---|
| `manyfold:view` | Browse and search Manyfold and open models |
| `manyfold:import` | Import files into the library |
| `settings:update` | Connect, change or disconnect Manyfold |

When you update, every group gets the Manyfold permissions that match its
MakerWorld permissions: `makerworld:view` brings `manyfold:view`, and
`makerworld:import` brings `manyfold:import`. This happens once, so a
permission you take away later stays away.

Until Manyfold is connected, the Manyfold tab is shown only to users who can
connect it.

---

## :material-api: API

| Endpoint | Method | Permission | Purpose |
|---|---|---|---|
| `/api/v1/manyfold/config` | GET | `settings:read` | URL, client ID and whether a secret is stored |
| `/api/v1/manyfold/config` | PUT | `settings:update` | Store the connection; an empty secret keeps the stored one |
| `/api/v1/manyfold/config` | DELETE | `settings:update` | Disconnect |
| `/api/v1/manyfold/config/test` | POST | `settings:update` | Sign in with the given values and count the models; stores nothing |
| `/api/v1/manyfold/status` | GET | `manyfold:view` | Whether Manyfold is connected, and its URL |
| `/api/v1/manyfold/models` | GET | `manyfold:view` | One page of models; `q` searches, `page` pages |
| `/api/v1/manyfold/models/{id}` | GET | `manyfold:view` | A model's details and files, with the library file of each earlier import |
| `/api/v1/manyfold/models/{id}/preview` | GET | `manyfold:view` | The model's preview image; 404 when it has none |
| `/api/v1/manyfold/import` | POST | `manyfold:import` | Import one file: `{"model_id", "file_id", "folder_id"}` |

Errors from Manyfold come back with a `code`, such as
`manyfold_credentials`, `manyfold_scope` or `manyfold_unreachable`. They use
HTTP 502, never 401, so a refused Manyfold secret doesn't look like an
expired Bambuddy session.

---

## :material-help-circle: Troubleshooting

**"Manyfold did not accept the client ID or secret"**
: Check both values in Manyfold under **Settings → API**.

**"The Manyfold application needs the read scope"**
: Edit the application in Manyfold and add the `read` scope.

**"Manyfold is limiting sign-ins"**
: Manyfold allows 10 sign-ins in 3 minutes. Bambuddy keeps its sign-in
  for about two hours, so this mostly happens while trying out credentials.
  Wait a few minutes.

**"Bambuddy can't reach Manyfold"**
: The URL must be reachable from the machine Bambuddy runs on, not just
  from your browser. In Docker, `localhost` is the Bambuddy container
  itself: use the host's address instead. HTTPS needs a certificate the
  Bambuddy host trusts.

**A model shows no files or isn't found**
: The application's owner can't see it in Manyfold.
