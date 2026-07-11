---
title: Windows Installer
description: Install Bambuddy natively on Windows with a self-contained .exe installer
---

# Windows Installer

The Windows installer is a self-contained `.exe` that bundles everything
Bambuddy needs: an embedded Python 3.13 distribution, the pre-built React
frontend, ffmpeg, and NSSM (the Windows service supervisor). **No Python,
Node, Docker, Git, or any other runtime is required on the target
machine.**

It installs Bambuddy as a Windows service running on
`http://localhost:8000`, so the dashboard is always available after a
reboot without any manual start.

---

## :material-download: Download

Latest stable release:

[:material-download: bambuddy-windows-x64-setup.exe](https://github.com/maziggy/bambuddy/releases/latest/download/bambuddy-windows-x64-setup.exe){ .md-button .md-button--primary }

For daily beta builds (newest features, less tested): browse the
[releases page](https://github.com/maziggy/bambuddy/releases) and pick the
most recent prerelease tagged `v<version>-daily.<date>`.

!!! info "Windows 10 / 11 x64 only"
    The installer targets 64-bit Windows 10 and 11. ARM64 (Surface Pro X,
    Snapdragon) is not currently supported — most Bambuddy deps don't ship
    ARM64 Windows wheels.

---

## :material-shield-alert: SmartScreen Warning on First Run

Until our [SignPath OSS code-signing](https://signpath.io/open-source)
approval lands, the installer is unsigned. Windows SmartScreen will warn:

> **Windows protected your PC** — Microsoft Defender SmartScreen prevented an unrecognised app from starting.

Click **More info** → **Run anyway** to proceed. The installer is built
in public on GitHub Actions ([windows-installer.yml](https://github.com/maziggy/bambuddy/blob/main/.github/workflows/windows-installer.yml))
so you can verify the build provenance directly.

Once signing is wired in, the warning will go away after a short
SmartScreen reputation-building period across the first signed releases.

---

## :material-cog-play: Installer Flow

When you run the `.exe`:

1. **UAC prompt** — admin elevation is required because the installer
   registers a Windows service and writes to `C:\Program Files\` and
   `C:\ProgramData\`. This is a one-time prompt during install only;
   normal Bambuddy use doesn't need admin.
2. **License + install directory** — defaults to `C:\Program Files\Bambuddy\`.
3. **Optional firewall rule** — checkbox to add a Windows Firewall inbound
   rule for TCP `8000` so devices on your LAN can reach the dashboard.
   Default: enabled. Uncheck if you only want loopback access from this
   machine.
4. **Optional desktop shortcut** — default: off.
5. **Install** — files copy, Bambuddy service registers and starts.
6. **Browser opens** — default browser navigates to `http://localhost:8000`.

---

## :material-folder-cog: Installation Layout

```text
C:\Program Files\Bambuddy\        Install root (managed by installer)
├── python\                       Embedded Python 3.13 + venv
├── app\
│   ├── backend\                  Backend source
│   ├── static\                   Frontend bundle (served by FastAPI)
│   └── gcode_viewer\             3D preview iframe assets
├── bin\
│   ├── nssm.exe                  Windows service supervisor
│   ├── ffmpeg.exe                Timelapse / camera processing
│   └── ffprobe.exe
├── service\                      Service install/uninstall scripts
├── bambuddy.ico                  App icon
└── unins000.exe                  Uninstaller

C:\ProgramData\Bambuddy\          Data root (preserved on uninstall)
├── data\
│   ├── bambuddy.db               SQLite database
│   ├── archive\                  Archived 3MF files + thumbnails
│   └── plate_calibration\        Plate detection cache
└── logs\
    ├── service-stdout.log        NSSM-captured uvicorn stdout
    ├── service-stderr.log        NSSM-captured uvicorn stderr
    └── *.log                     Bambuddy application logs
```

The split between `Program Files` (read-only after install, replaced on
update) and `ProgramData` (persistent user data) means **updates never
touch your database or archives**.

---

## :material-cog-play: Windows Service

Bambuddy runs as a Windows service named `Bambuddy`, registered to start
automatically at boot. The service runs as `LocalSystem`.

| Setting | Value |
|---------|-------|
| Service name | `Bambuddy` |
| Display name | `Bambuddy` |
| Startup type | Automatic |
| Account | `LocalSystem` |
| Restart on failure | Yes (NSSM-managed, 5s delay) |

!!! info "Why LocalSystem?"
    The optional Virtual Printer feature needs to bind privileged ports
    (322 / 990 / 8883) for the RTSPS camera proxy, FTPS file server, and
    MQTT broker that emulate a real Bambu printer for slicer uploads.
    Only `LocalSystem` or a similarly-privileged account can bind ports
    below 1024 on Windows. If you don't use Virtual Printer mode, the
    extra privilege isn't doing anything — it's there so you can flip
    the feature on later without a service-account change.

### Managing the service

```powershell
Get-Service Bambuddy            # Status
Start-Service Bambuddy          # Start
Stop-Service Bambuddy           # Stop
Restart-Service Bambuddy        # Restart
```

Or via the GUI: `services.msc` → find `Bambuddy`.

---

## :material-update: Updating

To update to a new version, just **download and run the newer installer
over the existing install**. Inno Setup detects the prior install,
stops the running Bambuddy service, replaces the program files, and
re-registers + restarts the service automatically.

- Your data in `C:\ProgramData\Bambuddy\` is untouched.
- Your settings (notifications, integrations, OAuth tokens) persist.
- The service stops for ~5 seconds during the file swap.

```text
1. Download the new bambuddy-windows-x64-setup.exe
2. Run it (UAC prompt)
3. Click through the wizard (your existing config is detected)
4. Service restarts on the new version automatically
```

No manual `Stop-Service` is needed — the installer handles that with a
pre-install hook. If you do see a "file in use" error, stop the service
manually (`Stop-Service Bambuddy` in admin PowerShell) and click Retry.

---

## :material-trash-can: Uninstalling

Standard Windows uninstall flow:

- **Settings → Apps** → search "Bambuddy" → **Uninstall**, or
- **Control Panel → Programs and Features** → Bambuddy → **Uninstall**

The uninstaller stops the service, removes the Windows Firewall rule,
deletes the `C:\Program Files\Bambuddy\` install tree, and deregisters
the service.

!!! info "Your data is preserved by default"
    `C:\ProgramData\Bambuddy\` (database, archives, logs) is **not**
    removed by the uninstaller. If you reinstall Bambuddy later, you'll
    pick up exactly where you left off. To wipe everything, delete the
    `C:\ProgramData\Bambuddy\` folder manually after uninstall.

---

## :material-file-document: Logs

Bambuddy writes three log streams on Windows:

| File | Source |
|------|--------|
| `C:\ProgramData\Bambuddy\logs\service-stdout.log` | NSSM-captured uvicorn stdout |
| `C:\ProgramData\Bambuddy\logs\service-stderr.log` | NSSM-captured uvicorn stderr + Python tracebacks |
| `C:\ProgramData\Bambuddy\logs\bambuddy.log` | Bambuddy's own rotating application log |

Quick log access:

```powershell
# Tail the service stdout
Get-Content "C:\ProgramData\Bambuddy\logs\service-stdout.log" -Tail 100 -Wait

# Tail Bambuddy's app log
Get-Content "C:\ProgramData\Bambuddy\logs\bambuddy.log" -Tail 100 -Wait
```

A Start Menu shortcut **Bambuddy → Bambuddy Logs** opens the logs folder
in Explorer for quick triage.

---

## :material-firewall: Windows Firewall

If you ticked **Add firewall rule** during install, an inbound TCP rule
is created for port 8000:

| Setting | Value |
|---------|-------|
| Name | `Bambuddy Dashboard` |
| Direction | Inbound |
| Protocol | TCP |
| Port | 8000 |
| Action | Allow |

This makes the dashboard reachable from other devices on your LAN at
`http://<windows-host-ip>:8000`. The uninstaller removes this rule.

!!! warning "Bambuddy is unauthenticated by default"
    With the firewall rule active, anyone on your LAN can open the
    Bambuddy dashboard without credentials. Open
    **Settings → Security** in the dashboard and enable authentication
    before relying on LAN access, especially on shared / guest networks.

---

## :material-lan: Virtual Printer + Network Interfaces

When you add a Virtual Printer, the **Bind IP** dropdown lists every
network interface on the Windows host that has an IPv4 address. On
Windows you may see entries you don't see on Linux:

- `Ethernet`, `Wi-Fi` — your real LAN adapters
- `vEthernet (WSL)`, `vEthernet (Default Switch)` — Hyper-V virtual switches
- `Tailscale` — if you have Tailscale installed
- `Bluetooth Network Connection` — if active

This is intentional — Bambuddy doesn't filter Windows-specific virtual
adapters because some users legitimately want to bind a VP to one
(common cases: binding to the Tailscale interface for remote slicing
across a tailnet, or to a WSL interface for slicing from Linux tools
running inside WSL).

For most users, pick the real LAN adapter that matches the subnet your
Bambu printer is on.

---

## :material-alert-circle: Troubleshooting

### Installer file-in-use error during update

If you see "permission denied" or "file in use" mid-install when
upgrading, the previous Bambuddy service is still holding `python.exe`
or its DLLs. The pre-install hook should stop the service automatically,
but if it didn't:

```powershell
# Admin PowerShell
Stop-Service Bambuddy
```

Then click **Retry** in the installer dialog.

### Service won't start after install

Check the NSSM stderr log first:

```powershell
Get-Content "C:\ProgramData\Bambuddy\logs\service-stderr.log" -Tail 50
```

Common causes:

- **Port 8000 already in use** by another application. Stop the other
  app or change Bambuddy's port via env var. (Port change UX is a
  v1.1 polish item — for now, edit `C:\Program Files\Bambuddy\service\install-service.bat`
  and re-run it as admin to re-register the service on a different port.)
- **Antivirus blocking the embedded Python** — quarantines `python.exe`.
  Add `C:\Program Files\Bambuddy\` to your AV's exclusion list.

### Browser doesn't open at end of install

The auto-open is best-effort; if your default-browser config is unusual
(e.g. a privacy launcher), it may not fire. Use the Start Menu shortcut
**Bambuddy → Open Bambuddy Dashboard**, or just point any browser at
`http://localhost:8000`.

### Virtual Printer dropdown is empty

If you see no interfaces when adding a VP: restart the service and
refresh the page. On some Windows configurations (heavy
firewall / endpoint protection) `psutil.net_if_addrs()` returns stale
data until the next service restart.

### Removing everything cleanly

```powershell
# 1. Uninstall via Settings → Apps
# 2. Then wipe persistent data:
Remove-Item "C:\ProgramData\Bambuddy" -Recurse -Force
```

---

## :material-source-branch: Build From Source

The installer is built in CI from the public GitHub Actions workflow at
[`.github/workflows/windows-installer.yml`](https://github.com/maziggy/bambuddy/blob/main/.github/workflows/windows-installer.yml).
Build inputs:

- [`installers/windows/build.py`](https://github.com/maziggy/bambuddy/blob/main/installers/windows/build.py) — stages embedded Python + deps + frontend + vendored NSSM + ffmpeg
- [`installers/windows/bambuddy.iss`](https://github.com/maziggy/bambuddy/blob/main/installers/windows/bambuddy.iss) — Inno Setup compiler script
- [`installers/windows/vendor/nssm.exe`](https://github.com/maziggy/bambuddy/tree/main/installers/windows/vendor) — vendored NSSM 2.24

To build locally you need Windows 10/11 + Python 3.11+ + Node.js 22 + Inno
Setup 6. See
[`installers/windows/README.md`](https://github.com/maziggy/bambuddy/blob/main/installers/windows/README.md)
for the build steps.

---

## :material-arrow-right-circle: Next Steps

After install:

1. Open Bambuddy at `http://localhost:8000`
2. Add your first Bambu printer — see [First Printer Setup](first-printer.md)
3. Optionally enable authentication under **Settings → Security**
