# Asset Tag Firmware Protocol Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the validated BLE UART example into a controllable warehouse asset tag firmware that accepts `FIND_ON`, `FIND_OFF`, and `STATUS?`.

**Architecture:** Keep using the Nordic SDK BLE UART / NUS example as the control channel. Add a small asset-tag command layer inside `main.c`: command comparison, finding state, LED control, and status notifications. Do not create a custom GATT service yet; nRF Connect remains the test client.

**Tech Stack:** nRF5 SDK 17.1.0, S112 v7.2.0, SEGGER Embedded Studio 8.28, nRF Command Line Tools, J-Link SWD, nRF Connect mobile.

---

## Scope Check

This plan covers firmware only. The Android app is a separate subsystem and should get its own plan after firmware commands are verified with nRF Connect.

Button and passive buzzer are not implemented in the first firmware pass because their exact GPIO wiring still needs confirmation. First pass uses P0.17/P0.18 LEDs and nRF Connect commands.

## File Structure

- Modify SDK working file: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c`
- Build project: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject`
- Generated app hex: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\Output\Release\Exe\ble_app_uart_pca10040e_s112.hex`
- Record patch after verification: `G:\Personalportfolio\LocationGET\firmware\patches\ble-uart-asset-tag-commands.patch`
- Update docs after verification: `G:\Personalportfolio\LocationGET\docs\progress.md`, `G:\Personalportfolio\LocationGET\firmware\README.md`, `G:\Personalportfolio\LocationGET\docs\troubleshooting.md`

## Task 1: Add Asset Tag Constants and LED Helpers

**Files:**
- Modify: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c`

- [ ] **Step 1: Add GPIO and delay includes**

Find the include block near the top of `main.c`. Add these two includes after `#include "nrf_pwr_mgmt.h"`:

```c
#include "nrf_gpio.h"
#include "nrf_delay.h"
```

- [ ] **Step 2: Add asset tag constants**

Find this line:

```c
#define UART_RX_BUF_SIZE                256
```

Add this block directly after it:

```c
#define L4_ASSET_ID                     "L4-001"
#define L4_STATUS_LED_PIN               17
#define L4_FIND_LED_PIN                 18
#define L4_FIND_BLINK_MS                150
```

- [ ] **Step 3: Add finding state and LED helpers**

Find the BLE/NUS instance definitions near the top of the file. Add this block after the global BLE instances and before the first function definition:

```c
static bool m_l4_finding = false;

static void l4_leds_init(void)
{
    nrf_gpio_cfg_output(L4_STATUS_LED_PIN);
    nrf_gpio_cfg_output(L4_FIND_LED_PIN);
    nrf_gpio_pin_set(L4_STATUS_LED_PIN);
    nrf_gpio_pin_clear(L4_FIND_LED_PIN);
}

static void l4_set_finding(bool finding)
{
    m_l4_finding = finding;

    if (!m_l4_finding)
    {
        nrf_gpio_pin_clear(L4_FIND_LED_PIN);
    }
}

static void l4_finding_process(void)
{
    if (m_l4_finding)
    {
        nrf_gpio_pin_toggle(L4_FIND_LED_PIN);
        nrf_delay_ms(L4_FIND_BLINK_MS);
    }
}
```

- [ ] **Step 4: Initialize LEDs**

In `main(void)`, find:

```c
    buttons_leds_init(&erase_bonds);
```

Add this line immediately after it:

```c
    l4_leds_init();
```

- [ ] **Step 5: Run build**

Run:

```powershell
& "C:\Program Files\SEGGER\SEGGER Embedded Studio 8.28\bin\emBuild.exe" -config Release -rebuild "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject"
```

Expected result:

```text
Build complete
```

## Task 2: Send Asset Status Notifications

