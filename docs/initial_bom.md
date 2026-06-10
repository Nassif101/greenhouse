# Initial BOM

This document defines the initial Version 1 bill of materials for the Modular Smart Greenhouse Control System.

This is a planning BOM, not a manufacturing-ready BOM. It captures the current part decisions, the major PCB building blocks, and the remaining `TBD` items that still need exact selection during schematic design.

## 1. Current Assumptions

- Main input supply: 12 V DC, 12.5 A, 150 W.
- Main MCU: STM32H743ZIT6.
- LED channels: 2 identical constant-current loads, each about 8.38 V at 1.05 A, about 8.8 W.
- LED driver candidate: TI LM3429.
- Fan: 12 V 2-wire DC fan, about 120 mm x 120 mm and about 0.55 A, exact model TBD.
- RS485 transceiver candidate: SN65LBC176QDRG4.
- Main power rails: one TPS54302 for 5 V and one TPS54302 for 3.3 V.
- Communication board: custom ESP32 Ethernet board based on ESP32-WROOM-32, LAN8720, and TLV75533PDBVR.
- Pump driver candidate: MAX4427CSA+T plus AOD4184A.
- Initial sensor set: SHT41A-AW1B-R2, DFRobot SEN0193, and VEML7700, with some headroom for extra analog sensors.

## 2. Core BOM

| Subsystem | Item | Qty | Status | Notes |
|---|---|---:|---|---|
| Power board | 12 V DC input connector | 1 | TBD | Select final connector style and current rating |
| Power board | TPS54302 5 V buck regulator | 1 | Selected candidate | Main 5 V rail |
| Power board | TPS54302 3.3 V buck regulator | 1 | Selected candidate | Main 3.3 V rail |
| Power board | Input fuse or eFuse | 1 | Recommended | Use a replaceable input fuse or eFuse as the first protection element |
| Power board | Reverse-polarity protection stage | 1 | Recommended | MOSFET ideal-diode style reverse-polarity stage |
| Power board | 12 V TVS diode | 1 | Recommended | Input surge clamp across the 12 V rail |
| Power board | Bulk input capacitor set | 1 set | TBD | Value depends on regulator and load transients |
| Power board | Current-monitoring device | 1 | Recommended | Useful for bring-up, logging, and fault detection |
| Main control board | STM32H743ZIT6 | 1 | Selected | Main real-time controller |
| Main control board | STM32 programming/debug header | 1 | TBD | SWD header format to be chosen |
| Main control board | MCU clock source | 1 | TBD | Crystal or oscillator choice pending |
| Main control board | Decoupling and local bulk capacitors | 1 set | TBD | Determined by schematic and layout |
| Driver board | TI LM3429 LED driver | 2 | Selected candidate | One per LED channel |
| Driver board | LED current-sense resistor network | 2 | TBD | Sized for 1.05 A target |
| Driver board | LED power inductor / diode / switching network | 2 sets | TBD | Final values depend on LM3429 design |
| Driver board | MAX4427CSA+T gate driver | 2 | Selected candidate | One planned for the pump and one for the 2-wire fan PWM stage |
| Driver board | AOD4184A MOSFET | 2 | Selected candidate | One planned for the pump and one for the 2-wire fan PWM stage |
| Driver board | Pump current-sense element | 1 | TBD | Needed for pump fault detection |
| Driver board | Flyback protection for pump path | 1 | TBD | Required for inductive load handling |
| Driver board | Fan connector and PWM power stage | 1 | TBD | 2-wire fan drive stage and wiring interface |
| Driver board | LED thermal sensor | 1-2 | TBD | Depends on thermal monitoring approach |
| Communication board | ESP32-WROOM-32 | 1 | Selected | Main network and MQTT controller |
| Communication board | LAN8720 Ethernet PHY | 1 | Selected | Ethernet interface PHY |
| Communication board | RJ45 connector with magnetics | 1 | TBD | Final footprint and pinout to be selected |
| Communication board | TLV75533PDBVR 3.3 V LDO | 1 | Selected | Local 3.3 V regulation for the custom communication board |
| Communication board | ESP32 programming/debug header | 1 | TBD | UART or other programming interface |
| RS485 interface | SN65LBC176QDRG4 | 1 | Selected candidate | Main RS485 transceiver |
| RS485 interface | RS485 TVS protection | 1 set | Recommended | Recommended for cable protection |
| RS485 interface | RS485 termination resistor | 1 | TBD | 120 ohm endpoint termination |
| RS485 interface | RS485 bias resistor network | 1 set | TBD | Final values depend on bus strategy |
| System I/O | Screw terminals or locking connectors | 1 set | TBD | For power, pump, LEDs, fan, and bus wiring |
| System I/O | Test points | 1 set | TBD | For rails, UART, RS485, and current sense nodes |
| System I/O | Status LEDs | 1 set | TBD | Power, fault, and communication indicators |

## 3. Off-Board / Electro-Mechanical Items

| Item | Qty | Status | Notes |
|---|---:|---|---|
| 12 V DC power supply, 12.5 A | 1 | Selected | Main external supply |
| 12 V DC pump, max 2.2 A | 1 | Selected concept | Exact model not yet listed in this BOM |
| LED strip / LED load, 8.38 V at 1.05 A | 2 | Selected electrical target | Same load assumed on both channels |
| 12 V 2-wire fan, about 120 mm x 120 mm, about 0.55 A | 1 | TBD | Exact model still open |
| SHT41A-AW1B-R2 | 1 | Selected | Air temperature and humidity sensor |
| DFRobot SEN0193 | 1 | Selected | Capacitive soil moisture sensor |
| VEML7700 | 1 | Selected | Ambient light sensor |
| Additional analog sensors | 1 set | TBD | Leave analog-input and power headroom for future expansion |
| RS485 field wiring and connectors | 1 set | TBD | Depends on enclosure and harness design |

## 4. Recommended Protection Strategy

For the first prototype, the power-input protection strategy should be:

- Input connector into a replaceable main fuse or eFuse.
- Reverse-polarity protection using a MOSFET ideal-diode style stage.
- A 12 V TVS diode across the input rail.
- Bulk input capacitance placed near the connector and the two TPS54302 regulators.
- Optional main-rail current sensing for telemetry and easier debugging.

This is a good prototype-friendly balance between safety, simplicity, and board area.

## 5. Items Still Needing Final Selection

- Exact connector families for field wiring.
- Exact fuse, reverse-polarity, TVS, and current-sense part numbers.
- Pump current-sense implementation details.
- Exact 2-wire fan model.
- Final fan PWM switching details, including PWM frequency and any added filtering or suppression.
- LED thermal-sensing implementation details.
- Passive values, compensation parts, and magnetics required by the buck regulators and LED drivers.
- Additional sensor part numbers beyond the current base set.

## 6. Next Step

The next useful refinement would be a schematic-oriented BOM pass where each `TBD` line is replaced with a real manufacturer part number, package, and design quantity.
