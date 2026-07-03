# Firmware folder — replace these placeholders

The **Reset My Duck** button on `/reset.html` reads `manifest.json` in this folder to
know what to install. Right now everything here is a **placeholder** so nothing real
gets flashed until you drop in your own build.

## What to put here

1. Your compiled ESP32 firmware binary, e.g. `deskduck-1.0.0.bin`, placed in this
   `/firmware/` folder.
2. An updated `manifest.json` that points at it.

## Editing manifest.json

- `version` — set your real version string (e.g. `"1.0.0"`).
- `chipFamily` — set the exact chip on your board. Common values:
  `"ESP32"`, `"ESP32-C3"`, `"ESP32-S2"`, `"ESP32-S3"`.
- `parts[].path` — the filename of your `.bin` (relative to this folder), and its
  flash `offset`.
  - If you export a **single merged/factory bin** from ESP-IDF or ESPHome, one part
    at offset `0` is usually correct.
  - If you have **separate** bootloader / partition-table / app bins, list each part
    with its own offset instead. Example:
    ```json
    "parts": [
      { "path": "bootloader.bin",  "offset": 4096 },
      { "path": "partitions.bin",  "offset": 32768 },
      { "path": "firmware.bin",    "offset": 65536 }
    ]
    ```
- `new_install_prompt_erase` — leave this `true`. The full erase is what wipes the
  duck's saved WiFi so it can be set up fresh on a new network.
- Delete the `_PLACEHOLDER_README` and `_KEEP` note keys once your real values are in.

## Testing

Open `deskduck.ca/reset` in **desktop Chrome or Edge**, plug an actual duck in over
USB, and click **Reset My Duck**. If the manifest or bin paths are wrong, the browser
will show an error instead of flashing — no harm done to the device.
