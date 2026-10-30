---
title: Camera Streaming
description: Watch your prints in real-time with MJPEG streaming
---

# Camera Streaming

Monitor your prints visually with live camera streaming directly from your Bambu Lab printer.

---

## :material-video: Live Streaming

Bambuddy provides MJPEG video streaming from your printer's built-in camera, or from an external network camera.

---

## :material-webcam: External Cameras

Connect external network cameras to replace the built-in printer camera. Useful for better angles, higher resolution, or printers in enclosures.

### Supported Camera Types

| Type | Description | Example URL/Path |
|------|-------------|------------------|
| **MJPEG** | Motion JPEG stream | `http://192.168.1.50/mjpeg` |
| **RTSP** | Real-Time Streaming Protocol | `rtsp://192.168.1.50:554/stream` |
| **Snapshot** | HTTP URL returning JPEG image | `http://192.168.1.50/snapshot.jpg` |
| **USB (V4L2)** | USB webcam connected to host | `/dev/video0` |

### Configuration

1. Go to **Settings** > **Camera**
2. Under **External Cameras**, click **Add external camera**, then search for the printer by name or location and pick it. Its camera fields open
3. Enter the camera URL. For a USB camera, pick the device from the list instead (see **USB Camera Setup** below)
4. Select the camera type, and the rotation if the camera is mounted turned
5. Click **Test** to verify connection

The list shows only printers that have an external camera, so it stays short however many printers you run. A closed row shows the camera type and where it points, such as `MJPEG · 192.168.1.61:1984`; click the row to open its fields, and only one printer is open at a time. The bin icon at the end of a row switches that printer back to its built-in camera.

!!! tip "RTSP Authentication"
    Include credentials in the URL: `rtsp://user:password@192.168.1.50:554/stream`
    Only users who may edit printers see the password; everyone else sees the URL with `[REDACTED]` in its place.

!!! note "Redirects"
    Bambuddy does not follow an RTSP camera that redirects to another address: the camera test fails and the server log names the address it was sent to (`RTSP camera redirected to ...`). Enter that address as the camera URL instead. MJPEG and snapshot cameras may redirect to another address the camera URL itself would be allowed to use, but never to the Bambuddy host or a cloud metadata address.

!!! tip "USB Camera Setup"
    With the camera type set to **USB Camera (V4L2)**, the **USB device** list shows the video devices the Bambuddy server can see, such as `HD Pro Webcam C920 (/dev/video0)`; pick the camera from it. Click the refresh button next to the list after plugging a camera in. A saved camera that is unplugged stays selected and shows as `/dev/video0 (not found)`.

    Device names come from `v4l2-ctl`; without it the list shows `video0` and so on. Install it with `sudo apt install v4l-utils`.

    **Enter path manually** switches to a text field for the device path (e.g. `/dev/video0`); **Choose a detected device** goes back to the list. Bambuddy streams from `/dev/video0` to `/dev/video99` only.

    In Docker, the container sees only the devices passed to it. If the list says no video devices are visible, add the camera to your `docker-compose.yml` and recreate the container:

    ```yaml
    services:
      bambuddy:
        devices:
          - /dev/video0:/dev/video0
    ```

!!! note "Several viewers, one connection"
    Bambuddy opens an external camera once per printer, however many people watch it: the Camera page, Cam Wall tiles, embedded viewers and stream overlays all share that one connection. A USB camera can only be read by one program at a time, so a second viewer used to fail with a blank picture. The connection closes a few seconds after the last viewer leaves; a page reload within that time reuses it. The viewer who opens the connection sets its frame rate for everyone: a Cam Wall tile opens it at 8 frames per second, the Camera page at 15.

!!! tip "Snapshot URL override (go2rtc, IP cameras with `/frame.jpeg`-style endpoints)"
    For **MJPEG**, **RTSP** and **USB** stream types, you can optionally provide a separate **Snapshot URL** below the live-stream URL (hover the info icon next to it for a reminder of what it does). When set, Bambuddy fetches single-frame captures (notification thumbnails, finish photos, timelapse, plate detection) from this URL via plain HTTP GET instead of opening the live stream.

    Useful when:

    - You're using **go2rtc** as a relay for a flaky USB / IP camera. go2rtc exposes both `/api/stream.mjpeg?src=<name>` (live) and `/api/frame.jpeg?src=<name>` (single frame). The frame endpoint reliably returns a clean image; the stream endpoint can emit a stale/black warm-up frame on every fresh connection.
    - Your IP camera ships a dedicated snapshot endpoint that's faster and more consistent than reading from the MJPEG stream.

    Click **Test** next to the snapshot URL to verify it returns a valid frame.

    Leave blank to use the default behaviour: capture from the live stream (with automatic warm-up-frame skip).

### Features with External Cameras

| Feature | Description |
|---------|-------------|
| **Live Streaming** | Replaces built-in camera in the viewer |
| **Finish Photos** | Captures from external camera on print complete |
| **Layer Timelapse** | Captures a frame on each layer change, stitches to video on completion |

### Layer-Based Timelapse

!!! note "External Cameras Only"
    Layer-based timelapse only works with external cameras (MJPEG, RTSP, Snapshot, or USB). Built-in printer cameras use the printer's own timelapse feature instead.

When an external camera is enabled, Bambuddy automatically:

1. **Captures a frame** each time the print layer changes
2. **Stores frames** in a temporary directory during printing
3. **Stitches video** using ffmpeg when print completes
4. **Attaches** the timelapse to the print archive

!!! note "With the printer's own timelapse on too"
    The archive keeps one video, and it is the external camera's: it replaces the printer's video, which usually arrives first, and the printer's file is deleted. A video stitched for a print whose archive is already printing again (the next plate of a shared archive, or a reprint) is not attached to the new print.

!!! note "ffmpeg Required"
    Layer timelapse requires ffmpeg to be installed (included in Docker image).

    The video is at most 1080p: frames from a larger camera are scaled down, keeping their shape, so a 4K camera gives a 1920 × 1080 video. The encoder is limited to two threads for the stitch, which keeps a 1080p stitch at about 300 MiB of memory. That matters in Docker or Kubernetes, where ffmpeg counts against Bambuddy's own memory limit.

!!! note "Long prints and slow hardware"
    Stitching may take half a second per layer, from 5 minutes up to an hour, so a print with thousands of layers on a Raspberry Pi still gets its video. If ffmpeg fails anyway, or Bambuddy stops while it runs, the frames are not deleted: they stay for about 24 hours in `timelapse_frames/<printer id>/<session>/` under Bambuddy's data folder (`DATA_DIR`, or the install folder when it is not set), and when ffmpeg fails the log names the folder. The folder's `STITCH_FAILED` file holds the ffmpeg command to make the video by hand. Without ffmpeg installed, the frames are deleted as before.

### Opening the Camera

1. Click the :material-camera: camera icon on any printer card
2. The camera opens in whichever view mode you used last (see below)
3. The stream starts automatically

#### From any page

The :material-camera: **Cameras** button opens a printer's camera as an
embedded overlay without going back to the Printers page. It sits at the end
of the **Printers** row in the sidebar (in its corner, on hover, when the
sidebar is collapsed) and, on narrow screens, in the header beside the
bug-report button. The **C** key opens it too. It lists the printers you can see. A printer whose camera is
already open is ticked; pick it again to close that camera. The button only
appears for users with the `camera:view` and `printers:read` permissions;
without both, no camera overlay is shown either.

### View Mode

The camera button is a split control. The icon opens the camera the way you
opened the last one; the caret beside it lets you choose:

