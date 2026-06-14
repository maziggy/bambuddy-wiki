---
title: Orca Cloud Profiles
description: Sync your OrcaSlicer Cloud profiles into Bambuddy for slicing.
---

# Orca Cloud Profiles

Sync your OrcaSlicer 2.4.0+ cloud-synced filament / process / printer
profiles into Bambuddy and use them alongside your Bambu Cloud, locally
imported, and standard bundled presets.

!!! info "Companion to Bambu Cloud Profiles"
    Orca Cloud profiles **don't replace** [Bambu Cloud Profiles](cloud-profiles.md) — both tabs live side by side in the Profiles page and you can connect to either, both, or neither. Profile precedence in the slicer is **Imported > Orca Cloud > Bambu Cloud > Standard**, so a locally-imported preset wins over the same name in either cloud, and an Orca Cloud preset wins over a Bambu Cloud one with the same name.

---

## :material-cloud: Overview

Orca Cloud Profiles lets you:

- **Sign in** to your Orca Cloud account from Bambuddy (four providers)
- **View** all profiles your Orca account has synced — filament, process, printer
- **Slice** with Orca profiles directly from the Bambuddy SliceModal
- **Assign** an Orca filament profile to an AMS slot from the Configure modal
- **Spool inventory**: pick Orca filaments when creating or editing a spool, including from a SpoolBuddy kiosk

The integration is read-only on the Orca Cloud side — Bambuddy lists and uses your profiles but doesn't create, edit, or delete them. Make changes in OrcaSlicer itself; refresh the tab in Bambuddy to pick them up.

---

## :material-key: Connecting

### Sign-in providers

Bambuddy offers four ways to sign in:

| Provider | Flow |
|----------|------|
| **Email + password** *(default)* | Submit credentials directly to Orca's auth backend; no browser redirect |
| **Google** | Browser-based OAuth via Orca's Supabase auth |
| **GitHub** | Browser-based OAuth via Orca's Supabase auth |
| **Apple** | Browser-based OAuth via Orca's Supabase auth |

### Email + password sign-in

1. Open **Profiles** → **Orca Cloud** tab
2. Click **Sign in with email and password** (the primary button)
3. Enter your Orca account credentials
4. Click **Sign in**

That's it — no paste step, no browser redirect.

### OAuth sign-in (Google / GitHub / Apple)

1. Open **Profiles** → **Orca Cloud** tab
2. Click the provider you want
3. A new tab opens at Orca's sign-in page — complete authentication there
4. Your browser will try to load a `localhost:41172/callback` URL and show *"This site can't be reached"* — **that's expected**, not an error
5. Copy the full URL from your browser's address bar
6. Paste it into the textarea in Bambuddy and click **Finish connecting**

