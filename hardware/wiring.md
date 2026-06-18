# Verified Wiring

This file records the wiring verified on the current EWT73-2G4M04S1A test
board.

## AHT20 Temperature and Humidity Sensor

```text
AHT20 VIN -> EWT73 3V3
AHT20 GND -> EWT73 GND
AHT20 SCL -> EWT73 P0.12
AHT20 SDA -> EWT73 P0.11
```

Notes:

- The purchased AHT20 module is marked `VIN`, not `VCC`.
- `VIN` must be connected to EWT73 `3V3`; leaving it unpowered causes
  `aht=fail`.
- The I2C address used by the firmware is `0x38`.

## Passive Buzzer

```text
Passive buzzer + -> EWT73 P0.15
Passive buzzer - -> EWT73 GND
```

Notes:

- The buzzer is driven from GPIO/PWM on `P0.15`.
- The buzzer does not connect to `3V3`.
- `GND` is shared, so AHT20 GND and buzzer GND can both connect to EWT73 GND.
