# MUGIC Firmware Flasher (web)

A browser-based flasher so a non-technical user can update a MUGIC device over the
USB-C cable — no terminal, no `idf.py`. It wraps [esptool-js] (the JavaScript port of
esptool) and talks to the board with the browser's **Web Serial** API.

## For end users

1. Open **`flash.html`** in **Google Chrome** or **Microsoft Edge** (Safari/Firefox are not
   supported — they have no Web Serial).
2. Plug the MUGIC in with a USB-C **data** cable.
3. Click **Connect device** and pick the serial port in the browser dialog.
4. Leave the **version dropdown** on the latest release — or pick an older version to revert —
   (or choose your own `main.bin` file), then click **Update firmware**.
5. Wait for **"Done — device restarted with new firmware."** Don't unplug while it works.

### If no device shows up when connecting
The MUGIC board uses a **WCH CH34x** USB-to-serial chip. macOS 13+ usually has the driver
built in; Windows and older macOS normally need it once:
- Driver: <https://www.wch-ic.com/downloads/CH34XSER_MAC_ZIP.html> (macOS) /
  the CH341SER package for Windows.

## Two ways to supply firmware

- **Hosted versions (recommended):** the page reads `versions.json` and shows a dropdown of
  the `main-<ver>.bin` releases hosted next to `flash.html`, newest first. The user picks one
  (default = latest) and clicks one button. This requires the page to be **served** over https
  or localhost (see below) — a bare `file://` double-click can't `fetch()` a sibling file in
  most browsers.
- **Choose a file:** the user selects a `main.bin` from their drive. Works even from
  `file://`. Good for developers / custom builds.

In both cases the file stays local — it is streamed straight to the device over serial and
is never uploaded anywhere.

## Publishing a new firmware version

The firmware picker is driven by `versions.json` + the `main-<ver>.bin` files in this folder.
To publish a release built in the private `mugic-firmware` repo:

1. Copy the built, version-verified image here as `main-<ver>.bin`
   (e.g. `release/main-2.1.3.bin` → `main-2.1.3.bin`).
2. Add an entry to `versions.json` (newest first) and set `"latest"` to the new version:
   ```json
   { "version": "2.1.3", "file": "main-2.1.3.bin", "label": "latest" }
   ```
   Use `"label": "original"` (or omit `label`) for older entries. The page reads the embedded
   version straight off the device, so the filename/manifest are just labels — but keep them
   honest (each `main-<ver>.bin` should embed that same `<ver>`; check with
   `esptool.py image_info --version 2 main-<ver>.bin`).
3. Keep `main.bin` as a copy of the latest — it's the fallback the page uses if `versions.json`
   is missing or fails to load.
4. Commit and push; GitHub Pages redeploys automatically.

## Hosting

Web Serial only runs in a **secure context** (https or `localhost`). This repo is published
via **GitHub Pages** (branch `main`, root) at <https://mugic-dev.github.io/flash-firmware/> —
pushing to `main` redeploys it. To serve locally over https for testing:

```
http-server -S -C server.cert -K server.key
```

(generate a self-signed `server.cert`/`server.key` with `openssl req -nodes -new -x509`).

## Board / flash parameters (baked into flash.html)

Taken from the project's `idf.py` output (`build/flasher_args.json`):

| What | Value |
|------|-------|
| Chip | ESP32-C2 (ESP8684) |
| Flash | `dio`, `60m`, `4MB`, baud `460800` |
| App (main.bin) offset | `0x20000` |

**Full-flash offsets** (advanced section only): `0x0` bootloader, `0x8000` partition-table,
`0x19000` ota_data_initial, `0x20000` main.bin. The advanced full flash is gated behind a
warning — it is only needed when the **partition table changes**, and a normal firmware
update should always use the app-only path.

## Maintenance

`flash.html` pins a specific `esptool-js` version from a CDN
(`https://unpkg.com/esptool-js@0.4.6/bundle.js`). Bump that URL to update esptool-js. To run
fully offline, download that bundle next to `flash.html` and point the import at `./bundle.js`.

[esptool-js]: https://github.com/espressif/esptool-js
