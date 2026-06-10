# Modular Smart Greenhouse Control System

A modular smart greenhouse automation system designed as a learning-focused engineering project covering STM32H7 embedded programming, PCB design, power electronics, motor control, LED driving, Ethernet communication, MQTT, Node-RED dashboards, environmental sensing, and Modbus RTU over RS485.

The goal of this project is not only to automate a greenhouse, but to build a scalable embedded control platform with a clean modular hardware and software architecture.

This project is intentionally over-engineered for learning purposes. Each major function is separated into its own hardware and software block so that the system can be designed, tested, debugged, and expanded in a structured way.

---

## Project Goals

The system is designed to control and monitor the main greenhouse subsystems:

* Automated watering
* LED grow lighting
* Air cooling and ventilation
* Environmental sensing
* Wired Ethernet communication
* MQTT messaging
* Node-RED dashboard
* Modular PCB design
* STM32H7 firmware development
* ESP32 Ethernet communication development
* Modbus RTU sensor communication
* Optional future camera monitoring

---

## Final Version 1 Hardware Scope

The first hardware version will include:

| Subsystem             | Version 1 Decision                    |
| --------------------- | ------------------------------------- |
| Main supply           | External 12 V DC power supply         |
| Main controller       | STM32H7                               |
| Communication board   | ESP32-WROOM-32 + LAN8720 Ethernet PHY |
| Main-to-comm protocol | JSON-lines over UART                  |
| Sensor bus            | Modbus RTU over RS485                 |
| Pump channels         | 1 pump                                |
| LED channels          | 2 LED strips                          |
| Fan channels          | 1 fan                                 |
| Valve channels        | None in version 1                     |
| Camera                | Not included in version 1             |

---

## Main System Architecture

```text
+------------------------------------------------------+
|                External 12 V DC Power Supply         |
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
|  Fan / cooling control                               |
|  RS485 Modbus RTU master                             |
|  Fault handling                                      |
|  Safety state machine                                |
|  Watchdog                                            |
|  Configuration storage                               |
+------------+----------------+------------------------+
             |                |
             |                |
             v                v
+-------------------+   +------------------------------+
| Driver Board      |   | Communication Board           |
|                   |   |                              |
| 1 pump driver     |   | ESP32-WROOM-32                |
| 2 LED drivers     |   | LAN8720 Ethernet PHY          |
| 1 fan driver      |   | RJ45 Ethernet connector       |
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

---

## System Philosophy

The system separates real-time control from networking and dashboard logic.

The STM32H7 main controller is responsible for:

* Reading sensors
* Running control logic
* Controlling actuators
* Handling safety
* Detecting faults
* Executing local autonomous operation
* Validating all commands before actuation

The ESP32 communication board is responsible for:

* Ethernet connectivity
* MQTT publishing
* MQTT command receiving
* Translating MQTT messages to internal UART messages
* Reporting network status
* Handling reconnect logic

The dashboard/server layer is responsible for:

* Visualization
* Manual user commands
* Data logging
* High-level automation
* Optional future image or AI processing

The STM32H7 must remain able to operate safely even if Ethernet, MQTT, Node-RED, or the PC/server goes offline.

---

# Hardware Modules

---

## 1. Power Distribution Board

The power board receives 12 V DC from an external power supply and generates all required system rails.

### Responsibilities

* Accept external 12 V DC input
* Generate regulated 5 V and 3.3 V rails
* Distribute the raw 12 V rail to actuators and driver circuits
* Protect against reverse polarity
* Protect against overcurrent
* Protect against voltage spikes
* Provide fusing or eFuse protection
* Monitor current consumption
* Distribute power to all system boards

### Planned Rails

| Rail  | Purpose                                                  |
| ----- | -------------------------------------------------------- |
| 12 V  | Pump, LED driver input, fan, actuator power              |
| 5 V   | Sensors, communication board input, peripheral modules   |
| 3.3 V | STM32H7, ESP32 logic, digital sensors, low-voltage logic |

### Recommended Protection

The power board should include:

* Main fuse or eFuse
* Reverse polarity protection
* TVS diode on the 12 V input
* Bulk input capacitance
* Buck converter for 5 V
* 3.3 V regulator
* Current monitoring
* Power-good indicators
* Test points for all rails

---

## 2. STM32H7 Main Control Board

The STM32H7 main board is the real-time control brain of the greenhouse.

### Responsibilities

* Run greenhouse control firmware
* Read local sensors
* Communicate with RS485 Modbus RTU sensor nodes
* Generate PWM signals
* Control pump, LED, and fan outputs through the driver board
* Receive commands from the communication board
* Validate all commands before execution
* Detect and handle faults
* Execute safe shutdown behavior
* Store local configuration
* Run watchdog supervision

### Main Board Interfaces

| Interface | Purpose                                         |
| --------- | ----------------------------------------------- |
| PWM       | Pump, LED, and fan control                      |
| ADC       | Soil sensors, current sense, analog sensors     |
| I²C       | Environmental sensors, EEPROM/FRAM, RTC         |
| SPI       | Optional storage or peripheral expansion        |
| UART      | ESP32 communication board link and debug        |
| RS485     | Modbus RTU sensor bus                           |
| GPIO      | Fault inputs, enable pins, buttons, status LEDs |
| SWD/JTAG  | Programming and debugging                       |

### Recommended STM32H7 Firmware Style

The firmware should be structured into clear layers:

```text
Application Layer
├── Greenhouse controller
├── Watering manager
├── Lighting manager
├── Cooling manager
├── Fault manager
├── Safety manager
└── Config manager

