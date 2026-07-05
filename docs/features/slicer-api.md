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
- You downloaded a model sliced for one printer (a MakerWorld import, a 3MF from a friend) and want to re-slice it for a **different** printer model &mdash; just pick the target printer in the slice modal.
- You want a "Print" button on MakerWorld imports that goes straight to the printer instead of opening Bambu Studio.

If you only slice from Bambu Studio / OrcaSlicer on your workstation and use Bambuddy as a print log, you don't need this.

---

## :material-cpu-64-bit: Platform requirements

Both sidecar images are published as **pre-built `linux/amd64` images** on GHCR and Docker Hub. No local build is required — `docker compose up -d` pulls the image and starts the service.

| Sidecar             | linux/amd64 | linux/arm64 (RPi 4 / 5, Apple Silicon Linux, ARM cloud VMs) |
|---------------------|:--------------:|:--------------------------------------------------------------:|
| `orca-slicer-api` (default profile) | yes &mdash; pre-built image | yes &mdash; via amd64 emulation, see notes below |
| `bambu-studio-api` (`--profile bambu`) | yes &mdash; pre-built image | yes &mdash; via amd64 emulation, see notes below |

Bambu Studio itself is x86_64-only on every platform (Linux, Windows, macOS Intel/Rosetta) and there is currently no public indication BambuLab plans to ship native ARM64 builds. OrcaSlicer's community ARM64 AppImage extraction fails under QEMU build emulation, and even when it works the OrcaSlicer CLI has known bugs blocking most Bambu-authored 3MFs (see [OrcaSlicer mid-2026 CLI breakage](#orcaslicer-mid-2026-cli-breakage)).

As a workaround on ARM64 hosts, the AMD64 sidecar images can be run under emulation. Performance will be worse than native, but should still be usable. Additional setup to enable emulation is required and described below. **These steps are only required for ARM64 hosts.**
Once this setup is completed, any further steps are the same as for AMD64 hosts.

### Additional requirements for linux/arm64 hosts

The commands below will install two new components:

- `qemu-user-static` &mdash; QEMU user-mode emulation binaries
- `binfmt-support` &mdash; kernel support for running foreign binaries

Similar commands are available for distributions not listed here, but the package names may differ.
The installation requires root privileges (prefix commands with `sudo` if necessary).

| Distribution | Command |
|-------------------------|---------|
| Ubuntu/Debian           | `apt-get update` and `apt-get install qemu-user-static binfmt-support` |
| Fedora / RHEL / CentOS  | `dnf install qemu-user-static qemu-user-binfmt` |

A reboot may be required after installing the packages on some systems.

### Additional requirements for Apple Silicon hosts

Docker Desktop for Mac ships with QEMU emulation enabled by default, so no extra setup should be required.

Troubleshooting:

- Ensure Docker Desktop is up to date and running
- Ensure that the "Use Rosetta for x86/amd64 emulation on Apple Silicon" option is enabled in Docker Desktop settings

---

## :material-rocket-launch: Quick start

The sidecar lives in the optional `slicer-api/` folder of the Bambuddy repo. It is a self-contained Docker Compose stack that pulls pre-built images from GHCR:

```bash
cd slicer-api/
cp .env.example .env       # adjust ports if you like

# OrcaSlicer only (default profile):
docker compose up -d
curl http://localhost:3003/health

# Both slicers:
docker compose --profile bambu up -d
curl http://localhost:3001/health   # bambu-studio-api
curl http://localhost:3003/health   # orca-slicer-api
```

First start pulls pre-built images from GHCR (~110 MB OrcaSlicer, ~220 MB Bambu Studio). No local build, no git in the BuildKit worker &mdash; works on **QNAP Container Station, Synology DSM**, and any other Docker environment without git pre-installed.

!!! info "Sidecar image channel"
    The Compose file defaults to `SIDECAR_TAG=latest` (current stable). To pin to a specific version, set `SIDECAR_TAG=bambuddy-X.Y.Z` in `.env` &mdash; each Bambuddy stable release publishes a matching sidecar image tag (e.g. `bambuddy-0.2.5`).

