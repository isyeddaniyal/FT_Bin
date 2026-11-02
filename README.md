# OTA upload example

Upload the contents of this folder to the root of the GitHub repository
`AishaKhatri/duc_ota_firmware` on branch `main`:

```text
duc_ota_firmware/
  6/
    info.json
    firmware-1.1.0.bin
```

Keep `6` at the repository root. Do not upload the enclosing `ota_example`
folder as another path level. The firmware is configured to request:

```text
https://raw.githubusercontent.com/AishaKhatri/duc_ota_firmware/refs/heads/main/6/info.json
```

## Exact JSON format

```json
{
  "hardware_version": 6,
  "firmware_version": "1.1.0",
  "url": "https://raw.githubusercontent.com/AishaKhatri/duc_ota_firmware/refs/heads/main/6/firmware-1.1.0.bin"
}
```

- `hardware_version`: integer matching `HW_REVISION_SELECT` (currently `6`).
- `firmware_version`: quoted `X.Y.Z` version matching `FIRMWARE_VERSION` in
  the binary you upload.
- `url`: direct raw download URL of that binary. It must return the binary
  without a GitHub login; do not use a GitHub `/blob/` page URL.

The JSON must be smaller than 512 bytes and the URL shorter than 256 bytes.

## Prepare the next update (example: 1.1.1)

1. Set `FIRMWARE_VERSION` to `"1.1.1"` in `include/main.h` before building.
   Keep `HW_REVISION_SELECT` set to the hardware you are deploying to.
2. Build with PlatformIO (`pio run`).
3. Copy `.pio/build/esp32-s3-devkitc-1-n16r8v/firmware.bin` into the GitHub
   `6/` folder and name the copy `firmware-1.1.1.bin`.
   Use the application `firmware.bin`, not a bootloader, partition table,
   or merged full-flash image.
4. Change `6/info.json` to:

```json
{
  "hardware_version": 6,
  "firmware_version": "1.1.1",
  "url": "https://raw.githubusercontent.com/AishaKhatri/duc_ota_firmware/refs/heads/main/6/firmware-1.1.1.bin"
}
```

5. Upload the new binary and JSON in the same commit, or upload the binary
   first and then the JSON. Use a new binary filename for each release.
6. Confirm both raw URLs open without a login. The device checks at
   03:00 Pakistan time (22:00 UTC on the previous day).

The binary included here is a copy of the current build, version `1.1.0`.
A device already running `1.1.0` skips this example, as intended. To deploy
new code to that device, build it with a different version and update both
JSON fields for that new binary. Changing only the JSON version can cause
repeated daily installations because the installed version would not match.

The current updater checks for a *different* version, not a strictly newer
version. Keep this manifest pointed at the release you intend to deploy.

If you use another GitHub owner/repository/branch, update `INFO_FILE_URL`
in `src/wifi/wifi_handle.cpp` and rebuild the device firmware as well as
updating the binary URL in this JSON.
