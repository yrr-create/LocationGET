# Windows Environment Setup

本目录记录 LocationGET 项目在 Windows 电脑上的环境配置。

## 当前电脑状态

检查日期：2026-06-05

已具备：

- Git
- ripgrep
- make
- arm-none-eabi-gcc
- Silicon Labs CP210x 串口设备，当前显示为 COM3

缺失或未确认：

- SEGGER Embedded Studio for ARM
- SEGGER J-Link Software and Documentation Pack
- Nordic Command Line Tools / nrfjprog
- 可被电脑识别的 J-Link / ARM V12 调试器

当前目录中的 `SEGGER Embedded Studio 8.28.lnk` 是坏快捷方式，目标路径不存在：

```text
C:\Program Files\SEGGER\SEGGER Embedded Studio 8.28\bin\emStudio.exe
```

安装 SEGGER Embedded Studio 后，需要重新创建或修正该快捷方式。

## 必装软件

### 1. SEGGER Embedded Studio for ARM

官方下载页：

```text
https://www.segger.com/downloads/embedded-studio/
```

安装类型选择 Windows ARM / Embedded Studio for ARM。

建议默认安装到：

```text
C:\Program Files\SEGGER\
```

安装后检查是否存在：

```text
C:\Program Files\SEGGER\SEGGER Embedded Studio <版本>\bin\emStudio.exe
```

### 2. SEGGER J-Link Software and Documentation Pack

官方下载页：

```text
https://www.segger.com/downloads/jlink/
```

安装后检查：

```powershell
where JLink.exe
where JLinkGDBServerCL.exe
```

如果电脑能识别 ARM V12/J-Link，SEGGER Embedded Studio 的 `Target -> Connect J-Link` 才能找到调试器。

### 3. Nordic Command Line Tools

可选但推荐。用于 `nrfjprog` 命令行擦除、烧录、复位。

Nordic 下载入口：

```text
https://www.nordicsemi.com/Products/Development-tools/nRF-Command-Line-Tools
```

安装后检查：

```powershell
where nrfjprog
nrfjprog --version
```

## 硬件连接检查

EWT73/E73 底板 USB 直插电脑只能提供：

- 供电
- 串口 COM

它不能替代 J-Link/SWD。

烧录需要：

```text
ARM V12/J-Link VTref -> EWT73 3V3
ARM V12/J-Link GND   -> EWT73 GND
ARM V12/J-Link SWDIO -> EWT73 SWDIO
ARM V12/J-Link SWCLK -> EWT73 SWCLK
RESET 可选
```

推荐上电顺序：

1. 断电接好 SWD 线。
2. EWT73 插 USB 给模块供电。
3. ARM V12/J-Link 插 USB。
4. 打开 SEGGER Embedded Studio。
5. 点击 `Target -> Connect J-Link`。

## 安装完成后的快捷方式修复

安装完成后，找到实际路径：

```powershell
Get-ChildItem -Path 'C:\Program Files','C:\Program Files (x86)' -Recurse -Filter emStudio.exe -ErrorAction SilentlyContinue
```

然后把 `environment\SEGGER Embedded Studio 8.28.lnk` 的目标改为实际路径，或删除旧快捷方式后重新创建。
