# Virtual Printer

The Virtual Printer feature allows Bambuddy to emulate a Bambu Lab printer on your network. This enables you to send prints directly from Bambu Studio or Orca Slicer to Bambuddy, even without a physical printer connected.

![Virtual Printer Settings](../assets/settings-virtual-printer.png){ .screenshot }

## Overview

When enabled, Bambuddy creates a virtual printer that:

- Appears automatically in Bambu Studio/Orca Slicer via SSDP discovery
- Accepts print jobs over secure TLS/MQTT connections
- Archives prints directly or queues them for review
- Works with the same workflow as sending to a real Bambu Lab printer

## Use Cases

- **Print Archiving**: Send prints to Bambuddy for archiving without starting them
- **Queue Building**: Build up a print queue before your printer is available
- **Print Farm Preparation**: Prepare jobs to distribute across multiple printers
- **Remote Slicing**: Slice on one computer and send to Bambuddy running elsewhere

---

## :warning: Setup Required

!!! danger "Important: Read Before Enabling"
    The virtual printer feature **requires additional system configuration** before it will work.
    Simply enabling it in the UI is not enough!

The virtual printer uses privileged port 990 (FTPS) which requires special handling on most systems.

### Required Ports

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| SSDP | 2021 | UDP | Printer discovery |
| MQTT | 8883 | TCP/TLS | Printer communication |
| FTPS | 990 | TCP/TLS | File transfer |

---

## Certificate Installation

!!! danger "Required Step"
    The virtual printer uses TLS encryption with a self-signed CA certificate.
    Bambu Studio and OrcaSlicer **do not use the system certificate store** - you must add the certificate directly to the slicer's certificate file.

### Step 1: Locate the CA Certificate

The Bambuddy CA certificate is at:

- **Native install**: `data/virtual_printer/certs/bbl_ca.crt`
- **Docker**: Extract with `docker cp bambuddy:/app/data/virtual_printer/certs/bbl_ca.crt ./bambuddy-ca.crt`

!!! note "Certificate Generation"
    The certificate is only generated when you first enable the virtual printer in the UI. If the file doesn't exist, enable the virtual printer first.

### Step 2: Replace the Last Certificate in Slicer

!!! danger "REPLACE, Don't Add"
    The slicer's `printer.cer` file contains multiple certificates. You must **replace the last certificate** with the Bambuddy CA. Do NOT simply append - having multiple certificates causes connection failures!

The `printer.cer` file is a chain of PEM certificates. Open it in a text editor and:

1. **Find the last certificate block** - scroll to the end of the file
2. **Delete everything from the last `-----BEGIN CERTIFICATE-----` to `-----END CERTIFICATE-----`**
3. **Paste the contents of `bambuddy-ca.crt` in its place**
4. **Save the file**
5. **Fully restart the slicer** (Cmd+Q on macOS, not just close the window)

**Certificate file locations:**

=== "macOS"
    - **Bambu Studio:** `/Applications/BambuStudio.app/Contents/Resources/cert/printer.cer`
    - **OrcaSlicer:** `/Applications/OrcaSlicer.app/Contents/Resources/cert/printer.cer`

=== "Windows"
    - **Bambu Studio:** `C:\Program Files\Bambu Studio\resources\cert\printer.cer`
    - **OrcaSlicer:** `C:\Program Files\OrcaSlicer\resources\cert\printer.cer`

=== "Linux"
    - **Bambu Studio:** `~/.local/share/BambuStudio/resources/cert/printer.cer`
    - **OrcaSlicer:** `~/.local/share/OrcaSlicer/resources/cert/printer.cer`