Service Layer
├── Scheduler
├── Logger
├── Message queue
├── Command parser
└── Diagnostics

Driver Layer
├── GPIO
├── ADC
├── PWM / timers
├── UART
├── I²C
├── SPI
├── RS485
└── Watchdog
```

The first implementation may use STM32 HAL/LL where useful, while gradually replacing selected drivers with lower-level or bare-metal implementations for learning.

---

## 3. Driver Board

The driver board contains the power electronics used to drive greenhouse actuators.

### Version 1 Driver Channels

| Channel   | Quantity | Driver Type                                      |
| --------- | -------: | ------------------------------------------------ |
| Pump      |        1 | MOSFET motor driver or dedicated DC motor driver |
| LED strip |        2 | Constant-current LED driver                      |
| Fan       |        1 | MOSFET PWM driver                                |
| Valve     |        0 | Not included in version 1                        |

### Responsibilities

* Drive one water pump
* Drive two LED strips
* Drive one fan
* Measure actuator current where needed
* Monitor thermal conditions
* Report driver faults to the STM32H7 main board

### Protection Features

The driver board should include:

* Flyback protection for inductive loads
* Gate resistors
* MOSFET gate pull-downs
* Current sensing
* Thermal sensing near LED drivers
* Fuse or eFuse protection
* Screw terminals
* Status LEDs
* Test points

The driver board should not make autonomous control decisions. It receives commands from the STM32H7 main controller and returns feedback or fault signals.

---

## 4. Communication Board

The communication board connects the greenhouse controller to the local network and MQTT broker.

Version 1 uses:

```text
ESP32-WROOM-32 + LAN8720 Ethernet PHY
```

The ESP32 uses its native Ethernet MAC with an external LAN8720 PHY and RJ45 connector.

A later version may replace the ESP32 communication board with an STM32-based Ethernet communication board using a native STM32 Ethernet MAC and external PHY.

---

### Communication Board Responsibilities

* Provide wired Ethernet connectivity
* Connect to the MQTT broker
* Publish greenhouse data to MQTT topics
* Subscribe to MQTT command topics
* Forward valid commands to the STM32H7 main board
* Receive status, sensor, and fault messages from the STM32H7 main board
* Report communication status
* Handle Ethernet reconnect logic
* Handle MQTT reconnect logic
* Optionally provide network time synchronization
* Optionally store network configuration

---

### ESP32 Ethernet Communication Board Hardware

Main components:

* ESP32-WROOM-32 module
* LAN8720 Ethernet PHY
* RJ45 connector with integrated magnetics or external magnetics
* Ethernet clock source according to the selected LAN8720 design
* 3.3 V regulator
* UART connector to STM32H7 main board
* USB-UART or programming header for ESP32 flashing
* EN/reset button
* BOOT button
* Status LEDs
* ESD protection on Ethernet lines
* Test points for Ethernet, UART, and power rails

### ESP32 Board Interfaces

| Interface | Purpose                                   |
| --------- | ----------------------------------------- |
| RMII      | ESP32 MAC to LAN8720 PHY                  |
| UART      | Communication with STM32H7 main board     |
| GPIO      | PHY reset, status LEDs, ready/fault lines |
| USB-UART  | Programming and debug                     |
| 3.3 V     | ESP32 and PHY supply                      |

---

## 5. Main Board to Communication Board Protocol

The STM32H7 main board and ESP32 communication board communicate over UART.

Version 1 protocol:

```text
JSON-lines over UART
```

Each message is one JSON object followed by a newline.

### Main-to-Comm Messages

The STM32H7 sends:

* Sensor data
* Actuator state
* Fault state
* System status
* Heartbeat
* Configuration values
* Command acknowledgements

Example:

```json
{"type":"sensor","name":"air_temperature","value":24.7,"unit":"C","status":"ok"}
```

Example:

```json
{"type":"actuator","name":"pump1","state":"off","status":"ok"}
```

Example:

```json
{"type":"fault","name":"low_water_level","state":"inactive"}
```

---

### Comm-to-Main Messages

The ESP32 communication board sends:

* Manual commands
* Mode commands
* Configuration updates
* Time synchronization
* Network status
* Broker status

Example:

```json
{"type":"command","id":55,"target":"pump1","state":"on","duration_s":10}
```

Example:

```json
{"type":"command","id":56,"target":"led1","brightness_percent":60,"ramp_time_s":5}
```

Example:

```json
{"type":"command","id":57,"target":"mode","value":"auto"}
```

---

### Command Acknowledgement

The STM32H7 must acknowledge every command.

Accepted command:

```json
{"type":"ack","id":55,"status":"accepted"}
```

Rejected command:

```json
{"type":"ack","id":55,"status":"rejected","reason":"low_water_level"}
```

The communication board forwards acknowledgements to MQTT so Node-RED can display whether a command was accepted or rejected.

---

## 6. Sensor Interface and RS485 Modbus RTU Bus

The sensor interface connects local and remote greenhouse sensors to the STM32H7 main board.

### Supported Sensor Interfaces

* Analog inputs
* I²C
* SPI
* 1-Wire
* Digital inputs
* RS485 Modbus RTU

### Planned Sensors

| Sensor              | Purpose                                  |
| ------------------- | ---------------------------------------- |
| Air temperature     | Climate monitoring                       |
| Air humidity        | Climate monitoring                       |
| Soil moisture       | Watering control                         |
| Soil temperature    | Root zone monitoring                     |
| Light intensity     | Lighting feedback                        |
| Water tank level    | Pump safety                              |
| Flow sensor         | Pump and irrigation feedback             |
| Optional CO₂ sensor | Future ventilation optimization          |
| Optional pH sensor  | Future nutrient monitoring               |
| Optional EC sensor  | Future nutrient concentration monitoring |

### RS485 Bus

Remote sensor boards communicate with the STM32H7 main board using Modbus RTU over RS485.

RS485 connector example:

```text
A
B
GND
+12V or +5V
```

RS485 design notes:

* Use twisted pair wiring for A/B
* Use daisy-chain topology
* Avoid star topology
* Add 120 ohm termination at both ends of the bus
* Add bias resistors where needed
* Add TVS protection for long cables
* Use Modbus RTU addressing for remote sensor nodes

RS485 does not require Ethernet. A regular 4-pin connector, screw terminal, JST, or industrial connector is enough.

---

# Software Architecture

---

## STM32H7 Main Firmware

The STM32H7 firmware is responsible for real-time greenhouse operation.

### Firmware Modules

```text
firmware/
├── app/
│   ├── greenhouse_controller.c
│   ├── watering_manager.c
│   ├── lighting_manager.c
│   ├── cooling_manager.c
│   ├── fault_manager.c
│   ├── safety_manager.c
│   └── config_manager.c
│
├── drivers/
│   ├── gpio_driver.c
│   ├── adc_driver.c
│   ├── pwm_driver.c
│   ├── uart_driver.c
│   ├── i2c_driver.c
│   ├── spi_driver.c
│   ├── rs485_driver.c
│   └── watchdog_driver.c
│
├── services/
│   ├── scheduler.c
│   ├── logger.c
│   ├── command_parser.c
│   ├── message_queue.c
│   └── diagnostics.c
│
└── main.c
```

### STM32H7 Responsibilities

The STM32H7 main controller handles:

* Sensor sampling
* Modbus RTU polling
* Control logic
* Actuator command generation
* Fault detection
* Safety shutdown
* Local automation
* Manual command validation
* Configuration handling
* Watchdog supervision

The STM32H7 must not depend on the dashboard for safe operation.

---

## ESP32 Communication Board Firmware

The ESP32 communication board firmware acts as a bridge between the STM32H7 main board and the MQTT broker.

### Firmware Modules

```text
esp32_ethernet_comm_board/
├── ethernet/
│   ├── ethernet_init.c
│   └── network_status.c
│
├── mqtt/
│   ├── mqtt_client.c
│   ├── mqtt_publish.c
│   ├── mqtt_subscribe.c
│   └── mqtt_reconnect.c
│
├── link/
│   ├── main_uart_link.c
│   ├── json_line_parser.c
│   └── command_forwarder.c
│
├── app/
│   ├── topic_mapper.c
│   ├── heartbeat_manager.c
│   ├── diagnostics.c
│   └── network_config.c
│
└── main.c
```

### ESP32 Responsibilities

The ESP32 communication board handles:

* Ethernet initialization
* DHCP or static IP configuration
* MQTT broker connection
* MQTT reconnect handling
* Publishing STM32H7 data to MQTT topics
* Receiving dashboard commands from MQTT
* Translating MQTT commands into JSON-line UART messages
* Sending commands to the STM32H7 main board
* Reporting network and MQTT status
* Reporting communication-board diagnostics

---

## MQTT Architecture

The greenhouse system uses MQTT for communication with the dashboard and server layer.

### Typical Network Setup

```text
Greenhouse Communication Board
        |
        | Ethernet
        v
