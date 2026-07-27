---
title: Bug Report
description: Submit bug reports directly from the Bambuddy UI
---

# Bug Report

Submit bug reports directly from the Bambuddy UI without leaving the application.

---

## :material-bug: Overview

The in-app bug report feature provides a quick way to report issues:

- **One-click access** — Floating bug button in the bottom-right corner
- **Screenshot support** — Upload, paste from clipboard, or drag & drop images
- **Interactive debug capture** — Start logging, reproduce the issue at your own pace, stop & submit
- **Privacy-first** — All sensitive data is sanitized before submission
- **GitHub integration** — Reports create GitHub issues automatically

---

## :material-stethoscope: Setup-problem check

When you open the bug report form, Bambuddy quietly runs a
[Connection Diagnostic](../reference/troubleshooting.md) against your printers.
If a printer has a connection problem — a blocked port, LAN Developer Mode
turned off, Docker bridge networking, wrong access code — the detected issue
and its fix are shown right in the form.

Most bug reports turn out to be setup issues, so it's worth reading this panel
first: fixing the highlighted problem often resolves things without a report.
You can still submit a report regardless.

---

## :material-send: Submitting a Report

1. Click the red **bug icon** in the bottom-right corner of any page
2. **Describe the issue** — What went wrong? Steps to reproduce?
3. **Add a screenshot** (optional) — Upload a file, paste from clipboard (Ctrl+V), or drag & drop an image onto the upload area. Images are automatically compressed to JPEG.
4. **Add your email** (optional) — If provided, your email will be included in a collapsed section of the GitHub issue so the maintainer can follow up.
5. Click **Start Debug Logging**

---

## :material-bug-play: Debug Log Capture

After clicking Start Debug Logging, a 3-step guided flow begins:

1. **:material-check-circle: Debug logging enabled** — Bambuddy enables debug-level logging and queries all connected printers for fresh status data
2. **:material-circle: Reproduce the issue now** — An elapsed timer counts up while you reproduce the bug at your own pace. Take as long as you need (auto-stops after 5 minutes).
3. **:material-circle-outline: Stop & submit report** — Click "Stop & Submit" when you're done reproducing

The logs are collected, sanitized, and submitted along with your description, screenshot, and system info.

---

## :material-shield-lock: Data Privacy

The bug report includes a detailed, expandable privacy notice explaining exactly what data is collected.

### Included in the report

- App version, OS, architecture, Python version
- Database statistics (counts only)
- Printer models, nozzle counts, firmware versions
- Connectivity status
- Integration status (Spoolman, MQTT, Home Assistant)
- Non-sensitive settings
- Network interface count
- Docker details
- Dependency versions
- Sanitized application logs (last 200 lines)

### Never included

- Printer names or serial numbers
- Access codes or passwords
- IP addresses
- Email addresses (except your own, if you provide it)
- API keys or tokens
- Webhook URLs
- Hostnames or usernames
- LDAP Distinguished Names (a DN's `CN` is the user's real name)

---

## :material-server: How It Works

Bug reports are submitted through a secure relay service on `bambuddy.cool`. This relay:

- Holds the GitHub API token server-side (never shipped in the application)
- Uploads screenshots and log files to GitHub
- Creates issues with appropriate labels
- Rate-limited to prevent abuse (5 reports per hour)

Your Bambuddy instance never needs a GitHub token — the relay handles all GitHub communication.
