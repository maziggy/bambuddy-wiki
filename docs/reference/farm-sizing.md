---
title: Running a Large Farm
description: What actually costs CPU, RAM and database throughput as your printer count grows
---

# Running a Large Farm

Bambuddy runs comfortably on a Raspberry Pi with a handful of printers. Past roughly
ten machines a few costs stop being negligible, and one of them dominates everything
else by an order of magnitude.

This page is about that: what actually scales, what does not, and which knobs are
worth turning before you buy a bigger server.

!!! tip "The short version"
    Camera streaming is almost the entire cost of a large farm, and only on
    **X1, X2, H2 and P2** printers. Everything else &mdash; MQTT, the queue, the
    archive &mdash; scales cheaply. If your server is pinned, look at cameras first
    and the database second.

---

## :material-video: Cameras: the dominant cost

How much a camera costs depends entirely on which protocol the printer speaks, and
the two families are not close.

| Models | Protocol | Server cost per open stream |
|--------|----------|-----------------------------|
| **A1, P1** | Chamber image over port 6000 | Negligible &mdash; JPEG frames arrive ready to display |
| **X1, X2, H2, P2** | RTSP (H.264) | **High** &mdash; a full software transcode to MJPEG |

Browsers cannot display an RTSP H.264 stream in an `<img>` element, so for the second
family Bambuddy decodes the video and re-encodes it as MJPEG in software. That
transcode is essentially the whole bill.

Two measurements contributed by users on their own hardware:

