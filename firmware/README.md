# Firmware

Firmware is based on Nordic nRF5 SDK BLE UART / Nordic UART Service.

The full Nordic SDK is not stored in this repository. The current local SDK path is:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560
```

The working SES project is:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject
```

`pca10040e` is Nordic's nRF52810 example target. The real hardware is the Ebyte E73-2G4M04S1A module on the EWT73 test board.

## Current Firmware Snapshot

The current project code is copied into:

```text
firmware\sdk_overlay\examples\ble_peripheral\ble_app_uart
```

This is not a standalone SDK. It is a record of the files changed from the Nordic example:

```text
main.c
l4_sensor\sensirion_arch_config.h
l4_sensor\sensirion_voc_algorithm.c
l4_sensor\sensirion_voc_algorithm.h
pca10040e\s112\config\sdk_config.h
pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject
```

To rebuild from a clean SDK, copy these files back to the same relative paths under `examples\ble_peripheral\ble_app_uart`.

## Device Protocol

BLE name:

```text
L4
```

Commands:

```text
findon
findoff
s?
```

Example status:

```text
id=L4-001,bat=100,state=normal,aht=ok,temp=30.0,hum=60.1,env=normal,sgp=ok,sgp_raw=30843,voc=88
```

Fields:

- `state`: `normal` or `finding`
- `aht`: AHT20 read status
- `temp`: temperature in Celsius
- `hum`: relative humidity percent
- `env`: temperature/humidity state, currently `normal`, `warning`, or `alarm`
- `sgp`: SGP40 read status
- `sgp_raw`: raw SGP40 signal for debugging
- `voc`: Sensirion VOC Index result

`bat=100` is still a placeholder. There is no real battery measurement yet.

## Current Hardware Features

- BLE UART command and notify path
- `findon`: P0.18 find LED and P0.15 buzzer
- `findoff`: stops find LED and buzzer
- Board button: stops finding
- AHT20 temperature/humidity over I2C
- GY-SGP / SGP40 raw gas signal over I2C
- Sensirion VOC Index calculation
- Background sensor sampling; `s?` returns cached values

## Burn

For normal application updates:

```powershell
cd G:\Personalportfolio\AssetTracker
.\burn.bat
```

For a fully erased chip, burn S112 first, then the application.

## Notes

The app should decide how to display `voc`, `temp`, `hum`, and alarm levels. Firmware keeps the returned field names stable and avoids UI wording.
