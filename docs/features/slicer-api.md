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

Slice opens a modal with three dropdowns &mdash; **Printer**, **Process**, **Filament** &mdash; populated from your imported [Local Profiles](local-profiles.md) or [Cloud Profiles](cloud-profiles.md). All three must be picked before the **Slice** button enables.

For 3MF inputs that already carry embedded settings (e.g. exports from Bambu Studio or OrcaSlicer), Bambuddy still applies your selected presets &mdash; but if the sidecar's CLI rejects that combination (a known issue with OrcaSlicer 2.3.x and the H2D printer), it transparently retries using the 3MF's *embedded* settings instead. The successful result is flagged with `used_embedded_settings: true` in the job state so you can tell which path won.

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

### Slice job stays "queued" forever
Check the Bambuddy logs for connection errors to the sidecar URL. Common causes:

- Sidecar container not running (`docker compose ps` to verify)
- `Sidecar URL` field in Settings doesn't match the actual host/port
- Bambuddy is running in Docker on a different network than the sidecar &mdash; use the host's LAN IP instead of `localhost`

### Profile resolver errors ("not compatible with printer")
The fork's profile resolver walks OrcaSlicer's `inherits:` chain to a root system profile and rewrites `from: "User"` &rarr; `from: "system"`. If you exported your preset from a non-stock OrcaSlicer build, the chain may not resolve cleanly. Workaround: re-export the preset from a stock OrcaSlicer install, or open an issue with the upstream profile bundled.

### OrcaSlicer + H2D segfault
OrcaSlicer 2.3.x has a CLI bug where `--load-settings` segfaults on H2D inputs. The Bambuddy backend automatically retries without `--load-settings` for 3MF inputs (using the file's embedded settings). The job still completes successfully with `used_embedded_settings: true`.

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
docker compose --profile bambu build --no-cache
docker compose --profile bambu up -d
```

`--no-cache` is needed because the Dockerfile downloads the AppImage inline; Docker won't re-fetch on a version change otherwise.