!!! warning "The localhost page failing IS the expected state"
    Orca Cloud's Supabase project only allows `localhost` as an OAuth callback target. Bambuddy isn't running on your browser's localhost, so the redirect can't reach Bambuddy directly — you ferry the URL across manually. This is the same friction OrcaSlicer's own desktop client navigates internally, just exposed to you in Bambuddy's case. The open feature request to broaden the allowlist is at [OrcaSlicer/OrcaSlicer#14028](https://github.com/OrcaSlicer/OrcaSlicer/issues/14028).

### After signing in

Once connected, the Orca Cloud tab shows your synced profiles grouped into **Filament / Process / Printer** columns, with the same filter dropdowns and detail view as the Bambu Cloud tab.

---

## :material-permission: Permissions

| Permission | Required for |
|------------|--------------|
| `orca_cloud:auth` | Sign in / sign out / read profiles / slice with Orca presets |

The permission is granted to the **Administrators** group by default. Add it to other groups in **Settings → Users & Groups → Groups** if you want non-admin users to access Orca Cloud.

!!! tip "API keys"
    To let an API key read Orca Cloud presets on its owner's behalf (useful for headless slicing pipelines), enable **Allow cloud access** on the key. The same scope covers both Bambu Cloud and Orca Cloud — same trust dimension, one toggle.

---

## :material-scissors-cutting: Slicing with Orca profiles

The **SliceModal** preset dropdowns surface profiles from all four tiers in priority order:

1. **Imported** — local OrcaSlicer / Bambu Studio profile imports (highest; ship with parsed type / colour metadata)
2. **Orca Cloud** (this tab) — synced OrcaSlicer Cloud profiles
3. **Bambu Cloud** — your Bambu Cloud account
4. **Standard** — the slicer's stock bundled profiles (unconditional fallback)

If a profile of the same name exists in multiple tiers (rare in practice), the higher-priority one wins. Status banners at the top of the modal flag each cloud independently — you'll see a *"Sign in to Orca Cloud"* hint if you're connected to Bambu Cloud but not Orca, and vice versa.

For **multi-color** prints, the per-plate filament pre-pick benefits from Orca Cloud's inline `filament_type` and `default_filament_colour` metadata (no per-preset lookup needed), giving the matcher more to work with than Bambu Cloud presets where this metadata isn't surfaced in the list response &mdash; but a locally-imported preset of the same name still wins on tier priority above the cloud match.

---

## :material-printer-3d-nozzle: AMS slot assignment

The **Configure AMS Slot** modal (both from the printer card and from SpoolBuddy) treats Orca profiles like local imports for the printer-firmware side: Bambu's firmware can't resolve Orca's UUID profile IDs, so Bambuddy derives a generic Bambu filament_id from the parsed material type (PLA → GFL99, PETG → GFG99, ABS → GFB99, etc.) and uses that for the slot's `tray_info_idx`. The slot mapping record is persisted with `preset_source='orca_cloud'` so Bambuddy can display the right profile name on hover and surface the right one on subsequent opens.

---

## :material-package-variant: Inventory and SpoolBuddy

Orca filament profiles appear in the filament-preset picker of:

- **Spool form** (Settings → Inventory → Add Spool / Edit Spool)
- **SpoolBuddy kiosk** Write-Tag page
- **SpoolBuddy AMS Configure** flow

Both clouds are fetched in parallel; either or both can be connected. If you only connect Orca, the picker simply shows Orca filaments where it would otherwise show Bambu Cloud ones.

---

## :material-debug-step-over: Troubleshooting

### "The Orca Cloud sign-in flow expired after 10 minutes"

The PKCE handshake state is held server-side for 10 minutes. If you took longer than that between clicking Connect and pasting back the callback URL (or the kiosk was reloaded mid-flow), click Connect again to start fresh — the new flow generates a fresh verifier.

### Profiles tab shows "Connected" but the list is empty

Your Orca account has no profiles synced yet, or you signed in with an account that doesn't have OrcaSlicer profile sync enabled. Open OrcaSlicer, enable cloud sync from Preferences, sync at least one filament profile, then click Refresh on the Bambuddy tab.

### The Configure AMS Slot picker doesn't show my Orca profile

If you're using an API key (e.g., from SpoolBuddy), check the key has **Allow cloud access** enabled — the same scope gates both Bambu Cloud and Orca Cloud reads. Otherwise the kiosk's request is silently rejected and no Orca profiles appear.

### Sign-in works, but the slicer doesn't see my Orca filament's metadata

Orca's filament profiles include `filament_type` and `default_filament_colour` inline, but only after the first sync round-trip. If you added a brand-new filament profile in OrcaSlicer and immediately tried to slice with it in Bambuddy, hit **Refresh** on the Orca Cloud tab to re-pull the latest snapshot.

---

## :material-link-variant: Related

- [Bambu Cloud Profiles](cloud-profiles.md) — the Bambu side of the dual-cloud setup
- [Local Profiles](local-profiles.md) — for offline / non-Orca / non-Bambu profile sources
- [Server-Side Slicing](slicer-api.md) — how preset tiers feed the slicer
- [API Keys & Webhooks](api-keys.md) — to expose cloud presets to headless callers
