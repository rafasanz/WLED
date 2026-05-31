# Fork maintenance

This fork tracks the WLED development branch while preserving support for the
dual USB-C ESP32-S3-N16R8 board connected to HUB75 panels.

## Remotes

- `origin`: `https://github.com/rafasanz/WLED.git`
- `upstream`: `https://github.com/Aircoookie/WLED.git`

## Fork-specific files

- `wled00/bus_manager.cpp` contains the `ESP32S3_N16R8_DUALUSB_HUB75_PINOUT`
  GPIO mapping.
- `.github/platformio_override.esp32s3_n16r8_hub75.ini` stores the tracked
  PlatformIO environment used by GitHub Actions.
- `.github/workflows/release-hub75.yml` builds and publishes the custom firmware
  when a `hub75-*` tag is pushed.
- `.github/workflows/nightly.yml` intentionally has no scheduled trigger in this
  fork. Upstream nightlies are useful for WLED development, but this fork only
  needs manually triggered troubleshooting builds and explicit HUB75 releases.
- `supports3D/` stores the STL models used to mount the ESP32-S3 board, HUB75
  connector and LED panel.

## Upstream sync history

### 2026-05-31

- Merged `upstream/main` at `d884a3e69`.
- Preserved the ESP32-S3-N16R8 dual USB-C HUB75 GPIO mapping.
- Updated the custom environment to use the current upstream-pinned HUB75 driver
  dependency and the default half-scan HUB75 LED type.
- Disabled scheduled nightly builds in this fork.
- Added the `supports3D/` folder with the three project-specific STL models and
  its own README.

## Updating from WLED

Run:

```powershell
git fetch upstream --tags --prune
git merge upstream/main
npm ci
npm run build
npm test
pio run -e esp32dev
pio run -e esp32s3_n16r8_hub75
git push origin main
```

Resolve merge conflicts by retaining the fork-specific files and the
`ESP32S3_N16R8_DUALUSB_HUB75_PINOUT` block.

## Publishing a HUB75 release

After validating the firmware, create and push an explicit tag:

```powershell
git tag hub75-YYYY-MM-DD-N
git push origin hub75-YYYY-MM-DD-N
```
