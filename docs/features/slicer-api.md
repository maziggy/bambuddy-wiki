---
title: Server-Side Slicing
description: Slice STL/3MF files without a desktop slicer using the optional slicer-api sidecar
---

# Server-Side Slicing (Slicer API)

Bambuddy can slice STL and 3MF files server-side using OrcaSlicer or Bambu Studio running headlessly inside Docker. The **Slice** button in File Manager, Archives, and the MakerWorld page dispatches a background job that produces a ready-to-print `.gcode.3mf` in the same folder &mdash; no desktop slicer install required.

This is **opt-in**. If you don't run the sidecar, all "Open in Slicer" flows continue to use the existing URI-scheme handoff to your desktop slicer, unchanged.

---

## :material-information: When to use it

- You run Bambuddy on a headless box (NAS, mini-PC, RPi 5) and want one-click slicing without bouncing through a desktop machine.
- You want re-slice on existing archives (e.g. swap filament, change layer height) and have the result land back in Bambuddy automatically.
- You want a "Print" button on MakerWorld imports that goes straight to the printer instead of opening Bambu Studio.

If you only slice from Bambu Studio / OrcaSlicer on your workstation and use Bambuddy as a print log, you don't need this.

---

## :material-rocket-launch: Quick start

The sidecar lives in the optional `slicer-api/` folder of the Bambuddy repo. It is a self-contained Docker Compose stack:

```bash
cd slicer-api/
cp .env.example .env       # adjust ports / versions if you like

# OrcaSlicer only (default profile):
docker compose up -d
curl http://localhost:3003/health

# Both slicers:
docker compose --profile bambu up -d
curl http://localhost:3001/health   # bambu-studio-api
curl http://localhost:3003/health   # orca-slicer-api
```

First build downloads the slicer's AppImage (~110 MB OrcaSlicer, ~220 MB Bambu Studio) and compiles the Node wrapper. Plan for **3&ndash;8 minutes per service**. Subsequent runs reuse the local image &mdash; instant start.

