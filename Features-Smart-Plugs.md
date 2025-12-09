# Smart Plug Integration

Control Tasmota-based smart plugs for automated power management of your 3D printers.

## Overview

Smart plug integration enables:
- Automatic power-on when prints start
- Safe power-off after prints complete (with cooldown)
- Scheduled power on/off times
- Power consumption monitoring
- Energy alerts

---

## Supported Devices

Any smart plug running [Tasmota](https://tasmota.github.io/docs/) firmware with HTTP API enabled:

### Popular Compatible Devices
- Sonoff S31 / S26
- Gosund smart plugs
- Teckin smart plugs
- Treatlife smart plugs
- Any ESP8266/ESP32-based plug with Tasmota

### Requirements
- Tasmota firmware installed
- HTTP API enabled (default)
- Same network as Bambuddy server

---

## Setting Up a Smart Plug

### Step 1: Flash Tasmota (if needed)
If your plug doesn't have Tasmota:
1. Follow the [Tasmota installation guide](https://tasmota.github.io/docs/Getting-Started/)
2. Configure WiFi to connect to your network
3. Note the plug's IP address

### Step 2: Add to Bambuddy
1. Go to **Settings** > **Smart Plugs**
2. Click **Add Plug**
3. Enter the plug's IP address
4. Click **Test** to verify connection
5. Enter a name (auto-filled from device if available)
6. Optionally add authentication credentials
7. Click **Add**

### Step 3: Link to Printer
1. In the plug settings, find **Linked Printer**
2. Select the printer this plug controls
3. The plug is now associated with that printer

---

## Automation Options

### Master Toggle
**Enabled**: Turns all automation on/off for this plug

### Auto Power On
When enabled, the plug turns on automatically when:
- A print starts on the linked printer
- A scheduled print is about to begin

### Auto Power Off
When enabled, the plug turns off after:
- A print completes successfully
- The delay mode conditions are met

---

## Delay Modes

Control when the printer powers off after a print completes:

### Time-Based Delay
Wait a fixed number of minutes:
- Range: 1-60 minutes
- Use case: Simple cooldown period
- Example: Power off 15 minutes after print ends

### Temperature-Based Delay
Wait until nozzle cools down:
- Default threshold: 50°C (configurable)
- Safer for the printer and environment
- Monitors actual nozzle temperature

> **Recommendation**: Use temperature-based delay for safety. Hot nozzles can be a fire hazard.

---

## Scheduled Power On/Off

Set daily schedules for automatic power control:

### Scheduled Power On
Turn on the plug at a specific time each day:
- Use case: Warm up printer before work hours
- Example: Power on at 8:00 AM

### Scheduled Power Off
Turn off the plug at a specific time each day:
- Use case: Safety shutoff at night
- Example: Power off at 11:00 PM

### Configuring Schedules
1. Expand the plug settings panel
2. Enable **Scheduled On** and/or **Scheduled Off**
3. Set the desired time for each
4. Schedules run daily at the specified times

---

## Power Monitoring

For Tasmota plugs with energy monitoring (e.g., Sonoff S31):

### Live Power Display
- Current wattage shown on plug card
- Updates in real-time

### Power Alerts
Get notified when power consumption exceeds a threshold:
1. Enable **Power Alert** in plug settings
2. Set **Power Threshold** in watts (e.g., 200W)
3. Receive notification when threshold exceeded

### Use Cases for Power Alerts
- Detect printer issues (unexpected high power)
- Confirm heater failures (power too low during printing)
- Verify printer is actively heating/printing

---

## Manual Control

Each plug card displays:
- **Status**: ON / OFF / Offline
- **Power**: Current consumption (if supported)
- **On/Off buttons**: Manual toggle

Click the buttons to manually control the plug at any time.

---

## Plug Settings Panel

Expand any plug card to access all settings:

| Setting | Description |
|---------|-------------|
| **Name** | Display name for the plug |
| **IP Address** | Network address of the plug |
| **Username/Password** | Authentication (if Tasmota requires it) |
| **Linked Printer** | Which printer this plug controls |
| **Enabled** | Master automation toggle |
| **Auto On** | Power on when print starts |
| **Auto Off** | Power off when print completes |
| **Delay Mode** | Time-based or temperature-based |
| **Delay Value** | Minutes or temperature threshold |
| **Scheduled On/Off** | Daily power schedule |
| **Power Alert** | Enable consumption alerts |
| **Power Threshold** | Wattage for alerts |

---

## Troubleshooting

### Plug Not Responding
1. Verify IP address is correct
2. Check plug is on the same network
3. Test via browser: `http://<plug-ip>/cm?cmnd=Power`
4. Access Tasmota web UI: `http://<plug-ip>`

### Authentication Issues
If Tasmota has a password:
1. Edit the plug in Bambuddy
2. Enter username and password
3. Save and test connection

### Auto Power-Off Not Working
1. Verify plug is linked to a printer
2. Check Enabled, Auto On, Auto Off toggles
3. For temperature mode: ensure printer is still connected
4. Check logs for error messages

### Power Monitoring Not Showing
- Not all plugs support power monitoring
- Requires Tasmota with energy module
- Check Tasmota console for power readings

---

## Best Practices

### Safety First
- Always use temperature-based delay for power-off
- Set a scheduled power-off time as a safety net
- Don't leave printers unattended for extended periods

### Network Reliability
- Use static IP or DHCP reservation for plugs
- Ensure strong WiFi signal at plug location
- Consider wired connection if reliability is critical

### Printer Compatibility
- Ensure your printer handles power cycling gracefully
- Some printers need manual intervention after power-on
- Test the automation flow before relying on it

---

## Tips

- Use power monitoring to track electricity costs
- Set up power alerts to detect print failures
- Combine with print scheduling for full automation
- Schedule power-off at night for safety and savings