Then in Bambuddy:

1. **Settings &rarr; Workflow &rarr; Slicer**
2. Pick your **Preferred Slicer** (OrcaSlicer or Bambu Studio) &mdash; this drives the API sidecar.
3. Toggle **Use Slicer API** on
4. Paste the **Sidecar URL** for the chosen slicer (defaults to `http://localhost:3003` for OrcaSlicer, `http://localhost:3001` for Bambu Studio)

The Slice button now appears on file cards.

!!! info "Pairing the API slicer with a different desktop slicer"
    The **Open in Slicer** dropdown right below **Preferred Slicer** controls only the desktop URI handoff (the button that hands a file off to your locally-installed slicer GUI). It defaults to **Same as API slicer** &mdash; pick **Bambu Studio** or **OrcaSlicer** there if you want them to differ. Common case: slice via the Bambu Studio sidecar (more reliable on Bambu-authored 3MFs) while keeping your local "Open in Slicer" button on OrcaSlicer.

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

Pre-pick is automatic and tries to match what the file was prepared with:

- **Printer** and **Process** default to the preset names embedded in the source 3MF's `project_settings.config` (what Bambu Studio / OrcaSlicer recorded when the project was saved), as long as those presets exist in one of your tiers. Files with no embedded slicer config &mdash; an STL, a plain model 3MF &mdash; fall back to the first available preset.
- Each **Filament** dropdown auto-selects against your imported / cloud / standard presets by exact `(filament_type, filament_colour)` match, biased toward presets compatible with the selected printer.

You can override any pick before slicing.

### Profiles filtered by the selected printer

The **Process** and **Filament** dropdowns are filtered by the printer you pick. With a printer selected, only the presets compatible with it show in the main list; presets that belong to a *different* Bambu model drop into an **Other printers** group at the bottom of the dropdown. Compatibility comes from a preset's own `compatible_printers` list (imported profiles) or the `@BBL <model>` suffix in its name (cloud / standard profiles). A preset with no detectable printer &mdash; a custom or renamed profile &mdash; is never hidden and stays in the main list. Switching the printer re-filters both dropdowns immediately and re-picks any selection the change left incompatible.

### Re-slicing for a different printer

The **Printer** dropdown defaults to the printer the source 3MF was prepared for, but is not constrained to it. A 3MF sliced for an X1C can be re-sliced for an H2D (or any other model), and vice versa &mdash; pick the target printer and slice as normal. The slicer regenerates the G-code from scratch using the target printer's bed size, kinematics, nozzle count, and start/end G-code; only the model geometry and paint/colour assignments carry over from the source file.

This makes [MakerWorld](makerworld.md) imports work regardless of which printer the model's creator used.

#### Cross-class re-slice (single-nozzle ↔ H2D)

Re-slicing between a single-nozzle printer (X1C, P1S, A1, P2S, …) and a dual-nozzle printer (H2D / H2D Pro) used to fail with cryptic slicer errors &mdash; *"G-code in unprintable area of multi-extruder printers"* when objects fell into the H2D's per-nozzle dead zones, or a hard slicer crash on multi-color projects. Bambuddy now detects the class change and auto-enables the slicer's **arrange** pass so objects laid out for the source bed are repositioned safely on the target. No extra setting; just pick the new printer and slice.

Two related behaviours come along for the ride:

- **Heterogeneous unused filaments are auto-substituted.** If the modal serves an ABS default into a slot the picked plate doesn't paint with, Bambuddy substitutes the slot-1 filament before slicing so the slicer's loaded-filament temperature validator doesn't reject a PLA print because of an ABS slot the G-code never touches.
- **The re-sliced archive's card shows a sensible cover image.** With `--arrange` on, the slicer doesn't always regenerate the per-plate preview; Bambuddy falls back to the source archive's `plate_N.png` (a render of what the same plate looked like on the source printer) so the card shows the model rather than a blank slot or MakerWorld marketing art.