!!! info "Sidecar updates aren't automatic"
    The Compose file builds from the `bambuddy/profile-resolver` branch tip. A plain `docker compose up -d` keeps using your originally-built image, even when the branch advances upstream. To pick up new endpoints and fixes, see [Updating](#updating) and rebuild with `--no-cache --pull`.

Then in Bambuddy:

1. **Settings &rarr; Workflow &rarr; Slicer**
2. Pick your **Preferred Slicer** (OrcaSlicer or Bambu Studio)
3. Toggle **Use Slicer API** on
4. Paste the **Sidecar URL** for the chosen slicer (defaults to `http://localhost:3003` for OrcaSlicer, `http://localhost:3001` for Bambu Studio)

The Slice button now appears on file cards.

---

## :material-numeric-3-circle: Ports

| Service          | Default host port | Notes                                                                                       |
|------------------|------------------:|---------------------------------------------------------------------------------------------|
| `orca-slicer-api`  | **3003**          | Bambuddy's [virtual-printer](virtual-printer.md) feature reserves 3000 and 3002             |
| `bambu-studio-api` | **3001**          | First free port in that range                                                              |

Override with `ORCA_API_PORT` / `BAMBU_API_PORT` in `slicer-api/.env`.

!!! warning "Port conflicts with virtual printers"
    Don't change `ORCA_API_PORT` to 3000 or 3002. Those ports are owned by Bambuddy's virtual-printer listener and changing the slicer-api port to either will cause `address already in use` at startup.

---

## :material-cog: How it works

The Slice flow runs server-side in the background:

```
Click Slice
    │
    ▼
Bambuddy enqueues a job and returns 202 + job_id
    │
    ├─► Modal closes immediately
    ├─► Toast tracker polls /api/v1/slice-jobs/{job_id}
    │
    ▼
Background task forwards source file + presets to the sidecar
    │
    ├─► Sidecar runs `OrcaSlicer-Soft --slice 1 --load-settings ... --load-filaments ... --outputdir ...`
    │
    ▼
Resulting .gcode.3mf saved to Bambuddy library / archives
    │
    ▼
Toast: "Slice complete" + library/archives list refreshes automatically
```

Jobs survive the lifetime of the Bambuddy process (kept in-memory for 30 minutes after completion). Restart Bambuddy and in-flight jobs are lost.

---

## :material-account-cog: Picking presets

Slice opens a modal with **Printer**, **Process**, and one or more **Filament** dropdowns &mdash; populated from your imported [Local Profiles](local-profiles.md), [Cloud Profiles](cloud-profiles.md), and the slicer-bundled standard tier. The **Filament** rows render dynamically based on the picked plate's actual AMS slot usage:

- **Single-color plate** &rarr; one filament dropdown.
- **Multi-color plate** &rarr; one dropdown per AMS slot the print uses, each labeled `Filament N (PLA)` with a colour swatch.

Pre-pick is automatic: each filament dropdown auto-selects against your imported / cloud / standard presets by exact `(filament_type, filament_colour)` match. You can override any pick before slicing.

### Plate picker

For multi-plate 3MFs the modal shows a plate picker first; pick the plate you want to slice, then the preset dropdowns appear for that plate's filament needs.

### How Bambuddy knows the per-plate filament list

| Source              | When                                       | Speed            |
|---------------------|--------------------------------------------|------------------|
| `slice_info.config` | The 3MF was already sliced by Bambu Studio | Instant          |
| Preview-slice       | Unsliced project file                      | 3&ndash;30 s first time, instant on repeat |
| Painted-face data   | Sidecar unreachable (fallback)             | Instant          |

For unsliced project files Bambuddy runs a fast **preview-slice** via the sidecar to discover the canonical filament list (the slicer's own logic determines which painted regions the print actually uses). Results are cached per `(file, plate)` keyed on file content, so opening the modal a second time on the same plate is instant. If the sidecar can't be reached, Bambuddy falls back to scanning the painted-face quadtree data with a noise threshold &mdash; less precise but better than zero filaments.

For 3MF inputs that already carry embedded settings (e.g. exports from Bambu Studio or OrcaSlicer), Bambuddy still applies your selected presets &mdash; but if the sidecar's CLI rejects that combination (see the OrcaSlicer caveat in [Troubleshooting](#orcaslicer-mid-2026-cli-breakage)), it transparently retries using the 3MF's *embedded* settings instead. The successful result is flagged with `used_embedded_settings: true` in the job state so you can tell which path won.

### Tier priority

Inside the SliceModal, dropdown sections are ordered **Imported &rarr; Cloud &rarr; Standard**, with auto-pick respecting the same priority when no metadata-aware match is found. Imported profiles win over cloud because they ship with parsed type / colour metadata, while cloud entries are listed by name only (Bambu Cloud rate-limits per-preset content fetches at the scale most users have). When a preset name appears in both tiers, Bambuddy backfills the cloud entry's metadata from the local entry so cross-listed profiles still get auto-picked correctly.

---

## :material-package: Slicer Bundles (.bbscfg)

Bambuddy can import a Bambu Studio **Printer Preset Bundle** (`.bbscfg`) and slice through it without re-uploading the JSON profile triplet on every print. Useful when you've curated a printer + process + filament set in Bambu Studio and want every Bambuddy slice to use that exact triplet.

### Export from Bambu Studio

1. Open Bambu Studio
2. **File &rarr; Export &rarr; Export Preset Bundle**
3. Pick **Printer preset bundle (.bbscfg)** &mdash; the first option
4. Save the `.bbscfg` file somewhere you can reach from your browser

The bundle is a zip containing one printer preset, every process preset that belongs to it, and every filament preset compatible with it &mdash; plus a `bundle_structure.json` manifest. Each inner JSON is a delta with an `inherits:` chain pointing at the BBL system presets baked into the slicer.

### Import into Bambuddy

1. **Settings &rarr; Slicer &rarr; Slicer Bundles**
2. Click **Upload bundle**, pick the `.bbscfg`
3. The card shows the printer name, number of process / filament presets, and a delete button

Re-uploading the same `.bbscfg` is a no-op &mdash; Bambuddy hashes the file content and reuses the existing import.

### Slice via a bundle

1. Open the slice modal on any library file or archive
2. The new **Slicer bundle** dropdown at the top of the modal lists every imported bundle
3. Pick a bundle &rarr; the cloud / local / standard preset dropdowns are replaced with bundle-scoped pickers (process and filament names from the bundle's contents). The printer is implicit (each `.bbscfg` ships with exactly one)
4. Slice as normal

The dispatch path is faster than the preset triplet for repeat slicing: instead of resolving three preset refs and uploading three JSONs to the sidecar per slice, Bambuddy just sends `bundle_id + preset names` and the sidecar materialises the JSONs from disk.

The preview slice is also bundle-aware: when a bundle is selected, the per-plate filament discovery slice runs against that bundle's process settings so the displayed gram numbers match what the real print will produce. Without a bundle picked, the preview falls back to the file's embedded settings (slot mapping is unchanged either way &mdash; that's a model property, not a process-settings property).

---

## :material-folder-arrow-right-outline: Where slice results land

| Source kind   | Destination                                                                              |
|---------------|------------------------------------------------------------------------------------------|
| Library file  | New `.gcode.3mf` in the **same folder** as the source                                    |
| Archive       | New archive with the printer/project metadata copied from the source, name suffixed `(re-sliced)` |
| MakerWorld    | After import, behaves like a Library file slice                                          |

Sliced output is always exported as `.gcode.3mf` (not plain `.gcode`) so File Manager can pull the embedded thumbnail. The badge shows `GCODE` (blue), and the displayed filename matches the source's print name when set.

---

## :material-bug: Troubleshooting

### "Failed to slice the model"
The sidecar wraps the CLI's stderr but doesn't surface it on the API by default. Re-run inside the container to see the underlying error:

```bash
docker exec orca-slicer-api /app/squashfs-root/AppRun --slice 1 \
    --load-settings "/path/to/printer.json;/path/to/preset.json" \
    --load-filaments /path/to/filament.json \
    --allow-newer-file --outputdir /tmp/out /path/to/model.3mf
```

### `/health` reports `version: "unknown"`
Cosmetic. The bundled binary works fine; the wrapper just couldn't parse the version string from the slicer's `--help` output. Bambu Studio uses a different `--help` format than OrcaSlicer (which is what the wrapper was originally tuned for).

The same wrapper bug also reports the `checks` field as `orcaslicer` for *both* sidecars (including `bambu-studio-api`). Both are cosmetic and don't indicate the wrong image &mdash; use the steps in the next section to confirm freshness.

### "Name cannot be empty" or "Only JSON files are allowed" when importing a `.bbscfg`
Your sidecar image was built before 2026-05-13, when the `/profiles/bundle` endpoint landed on `bambuddy/profile-resolver` (it had previously lived only on a feature branch the compose file didn't reference). Older images route bundle uploads through the generic preset-upload handler, which either rejects with "Name cannot be empty" (no `name` form field) or "Only JSON files are allowed" (the JSON multer filter doesn't accept `.bbscfg`). Rebuild:

```bash
cd slicer-api/
docker compose --profile bambu build --no-cache --pull
docker compose --profile bambu up -d
```

`--pull` is the key flag &mdash; without it BuildKit may reuse the cached git context and you'll end up with the same image. The support bundle surfaces both error reasons starting with Bambuddy 0.2.5.

### Slice job stays "queued" forever
Check the Bambuddy logs for connection errors to the sidecar URL. Common causes:

- Sidecar container not running (`docker compose ps` to verify)
- `Sidecar URL` field in Settings doesn't match the actual host/port
- Bambuddy is running in Docker on a different network than the sidecar &mdash; use the host's LAN IP instead of `localhost`

### Profile resolver errors ("not compatible with printer")
The fork's profile resolver walks OrcaSlicer's `inherits:` chain to a root system profile and rewrites `from: "User"` &rarr; `from: "system"`. If you exported your preset from a non-stock OrcaSlicer build, the chain may not resolve cleanly. Workaround: re-export the preset from a stock OrcaSlicer install, or open an issue with the upstream profile bundled.

### OrcaSlicer mid-2026 CLI breakage
OrcaSlicer 2.3.2 / 2.4.0-dev have known CLI bugs that block slicing many Bambu-authored 3MFs &mdash; see upstream [SoftFever/OrcaSlicer#12426](https://github.com/SoftFever/OrcaSlicer/issues/12426) (segfault on painted multi-extruder files) and [#13386](https://github.com/SoftFever/OrcaSlicer/issues/13386) (parameter-range strict-validation reject). **Bambu Studio is recommended** until the upstream fixes land &mdash; the `bambu-studio-api` service is a drop-in replacement with the same API surface. Switch via **Settings &rarr; Workflow &rarr; Preferred Slicer**.

For 3MF inputs that hit the CLI bugs anyway, Bambuddy automatically retries without `--load-settings` (using the file's embedded settings). The job still completes with `used_embedded_settings: true` flagged in the result.

---

## :material-source-fork: Sidecar source

The Bambuddy `slicer-api/` Compose file builds both services from a fork of [AFKFelix/orca-slicer-api](https://github.com/AFKFelix/orca-slicer-api), branch `bambuddy/profile-resolver`. The fork patches:

- **`inherits:` chain resolver** &mdash; walks user-cloned profiles to a root system profile
- **`from: "User"` &rarr; `"system"` rewrite** &mdash; OrcaSlicer CLI's compatibility check rejects user-marked profiles
- **`# ` clone-prefix strip** &mdash; OrcaSlicer GUI prefixes user clones with `# `, which the CLI doesn't accept
- **Sentinel-value strip** &mdash; removes `-1` and `""` placeholders that the CLI rejects as "not in range"

These patches are empirically required to slice real GUI exports without segfaulting the CLI. Once they land upstream, the Compose file can be flipped back to `ghcr.io/afkfelix/orca-slicer-api`.

The Compose file uses Docker's git-build-context, so you don't need to clone the fork manually &mdash; Docker pulls the repo at build time.

---

## :material-update: Updating

Bump the slicer versions in `slicer-api/.env`, then:

```bash
cd slicer-api/
docker compose --profile bambu build --no-cache --pull
docker compose --profile bambu up -d
```

Both flags matter:

- `--no-cache` is needed because the Dockerfile downloads the AppImage inline; Docker won't re-fetch on a version change otherwise.
- `--pull` forces BuildKit to re-fetch the git context (the `bambuddy/profile-resolver` branch tip). Without it, an older cached git context will silently be reused even with `--no-cache`, leaving you on the same sidecar code you originally built &mdash; this is the most common reason "I rebuilt and it still doesn't work" reports surface.

After the rebuild, the support package (Bambuddy 0.2.5+) records the sidecar's reported slicer version under `integrations.slicer_api.bambu_studio_version` / `orcaslicer_version`. Compare it against the value in `slicer-api/.env` to confirm the new image is the one actually running.

### Orphan containers after a rebuild

If `docker compose up -d` errors with

```
Error response from daemon: Conflict. The container name "/bambu-studio-api" is already
in use by container "..."
```

the existing container was created from an older `slicer-api/docker-compose.yml` whose image tags didn't carry the `bambuddy-` prefix (the rename happened when the bundle-import branch landed). Compose tracks containers by project labels &mdash; the old containers' labels don't match the current project, so `docker compose down` doesn't see them, but `container_name:` still pins the name.

One-time cleanup:

```bash
docker rm -f bambu-studio-api orca-slicer-api
docker compose --profile bambu up -d
```

Optionally clear the now-unreferenced old images:

```bash
docker image rm bambu-studio-api:bambu02.06.00.51 orca-slicer-api:resolver-orca2.3.2
```

Only required once &mdash; the next `up -d` cycle creates containers under the correct project labels and `docker compose down` works normally from then on.
