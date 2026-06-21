# 2026-06-14 Buzzer finding alert progress

## Current milestone

The AssetTracker tag now has a real sound-and-light finding behavior.

Verified behavior:

```text
findon  -> state=finding, P0.18 blinks, buzzer repeats
findoff -> state=normal, P0.18 turns off, buzzer stops
SW1/SW2 while finding -> state=normal, P0.18 turns off, buzzer stops
```

## Hardware

The passive buzzer is connected to:

```text
P0.15 -> buzzer signal
GND   -> buzzer ground
```

The first bring-up step used a one-shot boot test to prove:

```text
P0.15 can output
the buzzer wiring works
the passive buzzer can be driven by a GPIO square wave
```

After that test passed, the boot beep was disabled and the buzzer was moved into the finding behavior.

## Firmware behavior

Current firmware uses:

```text
L4_BUZZER_PIN = 15
l4_buzzer_init()
l4_buzzer_find_pattern()
l4_finding_process()
```

`l4_buzzer_find_pattern()` generates a square wave using GPIO set/clear with short microsecond delays. The main loop repeatedly calls `l4_finding_process()` while `m_l4_finding == true`, so the buzzer continues to sound during the whole finding state.

The important design choice is:

```text
state decides behavior
```

Commands and buttons only change `m_l4_finding`. The output behavior follows that state:

```text
m_l4_finding = true  -> light + buzzer finding alert
m_l4_finding = false -> normal state, alert stops
```

## Current command protocol

```text
findon  -> enter finding
findoff -> leave finding
s?      -> report state
```

Notify payload:

```text
id=L4-001,bat=100,state=normal
id=L4-001,bat=100,state=finding
```

## Next cleanup

The firmware still contains temporary GPIO scan helpers and the boot-test buzzer function. They are useful history, but should be cleaned before the project is treated as a polished firmware baseline:

```text
l4_button_scan_init()
l4_button_scan_process()
l4_send_button()
l4_buzzer_test_beep()
```

Keep the current behavior first; cleanup can be a separate commit after another successful build and flash.
