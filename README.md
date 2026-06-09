# Modular Smart Greenhouse Control System

A modular smart greenhouse automation system designed as a learning-focused engineering project covering embedded systems, power electronics, PCB design, control systems, MQTT communication, dashboards, and computer vision.

The goal of this project is not only to automate a greenhouse, but to build a scalable and modular platform that combines real-time embedded control with higher-level monitoring, logging, and AI-based plant observation.

---

## Project Goals

This project aims to control and monitor the main greenhouse subsystems:

* Automated watering
* LED grow light control
* Air cooling and ventilation
* Environmental sensing
* Camera-based monitoring
* MQTT communication
* Node-RED dashboard
* Modular PCB-based hardware
* STM32 firmware development
* Optional AI-based plant health analysis

The system is designed to be modular, expandable, and educational. Each subsystem is treated as an independent engineering block that can be developed, tested, and improved separately.

---

## Main Features

### Watering Control

* Pump control using a custom motor driver
* Soil moisture-based automatic watering
* Manual pump override
* Optional multi-zone watering
* Water tank level detection
* Flow sensor feedback
* Dry-run protection
* Pump current monitoring
* Fault detection and reporting

### Lighting Control

* Custom LED constant-current driver
* PWM dimming
* Manual brightness control
* Scheduled lighting cycles
* Sunrise/sunset ramping
* LED current monitoring
* Thermal protection
* Optional multi-channel grow lighting

### Cooling and Ventilation

* Fan speed control
* Temperature-based cooling
* Humidity-based ventilation
* Manual override
* Optional vent actuator control
* Optional misting or evaporative cooling
* Fault detection

### Environmental Monitoring

Planned sensors include:

* Air temperature
* Air humidity
* Soil moisture
* Soil temperature
* Light intensity
* Water tank level
* Water flow rate
* Optional CO₂ sensing
* Optional pH and EC sensing

### Communication and Dashboard

* MQTT-based communication
* Node-RED dashboard
* Real-time sensor display
* Manual actuator control
* System status monitoring
* Fault reporting
* Optional database logging
* Optional Grafana visualization

### Camera and AI Monitoring

* Camera-based greenhouse monitoring
* Periodic image capture
* Dashboard image display
* Timelapse generation
* Optional plant health detection
* Optional pest/disease detection using AI models

---

## System Architecture

The system is divided into several modular hardware and software blocks.

```text
+----------------------+
|   Node-RED Dashboard |
+----------+-----------+
           |
           | MQTT
           |
+----------v-----------+
|   MQTT Broker        |
|   Mosquitto          |
+----------+-----------+
           |
           | Wi-Fi / Ethernet / UART
           |
+----------v-----------+
| Communication Module |
| ESP32 / Ethernet     |
+----------+-----------+
           |
           | UART / SPI / CAN / RS485
           |
+----------v-----------+
| STM32 Main Control   |
| Real-Time Control    |
+----+----------+------+
     |          |
     |          |
+----v---+  +---v------+
|Sensors |  | Drivers  |
+--------+  +----------+
               |
       +-------+-------+
       |       |       |
     Pump    LEDs    Fans
```

---

## Hardware Modules

### 1. Main Controller PCB

The main controller board contains the STM32 microcontroller and handles real-time control.

Planned functions:

* Sensor reading
* PWM generation
* Pump control logic
* LED control logic
* Fan control logic
* Fault management
* Communication with the ESP32 or gateway module
* Data logging
* Watchdog supervision

Possible MCU options:

* STM32H743
* STM32F4 series
* STM32G4 series

### 2. Power PCB

The power board distributes and protects system power.

Planned features:

* Main DC input
* 12 V / 24 V power distribution
* 5 V buck converter
* 3.3 V regulator
* Fuses or resettable fuses
* Reverse polarity protection
* TVS diode protection
* Current sensing
* Power status LEDs

### 3. Driver PCB

The driver board handles power outputs.

Planned drivers:

