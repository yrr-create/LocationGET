# 2026-06-22 GY-SGP I2C Bring-up

## Goal

Add the GY-SGP gas/VOC sensor module to the existing I2C bus and verify that
the nRF52810 can see it before writing the full sensor driver.

## Wiring

```text
GY-SGP VIN -> EWT73 3V3
GY-SGP GND -> EWT73 GND
GY-SGP SCL -> EWT73 P0.12
GY-SGP SDA -> EWT73 P0.11
```

The GY-SGP module shares the same I2C bus as AHT20:

```text
AHT20 address:  0x38
GY-SGP address: 0x59
```

## Verification

nRF Connect command:

```text
scan
```

Verified response:

```text
i2c=0x38,0x59,
```

## Debug Notes

The first scan implementation used `nrf_drv_twi_rx()` against each address.
That only found AHT20:

```text
i2c=0x38,
```

After changing the scan to an address write probe, the firmware found both
devices:

```text
i2c=0x38,0x59,
```

This means the original issue was not the wiring. The GY-SGP module does not
respond to the simple direct-read scan pattern used at first.

## Next Step

Do not add VOC logic yet. First add a minimal `sgp=ok` check by sending a valid
GY-SGP / SGP40 command to address `0x59` and verifying that the device responds.