MQTT Broker
        |
        v
Node-RED Dashboard
```

The MQTT broker may run on:

* A PC
* A mini PC
* A Raspberry Pi
* A local server
* A Docker container

Node-RED may run on the same machine as the MQTT broker.

---

## MQTT Topic Structure

### Sensor Topics

```text
greenhouse/sensors/air_temperature
greenhouse/sensors/air_humidity
greenhouse/sensors/soil_moisture/zone1
greenhouse/sensors/soil_temperature/zone1
greenhouse/sensors/light_lux
greenhouse/sensors/water_level
greenhouse/sensors/flow_rate
```

### Actuator State Topics

```text
greenhouse/actuators/pump1/state
greenhouse/actuators/led1/brightness
greenhouse/actuators/led2/brightness
greenhouse/actuators/fan1/speed
```

### Command Topics

```text
greenhouse/commands/pump1/set
greenhouse/commands/led1/set
greenhouse/commands/led2/set
greenhouse/commands/fan1/set
greenhouse/commands/mode/set
```

### System Topics

```text
greenhouse/system/status
greenhouse/system/faults
greenhouse/system/heartbeat
greenhouse/system/power
greenhouse/system/network
```

---

## MQTT Payload Examples

### Sensor Payload

```json
{
  "value": 24.7,
  "unit": "C",
  "status": "ok"
}
```

### Pump State Payload

```json
{
  "state": "off",
  "status": "ok"
}
```

### LED State Payload

```json
{
  "brightness_percent": 60,
  "status": "ok"
}
```

### Fan State Payload

```json
{
  "speed_percent": 50,
  "status": "ok"
}
```

### Pump Command Payload

```json
{
  "state": "on",
  "duration_s": 10
}
```

### LED Command Payload

```json
{
  "brightness_percent": 60,
  "ramp_time_s": 5
}
```

### Fan Command Payload

```json
{
  "speed_percent": 70
}
```

### Command Acknowledgement Payload

```json
{
  "id": 55,
  "status": "accepted"
}
```

Rejected command:

```json
{
  "id": 55,
  "status": "rejected",
  "reason": "low_water_level"
}
```

---

## MQTT Message Rules

Recommended rules:

| Message Type         | QoS | Retained |
| -------------------- | --: | -------- |
| Live sensor readings |   0 | No       |
| Heartbeat            |   0 | No       |
| Fault state          |   1 | Yes      |
| Actuator state       |   1 | Yes      |
| Configuration values |   1 | Yes      |
| Manual commands      |   1 | No       |
| Emergency commands   |   1 | No       |

One-shot commands should not be retained. Retained command messages can cause old commands to be executed after reboot.

---

## Node-RED Workflow

Node-RED is used as a visual automation and dashboard server.

It connects to the MQTT broker and can:

* Subscribe to sensor topics
* Display live sensor data
* Display actuator states
* Display faults
* Send manual control commands
* Create basic automation flows
* Store data in a database
* Forward data to Grafana or other tools
* Trigger alerts

### Example Sensor Flow

```text
MQTT input
    |
    v
