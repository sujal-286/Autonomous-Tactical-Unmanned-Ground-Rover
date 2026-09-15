# Autonomous Tactical Unmanned Ground Rover

> Final-year Engineering Project  
> Platform: Arduino Uno · Raspberry Pi 4B · Adafruit Motor Shield V1 · 4× BO Motors

---

## Project Overview

The **Autonomous Tactical Unmanned Ground Rover** is a mobile unmanned ground platform designed for remote monitoring, autonomous navigation, obstacle avoidance, environmental sensing, and operator control through a graphical user interface.

The rover supports two primary operating modes:

- **Manual Mode** — the operator drives the rover through the GUI using directional controls and an emergency stop.
- **Autonomous Mode** — the rover independently navigates, detects and avoids obstacles, and continues roaming without continuous operator input.

Three HC-SR04 ultrasonic sensors (front, left, right) provide real-time proximity data used by the obstacle-avoidance system and visualised as a sweep radar in the GUI. A LIS3DH IMU provides accelerometer and heading data used for accurate position tracking and map generation, replacing the earlier dead-reckoning estimate. Environmental monitoring is performed using a DHT22 (temperature and humidity) and MQ-5 (gas) sensor. A machine-learning model running on the Raspberry Pi 4B supports intelligent identification of abnormal environmental patterns. LoRa (RA-02 modules) provides the long-range wireless communication link between the rover and the remote base station.

A custom PySide6 desktop GUI serves as the centralized base-station interface for all rover operations — control, radar, LIDAR-style mapping, environmental monitoring, anomaly alerts, heatmap, and system status.

---

## System Architecture

```
┌──────────────────────────────────────────────┐
│            LAYER 1 — ROVER HARDWARE          │
│                                              │
│  HC-SR04 ×3 (Front / Left / Right)          │
│  LIS3DH IMU (Accelerometer + Heading)        │
│  DHT22 (Temperature + Humidity)              │
│  MQ-5 (Gas)                                  │
│           ↓                                  │
│       Arduino Uno                            │
│  Low-level sensor acquisition                │
│  Motor control · Obstacle avoidance          │
│           ↓  UART / USB Serial               │
│      Adafruit Motor Shield V1                │
│       4× BO DC Gear Motors                   │
└──────────────────┬───────────────────────────┘
                   │ UART / USB Serial
┌──────────────────▼───────────────────────────┐
│         LAYER 2 — HIGH-LEVEL PROCESSING      │
│                                              │
│             Raspberry Pi 4B                  │
│  · Sensor data processing                    │
│  · IMU-based position tracking               │
│  · Map and path visualization                │
│  · ML inference (anomaly detection)          │
│  · Environmental monitoring                  │
│  · GUI backend · TCP server                  │
└──────────────────┬───────────────────────────┘
                   │ LoRa (RA-02)
┌──────────────────▼───────────────────────────┐
│       LAYER 3 — WIRELESS COMMUNICATION       │
│  LoRa modules · Long-range telemetry         │
│  Rover status · Sensor data · Commands       │
└──────────────────┬───────────────────────────┘
                   │
┌──────────────────▼───────────────────────────┐
│       LAYER 4 — REMOTE BASE STATION          │
│                                              │
│  Operator · PySide6 GUI                      │
│  Manual control · Autonomous commands        │
│  Ultrasonic radar · LIDAR-style map          │
│  Environmental data · Anomaly alerts         │
│  Heatmap · ML status · System status         │
└──────────────────────────────────────────────┘
```

---

## Hardware

