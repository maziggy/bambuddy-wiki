---
title: Windows Installer
description: Install Bambuddy natively on Windows with an interactive PowerShell installer
---

# Windows Installer

The Windows installer sets up Bambuddy natively on Windows without Docker.

It checks the required tools, clones Bambuddy, creates a Python virtual environment,
installs dependencies, creates a start script, adds optional firewall access, and can
register Bambuddy as a Windows Service.

---

## :rocket: Quick Start

Run PowerShell as Administrator and start the installer:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
.\Install-Bambuddy.ps1
```

The installer will guide you through the setup.

It asks for:

- Installation directory
- Bambuddy port
- Firewall rule creation
- Optional Windows Service registration

!!! tip "Recommended"
    Use the default installation path unless you have a specific reason to change it.

---

## :material-folder-cog: Default Installation Path

By default, Bambuddy is installed to:

```text
C:\Bambuddy
```

This path is recommended because Bambuddy is a Git-based Python application that
creates a virtual environment, writes logs, and can be updated later.

Using `C:\Bambuddy` avoids common permission issues with `C:\Program Files`.

!!! info "Custom path supported"
    During setup, the installer asks whether the default path should be used.
    You can provide a custom installation path if needed.

---

## :material-shield-account: Administrator Rights

The installer checks whether it is running with administrator rights.

If it is not elevated, it relaunches itself through UAC.

Administrator rights are required for:

- Installing Git or Python with `winget`
- Creating firewall rules
- Adjusting folder permissions
- Registering Bambuddy as a Windows Service
- Replacing an existing Bambuddy service

---

## :material-source-branch: Git Detection

The installer checks whether Git is available.

If Git is missing, it installs Git automatically with `winget`:

```powershell
winget install --id Git.Git --exact --silent --accept-package-agreements --accept-source-agreements
```

After installation, the script refreshes the current PowerShell `PATH`.

---

## :material-language-python: Python Detection

The installer checks for Python 3 using:

```powershell
python --version
py -3 --version
```

If Python 3 is missing, it installs Python automatically with `winget`:

```powershell
winget install --id Python.Python.3.12 --exact --silent --accept-package-agreements --accept-source-agreements
```

---

## :material-console-line: Installation Flow

The installer performs the following setup steps:

```powershell
git clone https://github.com/maziggy/bambuddy.git
cd bambuddy
python -m venv venv
.\venv\Scripts\python.exe -m pip install --upgrade pip
.\venv\Scripts\pip.exe install -r requirements.txt
```

The generated start script runs Bambuddy with Uvicorn:

```powershell
.\venv\Scripts\python.exe -m uvicorn backend.app.main:app --host 0.0.0.0 --port 8000
```

---

## :material-network: Port Selection

During setup, the installer asks which port Bambuddy should use.

Default port:

```text
8000
```

Example URLs:

```text
http://localhost:8000
http://<windows-host-ip>:8000
```

!!! tip "Choose a free port"
    If another application already uses port `8000`, choose another port such as
    `8010` or `8080`.

---

## :material-shield-lock: Firewall Rule

The installer can optionally create an inbound Windows Firewall rule for the selected port.

Example rule:

| Setting | Value |
|---------|-------|
| Name | `Bambuddy TCP 8000` |
| Direction | Inbound |
| Protocol | TCP |
| Action | Allow |

If Bambuddy should only be accessed locally, the firewall rule is optional.

---

## :material-script-text: Generated Start Script

The installer creates a reusable start script:

```text
C:\Bambuddy\Start-Bambuddy.ps1
```

Manual start:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Bambuddy\Start-Bambuddy.ps1"
```

The start script:

- Sets the Bambuddy working directory
- Uses the virtual environment Python executable
- Starts Uvicorn
- Uses the selected port

---

## :material-file-document: Logging

The installer creates log files in the installation directory.

| File | Description |
|------|-------------|
| `install.log` | Installer actions, dependency checks, setup progress, and errors |
| `bambuddy-runtime.log` | Runtime output from Bambuddy |
| `bambuddy-runtime-error.log` | Runtime errors and stderr output |

Default paths:

```text
C:\Bambuddy\install.log
C:\Bambuddy\bambuddy-runtime.log
C:\Bambuddy\bambuddy-runtime-error.log
```

!!! warning "Avoid double logging"
    When Bambuddy is registered as a Windows Service through NSSM, NSSM handles
    stdout and stderr logging. The generated `Start-Bambuddy.ps1` should not write
    directly to the same runtime log files.

---

## :material-cog-play: Windows Service

At the end of the installation, the installer asks whether Bambuddy should be
registered as a Windows Service.

If enabled, Bambuddy can:

- Start automatically after a Windows reboot
- Run without an open PowerShell window
- Be controlled with Windows service commands
- Restart automatically if the process exits unexpectedly

Service name:

```text
Bambuddy
```

---

## :material-package-variant: NSSM

The installer uses NSSM to run Bambuddy as a Windows Service.

NSSM is used because a normal PowerShell script or Python process does not behave
like a native Windows Service by itself.

