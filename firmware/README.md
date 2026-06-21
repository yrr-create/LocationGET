# Firmware Notes

固件基于 Nordic nRF5 SDK 的 BLE UART / Nordic UART Service 示例改造。

## SDK 约定

本仓库不提交完整 nRF5 SDK。当前本机 SDK 路径：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560
```

当前主要工程：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject
```

`pca10040e` 是 Nordic 给 nRF52810 示例使用的工程配置，不是本项目的真实开发板型号。

## 烧录顺序

BLE 工程需要 S112 SoftDevice。第一次烧录或擦全片后：

1. `eraseall`
2. 烧录 S112 SoftDevice
3. 烧录 application hex
4. reset

后续只改应用层时，通常只烧 application hex 并 reset。仓库根目录的 `burn.bat` 用于这个日常流程。

## 当前 BLE 协议

设备名：

```text
L4
```

当前命令：

```text
findon
findoff
s?
```

当前返回示例：

```text
id=L4-001,bat=100,state=normal,aht=ok,temp=32.2,hum=73.8,env=warning
id=L4-001,bat=100,state=finding,aht=ok,temp=32.2,hum=73.8,env=warning
id=L4-001,bat=100,state=normal,aht=fail
```

`bat=100` 目前是占位值，还没有真实电池采样。

## 当前功能点

- NUS RX 接收手机命令。
- NUS TX Notify 返回状态。
- `findon` 控制 P0.18 查找灯和 P0.15 蜂鸣器。
- `findoff` 或板载按钮停止查找。
- AHT20 通过 TWI/I2C 读取温湿度。
- 温湿度超过阈值后返回 `env=warning` 或 `env=alarm`。
- `env=alarm` 且非查找状态时，P0.17 快闪，蜂鸣器低频报警。

## AHT20 / TWI

需要在 SES 工程中加入两个源文件，否则会出现 TWI 链接错误：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\integration\nrfx\legacy\nrf_drv_twi.c
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\modules\nrfx\drivers\src\nrfx_twi.c
```

`sdk_config.h` 需要打开：

```c
#define NRFX_TWI_ENABLED 1
#define NRFX_TWI0_ENABLED 1
#define TWI_ENABLED 1
#define TWI0_ENABLED 1
#define TWI0_USE_EASY_DMA 0
```

当前接线：

```text
AHT20 VIN -> EWT73 3V3
AHT20 GND -> EWT73 GND
AHT20 SCL -> EWT73 P0.12
AHT20 SDA -> EWT73 P0.11
```

## 蜂鸣器

当前接线：

```text
Passive buzzer + -> EWT73 P0.15
Passive buzzer - -> EWT73 GND
```

当前策略：

- 查找状态下持续间歇鸣叫。
- 环境 alarm 且不在查找状态时，低频报警。
- warning 不响，只通过 BLE 状态字段提示。

## 已保存 patch

`firmware/patches/` 记录的是开发过程中的关键补丁。部分早期 patch 里仍有旧字符串，例如 `hello from LocationGET`，它们是历史记录，不代表当前最终固件协议。

当前正式说明以本文件和仓库根目录 README 为准。
