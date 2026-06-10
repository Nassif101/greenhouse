# Requirements

This document defines the Phase 0 requirements for the Modular Smart Greenhouse Control System.

## 1. Prototype Scope

The first version is a compact smart greenhouse prototype intended for learning, testing, and demonstrating a modular embedded automation architecture.

Approximate greenhouse dimensions:

```text
40 cm × 40 cm × 60 cm
```

Approximate internal volume:

```text
0.40 m × 0.40 m × 0.60 m = 0.096 m³
```

## 2. Confirmed System Decisions

| Item | Decision |
|---|---|
| Main supply | 12 V DC external power supply |
| Power supply current | 12.5 A |
| Available power | 150 W |
| Main controller | STM32H743ZIT6 |
| Communication board | ESP32-WROOM-32 + LAN8720 Ethernet PHY |
| Main-to-comm protocol | JSON-lines over UART |
| Sensor bus | Modbus RTU over RS485, initial transceiver candidate SN65LBC176QDRG4 |
| Pump channels | 1 |
| LED channels | 2 |
| Fan channels | 1 |
| Valve channels | 0 |
| Camera | Not included in version 1 |
| Communication board power | 5 V input, custom board with TLV75533PDBVR 3.3 V LDO |
| LED channel target | 8.38 V forward voltage at 1.05 A, about 8.8 W max per channel |
| LED driver candidate | TI LM3429 |
| Fan selection | 12 V 2-wire DC fan, about 120 mm x 120 mm, about 2000 RPM class, about 0.55 A, exact model TBD |
| Main regulators | Two TPS54302 buck regulators, one for 5 V and one for 3.3 V |
| Pump driver candidate | MAX4427CSA+T gate driver + AOD4184A MOSFET |
| Fan driver candidate | MAX4427CSA+T gate driver + AOD4184A MOSFET |

## 3. Functional Requirements

### Watering

The system shall:

- Control one 12 V DC pump.
- Support automatic watering based on soil moisture.
- Support manual pump commands through the dashboard.
- Disable the pump if a low-water condition is detected.
- Disable the pump if overcurrent is detected.
- Optionally detect no-flow conditions using a flow sensor.
- Report pump state and pump faults through MQTT.

### Lighting

The system shall:

- Control two constant-current LED strip channels.
- Target approximately 8.38 V forward voltage at 1.05 A per LED channel.
- Support PWM dimming.
- Support manual brightness commands through the dashboard.
- Support automatic lighting schedules in firmware.
- Monitor LED driver temperature if thermal sensing is implemented.
- Report LED brightness and LED faults through MQTT.

### Cooling

The system shall:

- Control one 12 V 2-wire DC fan using a MOSFET-switched PWM drive stage.
- Generate a PWM control signal for the fan gate driver.
- Support manual and automatic fan-speed control through PWM duty control.
- Detect fan failure where possible using electrical or environmental feedback if implemented.
- Report commanded fan output and fan faults through MQTT.

### Sensing

The system shall support:

- Air temperature sensing.
- Air humidity sensing.
- Soil moisture sensing.
- Soil temperature sensing.
- Light intensity sensing.
- Water tank level sensing.
- Optional flow sensing.
- Some extra analog sensor headroom for future prototype expansion.
- Optional future CO₂, pH, and EC sensing.

Remote sensors shall communicate using Modbus RTU over RS485.

### Communication

The system shall:

- Use the ESP32-WROOM-32 + LAN8720 communication board for Ethernet connectivity.
- Connect to an MQTT broker over Ethernet.
- Publish sensor, actuator, fault, and system-status data.
- Subscribe to command topics.
- Forward dashboard commands to the STM32H7 over UART.
- Use JSON-lines as the first internal protocol format.
- Continue safe local operation if the network or broker is offline.

## 4. Non-Functional Requirements

The system should be:

- Modular.
- Debuggable.
- Expandable.
- Safe under communication failure.
- Easy to test subsystem by subsystem.
- Suitable for PCB-based hardware development.
- Structured enough for GitHub documentation and future portfolio use.

## 5. Safety Requirements

The STM32H7 main controller shall remain responsible for all safety decisions.

The system shall:

- Reject unsafe pump commands.
- Disable the pump on low water level.
- Disable the pump on overcurrent.
- Disable or reduce LED output on overtemperature.
- Detect fan failure where possible.
- Reject invalid MQTT/dashboard commands.
- Continue local control if MQTT, Ethernet, or Node-RED is offline.
- Use watchdog supervision.
- Enter safe shutdown on critical faults.

## 6. System State Machine

The STM32H7 firmware shall use a clear state machine:

```text
INIT
SELF_TEST
IDLE
AUTO_CONTROL
MANUAL_CONTROL
FAULT
SAFE_SHUTDOWN
```

| State | Description |
|---|---|
| INIT | Hardware initialization |
| SELF_TEST | Check sensors, power rails, drivers, and communication |
| IDLE | System ready, no active control action |
| AUTO_CONTROL | Automatic watering, lighting, and cooling |
| MANUAL_CONTROL | Manual dashboard/local commands allowed |
| FAULT | Non-critical fault detected |
| SAFE_SHUTDOWN | Critical fault, actuators disabled |

## 7. Open Requirements

The following still need to be defined or measured:

- Exact 2-wire fan model.
- Exact power-input protection parts and connector families.
- Final list of extra analog sensors beyond the currently selected base set.
