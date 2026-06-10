# Power Budget

This document defines the initial Phase 0 power budget for the Modular Smart Greenhouse Control System.

## 1. Power Supply

Selected supply:

```text
12 V DC, 12.5 A
```

Available power:

```text
P = V × I
P = 12 V × 12.5 A
P = 150 W
```

## 2. Power Architecture

```text
12 V DC Power Supply
        |
        v
Power Distribution Board
        |
        +--> 12 V rail
        +--> 5 V rail
        +--> 3.3 V rail
```

## 3. Power Rails

| Rail | Purpose |
|---|---|
| 12 V | Pump, LED driver input, fan, actuator power |
| 5 V | Sensors, communication board input, peripheral modules |
| 3.3 V | STM32H7, ESP32 logic, Ethernet PHY, low-voltage sensors |

The ESP32 communication board will be powered from 5 V and will generate its own local 3.3 V rail.

## 4. Known and Reserved Loads

| Load | Voltage | Current | Power | Status |
|---|---:|---:|---:|---|
| Pump | 12 V | 2.2 A max | 26.4 W | Known |
| LED strip 1 | TBD | TBD | TBD | Needs measurement |
| LED strip 2 | TBD | TBD | TBD | Needs measurement |
| Fan | 12 V | 0.5 A reserved | 6 W | Placeholder |
| STM32H7 main board | 3.3 V / 5 V | TBD | 1-3 W | Estimate |
| ESP32 + LAN8720 board | 5 V input | TBD | 1-3 W | Estimate |
| Sensors | 3.3 V / 5 V / 12 V | TBD | 1-5 W | Estimate |
| Driver losses | 12 V | TBD | TBD | Depends on load |

## 5. Known Subtotal Without LEDs

Known/reserved loads:

```text
Pump:                    26.4 W
Fan placeholder:          6.0 W
Control electronics:      3-6 W
Sensors:                  1-5 W
```

Approximate subtotal without LEDs:

```text
36-43 W
```

Approximate remaining power for LEDs and driver losses:

```text
150 W - 43 W ≈ 107 W
```

## 6. Pump Driver Margin

Pump specification:

```text
Voltage: 12 V DC
Maximum current: 2.2 A
Maximum power: 26.4 W
```

The pump driver should not be designed exactly for 2.2 A.

Recommended design target:

```text
Pump driver current capability: at least 4 A
```

Reason: DC motors and pumps may draw higher current during startup, blockage, or stall.

## 7. LED Driver Unknowns

The LED strips are constant-current loads with 21 LEDs each.

The following must be measured or defined before final LED driver design:

- LED forward voltage per LED.
- LED strip total forward voltage.
- LED target current per strip.
- LED maximum power per strip.
- LED thermal requirements.
- LED arrangement: all-series or series-parallel.

## 8. LED Driver Topology Decision

The LED electrical structure determines the driver topology.

If the LED strip forward voltage is lower than the supply voltage:

```text
12 V buck constant-current driver may be possible
```

If the LED strip forward voltage is higher than the supply voltage:

```text
Boost or buck-boost constant-current driver is required
```

Because each strip has 21 LEDs, the LED arrangement must be confirmed before finalizing the LED driver.

## 9. Fan Budget

Fan type:

```text
12 V, 4-wire PWM fan
```

Reserved fan budget:

```text
12 V × 0.5 A = 6 W
```

This value must be updated after selecting the exact fan model.

## 10. Power Board Requirements

The power board shall include:

- 12 V input connector.
- Main fuse or eFuse.
- Reverse polarity protection.
- TVS diode on 12 V input.
- Bulk input capacitance.
- 5 V buck converter.
- 3.3 V regulator.
- Rail test points.
- Power-good indicators.
- Current monitoring where useful.

## 11. Current Budget Summary

| Category | Estimated Power |
|---|---:|
| Pump | 26.4 W |
| Fan reserve | 6 W |
| Control electronics | 3-6 W |
| Sensors | 1-5 W |
| LEDs | TBD |
| Driver losses | TBD |
| Total without LEDs | 36-43 W |
| Available supply power | 150 W |

## 12. Initial Conclusion

The selected 12 V / 12.5 A supply should be sufficient for the first prototype if the combined LED load remains below approximately 80-100 W including driver losses.

The LED strips are the main remaining power-budget uncertainty.