| Mode | Description |
|------|-------------|
| **New Window** | Opens camera in a separate browser window (default) |
| **Embedded Overlay** | Shows camera as a floating overlay on the main screen |

Picking a mode from the menu opens the camera that way *and* makes it the mode
the icon uses from then on, so switching costs one click rather than a trip to
the settings page. A tick marks the mode currently in effect.

The choice is remembered in your own browser, so two people watching the same
printers can each use the view they prefer. If you have permission to change
settings, it is also saved as the default that a browser which has never chosen
starts from - a new machine or a fresh profile opens cameras the same way.

Camera tiles on the [Cam Wall](#cam-wall-on-a-tv-or-kiosk) follow the same
choice, since a tile has no room for a menu of its own.

!!! note "Moved in 1.2.6"
    This replaces the **Camera View Mode** dropdown that used to live in
    **Settings** > **General** > **Camera**. Your existing choice carries over
    as the starting default.

#### Embedded Viewer Features

When using embedded mode, the camera appears as a floating window:

- **Draggable**: Click and drag the header to reposition
- **Resizable**: Drag the bottom-right corner to resize
- **Persistent**: Position and size are remembered per printer across sessions
- **On every page**: Open cameras stay open as you move between pages, and reopen after a reload
- **Minimize**: Click the minimize button to collapse to title bar; the stream stops until you expand it again
- **Pop out**: :material-open-in-new: opens the camera in its own window and closes the overlay. If the browser blocks the popup, the overlay stays.
- **Close**: Click X to close the viewer
- **Multi-Viewer**: Open cameras for multiple printers simultaneously

A camera window has a :material-picture-in-picture-bottom-right: **Show as an
overlay in Bambuddy** button that does the reverse: the camera opens as an
overlay in the Bambuddy tab the window was opened from, or in another open
Bambuddy tab if that one is gone or is signed in as someone who cannot see
the printer, and the window closes. If no such tab is open, the window stays
and says so. A camera page you reached any other way (a bookmark, a link, a
tab you opened yourself) cannot be closed by the page: it goes back to the
Bambuddy page before it instead, or to the start page, and has a :material-arrow-left: **Back** button next to the
printer's name for the same reason.

!!! note "iOS home-screen web app"
    Bambuddy added to the home screen ("Add to Home Screen" on iOS) has no
    second window to open: iOS would load the camera into the app itself, full
    screen. There, cameras always open as the embedded overlay, the camera
    button offers no window mode, and a Cam Wall tile opens the camera page in
    place, with a **Back** button to return. Bambuddy installed as an app on
    a desktop or Android keeps both modes: a window opens normally there.

!!! tip "Embedded Mode Benefits"
    Embedded mode keeps you on the main screen while monitoring prints - no need to switch between browser windows. Open multiple viewers to monitor your entire print farm at once.

### Stream Controls

| Button | Action |
|:------:|--------|
| **Live** | Real-time MJPEG video stream |
| **Snapshot** | Single still image (lower bandwidth) |
| :material-refresh: | Restart the stream |
| :material-fullscreen: | Enter fullscreen mode |

---

## :material-magnify: Zoom & Pan

Zoom in on your camera feed to inspect print details.

### Zoom Controls

| Method | Action |
|--------|--------|
| **Mouse wheel** | Scroll up to zoom in, down to zoom out |
| **+/- buttons** | Click the zoom buttons in the corner |
| **Zoom indicator** | Click the percentage to reset to 100% |
| **Pinch gesture** | Two-finger pinch on touch devices |

Zoom range: **100% - 400%**

### Pan Controls

When zoomed in (>100%), you can pan around the image:

| Platform | How to Pan |
|----------|------------|
| **Desktop** | Click and drag the image |
| **Mobile** | Drag with one finger when zoomed |
| **During pinch** | Move both fingers while pinching |

The pan range automatically adjusts based on zoom level - at higher zoom, you can pan further to explore the entire image.

!!! tip "Reset Zoom"
    Click the zoom percentage indicator (e.g., "200%") to instantly reset to 100% view.

---

## :material-camera: Snapshot Mode

For lower bandwidth usage, use snapshot mode:

- Captures a single frame on demand
- Click refresh to get a new snapshot
- Ideal for cellular connections or slow networks

---

## :material-cog: Technical Details

### How Streaming Works

```mermaid
graph LR
    A[Printer Camera] -->|RTSP| B[ffmpeg]
    B -->|MJPEG| C[Bambuddy API]
    C -->|HTTP Stream| D[Browser]
```

1. Printer exposes camera via **RTSP** (Real Time Streaming Protocol)
2. **ffmpeg** converts RTSP to MJPEG (Motion JPEG)
3. Bambuddy serves the **MJPEG stream** to your browser
4. Browser displays frames in an `<img>` tag

### Requirements

| Requirement | Details |
|-------------|---------|
| **ffmpeg** | Must be installed on Bambuddy server |
| **Camera enabled** | Must be enabled in printer settings |
| **Developer Mode** | Camera access requires Developer Mode |
| **Network access** | Server must be able to reach printer IP |

!!! tip "Docker Users"
    Camera streaming works with Docker's default bridge networking in most setups (NAT handles routing automatically).

    If you have issues, try `network_mode: host` - see [Docker Installation](../getting-started/docker.md#network-mode-host).

### Installing ffmpeg

=== ":material-ubuntu: Ubuntu/Debian"

    ```bash
    sudo apt install ffmpeg
    ```

=== ":material-apple: macOS"

    ```bash
    brew install ffmpeg
    ```

=== ":material-microsoft-windows: Windows"

    Download from [ffmpeg.org](https://ffmpeg.org/download.html) and add to PATH.

=== ":material-docker: Docker"

    ffmpeg is included in the Docker image.

---

## :material-movie-edit: Timelapse Editor

Edit timelapse videos directly in Bambuddy before downloading or sharing.

### Opening the Editor

1. Go to **Archives** and select a completed print
2. Click on the timelapse video thumbnail
3. Click **Edit** to open the timelapse editor

### Editing Features

| Feature | Description |
|---------|-------------|
| **Trim** | Set start and end points to remove unwanted footage |
| **Speed** | Adjust playback speed from 0.25x to 4x |
| **Music** | Add your own audio file (MP3, WAV, M4A, AAC, OGG) with a volume slider |
| **Preview** | Watch your edits before saving |

### Saving

1. Make your edits
2. Click **Save**: the edited video replaces the original timelapse
3. The **Save** button shows the progress; **Cancel** next to it stops processing and keeps the original

Processing runs on the server with FFmpeg and carries on if you close the editor. See [Timelapse Editor](archiving.md#timelapse-editor) for details.

---

## :material-image-area: Camera Snapshots on Print Complete

Bambuddy can automatically capture a camera snapshot when prints complete:

1. Go to **Settings** > **General**
2. Enable **Capture snapshot on print complete**
3. Snapshots are saved to the archive

This creates a visual record of your completed prints!

!!! note "Photo is captured before the bed drops"
    On most Bambu printers the end-gcode lowers the build plate as soon as the print finishes, which used to leave the snapshot framed on an almost-empty chamber. Bambuddy now sources the photo from a brief timelapse the printer records during the print — the firmware stops recording after the toolhead parks but *before* the bed drops, so the photo frames the finished print correctly. If you enabled timelapse for the print job the file is kept; otherwise Bambuddy deletes the timelapse from the printer's SD card automatically once the photo is captured.

    This currently applies to prints **dispatched through Bambuddy** (queue, reprint, print-now from File Manager). Prints started directly on the printer's touchscreen, via the Bambu Handy app, or via Bambu Studio's "Send" function fall back to a live camera grab at completion — so the bed-down framing limitation still applies in that case. Track this in [#1397](https://github.com/maziggy/bambuddy/issues/1397).

---

## :material-lightbulb-on: Chamber Light for the Camera

If you keep the chamber light off between uses, the live view is black and the photos in your notifications are too. Bambuddy can switch the light on whenever the camera is used:

1. Go to **Settings** > **Camera**
2. Under **Turn on chamber light for camera**, choose:
    - **All printers**, so every printer does it, or
    - **Selected printers**, then click **Add printer**, search for a printer by name or location and pick it. Each picked printer shows as a chip; click its **×** to take it off the list.
3. Optionally set **Wait before snapshot** (2 seconds by default, up to 5). It applies to every printer.

**Off** is the default, so no printer's light changes until you choose otherwise.

The light then turns on:

- while anyone watches the live view, including the camera wall and a [streaming overlay](#streaming-overlay-for-obs)
- for automatic pictures: notification photos, the live finish photo, and the snapshot that Home Assistant and other automations fetch from `/camera/snapshot`

It turns off again 15 seconds after the last use, so a viewer that reconnects or the camera wall's refresh doesn't make it flash.

The [plate check](#build-plate-empty-detection) at print start lights the chamber on every printer, whatever this setting says, because detection needs a lit plate, and switches it back off afterwards. On a printer this setting covers, it shares the light with the uses above: a live view opened during the check keeps it on.

**Wait before snapshot** gives the light, and any room light Home Assistant syncs to it, time to come on before an automatic picture is taken. It only applies when the light was off; the live view never waits.

!!! note "Bambuddy only turns off a light it turned on"
    If the light was already on, it stays on. If you switch the light by hand on the printer card while the camera is in use, the light is yours again and stays as you set it.

!!! note "What doesn't switch the light"
    The layer timelapse, the Obico failure check and the frames Bambuddy keeps during a print for the finish photo capture all through a print, so the light would flash with every frame. They use whatever light the printer has. A finish photo taken from the printer's own timelapse video was filmed during the print, so it isn't lit either.

---

## :material-scan-helper: Build Plate Empty Detection

Automatically detect if objects are left on the build plate before a print starts. If detected, the print is immediately paused and you receive a notification.

### How It Works

1. **Calibrate** - Capture reference images of your empty build plate
2. **Enable** - Toggle plate detection on for the printer
3. **Auto-Check** - When any print starts, Bambuddy compares the current camera view to your references
4. **Auto-Pause** - If objects are detected, the print is immediately paused

### Calibration

Store up to **5 reference images** per printer for different plate types (textured, smooth, high-temp, etc.):

1. Click the **scan icon** on the printer card to open the modal
2. Ensure the build plate is **completely empty** and **chamber light is ON**
3. Click **Calibrate Empty Plate**
4. Optionally add a label (e.g., "Textured PEI", "Cool Plate")
5. Repeat for other plate types you use

!!! tip "Multiple References"
    The system automatically selects the best-matching reference when checking. Calibrate all your commonly used plates for accurate detection.

### Enabling Detection

Use the **split button** on the printer card:

| Button Part | Action |
|-------------|--------|
| **Main (scan icon)** | Toggles detection on/off |
| **Chevron (▼)** | Opens calibration/management modal |

When enabled, the button shows a **green border**.

### ROI (Region of Interest)

Adjust which part of the camera view is analyzed:

1. Open the plate detection modal
2. Scroll to **Detection Area (ROI)**
3. Click **Edit**
4. Use the sliders to adjust X, Y, Width, and Height
5. Click **Save**

The green box in the preview shows the detection area. Focus it on just the build plate to avoid false positives from the printer frame or background.

### Managing References

In the plate detection modal:

- View thumbnails of saved references
- Click label to edit (e.g., "High Temp Plate")
- Click **X** on thumbnail to delete
- See timestamps for when each was calibrated

### Notifications

When objects are detected:

- **Print pauses immediately**
- **Toast notification** appears in Bambuddy
- **Push notification** sent (if configured)
- **WebSocket event** broadcast for integrations

### Requirements

| Requirement | Details |
|-------------|---------|
| **OpenCV** | `opencv-python-headless` must be installed |
| **Chamber Light** | Should be ON for reliable detection |
| **Calibration** | At least one reference image required |

!!! note "Docker"
    OpenCV is included in the Docker image.

### How Detection Works

1. Captures current camera frame (or uses buffered frame if stream is active)
2. Applies heavy Gaussian blur to both current and reference images
3. Normalizes both images for consistent comparison
4. Extracts the ROI region
5. Calculates pixel difference percentage
6. If difference > 1%, plate is considered "not empty"

When the check cannot run (no camera image, an image that cannot be read, an analysis error, or no calibration to compare against), the print is not held, but it is not treated as an empty plate either: users who can control printers get a warning saying why, and the log records it. **Check Plate** in the plate detection modal shows the same reason.

### Troubleshooting

**False Positives (detects objects when empty)**

- Calibrate with chamber light ON (same as during prints)
- Adjust ROI to exclude printer frame edges
- Add multiple calibrations for different lighting conditions

**False Negatives (doesn't detect objects)**

- Ensure chamber light is ON
- Recalibrate - plate surface may have changed
- Check that objects are within the ROI area

---

## :material-tune: Stream Settings

### Frame Rate

The default frame rate is 15 FPS. You can adjust this in the URL:

```
/api/v1/printers/{id}/camera/stream?fps=20
```

| FPS | Use Case |
|-----|----------|
| 5 | Low bandwidth / A1/P1 cameras (hardware limit) |
| 10-15 | Balanced (15 is default) |
| 20-25 | Smoother video |
| 30 | Maximum quality (X1/H2/P2 only) |

!!! note "FPS Limits by Camera Type"
    - **External cameras**: Max 15 FPS
    - **A1/P1 printers**: Max 5 FPS (hardware limitation)
    - **X1/H2/P2 printers**: Max 30 FPS

!!! note "Higher FPS = More Bandwidth"
    Higher frame rates consume more network bandwidth and server resources.

    This applies to the connection between Bambuddy and your browser only. The
    printer always sends its camera stream at full rate; the FPS setting decides
    how many of those frames are passed on to the viewer.

### Shared Streams and FPS

Most Bambu printers accept only **one** camera connection at a time. To let you
watch the same printer in several places at once - a cam-wall tile, the embedded
viewer, a popup window - Bambuddy opens a single stream to the printer and fans
the frames out to every viewer.

One consequence is worth knowing about:

!!! warning "`?fps=` applies to the viewer that opens the stream"
    The frame rate is set by whichever viewer starts the shared stream. Viewers
    that join afterwards receive that same rate, whatever they asked for in
    `?fps=`, because they attach to the stream that is already running.

    In practice this means the same action can give you a different frame rate
    depending on what was already open. Opening the fullscreen camera at 15 FPS
    while a cam-wall tile is already streaming at 8 FPS gives you 8 FPS.

The shared stream is torn down a few seconds after the last viewer leaves. The
next viewer to open the camera starts a fresh stream at its own requested rate,
so closing every viewer and reopening is what puts a new `?fps=` into effect.

!!! tip "Getting a predictable rate"
    If you drive the stream endpoint from your own client and need a specific
    frame rate, make sure no other viewer has the camera open first. `POST
    /api/v1/printers/{id}/camera/stop` tears the shared stream down, but only
    when no viewer is attached - it deliberately refuses while somebody else is
    watching, so that one client cannot cut off another.

    External cameras (RTSP, USB) run one stream per viewer. Pass the same
    `viewer=<id>` on your stream URL and on the stop call: the stop then ends
    your own stream, plus any that has sent no picture for 10 seconds, and
    leaves other viewers' streams running. A stop without `viewer` ends every
    stream of the printer.

---

## :material-connection: Stream Cleanup

Bambuddy properly cleans up camera streams:

- **Window close** - Stream stops automatically
- **Tab hidden** - Stream pauses to save resources
- **Page unload** - ffmpeg process terminated
- **Refresh** - Old stream stopped, new one started

This prevents orphaned ffmpeg processes from consuming resources.

!!! note "USB (V4L2) camera cleanup"
    External **USB** camera streams are tracked the same way as the built-in RTSP path, so a stream that ends abruptly (browser tab/popup closed, dropped connection) is cleaned up two ways: immediately by the explicit stop call when the viewer closes, and — as a fallback if that's missed — by a background janitor that runs about once a minute and kills any leaked `ffmpeg` still holding `/dev/videoN`. If you ever find a USB camera's LED stuck on after closing the view, reopening the view or waiting for the next janitor pass releases the device; it no longer stays locked indefinitely.

    The janitor only touches Bambuddy's own `ffmpeg`: one Bambuddy started, or one left over from an earlier run and since handed to the system's init process (as Docker and a crashed service leave them). Other software on the same machine that reads a USB camera with `ffmpeg` (motion, a webcam server) is left alone.

---

## :material-refresh: Auto-Reconnect

Bambuddy automatically detects and recovers from stalled camera streams:

### Stall Detection

The browser periodically checks if the stream is still receiving frames:

- **Check interval**: Every 5 seconds
- **Detection**: Compares last frame timestamp
- **Threshold**: Stalled if no new frames for >5 seconds

### Automatic Recovery

When a stall is detected:

1. **Detects** that no frames have been received
2. **Closes** the stalled connection
3. **Reconnects** automatically
4. **Resumes** streaming

!!! tip "Network Interruptions"
    If your network briefly drops, the stream will automatically recover once the connection is restored.

### Frozen Picture

Bambuddy also watches what the camera sends, not only that it sends something. A built-in (RTSP) camera that keeps sending the same frame for 20 seconds counts as frozen. This happens when the connection to the printer has dropped but `ffmpeg` keeps repeating its last frame. Bambuddy then restarts `ffmpeg` and reconnects to the printer, and open viewers stay connected. The log shows `RTSP output frozen … restarting ffmpeg` when this happens.

An external RTSP camera is watched the same way; its log line reads `External RTSP picture frozen … restarting ffmpeg`. Some external cameras legitimately repeat their picture for longer, such as a slow source or a surveillance camera's smart codec on a static night scene. If the picture after a restart turns out to be a new one, Bambuddy waits three times the longest gap it has seen between changes (up to 5 minutes) before the next check, so such a camera is restarted at most once.

A working camera practically never sends two identical frames, because sensor noise changes every one. In a completely dark chamber the picture can come out identical, though. If the picture after a restart is exactly the same as before, Bambuddy treats it as a still picture and only checks again every 5 minutes, until it changes.

---

## :material-stethoscope: Built-in Camera Diagnostic

Before opening a ticket, click **Diagnose** in the camera viewer. The diagnostic runs staged checks against your printer's camera path and tells you exactly which stage failed and what to do about it.

### How to open it

Two entry points, both in the embedded camera viewer:

- **Stethoscope icon** in the control bar, between Refresh and Fullscreen &mdash; available any time the camera is open, even when the stream is working. Use this to test before sending a print, after a firmware update, or to confirm the camera link is healthy.
- **Diagnose button** next to **Retry** in the "Camera unavailable" overlay &mdash; shown when the stream is in an error state.

### What it checks

| Stage | What it tests |
|-------|---------------|
| **Network reachability** | Opens a TCP socket to the printer's camera port (`322` for X1 / X1C / X1E / X2D / P2S / H2 series, `6000` for A1 / A1 Mini / P1P / P1S). Distinguishes timeout from refused so the remediation hint is specific. |
| **Frame capture** | Captures one JPEG end-to-end via the same pipeline that powers `/camera/snapshot`. Covers authentication, protocol handshake, and the first keyframe in one step. |

If the camera is currently streaming with a fresh frame, the diagnostic short-circuits with **Live stream active** and reports success without opening a fresh socket &mdash; opening a second socket would kick the live viewer off on most Bambu firmwares.

If instead another snapshot is already being taken at that moment (an Obico poll, the printer wall refreshing), the frame capture stage shares that capture rather than opening a competing connection, and is labelled `coalesced_capture` in the stage detail. The pass is still real evidence the camera works &mdash; a JPEG came back &mdash; but the timing shown is mostly time spent waiting for the other capture, so don't read it as a connection speed. Run the diagnostic again a moment later for a clean measurement.

### What the result tells you

The modal renders each stage with a green check, red X, or grey "skipped" marker, plus a translated remediation message at the bottom. Common outcomes:

| Summary | What happened | What to do |
|---------|---------------|------------|
| **Camera is working** | All stages passed. | Nothing &mdash; the camera path is healthy. |
| **Live stream active** | Someone is already watching the camera. | Nothing &mdash; real-world proof beats any synthetic test. |
| **Printer not reachable** | TCP connect timed out or returned "no route to host". | Check the printer's IP address, that it's powered on, and that Bambuddy can reach it on the network. |
| **Camera port closed** | TCP connect was refused &mdash; printer is up, camera port isn't accepting. | Enable **LAN-only mode** and **Developer Mode** in the printer's settings. |
| **No frames received** | TCP connect succeeded but no JPEG came back within 15 seconds. | Check that the camera is enabled in the printer's settings. If you just updated firmware, try power-cycling the printer. |

### Metadata for support tickets

The bottom row of the modal shows three values used for triage:

- **Protocol** &mdash; `rtsp` (X1 / H2 / P2 family) or `chamber_image` (A1 / P1 family)
- **Port** &mdash; `322` or `6000`
- **Profile** &mdash; `default` (the historical X1/H2 fast-startup tuning) or a model name (e.g. `P2S`) when Bambuddy has model-specific tuning for that camera

If you do end up opening a ticket, a screenshot of this modal is more useful than a long log paste &mdash; it tells me which stage failed, how long it took, and which tuning profile your printer is on.

---

## :material-help-circle: Troubleshooting

!!! tip "Try the built-in diagnostic first"
    The [camera diagnostic](#built-in-camera-diagnostic) above will tell you which of the steps below is the actual fix in 5&ndash;15 seconds. Skip to it before working through the checklist.

### Stream Won't Start

1. **Is the printer on?** Camera requires power
2. **Is camera enabled?** Check printer settings
3. **Is ffmpeg installed?** Required for streaming
4. **Is Developer Mode enabled?** Required for camera access
5. **Running in Docker?** Try `network_mode: host` if having issues

### Docker: Camera Not Working

If camera streaming doesn't work in Docker, try host network mode:

```yaml
# docker-compose.yml
services:
  bambuddy:
    build: .
    network_mode: host
    # Remove the ports: section when using host mode
```

Note: Docker's default bridge networking with NAT works in most setups. Host mode is only needed if your network configuration prevents NAT'd traffic from reaching the printer.

### Stream Freezes

- Network congestion or WiFi issues
- Try lowering the FPS
- Check printer WiFi signal strength
- Try snapshot mode instead

### High Latency

MJPEG streaming typically has 1-3 seconds of latency. This is normal and due to:

- RTSP buffering
- ffmpeg processing
- HTTP streaming

### Camera Shows Black

- Camera may be initializing
- Try refreshing the stream
- Check if camera works in Bambu Studio

### Use go2rtc as a relay (workaround)

When the built-in stream is unreliable for your printer or network — connects then drops, shows black after a few seconds, won't connect at all behind a reverse proxy, or works in Bambu Studio but not in Bambuddy — you can run [go2rtc](https://github.com/AlexxIT/go2rtc) as a relay between the printer and Bambuddy. go2rtc handles the RTSPS handshake, holds the single camera connection the printer allows, and exposes a plain MJPEG URL that Bambuddy consumes via the **External Camera** feature above.

This bypasses Bambuddy's built-in RTSP / chamber-image stream entirely. Useful for:

- Printers behind firmware quirks that cause the built-in stream to drop after a few seconds
- Cross-subnet setups where Bambuddy can reach the printer for MQTT/FTP but the camera handshake is flaky
- Reverse-proxy deployments where the MJPEG response gets buffered upstream
- Sharing the single allowed camera connection across multiple clients (Frigate, OBS, Home Assistant, Bambuddy) — go2rtc holds one upstream, fans out N downstreams

#### go2rtc config (P2S example)

Add to your `go2rtc.yaml`. Replace `SERIAL#` with your printer's serial number (used as the RTSPS password) and `192.168.101.41` with your printer's IP:

```yaml
streams:
  # Primary RTSPS pull from the printer — one connection, held open by go2rtc.
  p2s_camera:
    - rtsps://bblp:SERIAL#@192.168.101.41:322/streaming/live/1
  # Transcode to MJPEG at 2 fps for Bambuddy's External Camera.
  p2s_mjpeg:
    - "ffmpeg:http://127.0.0.1:1984/api/stream.mp4?src=p2s_camera#video=mjpeg#args=-r 2"
```

The same shape works for any Bambu printer that speaks RTSPS on port `322` — **X1 / X1C / X1E / X2D / P2S / H2C / H2D / H2D Pro / H2S**. A1 / A1 Mini / P1P / P1S use a different proprietary chamber-image protocol on port `6000` and are NOT addressable through go2rtc this way; if your built-in stream fails on one of those, go2rtc isn't the workaround.

#### Bambuddy setup

1. Find the MJPEG URL go2rtc is now serving — substitute go2rtc's host IP for `192.168.101.29`:

   ```
   http://192.168.101.29:1984/api/stream.mjpeg?src=p2s_mjpeg
   ```

2. In Bambuddy → **Settings** → **Camera** → enable **External Camera** for the affected printer.
3. Paste the go2rtc URL above into the **Camera URL** field, set type to **MJPEG**, and click **Test**.
4. Save.

From here Bambuddy will use the go2rtc stream for live viewing, notification thumbnails, finish photos, and (if you've enabled them) layer timelapse and plate-empty detection — everything that previously hit the built-in camera path.

!!! tip "Optional: separate snapshot endpoint"
    go2rtc also exposes `/api/frame.jpeg?src=<name>` — a fast single-frame endpoint that's more reliable than reading one frame from the MJPEG stream. If your camera notifications or finish photos look slow or come up black, set this URL in the **Snapshot URL** field below the live-stream URL on the same External Camera config screen. See the [Snapshot URL override tip](#external-cameras) above.

---

## :material-api: API Endpoints

For developers and integrations:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/printers/{id}/camera/stream` | GET | MJPEG stream |
| `/api/v1/printers/{id}/camera/snapshot` | GET | Single JPEG frame. While someone watches the live view, this is the live view's frame; while the live view is still starting it answers `503` with `Retry-After: 2` rather than taking the camera from the viewer. An external camera with its own snapshot URL is always fetched from that URL |
| `/api/v1/printers/{id}/camera/stop` | POST | Stop active streams |
| `/api/v1/printers/{id}/camera/test` | GET | Test camera connection (returns success/failure only) |
| `/api/v1/printers/{id}/camera/diagnose` | POST | Run staged diagnostic (returns per-stage results and remediation code) |

### Example: Embed in OBS

Use the stream URL directly:

```
http://your-bambuddy-server:8000/api/v1/printers/1/camera/stream
```

Add as a **Browser Source** or **Media Source** in OBS.

---

## :material-video-box: Streaming Overlay for OBS

Bambuddy includes a dedicated overlay page designed for live streaming. It combines the camera feed with real-time print status in a single embeddable view.

### Accessing the Overlay

Navigate to:

```
http://your-bambuddy-server:8000/overlay/{printer_id}
```

For example: `http://192.168.1.100:8000/overlay/1`

!!! note "Authentication depends on your setup"
    If Bambuddy runs **without login** (no authentication enabled), the overlay
    works with the plain URL above — nothing more to do.

    If Bambuddy runs **with login enabled**, the overlay's data (print status and
    camera feed) is protected just like the rest of the app. It works in a browser
    where you are already signed in, but **OBS is a fresh browser with no login**,
    so the plain URL shows a blank overlay. Add a **Streaming Overlay token** to the
    URL (see below). This is unrelated to how you reach the server — a reverse proxy,
    Cloudflare Tunnel, or remote domain does not change it; a token is what OBS needs.

### Streaming Overlay token (login-enabled deployments)

1. Go to **Settings → Camera → Streaming Overlay**.
2. Choose **Create overlay token**, give it a name and expiry, and create it.
   The builder uses the token returned at creation. You can also create one in
   **Camera API Tokens** with the **Streaming Overlay** scope and copy it into
   **Manual token**.
3. Choose the printer and appearance settings, then choose **Copy** next to the
   URL.
4. Paste the complete URL into OBS and keep it somewhere safe.

While a token is being created, Cancel, manual entry and import are disabled,
so the token returned at creation cannot be lost. If creation fails, the
server's error is shown and those controls become available again. Import is
also unavailable while a logo upload or removal is running.

Tokens are stored as hashes and cannot be retrieved later. The builder holds
the token only in memory and clears it when you leave or reload the page.
**Manual token** and the generated URL are masked until you choose **Show
token**; **Copy** always copies the complete, working URL.

To edit an existing browser source, paste its complete URL into **Existing
overlay URL** and choose **Import URL**, or paste its token into **Manual
token**. Import reads the URL in the browser; it does not contact the server
in it. It takes the printer, token, layout, fields, text size, FPS, artwork,
camera, background transparency and branding settings. Like the overlay page,
it clamps FPS to 1–30 and ignores fields and parameters it doesn't know. A URL
that isn't an overlay URL, has more than one token, or contains a user name or
password is rejected without changing the current settings. The generated URL
uses this Bambuddy installation, so check the selected printer when importing
from another installation. Copy the updated URL back into OBS after changing
settings.

!!! note "Lost every copy of a URL?"
    A token cannot be shown again. Create a new one, update your browser
    sources, then revoke the old token.

![Streaming Overlay builder with URL import and masked manual token entry](../assets/images/streaming-overlay/token-builder.png)

The URL then looks like:

```
http://192.168.1.100:8000/overlay/1?token=bblt_xxxxxxxx_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

!!! warning "Treat the URL like a key"
    Anyone who can read the URL can watch that printer's stream and see the print
    filename. It cannot see the printer's address or access code. Revoke the token
    (same Settings page) to cut the overlay off. A **Streaming Overlay** token only
    opens the overlay for one printer at a time — it is a narrower grant than a Cam
    Wall token and cannot enumerate your other printers.

### What's Included

The overlay displays:

| Element | Description |
|---------|-------------|
| **Camera Feed** | Full-screen live camera view |
| **Bambuddy Logo** | Branding in top-right corner (links to GitHub) |
| **Printer Name / Model** | Printer name and/or model, e.g. `Big Mumma · H2D` (both off by default — see Show/Hide Elements) |
| **Filename** | Current print file name |
| **Status** | Printing, Paused, Idle, etc. |
| **Progress Bar** | Visual progress with percentage |
| **Layer Count** | Current layer / Total layers |
| **Time Remaining** | Estimated time left |
| **ETA** | Estimated completion time |
| **Nozzle / Bed / Chamber** | Live temperatures, with the target while the heater is still climbing (off by default — see Show/Hide Elements) |

### OBS Setup

1. In OBS, click **+** under Sources
2. Select **Browser**
3. Enter the overlay URL (e.g., `http://192.168.1.100:8000/overlay/1`)
4. Set width and height to match your scene (e.g., 1920x1080, or 1080x1920 for a portrait URL)
5. Click **OK**

!!! tip "Single Source"
    The overlay combines camera and status - you only need one browser source, not separate camera and text sources.

### Customization

!!! tip "Build the URL in the UI"
    **Settings → Camera → Streaming Overlay** has a builder: pick the printer,
    tick the fields you want, set layout, size, artwork, frame rate and branding,
    paste in a token if you need one, and copy the finished URL. It also has a preview so you can see
    the result before pasting it into OBS. The parameters below are what it
    produces, documented for anyone assembling a URL by hand or scripting one.

    The builder remembers your choices in the browser you used, so it reopens
    as you left it. The token is never stored: enter or import it again, or
    create a new one. **Reset choices** at the top puts every choice back to
    its default and leaves the token and the uploaded logo alone. In another
    browser, or for another admin, the builder starts from the defaults.

Customize the overlay using query parameters:

#### Landscape, portrait, or both

Choose **Layout** independently of **Artwork**. Both Classic and Version 2 support all three choices:

| Layout | URLs and previews | OBS browser-source dimensions |
| --- | --- | --- |
| Landscape, the default | One landscape URL and preview | 1920 × 1080 |
| Portrait | One portrait URL and preview | 1080 × 1920 |
| Both | Separate landscape and portrait URLs and simultaneous previews | One source at each size |

Each URL has its own **Copy** and **Open** actions. In Both mode, add the two URLs as separate browser sources to your landscape and portrait scenes. They share the selected printer, fields, text size, camera visibility, frame rate, branding and token. They can run at the same time.

**Show preview** starts only the selected previews. **Hide preview** or leaving the settings page removes them and closes their streams. Changes to the settings update both the URLs and visible previews. Previews use the recommended source dimensions, scaled down to fit the settings card.

Landscape URLs omit the layout parameter to preserve Classic text sizing at any source resolution. Portrait URLs select a fixed composition:

```text
/overlay/1
/overlay/1?layout=portrait
/overlay/1?layout=portrait&artwork=2
```

Portrait URLs keep their 1080 × 1920 composition when opened in a differently sized browser window, scaled to fit with empty space where needed. OBS makes that space transparent; a normal browser tab shows the page background there. Use the recommended dimensions in OBS to fill the source. `both` is a builder choice, not a URL parameter. Landscape URLs keep their viewport behaviour, including Classic text sizing at 1280 × 720. Existing explicit `layout=landscape` URLs remain supported and use a fixed 1920 × 1080 canvas. Unrecognised layout values follow the viewport.

Classic portrait uses larger, wrapping text and fits the camera without stretching it. Version 2 uses its portrait header, cropped camera and information panel. Both respect camera rotation. For all URLs, disconnected printers hide stale progress and temperatures, and finished prints do not also show an idle message. The Status toggle controls status text, including the offline message.

![Both layouts in the overlay builder](../images/stream-overlay/layouts-after.png)

*Version 2 with both previews. The image uses simulated data and a camera test pattern, not a live printer.*

#### Size

```
/overlay/1?size=small
/overlay/1?size=medium   (default)
/overlay/1?size=large
```

| Size | Text | Logo | Best For |
|------|------|------|----------|
| `small` | Compact | Small | Picture-in-picture |
| `medium` | Standard | Medium | Normal streams |
| `large` | Large | Large | Full-screen focus |

#### Frame Rate (FPS)

Control the camera stream frame rate:

```
/overlay/1?fps=30
```

| FPS | Description |
|-----|-------------|
| `1-5` | Low bandwidth, minimal resource usage |
| `10-15` | Default, balanced quality (15 is default) |
| `20-25` | Smoother video |
| `30` | Maximum quality |

FPS is automatically clamped between 1 and 30. The backend may further limit based on camera type (A1/P1 cameras max out at ~5 FPS).

If another viewer already has the printer's camera open, the overlay joins that
viewer's stream and runs at its frame rate - see
[Shared Streams and FPS](#shared-streams-and-fps).

#### Version 2 artwork (portrait and landscape)

In **Settings → Camera → Streaming Overlay**, set **Artwork** to **Version 2**, select the fields to show, and copy the generated URL into your OBS browser source. **Classic** is the default, and existing URLs keep the original layout unless you add `artwork=2`.

```
/overlay/1?artwork=2&show=printer,model,filename,status,progress,layers,eta,nozzle,bed,chamber
```

Set the browser source dimensions in OBS to the composition you want:

- **Portrait, 1080 × 1920:** printer identity and Bambuddy logo above a cropped camera view, with print information below.
- **Landscape, 1920 × 1080:** a full-screen camera behind printer identity and a translucent information panel along the bottom.

Screenshots below show the layouts with simulated print data and a camera test pattern, not a live printer feed.

![Classic landscape overlay](../images/stream-overlay/original-landscape.png)

*Classic, for comparison*

![Version 2 landscape overlay](../images/stream-overlay/updated-landscape.png)

*Version 2, landscape*

![Version 2 portrait overlay](../images/stream-overlay/updated-portrait.png){ width="360" }

*Version 2, portrait*

Without a `layout` parameter, Version 2 follows the browser source viewport. The builder omits this parameter for Landscape and adds `layout=portrait` for Portrait, as described above.

The camera keeps its aspect ratio and fills the available area by cropping. Portrait cropping can hide the sides of the build plate. Camera rotation and the frame-rate setting still apply.

All field toggles remain independent. Hidden or unavailable readings leave no empty boxes. Printer name and model appear in the header. The **Text size** setting adjusts the new layout too. `camera=false` hides the camera without starting a stream. A disconnected printer shows **Printer offline** when Status is enabled and hides stale progress and temperature readings. Finished and failed jobs show their status without an idle message.

When login is enabled, use the same **Streaming Overlay** token as the Classic overlay. Version 2 does not grant additional access.

Selecting **Version 2** also reveals a **Background transparency** slider. Increase it from 0% to 100% to fade the dark backgrounds while keeping text, logos and the camera fully visible. At 0%, the original backgrounds are preserved; at 100%, the backgrounds are transparent. Hide the camera if you want only the overlay information over another OBS source.

The generated URL includes `backgroundTransparency=65`, for example, when the slider is above zero. Classic ignores this parameter. Missing or invalid values preserve the default; numbers outside 0–100 are clamped to that range. Switching back to Classic hides the slider and omits the parameter without losing the slider selection for Version 2.

![Version 2 background transparency slider](../images/stream-overlay/branding-transparency-slider.png)

At 100%, a background placed behind the overlay remains visible. The checkerboard below illustrates transparent pixels and is not part of the overlay.

![Version 2 with fully transparent backgrounds](../images/stream-overlay/branding-transparent.png)

#### Custom logo and progress colours

In **Settings → Camera → Streaming Overlay → Branding**, upload a PNG or WebP logo. Transparency is preserved. Images must be at most 2 MiB and 4 million pixels; animated images are not supported. Bambuddy stores a sanitized PNG on the server, resized to fit within 512 × 512 pixels.

Enable **Custom logo** to include it in the generated overlay URL. Both Classic and Version 2 place it above the Bambuddy logo, preserving its proportions and limiting its size. The existing Bambuddy mark stays visible.

There is one shared logo per Bambuddy installation. Replacing it changes what every logo-enabled overlay loads next time it opens or refreshes. **Remove** deletes the saved logo and clears the builder's selection. Refresh existing OBS browser sources after replacing or removing the logo. Without a saved logo, the original layout is used. Unchecking **Custom logo** only removes it from the generated URL.

Use **From colour** and **To colour**, either their pickers or six-digit hex inputs, to colour the filled part of the progress bar, the Progress label, and the percentage from left to right. Set both to the same value for a solid colour. **Reset colours** restores the artwork's original colours, including its state-dependent styling. Invalid hex input does not replace the last valid selection.

In Version 2, custom progress colours override the state-dependent colours of the progress bar, Progress label, and percentage. The selected gradient stays in use when the printer state changes. Use **Reset colours** to restore the state-dependent colours.

```text
/overlay/1?logo=1&progressFrom=%23ff0088&progressTo=%230088ff
```

`logo=1` opts into the saved logo. `progressFrom` and `progressTo` must both be valid `#RRGGBB` colours; URL-encode the `#` as `%23` when writing URLs by hand. Existing URLs without these parameters retain their original appearance. Colours are stored in the URL, while the logo is stored on the server.

When login is enabled, uploading or removing the shared logo requires `settings:update`. The settings preview requires `settings:read`. OBS accesses it with the same **Streaming Overlay** token used for the status feed; camera-only and Cam Wall tokens cannot read it.

![Classic overlay with custom logo and gradient](../images/stream-overlay/branding-classic.png)

![Version 2 with custom logo and gradient](../images/stream-overlay/branding-version2.png)

#### Status-Only Mode (No Camera)

Hide the camera feed and show only the status overlay on a black background:

```
/overlay/1?camera=false
```

This is useful for:

- Streamers who want status info without camera
- Low-bandwidth scenarios
- Creating a minimal status display

Set `camera=false` or `camera=0` to hide the camera. Default is `camera=true`.

#### Show/Hide Elements

Control which elements are displayed:

```
/overlay/1?show=progress,layers,eta,filename,status
```

Available elements:

| Element | Description |
|---------|-------------|
| `progress` | Progress bar with percentage |
| `layers` | Layer count (current/total) |
| `eta` | Time remaining and ETA |
| `filename` | Print file name |
| `status` | Status text (Printing, Paused, etc.), including the offline message |
| `printer` | Printer name |
| `model` | Printer model (off by default; omitted if unknown) |
| `nozzle` | Nozzle temperature (both nozzles on a dual-nozzle printer) |
| `bed` | Bed temperature |
| `chamber` | Chamber temperature |

`printer` and `model` are independent and share one line: with both, the overlay shows `Big Mumma · H2D`; with only `model`, just `H2D`. The model comes from the printer's settings in Bambuddy and is left out, separator included, when none is stored. A long name and model is cut off with an ellipsis rather than running past the edge of a narrow OBS source. In the builder, tick **Printer model** under **Fields to show**. It is off by default, so an overlay URL already in OBS looks the same until you replace it with one that includes `model`.

Temperatures are shown whether or not a print is running — a preheating printer is exactly when they are worth watching. Each reading appears only when the printer reports it, so `chamber` produces nothing on a P1 or A1: those models publish a chamber value with no real sensor behind it, and Bambuddy leaves it out rather than putting a number on screen that means nothing.

They are **not** in the default set, so an overlay URL you are already using looks the same after upgrading. Add them explicitly:

```
/overlay/1?show=progress,eta,nozzle,bed
```

**Examples:**

```
# Progress, ETA and the hotend
/overlay/1?show=progress,eta,nozzle

# Show only progress and ETA
/overlay/1?show=progress,eta

# Print details with printer name and model
/overlay/1?show=progress,layers,eta,filename,status,printer,model

# Minimal overlay - just progress
/overlay/1?show=progress
```

#### Combined Parameters

```
# Large overlay with max FPS
/overlay/1?size=large&fps=30&show=progress,eta,filename

# Status-only with selected elements
/overlay/1?camera=false&show=progress,eta,status

# Minimal status display
/overlay/1?camera=false&size=small&show=progress
```

### Features

- **Real-Time Updates** - Status updates via WebSocket with polling fallback
- **Auto-Reconnect** - Camera stream automatically reconnects on errors
- **Gradient Overlay** - Text appears over a gradient for readability
- **Responsive** - Adapts to different browser source sizes
- **Branding** - Bambuddy logo always visible (links to GitHub)

### Idle State

When no print is running, the overlay shows:

- Camera feed (still active)
- The status line, such as "Idle", "Finished" or "Printer offline", when `status` is in `show=`
- Bambuddy logo

Progress, layers, time remaining and temperatures are hidden while the printer is offline. Without `status` in `show=`, an idle or offline printer shows no status text.

### Troubleshooting

**Overlay not loading in OBS**

- If login is enabled, make sure the URL includes a **Streaming Overlay token**
  (`?token=...`). A blank overlay that works when *you* open it but not in OBS is
  almost always a missing token — OBS has no login session. See "Streaming Overlay
  token" above.
- Verify the URL works in a **private/incognito** browser window (not your normal
  one, which is signed in). If it fails there, it will fail in OBS for the same
  reason.
- Check that OBS can reach your Bambuddy server
- Try refreshing the browser source in OBS

**Camera not showing**

- Ensure the printer is connected
- Check that camera streaming works in Bambuddy directly
- The overlay uses the same camera stream as the main app

**Camera freezes after running for a while**

The overlay renews its camera connection every 60 seconds, even if the browser
reports no image error. Reported image errors retry after three seconds. Each
new connection starts a fresh 60-second renewal period. The overlay keeps its
saved URL, token, FPS setting, rotation, and status display during renewal.
Recovery is disabled in status-only mode (`camera=false`).

Healthy connections are renewed too. The old connection is closed a moment
before the new one opens, so the picture blinks briefly; leaving it open would
pile up one connection per minute in the browser source, and a USB camera
serves only one reader. A renewal joins the camera stream Bambuddy already has
open for the printer and shows the latest frame at once. Renewal can't bring back a printer that
is offline or whose camera has stopped: the overlay picks the picture up again
once the printer sends frames.

If the camera remains frozen, check whether it works on Bambuddy's printers
page, then try refreshing the browser source. When reporting the problem,
include your Bambuddy version, browser-source application, and approximate time
until the freeze. Do not share the token or full private overlay URL.

**Status not updating**

- WebSocket connection may have failed
- Overlay falls back to polling every 2 seconds
- Refresh the browser source to reconnect

---

## :material-key-chain: Long-Lived Camera Tokens

For Home Assistant, Frigate, kiosks, or any external integration that needs a stable camera URL, mint a long-lived token instead of refreshing the 60-minute browser tokens on a cron.

### Creating a Token

1. Go to **Settings** → **Camera**
2. Scroll to **Camera API Tokens** (below External Cameras)
3. Enter a descriptive name (e.g., `Home Assistant`, `Kitchen Kiosk`, `Frigate`)
4. Pick a **scope** (see below)
5. Pick a lifetime (1–365 days, default 90)
6. Click **Create**

For every scope, the plaintext token is returned **exactly once**, at creation. Save it from the copy-to-clipboard modal, or copy the complete URL when creating a token directly in the [overlay builder](#streaming-overlay-token-login-enabled-deployments). It cannot be retrieved again.

### Scopes

| Scope | Reaches |
|-------|---------|
| **Camera stream** | The camera stream and snapshot endpoints, nothing else. This is the right choice for Home Assistant, Frigate, or anything embedding a single camera. |
| **Cam Wall** | The camera streams **plus** the read-only Cam Wall feed, which lists every printer's name, connection state and print progress — but never the print filename. Choose this only for a [Cam Wall display](#cam-wall-on-a-tv-or-kiosk). |
| **Streaming Overlay** | One printer's camera stream **plus** its live print status for the [streaming overlay](#streaming-overlay-for-obs) — including the filename shown on screen. Choose this for an OBS browser source when login is enabled. |

The three are separate grants that never widen each other: a Camera stream token
is refused by both the Cam Wall and overlay feeds; a Cam Wall token is refused by
the overlay feed (it names the file, which a Cam Wall token is trusted never to
expose) and vice versa; and creating any one token does not change what an
existing token can do. None of them can read a printer's IP address, serial
number or access code, or call any other Bambuddy API.

### Using the Token

Append the token as a query parameter to any camera-stream URL:

```
http://your-bambuddy/api/v1/printers/<printer_id>/camera/stream?token=bblt_<prefix>_<secret>
```

The same token works for snapshots:

```
http://your-bambuddy/api/v1/printers/<printer_id>/camera/snapshot?token=bblt_<prefix>_<secret>
```

### Home Assistant Example

```yaml
camera:
  - platform: mjpeg
    name: Bambu Lab X1C
    mjpeg_url: http://bambuddy.local:8000/api/v1/printers/1/camera/stream?token=bblt_a1b2c3d4_…
    still_image_url: http://bambuddy.local:8000/api/v1/printers/1/camera/snapshot?token=bblt_a1b2c3d4_…
```

### Cam Wall on a TV or Kiosk

The Cam Wall has its own URL, so you can bookmark it or point a wall-mounted
screen at it:

```
http://your-bambuddy/camwall
```

Opened in a browser you're signed in to, that's the same wall the Printers page
shows — tiles stay clickable, and the settings popover works as usual.

A TV or a Raspberry Pi in kiosk mode has no login, so it authenticates with a
**Cam Wall**-scoped token in the URL instead:

```
http://your-bambuddy/camwall?token=bblt_<prefix>_<secret>
```

When you create a Cam Wall token, Bambuddy shows you this URL fully assembled —
copy it straight into the kiosk browser.

A token wall is deliberately reduced to what a passive display needs:

- **No settings, no click-through.** There is nobody standing at a TV.
- **Compact status overlay only.** The state badge is shown; the print filename
  is not. The feed behind the page does not serve filenames at all, so the part
  on the bed is never named to a room anyone can walk into.
- **No printer addresses or access codes**, for the same reason.

Optional URL parameters, if the defaults don't suit the screen:

| Parameter | Meaning | Range |
|-----------|---------|-------|
| `maxLive` | How many tiles stream live at once; the rest poll snapshots | 1–16 |
| `interval` | Seconds between snapshot refreshes on non-live tiles | 2–60 |
| `status` | Status overlay: `off` or `compact` (a token wall cannot select `full`) | — |
| `size` | Tile size: `s`, `m`, `l` or `xl` (one tile across the full width) | — |

```
http://your-bambuddy/camwall?token=bblt_…&maxLive=9&interval=10&size=l
```

While the page is visible, live tiles reconnect their stream once a minute,
so a camera that dropped out and came back shows a picture again within a
minute, with no reload. If the camera stays down, the tile shows **No signal**
after its reconnect and one retry bring no picture; a printer's built-in camera
keeps showing its last picture until Bambuddy stops retrying it. A tile showing
**No signal** stays that way until the page is reloaded or the tile changes
between live, snapshot and paused.

!!! warning "The URL is the credential"
    Anyone who can read that URL — off the screen, out of the browser history,
    out of the kiosk's config file — can watch the wall. Treat it like a key. If
    a display is retired or compromised, revoke the token: the wall goes dark on
    its next request.

### Security & Limits

- **Maximum lifetime is 365 days.** Bambuddy explicitly rejects "never expires" because a leaked permanent token would be irrevocable footgun-by-design.
- **Tokens are stored as a hash.** A DB dump can't be replayed against the camera endpoint, and a token cannot be recovered from its hash. Token lists show names and dates, never the token itself. Treat saved browser-source URLs as credentials.
- **Browser memory only.** The builder does not save entered, imported or newly created credentials in browser storage. Leaving or reloading clears them.
- **Scoped, and scopes don't leak into each other.** A Camera stream token reaches only the stream and snapshot endpoints; a Cam Wall token additionally reaches the Cam Wall feed; a Streaming Overlay token additionally reaches one printer's overlay status and the shared overlay logo. None can call any other Bambuddy API, and none exposes an IP address, serial number or access code.
- **Revocable at any time.** Owners can revoke their own tokens; admins can revoke anyone's from the same panel.
- **Last-used timestamp** is shown so you can identify dead config and clean up.

### Permission Requirements

Creating and managing camera tokens requires the `camera:view` permission — the same permission already needed for the existing 60-minute browser-side stream tokens. Default Viewers and Operators groups have it.

To delegate token management to a non-admin user, ensure they're in a group with both `camera:view` and `settings:read` (so they can reach Settings → Camera).

### Revoking a Token

1. Go to **Settings** → **Camera** → **Camera API Tokens**
2. Find the token in the list (use the `lookup_prefix` to identify it if you've forgotten the name)
3. Click **Revoke**, confirm in the modal

Any device using that token will lose access on the next request. There's no grace period.

### Admin "All Users" View

Administrators see an additional table titled **All users (admin view)** below their own tokens. It lists every active long-lived token across all users — useful for triage if a token is suspected of being leaked.

---

## :material-lightbulb: Tips

!!! tip "Timelapse Videos"
    The printer creates timelapse videos automatically. Bambuddy detects the new timelapse after each print and attaches it to the archive. X1/A1 series save MP4 files; P1 series save AVI files — both are supported. AVI files are automatically converted to MP4 in the background for browser playback. If automatic detection fails (e.g., the printer was slow to encode), use the **Scan for Timelapse** button in the archive to manually trigger a scan. View and edit them using the built-in timelapse editor.

!!! tip "Multiple Cameras"
    In embedded mode, open multiple camera viewers simultaneously - each remembers its own position and size. Great for monitoring your entire print farm on one screen.

!!! tip "Mobile Viewing"
    Camera streaming works on mobile devices with full touch support. Pinch to zoom, drag to pan when zoomed. Access from the printer card camera icon.

!!! tip "Bandwidth Conservation"
    Close camera windows when not actively watching to save server resources and bandwidth.

!!! tip "Running more than a handful of printers?"
    On X1, X2, H2 and P2 the stream is transcoded in software, which costs roughly a
    CPU core per open 1080p camera — so a large farm hits a ceiling on *simultaneous
    streams*, not on printer count. See
    [Running a Large Farm](../reference/farm-sizing.md) for the measured figures and
    the settings that matter.