### Plate picker

For multi-plate 3MFs the modal shows a plate picker first; pick the plate you want to slice, then the preset dropdowns appear for that plate's filament needs.

### "Slice all plates" toggle

Multi-plate projects &mdash; parted statues, multi-part kits, calibration stacks &mdash; get a **Slice all N plates** checkbox in the action bar. With it on:

- Filament dropdowns expand to the *union* of every plate's slot needs (a slot a plate-2 part paints with but plate 1 doesn't is now selectable; without the toggle the modal only showed the picked plate's slots).
- The action button label flips to "Slice all N plates".
- The slicer produces a **single `.gcode.3mf`** with every plate's G-code inside (one Bambuddy archive, all plates).
- For cross-class slice-all, Bambuddy loops per plate behind the scenes (the slicer's `--arrange` is project-wide and would otherwise consolidate every plate's objects onto one bed) and merges the per-plate outputs into one multi-plate 3MF locally. The progress toast shows "Plate 2 of 5 &mdash; Generating G-code (47%)" through the loop. Wall-clock cost is roughly N × the per-plate slice time.

The toggle is hidden on STL / single-plate sources where it'd be meaningless.

### How Bambuddy knows the per-plate filament list

| Source              | When                                       | Speed            |
|---------------------|--------------------------------------------|------------------|
| `slice_info.config` | The 3MF was already sliced by Bambu Studio | Instant          |
| Preview-slice       | Unsliced project file                      | 3&ndash;30 s first time, instant on repeat |
| Painted-face data   | Sidecar unreachable (fallback)             | Instant          |

