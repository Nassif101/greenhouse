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
| LED strip 1 | 8.38 V forward | 1.05 A | 8.8 W | Measured LED load |
| LED strip 2 | 8.38 V forward | 1.05 A | 8.8 W | Measured LED load |
| Fan | 12 V | 0.55 A reserved | 6.6 W | 2-wire fan placeholder until exact model is selected |
| STM32H7 main board | 3.3 V / 5 V | TBD | 1-3 W | Estimate |
| ESP32 + LAN8720 board | 5 V input | TBD | 1-3 W | Estimate |
| Sensors | 3.3 V / 5 V / 12 V | TBD | 1-6 W | Includes headroom for extra analog sensors |
| Driver losses | 12 V | TBD | TBD | Depends on load |

## 5. Known Subtotal With Measured LED Load

Known/reserved loads:

```text
Pump:                    26.4 W
LED strip 1:              8.8 W
LED strip 2:              8.8 W
Fan:                      6.6 W
Control electronics:      3-6 W
Sensors:                  1-6 W
```

Approximate subtotal before driver losses:

```text
54.6-62.6 W
```

Approximate remaining power for driver losses and future margin:

```text
150 W - 62.6 W ≈ 87.4 W
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

## 7. LED Driver Remaining Design Inputs

The LED strips are now provisionally budgeted as constant-current loads at:

```text
Forward voltage: 8.38 V
Drive current:   1.05 A
Power:           8.8 W per channel
```

The following still need to be confirmed before final LED driver design:

- LED thermal requirements.
- Expected driver efficiency and thermal loss at full power.
- Final dimming/control implementation details around the selected driver.

## 8. LED Driver Topology Decision

The measured LED load is below the 12 V supply rail:

```text
8.38 V < 12 V
```

This makes a 12 V-input buck-style constant-current LED driver a suitable starting point.

Current candidate LED driver IC:

```text
TI LM3429
```

## 9. Fan Budget

Fan type:

```text
12 V 2-wire DC fan, about 0.55 A, exact model TBD
```

Reserved fan budget:

```text
12 V × 0.55 A = 6.6 W
```

This value should be rechecked once the final 2-wire fan model and startup current requirements are confirmed.

## 10. Power Board Requirements

The power board shall include:

- 12 V input connector.
- Main fuse or eFuse.
- Reverse polarity protection.
- TVS diode on 12 V input.
- Bulk input capacitance.
- 5 V buck converter based on TPS54302.
- 3.3 V buck converter based on TPS54302.
- Communication-board TLV75533PDBVR local 3.3 V LDO from the 5 V rail.
- Rail test points.
- Power-good indicators.
- Current monitoring where useful.

## 11. Current Budget Summary

| Category | Estimated Power |
|---|---:|
| Pump | 26.4 W |
| Fan reserve | 6.6 W |
| Control electronics | 3-6 W |
| Sensors | 1-6 W |
| LEDs | 17.6 W |
| Driver losses | TBD |
| Total before driver losses | 54.6-62.6 W |
| Available supply power | 150 W |

## 12. Initial Conclusion

The selected 12 V / 12.5 A supply should be comfortably sufficient for the first prototype with the current actuator set.

With two LED channels at about 17.6 W combined output, the main remaining power-budget uncertainties are driver losses, the final sensor set, and any additional analog front-end circuitry added later.