| Hardware | Camera | Software transcode | Alternative |
|----------|--------|--------------------|-------------|
| P2S, 1080p15 ([#2706](https://github.com/maziggy/bambuddy/issues/2706)) | H.264 | **~101% CPU**, ~11.3 Mbps | ~1&ndash;9% CPU with H.264 passthrough |
| Intel N100, H2D 1680x1080 @ 15fps ([#2621](https://github.com/maziggy/bambuddy/issues/2621)) | H.264 | **47.9% CPU** | 5.66% with Intel Quick Sync |

!!! warning "Rule of thumb"
    Budget **roughly one CPU core per open 1080p RTSP stream**. Your practical
    ceiling for simultaneous live cameras is therefore somewhere near
    *(cores &minus; 2)*, whatever your printer count is. A 58-printer farm cannot
    display 58 live cameras on any server you would want to buy &mdash; not because
    of the printer count, but because of the transcode.

### What does *not* cost you anything

Three things are cheaper than people expect, and knowing this saves a lot of
unnecessary hardware:

- **Idle printers cost nothing.** Bambuddy never streams a camera in the background.
  A stream exists only while something is actually watching it, and is torn down a
  few seconds after the last viewer leaves.
- **Extra viewers cost nothing.** Bambuddy opens **one** upstream connection per
  printer and fans it out to every viewer. Five people watching the same printer is
  one stream, not five. (This is also why it works at all &mdash; most Bambu
  firmwares allow only a single camera connection.)
- **Off-screen Cam Wall tiles cost nothing.** A tile only draws when it is at least
  40% visible; scrolled out of view, it stops using the network entirely.

So the load is driven by **how many cameras are open at once**, not by how many
printers you own or how many people are logged in.

---

## :material-view-grid: Tuning the Cam Wall

The [Cam Wall](../features/camera.md#cam-wall-on-a-tv-or-kiosk) is where a large farm
usually meets this limit, because it is the one screen that tries to show everything
at once. It has two settings built for exactly this, in the popover behind the gear
icon:

| Setting | Default | Range | What it does |
|---------|---------|-------|--------------|
| **Live streams** | 4 | 1&ndash;16 | How many visible tiles stream live. The rest fall back to periodic snapshots. |
| **Snapshot interval** | 8s | 2&ndash;60s | How often a non-live tile refreshes its still image. |

On a big farm, start at **4 live streams and a 20&ndash;30 second snapshot interval**,
then raise the live count only if the server has headroom.

!!! warning "Snapshots are not free on X1/X2/H2/P2"
    This is the counter-intuitive part. When a tile takes a snapshot of a printer that
    has no live stream running, Bambuddy opens a **fresh** RTSP connection and starts a
    **fresh** transcoder just to grab one frame. The connection setup, not the frame,
    is the expensive part.

    A wall with 40 visible tiles on an 8-second interval is therefore starting several
    of these per second, continuously. Raising the interval to 30 seconds cuts that by
    nearly four. Making the browser window smaller &mdash; so fewer tiles are on screen
    at once &mdash; cuts it further, and costs you nothing.

    On A1 and P1 printers snapshots are cheap, because there is no transcoder to start.

!!! note "These settings live in the browser"
    Live count and snapshot interval are stored per browser, not on the server. Every
    additional screen showing a wall multiplies the load, and setting them on your
    laptop does nothing for the TV in the workshop. For a kiosk display, set them in
    the URL instead: `?maxLive=4&interval=30`. See
    [Cam Wall on a TV or kiosk](../features/camera.md#cam-wall-on-a-tv-or-kiosk).

---

## :material-eye-check: AI failure detection

[AI failure detection](../features/failure-detection.md) is **off by default**. When
you switch it on, check the per-printer selection before you walk away: with nothing
selected it monitors **every** printer.

It captures a frame from each *actively printing* machine on a fixed interval
(default 10 seconds, minimum 5) and sends it for inference. On an RTSP farm that is a
transcoder start per printer per cycle, around the clock, with nobody watching a
screen. On a large farm this is the one camera load that runs unattended, so it is
worth being deliberate about:

- Select only the printers that genuinely need watching.
- Raise the poll interval. Failures develop over minutes, not seconds; 30&ndash;60
  seconds is usually plenty.

---

## :material-database: Move the database to PostgreSQL

Unrelated to CPU load, and just as important at scale.

SQLite allows exactly **one writer at a time** across the whole database. Bambuddy
writes from several places at once &mdash; the queue scheduler, archive status
updates, runtime tracking, MQTT event callbacks, the energy snapshot loop and the FTP
retry loop. On a small setup those writes rarely collide. On a large farm, dispatch
and completion events overlap constantly.

When the wait times out, the write fails with:

```
(sqlite3.OperationalError) database is locked
```

Some code paths retry that; not all of them do. The result is occasional lost archive
status updates and runtime records rather than a visible crash, which is what makes it
easy to miss.

!!! tip "Recommended above ~10 printers"
    Migration is a backup, a `DATABASE_URL`, a restart and a restore &mdash; all from
    the Settings UI, with no command line and no data conversion on your part. See
    [PostgreSQL Support](../features/postgresql.md#migrating-from-sqlite).

Large PostgreSQL farms can also raise the connection pool with `DB_POOL_SIZE` and
`DB_MAX_OVERFLOW` if you see pool timeouts under load
([details and the connection-ceiling arithmetic](../features/postgresql.md#tuning-for-large-printer-farms)).

---

## :material-check-all: What scales cheaply

For completeness, the things that are *not* worth worrying about:

- **One MQTT connection per printer**, held open. Light, and mostly idle.
- **A 30-second REST fallback poll** per printer, covering the handful of fields MQTT
  does not push.
- **Queue, archive, statistics and search.** Bounded by activity, not by fleet size.

There are Bambuddy installations running **90+ registered printers**. Fleet count on
its own is not the constraint &mdash; concurrent camera streams and database write
contention are.

---

## :material-clipboard-check: Sizing checklist

If a large farm is loading your server, work down this list in order:

1. **Count your open cameras, not your printers.** Close viewers and kiosk tabs, and
   switch the Printers page to card view. Does the load drop? Then it is cameras.
2. **Check what is at the top of `top` or `docker stats`.** Several `ffmpeg`
   processes points at cameras; the `bambuddy` process itself points somewhere else.
3. **Check your model mix.** A1 and P1 machines are nearly free. If your fleet is
   mostly X1/X2/H2/P2, the transcode is your ceiling.
4. **Lower the Cam Wall live count and raise the snapshot interval** &mdash; on every
   browser and kiosk showing a wall.
5. **Check AI failure detection** is either off or scoped to the printers that need it.
6. **Move to PostgreSQL** if you are above ten printers and still on SQLite.

---

## :material-map-marker-path: What is being done about it

The transcode is an architectural limit, not a configuration mistake, and there are
two open tracks to remove it:

- **[#2621](https://github.com/maziggy/bambuddy/issues/2621) &mdash; Intel Quick Sync
  acceleration.** Moves the decode and encode onto the GPU. A working prototype
  measured roughly 8.5x lower CPU.
- **[#2706](https://github.com/maziggy/bambuddy/issues/2706) &mdash; H.264
  passthrough.** Removes the transcode altogether by handing the printer's existing
  H.264 stream to the browser, which decodes it in hardware. Measured at about 101%
  CPU down to 1&ndash;9%, and 11 Mbps down to 1 Mbps.

The second is the one that makes a large wall of live cameras realistic.

!!! tip "Running a big farm? Say so on those issues"
    Fleet size on those threads is what drives prioritisation, and there is no other
    way for us to know. If you are running dozens of printers and this limit is
    costing you, a comment saying so is genuinely the most useful thing you can
    contribute.

---

## :material-link-variant: Related

- [Camera Streaming](../features/camera.md) &mdash; stream modes, Cam Wall, kiosk tokens
- [PostgreSQL Support](../features/postgresql.md) &mdash; setup, migration, pool tuning
- [AI Failure Detection](../features/failure-detection.md) &mdash; configuration and scoping
- [Supported Printers](printers.md) &mdash; which models are in which camera family
- [Troubleshooting](troubleshooting.md) &mdash; symptom-first index
