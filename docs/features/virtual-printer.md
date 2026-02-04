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
- **🌐 Remote Printing**: Print from anywhere via Proxy Mode (see below)

---

## :material-earth: Proxy Mode - Remote Printing

!!! success "NEW FEATURE"
    Proxy Mode enables **remote printing from anywhere in the world** through a secure TLS relay.

![Proxy Mode Architecture](../assets/proxy-mode-diagram.png){ .screenshot }

### What is Proxy Mode?

Unlike the standard virtual printer modes that archive files locally, **Proxy Mode** forwards your print jobs directly to a real Bambu Lab printer. Bambuddy acts as a secure relay between your slicer and your printer.

### How It Works

1. **Select the target printer** (must be in LAN mode)
2. **For cross-network:** select the slicer network interface for SSDP relay
3. **Enable the proxy** - printer appears in slicer discovery via SSDP
4. **Connect** using the printer's access code
5. **Print as normal** - traffic is relayed through Bambuddy
6. **Camera streaming** requires NAT/IP forwarding (see below)

### Key Benefits

| Feature | Description |
|---------|-------------|
| :lock: **End-to-end encryption** | TLS encryption from slicer through Bambuddy to printer |
| :globe_with_meridians: **No cloud dependency** | Your data never touches third-party servers |
| :key: **Uses printer's credentials** | No additional passwords - use your printer's access code |
| :zap: **Full protocol support** | Both FTP (file transfer) and MQTT (control) are proxied |
| :chart_with_upwards_trend: **Connection monitoring** | Real-time status showing active connections |

### Proxy Mode Ports

| Protocol | Bambuddy Listen Port | Printer Port | Purpose |
|----------|---------------------|--------------|---------|
| FTP/FTPS | 990 | 990 | File transfer (3MF uploads) |
| MQTT/TLS | 8883 | 8883 | Printer control & status |

!!! note "Privileged Port"
    Port 990 is a privileged port. For bare metal installs using systemd, add `AmbientCapabilities=CAP_NET_BIND_SERVICE` to your service file. For Docker with host networking, no additional configuration is needed.

### Requirements

**Printer Requirements:**

- Bambu Lab printer in **LAN Mode** (Developer Mode)
- Printer must be accessible from Bambuddy on your local network
- Printer's IP address and access code

**Network Requirements:**

- Bambuddy server accessible from the internet (or your remote network)
- Ports **990** (FTP) and **8883** (MQTT) reachable from the remote slicer
- Static IP or dynamic DNS for your Bambuddy server

**Cross-Network (Dual-Homed) Setup:**

For setups where Bambuddy has interfaces on two networks (e.g., printer on LAN A, slicer on LAN B):

- Select the "Slicer Network Interface" in proxy settings to enable SSDP relay
- Bambuddy will re-broadcast printer SSDP on the slicer's network
- The slicer discovers the printer automatically via SSDP
- Camera streaming requires additional NAT/iptables rules (RTSP port 322)

---

### Setting Up Proxy Mode

#### Step 1: Configure Ports for Your Installation

Choose your installation type below:

=== "Docker with Host Mode (Linux)"

    **No additional port configuration needed!**

    Host mode exposes all ports directly. Just ensure your firewall allows the ports:

    ```bash
    # UFW
    sudo ufw allow 990/tcp comment "Bambuddy Proxy FTP"
    sudo ufw allow 8883/tcp comment "Bambuddy Proxy MQTT"

    # Or firewalld
    sudo firewall-cmd --permanent --add-port=990/tcp
    sudo firewall-cmd --permanent --add-port=8883/tcp
    sudo firewall-cmd --reload
    ```

