# :material-tools: SpoolBuddy Assembly Guide

This guide walks you through assembling the SpoolBuddy from 3D-printed parts and electronics. Follow the steps in order — each section builds on the previous one.

---

## :material-printer-3d: Download Printed Parts

All parts are designed for SpoolBuddy and printed in PLA or PETG. Download the full project as a single **`.3mf`** file (recommended — all parts pre-arranged for slicing) or grab individual **`.stl`** files if you only need specific pieces.

**[:material-download: SpoolBuddy.3mf — full project (all parts)](../assets/3d-models/SpoolBuddy.3mf)**

| Part | File | Description |
|------|------|-------------|
| Case — Part 1 (Electronics) | [:material-download: case_part_1_electronics.stl](../assets/3d-models/case_part_1_electronics.stl) | Electronics base — holds scale module, display rail guides |
| Case — Part 2 (Scale) | [:material-download: case_part_2_scale.stl](../assets/3d-models/case_part_2_scale.stl) | Scale housing — load cell channel, joins to Part 1 via dovetails |
| Case Cover | [:material-download: case_cover.stl](../assets/3d-models/case_cover.stl) | Lid for the scale housing (Case Part 2) |
| Case Back Cover | [:material-download: case_backcover.stl](../assets/3d-models/case_backcover.stl) | Back cover for the main enclosure |
| Display Holder | [:material-download: display_holder.stl](../assets/3d-models/display_holder.stl) | Back bracket for the display, screws to case |
| Display Cover | [:material-download: display_cover.stl](../assets/3d-models/display_cover.stl) | Front bezel frame, clips over the display |
| NFC Reader Cover | [:material-download: nfc_reader_cover.stl](../assets/3d-models/nfc_reader_cover.stl) | Cover for the PN5180 module on the spool plate underside |
| Spool Holder | [:material-download: spool_holder.stl](../assets/3d-models/spool_holder.stl) | Spool plate — mounts on load cell, holds filament spool |
| Spool Stopper | [:material-download: spool_stopper.stl](../assets/3d-models/spool_stopper.stl) | Curved arc, clips into spool plate rim |

---

## :material-package-variant: Parts Inventory

Before starting, confirm you have all printed parts:

| Part | Qty | Notes |
|------|-----|-------|
| Case Part 1 — Electronics | 1 | Holds scale module, has display rail guides |
| Case Part 2 — Scale | 1 | Load cell channel, dovetail joins to Part 1 |
| Case Cover | 1 | Snaps onto Case Part 2 (scale housing) |
| Case Back Cover | 1 | Closes the back of the case, shared screws with display holder |
| Display Holder | 1 | Bracket screwed into display standoffs, slides into case rails |
| Display Front Cover Frame | 1 | Beveled bezel frame, clips over the front of the display |
| Spool Plate | 1 | Multiple hole positions for adjusting distance from display |
| Spool Stopper | 1 | Curved arc, clips into spool plate cutout |
| NFC Module Cover | 1 | Clips onto spool plate underside |

---

## :material-screw-flat-top: Hardware Required

| Screw | Qty | Used for |
|-------|-----|----------|
| M2.5 × 16 mm | 4 | Display back cover → case standoffs |
| M2.5 × 12 mm | 2 | Display assembly → case rail lock |
| M5 × 12 mm | 2 | Load cell → Case Part 2 |
| M4 × 20 mm | 1 | Spool plate → load cell |

!!! tip "Nuts"
    Depending on whether your screw holes are threaded or through-holes, you may need matching nuts. Check the fit with a screw before assembly.

---

## :material-printer-3d: Before You Print — NFC Module Cover

!!! warning "Do this before printing the NFC Module Cover"
    The NFC Module Cover has a circular wire exit hole on the right side. If your NFC wiring requires a larger or differently shaped cutout, **use a negative body/part in your slicer** to enlarge it before printing. It is much easier to do this now than to modify a finished print.

---

## Step 1 — Join Case Part 1 and Case Part 2

1. Align the **two dovetail joints** on Case Part 1 (Electronics Base) and Case Part 2 (Scale Housing)
2. Slide the two parts together until they are fully seated
3. No screws are required — the dovetails hold the parts together firmly
4. If you want extra rigidity, a small amount of superglue can be applied to the joint, but this is usually not necessary

---

## Step 2 — Install the Scale Module (NAU7802)

1. Place the NAU7802 PCB onto the **4 standoffs** inside Case Part 1
2. Press down until the board seats on all four standoffs
3. Route the NAU7802 cables through the **oval cable routing hole** in the case wall

---

## Step 3 — Mount the Load Cell

1. Lower the load cell beam into the **channel** in Case Part 2 from above
2. Secure it with **2x M5 × 12 mm screws** through the top holes
3. Route the load cell wires through the **oval cutout** on the side wall of Case Part 2

---

## Step 4 — Fit the Case Cover

1. Align the Case Cover over Case Part 2 with the open gap facing the front (towards the load cell beam)
2. Press down until the **two rear clips** snap in and hold the lid in place
3. The front gap is intentional — it allows the load cell beam to pass through freely

---

## Step 5 — Display Sub-Assembly

!!! note "LAFVIN display standoffs"
    The LAFVIN 7" HDMI Touch display comes with standoffs in the box. Use these to mount the Raspberry Pi — no separate standoffs are needed.

1. Mount the Raspberry Pi onto the back of the display using the included standoffs
2. Connect the HDMI and USB cables between the Pi and the display
3. Attach the **Display Holder** to the back of the display — secure with **2x M2.5 × 16 mm screws** through the two bottom holes into the display standoffs
4. Press the **Display Front Cover Frame** onto the front of the display until the retention clip engages

---

## Step 6 — Install Display Assembly into Case

1. Tilt the display assembly at an angle and align it with the **two sliding rail guides** on Case Part 1
2. Slide the assembly along the rails until it is fully seated

---

## Step 7 — Install Case Back Cover

1. Place the **Case Back Cover** against the back of the case, aligning it over the display holder
2. Secure with the **2x M2.5 × 16 mm screws** through the Case Back Cover holes into the top standoffs of the Display Holder — this locks the Case Back Cover, the Display Holder, and the display together
3. Fasten the **2x M2.5 × 12 mm screws** into the Case Part 1 holes to fully lock the display assembly in place

---

## Step 8 — Spool Plate Sub-Assembly

1. Mount the **PN5180 NFC module** into the bracket on the underside of the spool plate
2. Route the NFC cables through the wire cutout in the NFC Module Cover
3. Clip the **NFC Module Cover** onto the underside bracket — it secures with side clips, no screws needed

---

## Step 9 — Mount the Spool Plate

1. Choose a **screw hole position** on the spool plate based on how far you want it from the display
2. Lower the spool plate onto the load cell, aligning the chosen hole with the load cell mounting point
3. Secure with **1x M4 × 20 mm screw** from the top

---

## Step 10 — Attach the Spool Stopper

1. Align the curved spool stopper arc with the **cutout on the spool plate rim**
2. Press/clip it into place — no screws needed

---

## :material-check-circle: Assembly Complete

Your SpoolBuddy is now fully assembled. Before powering on:

- Check all screw connections are snug
- Confirm the load cell beam moves freely through the case lid gap
- Verify NFC module cables are routed and not pinched
- Plug in the angled USB-C connector to the Pi's USB-C port — the cable exits through the cutout on the back of the case

Proceed to the [Installation](installation.md) guide to set up the software.
