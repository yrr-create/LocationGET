# Hardware Notes

本项目硬件主线是 EWT73 测试底板 + E73-2G4M04S1A 模块。模块核心芯片是 Nordic nRF52810。

## 重要说明

Nordic SDK 里使用 `pca10040e` 示例，是为了匹配 nRF52810 的内存和 SoftDevice 配置，不代表本项目使用 Nordic DK。实际 GPIO、LED、按钮、蜂鸣器和传感器接线都以 EWT73 底板实测为准。

## 已验证板载资源

| 功能 | GPIO | 当前用途 |
| --- | --- | --- |
| PWR LED | 不受固件控制 | 电源指示 |
| P0.17 | 状态灯 | 环境 alarm 时快闪 |
| P0.18 | 查找灯 | `findon` 查找提示 |
| SW1 | P0.14 | 停止查找 |
| SW2 | P0.13 | 停止查找 |

## 当前外设

### AHT20

```text
AHT20 VIN -> EWT73 3V3
AHT20 GND -> EWT73 GND
AHT20 SCL -> EWT73 P0.12
AHT20 SDA -> EWT73 P0.11
```

说明：

- 当前 AHT20 小板标的是 `VIN`，不是 `VCC`。
- 如果 VIN 没接 3V3，固件会返回 `aht=fail`。
- 已验证 I2C 地址：`0x38`。

### GY-SGP

```text
GY-SGP VIN -> EWT73 3V3
GY-SGP GND -> EWT73 GND
GY-SGP SCL -> EWT73 P0.12
GY-SGP SDA -> EWT73 P0.11
```

说明：

- GY-SGP 和 AHT20 共用同一组 I2C：`P0.11 SDA` / `P0.12 SCL`。
- 已验证 I2C 地址：`0x59`。
- 早期用 `nrf_drv_twi_rx()` 直接读扫描时，只能扫到 AHT20 的 `0x38`。
- 改为地址写探测后，可以同时扫到 `0x38` 和 `0x59`。

当前扫描结果：

```text
i2c=0x38,0x59,
```

### 无源蜂鸣器

```text
蜂鸣器 + -> EWT73 P0.15
蜂鸣器 - -> EWT73 GND
```

蜂鸣器由 GPIO/PWM 驱动，不接 3V3。GND 是公共参考点，AHT20、GY-SGP 和蜂鸣器可以共用 EWT73 GND。

## SWD 下载接线

烧录和调试需要 J-Link / SWD，不是普通 USB-TTL。

```text
J-Link VTref -> EWT73 3V3
J-Link GND   -> EWT73 GND
J-Link SWDIO -> EWT73 SWDIO
J-Link SWCLK -> EWT73 SWCLK
RESET        -> 可选
```
