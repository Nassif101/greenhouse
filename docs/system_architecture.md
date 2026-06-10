# System Architecture

This document describes the final Version 1 architecture of the Modular Smart Greenhouse Control System.

## 1. Architecture Overview

The system is divided into five main hardware blocks:

1. Power Distribution Board
2. STM32H7 Main Control Board
3. Driver Board
4. ESP32 Ethernet Communication Board
5. Sensor Interface / RS485 Modbus RTU Bus

The system uses an external 12 V DC power supply. The power board distributes 12 V and generates regulated 5 V and 3.3 V rails.

The STM32H7 main controller handles real-time control and safety. The ESP32 communication board handles Ethernet and MQTT.

## 2. High-Level Block Diagram

```text
+------------------------------------------------------+
|                External 12 V DC Power Supply         |
|                12 V, 12.5 A                          |
+---------------------------+--------------------------+
                            |
                            v
+------------------------------------------------------+
|                Power Distribution Board              |
|                                                      |
|  Input protection                                    |
|  Reverse polarity protection                         |
|  Fuse / eFuse protection                             |
|  TVS surge protection                                |
|  Current monitoring                                  |
|                                                      |
|  Outputs:                                            |
|  - 12 V rail                                         |
|  - 5 V rail                                          |
|  - 3.3 V rail                                        |
+---------------------------+--------------------------+
                            |
                            v
+------------------------------------------------------+
|                STM32H7 Main Control Board            |
|                                                      |
|  Real-time greenhouse logic                          |
|  Sensor reading                                      |
|  Pump control                                        |
|  LED control                                         |
|  Fan control                                         |
|  RS485 Modbus RTU master                             |
|  Fault handling                                      |
|  Safety state machine                                |
|  Watchdog                                            |
+------------+----------------+------------------------+
             |                |
             |                |
             v                v
+-------------------+   +------------------------------+
| Driver Board      |   | Communication Board           |
|                   |   |                              |
| 1 pump driver     |   | ESP32-WROOM-32                |
| 2 LED drivers     |   | LAN8720 Ethernet PHY          |
| 1 fan interface   |   | RJ45 Ethernet connector       |
| Current feedback  |   | MQTT client                   |
| Thermal feedback  |   | UART link to STM32H7          |
+-------------------+   +---------------+--------------+
                                         |
                                         |
                                         v
                                +----------------+
                                | MQTT Broker    |
                                | Mosquitto      |
                                +----------------+
                                         |
                                         v
                                +----------------+
                                | Node-RED       |
                                | Dashboard      |
                                +----------------+
```

## 3. Power Distribution Board

The power board receives the external 12 V supply and provides the required system rails.

### Inputs

```text
12 V DC, up to 12.5 A
```

### Outputs

| Output | Purpose |
|---|---|
| 12 V | Pump, LED drivers, fan |
| 5 V | Communication board, sensors, peripheral modules |
| 3.3 V | STM32H7, logic, low-voltage sensors |

### Required Features

- Main input protection.
- Reverse polarity protection.
- Fuse or eFuse.
- TVS diode.
- 5 V buck converter.
- 3.3 V regulator.
- Current monitoring.
- Power-good indicators.
- Test points.

## 4. STM32H7 Main Control Board

The STM32H7 is the real-time controller.

### Responsibilities

- Sensor acquisition.
- Modbus RTU polling over RS485.
- Pump control.
- LED dimming and scheduling.
- Fan PWM control.
- Fault detection.
- Safety decision-making.
- Communication with ESP32 board over UART.
- Watchdog handling.
- Configuration storage.

### Important Rule

The STM32H7 must remain the authority for all actuator decisions.

Node-RED and MQTT may request actions, but the STM32H7 decides whether an action is safe.

## 5. Driver Board

The driver board controls the actuators.

### Version 1 Channels

| Actuator | Quantity | Notes |
|---|---:|---|
| Pump | 1 | 12 V, max 2.2 A, driver designed for at least 4 A |
| LED strips | 2 | Constant-current loads, 21 LEDs each |
| Fan | 1 | 12 V 4-wire PWM fan |
| Valves | 0 | Not included in version 1 |

### Driver Board Features