* Pump motor driver
* LED constant-current driver
* Fan driver
* Solenoid valve driver
* Optional heater/mister driver

Protection features:

* Flyback diodes
* Gate resistors
* Gate pull-downs
* Current sensing
* Thermal considerations
* Screw terminal outputs
* Test points

### 4. Sensor Interface PCB

The sensor board or sensor interface layer connects environmental sensors to the main controller.

Supported sensor types:

* Analog sensors
* I²C sensors
* 1-Wire sensors
* RS485/Modbus sensors
* Digital switches
* Flow sensors

### 5. Communication Module

The communication module connects the embedded controller to the software layer.

Possible options:

* ESP32-S3 for Wi-Fi and MQTT
* Ethernet module for wired networking
* Raspberry Pi for dashboard, database, and AI processing

---

## Software Architecture

### Embedded Firmware

The STM32 firmware is responsible for real-time operation.

Planned firmware modules:

* GPIO driver
* ADC driver
* PWM/timer driver
* UART driver
* I²C driver
* SPI driver
* Sensor manager
* Pump controller
* LED controller
* Fan controller
* Fault manager
* Communication manager
* Configuration manager
* Watchdog manager
* Data logger

### Communication Layer

MQTT will be used as the main communication protocol between the greenhouse controller and the dashboard/server.

Example MQTT topics:

```text
greenhouse/sensors/air_temperature
greenhouse/sensors/air_humidity
greenhouse/sensors/soil_moisture/zone1
greenhouse/sensors/light_lux
greenhouse/actuators/pump1/state
greenhouse/actuators/led/channel1/brightness
greenhouse/actuators/fan1/speed
greenhouse/system/faults
greenhouse/system/heartbeat
greenhouse/commands/pump1
greenhouse/commands/led/channel1
greenhouse/config/watering_threshold
```

Example MQTT payload:

```json
{
  "value": 42.7,
  "unit": "%",
  "timestamp": "2026-06-09T12:00:00Z",
  "status": "ok"
}
```

### Dashboard

Node-RED will be used to create a local dashboard.

Planned dashboard features:

* Live sensor values
* Pump control
* LED brightness control
* Fan speed control
* System mode selection
* Fault display
* Watering history
* Lighting schedule
* Camera image display

Optional additions:

* InfluxDB for data storage
* Grafana for long-term visualization
* Home Assistant integration

### AI / Vision Layer

The vision layer may run on a Raspberry Pi, Jetson, or mini PC.

Planned features:

* Periodic image capture
* Timelapse generation
* Basic OpenCV image analysis
* Plant growth monitoring
* Leaf color tracking
* Optional pest/disease detection

---

## Development Phases

### Phase 0: Requirements and Planning

* [ ] Define greenhouse size
* [ ] Define supply voltage
* [ ] Define number of watering zones
* [ ] Define number of LED channels
* [ ] Define number of fan channels
* [ ] Define required sensors
* [ ] Create system block diagram
* [ ] Create power budget
* [ ] Create initial BOM
* [ ] Define safety requirements

### Phase 1: Prototype

* [ ] Test STM32 development board
* [ ] Test pump driver
* [ ] Test LED driver prototype
* [ ] Test fan driver
* [ ] Read soil moisture sensor
* [ ] Read air temperature/humidity sensor
* [ ] Publish sensor data using MQTT
* [ ] Create basic Node-RED dashboard
* [ ] Implement manual actuator control

### Phase 2: STM32 Firmware

* [ ] GPIO driver
* [ ] ADC driver
* [ ] PWM driver
* [ ] Timer configuration
* [ ] UART communication
* [ ] I²C sensor communication
* [ ] Watchdog setup
* [ ] Cooperative scheduler
* [ ] Sensor manager
* [ ] Pump control logic
* [ ] LED control logic
* [ ] Fan control logic
* [ ] Fault manager
* [ ] Configuration storage

### Phase 3: Communication and Dashboard

