# Firmware Notes

## SDK 放置约定

完整 Nordic SDK 不放进本仓库。仓库只记录版本、路径和本项目后续自己改出来的固件工程。

推荐本机目录：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560
```

如果换电脑开发，先下载并解压同版本 SDK 到上面的目录，或者放到自己的稳定目录后，在本文件和工程设置中同步记录。不要依赖 `C:\Users\Administrator\Documents\Codex\...` 这类临时目录。

## 基准工程

亮灯/GPIO：

```text
nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\pca10040e\blank
```

SEGGER Embedded Studio 中要打开的 solution 文件是：

```text
nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\pca10040e\blank\ses\blinky_pca10040e.emProject
```

当前电脑推荐直接打开：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\pca10040e\blank\ses\blinky_pca10040e.emProject
```

BLE UART：

```text
nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112
```

SEGGER Embedded Studio 中要打开的 solution 文件是：

```text
nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject
```

注意：SEGGER Embedded Studio 使用 `.emProject` 作为工程/solution 文件，不是 Visual Studio 的 `.sln`。窗口标题显示 `blinky_pca10040e - SEGGER Embedded Studio` 时，说明已经打开了 `blinky_pca10040e` 这个 solution。

## 建议策略

先不要把完整 SDK 复制进本仓库。SDK 体积大，且属于第三方依赖。建议记录 SDK 下载版本和工程路径，后续只提交自己修改过的项目副本或补丁。

正式开始写固件时，可以建立：

```text
firmware/location_tracker_nrf52810/
```

从 `ble_app_uart\pca10040e\s112` 复制最小工程，再把工程内 SDK 相对路径调整为本机 SDK 路径或仓库约定的 SDK 路径。这个步骤会生成本项目自己的 SES solution，目前还没有做。

后续在自己的 solution 中逐步加入：

1. 自定义设备名
2. 固定测试字符串发送
3. `findon` / `findoff` 命令解析
4. P0.17 / P0.18 查找灯控制
5. `s?` 状态返回
6. 板载按钮停止查找
7. 无源蜂鸣器 PWM 发声

GPS 坐标上报暂时不作为主线。当前项目定位改为 BLE 仓库资产标签，位置由手机或网关扫描 RSSI、时间和区域记录推断。

## 烧录顺序

BLE UART 阶段需要：

1. Erase chip
2. Program S112 SoftDevice hex
3. Program application hex
4. Reset

亮灯 `blank` 阶段不需要 SoftDevice。

原因：

```text
S112 SoftDevice      = Nordic BLE 协议栈，负责广播、连接、GATT/ATT 和 radio 调度
BLE UART application = 本项目应用，负责设备名、NUS 服务和数据收发逻辑
```

`ble_app_uart_pca10040e_s112` 不是裸机 `blank` 程序。它会调用 SoftDevice API，例如：

```c
nrf_sdh_enable_request();
sd_ble_gap_device_name_set();
sd_ble_gap_adv_start();
ble_nus_init();
```

因此 Flash 中必须先存在 S112。该工程的典型内存布局是：

```text
0x00000  -> S112 SoftDevice
0x19000  -> BLE UART application
```

如果只烧 application，不烧 S112，BLE 程序会启动失败或进入错误处理。前面的 blinky `blank` 工程只控制 GPIO，所以不需要 SoftDevice。

## 已验证的 GPIO 试验

2026-06-07 已确认 EWT73 板上 `P0.17` 和 `P0.18` 可由固件交替控制。

本机 SDK 修改位置：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\main.c
```

仓库中保存了对应 patch，换电脑时可参考或重新应用：

```text
firmware\patches\blinky-ewt73-p017-p018.patch
```

## BLE UART / SES 8.28 兼容补丁

2026-06-07 已确认 BLE UART / S112 示例在 SEGGER Embedded Studio 8.28 下需要本地兼容处理后才能编译。

本机已修改：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\flash_placement.xml
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\components\libraries\uart\retarget.c
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c
```

仓库中保存了对应 patch：

```text
firmware\patches\ble-uart-ses828-compat.patch
```

详细原因和错误原文见：

```text
docs\troubleshooting.md
```

## BLE UART hello 发送试验

2026-06-07 已验证手机开启 `6e400003-b5a3-f393-e0a9-e50e24dcca9e` Notify 后，板子可通过 `ble_nus_data_send()` 主动发送文本到手机。

关键位置：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c
```

关键事件：

```c
BLE_NUS_EVT_COMM_STARTED
```

该事件表示手机已经开启 NUS TX characteristic 的 Notify。此时发送：

```c
ble_nus_data_send(&m_nus, hello, &length, m_conn_handle);
```

手机已收到：

```text
hello from LocationGET
```

仓库中保存了对应 patch：

```text
firmware\patches\ble-uart-send-hello-on-notify.patch
```

## AHT20 TWI driver files

For the AHT20 temperature/humidity sensor, the firmware uses the nRF5 SDK
legacy TWI driver:

```c
#include "nrf_drv_twi.h"
```

In `sdk_config.h`, enabling the TWI switches is not enough. SEGGER Embedded
Studio only compiles source files that are explicitly listed in the `.emProject`
file. Add these two files to the SES project under `nRF_Drivers`:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\integration\nrfx\legacy\nrf_drv_twi.c
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\modules\nrfx\drivers\src\nrfx_twi.c
```

Required `sdk_config.h` switches:

```c
#define NRFX_TWI_ENABLED 1
#define NRFX_TWI0_ENABLED 1
#define TWI_ENABLED 1
#define TWI0_ENABLED 1
#define TWI0_USE_EASY_DMA 0
```

Current AHT20 wiring:

```text
EWT73 P0.11 -> AHT20 SDA
EWT73 P0.12 -> AHT20 SCL
EWT73 3V3   -> AHT20 VCC
EWT73 GND   -> AHT20 GND
```

## 下一步固件协议

第 5 关进入资产标签控制协议。先继续使用 nRF Connect 手动写入 BLE UART RX characteristic 验证设备端，再开发 Android App。

手机写入命令：

```text
findon
findoff
s?
```

板子 Notify 返回：

```text
id=L4-001,bat=100,state=normal
id=L4-001,bat=100,state=finding
```

第一阶段 `bat=100` 是占位数据。真实电量需要后续确认供电方式、电池分压和 ADC 采样方案。

## BLE UART 资产标签命令

2026-06-10 已验证第 5 关核心命令：

```text
findon
findoff
s?
```

手机通过 NUS RX characteristic 写入命令，板子通过 NUS TX Notify 返回：

```text
id=L4-001,bat=100,state=finding
id=L4-001,bat=100,state=normal
```

本机 SDK 修改位置：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c
```

本次硬件观察：

- `PWR` 是电源指示灯，不受固件控制。
- `P0.17` 作为状态灯预留。
- `P0.18` 作为查找灯。
- 当前底板 LED 是低电平亮，高电平灭，因此 `nrf_gpio_pin_set(L4_FIND_LED_PIN)` 才是关闭查找灯。

仓库中保存了对应 patch：

```text
firmware\patches\ble-uart-asset-tag-commands.patch
```
