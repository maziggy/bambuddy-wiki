# Real-time Monitoring

Bambuddy provides live monitoring of all your connected Bambu Lab printers through WebSocket-based real-time updates.

## Printer Status

Each printer card displays real-time information:

### Connection Status
- **Green indicator**: Connected and communicating
- **Red indicator**: Disconnected or connection error
- **Yellow indicator**: Connecting or reconnecting

### Temperature Readouts
| Sensor | Description |
|--------|-------------|
| Nozzle | Current hotend temperature |
| Bed | Heated bed temperature |
| Chamber | Enclosure temperature (if available) |

### Print Progress
When a print is active:
- **Progress bar**: Visual completion percentage
- **Current layer**: Layer X of Y
- **Time remaining**: Estimated time to completion
- **Filament used**: Grams consumed

---

## HMS Error Monitoring

The Health Management System (HMS) monitors printer health:

### Status Indicator
Always visible on printer cards:
- **Green "OK"**: No issues detected
- **Yellow**: Minor issues or warnings
- **Orange**: Serious errors requiring attention
- **Red**: Fatal errors - stop printing

### Error Severity Levels
| Level | Color | Action Required |
|-------|-------|-----------------|
| Info | Blue | Informational only |
| Common | Yellow | Check when convenient |
| Serious | Orange | Address before next print |
| Fatal | Red | Immediate attention needed |

### Error Details
Click the HMS indicator to see:
- Error code and description
- Affected component
- Recommended action
- Link to Bambu Lab support article

---

## MQTT Debug Logging

Built-in debugging tool for printer communication:

### Starting Debug Logging
1. Go to printer settings
2. Click "Start MQTT Debug"
3. Messages are captured in real-time

### Viewing Messages
- **Incoming**: Messages from printer to Bambuddy
- **Outgoing**: Commands sent to printer
- **JSON payloads**: Expandable for detailed inspection
- **Auto-refresh**: New messages appear automatically

### Use Cases
- Troubleshooting connection issues
- Understanding printer behavior
- Debugging automation problems
- Reporting issues to developers

---

## AMS (Automatic Material System)

For printers with AMS units:

### Slot Status
- **Filament color**: Visual swatch for each slot
- **Material type**: PLA, PETG, ABS, etc.
- **Remaining**: Estimated filament left
- **Temperature**: Drying chamber temp (if applicable)

### Humidity Monitoring
- Current humidity level in AMS
- Warning indicators for high humidity

---

## Camera Feed

Access your printer's camera:

1. Click the camera icon on a printer card
2. View live camera feed
3. Take snapshots
4. Monitor print progress visually

### Camera Page
Dedicated camera view with:
- Larger video display
- Multiple printer support
- Snapshot history

---

## WebSocket Architecture

Bambuddy uses WebSocket for real-time updates:

```
Printer → MQTT → Bambuddy Backend → WebSocket → Browser
```

### Connection Handling
- Automatic reconnection on disconnect
- State synchronization on reconnect
- Efficient delta updates (only changed data)

### Performance
- Low latency updates (<1 second typical)
- Minimal bandwidth usage
- Multiple browser tabs supported

---

## Notifications on Status Changes

Configure alerts for printer events:
- **Printer Offline**: When connection is lost
- **Printer Error**: When HMS errors occur
- **Print Complete**: When a job finishes
- **Print Failed**: When a print fails

See [Push Notifications](Features-Notifications) for setup details.

---

## Tips

- Keep the Printers page open for at-a-glance monitoring
- Use the camera page for visual confirmation of print quality
- Enable HMS error notifications to catch problems early
- Check MQTT debug logs if a printer behaves unexpectedly