| Component               | Part                          | Role                                              |
|-------------------------|-------------------------------|---------------------------------------------------|
| Microcontroller         | Arduino Uno                   | Low-level control, sensor acquisition, motor drive |
| Single-board computer   | Raspberry Pi 4B               | High-level processing, ML, GUI backend, TCP server |
| Motor driver            | Adafruit Motor Shield V1      | 4-channel H-bridge, drives all 4 motors           |
| Drive motors            | 4× BO DC gear motors          | Rover locomotion                                  |
| Ultrasonic sensors      | 3× HC-SR04                    | Obstacle detection — front, left, right           |
| IMU                     | LIS3DH                        | Accelerometer + heading for map and navigation    |
| Temperature / Humidity  | DHT22                         | Environmental monitoring                          |
| Gas sensor              | MQ-5                          | Gas-level environmental monitoring                |
| Wireless communication  | RA-02 LoRa modules            | Long-range rover ↔ base-station link              |
| Power supply            | External battery pack (6–9 V) | Motor and electronics power                       |

---

## Software Stack

| Layer          | Technology                                        |
|----------------|---------------------------------------------------|
| Arduino        | C++ · Arduino framework · AFMotor · ArduinoJson   |
| RPi backend    | Python 3 · pyserial · socket · threading          |
| GUI            | Python 3 · PySide6                                |
| ML             | scikit-learn / TensorFlow (on RPi)                |
| Communication  | JSON over TCP (Wi-Fi) + LoRa                      |

---

## Features

### 1. Manual Control
The operator drives the rover from the GUI using directional buttons or arrow keys. An emergency stop button immediately halts all motors. A 600 ms watchdog on the Arduino automatically stops the motors if the connection is lost.

### 2. Autonomous Navigation and Obstacle Avoidance
In autonomous mode the rover roams continuously. Three HC-SR04 sensors provide proximity data. A rule-based avoidance system uses a 15 cm safety threshold: front blockage triggers a reverse and curve manoeuvre; side blockage steers the rover away while continuing forward. The rover resumes forward movement once the path is clear.

### 3. LIS3DH IMU — Navigation and Mapping
The LIS3DH accelerometer and heading sensor is integrated with the RPi to provide accurate position tracking for map generation and navigation. IMU data replaces the earlier timer-based dead-reckoning estimate, improving map accuracy and enabling more reliable autonomous path following.

### 4. Ultrasonic Radar
The GUI displays a real-time sweep radar showing the three sensor beams. Each beam is coloured green up to the detected obstacle distance and red beyond. Obstacle dots persist on the display until the sensor clears. A rotating sweep line provides the visual radar effect.

### 5. LIDAR-style Map
A top-down map rendered in the GUI shows the rover's estimated position, movement path, and detected obstacle cloud. The rover icon is always centred; the world scrolls around it. Obstacle dots fade over time as the rover moves away. Position is computed from IMU heading data and time-based distance estimation.

### 6. Destination-based Path Planning
In autonomous mode the operator selects a destination on the map. The rover computes a route from its current estimated position to the target and follows the planned path while continuously checking for obstacles and adjusting as needed. The path and destination are visualised on the map.

### 7. Environmental Monitoring
DHT22 readings (temperature, humidity) and MQ-5 gas readings are collected, processed on the RPi, and displayed in the GUI. Sensor data is updated in real time.

### 8. Temperature Anomaly Detection
Temperature readings are monitored continuously. When an abnormal condition is identified the system generates an alert displayed in the GUI. Detection is supported by both rule-based thresholds and the ML model.

### 9. Heatmap
Environmental sensor data collected at different rover positions is rendered as a spatial heatmap in the GUI. This allows the operator to identify areas with elevated temperature or gas readings across the traversed area.

### 10. Machine Learning
A machine-learning model trained on environmental data runs locally on the Raspberry Pi. It identifies abnormal environmental patterns and supports the anomaly-detection system, operating without cloud dependency.

### 11. LoRa Communication
RA-02 LoRa modules provide the wireless communication layer between the rover and the base station. Sensor data, rover status, obstacle information, and mode information are transmitted to the base station. Control commands are sent from the base station to the rover.

### 12. PySide6 Base Station GUI
A custom tabbed desktop GUI provides the operator interface:

