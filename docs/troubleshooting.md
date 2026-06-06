# Troubleshooting Log

这里记录原始错误、分析和处理结果。原则：先贴完整 Output，再写判断，避免被错误总结带偏。

## 已知问题

### 1. `.text section is larger than specified size`

可能原因：

- 打开了不适合 nRF52810 的工程，例如 `pca10040` / nRF52832 方向
- 只改了 `Options -> Device`，但链接脚本和内存布局没有跟着变
- 工程配置被多次手动修改后不一致

处理方向：

- 亮灯先用 `examples\peripheral\blinky\pca10040e\blank`
- BLE UART 用 `examples\ble_peripheral\ble_app_uart\pca10040e\s112`
- 不要手动拼一个不存在的 `peripheral\blinky\pca10040e\s112` 路径

### 2. `components/modules/external` 被误判缺失

已验证：用户提供的 `nRF5_SDK_17.1.0_ddde560.zip` 内有 `components`、`modules`、`external`、`examples`。`s112nrf52720.zip` 只是 SoftDevice/API 包，不是 SDK。

处理方向：

- 不要因为 `s112nrf52720.zip` 只有少量文件就判断 SDK 缺失
- SDK 以 `nRF5_SDK_17.1.0_ddde560.zip` 为准

### 3. Windows + GCC Makefile 路径问题

现象：

```text
fatal error: nrf_section_iter.h: No such file or directory
```

但文件实际存在。

处理方向：

```bash
make ABSOLUTE_PATHS=1 PASS_INCLUDE_PATHS_VIA_FILE=1
```

如果 GCC 版本太新导致 Nordic 旧代码告警被 `-Werror` 放大，可先用 SES 或安装 Nordic 当年推荐工具链。

### 4. SES 8.28 编译 blinky 报 `.text/.rodata section is larger than specified size`

日期：2026-06-06

现象：
```text
error: .text section is larger than specified size
error: .rodata section is larger than specified size
```

本次确认：
- `blinky_pca10040e.emProject` 中 nRF52810 的 Flash/RAM 总大小是正确的：
  - `FLASH_SIZE=0x30000`，即 192KB
  - `RAM_SIZE=0x6000`，即 24KB
- 真正触发错误的是 `ses\flash_placement.xml` 里这两行：

```xml
<ProgramSection alignment="4" load="Yes" name=".text" size="0x4" />
<ProgramSection alignment="4" load="Yes" name=".rodata" size="0x4" />
```

处理方式：
- 不删除 `FLASH_SIZE=0x30000`
- 不把芯片改成 nRF52832
- 只删除 `.text` 和 `.rodata` 的 `size="0x4"`：

```xml
<ProgramSection alignment="4" load="Yes" name=".text" />
<ProgramSection alignment="4" load="Yes" name=".rodata" />
```

验证结果：
```text
emBuild.exe -config Debug -rebuild blinky_pca10040e.emProject
```

在当前电脑上已通过，并生成：

```text
Output\Debug\Exe\blinky_pca10040e.elf
Output\Debug\Exe\blinky_pca10040e.hex
Output\Debug\Exe\blinky_pca10040e.map
```

## 新问题记录模板

### 标题

日期：

阶段：

原始 Output：

```text

```

我当时做了什么：

- 

判断：

- 

下一步验证：

- 
