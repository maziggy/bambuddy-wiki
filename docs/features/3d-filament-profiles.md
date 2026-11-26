---
title: 3D Filament Profiles
description: Look up spools on 3D Filament Profiles by barcode, with store prices and a Reorder link.
---

# 3D Filament Profiles

[3D Filament Profiles](https://3dfilamentprofiles.com) is a database of filaments with their colours, weights, temperatures and store prices. Connect your account, and the spool form can fill a spool in from a barcode and show where to buy it again.

---

## :material-link-variant: Connecting

Each user connects their own 3D Filament Profiles account. There is no API key to copy.

1. Open **Settings** → **Filament**.
2. On the **3D Filament Profiles** card, click **Connect 3D Filament Profiles**.
3. Bambuddy shows a short code. Click **Open 3D Filament Profiles**, sign in there if asked, check that the page shows the same code, and approve it.
4. Bambuddy connects as soon as you approve. The code expires after 10 minutes; if it does, or if you denied it, click **Connect** again.

Once connected, the card shows the account, its plan and the requests left this month. **Disconnect** ends the connection on both sides. 3D Filament Profiles also ends a connection that is left unused for 90 days, or that you remove from your account on their site; the card then shows **Connect** again.

### Plans

| Account | What works | Requests a month |
|---------|------------|------------------|
| Free | Looking up filaments, store prices | 100 |
| API access (paid, or with a Silver or Gold Patreon membership) | The same, and later spool sync | 1,000, 1,500 or 2,500 |

Every lookup counts as one request, and choosing one of several barcode matches counts as another. Bambuddy keeps a filament's details for a day, so opening a linked spool again doesn't use up requests.

---

## :material-barcode-scan: Looking up a spool

When you're connected, the **Filament** tab of the spool form has a **3D Filament Profiles** box. Enter any of these and press **Look up** (or Enter):

- a **barcode** from the spool's box or label, 8 to 14 digits. A USB barcode scanner types the digits and presses Enter for you.
- a **3D Filament Profiles filament id**, such as `101`.
- a **link** to a filament's page, such as `https://3dfilamentprofiles.com/filament/details/101`.

If a barcode matches several filaments (several colours often share one), pick the right one from the list.

The form is then filled in with the brand, material, type, colour name and colour, the net weight and the weight of the empty spool. The **slicer preset** is filled in too, but only when the spool has none: for brands other than Bambu Lab, 3D Filament Profiles gives Bambu Studio's generic preset (*Generic PLA* for a PLA), and a preset you chose yourself is more specific. Check the fields, then save the spool as usual.

In Spoolman mode the lookup works the same way. It replaces a filament picked from the Spoolman catalogue, as editing those fields does.

## :material-cart: Store prices and Reorder

A looked-up spool stays linked to its 3D Filament Profiles filament. When you open it again, the box shows the filament's current store price for each region 3D Filament Profiles tracks, whether it is in stock, and a **Reorder** link to the shop. **View on 3D Filament Profiles** opens the filament's page there.

The link is kept when you copy a spool or add several at once. **Unlink** removes it; saving the spool then removes it for good.

A user who hasn't connected an account still sees which filament a spool is linked to, with the link to its page, but not the prices.

---

## :material-permission: Permissions

| Permission | Required for |
|------------|--------------|
| `threedfp:auth` | Connecting an account, looking up filaments, seeing store prices |
| `inventory:update` | Linking a spool to a filament, or unlinking it |

Administrators and Operators have `threedfp:auth` by default, Viewers don't. An API key can use its owner's connection when **Allow cloud access** is on for it.

!!! note "Spool sync"
    Keeping spools and their remaining weight the same in Bambuddy and on 3D Filament Profiles needs an account with API access. It is not part of Bambuddy yet.