=== "Docker with Bridge Mode (macOS/Windows)"

    Add port mappings to your `docker-compose.yml`:

    ```yaml
    services:
      bambuddy:
        image: ghcr.io/maziggy/bambuddy:latest
        container_name: bambuddy
        ports:
          - "${PORT:-8000}:8000"   # Web UI
          - "990:990"              # Proxy FTP
          - "8883:8883"            # Proxy MQTT
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

    Then restart the container:

    ```bash
    docker-compose down && docker-compose up -d
    ```

    !!! warning "No SSDP Discovery"
        Bridge mode doesn't support SSDP discovery. You must add the printer manually in your slicer using the IP address.

=== "Native Installation (Linux)"

    For systemd-managed services, add capability for privileged port binding:

    ```ini
    # In your bambuddy.service file
    AmbientCapabilities=CAP_NET_BIND_SERVICE
    ```

    Then open ports in your firewall:

    ```bash
    # UFW (Ubuntu/Debian)
    sudo ufw allow 990/tcp comment "Bambuddy Proxy FTP"
    sudo ufw allow 8883/tcp comment "Bambuddy Proxy MQTT"

    # Or iptables
    sudo iptables -A INPUT -p tcp --dport 990 -j ACCEPT
    sudo iptables -A INPUT -p tcp --dport 8883 -j ACCEPT

    # Make iptables rules persistent (Debian/Ubuntu)
    sudo apt install iptables-persistent
    sudo netfilter-persistent save
    ```

=== "Unraid / Synology / TrueNAS"

    If using **host network mode**, open firewall ports as shown in the Linux Native tab.

    If using **bridge mode**, add port mappings in the container configuration:

    - `990:990` (TCP) - Proxy FTP
    - `8883:8883` (TCP) - Proxy MQTT

#### Step 2: Configure Remote Access (Router Port Forwarding)

To access from outside your home network, forward these ports on your router:

| External Port | Internal Port | Protocol | Destination |
|---------------|---------------|----------|-------------|
| 990 | 990 | TCP | Bambuddy server IP |
| 8883 | 8883 | TCP | Bambuddy server IP |

!!! tip "Alternative: Reverse Proxy or Tunnel"
    Instead of direct port forwarding, you can use:

    - **Cloudflare Tunnel** - Free, secure tunneling without opening ports
    - **Tailscale/WireGuard** - VPN-based access
    - **nginx/Caddy/Traefik** - Reverse proxy with TLS termination

!!! warning "Security Note"
    These ports will be exposed to the internet. The connection is protected by TLS encryption and your printer's access code. For additional security, consider using a reverse proxy with authentication.

#### Step 3: Enable Proxy Mode in Bambuddy

1. Go to **Settings → Virtual Printer**
2. Select **Proxy** mode from the mode options
3. Select your **Target Printer** from the dropdown
4. Click the toggle to **Enable Virtual Printer**
5. Verify the status shows "Running" with the proxy ports

#### Step 5: Configure Your Slicer

In Bambu Studio or OrcaSlicer:

1. Go to **Device** → **Add Printer** → **Add printer manually**
2. Enter your Bambuddy server's **external IP or hostname**
3. Enter your **printer's access code** (not a Bambuddy password)
4. The printer should connect and show as online

#### Step 6: Print!

Select a model, slice it, and click **Print**. The job will be:

1. Sent to Bambuddy over the internet (encrypted)
2. Relayed to your printer on the local network (encrypted)
3. Started on your printer just like a local print

### Proxy Mode Use Cases

| Use Case | Description |
|----------|-------------|
| **Remote Print Farm** | Manage workshop printers from home or office |
| **Traveling Makers** | Start prints while on vacation or business trips |
| **Multi-Location** | Access printers at different sites through one Bambuddy |
| **Team Workshop** | Let team members print without local network access |

### Proxy Mode vs Other Modes

| Feature | Archive | Review | Queue | **Proxy** |
|---------|---------|--------|-------|-----------|
| Files stored locally | ✅ | ✅ | ✅ | ❌ |
| Sends to real printer | ❌ | ❌ | ❌ | ✅ |
| Remote printing | ❌ | ❌ | ❌ | ✅ |
| Requires target printer | ❌ | ❌ | ❌ | ✅ |
| Uses printer's access code | ❌ | ❌ | ❌ | ✅ |

### Proxy Mode Troubleshooting

**Slicer can't connect:**

- Verify port forwarding is configured correctly
- Check that Bambuddy's proxy is running (green status indicator)
- Ensure your firewall allows incoming connections on ports 990 and 8883

**Authentication fails:**

- Use your **printer's access code**, not a Bambuddy password
- The access code is the same one you'd use for LAN mode printing

**Connection drops during transfer:**

- Large files may timeout on slow connections
- Check your internet upload speed

**Printer shows offline in slicer:**

- Verify the target printer is online in Bambuddy
- Check that the printer is in LAN mode
- Restart the proxy by toggling it off and on

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

### Step 2: Append the Bambuddy CA Certificate to Slicer

The slicer's `printer.cer` file contains PEM certificates. You need to **append** the Bambuddy CA certificate to this file.

Open `printer.cer` in a text editor and:

1. **Go to the end of the file**
2. **Paste the entire contents of `bambuddy-ca.crt`** after the last `-----END CERTIFICATE-----`
3. **Save the file**
4. **Fully restart the slicer** (Cmd+Q on macOS, not just close the window)

!!! tip "Keep Original Certificates"
    Appending (rather than replacing) preserves your ability to connect to physical Bambu Lab printers while also enabling the virtual printer.

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

!!! warning "When to Update the Certificate"
    You must update the `printer.cer` file whenever:

    - **First-time setup** - append the Bambuddy CA certificate
    - **New Bambuddy installation** - each install generates a unique CA
    - **Switching Bambuddy hosts** - each host has its own CA (unless you [share the CA](#multiple-bambuddy-hosts))
    - **After slicer updates** - updates may restore the original certificate file, requiring you to append again

### Certificate Persistence

The CA certificate is generated once and persists across Bambuddy restarts. If you switch between Docker and native installations, share the certificate directory to avoid regenerating:

```yaml
volumes:
  - ./virtual_printer/certs:/app/data/virtual_printer/certs
