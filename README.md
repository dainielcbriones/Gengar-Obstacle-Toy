# Gengar Obstacle Toy

![Microcontroller](https://img.shields.io/badge/Microcontroller-ESP32--S3-blue)
![Firmware](https://img.shields.io/badge/Firmware-C%2B%2B-00599C?logo=cplusplus&logoColor=white)
![IDE](https://img.shields.io/badge/IDE-Arduino-00979D?logo=arduino&logoColor=white)
![Sensor](https://img.shields.io/badge/Sensor-HC--SR04-green)
![Display](https://img.shields.io/badge/Display-GC9A01-orange)
[![TRIOE Dashboard](https://img.shields.io/badge/TRIOE-Dashboard-blueviolet)](https://hub.trioe.dev/shared/c6204a66fd938435441db2847fe2ab5e2bafa726bbacab77ad1d0658a43ca833)
![3D Printing](https://img.shields.io/badge/3D%20Printing-Custom%20Enclosure-red)

<img width="2048" height="921" alt="Gengar Obstacle Toy" src="https://github.com/user-attachments/assets/c97d8717-68ff-4fe1-a848-e8d407195234" />

A Gengar-inspired tabletop obstacle detection toy powered by a **TRIOE ESP32-S3**. It uses an **HC-SR04** ultrasonic sensor to detect nearby objects within a 2-meter range and a **GC9A01** round LCD to display smooth, flicker-free animated facial reactions. Its integrated **TRIOE Hub** web dashboard provides real-time cloud monitoring of object detection status, facial reaction state, and measured distance.

---

## Features

- **Filtered Ultrasonic Object Detection** – Reads 5 ultrasonic samples every `100 ms` and applies a median filter for stable distance measurement between `0.02 m` and `2.00 m`.
- **Flicker-Free Animated Gengar Face** – Renders smooth Bezier-curved eyes, dynamic mouth, and teeth at ~33 FPS (`30 ms` refresh interval) on a `240x240` round GC9A01 LCD using `GFXcanvas16`.
- **5-Stage Real-Time Facial Reactions** – Automatically transitions between **Idle**, **Calm**, **Alert**, **Angry**, and **Rage** (complete with a red anime anger symbol and screen shake) based on object proximity.
- **Lifelike Idle Eye Behavior** – Cycles through random eye movements (`IDLE_LOOK`, `IDLE_THINK`, `IDLE_CENTER`, and `IDLE_QUICK_GLANCE`) along with natural blinking when no obstacle is present.
- **Dynamic Mouth & Teeth Animation** – Smoothly interpolates mouth openness and reveals upper and lower fangs as an object approaches closer.
- **TRIOE Cloud Telemetry** – Transmits `distance`, `reaction`, and `object` status to the TRIOE Hub dashboard every `1000 ms` over Wi-Fi.
- **Serial Monitor Diagnostics** – Outputs live distance (`m`), reaction state, detection status, and Wi-Fi connection status every `500 ms` at `115200` baud.
- **Custom 3D-Printed Enclosure** – Houses the electronics inside a compact, Gengar-inspired tabletop shell.

---

## Hardware Components

| Component | Description |
| :--- | :--- |
| **TRIOE ESP32-S3** | Main microcontroller with built-in Wi-Fi |
| **HC-SR04 Ultrasonic Sensor** | Obstacle detection and distance measurement (`0.02 m – 2.00 m`) |
| **GC9A01 Round LCD (1.28")** | `240x240` SPI circular display for animated facial expressions |
| **3D-Printed Enclosure** | Custom Gengar-inspired tabletop body |
| **Connecting Wires** | Jumper wires for SPI, sensor, and power connections |

---

## Pin Configuration

| Component | Pin | ESP32-S3 GPIO |
| :--- | :--- | :--- |
| **HC-SR04** | `TRIG` | `GPIO 1` (`D1`) |
| **HC-SR04** | `ECHO` | `GPIO 2` (`D2`) |
| **GC9A01 LCD** | `SCL` (SCLK) | `GPIO 4` (`D4`) |
| **GC9A01 LCD** | `SDA` (MOSI) | `GPIO 5` (`D5`) |
| **GC9A01 LCD** | `DC` | `GPIO 6` (`D6`) |
| **GC9A01 LCD** | `CS` | `GPIO 7` (`D7`) |
| **GC9A01 LCD** | `RST` | `GPIO 8` (`D8`) |
| **All Components** | `GND` | `GND` |
| **HC-SR04 / GC9A01** | `VCC` | `5V` |

---

## System Overview

The TRIOE ESP32-S3 continuously triggers the HC-SR04 ultrasonic sensor and filters out noise using a 5-reading median filter. Based on the computed distance in meters, the firmware updates the character's emotional state (`FaceState`), animates the eyes, eyebrows, mouth, and shake offset in an off-screen `GFXcanvas16` buffer, pushes the frame to the GC9A01 LCD via SPI, and publishes telemetry packets to TRIOE Hub.

### How It Works

1. **Object Detection (`100 ms`)** – The HC-SR04 takes 5 distance samples and computes the median distance.
2. **State Evaluation** – The ESP32-S3 maps the distance (`0.02 m – 2.00 m`) to one of five reaction states.
3. **Canvas Rendering (`30 ms`)** – Eye tracking, blinking, mouth interpolation, and shake offsets are drawn onto a `240x240` buffer and sent to the GC9A01 LCD.
4. **Serial Logging (`500 ms`)** – Distance, reaction, object status, and Wi-Fi state are printed to the Serial Monitor.
5. **Cloud Telemetry (`1000 ms`)** – Real-time readings (`distance`, `reaction`, `object`) are sent to the TRIOE dashboard over HTTPS.

---

## Facial Reaction System

| Reaction | Distance Range | Visual & Animation Behavior |
| :--- | :--- | :--- |
| **Idle** | `> 2.00 m` (No Object) | Closed mouth, relaxed eyes, natural blinking, and random glancing/thinking eye movements. |
| **Calm** | `0.51 m – 2.00 m` | Slight mouth opening (`20%–35%`), subtle brow line, gentle idle sway. |
| **Alert** | `0.31 m – 0.50 m` | Wider mouth (`65%`) with lower teeth visible and thicker furrowed brows. |
| **Angry** | `0.16 m – 0.30 m` | Wide open mouth (`85%`), steep angry eyebrows, inner brow creases, and faster body shake. |
| **Rage** | `<= 0.15 m` | Maximum mouth opening (`100%`), heavy furrowed brows, flashing red anger mark (`rageSymbolBitmap`), and rapid intense shake. |

---

## TRIOE Web Dashboard

The Gengar Obstacle Toy integrates with the **TRIOE Hub** platform for real-time monitoring of sensor readings and device state.

### Telemetry Keys Sent

- **`distance`** (`meters`) – Current filtered object distance (`0.00` when no object is detected).
- **`reaction`** (`string`) – Current character expression (`IDLE`, `CALM`, `ALERT`, `ANGRY`, `RAGE`).
- **`object`** (`string`) – Detection status (`OBJECT DETECTED` or `NO OBJECT`).

### Access Live Dashboard

- **Dashboard Platform:** TRIOE Hub
- **Project Device:** Gengar Obstacle Toy
- **Live Dashboard:** [![Open TRIOE Dashboard](https://img.shields.io/badge/Open_Dashboard-TRIOE_Hub-blueviolet?style=for-the-badge)](https://hub.trioe.dev/shared/c6204a66fd938435441db2847fe2ab5e2bafa726bbacab77ad1d0658a43ca833)

---

## Software and Firmware

The firmware is written in **C++** using the **Arduino IDE** for the ESP32-S3 platform.

### Required Libraries

- **Adafruit GFX Library** – Core graphics primitives and `GFXcanvas16` off-screen framebuffer.
- **Adafruit GC9A01A Library** – Driver for the `240x240` round TFT LCD.
- **Trioe Library (`Trioe.h` & `TRIOE_ROOT_CA.h`)** – Secure HTTPS telemetry client for TRIOE Hub.
- **WiFi & SPI** – Built-in ESP32 Arduino core libraries.

---

## Getting Started

1. **Prepare the Hardware**
   Gather the TRIOE ESP32-S3, HC-SR04 ultrasonic sensor, GC9A01 round LCD, jumper wires, and the custom 3D-printed Gengar enclosure.

2. **Wire the Components**
   Connect the HC-SR04 and GC9A01 display to the TRIOE ESP32-S3 following the [Pin Configuration](#pin-configuration) table.

3. **Install Libraries in Arduino IDE**
   Install **ESP32 Board Support**, **Adafruit GFX Library**, **Adafruit GC9A01A**, and the **Trioe** library in your Arduino IDE.

4. **Configure Credentials**
   Open `Gengar_Obstacle_Toy.ino` and replace the placeholder values with your own Wi-Fi and TRIOE device credentials:
   ```cpp
   const char* WIFI_SSID     = "YOUR_WIFI_SSID";
   const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";
   const char* TELEMETRY_URL = "[https://hub.trioe.dev/api/v1/devices/YOUR_DEVICE_ID/telemetry/](https://hub.trioe.dev/api/v1/devices/YOUR_DEVICE_ID/telemetry/)";
   const char* API_KEY       = "YOUR_TRIOE_API_KEY";
   ```

5. **Upload the Firmware**
   Select your **ESP32-S3** board and active **COM port** in Arduino IDE, then click **Upload**. Open the Serial Monitor at **`115200` baud** to verify Wi-Fi and TRIOE initialization.

6. **Monitor on TRIOE Hub**
   Open the TRIOE Web Dashboard link above to view real-time distance measurements, object detection alerts, and Gengar's facial reactions.

---

## Project Applications

- Interactive tabletop companion toy
- Ultrasonic obstacle detection demonstration
- Embedded systems & SPI graphics learning project
- Real-time IoT telemetry and cloud dashboard monitoring
- 3D printing and microcontroller prototyping

### Built With

![ESP32-S3](https://img.shields.io/badge/ESP32--S3-E7352C?style=for-the-badge&logo=espressif&logoColor=white)
![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![TRIOE Hub](https://img.shields.io/badge/TRIOE_Hub-6A0DAD?style=for-the-badge)

*Developed as an embedded systems project integrating electronics, firmware development, 3D modeling, and IoT technology.*