| Tab      | Contents                                                                 |
|----------|--------------------------------------------------------------------------|
| CONTROL  | Mode selection · D-pad · Obstacle avoidance toggle · E-stop · Link status · Quick sensor readout |
| SENSORS  | 3-beam sweep radar · Large distance displays with colour-coded bars      |
| MAP      | LIDAR-style position map · Path trace · Obstacle cloud · Destination pin |
| CONSOLE  | System log · All connection and status events                            |

---

## Communication Protocol

### GUI → RPi → Arduino

| Message | Effect |
|---|---|
| `{"type":"hello"}` | Handshake initiation |
| `{"type":"drive","vx":-100..100,"vy":-100..100}` | Differential drive command |
| `{"type":"stop"}` | Immediate motor stop |
| `{"type":"mode","mode":"autonomous"}` | Switch to autonomous mode |
| `{"type":"mode","mode":"manual"}` | Switch to manual mode |
| `{"type":"obstacle","enabled":true/false}` | Toggle obstacle avoidance |

### Arduino → RPi → GUI

| Message | Meaning |
|---|---|
| `{"type":"arduino_ready"}` | Arduino boot complete |
| `{"type":"sensors","front":N,"right":N,"left":N}` | Ultrasonic sensor packet (80 ms interval) |
| `{"type":"rover_status","online":true/false}` | Arduino online status |

The RPi attaches a `"ts"` Unix timestamp to every sensor packet before forwarding to the GUI, enabling accurate Δt calculation for position tracking.

---

## Obstacle Avoidance Logic (Autonomous Mode)

```
All sensors reading every 80 ms.

IF front sensor > 15 cm AND sides clear:
    Move forward at full speed.

IF front sensor < 15 cm:
    Stop immediately.
    Reverse for 5.5 s.
    Curve right for 5.5 s (inner wheel at reduced speed, outer at full).
    Resume forward.

IF right sensor < 15 cm (while moving forward):
    Reduce right-side motor speed → rover curves left while continuing.

IF left sensor < 15 cm (while moving forward):
    Reduce left-side motor speed → rover curves right while continuing.

IF both sides < 15 cm:
    Continue straight — front sensor handles any forward blockage.
```

---

## Motor Wiring

| Motor        | Shield Terminal |
|--------------|-----------------|
| Front-left   | M1              |
| Front-right  | M2              |
| Rear-left    | M3              |
| Rear-right   | M4              |

If a motor spins the wrong direction, swap its two wires in the terminal block — no code change required.

---

## Sensor Wiring (HC-SR04)

| Sensor | TRIG | ECHO |
|--------|------|------|
| Front  | A2   | A3   |
| Left   | A4   | A5   |
| Right  | A0   | A1   |

All sensors share the Arduino 5 V rail and GND.

---

## Quick Start

### 1. Arduino
```
Libraries: AFMotor  ArduinoJson
Upload: v3.1/rover_arduino.ino
Connect Arduino to RPi via USB cable.
```

### 2. Raspberry Pi
```bash
pip3 install pyserial
python3 rover_server.py
# Auto-start: enable rover.service via systemd
```

### 3. PC (Base Station)
```bash
pip install PySide6
python gui.py
# Click CONNECT in the toolbar
```

---

## Version History

| Version | Description |
|---------|-------------|
| **v1.1** | Arduino-only basic movement — forward, backward, left, right demo loop |
| **v1.2** | PC GUI (Tkinter) sends single-character commands to Arduino over USB serial |
| **v2.1** | Raspberry Pi introduced as TCP bridge — JSON protocol, PySide6 GUI, watchdog safety stop |
| **v2.2** | HC-SR04 obstacle avoidance added — 3 sensors, 15 cm threshold, real-time sensor JSON stream |
| **v3.1** | Full autonomous mode, sweep radar, LIDAR-style map, LIS3DH IMU navigation, tabbed base-station GUI |
---

*Autonomous Tactical Unmanned Ground Rover — Final Year Engineering Project*
