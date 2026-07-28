# Intel Quick Sync for Camera Processing

Bambuddy can use Intel Quick Sync Video for hardware-accelerated camera stream processing.

In this mode, the H.264 stream is decoded using `h264_qsv`, while MJPEG for the Bambuddy interface is encoded using `mjpeg_qsv`.

On a test system with an Intel N100, processing a single H2D stream at 1680x1080 and 15 FPS resulted in an average FFmpeg process load of 47.90% in software mode and 5.66% with QSV enabled. Intel video engine utilization stayed between 8% and 11%.

When processing streams from multiple printers at the same time, GPU load will be higher and should be monitored separately.

Software processing remains the default mode. Intel Quick Sync must be enabled manually in the camera settings.

## Requirements

Intel Quick Sync requires:

- an Intel GPU with Quick Sync Video support;
- access to a DRM render device matching `/dev/dri/renderD*`;
- read and write access to the render device for the user running Bambuddy;
- FFmpeg with `h264_qsv` and `mjpeg_qsv` support;
- an installed Intel media driver;
- a working QSV/oneVPL runtime.

Check for available render devices:

```bash
ls -la /dev/dri/renderD*
```

Bambuddy automatically tests available `/dev/dri/renderD*` devices and selects the first one that successfully initializes QSV.

## Installing Required Packages

On Debian, FFmpeg, the Intel media driver, and `vainfo` can usually be installed with:

```bash
apt update
apt install ffmpeg intel-media-va-driver vainfo
```

Some systems may also require the oneVPL runtime:

```bash
apt install libvpl2
```

## Checking Hardware Compatibility

Check the capabilities exposed by the Intel GPU and VAAPI driver:

```bash
vainfo
```

The command must complete without errors related to `/dev/dri` access, driver loading, or VAAPI initialization.

The Bambuddy QSV pipeline requires:

```text
VAProfileJPEGBaseline : VAEntrypointEncPicture
VAProfileH264Main     : VAEntrypointVLD
```

Another supported H.264 profile may be present instead of `VAProfileH264Main`, for example:

```text
VAProfileH264High : VAEntrypointVLD
```

To show only the required capabilities:

```bash
vainfo 2>/dev/null   | grep -E 'VAProfileJPEGBaseline.*VAEntrypointEncPicture|VAProfileH264(Main|High).*VAEntrypointVLD'
```

The output must contain:

- `VAProfileJPEGBaseline` with `VAEntrypointEncPicture`;
- at least one H.264 profile with `VAEntrypointVLD`.

If either capability is missing, the Bambuddy QSV pipeline will not work.

## Checking FFmpeg

Check for the required decoder and encoder:

```bash
ffmpeg -hide_banner -decoders | grep h264_qsv
ffmpeg -hide_banner -encoders | grep mjpeg_qsv
```

Both commands must return the corresponding codec.

## Checking Permissions

Check the owner and group of the render device:

```bash
ls -ln /dev/dri/renderD*
```

Test access as the Bambuddy service user:

```bash
sudo -u <bambuddy-user> test -r /dev/dri/renderD128 && echo readable
sudo -u <bambuddy-user> test -w /dev/dri/renderD128 && echo writable
```

Replace `renderD128` with the actual device path when necessary.

The service user usually needs to be a member of the `render` group:

```bash
usermod -aG render <bambuddy-user>
```

Restart Bambuddy after changing group membership:

```bash
systemctl restart bambuddy
```

To identify the service user:

```bash
systemctl cat bambuddy
```

Check the user's current groups:

```bash
id <bambuddy-user>
```

## Enabling Intel Quick Sync

In the Bambuddy interface:

1. Open `Settings`.
2. Open the camera settings section.
3. Select `Intel Quick Sync` as the video processing mode.
4. Run the QSV diagnostic.

The diagnostic checks:

