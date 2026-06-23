# Hardware

Current prototype hardware:

- Ebyte EWT73 test board
- E73-2G4M04S1A BLE module
- Nordic nRF52810 SoC
- AHT20 temperature and humidity sensor
- GY-SGP / SGP40 gas sensor
- Passive buzzer

The Nordic SDK project uses the `pca10040e` target because Nordic provides that
example configuration for nRF52810. It does not mean this project uses a Nordic
PCA10040E board. GPIO, buttons, LEDs, buzzer, sensors, power, and SWD wiring are
based on the EWT73 test board.

## Verified Pins

| Function | Pin | Current use |
| --- | --- | --- |
| PWR LED | Board power LED | Not controlled by firmware |
| Status LED | P0.17 | Reserved for status / environment alarm |
| Find LED | P0.18 | `findon` visual indicator |
| SW1 | P0.14 | Stop finding |
| SW2 | P0.13 | Stop finding |
| Buzzer | P0.15 | Find alert and alarm alert |
| I2C SDA | P0.11 | AHT20 and SGP40 |
| I2C SCL | P0.12 | AHT20 and SGP40 |

Detailed wiring is in `hardware/wiring.md`.

## Vendor Documents

Local vendor PDFs are kept under `hardware/vendor/` for reference, but they are
not committed to the repository. Use the official Ebyte product page or local
copies when checking pinout and schematic details.
