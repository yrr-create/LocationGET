# Windows Environment

本目录记录 AssetTracker 在 Windows 上用到的工具链。这里不是安装包目录，只记录版本、路径和踩坑点。

## 已验证工具

| 工具 | 用途 |
| --- | --- |
| SEGGER Embedded Studio 8.28 | 打开和编译 Nordic nRF5 SDK 示例工程 |
| SEGGER J-Link Software | SWD 下载、调试、J-Link Commander |
| Nordic Command Line Tools | 使用 `nrfjprog` 烧录、擦除、复位 |
| Git | 项目版本管理 |
| ripgrep | 快速搜索项目文件 |

## 常用命令

PowerShell 里进入项目目录：

```powershell
Set-Location G:\Personalportfolio\LocationGET
```

注意：`cd /d G:\Personalportfolio` 是 CMD 写法，不是 PowerShell 写法。

查找 J-Link：

```powershell
nrfjprog --ids
```

检查能否连到 nRF52810：

```powershell
& "C:\Program Files (x86)\SEGGER\JLink\JLink.exe" -device nRF52810_xxAA -if SWD -speed 100 -autoconnect 1
```

不同电脑上 J-Link 可能安装在 `C:\Program Files\SEGGER\...` 或 `C:\Program Files (x86)\SEGGER\...`，路径要以本机实际安装为准。

## 烧录方式

当前仓库根目录有 `burn.bat`，用于烧录应用程序 hex 并 reset：

```powershell
.\burn.bat
```

如果是第一次烧 BLE 工程，先确认 S112 SoftDevice 已经烧入。只改应用代码时，一般重新烧应用 hex 即可。

## 已知问题

- EWT73 的 USB 口可以供电，也可能提供串口，但不能替代 SWD 下载器。
- `nrfjprog --ids` 能看到 J-Link，只说明电脑识别到下载器；还要看 `VTref` 和 `Cortex-M4 identified` 才能确认目标板连接正常。
- PowerShell 中路径有空格时，要用 `& "完整路径"` 执行 exe。