- FFmpeg availability;
- available `/dev/dri/renderD*` devices;
- `h264_qsv` and `mjpeg_qsv` availability;
- VAAPI device initialization;
- derived QSV device initialization;
- hardware encoding of a test MJPEG frame.

The diagnostic uses the same device initialization chain as the live camera stream:

```text
VAAPI device -> QSV device -> hardware frame upload -> mjpeg_qsv
```

## Automatic Device Selection

Bambuddy does not use a hardcoded `/dev/dri/renderD128` path.

On the first QSV start, Bambuddy:

1. Finds `/dev/dri/renderD*` devices.
2. Checks access permissions.
3. Runs a QSV probe on each device.
4. Selects the first working device.
5. Caches the selected path for the lifetime of the Bambuddy process.

Running the diagnostic manually forces the cache to refresh.

After Bambuddy restarts, device detection runs again.

This also supports systems where the Intel GPU is exposed as, for example:

```text
/dev/dri/renderD129
```

## Automatic Fallback

If Intel Quick Sync is selected but no compatible device is found, Bambuddy automatically falls back to software processing.

If the QSV process fails to start or exits with a QSV, MFX, or VAAPI hardware error, Bambuddy also switches that stream to software processing.

A normal RTSP network disconnect or reconnect is not treated as a GPU failure. Bambuddy will continue using QSV when the connection is restored.

Fallback is performed only once per stream, preventing endless restart and fallback loops.

## Diagnostics

### Checking the Log

Follow the Bambuddy log:

```bash
journalctl -u bambuddy -f
```

A successful QSV startup produces messages similar to:

```text
Using RTSP protocol for H2D with intel_qsv video processing
Using Intel Quick Sync render device /dev/dri/renderD128
Starting RTSP camera stream
```

A software fallback produces a message containing:

```text
falling back to software video processing
```

Show only QSV-related messages:

```bash
journalctl -u bambuddy --since "10 minutes ago"   | grep -Ei 'quick sync|qsv|mfx|vaapi|renderD|falling back'
```

### Render Device Is Missing

Example error:

```text
/dev/dri/renderD*: No such file or directory
```

Check whether the Intel driver is loaded:

```bash
lsmod | grep i915
```

If there is no output:

1. Make sure the Intel integrated GPU is enabled in BIOS.
2. Reboot the system.
3. Check `/dev/dri/renderD*` again.

When Bambuddy runs in a VM or container, pass the GPU through to the guest system. See the Proxmox LXC section below.

### Permission Denied

Identify the Bambuddy service user:

```bash
systemctl show bambuddy -p User
```

Check render device permissions and the user's groups:

```bash
ls -ln /dev/dri/renderD*
id <bambuddy-user>
```

Add the Bambuddy user to the `render` group:

```bash
usermod -aG render <bambuddy-user>
systemctl restart bambuddy
```

If Bambuddy runs in a container and the numeric group ID does not match the render device group ID, fix the group mapping or device permissions on the host.

### `vainfo` Does Not Start or Cannot See the GPU

Test a specific render device:

```bash
vainfo --display drm --device /dev/dri/renderD128
```

Replace the path when another render device is used.

If the command reports a driver loading error, reinstall the VAAPI driver and oneVPL runtime:

```bash
apt install --reinstall intel-media-va-driver libvpl2 vainfo
```

Then run `vainfo` again.

### FFmpeg Does Not Show `h264_qsv` or `mjpeg_qsv`

Check the codecs:

```bash
ffmpeg -hide_banner -decoders | grep h264_qsv
ffmpeg -hide_banner -encoders | grep mjpeg_qsv
```

If either command returns no output, the installed FFmpeg build does not contain the required codecs. Install or build FFmpeg with `h264_qsv` and `mjpeg_qsv` support.

### Built-in QSV Diagnostic Fails

Fix the stage marked as failed in the diagnostic panel:

