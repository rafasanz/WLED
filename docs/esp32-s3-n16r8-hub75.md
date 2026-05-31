# ESP32-S3-N16R8 + HUB75 wiring for WLED

This setup targets the common dual USB-C ESP32-S3-N16R8 development board shown in the photo, using a custom WLED build environment named `esp32s3_n16r8_hub75`.

## Firmware environment

Build with:

```powershell
pio run -e esp32s3_n16r8_hub75
```

The generated firmware is typically written to:

```text
.pio/build/esp32s3_n16r8_hub75/firmware.bin
```

Flash with:

```powershell
pio run -e esp32s3_n16r8_hub75 -t upload
```

Use the USB-to-UART Type-C port for the most reliable flashing and serial logs.

## Publish a GitHub release from your fork

This repo includes a dedicated GitHub Actions workflow for the `esp32s3_n16r8_hub75` environment. Push a tag that starts with `hub75-` and GitHub will build the firmware and publish it as a release in your fork.

Example:

```powershell
git tag hub75-YYYY-MM-DD-N
git push origin hub75-YYYY-MM-DD-N
```

The workflow copies `.github/platformio_override.esp32s3_n16r8_hub75.ini` into the temporary CI override file before building, so keep that tracked file updated whenever you change your local HUB75 build settings.

## HUB75 signal mapping

Use the panel signal names printed on the HUB75 connector or adapter board. The physical IDC pin numbering can vary a bit across HUB75 vs HUB75E panels, but the signal names are consistent.

| HUB75 signal | ESP32-S3 GPIO | Board header label |
| --- | --- | --- |
| `R1` | `GPIO1` | `1` |
| `G1` | `GPIO2` | `2` |
| `B1` | `GPIO42` | `42` |
| `R2` | `GPIO41` | `41` |
| `G2` | `GPIO40` | `40` |
| `B2` | `GPIO39` | `39` |
| `A` | `GPIO45` | `45` |
| `B` | `GPIO48` | `48` |
| `C` | `GPIO47` | `47` |
| `D` | `GPIO21` | `21` |
| `E` | `GPIO38` | `38` |
| `LAT` / `STB` | `GPIO4` | `4` |
| `OE` | `GPIO3` | `3` |
| `CLK` | `GPIO18` | `18` |
| `GND` | `GND` | any `GND` |

For the current `64x32 1/16 scan` panel, the physical `E` wire is normally not required. This custom build still reserves `GPIO38` for HUB75 so the same firmware stays compatible with `64x64` / `HUB75E` panels, but on a real `64x32 1/16` panel you can usually leave `E` unconnected.

## Quick wiring sketch

```text
ESP32-S3-N16R8                 HUB75 panel
---------------------------    ----------------
GPIO1   ---------------------> R1
GPIO2   ---------------------> G1
GPIO42  ---------------------> B1
GPIO41  ---------------------> R2
GPIO40  ---------------------> G2
GPIO39  ---------------------> B2
GPIO45  ---------------------> A
GPIO48  ---------------------> B
GPIO47  ---------------------> C
GPIO21  ---------------------> D
GPIO38  ---------------------> E   (needed for 64x64 / HUB75E panels)
GPIO4   ---------------------> LAT / STB
GPIO3   ---------------------> OE
GPIO18  ---------------------> CLK
GND     ---------------------> GND

External 5V PSU -------------> panel 5V
PSU GND ---------------------> panel GND
PSU GND ---------------------> ESP32 GND
```

## Important power notes

- Do not power the HUB75 panel from the ESP32 board.
- Power the panel from a dedicated 5V supply sized for the panel current.
- Always share ground between the panel power supply and the ESP32.
- Keep the ribbon cable and jumper wires as short as practical.
- If the panel is unstable, dim colors look wrong, or there is ghosting, add a 3.3V to 5V buffer such as `74HCT245`/`74AHCT245`.
- A bulk capacitor near the panel power input is strongly recommended, especially on larger panels.

## WLED LED settings after flashing

After first boot, go to `Config -> LED Preferences` and add a HUB75 bus:

- Select `HUB75 (Half Scan)` for common indoor `64x32` or `64x64` panels.
- Select `HUB75 (Quarter Scan)` for outdoor 1/4 scan panels.
- Set panel width and height to your module size.
- Set panel count, rows, and columns to match your physical arrangement.
- Reboot after changing HUB75 settings. WLED notes that HUB75 changes require a reboot.

## Optional INMP441 microphone

This firmware already includes the `audioreactive` usermod, so an `INMP441` can be added later without recompiling the firmware.

- The digital microphone type and I2S GPIOs are runtime configurable in `Config -> Usermods -> AudioReactive`.
- After changing I2S microphone settings or GPIOs, save and reboot WLED.
- In this build, the I2S pins were intentionally left unassigned at compile time, so the microphone can be enabled later purely by wiring and software configuration.

### Recommended INMP441 wiring for this board

Use these free GPIOs to avoid conflicts with HUB75 on this build:

| INMP441 pin | ESP32-S3 GPIO | Notes |
| --- | --- | --- |
| `SD` | `GPIO10` | I2S data in |
| `WS` / `LRCL` | `GPIO11` | I2S word select |
| `SCK` / `BCLK` | `GPIO12` | I2S bit clock |
| `L/R` | `GND` | good default starting point |
| `VDD` | `3V3` | microphone power |
| `GND` | `GND` | common ground |

In WLED, configure:

```text
Config -> Usermods -> AudioReactive
digitalmic:type = Generic I2S
digitalmic:pin[] = [10, 11, 12, -1]
```

Pin order in `digitalmic:pin[]` is:

- `pin[0] = SD`
- `pin[1] = WS`
- `pin[2] = SCK`
- `pin[3] = MCLK`

For `INMP441`, `MCLK` is not used, so keep it at `-1`.

### Pin conflict notes

- Do not use HUB75 pins for the microphone: `1, 2, 3, 4, 18, 21, 38, 39, 40, 41, 42, 45, 47, 48`.
- This build also reserves `0`, `14`, and `15` for non-HUB75 defaults.
- Prefer avoiding `43` and `44` if you want to keep USB-UART flashing and serial logs simple.
- Prefer avoiding `35`, `36`, and `37` on ESP32-S3-R8 boards because those are typically tied to octal flash / PSRAM usage.

## Notes for common panel types

- `64x32 1/16 scan` panels usually do not need the `E` line physically connected.
- In this specific firmware, `GPIO38` is still reserved for `E` so the same binary stays ready for `64x64` / `HUB75E`.
- `64x64` panels do need `E`, so keep `GPIO38` connected.
- Some panels need a different driver mode or lower brightness. If you see ghosting, you can try adding `-D WLED_HUB75_MAX_BRIGHTNESS=239` to the build flags.