- Pump driver.
- Two constant-current LED driver channels.
- 4-wire fan interface.
- Current sensing where useful.
- LED thermal monitoring.
- Flyback protection for inductive loads.
- Fault feedback to STM32H7.
- Screw terminal or locking connectors.
- Test points.
- Status LEDs.

## 6. ESP32 Ethernet Communication Board

The communication board handles networking and MQTT.

### Version 1 Hardware

```text
ESP32-WROOM-32 + LAN8720 Ethernet PHY
```

### Responsibilities

- Ethernet initialization.
- DHCP or static IP configuration.
- MQTT broker connection.
- MQTT publish/subscribe.
- Reconnect handling.
- JSON-lines UART communication with STM32H7.
- Forwarding STM32H7 data to MQTT.
- Forwarding MQTT commands to STM32H7.
- Reporting network and broker status.

### Hardware Blocks

- ESP32-WROOM-32.
- LAN8720 Ethernet PHY.
- RJ45 connector with magnetics.
- 3.3 V local regulator.
- 5 V input from power board.
- UART connector to STM32H7.
- EN/reset button.
- BOOT button.
- Programming/debug header.
- Status LEDs.
- ESD protection.

## 7. Main-to-Comm Protocol

The STM32H7 and ESP32 communicate using JSON-lines over UART.

Each message is:

```text
one JSON object + newline
```

### Example Sensor Message

```json
{"type":"sensor","name":"air_temperature","value":24.7,"unit":"C","status":"ok"}
```

### Example Command Message

```json
{"type":"command","id":55,"target":"pump1","state":"on","duration_s":10}
```

### Example Acknowledgement

```json
{"type":"ack","id":55,"status":"accepted"}
```

### Example Rejection

```json
{"type":"ack","id":55,"status":"rejected","reason":"low_water_level"}
```

## 8. MQTT Architecture

The ESP32 communication board is the MQTT client.

### Data Path

```text
STM32H7
  |
  | UART JSON-lines
  v
ESP32 Communication Board
  |
  | MQTT over Ethernet
  v
Mosquitto Broker
  |
  v
Node-RED Dashboard
```

### Command Path

```text
Node-RED Dashboard
  |
  v
Mosquitto Broker
  |
  v
ESP32 Communication Board
  |
  | UART JSON-lines
  v
STM32H7
```

## 9. MQTT Topics

### Sensors

```text
greenhouse/sensors/air_temperature
greenhouse/sensors/air_humidity
greenhouse/sensors/soil_moisture/zone1
greenhouse/sensors/soil_temperature/zone1
greenhouse/sensors/light_lux
greenhouse/sensors/water_level
greenhouse/sensors/flow_rate
```

### Actuator States

```text
greenhouse/actuators/pump1/state
greenhouse/actuators/led1/brightness
greenhouse/actuators/led2/brightness
greenhouse/actuators/fan1/speed
```

### Commands

```text
greenhouse/commands/pump1/set
greenhouse/commands/led1/set
greenhouse/commands/led2/set
greenhouse/commands/fan1/set
greenhouse/commands/mode/set
```

### System

```text
greenhouse/system/status
greenhouse/system/faults
greenhouse/system/heartbeat
greenhouse/system/power
greenhouse/system/network
```

## 10. Sensor Interface and Modbus RTU

Remote sensors use Modbus RTU over RS485.

### RS485 Connector

```text
A
B
GND
+12V or +5V
```

### Bus Rules

- Use twisted pair for A/B.
- Use daisy-chain topology.
- Avoid star topology.
- Add 120 ohm termination at both ends.
- Add bias resistors where needed.
- Add TVS protection for long cables.
- Assign unique Modbus slave addresses.

## 11. Node-RED Role

Node-RED runs on a PC, mini PC, server, Raspberry Pi, or Docker container in the same network.

Node-RED is used for:

- Dashboard visualization.
- Manual control widgets.
- Fault display.
- Command acknowledgement display.
- Optional data logging.
- Optional automation flows.

Node-RED does not replace STM32H7 safety logic.

## 12. Future Expansion

Possible future expansions:

- Camera monitoring.
- PC/server-based OpenCV or AI analysis.
- Database logging with InfluxDB.
- Grafana visualization.
- Valve channels.
- Multi-zone watering.
- STM32 Ethernet communication board.
- CAN-based modular expansion.