For unsliced project files Bambuddy runs a fast **preview-slice** via the sidecar to discover the canonical filament list (the slicer's own logic determines which painted regions the print actually uses). Results are cached per `(file, plate)` keyed on file content, so opening the modal a second time on the same plate is instant. If the sidecar can't be reached, Bambuddy falls back to scanning the painted-face quadtree data with a noise threshold &mdash; less precise but better than zero filaments.

For 3MF inputs that already carry embedded settings (e.g. exports from Bambu Studio or OrcaSlicer), Bambuddy still applies your selected presets &mdash; but if the sidecar's CLI rejects that combination (see the OrcaSlicer caveat in [Troubleshooting](#orcaslicer-mid-2026-cli-breakage)), it transparently retries using the 3MF's *embedded* settings instead. The successful result is flagged with `used_embedded_settings: true` in the job state so you can tell which path won.

### Tier priority

Inside the SliceModal, dropdown sections are ordered **Imported &rarr; Orca Cloud &rarr; Bambu Cloud &rarr; Standard**, with auto-pick respecting the same priority when no metadata-aware match is found. Imported profiles win over cloud because they ship with parsed type / colour metadata, while cloud entries are listed by name only (Bambu Cloud rate-limits per-preset content fetches at the scale most users have). When a preset name appears in multiple tiers, Bambuddy backfills the cloud entry's metadata from the imported entry so cross-listed profiles still get auto-picked correctly. The standard tier is the slicer sidecar's stock bundled profiles &mdash; the unconditional fallback if nothing else resolves.

---

## :material-package-variant-closed-remove: Slicer Bundles (removed in 0.2.5)

Bundle import as a managed unit &mdash; the old **Settings &rarr; Slicer &rarr; Slicer Bundles** panel that let you upload a `.bbscfg` and pick its printer + process + filament triplet from a single dropdown &mdash; was removed in 0.2.5. The settings panel still exists; it now shows a permanent notice with the alternatives below.

**Why:** Bambu Studio's `.bbscfg` export strips the system process and filament presets it relies on, so an imported bundle left users without working process presets and slicing silently fell back to the 3MF's embedded settings on STL inputs. Bundle mode also hid the standard tier behind a constrained dropdown.

**Use these instead:**

- **Individual preset imports** &mdash; the same `.bbscfg` / `.bbsflmt` / `.orca_filament` / `.zip` / `.json` files still import their contained presets through [Local Profiles](local-profiles.md) (each preset lands in its own slot and is picked through the normal Printer / Process / Filament dropdowns)
- **[Orca Cloud Profiles](orca-cloud-profiles.md)** &mdash; sync OrcaSlicer's cloud-synced triplets directly
- **[Cloud Profiles](cloud-profiles.md)** &mdash; Bambu Cloud presets, when you have the account

Slice-time lookup order is **Imported &rarr; Orca Cloud &rarr; Bambu Cloud &rarr; Standard** (see [Tier priority](#tier-priority) above), unchanged across cloud and standard paths.

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

### "input preset file invalid" / CLI returns `-5` when slicing via a Cloud preset
Some Bambu Cloud and Orca Cloud filament presets ship with `type` set to "printer" / "print" and a routinely-empty `from` field; the Bambu Studio CLI's `--load-settings` parser rejects both as invalid. From 0.2.5, Bambuddy normalises both fields per slot before sending the payload to the sidecar &mdash; `type` is forced to match the slot (filament / process / printer) and `from` is pinned to `"system"`. Pull the current Bambuddy image to pick up the fix; no sidecar action required.

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

Both sidecar images are published to two registries:

- GHCR: `ghcr.io/maziggy/orca-slicer-api` and `ghcr.io/maziggy/bambu-studio-api`
- Docker Hub: `docker.io/maziggy/orca-slicer-api` and `docker.io/maziggy/bambu-studio-api`

Each stable Bambuddy release publishes two tags per image: `:latest` (current stable) and `:bambuddy-X.Y.Z` (immutable pin matching the Bambuddy version).

Both images are built from the [`maziggy/orca-slicer-api`](https://github.com/maziggy/orca-slicer-api) fork, branch `bambuddy/profile-resolver`. The fork patches:

- **`inherits:` chain resolver** &mdash; walks user-cloned profiles to a root system profile
- **`from: "User"` &rarr; `"system"` rewrite** &mdash; OrcaSlicer CLI's compatibility check rejects user-marked profiles
- **`# ` clone-prefix strip** &mdash; OrcaSlicer GUI prefixes user clones with `# `, which the CLI doesn't accept
- **Sentinel-value strip** &mdash; removes `-1` and `""` placeholders that the CLI rejects as "not in range"

These patches are empirically required to slice real GUI exports without segfaulting the CLI. Once they land upstream, the Compose file can be flipped back to `ghcr.io/afkfelix/orca-slicer-api`.

### Building from source (advanced)

If you want to roll your own sidecar image &mdash; tweaking the resolver, testing a newer slicer AppImage, etc. &mdash; clone the fork and use Docker's git build context:

```yaml
# In docker-compose.yml, replace the `image:` line with:
build:
  context: https://github.com/maziggy/orca-slicer-api.git#bambuddy/profile-resolver
  dockerfile: Dockerfile          # or Dockerfile.bambu-studio
```

This requires `git` in your Docker BuildKit worker. QNAP Container Station and Synology DSM do not ship git by default &mdash; on those platforms, stick with the pre-built images.

---

## :material-update: Updating

```bash
cd slicer-api/
docker compose pull
docker compose --profile bambu up -d
```

That's it &mdash; Compose pulls the current `:latest` (or whatever `SIDECAR_TAG` you've pinned to in `.env`) and recreates the containers.

To roll back to the sidecar that shipped with a previous Bambuddy release, set `SIDECAR_TAG=bambuddy-X.Y.Z` in `.env` and re-run the two commands above.

After the update, the support package (Bambuddy 0.2.5+) records the sidecar's reported slicer version under `integrations.slicer_api.bambu_studio_version` / `orcaslicer_version`. Compare against the released image tag to confirm the new image is the one actually running.

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