```

### Multiple Bambuddy Hosts

Each Bambuddy installation generates its own unique CA certificate.

!!! warning "One Bambuddy CA at a Time"
    When switching between Bambuddy hosts with different CAs, remove the old Bambuddy CA and append the new one. Having multiple Bambuddy CAs may cause confusion about which host to connect to.

**Option 1: Share the CA (Recommended)**

Copy the `certs/` directory from one host to all others:

```bash
# Copy certs from host1 to host2
scp -r host1:/path/to/data/virtual_printer/certs/ host2:/path/to/data/virtual_printer/
```

Then restart Bambuddy on host2. All hosts will use the same CA, so one certificate in the slicer works for all.

**Option 2: Update Certificate When Switching Hosts**

Each time you switch to a different Bambuddy host:

1. Extract the CA from the new host
2. Open the slicer's `printer.cer` file
3. Remove the old Bambuddy CA (if present) and append the new one
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

**Firewall rules (if using firewalld):**

```bash
sudo firewall-cmd --permanent --add-port=2021/udp  # SSDP
sudo firewall-cmd --permanent --add-port=8883/tcp  # MQTT
sudo firewall-cmd --permanent --add-port=990/tcp   # FTPS
sudo firewall-cmd --permanent --add-port=9990/tcp  # FTPS internal
sudo firewall-cmd --reload
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

**Firewall rules (if using firewalld):**

```bash
sudo firewall-cmd --permanent --add-port=2021/udp  # SSDP
sudo firewall-cmd --permanent --add-port=8883/tcp  # MQTT
sudo firewall-cmd --permanent --add-port=990/tcp   # FTPS
sudo firewall-cmd --permanent --add-port=9990/tcp  # FTPS internal
sudo firewall-cmd --reload
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

1. **Verify Bambuddy CA is in slicer's certificate file**:
   ```bash
   # Check that the Bambuddy CA appears in printer.cer
   grep -A1 "BEGIN CERTIFICATE" "/Applications/BambuStudio.app/Contents/Resources/cert/printer.cer" | tail -4
   ```

2. **Wrong certificate?** If you have multiple Bambuddy hosts or reinstalled, you must update the certificate. See [Step 2](#step-2-append-the-bambuddy-ca-certificate-to-slicer).

3. **Verify certificate fingerprints match**:
   ```bash
   # On Bambuddy server (Docker)
   docker exec bambuddy openssl x509 -in /app/data/virtual_printer/certs/bbl_ca.crt -noout -fingerprint -sha1

   # On Bambuddy server (native)
   openssl x509 -in data/virtual_printer/certs/bbl_ca.crt -noout -fingerprint -sha1
   ```
   Verify this fingerprint appears in one of the certificates in your slicer's `printer.cer`.

4. **Fully restart the slicer** - Cmd+Q on macOS, or End Task on Windows. Just closing the window is not enough.

5. **Regenerate certificates** (last resort):
   ```bash
   # Delete certs and restart Bambuddy
   rm -rf /path/to/data/virtual_printer/certs/
   # Then disable and re-enable virtual printer in UI
   ```
   You'll need to remove the old Bambuddy CA from the slicer and append the new one.

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