JSON parser
    |
    v
Dashboard gauge/chart
```

### Example Command Flow

```text
Dashboard button/slider
    |
    v
Command formatter
    |
    v
MQTT output
```

Node-RED should not replace the STM32H7 safety logic. It may request actuator actions, but the STM32H7 decides whether those actions are safe.

---

## System State Machine

The STM32H7 main controller should use a clear system state machine.

```text
INIT
SELF_TEST
IDLE
AUTO_CONTROL
MANUAL_CONTROL
FAULT
SAFE_SHUTDOWN
```

### State Descriptions

| State          | Description                                         |
| -------------- | --------------------------------------------------- |
| INIT           | Hardware initialization                             |
| SELF_TEST      | Sensor, actuator, and communication checks          |
| IDLE           | System ready, no active control action              |
| AUTO_CONTROL   | Local automatic watering, lighting, and cooling     |
| MANUAL_CONTROL | User commands allowed through dashboard/local input |
| FAULT          | Fault detected, affected subsystem disabled         |
| SAFE_SHUTDOWN  | Critical condition, actuators disabled              |

---

## Fault Handling

Planned fault cases:

| Fault                      | Expected Response                |
| -------------------------- | -------------------------------- |
| Low water level            | Disable pump                     |
| Pump overcurrent           | Shut pump down                   |
| No flow detected           | Shut pump down                   |
| LED driver overtemperature | Dim or shut LED down             |
| Fan failure                | Raise warning                    |
| Sensor timeout             | Use fallback mode or raise fault |
| Communication timeout      | Continue local control           |
| Invalid command            | Reject command                   |
| Power rail fault           | Disable affected subsystem       |
| Watchdog reset             | Log reset and restart safely     |

The communication board and dashboard may report faults, but the STM32H7 main controller is responsible for making safety decisions.

---

## Optional Future Camera and AI Architecture

Camera and AI are not included in version 1.

A future version may use PC/server-based image analysis.

```text
Camera
   |
   v
