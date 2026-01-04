# Virtual Printer

The Virtual Printer feature allows Bambuddy to emulate a Bambu Lab printer on your network. This enables you to send prints directly from Bambu Studio or Orca Slicer to Bambuddy, even without a physical printer connected.

![Virtual Printer Settings](../assets/settings-virtual-printer.png){ .screenshot }

## Overview

When enabled, Bambuddy creates a virtual printer that:

- Appears automatically in Bambu Studio/Orca Slicer via SSDP discovery
- Accepts print jobs over secure TLS/MQTT connections
- Can queue prints for later or auto-start them on a connected printer
- Works with the same workflow as sending to a real Bambu Lab printer

## Use Cases

- **Print Archiving**: Send prints to Bambuddy for archiving without starting them
- **Queue Building**: Build up a print queue before your printer is available
- **Print Farm Preparation**: Prepare jobs to distribute across multiple printers
- **Remote Slicing**: Slice on one computer and send to Bambuddy running elsewhere

## Configuration

### Enabling the Virtual Printer

1. Go to **Settings** in Bambuddy
2. Scroll to the **Virtual Printer** section
3. Toggle **Enable Virtual Printer** to on
4. Set your preferred **Access Code** (8 digits, like a real Bambu printer)
5. Choose a **Mode**:
   - **Queue**: Prints go to pending uploads for manual review
   - **Auto-start**: Prints automatically start on the default printer

### Settings

| Setting | Description |
|---------|-------------|
| **Enable Virtual Printer** | Turn the virtual printer on/off |
| **Access Code** | 8-digit code for authentication (like LAN mode access code) |
| **Mode** | Queue (pending uploads) or Auto-start (immediate print) |
| **Printer Model** | The printer model to emulate (see below) |

### Printer Model Selection

Choose which Bambu printer model the virtual printer should emulate. This affects how slicers detect and interact with the virtual printer.

| Model Code | Printer |
|------------|---------|
| BL-P001 | X1C (default) |
| BL-P002 | X1 |
| BL-P003 | X1E |
| C11 | P1S |
| C12 | P1P |
| C13 | P2S |
| N2S | A1 |
| N1 | A1 Mini |
| O1D | H2D |
| O1C | H2C |
| O1S | H2S |

!!! note "Model Change"
    Changing the printer model requires disabling and re-enabling the virtual printer. The change takes effect when the virtual printer restarts.

## Adding to Bambu Studio / Orca Slicer

### Automatic Discovery

1. Ensure the virtual printer is enabled in Bambuddy
2. In Bambu Studio/Orca Slicer, go to **Device** tab
3. Click **Refresh** or wait for discovery
4. The virtual printer "Bambuddy" should appear in the device list
5. Click to add it, entering the access code when prompted

### Manual Addition

If automatic discovery doesn't work (e.g., different subnets):

1. In Bambu Studio, go to **Device** → **Add Printer**
2. Select **Add printer by IP**
3. Enter the IP address of your Bambuddy server
4. Enter the access code
5. The printer will be added to your device list

## Sending Prints

Once added, sending prints works exactly like a real printer:

1. Slice your model in Bambu Studio/Orca Slicer
2. Click **Print** or **Send to Printer**
3. Select the Bambuddy virtual printer
4. Click **Send**

### Queue Mode

In queue mode, prints appear in the **Pending Uploads** panel:

- Review the print details and thumbnail
- Send to any connected printer
- Delete unwanted prints
- Prints are archived automatically

### Auto-Start Mode

In auto-start mode:

- Prints are immediately sent to your default printer
- The print starts automatically if the printer is ready
- You can still cancel from the printer or Bambuddy

## Docker Configuration

The virtual printer requires direct network access for SSDP discovery. When running in Docker on Linux, host network mode is required.

### Complete docker-compose.yml

```yaml
services:
  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    build: .
    # Usage:
    #   docker compose up -d          → pulls pre-built image from ghcr.io
    #   docker compose up -d --build  → builds locally from source
    container_name: bambuddy
    #
    # LINUX: Use host mode for printer discovery and camera streaming
    network_mode: host
    #
    # macOS/WINDOWS: Docker Desktop doesn't support host mode.
    # Comment out "network_mode: host" above and uncomment "ports:" below.
    # Note: Printer discovery won't work - add printers manually by IP.
    #ports:
    #  - "8000:8000"
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
      #
      # OPTIONAL: Share virtual printer certs with native installation
      # Uncomment the line below if you switch between Docker and native,
      # so the slicer doesn't need to re-trust certificates each time.
      # - ./virtual_printer:/app/data/virtual_printer
    environment:
      - TZ=Europe/Berlin
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
```

### Key Configuration

| Setting | Purpose |
|---------|---------|
| `network_mode: host` | Required for SSDP discovery (Linux only) |
| `bambuddy_data` volume | Persists database, archives, and TLS certificates |

### macOS / Windows Users

Docker Desktop on macOS and Windows doesn't support host network mode. You'll need to:

1. Comment out `network_mode: host`
2. Uncomment the `ports:` section
3. Add printers manually by IP address (auto-discovery won't work)
4. Virtual printer discovery also won't work from slicers

!!! warning "Certificate Changes"
    If certificates are regenerated (new container without volume), you'll need to remove and re-add the printer in your slicer.

## Troubleshooting

### Printer Not Appearing in Slicer

1. **Check virtual printer is enabled** in Bambuddy settings
2. **Verify network connectivity** - slicer and Bambuddy must be on the same network
3. **Check firewall rules** - ports 1990 (SSDP), 8883 (MQTT), and 9990 (FTPS) must be open
4. **Try manual addition** using the IP address

### Connection Refused / Error -1

1. **Verify access code** matches in both Bambuddy and slicer
2. **Check certificate trust** - remove and re-add the printer in slicer
3. **Docker users**: Ensure `network_mode: host` is set

### Authentication Failed

1. The access code in slicer must match Bambuddy settings
2. Try removing the printer from slicer and re-adding it
3. Check Bambuddy logs for authentication errors

### Prints Not Appearing

1. **Queue mode**: Check the Pending Uploads panel
2. **Auto-start mode**: Check if the default printer is configured and online
3. Review Bambuddy logs for any upload errors

## Technical Details

### Protocols Used

| Protocol | Port | Purpose |
|----------|------|---------|
| SSDP | 1990 (UDP) | Printer discovery |
| MQTT/TLS | 8883 | Command and status communication |
| FTPS | 9990 | File transfer (3MF uploads) |

### Security

- All connections use TLS 1.3 encryption
- Self-signed certificates are auto-generated
- Access code authentication required for all connections
- Certificates are stored in `virtual_printer/certs/`

### Limitations

- Only one virtual printer instance per Bambuddy installation
- Requires `network_mode: host` in Docker for discovery
- Slicer must trust the self-signed certificate (added on first connection)
