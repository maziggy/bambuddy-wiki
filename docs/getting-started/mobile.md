---
title: Mobile Access
description: Access Bambuddy from your phone or tablet
---

# Mobile Access

Bambuddy is fully responsive and works great on mobile devices. Access your print farm from anywhere on your network.

---

## :material-cellphone: Progressive Web App (PWA)

Bambuddy can be installed as a PWA for a native app-like experience — full screen, home screen icon, and no browser UI. However, PWA support varies significantly between browsers and platforms.

### :material-lock: HTTPS Requirement

!!! warning "Chromium Browsers Require HTTPS"
    **Chrome, Edge, and Samsung Internet** require a secure connection (HTTPS) to install a PWA. If you access Bambuddy over plain HTTP (e.g., `http://192.168.1.100:8000`), the install option **will not appear** in these browsers.

    **Safari on iOS** does **not** have this restriction — Add to Home Screen works over plain HTTP.

| Connection | Safari (iOS) | Chrome / Edge | Firefox |
|:-----------|:------------:|:-------------:|:-------:|
| `https://bambuddy.local` | :material-check: Works | :material-check: Works | No PWA support |
| `http://192.168.1.100:8000` | :material-check: Works | :material-close: Blocked | No PWA support |
| `http://localhost:8000` | :material-check: Works | :material-check: Works | No PWA support |

### :material-apple: Installing on iOS (Safari)

Safari does not show an install prompt — you add the app manually:

