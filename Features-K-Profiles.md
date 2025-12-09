# K-Profiles (Pressure Advance)

Manage pressure advance settings directly on your printers for improved print quality.

## What Are K-Profiles?

K-profiles store pressure advance (also called Linear Advance) settings:
- Compensate for filament compression in the extruder
- Reduce corner bulging and improve sharp edges
- Different values needed for different filaments and speeds

### K-Value Basics
| Material | Typical K-Value Range |
|----------|----------------------|
| PLA | 0.01 - 0.06 |
| PETG | 0.02 - 0.10 |
| ABS | 0.02 - 0.08 |
| TPU | 0.10 - 0.30 |

> Lower values = less compensation. Higher values = more compensation.

---

## Accessing K-Profiles

1. Go to **Settings** > **K-Profiles**
2. Select a connected printer from the dropdown
3. Choose a nozzle size (0.2, 0.4, 0.6, 0.8mm)
4. Profiles are loaded from the printer

---

## Viewing Profiles

### Profile Cards
Each profile displays:
- **K-Value**: The pressure advance factor
- **Profile Name**: Usually the filament name
- **Filament ID**: Material identifier
- **Flow Type**: HF (High Flow) or S (Standard)

### Filtering Options
- **Search**: Filter by profile name or filament ID
- **Nozzle Size**: 0.2, 0.4, 0.6, 0.8mm
- **Flow Type**: All, HF Only, or S Only
- **Extruder**: All, Left Only, or Right Only (dual-nozzle)

---

## Dual-Nozzle Printers (H2 Series)

For H2D, H2C, and H2S printers with dual nozzles:

### Automatic Detection
Bambuddy detects nozzle count from MQTT temperature data:
- Single nozzle: Standard interface
- Dual nozzle: Left/Right column layout

### Column Layout
Profiles are organized by extruder:
- **Left Column**: Left extruder profiles
- **Right Column**: Right extruder profiles

### Extruder Filter
Show profiles for one extruder only:
- All (default)
- Left Only
- Right Only

---

## Editing K-Profiles

1. Click on any profile card
2. Edit modal opens with current values
3. Modify the K-value
4. Click **Save**
5. Profile is updated on the printer

### K-Value Guidelines
- Start with recommended values for your material
- Increase if you see corner bulging
- Decrease if you see gaps at corners
- Small changes (0.01-0.02) make noticeable differences

---

## Adding K-Profiles

1. Click **Add Profile** in the header
2. Select a filament from the dropdown
3. Choose flow type (High Flow or Standard)
4. Choose nozzle size
5. For dual-nozzle: Select Left or Right extruder
6. Enter the K-value
7. Click **Save**

### Filament Selection
The filament dropdown shows:
- Filaments already calibrated on the printer
- Materials from existing K-profiles

> **Note**: New filaments must first be calibrated in Bambu Studio before they appear in this dropdown.

---

## Deleting K-Profiles

1. Click on the profile card
2. Click the trash icon
3. Confirm deletion
4. Profile is removed from the printer

---

## Calibrating K-Values

Bambu Lab printers can auto-calibrate K-values:

### In Bambu Studio
1. Go to Calibration menu
2. Select "Pressure Advance"
3. Choose filament and settings
4. Run calibration print
5. Results saved to printer automatically

### In Bambuddy
After calibration in Bambu Studio:
1. Profiles appear in K-Profiles page
2. View and fine-tune values
3. Add profiles for specific use cases

---

## Understanding Flow Types

### High Flow (HF)
- For high-speed printing
- Faster extrusion rates
- Usually needs lower K-value

### Standard (S)
- Normal printing speeds
- Standard extrusion rates
- Baseline K-value

> Create separate profiles for HF and Standard if you print at varying speeds.

---

## Best Practices

### Per-Material Profiles
Create profiles for each material you use:
- PLA (various brands may differ)
- PETG
- ABS
- Specialty materials

### Per-Nozzle Profiles
Different nozzle sizes need different values:
- 0.2mm: Usually lower K-values
- 0.4mm: Standard K-values
- 0.6mm+: May need adjustment

### Testing Changes
After modifying K-values:
1. Print a test object (sharp corners work well)
2. Examine corner quality
3. Adjust K-value if needed
4. Repeat until satisfied

---

## Troubleshooting

### No Profiles Showing
- Ensure printer is connected
- Select the correct nozzle size
- Check if any profiles exist (calibrate first)

### Can't Add New Filaments
- Filaments must be calibrated in Bambu Studio first
- Bambuddy reads filament list from existing profiles
- Run a calibration print to add new filaments

### Dual-Nozzle Not Detected
- Ensure printer is connected and communicating
- Nozzle count detected from temperature data
- Try disconnecting and reconnecting

### Changes Not Saving
- Verify printer connection is active
- Check for error messages
- Try refreshing the page

---

## Tips

- Calibrate new filaments in Bambu Studio first
- Fine-tune values in Bambuddy for quick adjustments
- Document your optimal K-values for reference
- Different brands of same material may need different values
- Test with a simple calibration print after changes
