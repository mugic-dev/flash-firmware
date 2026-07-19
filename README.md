# MUGIC Firmware Flasher (web)

A browser-based flasher so a non-technical user can update a MUGIC device over the
USB-C cable — no terminal, no `idf.py`. It wraps [esptool-js] (the JavaScript port of
esptool) and talks to the board with the browser's **Web Serial** API.

## For end users

1. Open **`flash.html`** in **Google Chrome** or **Microsoft Edge** (Safari/Firefox are not
   supported — they have no Web Serial).
2. Plug the MUGIC in with a USB-C **data** cable.
3. Click **Connect device** and pick the serial port in the browser dialog.
4. Leave **"Use the firmware included with this page"** selected (or choose your own
   `main.bin`) and click **Update firmware**.
5. Wait for **"Done — device restarted with new firmware."** Don't unplug while it works.

### If no device shows up when connecting
The MUGIC board uses a **WCH CH34x** USB-to-serial chip. macOS 13+ usually has the driver
built in; Windows and older macOS normally need it once:
- Driver: <https://www.wch-ic.com/downloads/CH34XSER_MAC_ZIP.html> (macOS) /
  the CH341SER package for Windows.

## Two ways to supply `main.bin`

- **Bundled (recommended):** drop a known-good `main.bin` next to `flash.html`. The page
  `fetch()`es it so the user just clicks one button. This requires the page to be **served**
  over https or localhost (see below) — a bare `file://` double-click can't fetch a sibling
  file in most browsers.
- **Choose a file:** the user selects a `main.bin` from their drive. Works even from
  `file://`. Good for developers / custom builds.

In both cases the file stays local — it is streamed straight to the device over serial and
is never uploaded anywhere.

## Hosting

Web Serial only runs in a **secure context** (https or `localhost`). Host `flash.html`
(and, for the bundled option, `main.bin`) the same way as `ble_website`, e.g.:

```
cd flash_website
http-server -S -C ../ble_website/server.cert -K ../ble_website/server.key
```

or publish it as a static page next to the BLE configurator.

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
