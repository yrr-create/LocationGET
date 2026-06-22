# Progress Log

这里记录当前项目主线和关键里程碑。更细的每日记录保存在同目录下的 dated progress 文件中。

## Current Direction

项目名称：AssetTracker

目标：做一个 BLE 仓库资产标签原型。标签可被手机扫描和连接，可通过 BLE 命令触发查找，可用 LED / 蜂鸣器提示位置，并能上报温湿度和环境风险状态。

项目不再走 GPS 坐标上报路线。当前板子没有 GPS，资产位置由手机或网关通过 RSSI、最后出现时间和区域记录来推断。

## Milestones

### 2026-06-07 Blinky / GPIO

- 确认 E73-2G4M04S1A 基于 nRF52810。
- 使用 Nordic SDK `pca10040e` 示例完成 blinky 编译和烧录。
- 验证 J-Link / SWD 可以连接目标芯片。
- 实测 P0.17 / P0.18 可由固件控制。

关键连接成功输出：

```text
VTref=3.2xxV
Cortex-M4 identified.
```

### 2026-06-07 BLE UART

- 使用 BLE UART / Nordic UART Service 示例。
- 手机 nRF Connect 能扫描并连接设备。
- 开启 Notify 后，手机能收到固件发送的文本。

### 2026-06-10 Asset Tag Commands

- 增加资产标签命令：

```text
findon
findoff
s?
```

- `findon`：进入查找状态。
- `findoff`：退出查找状态。
- `s?`：返回当前状态。
- P0.18 用作查找灯。

### 2026-06-14 Buttons and Buzzer

- 实测板载按钮：

```text
SW1 = P0.14
SW2 = P0.13
```

- 按钮可用于停止查找。
- 无源蜂鸣器接入 P0.15。
- 查找状态下蜂鸣器按节奏鸣叫。

### 2026-06-17 AHT20

- AHT20 接入 TWI/I2C。
- 手机端通过 `s?` 可以看到温湿度。
- 已确认 AHT20 小板 VIN 必须接 EWT73 3V3，否则返回 `aht=fail`。

当前接线：

```text
AHT20 VIN -> EWT73 3V3
AHT20 GND -> EWT73 GND
AHT20 SCL -> EWT73 P0.12
AHT20 SDA -> EWT73 P0.11
```

### 2026-06-21 Environment Warning / Alarm

- 状态返回增加 `env=normal/warning/alarm`。
- warning：只通过 BLE 状态字段提示。
- alarm：非查找状态下 P0.17 快闪，蜂鸣器低频报警。
- 查找状态优先级高于环境报警。

当前返回示例：

```text
id=L4-001,bat=100,state=normal,aht=ok,temp=32.2,hum=73.8,env=warning
```

### 2026-06-22 GY-SGP I2C Bring-up

- GY-SGP 已接入同一组 I2C 总线。
- AHT20 和 GY-SGP 可以同时挂在 `P0.11 SDA` / `P0.12 SCL`。
- AHT20 地址：`0x38`。
- GY-SGP 地址：`0x59`。

当前接线：

```text
GY-SGP VIN -> EWT73 3V3
GY-SGP GND -> EWT73 GND
GY-SGP SCL -> EWT73 P0.12
GY-SGP SDA -> EWT73 P0.11
```

验证结果：

```text
i2c=0x38,0x59,
```

调试结论：

- 使用 `nrf_drv_twi_rx()` 直接读扫描时，只能看到 AHT20 的 `0x38`。
- 改成地址写探测后，可以看到 `0x38` 和 `0x59`。
- 说明 GY-SGP 接线和 I2C 地址验证通过，下一步再做 SGP 数据读取。

## Next

- 对 `0x59` 发 GY-SGP / SGP40 命令，先验证 `sgp=ok`。
- 稳定后再把状态字段扩展成 `gas` / `voc` / `sgp`。
- 做 Android App MVP：扫描、RSSI、连接、状态显示、查找按钮。
- 决定是否把当前 Nordic SDK 示例整理成仓库内独立固件工程。
