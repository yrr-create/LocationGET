# 2026-06-14 BLE command control and button stop progress

## Current milestone

The LocationGET firmware has a working BLE asset-tag control loop using nRF Connect:

```text
nRF Connect writes to BLE UART RX -> E73 board executes command
E73 board sends BLE UART TX Notify -> phone receives current asset state
```

## Verified commands

Current firmware command text:

```text
findon  -> enter finding state
findoff -> leave finding state
s?      -> report current state
```

Current Notify payload:

```text
id=L4-001,bat=100,state=normal
id=L4-001,bat=100,state=finding
```

## Verified hardware behavior

```text
P0.18 = find LED
P0.18 is active-low on the current EWT73 board
findon  -> P0.18 blinks
findoff -> P0.18 turns off
```

Board buttons found by GPIO scan:

```text
SW1 = P0.14
SW2 = P0.13
```

Current accepted product behavior:

```text
finding state + press SW1 or SW2 -> stop finding, turn off P0.18, notify state=normal
```

This is the preferred warehouse workflow for now: the phone or management side starts the find action, and the person who finds the physical tag presses the board button to confirm and stop the alert.

## Not a current requirement

Button toggle is not part of today's accepted behavior. The button does not need to start finding from normal state at this stage.

## Next step

Add the passive buzzer:

```text
findon  -> LED blink + buzzer sound
findoff -> LED off + buzzer off
button  -> LED off + buzzer off + state=normal
```