The installer stores NSSM in:

```text
C:\Bambuddy\nssm\nssm.exe
```

!!! info "Why NSSM?"
    Windows services are expected to communicate with the Service Control Manager.
    Uvicorn and PowerShell do not do that directly. NSSM wraps and supervises the
    process.

---

## :material-play-circle: Managing the Service

Check status:

```powershell
Get-Service Bambuddy
```

Start:

```powershell
Start-Service Bambuddy
```

Stop:

```powershell
Stop-Service Bambuddy
```

Restart:

```powershell
Restart-Service Bambuddy
```

Show service configuration:

```powershell
sc.exe qc Bambuddy
```

Check startup mode:

```powershell
Get-CimInstance Win32_Service -Filter "Name='Bambuddy'" |
Select-Object Name, State, StartMode, PathName
```

Expected startup mode:

```text
StartMode : Auto
```

---

## :material-refresh-auto: Automatic Startup

When registered as a Windows Service, Bambuddy starts automatically after a system reboot.

The NSSM service is configured with:

```text
SERVICE_AUTO_START
```

The process restart behavior is configured as:

```text
AppExit Default Restart
AppRestartDelay 5000
```

This means Bambuddy is restarted after roughly five seconds if it exits unexpectedly.

---

## :material-update: Updating

If the Bambuddy repository already exists, the installer can update it with:

```powershell
git pull
```

After updating, dependencies are installed again from `requirements.txt`.

!!! tip "Service users"
    If Bambuddy runs as a Windows Service, stop the service before updating and start
    it again afterwards.

```powershell
Stop-Service Bambuddy
.\Install-Bambuddy.ps1
Start-Service Bambuddy
```

---

## :material-console: Useful Commands

### Manual Start

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Bambuddy\Start-Bambuddy.ps1"
```

### View Service Status

```powershell
Get-Service Bambuddy
```

### View Runtime Logs

```powershell
Get-Content "C:\Bambuddy\bambuddy-runtime.log" -Tail 100 -Wait
```

### View Runtime Errors

```powershell
Get-Content "C:\Bambuddy\bambuddy-runtime-error.log" -Tail 100 -Wait
```

### Remove Service

```powershell
Stop-Service Bambuddy -Force -ErrorAction SilentlyContinue
C:\Bambuddy\nssm\nssm.exe remove Bambuddy confirm
```

---

## :material-alert-circle: Troubleshooting

### Installer log is locked

If `install.log` is locked, close old PowerShell windows or previous installer sessions.

Then remove the old log if needed:

```powershell
Remove-Item "C:\Bambuddy\install.log" -Force -ErrorAction SilentlyContinue
```

---

### Runtime log is locked

If `bambuddy-runtime.log` shows repeated file lock errors, make sure the start script
does not write directly to the same file that NSSM uses for stdout or stderr redirection.

Recommended setup:

- `Start-Bambuddy.ps1` writes only to stdout
- NSSM redirects stdout to `bambuddy-runtime.log`
- NSSM redirects stderr to `bambuddy-runtime-error.log`

---

### Service does not start

First test the start script manually:

```powershell
powershell.exe -ExecutionPolicy Bypass -File "C:\Bambuddy\Start-Bambuddy.ps1"
```

If it works manually but not as a service, check the NSSM configuration:

```powershell
C:\Bambuddy\nssm\nssm.exe edit Bambuddy
```

Also check:

```text
C:\Bambuddy\bambuddy-runtime.log
C:\Bambuddy\bambuddy-runtime-error.log
```

---

### Delete and recreate the service

```powershell
Stop-Service Bambuddy -Force -ErrorAction SilentlyContinue
C:\Bambuddy\nssm\nssm.exe remove Bambuddy confirm
```

Then rerun the installer and register the service again.

---

### Git clone fails

Check whether the installation directory is writable:

```powershell
New-Item -Path "C:\Bambuddy\write-test.txt" -ItemType File -Force
Remove-Item "C:\Bambuddy\write-test.txt" -Force
```

If writing works but Git still fails, check antivirus or Windows Defender Controlled Folder Access:

```powershell
Get-MpPreference | Select-Object EnableControlledFolderAccess
```

---

### `takeown /D Y` error on non-English Windows

On German Windows systems, `takeown /D Y` can fail because the localized answer for
"yes" is `J`.

The installer avoids this by using `icacls` and SID-based permission grants instead
of localized `takeown` prompts.

---

## :material-security: Security Notes

- Run the installer only from a trusted source
- Review the PowerShell script before executing it
- Do not expose Bambuddy directly to the internet without protection
- Use a reverse proxy, VPN, or firewall rules for remote access
- Keep Git, Python, Bambuddy, and dependencies updated

---

## :material-arrow-right-circle: Next Steps

After installation:

1. Open Bambuddy in your browser
2. Add your first printer
3. Check the runtime logs if anything does not start correctly
4. Register the Windows Service if Bambuddy should run permanently

Continue with [First Printer Setup](first-printer.md).