- render device not found: check `/dev/dri/renderD*`;
- permission denied: fix access for the Bambuddy service user;
- codec missing: replace or rebuild FFmpeg;
- VAAPI, MFX, or QSV error: check `vainfo`, the Intel media driver, and the oneVPL runtime.

Run the diagnostic again after fixing the reported issue.

### The Stream Works but Uses Software Processing

Find the fallback reason:

```bash
journalctl -u bambuddy --since "10 minutes ago"   | grep -Ei 'quick sync|qsv|mfx|vaapi|renderD|falling back'
```

After fixing the hardware error, restart Bambuddy:

```bash
systemctl restart bambuddy
```

The render device will be detected and tested again on startup.

### The Diagnostic Passes but the Camera Does Not Start

In this case, QSV works and the problem is between Bambuddy and the printer camera.

Check:

- whether the printer is powered on;
- whether the printer is reachable over the network;
- whether the access code is correct;
- whether RTSP is enabled;
- whether another client is already using the camera stream.

QSV drivers do not need to be restarted after fixing a network or camera issue.

## Tested Configuration

The feature was tested with:

- Intel Processor N100;
- Debian 13;
- Bambuddy running in a Proxmox LXC container;
- render device `/dev/dri/renderD128`;
- H.264 hardware decoding through `h264_qsv`;
- MJPEG hardware encoding through `mjpeg_qsv`;
- a Bambu Lab H2D camera using RTSP.

In this configuration, the render device is detected automatically, the diagnostic passes, and the RTSP/MJPEG stream runs without falling back to software processing.

## Optional: Passing the GPU Through to Proxmox LXC

When Bambuddy is installed directly on a PC, `/dev/dri` is usually already available to the operating system.

For virtual machines, Intel GPU passthrough depends on the hypervisor and VM configuration. That scenario is not covered here.

### Checking the Proxmox Host

Check for the Intel render device on the host:

```bash
ls -la /dev/dri
```

Check whether the Intel driver is loaded:

```bash
lsmod | grep i915
```

### Passing `/dev/dri` into LXC

Open the container configuration:

```text
/etc/pve/lxc/<CTID>.conf
```

Add:

```ini
lxc.cgroup2.devices.allow: c 226:* rwm
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
```

Restart the container:

```bash
pct restart <CTID>
```

Check the device inside the container:

```bash
ls -la /dev/dri
```

### Permissions Inside LXC

On the Proxmox host, find the numeric GID of the render device:

```bash
ls -ln /dev/dri/renderD*
```

Example:

```text
crw-rw---- 1 0 104 226, 128 Jul 28 12:00 /dev/dri/renderD128
```

Here, `104` is the GID of the group owning the device.

Inside the container, create a group with the same GID and add the Bambuddy user to it:

```bash
groupadd -g 104 render-host
usermod -aG render-host <bambuddy-user>
```

If a group with that GID already exists, use it instead:

```bash
getent group 104
usermod -aG <existing-group> <bambuddy-user>
```

Restart Bambuddy after changing group membership:

```bash
systemctl restart bambuddy
```

Test access as the service user:

```bash
sudo -u <bambuddy-user> test -r /dev/dri/renderD128 && echo readable
sudo -u <bambuddy-user> test -w /dev/dri/renderD128 && echo writable
```

Both commands must confirm access.

In an unprivileged LXC container, matching the GID inside the container may still be insufficient because of UID/GID mapping. In that case, configure `lxc.idmap` in `/etc/pve/lxc/<CTID>.conf` so the host render device GID is passed through unchanged.

Example for GID `104`:

```ini
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 104
lxc.idmap: g 104 104 1
lxc.idmap: g 105 100105 65431
```

Allow root on the host to use this GID in the container mapping:

```bash
echo "root:104:1" >> /etc/subgid
```

After changing `lxc.idmap`, fully stop and start the container:

```bash
pct stop <CTID>
pct start <CTID>
```

Then test access as the Bambuddy user again and rerun the built-in QSV diagnostic.

