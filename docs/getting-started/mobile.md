---
title: Mobile Access
description: Access Bambuddy from your phone or tablet
---

# Mobile Access

Bambuddy is fully responsive and works great on mobile devices. Access your print farm from anywhere on your network.

---

## :material-cellphone: Progressive Web App (PWA)

Bambuddy can be installed as a PWA on your phone for a native app-like experience.

### Installing on iOS

1. Open Bambuddy in Safari
2. Tap the **Share** button :material-share:
3. Scroll down and tap **Add to Home Screen**
4. Tap **Add** to confirm

### Installing on Android

1. Open Bambuddy in Chrome
2. Tap the **menu** button :material-dots-vertical:
3. Tap **Install app** or **Add to Home Screen**
4. Tap **Install** to confirm

!!! tip "Full Screen Experience"
    Once installed as a PWA, Bambuddy opens in full screen without browser UI, just like a native app!

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

    Set up a VPN (like WireGuard or Tailscale) to securely access your home network from anywhere.

=== "Reverse Proxy + HTTPS"

    Set up a reverse proxy with HTTPS. See the [Docker guide](docker.md#reverse-proxy-nginx) for Nginx configuration.

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
