# 2026-06-17 AHT20 TWI bring-up

## Current goal

Add an AHT20 temperature/humidity module to the BLE asset tag and verify that
the nRF52810 can detect the sensor over I2C/TWI before converting raw data into
temperature and humidity.

## Hardware wiring

Current sensor wiring:

```text
EWT73 P0.11 -> AHT20 SDA
EWT73 P0.12 -> AHT20 SCL
EWT73 3V3   -> AHT20 VCC
EWT73 GND   -> AHT20 GND
```

The buzzer remains on:

```text
EWT73 P0.15 -> passive buzzer +
EWT73 GND   -> passive buzzer -
```

GND is a shared reference. If multiple peripherals need GND, use a breadboard
GND rail or a splitter from one EWT73 GND pin.

## Firmware pins and address

Planned definitions in `main.c`:

```c
#define L4_AHT20_SDA_PIN 11
#define L4_AHT20_SCL_PIN 12
#define L4_AHT20_ADDR    0x38
```

## Required SDK configuration

File:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\config\sdk_config.h
```

Required switches:

```c
#define NRFX_TWI_ENABLED 1
#define NRFX_TWI0_ENABLED 1
#define TWI_ENABLED 1
#define TWI0_ENABLED 1
#define TWI0_USE_EASY_DMA 0
```

## Required SES project files

This is the important SES step. Enabling `sdk_config.h` only enables driver code
paths. It does not add implementation files to the SEGGER Embedded Studio
project.

Add these two files to the SES project under `nRF_Drivers`:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\integration\nrfx\legacy\nrf_drv_twi.c
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\modules\nrfx\drivers\src\nrfx_twi.c
```

If they are missing, common build or link errors include unresolved references
to `nrf_drv_twi_*` or `nrfx_twi_*`.

## First verification target

Before parsing real temperature and humidity, `s?` should report whether the
sensor can be reached:

```text
id=L4-001,bat=100,state=normal,aht=ok
```

or:

```text
id=L4-001,bat=100,state=normal,aht=fail
```

Only after `aht=ok` is stable should the firmware add fields such as `temp` and
`hum`.

## End-of-day checkpoint

Verified:

```text
s? -> aht=ok
```

This means the current wiring and TWI presence check can reach the AHT20 at
address `0x38`.

Not finished yet:

```text
temp/humidity conversion
l4_aht20_read()
status fields: temp=..., hum=...
```

The next development step is to complete `l4_aht20_read()`:

```text
1. TX command: 0xAC 0x33 0x00
2. delay 80 ms
3. RX 6 bytes
4. parse 20-bit humidity and 20-bit temperature
5. report temp/hum in the BLE status string
```

Important WIP note: if a partial `l4_aht20_read()` function was typed into the
local SDK `main.c`, finish or remove that partial function before the next
Release build.
