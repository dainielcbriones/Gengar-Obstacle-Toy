# Gengar Obstacle Toy

![Microcontroller](https://img.shields.io/badge/Microcontroller-ESP32--S3-blue)
![Firmware](https://img.shields.io/badge/Firmware-C%2B%2B-orange)
![IDE](https://img.shields.io/badge/IDE-Arduino-00979D)
![Sensor](https://img.shields.io/badge/Sensor-HC--SR04-green)
![Display](https://img.shields.io/badge/Display-GC9A01-purple)
![Dashboard](https://img.shields.io/badge/Dashboard-TRIOE-blueviolet)
![3D Printing](https://img.shields.io/badge/3D%20Printing-Custom%20Enclosure-lightgrey)

<img width="2048" height="921" alt="Gengar Obstacle Toy" src="https://github.com/user-attachments/assets/c97d8717-68ff-4fe1-a848-e8d407195234" />

A Gengar-inspired tabletop obstacle detection toy powered by a TRIOE ESP32-S3. It uses an HC-SR04 ultrasonic sensor to detect nearby objects and a GC9A01 round LCD to display visual reactions. Its web dashboard provides real-time monitoring of object detection status, reaction, and distance. The project combines 3D printing, electronics, sensors, and web-based monitoring.

---

## Features

- **Ultrasonic Object Detection** – Detects nearby objects using an HC-SR04 ultrasonic sensor within a 2-meter range.
- **Animated Gengar-Inspired Face** – Displays a Gengar-inspired character on a 1.28-inch round GC9A01 LCD.
- **Real-Time Facial Reactions** – Automatically changes expressions based on object distance, including Calm, Alert, Angry, Rage, and Idle.
- **Automatic Eye Movement** – Features natural eye movements, random glances, and blinking animations.
- **Dynamic Mouth Animation** – Adjusts mouth animations according to the detected object's distance.
- **Real-Time Distance Monitoring** – Measures the distance between the sensor and nearby objects.
- **Wi-Fi Connectivity** – Connects to a wireless network for cloud data transmission.
- **TRIOE Dashboard Integration** – Sends sensor readings and detection information to the TRIOE cloud dashboard.
- **Object Detection Status** – Provides real-time updates indicating whether an object is detected.
- **Automatic Sensor Updates** – Continuously reads sensor data every 100 milliseconds.
- **Serial Monitor Monitoring** – Displays distance measurements, facial reactions, object status, and Wi-Fi connection.
- **3D-Printed Enclosure** – Features a custom-designed enclosure that houses the electronics and gives the project its Gengar-inspired appearance.

---

## Hardware Components

| Component | Description |
|---|---|
| TRIOE ESP32-S3 | Main microcontroller |
| HC-SR04 Ultrasonic Sensor | Obstacle detection and distance measurement |
| GC9A01 Round LCD | Displays animated facial expressions |
| 3D-Printed Enclosure | Custom Gengar-inspired body |
| Connecting Wires | Electrical connections |

---

## Pin Configuration

| Component | Pin | ESP32-S3 |
|---|---|---|
| HC-SR04 | TRIG | D1 |
| HC-SR04 | ECHO | D2 |
| GC9A01 | SCL | D4 |
| GC9A01 | SDA | D5 |
| GC9A01 | DC | D6 |
| GC9A01 | CS | D7 |
| GC9A01 | RST | D8 |
| All Components | GND | GND |
| HC-SR04 / LCD | VCC | 5V |

---

## System Overview

The Gengar Obstacle Toy uses an ultrasonic sensor to continuously measure the distance of nearby objects. The TRIOE ESP32-S3 processes the sensor readings and determines the appropriate facial expression based on the detected distance.

The GC9A01 round LCD displays animated facial reactions, including eye movements, blinking, and mouth animations. At the same time, the ESP32-S3 connects to Wi-Fi and sends real-time sensor information to the TRIOE dashboard.

### How It Works

1. **Object Detection** – The HC-SR04 sensor detects nearby objects.
2. **Distance Measurement** – The ESP32-S3 processes the distance readings.
3. **Facial Reaction** – The LCD automatically displays an expression based on the object's distance.
4. **Animation** – The character performs eye movements, blinking, and mouth animations.
5. **Dashboard Monitoring** – Sensor readings and object detection status are transmitted to the TRIOE dashboard.

---

## Facial Reaction System

| Reaction | Description |
|---|---|
| Idle | Default character expression |
| Calm | Displays a relaxed expression |
| Alert | Reacts to a nearby object |
| Angry | Displays an angry expression |
| Rage | Displays an intense reaction when an object is extremely close |

---

## Software and Firmware

The project uses Arduino IDE to program the TRIOE ESP32-S3 microcontroller.

### Main Technologies

- **Arduino IDE** – Firmware development and uploading.
- **C++** – Microcontroller programming.
- **ESP32-S3** – Main processing and Wi-Fi connectivity.
- **Adafruit GFX Library** – Graphics rendering.
- **Adafruit GC9A01A Library** – Round LCD display control.
- **TRIOE Dashboard** – Real-time sensor monitoring and data visualization.

---

## Getting Started

### 1. Prepare the Hardware

Gather the required electronic components, including the TRIOE ESP32-S3, HC-SR04 ultrasonic sensor, and GC9A01 round LCD.

### 2. Assemble the Components

Connect the ultrasonic sensor and round LCD to the TRIOE ESP32-S3 according to the pin configuration.

### 3. Prepare the Software

Install Arduino IDE and the required libraries for the ESP32-S3 and GC9A01 display.

### 4. Upload the Firmware

Open the Arduino project, configure the required settings, select the correct ESP32-S3 board and COM port, and upload the firmware.

### 5. Connect to the Dashboard

Configure the Wi-Fi connection and TRIOE dashboard integration to monitor real-time distance measurements, facial reactions, and object detection status.

---

## Project Applications

- Interactive tabletop toy
- Ultrasonic obstacle detection demonstration
- Embedded systems learning project
- Sensor and display integration
- IoT and real-time dashboard monitoring
- 3D printing and electronics prototyping

---

## Project Overview

The Gengar Obstacle Toy demonstrates the integration of embedded systems, ultrasonic sensing, animated LCD graphics, wireless communication, and 3D printing.

It combines interactive character animations with real-time obstacle detection and dashboard monitoring in a compact tabletop device.

### Built With

![ESP32](https://img.shields.io/badge/ESP32--S3-Microcontroller-blue)
![C++](https://img.shields.io/badge/C%2B%2B-Programming-orange)
![Arduino](https://img.shields.io/badge/Arduino-IDE-00979D)
![IoT](https://img.shields.io/badge/IoT-Wi--Fi-green)
![3D Printing](https://img.shields.io/badge/3D%20Printing-Enclosure-lightgrey)

---

**Developed as an embedded systems project integrating electronics, firmware development, 3D modeling, and IoT technology.**