**Files:**
- Modify: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c`

- [ ] **Step 1: Add a status sender**

Add this function after `l4_finding_process()`:

```c
static void l4_send_status(void)
{
    uint32_t err_code;
    uint8_t normal_status[]  = "id=L4-001,bat=100,state=normal\r\n";
    uint8_t finding_status[] = "id=L4-001,bat=100,state=finding\r\n";
    uint8_t * p_status       = m_l4_finding ? finding_status : normal_status;
    uint16_t length          = (uint16_t)strlen((char *)p_status);

    err_code = ble_nus_data_send(&m_nus, p_status, &length, m_conn_handle);
    if ((err_code != NRF_ERROR_INVALID_STATE) &&
        (err_code != NRF_ERROR_RESOURCES) &&
        (err_code != NRF_ERROR_NOT_FOUND))
    {
        APP_ERROR_CHECK(err_code);
    }
}
```

- [ ] **Step 2: Replace hello notification with status**

In `nus_data_handler()`, find the `BLE_NUS_EVT_COMM_STARTED` branch. Replace its current hello-sending body with:

```c
    if (p_evt->type == BLE_NUS_EVT_COMM_STARTED)
    {
        l4_send_status();
    }
```

- [ ] **Step 3: Build**

Run the same SES build command:

```powershell
& "C:\Program Files\SEGGER\SEGGER Embedded Studio 8.28\bin\emBuild.exe" -config Release -rebuild "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject"
```

Expected result:

```text
Build complete
```

## Task 3: Parse FIND_ON, FIND_OFF, and STATUS?

**Files:**
- Modify: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c`

- [ ] **Step 1: Add command comparison helper**

Add this function after `l4_send_status()`:

```c
static bool l4_command_equals(ble_nus_evt_t * p_evt, char const * p_command)
{
    uint16_t rx_len = p_evt->params.rx_data.length;
    uint8_t const * p_data = p_evt->params.rx_data.p_data;
    size_t command_len = strlen(p_command);

    while ((rx_len > 0) &&
           ((p_data[rx_len - 1] == '\r') || (p_data[rx_len - 1] == '\n')))
    {
        rx_len--;
    }

    return (rx_len == command_len) &&
           (memcmp(p_data, p_command, command_len) == 0);
}
```

- [ ] **Step 2: Add command handling at the start of RX branch**

In `nus_data_handler()`, find:

```c
    else if (p_evt->type == BLE_NUS_EVT_RX_DATA)
    {
        uint32_t err_code;
```

Add this block immediately after `uint32_t err_code;`:

```c
        if (l4_command_equals(p_evt, "FIND_ON"))
        {
            l4_set_finding(true);
            l4_send_status();
            return;
        }

        if (l4_command_equals(p_evt, "FIND_OFF"))
        {
            l4_set_finding(false);
            l4_send_status();
            return;
        }

        if (l4_command_equals(p_evt, "STATUS?"))
        {
            l4_send_status();
            return;
        }
```

- [ ] **Step 3: Build**

Run:

```powershell
& "C:\Program Files\SEGGER\SEGGER Embedded Studio 8.28\bin\emBuild.exe" -config Release -rebuild "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject"
```

Expected result:

```text
Build complete
```

## Task 4: Blink While Finding

**Files:**
- Modify: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\main.c`

- [ ] **Step 1: Update the main loop**

In `main(void)`, find the forever loop:

```c
    for (;;)
    {
        idle_state_handle();
    }
```

Replace it with:

```c
    for (;;)
    {
        l4_finding_process();
        idle_state_handle();
    }
