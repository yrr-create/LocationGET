# Progress Log

用于记录每天做了什么、卡在哪里、下一步是什么。每次改完建议提交 Git。

## 2026-06-04

### 已确认

- 模块型号：Ebyte E73-2G4M04S1A
- 芯片方向：nRF52810
- SDK 方向：nRF5 SDK + S112
- 官方示例基准：`pca10040e`
- 项目路线：先 blinky，再 BLE UART，再 GPS 坐标上报

### 当前阶段

- 第 1 关：亮灯工程已编译通过
- 第 2 关：准备连接 ARM V12/J-Link 和 EWT73 SWD

### 下一步

1. 在 SEGGER Embedded Studio 中打开 `examples\peripheral\blinky\pca10040e\blank\ses\blinky_pca10040e.emProject`
2. ARM V12/J-Link 接 EWT73 的 `3V3/GND/SWDIO/SWCLK`
3. 执行 `Target -> Connect J-Link`
4. 如果连接成功，再执行 `Target -> Download blinky_pca10040e`
5. 如果连接失败，把完整 Target Output 复制到 `docs/troubleshooting.md`

### Solution 状态

- 当前使用 Nordic SDK 官方 solution：`blinky_pca10040e.emProject`
- 当前仓库还没有自己的 `location_tracker_nrf52810.emProject`
- 等第 2、3 关跑通后，再从 `ble_app_uart_pca10040e_s112.emProject` 建立本项目自己的 solution

## 2026-06-07

### 做了什么

- 使用 `blinky_pca10040e.emProject` 完成编译、下载和运行验证。
- 修复 SES 8.28 对旧 SDK `flash_placement.xml` 中 `.text/.rodata size="0x4"` 的兼容问题。
- 将官方 blinky 从 `boards.h` / `bsp_board_led_invert()` 改为直接使用 `nrf_gpio.h` 控制 GPIO。
- 明确验证 EWT73 板上 `P0.17` 和 `P0.18` 可以由固件交替控制。

### 结果

- J-Link 连接成功时的关键输出：

```text
VTref=3.290V
Cortex-M4 identified.
```

- 烧录命令：

```powershell
nrfjprog --family NRF52 --program "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\pca10040e\blank\ses\Output\Debug\Exe\blinky_pca10040e.hex" --sectorerase --clockspeed 100
nrfjprog --family NRF52 --reset --clockspeed 100
```

- 烧录结果：

```text
Erase file - Done erasing
Program file - Done programming
Applying system reset.
Run.
```

- 观察结果：`P0.17` 和 `P0.18` 交替闪，说明第 3 关 GPIO 运行证明完成。

### 已保存的改法

- 当前本机 SDK 已修改：

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\peripheral\blinky\main.c
```

- 仓库内保存了可复现 patch：

```text
firmware\patches\blinky-ewt73-p017-p018.patch
```

### 下一步

- 进入第 4 关：基于 `ble_app_uart_pca10040e_s112.emProject` 做 BLE UART。
- 目标是手机 nRF Connect 能扫描到设备、连接设备，并收到固件发出的测试字符串。

## 进度模板

### YYYY-MM-DD

### 做了什么

- 

### 结果

- 

### 遇到的问题

- 

### 下一步

- 
