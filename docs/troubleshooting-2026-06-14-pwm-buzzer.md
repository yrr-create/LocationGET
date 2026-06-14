# 2026-06-14 PWM buzzer troubleshooting

## Context

Stage: replace GPIO delay-loop buzzer with nRF52 PWM driver.

Project:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject
```

Firmware file:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c
```

Config file:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\config\sdk_config.h
```

## Problem A: compile error

Output:

```text
NRFX_PWM0_INST_IDX undeclared here (not in a function)
```

Cause:

`main.c` uses the legacy PWM driver:

```c
#include "nrf_drv_pwm.h"
static nrf_drv_pwm_t m_l4_pwm = NRF_DRV_PWM_INSTANCE(0);
```

In nRF5 SDK 17, `nrf_drv_pwm` is a legacy wrapper over `nrfx_pwm`. Therefore both the nrfx driver switch and the legacy driver switch must be enabled.

Required `sdk_config.h` values:

```c
#define NRFX_PWM_ENABLED 1
#define NRFX_PWM0_ENABLED 1
#define PWM_ENABLED 1
#define PWM0_ENABLED 1
```

Observed mistake:

```c
#define NRFX_PWM_ENABLED 1
#define NRFX_PWM0_ENABLED 1
#define PWM_ENABLED 1
#define PWM0_ENABLED 0
```

`PWM0_ENABLED` was still disabled, so the instance index was not generated.

## Problem B: link error

After all four switches were enabled, compile passed but link failed:

```text
undefined reference to `nrfx_pwm_init'
undefined reference to `nrfx_pwm_simple_playback'
undefined reference to `nrfx_pwm_stop'
```

Cause:

The header and config were enabled, but the implementation file was not part of the SES project build.

Fix in SEGGER Embedded Studio:

```text
nRF_Drivers -> right click -> Add Existing File...
```

Add:

```text
G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\modules\nrfx\drivers\src\nrfx_pwm.c
```

## Lesson

Enabling a driver in `sdk_config.h` does not automatically add the source file to an existing SES project. The macro enables code paths; the `.c` implementation still has to be compiled by the project.