PC / Server
   |
   v
OpenCV / AI model
   |
   v
MQTT results
   |
   v
Node-RED Dashboard
```

Possible future features:

* Periodic image capture
* Timelapse generation
* Leaf color analysis
* Growth tracking
* Pest detection
* Disease detection
* Water stress estimation

Embedded AI may be explored later, but it is not part of the first hardware revision.

---

# Development Phases

---

## Phase 0: Requirements and Planning

* [x] Define greenhouse size: 40 × 40 × 60 cm
* [x] Select external power supply: 12 V, 12.5 A
* [x] Select main controller: STM32H7
* [x] Select communication board: ESP32-WROOM-32 + LAN8720
* [x] Select main-to-comm protocol: JSON-lines over UART
* [x] Select sensor bus: Modbus RTU over RS485
* [x] Define actuator count: 1 pump, 2 LED strips, 1 fan, no valves
* [x] Define pump voltage/current: 12 V, max 2.2 A
* [x] Define fan type: 12 V 4-wire PWM fan
* [x] Define communication board power approach: 5 V input with onboard 3.3 V regulator
* [x] Create initial power budget
* [x] Define safety requirements
* [x] Define system state machine
* [x] Measure LED strip forward voltage
* [x] Define LED strip target current
* [x] Define LED strip maximum power
* [ ] Select exact 3-wire PWM fan
* [x] Select exact STM32H7 part or development board
* [x] Select exact RS485 transceiver
* [x] Select exact 5 V and 3.3 V regulators
* [x] Create initial BOM
* [ ] Create final schematic block diagram


---

## Phase 1: Basic STM32H7 Prototype

* [ ] Select STM32H7 development board
* [ ] Test GPIO
* [ ] Test PWM output
* [ ] Test ADC readings
* [ ] Test I²C sensor reading
* [ ] Test UART JSON-lines transmission
* [ ] Test pump driver
* [ ] Test LED driver prototype
* [ ] Test fan driver
* [ ] Send test data to PC over UART

---

## Phase 2: ESP32 Ethernet Communication Prototype

* [x] Select ESP32-WROOM-32 module
* [x] Select LAN8720 Ethernet PHY module or PCB implementation
* [x] Bring up ESP32 Ethernet
* [x] Connect to local network
* [x] Connect to MQTT broker
* [x] Publish test MQTT messages
* [x] Subscribe to command topics
* [ ] Implement UART link to STM32H7
* [ ] Parse JSON-lines from STM32H7
* [ ] Forward STM32H7 sensor data to MQTT
* [ ] Forward MQTT commands to STM32H7
* [ ] Implement heartbeat
* [ ] Implement reconnect logic

---

## Phase 3: Node-RED and MQTT Dashboard

* [ ] Install Mosquitto MQTT broker
* [ ] Install Node-RED
* [ ] Create sensor dashboard
* [ ] Create pump control widget
* [ ] Create LED1 brightness widget
* [ ] Create LED2 brightness widget
* [ ] Create fan control widget
* [ ] Create system status panel
* [ ] Create fault display
* [ ] Add command acknowledgement display
* [ ] Optional: add database logging
* [ ] Optional: add Grafana dashboard

---

## Phase 4: STM32H7 Firmware Architecture

* [ ] Create STM32H7 firmware project structure
* [ ] Implement GPIO driver
* [ ] Implement ADC driver
* [ ] Implement PWM/timer driver
* [ ] Implement UART driver
* [ ] Implement I²C driver
* [ ] Implement SPI driver
* [ ] Implement RS485 driver
* [ ] Implement Modbus RTU master
* [ ] Implement sensor manager
* [ ] Implement watering manager
* [ ] Implement lighting manager
* [ ] Implement cooling manager
* [ ] Implement fault manager
* [ ] Implement safety manager
* [ ] Implement watchdog
* [ ] Implement configuration storage
* [ ] Implement JSON-lines communication protocol

---

## Phase 5: PCB Design

* [ ] Power board schematic
* [ ] Power board layout
* [ ] STM32H7 main board schematic
* [ ] STM32H7 main board layout
* [ ] Driver board schematic
* [ ] Driver board layout
* [ ] ESP32-WROOM-32 + LAN8720 communication board schematic
* [ ] ESP32-WROOM-32 + LAN8720 communication board layout
* [ ] Sensor interface schematic
* [ ] Sensor interface layout
* [ ] Add debug headers
* [ ] Add test points
* [ ] Add connector labels
* [ ] Run ERC checks
* [ ] Run DRC checks
* [ ] Order prototype PCBs

---

## Phase 6: Integration Testing

* [ ] Test power rails
* [ ] Test STM32H7 boot and debug
* [ ] Test sensor readings
* [ ] Test pump output
* [ ] Test LED1 output
* [ ] Test LED2 output
* [ ] Test fan output
* [ ] Test RS485 Modbus RTU communication
* [ ] Test ESP32 Ethernet communication
* [ ] Test MQTT publishing
* [ ] Test MQTT command receiving
* [ ] Test Node-RED dashboard
* [ ] Test manual control
* [ ] Test automatic watering
* [ ] Test lighting schedule
* [ ] Test cooling logic
* [ ] Test fault handling
* [ ] Test watchdog recovery

---

## Phase 7: Future Camera and Analysis

* [ ] Decide camera type
* [ ] Capture periodic images
* [ ] Store timestamped images
* [ ] Display latest image in dashboard
* [ ] Generate timelapse
* [ ] Run basic OpenCV analysis
* [ ] Publish analysis results over MQTT
* [ ] Explore AI model inference on PC/server

---

## Phase 8: Future STM32 Ethernet Communication Board

* [ ] Select STM32 with native Ethernet MAC
* [ ] Select Ethernet PHY
* [ ] Design STM32 Ethernet comm board
* [ ] Port MQTT gateway firmware
* [ ] Test STM32 Ethernet bring-up
* [ ] Test MQTT connection
* [ ] Validate compatibility with existing STM32H7 main board protocol
* [ ] Compare ESP32 and STM32 communication implementations

---

# Repository Structure

```text
smart-greenhouse/
│
├── README.md
├── docs/
│   ├── system_architecture.md
│   ├── requirements.md
│   ├── power_budget.md
│   ├── mqtt_topics.md
│   ├── node_red_workflow.md
│   ├── communication_board.md
│   ├── rs485_modbus_sensor_bus.md
│   ├── wiring.md
│   ├── pcb_notes.md
│   └── test_plan.md
│
├── firmware/
│   ├── stm32h7_main_controller/
│   │   ├── Core/
│   │   ├── Drivers/
│   │   ├── App/
│   │   └── README.md
│   │
│   ├── esp32_ethernet_comm_board/
│   │   ├── src/
│   │   └── README.md
│   │
│   └── future_stm32_ethernet_comm_board/
│       ├── Core/
│       ├── Drivers/
│       └── README.md
│
├── hardware/
│   ├── power_board/
│   ├── stm32h7_main_controller_board/
│   ├── driver_board/
│   ├── esp32_ethernet_comm_board/
│   ├── future_stm32_ethernet_comm_board/
│   ├── sensor_interface/
│   └── mechanical/
│
├── software/
│   ├── mqtt_broker/
│   ├── node_red/
│   ├── database/
│   ├── dashboard/
│   └── future_vision_analysis/
│
├── tests/
│   ├── firmware_tests/
│   ├── hardware_tests/
│   └── integration_tests/
│
├── media/
│   ├── images/
│   ├── diagrams/
│   └── videos/
│
└── LICENSE
```

---

# Learning Objectives

This project is designed to build practical experience in:

* STM32H7 firmware development
* Bare-metal embedded programming
* Timer and PWM control
* ADC and sensor reading
* UART, I²C, SPI, and RS485 communication
* Modbus RTU
* ESP32 Ethernet development
* LAN8720 Ethernet PHY integration
* MQTT communication
* Node-RED dashboard development
* Power electronics
* Motor driver design
* Constant-current LED driver design
* PCB schematic and layout design
* Fault handling and safety logic
* Modular embedded system architecture
* System integration and testing

---

# Future Improvements

* Multi-zone irrigation
* Valve support
* Advanced lighting profiles
* CO₂ monitoring and control
* Nutrient dosing
* pH and EC sensing
* InfluxDB and Grafana integration
* Home Assistant integration
* Camera-based plant health monitoring
* AI-based pest/disease detection
* Predictive watering model
* Remote firmware updates
* CAN-based modular expansion
* Future STM32 Ethernet communication board

---

# Current Project Status

Current phase:

```text
System architecture and prototype planning
```

Current confirmed decisions:

* 12 V external power supply
* STM32H7 main controller
* ESP32-WROOM-32 communication board
* LAN8720 Ethernet PHY
* MQTT over Ethernet
* Node-RED dashboard
* JSON-lines over UART between STM32H7 and ESP32
* Modbus RTU over RS485 for sensor bus
* One pump
* Two LED strips
* One fan
* No valves in version 1
* No camera in version 1

Current focus:

* Power board architecture
* STM32H7 main controller design
* Pump driver design
* LED driver design
* ESP32 Ethernet communication board
* MQTT and Node-RED communication flow
* Modular PCB structure