```

- [ ] **Step 2: Build**

Run:

```powershell
& "C:\Program Files\SEGGER\SEGGER Embedded Studio 8.28\bin\emBuild.exe" -config Release -rebuild "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\ble_app_uart_pca10040e_s112.emProject"
```

Expected result:

```text
Build complete
```

## Task 5: Flash and Verify with nRF Connect

**Files:**
- Use generated hex: `G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\Output\Release\Exe\ble_app_uart_pca10040e_s112.hex`

- [ ] **Step 1: Verify J-Link target first**

Run:

```powershell
& "C:\Program Files\SEGGER\JLink_V794e\JLink.exe" -device nRF52810_xxAA -if SWD -speed 100 -autoconnect 1
```

Expected output includes:

```text
VTref=3.2
Cortex-M4 identified.
```

Type `exit` before running `nrfjprog`.

- [ ] **Step 2: Program application only**

SoftDevice is already present from the BLE UART stage. Program only the application:

```powershell
nrfjprog --family NRF52 --program "G:\Personalportfolio\NordicSDK\nRF5_SDK_17.1.0_ddde560\examples\ble_peripheral\ble_app_uart\pca10040e\s112\ses\Output\Release\Exe\ble_app_uart_pca10040e_s112.hex" --verify --sectorerase --clockspeed 100
nrfjprog --family NRF52 --reset --clockspeed 100
```

Expected output includes:

```text
Program file - Done programming
Verify file - Done verifying
Applying system reset.
Run.
```

- [ ] **Step 3: Verify phone behavior**

In nRF Connect:

1. Scan for `L4-001`.
2. Connect to `L4-001`.
3. Open NUS TX Notify.
4. Confirm initial status:

```text
id=L4-001,bat=100,state=normal
```

5. Write this to NUS RX:

```text
FIND_ON
```

Expected:

```text
id=L4-001,bat=100,state=finding
```

P0.18 blinks.

6. Write this to NUS RX:

```text
FIND_OFF
```

Expected:

```text
id=L4-001,bat=100,state=normal
```

P0.18 stops.

7. Write this to NUS RX:

```text
STATUS?
```

Expected:

```text
id=L4-001,bat=100,state=normal
```

## Task 6: Save Patch and Docs

**Files:**
- Create: `G:\Personalportfolio\LocationGET\firmware\patches\ble-uart-asset-tag-commands.patch`
- Modify: `G:\Personalportfolio\LocationGET\docs\progress.md`
- Modify: `G:\Personalportfolio\LocationGET\firmware\README.md`
- Modify: `G:\Personalportfolio\LocationGET\docs\troubleshooting.md` only if a new error appears

- [ ] **Step 1: Save SDK diff as a patch**

Run from the SDK BLE UART example root if it is a Git checkout. If it is not a Git checkout, use manual diff tooling and save the result to the patch file:

```powershell
git diff -- "examples/ble_peripheral/ble_app_uart/main.c" > "G:\Personalportfolio\LocationGET\firmware\patches\ble-uart-asset-tag-commands.patch"
```

If SDK is not under Git, record the exact changed code in the patch file before committing.

- [ ] **Step 2: Update progress**

Append a dated section to `G:\Personalportfolio\LocationGET\docs\progress.md` with:

```text
2026-06-08
- Implemented FIND_ON / FIND_OFF / STATUS? protocol.
- Verified nRF Connect can control L4-001.
- Verified P0.18 finding LED behavior.
- Next: confirm button GPIO and prepare passive buzzer PWM.
```

- [ ] **Step 3: Commit and push**

Run:

```powershell
git status --short
git add firmware/patches/ble-uart-asset-tag-commands.patch docs/progress.md firmware/README.md docs/troubleshooting.md
git commit -m "docs: record asset tag command firmware"
git push
```

Expected result:

```text
main -> main
```

## Self-Review

Spec coverage:

- BLE tag name: covered by existing `DEVICE_NAME "L4-001"` and nRF Connect verification.
- `FIND_ON`: Task 3 command parsing and Task 4 LED blinking.
- `FIND_OFF`: Task 3 command parsing and LED stop behavior.
- `STATUS?`: Task 2 status notification and Task 3 command parsing.
- P0.17/P0.18 first: Task 1 and Task 4.
- Button and passive buzzer: intentionally deferred until GPIO/wiring is confirmed.

Completion marker scan:

- No unfinished marker text is used.

Type consistency:

- Helper names are consistent: `l4_leds_init`, `l4_set_finding`, `l4_finding_process`, `l4_send_status`, `l4_command_equals`.
