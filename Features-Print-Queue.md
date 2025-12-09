# Print Queue & Scheduling

Schedule prints for specific times with powerful automation including smart plug integration for automatic power control.

## Overview

The print queue allows you to:
- Queue multiple prints per printer
- Schedule prints for specific date/time
- Automatically power on printers before scheduled prints
- Automatically power off after prints complete
- Drag-and-drop queue reordering

---

## Adding Prints to Queue

### From Archives
1. Go to the **Archives** page
2. Right-click an archive
3. Select **Schedule** (or click the calendar icon)
4. Choose options:
   - **Printer**: Select target printer
   - **Scheduled Time**: Leave empty for "next available" or pick a specific time
   - **Auto Power Off**: Turn off printer when complete
5. Click **Add to Queue**

### Queue Options
| Option | Description |
|--------|-------------|
| **Printer** | Which printer to send the job to |
| **Scheduled Time** | When to start (empty = ASAP) |
| **Auto Power Off** | Power off printer after completion |

---

## Managing the Queue

### Queue Page
Access the dedicated queue page to see all queued prints across all printers.

### Per-Printer Queue
Click the queue icon on any printer card to see that printer's queue.

### Queue Item States
| State | Description |
|-------|-------------|
| **Pending** | Waiting for scheduled time or previous print |
| **Waiting** | Scheduled time reached, waiting for printer |
| **Uploading** | Transferring 3MF to printer |
| **Printing** | Currently printing |
| **Completed** | Print finished successfully |
| **Failed** | Print failed or was cancelled |

### Reordering
Drag and drop queue items to change print order:
1. Click and hold the grip handle
2. Drag to new position
3. Release to reorder

> Note: Only pending items can be reordered. Active prints cannot be moved.

### Cancelling Prints
- **Pending items**: Click the X button to remove
- **Active prints**: Click the stop button (with confirmation)

---

## Automation Flow

When a scheduled print is ready to start:

```
1. Scheduled time reached
        ↓
2. Smart plug powers ON (if configured)
        ↓
3. Wait for printer connection (up to 2 minutes)
        ↓
4. Upload 3MF via FTP
        ↓
5. Start print command sent
        ↓
6. Monitor progress in real-time
        ↓
7. Print completes
        ↓
8. If "Auto Power Off" enabled:
   - Wait for nozzle to cool below 50°C
   - Power off smart plug
```

---

## Smart Plug Integration

For full automation, link a smart plug to your printer. See [Smart Plugs](Features-Smart-Plugs.md) for setup.

### Pre-Print Power On
When a scheduled print is about to start:
1. Bambuddy sends power-on command to smart plug
2. Waits for printer to boot and connect
3. Proceeds with print upload once connected

### Post-Print Power Off
When a print completes with "Auto Power Off" enabled:
1. Waits for nozzle temperature to drop below threshold
2. Sends power-off command to smart plug
3. Marks queue item as complete

---

## Scheduling Tips

### Overnight Prints
Schedule long prints to start overnight:
1. Queue the print with a late evening start time
2. Enable "Auto Power Off"
3. Printer runs while you sleep and turns off when done

### Morning Warm-Up
Schedule a print to start before you arrive:
1. Set start time 30 minutes before arrival
2. Printer warms up and starts
3. Print is running when you get there

### Batch Printing
Queue multiple prints for sequential execution:
1. Add all prints to the queue (no scheduled time)
2. First print starts immediately
3. Each subsequent print starts when the previous completes

---

## Error Handling

### Connection Timeout
If the printer doesn't connect within 2 minutes after power-on:
- Queue item marked as failed
- Error message displayed
- Smart plug remains on (manual intervention needed)

### Upload Failure
If FTP upload fails:
- Automatic retry (up to 3 attempts)
- Queue item marked as failed if all retries fail

### Print Failure
If the print fails mid-job:
- Queue item marked as failed
- Next queued item does NOT auto-start
- Manual intervention required

---

## Queue Notifications

Get notified about queue events:
- **Print Started**: When a queued print begins
- **Print Completed**: When a queued print finishes
- **Print Failed**: When a queued print fails

Configure notifications in [Push Notifications](Features-Notifications.md).

---

## Limitations

- One active print per printer at a time
- Scheduled prints require the printer to be available
- Smart plug automation requires compatible Tasmota device
- Queue items expire after 24 hours if printer remains unavailable

---

## Tips

- Use "Auto Power Off" for energy savings and safety
- Schedule long prints during off-peak electricity hours
- Keep the queue page open to monitor progress
- Set up notifications to know when prints complete