1. Open Bambuddy in **Safari** (other iOS browsers don't support PWA)
2. Tap the **Share** button :material-share:
3. Scroll down and tap **Add to Home Screen**
4. Tap **Add** to confirm

!!! note "Safari Only"
    On iOS, only Safari supports PWA installation. Chrome, Firefox, and other browsers on iOS cannot install PWAs to the home screen.

### :material-android: Installing on Android (Chrome / Edge)

1. Open Bambuddy in Chrome or Edge **over HTTPS**
2. Tap the **menu** button :material-dots-vertical:
3. Tap **Install app** or **Add to Home Screen**
4. Tap **Install** to confirm

If the install option does not appear, check that:

- You are using **HTTPS** (not plain HTTP)
- The page has fully loaded
- You are not in incognito/private mode

### :material-monitor: Installing on Desktop

| Browser | PWA Support |
|:--------|:-----------:|
| Chrome | :material-check: Yes (HTTPS required) |
| Edge | :material-check: Yes (HTTPS required) |
| Firefox | :material-close: No (removed PWA support) |
| Safari (macOS) | :material-close: No |

In Chrome or Edge, look for the install icon (:material-download:) in the address bar, or go to **Menu → Install Bambuddy**.

### :material-shield-check: Setting Up HTTPS for Local Network

If you want PWA installation in Chrome/Edge, you need HTTPS. Here are your options:

=== "Reverse Proxy (Recommended)"

    Use **Caddy** or **nginx** as a reverse proxy with a self-signed or local CA certificate:

    ```
    # Example Caddy config (automatic self-signed cert)
    bambuddy.local:443 {
        reverse_proxy localhost:8000
        tls internal
    }
    ```

    Then access Bambuddy at `https://bambuddy.local`.

=== "Tailscale"

    [Tailscale](https://tailscale.com/) provides automatic HTTPS certificates for devices on your tailnet:

    1. Install Tailscale on your Bambuddy server
    2. Enable [HTTPS certificates](https://tailscale.com/kb/1153/enabling-https)
    3. Access via `https://your-machine.your-tailnet.ts.net:8000`

=== "Chrome Flag (Development Only)"

    For testing purposes only, you can tell Chrome to treat a specific HTTP origin as secure:

    1. Open `chrome://flags/#unsafely-treat-insecure-origin-as-secure`
    2. Add your Bambuddy URL (e.g., `http://192.168.1.100:8000`)
    3. Restart Chrome

    !!! danger "Not Recommended for Production"
        This flag lowers browser security. Use only for testing PWA installation on your own network.

### :material-compare: PWA Feature Comparison

| Feature | Safari (iOS) | Chrome (Android) | Chrome (Desktop) | Edge (Desktop) |
|:--------|:------------:|:----------------:|:-----------------:|:--------------:|
| Home screen icon | :material-check: | :material-check: | :material-check: | :material-check: |
| Full screen (no browser UI) | :material-check: | :material-check: | :material-check: | :material-check: |
| Works over HTTP | :material-check: | :material-close: | :material-close: | :material-close: |
| Automatic install prompt | :material-close: | :material-check: | :material-check: | :material-check: |
| Offline caching | :material-check: | :material-check: | :material-check: | :material-check: |
| Push notifications | iOS 16.4+ | :material-check: | :material-check: | :material-check: |
| Background sync | :material-close: | :material-check: | :material-check: | :material-check: |

!!! tip "Best Experience"
    For the best PWA experience on all platforms, set up HTTPS using one of the methods above. This unlocks PWA installation in Chrome and Edge while also securing your connection.

---

## :material-gesture-tap: Touch-Friendly Interface

The mobile interface is optimized for touch:

<div class="feature-grid" markdown>

<div class="feature-card" markdown>
### :material-menu: Hamburger Navigation
Tap the menu icon to open the navigation drawer on smaller screens.
</div>

<div class="feature-card" markdown>
### :material-gesture-tap-hold: Long Press Menus
Long press on cards to open context menus (like right-click on desktop).
</div>

<div class="feature-card" markdown>
### :material-gesture-swipe: Large Touch Targets
All buttons and interactive elements are at least 44px for easy tapping.
</div>

<div class="feature-card" markdown>
### :material-cellphone-screenshot: Safe Area Support
Works with notched devices (iPhone X+, etc.) with proper safe area insets.
</div>

</div>

---

## :material-monitor-cellphone: Responsive Layouts

### Printers Page

| Desktop | Mobile |
|---------|--------|
| Multi-column grid | Single column cards |
| Side-by-side AMS | Stacked AMS display |
| Full temperature readouts | Compact temp display |

### Archives Page

| Desktop | Mobile |
|---------|--------|
| Card grid with filters | Single column with collapsible filters |
| Right-click context menu | Long press for menu |
| Inline editing | Modal editing |

### Statistics Page

| Desktop | Mobile |
|---------|--------|
| Multi-widget dashboard | Single column widgets |
| Side-by-side charts | Stacked charts |
| Drag to reorder | Simplified ordering |

---

## :material-wifi: Accessing from Your Network

### Local Network Access

Access Bambuddy from any device on your local network:

1. Find your server's IP address (e.g., `192.168.1.100`)
2. Open `http://192.168.1.100:8000` on your phone
3. Bookmark or install as PWA

!!! tip "Finding Your IP"
    On Linux/macOS: `ip addr` or `ifconfig`

    On Windows: `ipconfig`

### Remote Access (Advanced)

For access outside your home network:

=== "VPN (Recommended)"

    Set up a VPN (like WireGuard or Tailscale) to securely access your home network from anywhere. Tailscale also provides [automatic HTTPS certificates](https://tailscale.com/kb/1153/enabling-https) which enables PWA installation in Chrome/Edge.

=== "Reverse Proxy + HTTPS"

    Set up a reverse proxy with HTTPS. See the [Docker guide](docker.md#reverse-proxy-nginx) for Nginx configuration. This also enables PWA installation in Chrome/Edge — see [HTTPS setup](#setting-up-https-for-local-network) above.

!!! warning "Security"
    Never expose Bambuddy directly to the internet without proper authentication and HTTPS. Your printer access codes would be vulnerable!

---

## :material-keyboard: Mobile Shortcuts

While physical keyboard shortcuts don't apply on mobile, Bambuddy provides quick-access buttons:

| Action | How to Access |
|--------|---------------|
| Add printer | Floating action button on Printers page |
| Quick search | Search icon in header |
| Refresh | Pull down to refresh (PWA) |
| Context menu | Long press on cards |

---

## :material-bell-ring: Mobile Notifications

Get push notifications on your phone when prints complete, fail, or encounter errors.

Bambuddy supports:

- **ntfy** - Install the ntfy app and subscribe to your topic
- **Telegram** - Receive messages from your Bambuddy bot
- **Pushover** - Professional push notification app
- **WhatsApp** - Via CallMeBot integration

[:material-arrow-right: Set up notifications](../features/notifications.md)

---

## :material-camera: Camera on Mobile

Watch your prints from anywhere:

1. Tap the camera icon on any printer card
2. Choose **Live** for real-time streaming or **Snapshot** for still images
3. Pinch to zoom on the live feed

!!! note "Bandwidth"
    Live streaming uses more data than snapshots. On cellular, consider using snapshot mode.

---

## :material-battery-alert: Tips for Mobile Use

1. **Use WiFi** - Live streaming and real-time updates work best on WiFi
2. **Enable notifications** - Get alerted without keeping the app open
3. **Install as PWA** - Faster launch and better experience
4. **Bookmark important pages** - Quick access to specific printers or stats

---

## :checkered_flag: Next Steps

<div class="quick-start" markdown>

[:material-bell-ring: **Set Up Notifications**<br><small>Never miss a print event</small>](../features/notifications.md)

[:material-camera: **Camera Streaming**<br><small>Watch prints remotely</small>](../features/camera.md)

[:material-chart-line: **Statistics**<br><small>Track your prints</small>](../features/statistics.md)

</div>
