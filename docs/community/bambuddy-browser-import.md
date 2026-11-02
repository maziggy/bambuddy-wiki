---
title: Bambuddy Browser Import
description: Chrome and Firefox extension that imports MakerWorld print profiles into your Bambuddy library — pick the plates, pick the folder, one click
---

# Bambuddy Browser Import

A browser extension for Chrome and Firefox that imports MakerWorld print profiles straight into your Bambuddy library from the model page. Open a model on makerworld.com, click the toolbar icon, tick the plates you want, choose a library folder, and hit **Import**. Bambuddy does the download through its own MakerWorld integration &mdash; the extension is just a better front door to it.

![The popup open on a MakerWorld model page](https://bambuddy-import.rojo.dev/docs/screenshot.jpg)

**Author:** [rojosinalma](https://github.com/rojosinalma)
**Repository:** [github.com/rojosinalma/bambuddy-browser-import](https://github.com/rojosinalma/bambuddy-browser-import)
**Website:** [bambuddy-import.rojo.dev](https://bambuddy-import.rojo.dev)
**License:** MIT
**Status:** Independent community project, based on wolfrage76's [MakerWorld Import Extension](https://github.com/wolfrage76/Bambuddy-Extension)

---

## What you need

- A running Bambuddy instance reachable from your browser (LAN address or reverse proxy both work).
- A Bambuddy [API key](../features/api-keys.md) with **Manage Library** and **Allow cloud access** enabled.
- A Bambu Cloud account linked in Bambuddy under **Settings &rarr; Bambu Cloud** &mdash; Bambuddy uses its token to download from MakerWorld, exactly as with the built-in [MakerWorld import](../features/makerworld.md).

---

## Install

- **Chrome / Chromium / Edge / Brave:** [Chrome Web Store](https://chromewebstore.google.com/detail/kcffnknnapkmkabmkbghccgjfcgiobgb), or download the `.zip` from the [latest release](https://github.com/rojosinalma/bambuddy-browser-import/releases/latest) and load it unpacked from `chrome://extensions` with Developer mode on.
- **Firefox (140+):** [Firefox Add-ons](https://addons.mozilla.org/firefox/addon/bambuddy-browser-import/).

Store listings may still be under review shortly after a release; the GitHub release always has the current build.

Then click the extension icon &rarr; gear, enter your Bambuddy URL and API key, and **Test Connection**. Saving asks the browser for permission to reach that one origin &mdash; the extension holds no blanket host permission.

---

## What it does

- **Multi-select** &mdash; import one plate, a handful, or all of them at once. Each card shows print time, colour count, AMS requirement, rating and target printer.
- **Folder picker** &mdash; choose the target library folder from your folder tree, or create a new one from the popup. The last choice is remembered.
- **Background imports** &mdash; the job runs in the extension's background worker, so closing the popup doesn't cancel it. The toolbar badge counts down the remaining plates, reopening the popup shows per-plate status, and a desktop notification fires when everything has landed.
- **Instant popup** &mdash; the model is resolved the moment the page finishes loading and cached briefly, so the popup opens with the profile list already in hand.
- **Open in Bambuddy** &mdash; jumps to the folder the plates were saved to in the File Manager.
- The profile you clicked on MakerWorld (or MakerWorld's default) is pre-selected.

---

## How it talks to Bambuddy

Everything goes through the public REST API with an `X-API-Key` header, no internal endpoints:

| Endpoint | Used for |
|---|---|
| `POST /api/v1/makerworld/resolve` | Model metadata and print profiles for the page you're on |
| `POST /api/v1/makerworld/import` | One call per selected plate, with the chosen `folder_id` |
| `GET /api/v1/library/folders` · `POST` | Folder picker and inline folder creation |
| `GET /api/v1/makerworld/thumbnail` | Thumbnails via Bambuddy's proxy, so your IP never touches MakerWorld's CDN |
| `GET /health` · `GET /api/v1/system/info` · `GET /api/v1/makerworld/status` | The *Test Connection* checks |

The extension has no backend and collects no data; the URL and API key you enter stay in the browser's local extension storage. See the [privacy policy](https://bambuddy-import.rojo.dev/PRIVACY.html).

---

## Support

Issues and feature requests for the extension belong on its [GitHub repository](https://github.com/rojosinalma/bambuddy-browser-import/issues). The Bambuddy team doesn't maintain it.

For Bambuddy-side questions (API keys, Bambu Cloud login, the MakerWorld feature itself), use the [Discord](https://discord.gg/aFS3ZfScHM).
