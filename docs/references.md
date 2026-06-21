# References

这些资料按当前项目需要整理，不再走 GPS 路线。

## Nordic / BLE

- [nRF5 SDK documentation](https://docs.nordicsemi.com/r/bundle/nrf5_SDK_v17.1.1/page/index.html)
- [Developing for nRF52810](https://docs.nordicsemi.com/r/bundle/nrf5_sdk_v15.2.0/page/nrf52810_user_guide.html)
- [Nordic UART Service example](https://docs.nordicsemi.com/r/bundle/nrf5_SDK_v17.1.1/page/ble_sdk_app_nus_eval.html)
- [S112 SoftDevice](https://www.nordicsemi.com/Products/Development-software/S112)
- [nRF52810 product page](https://www.nordicsemi.com/Products/nRF52810)

## Hardware

- [Ebyte E73-2G4M04S1A product page](https://www.ebyte.com/product/305.html)
- E73 / EWT73 user manual and schematic: see `hardware/vendor/` when local vendor PDFs are available.
- AHT20 datasheet: use it for I2C command format, conversion formula, and operating range.
- GY-SGP / SGP40 module docs: planned for VOC / gas-related extension.

## Reading Order

1. nRF52810 and `pca10040e`: understand why this SDK example is used for nRF52810.
2. Nordic UART Service: understand how phone commands enter firmware and how Notify returns status.
3. GPIO / PWM / TWI in nRF5 SDK: used by LED, buzzer, buttons, and AHT20.
4. Android BLE scan / GATT write / Notify / RSSI: needed when replacing nRF Connect with an app.

Nordic Academy BLE Fundamentals is useful for BLE concepts, but its code examples are mostly nRF Connect SDK / Zephyr style, not nRF5 SDK style.