* [ ] MQTT broker setup
* [ ] MQTT topic structure
* [ ] ESP32 communication bridge
* [ ] Node-RED dashboard
* [ ] Manual control commands
* [ ] Sensor data display
* [ ] Fault display
* [ ] Data logging
* [ ] Optional InfluxDB integration
* [ ] Optional Grafana dashboard

### Phase 4: PCB Design

* [ ] Main controller PCB schematic
* [ ] Main controller PCB layout
* [ ] Power PCB schematic
* [ ] Power PCB layout
* [ ] Driver PCB schematic
* [ ] Driver PCB layout
* [ ] Sensor interface PCB schematic
* [ ] Sensor interface PCB layout
* [ ] Add test points
* [ ] Add debug headers
* [ ] Add connector labels
* [ ] Perform ERC/DRC checks
* [ ] Order PCB prototypes
* [ ] Assemble and test boards

### Phase 5: Integration Testing

* [ ] Test power rails
* [ ] Test STM32 boot/debug
* [ ] Test pump output
* [ ] Test LED output
* [ ] Test fan output
* [ ] Test sensor readings
* [ ] Test MQTT communication
* [ ] Test manual dashboard control
* [ ] Test automatic watering
* [ ] Test lighting schedule
* [ ] Test cooling logic
* [ ] Test fault detection
* [ ] Test watchdog recovery

### Phase 6: Camera and AI

* [ ] Connect camera
* [ ] Capture periodic images
* [ ] Display image in dashboard
* [ ] Save timestamped images
* [ ] Create timelapse
* [ ] Implement basic plant color analysis
* [ ] Explore AI model for plant health
* [ ] Explore pest/disease detection

### Phase 7: Enclosure and Deployment

* [ ] Select enclosure
* [ ] Add cable glands
* [ ] Mount PCBs
* [ ] Label connectors
* [ ] Separate power and signal wiring
* [ ] Test in humid environment
* [ ] Add ventilation or sealing
* [ ] Perform long-duration test
* [ ] Document final wiring

---

## Repository Structure

```text
smart-greenhouse/
│
├── README.md
├── docs/
│   ├── system_architecture.md
│   ├── requirements.md
│   ├── mqtt_topics.md
│   ├── wiring.md
│   ├── pcb_notes.md
│   └── test_plan.md
│
├── firmware/
│   ├── stm32_main_controller/
│   │   ├── Core/
│   │   ├── Drivers/
│   │   ├── App/
│   │   └── README.md
│   │
│   └── esp32_comms/
│       ├── src/
│       └── README.md
│
├── hardware/
│   ├── main_controller_pcb/
│   ├── power_pcb/
│   ├── driver_pcb/
│   ├── sensor_interface_pcb/
│   └── mechanical/
│
├── software/
│   ├── node_red/
│   ├── mqtt_broker/
│   ├── dashboard/
│   ├── database/
│   └── ai_vision/
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

## Learning Objectives

This project is designed to build practical experience in:

* STM32 bare-metal programming
* Timer and PWM control
* ADC and sensor reading
* UART, I²C, SPI, RS485, and CAN communication
* Power electronics
* Motor driver design
* LED constant-current driver design
* PCB schematic and layout design
* MQTT communication
* Node-RED dashboard development
* Data logging
* Embedded fault handling
* System integration
* Computer vision
* Edge AI basics
* Greenhouse automation

---

## Future Improvements

* Multi-zone irrigation
* Closed-loop light control
* CO₂ monitoring and control
* Nutrient dosing
* pH and EC monitoring
* Solar power integration
* Battery backup
* Mobile dashboard
* Home Assistant integration
* AI-based plant health scoring
* Predictive watering model
* Remote firmware updates
* Modular plug-and-play sensor nodes

---

## Status

This project is currently in the design and prototyping phase.

Current focus:

* Pump driver design
* LED driver design
* System architecture
* STM32 firmware planning
* Modular PCB structure
* MQTT and Node-RED integration