!!! warning "When to Replace the Certificate"
    You must replace the certificate whenever:

    - **First-time setup** - replace the original last cert with Bambuddy CA
    - **New Bambuddy installation** - each install generates a unique CA
    - **Switching Bambuddy hosts** - each host has its own CA (unless you [share the CA](#multiple-bambuddy-hosts))
    - **After slicer updates** - updates may restore the original certificate file

### Certificate Persistence

The CA certificate is generated once and persists across Bambuddy restarts. If you switch between Docker and native installations, share the certificate directory to avoid regenerating:

```yaml
volumes:
  - ./virtual_printer/certs:/app/data/virtual_printer/certs
```

### Multiple Bambuddy Hosts

Each Bambuddy installation generates its own unique CA certificate.

!!! danger "One CA Only"
    The slicer can only work with ONE Bambuddy CA. Each time you switch hosts, you must replace the certificate in the slicer.

**Option 1: Share the CA (Recommended)**

Copy the `certs/` directory from one host to all others:

```bash
# Copy certs from host1 to host2
scp -r host1:/path/to/data/virtual_printer/certs/ host2:/path/to/data/virtual_printer/
```

Then restart Bambuddy on host2. All hosts will use the same CA, so one certificate in the slicer works for all.

**Option 2: Replace Certificate When Switching Hosts**

Each time you switch to a different Bambuddy host:

1. Extract the CA from the new host
2. Open the slicer's `printer.cer` file
3. Replace the last certificate block with the new CA
4. Fully restart the slicer

---

## Platform Setup

Choose your platform below for specific setup instructions.

### Linux (Native Installation)

Port 990 is a privileged port. You need iptables rules to redirect it:

```bash
# Redirect incoming traffic on port 990 to 9990
sudo iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990

# Redirect localhost traffic on port 990 to 9990
sudo iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990
```

**Make rules persistent:**

=== "Debian/Ubuntu"
    ```bash
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

=== "Fedora/RHEL"
    ```bash
    sudo dnf install iptables-services
    sudo service iptables save
    ```

=== "Arch Linux"
    ```bash
    sudo iptables-save > /etc/iptables/iptables.rules
    sudo systemctl enable iptables
    ```

**Firewall rules (if using UFW):**

```bash
sudo ufw allow 2021/udp  # SSDP
sudo ufw allow 8883/tcp  # MQTT
sudo ufw allow 990/tcp   # FTPS
sudo ufw allow 9990/tcp  # FTPS internal
```

---

### Docker (Linux)

Docker requires host networking for SSDP discovery to work.

**docker-compose.yml:**

```yaml
services:
  bambuddy:
    image: ghcr.io/maziggy/bambuddy:latest
    container_name: bambuddy
    network_mode: host  # Required for SSDP discovery
    volumes:
      - bambuddy_data:/app/data
      - bambuddy_logs:/app/logs
    environment:
      - TZ=Europe/Berlin
    restart: unless-stopped

volumes:
  bambuddy_data:
  bambuddy_logs:
```

**You still need iptables rules on the host:**

```bash
sudo iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990
sudo iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990
```

**Make rules persistent (Debian/Ubuntu):**

```bash
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

**Firewall rules (if using UFW):**

```bash
sudo ufw allow 2021/udp  # SSDP
sudo ufw allow 8883/tcp  # MQTT
sudo ufw allow 990/tcp   # FTPS
sudo ufw allow 9990/tcp  # FTPS internal
```

---

### Docker (macOS / Windows)

!!! warning "Limited Support"
    Docker Desktop on macOS and Windows doesn't support host network mode.
    SSDP discovery **will not work** - you must add the printer manually by IP.

1. Use bridge networking with port mapping
2. Add the printer manually in your slicer using the host IP address
3. Virtual printer discovery from slicers won't work

---

### Unraid

1. Set **Network Type** to `host` in container settings

2. Add iptables rules via Unraid terminal:
   ```bash
   iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990
   iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990
   ```

3. Make rules persistent by adding to `/boot/config/go`:
   ```bash
   echo "iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990" >> /boot/config/go
   echo "iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990" >> /boot/config/go
   ```

---

### Synology NAS

1. Use **Host Network** in Container Manager

2. SSH into your NAS and add iptables rules:
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990
   ```

3. Create a scheduled task to restore rules on boot:
   - Control Panel → Task Scheduler → Create → Triggered Task → User-defined script
   - Event: Boot-up
   - Script: `iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990`

---

### TrueNAS SCALE

1. Use **Host Network** when creating the app/container

2. Add iptables rules via shell:
   ```bash
   iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990
   iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990
   ```

---

### Proxmox LXC

1. Enable **nesting** in container options (for iptables support)

2. Inside the LXC:
   ```bash
   apt install iptables
   iptables -t nat -A PREROUTING -p tcp --dport 990 -j REDIRECT --to-port 9990
   iptables -t nat -A OUTPUT -o lo -p tcp --dport 990 -j REDIRECT --to-port 9990

   # Make persistent
   apt install iptables-persistent
   netfilter-persistent save
   ```

---

## Configuration

### Enabling the Virtual Printer

1. Complete the platform setup above first
2. Go to **Settings** in Bambuddy
3. Scroll to the **Virtual Printer** section
4. Set an **Access Code** (exactly 8 characters)
5. Toggle **Enable Virtual Printer** to on
6. Choose your **Archive Mode**:
   - **Immediate**: Files are archived automatically when received
   - **Queue for Review**: Files go to pending uploads for manual review

### Printer Model Selection

Choose which Bambu printer model the virtual printer should emulate:

| SSDP Code | Printer | Serial Prefix |
|-----------|---------|---------------|
| 3DPrinter-X1-Carbon | X1C | 00M |
| 3DPrinter-X1 | X1 | 00M |
| C13 | X1E | 03W |
| C11 | P1P | 01S |
| C12 | P1S | 01P |
| N7 | P2S | 22E |
| N2S | A1 | 039 |
| N1 | A1 Mini | 030 |
| O1D | H2D | 094 |
| O1S | H2S | 094 |

!!! note "Model Change"
    Changing the printer model will automatically restart the virtual printer. You may need to refresh the device list in your slicer.

---

## Adding to Bambu Studio / Orca Slicer

### Automatic Discovery

1. Ensure the virtual printer is enabled and running
2. In Bambu Studio/Orca Slicer, go to **Device** tab
3. Click **Refresh** or wait for discovery
4. The virtual printer "Bambuddy" should appear
5. Click to add it, entering the access code when prompted

### Manual Addition

If automatic discovery doesn't work:

1. In Bambu Studio, go to **Device** → **Add Printer**
2. Select **Add printer by IP**
3. Enter the IP address of your Bambuddy server
4. Enter the access code
5. The printer will be added to your device list

---

## Sending Prints to Bambuddy

!!! warning "Use Send, Not Print"
    You must use the **Send** button, not the **Print** button!

    - **Send** → Transfers the file to Bambuddy (correct)
    - **Print** → Attempts to start printing immediately (won't work)

### How to Send a Print

1. Slice your model as usual
2. Select "Bambuddy" (or your virtual printer name) from the printer dropdown
3. Click the **Send** button (next to the Print button)
4. The file will be transferred to Bambuddy

![Send Button](../assets/slicer-send-button.png){ .screenshot }

### What Happens Next

Depending on your **Archive Mode** setting:

- **Immediate**: The file is automatically archived
- **Queue for Review**: The file appears in **Pending Uploads** for you to review, assign to a project, add notes, or queue for printing

!!! tip "Send Button Location"
    In Bambu Studio/OrcaSlicer, the Send button is typically a small icon next to the large Print button, or accessible via the dropdown arrow on the Print button.

---

## Troubleshooting

### Printer Not Appearing in Slicer

1. **Check virtual printer is enabled** and showing "Running" status
2. **Verify iptables rules are active:**
   ```bash
   sudo iptables -t nat -L -n | grep 990
   ```
3. **Check firewall** - ports 2021/udp, 8883/tcp, 990/tcp must be open
4. **Same network** - slicer and Bambuddy must be on the same subnet

### FTP Error / Connection Reset

1. **Verify iptables rules** are correctly configured
2. **Check permissions** on the uploads directory
3. **Check no other FTP server** is using port 990
4. **Review logs** for specific error messages

### "Wrong Printer Model" Error

The slicer's selected printer profile must match the virtual printer model in Bambuddy settings.

### Permission Denied Errors

If you see "Permission denied" in the logs:

```bash
# Fix ownership of the virtual printer directory
sudo chown -R $(whoami):$(whoami) /path/to/bambuddy/data/virtual_printer
```

### Authentication Failed

1. Verify access code matches in both Bambuddy and slicer
2. Access code must be exactly 8 characters
3. Try removing and re-adding the printer in your slicer

### TLS Connection Failed / Error -1

This typically means the slicer doesn't trust the virtual printer's certificate:

1. **Verify certificate is correct in slicer**:
   ```bash
   # Check the last certificate in printer.cer - should be "Virtual Printer CA"
   tail -20 "/Applications/BambuStudio.app/Contents/Resources/cert/printer.cer"
   ```

2. **Wrong certificate?** If you have multiple Bambuddy hosts or reinstalled, you must **replace** the certificate. See [Step 2](#step-2-replace-the-last-certificate-in-slicer).

3. **Verify certificate fingerprints match**:
   ```bash
   # On Bambuddy server (Docker)
   docker exec bambuddy openssl x509 -in /app/data/virtual_printer/certs/bbl_ca.crt -noout -fingerprint -sha1

   # On Bambuddy server (native)
   openssl x509 -in data/virtual_printer/certs/bbl_ca.crt -noout -fingerprint -sha1
   ```
   Compare with the fingerprint of the last cert in your slicer's `printer.cer`.

4. **Fully restart the slicer** - Cmd+Q on macOS, or End Task on Windows. Just closing the window is not enough.

5. **Regenerate certificates** (last resort):
   ```bash
   # Delete certs and restart Bambuddy
   rm -rf /path/to/data/virtual_printer/certs/
   # Then disable and re-enable virtual printer in UI
   ```
   You'll need to replace the certificate in the slicer again.

---

## Technical Details

### Security

- All connections use TLS encryption
- Self-signed certificates are auto-generated
- Access code authentication required for all connections
- Certificates are stored in `data/virtual_printer/certs/`

### Limitations

- Only one virtual printer instance per Bambuddy installation
- Requires `network_mode: host` in Docker for discovery
- Slicer must trust the self-signed certificate on first connection
- macOS/Windows Docker: No auto-discovery support
