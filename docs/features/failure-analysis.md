---
title: Failure Analysis
description: Identify patterns and correlate failures with conditions
---

# Failure Analysis

Understand why prints fail by analyzing patterns and correlating failures with environmental conditions, materials, and settings.

---

## :material-chart-timeline-variant: Failure Dashboard

The Failure Analysis widget on the Statistics page shows:

### Failure Rate

Overall success vs failure percentage:

```
Success: 89%  |  Failed: 8%  |  Stopped: 3%
```

### Failure Trend

Track if failure rate is:

- :material-trending-down:{ style="color: #4caf50" } **Improving** - Fewer failures over time
- :material-trending-neutral: **Stable** - Consistent rate
- :material-trending-up:{ style="color: #f44336" } **Worsening** - More failures recently

---

## :material-link-variant: Correlation Analysis

Identify what's causing failures by analyzing correlations:

### By Material

| Material | Success Rate | Failures |
|----------|:------------:|:--------:|
| PLA | 95% | 3 |
| PETG | 88% | 8 |
| ABS | 75% | 12 |
| TPU | 70% | 6 |

Spot materials that need attention.

### By Printer

| Printer | Success Rate | Failures |
|---------|:------------:|:--------:|
| Workshop X1C | 92% | 5 |
| Office P1S | 85% | 10 |
| Garage A1 | 78% | 15 |

Identify problematic machines.

### By Time of Day

| Time | Success Rate |
|------|:------------:|
| Morning (6-12) | 91% |
| Afternoon (12-18) | 89% |
| Evening (18-24) | 88% |
| Night (0-6) | 85% |

Environmental factors may affect overnight prints.

### By Duration

| Print Length | Success Rate |
|-------------|:------------:|
| < 1 hour | 95% |
| 1-4 hours | 91% |
| 4-12 hours | 87% |
| > 12 hours | 80% |

Longer prints have more opportunities to fail.

---

## :material-weather-cloudy: Environmental Factors

If you have environmental data:

### Temperature Correlation

| Room Temp | Success Rate |
|-----------|:------------:|
| 18-22°C | 92% |
| 22-26°C | 88% |
| > 26°C | 82% |

### Humidity Correlation

| Humidity | Success Rate |
|----------|:------------:|
| < 40% | 91% |
| 40-60% | 87% |
| > 60% | 78% |

High humidity affects filament quality.

---

## :material-format-list-bulleted-type: Common Failure Modes

### Adhesion Failures

**Symptoms:**

- Print detaches from bed
- Warping corners
- First layer issues

**Correlations to check:**

- Bed temperature settings
- Bed cleanliness (time since last clean)
- Material type (ABS/PETG more prone)
- Room temperature changes

### Layer Adhesion

**Symptoms:**

- Delamination between layers
- Weak parts
- Cracks along layer lines

**Correlations to check:**

- Nozzle temperature
- Print speed
- Cooling fan settings
- Humidity (wet filament)

### Stringing/Oozing

**Symptoms:**

- Strings between parts
- Blobs on surface

**Correlations to check:**

- Retraction settings
- Temperature (too high)
- Travel speed

### Spaghetti

**Symptoms:**

- Tangled mess of filament
- Print knocked off bed

**Correlations to check:**

- Adhesion (root cause)
- AI detection (was it enabled?)
- Print complexity

---

## :material-text-box-search: Investigating Failures

### Step 1: Filter Failed Prints

1. Go to **Archives**
2. Filter by **Status: Failed**
3. Review the list

### Step 2: Look for Patterns

Check what failed prints have in common:

- Same printer?
- Same material?
- Same time period?
- Similar settings?

### Step 3: Compare with Success

Use Archive Comparison to compare a failed print with a successful one:

1. Select a failed print and similar successful print
2. Right-click > Compare
3. Look for setting differences

### Step 4: Document Findings

Add notes to failed archives:

- What you observed
- Suspected cause
- Changes made

---

## :material-tag: Failure Tags

Tag failed prints for easier analysis:

| Tag | Use For |
|-----|---------|
| `adhesion-fail` | Bed adhesion issues |
| `layer-fail` | Layer adhesion issues |
| `spaghetti` | Complete failures |
| `warping` | Warping issues |
| `stringing` | Quality issues |

Filter by tags to see specific failure types.

---

## :material-camera: Photo Documentation

Add photos to failed prints:

1. Open the failed archive
2. Click **Add Photo**
3. Upload images of the failure

Visual documentation helps:

- Remember what went wrong
- Share with others for help
- Track if fixes work

---

## :material-trending-up: Improving Success Rate

### Quick Wins

1. **Dry your filament** - Wet filament causes many failures
2. **Clean your bed** - Regular cleaning improves adhesion
3. **Level your bed** - Run auto-leveling regularly
4. **Control temperature** - Avoid drafts and temp swings

### Medium-term Fixes

1. **Tune materials** - Create profiles for each material
2. **Calibrate flow** - Ensure correct extrusion
3. **Check hardware** - Worn nozzles, loose belts

### Long-term Solutions

1. **Track patterns** - Use failure analysis regularly
2. **Document changes** - Note what worked
3. **Preventive maintenance** - Follow maintenance schedules

---

## :material-lightbulb: Tips

!!! tip "Don't Delete Failures"
    Keep failed prints in your archive - they're valuable data for analysis.

!!! tip "Consistent Tagging"
    Use consistent tags for failures to enable filtering and pattern detection.

!!! tip "Photo Everything"
    Photos of failures are invaluable for diagnosis and tracking improvements.

!!! tip "Check the Obvious"
    Most failures are due to: wet filament, dirty bed, or temperature issues.

!!! tip "One Change at a Time"
    When fixing issues, change one thing at a time to know what worked.
