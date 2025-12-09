# Maintenance Tracker

Schedule and track printer maintenance to keep your printers in optimal condition.

## Overview

The maintenance tracker helps you:
- Define maintenance tasks with intervals
- Track when maintenance was last performed
- Get reminders when maintenance is due
- Log maintenance history

---

## Maintenance Types

### Default Maintenance Tasks

Bambuddy includes common maintenance types:

| Task | Description | Suggested Interval |
|------|-------------|-------------------|
| Nozzle Change | Replace worn nozzle | 500 print hours |
| Lubrication | Lubricate linear rails and lead screws | 200 print hours |
| Belt Tension | Check and adjust belt tension | 100 print hours |
| Clean Build Plate | Deep clean the print surface | 50 print hours |
| Clean Nozzle | Remove debris from nozzle | 20 print hours |
| Firmware Update | Check for firmware updates | 30 days |

### Custom Maintenance Types

Create your own maintenance tasks:
1. Go to **Settings** > **Maintenance**
2. Click **Add Maintenance Type**
3. Configure:
   - **Name**: Task description
   - **Interval Type**: Print hours or calendar days
   - **Interval Value**: How often (e.g., every 100 hours)
   - **Description**: Detailed instructions (optional)
4. Click **Save**

---

## Interval Types

### Print Hours
Maintenance triggered based on actual printing time:
- Tracks cumulative print hours per printer
- More accurate for wear-based maintenance
- Example: "Every 500 print hours"

### Calendar Days
Maintenance triggered based on time passed:
- Simple date-based tracking
- Good for time-sensitive tasks
- Example: "Every 30 days"

---

## Tracking Maintenance

### Maintenance Page

View all maintenance for all printers:
- Status indicators (OK, Due, Overdue)
- Next due date/hours
- Last completed date

### Per-Printer View

See maintenance for a specific printer:
1. Go to Printers page
2. Click maintenance icon on printer card
3. View that printer's maintenance status

### Status Indicators

| Status | Color | Meaning |
|--------|-------|---------|
| OK | Green | Not due yet |
| Due Soon | Yellow | Due within 10% of interval |
| Due | Orange | Maintenance interval reached |
| Overdue | Red | Past due date/hours |

---

## Logging Maintenance

When you perform maintenance:

1. Find the maintenance task
2. Click **Log Maintenance**
3. Optionally add notes
4. Click **Save**

The tracker updates:
- Last performed date
- Next due date/hours calculated
- Entry added to history

---

## Maintenance History

View past maintenance for any task:
1. Click on a maintenance task
2. View history log showing:
   - Date performed
   - Print hours at time of maintenance
   - Notes (if any)
   - Who performed it (if tracked)

---

## Notifications

Get notified when maintenance is due:

### Enabling Notifications
1. Go to **Settings** > **Notifications**
2. Enable **Maintenance Due** event
3. Configure provider(s)

### Notification Timing
- Sent when maintenance becomes due
- Reminder sent if still due after 24 hours
- No spam—one notification per due task

---

## Managing Maintenance Types

### Editing Types
1. Go to **Settings** > **Maintenance**
2. Click on a maintenance type
3. Modify settings
4. Click **Save**

### Deleting Types
1. Go to **Settings** > **Maintenance**
2. Click delete icon on a type
3. Confirm deletion

> Deleting a type removes it from all printers.

### Per-Printer Customization
Different printers may need different intervals:
1. Go to printer's maintenance view
2. Override interval for that printer
3. Original type remains unchanged

---

## Print Hour Tracking

Bambuddy automatically tracks print hours:
- Accumulated during active prints
- Stored per printer
- Used for hour-based maintenance intervals

### Viewing Print Hours
- Displayed on printer cards
- Available in statistics
- Shown in maintenance calculations

### Resetting Print Hours
After major service (like a rebuild):
1. Go to printer settings
2. Find "Print Hours" section
3. Click "Reset" or enter new value
4. Maintenance intervals recalculate

---

## Best Practices

### Regular Schedule
- Check maintenance page weekly
- Address "Due" items promptly
- Don't ignore "Overdue" warnings

### Document Everything
- Add notes when logging maintenance
- Record any issues found
- Track parts replaced

### Customize for Your Printers
- Adjust intervals based on usage
- Add tasks specific to your workflow
- Remove irrelevant default tasks

### Preventive vs Reactive
- Preventive maintenance prevents failures
- Cheaper than repairs
- Less downtime

---

## Tips

- Set conservative intervals initially, adjust based on experience
- Use calendar-based intervals for firmware/software updates
- Use print-hour intervals for wear-related maintenance
- Enable notifications to never miss maintenance
- Log maintenance immediately after performing it
