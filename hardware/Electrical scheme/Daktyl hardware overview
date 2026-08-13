# Hardware Architecture & Electrical Setup

## 1. System Overview
*   **Microcontroller:** ESP32 (Standard 30-pin layout)
*   **Operating Voltage:** 3.3V Logic
*   **Power Input:** 5V via `VIN` (stepped down to 3.3V by ESP32 onboard regulator)

## 2. Global Power & Filtering
To prevent brownouts during ESP32 Wi-Fi/Bluetooth initialization, the `+3V3` rail includes the following bulk and decoupling capacitors placed in parallel:
*   **C1:** 100nF Ceramic Capacitor (High-frequency decoupling)
*   **C2:** 470uF Electrolytic Capacitor (Bulk energy storage)
*   **C3:** 470uF Electrolytic Capacitor (Bulk energy storage)

## 3. ESP32 Pin Mapping & Net Labels

### Left Header (J1)
| Pin Name | GPIO | Net Label | Function / Notes |
| :--- | :--- | :--- | :--- |
| EN | - | *NC* | No Connect |
| VP | GPIO 36 | `JOY_SW` | Joystick Button (Active HIGH, Ext 10k Pull-down) |
| VN | GPIO 39 | `LADDER_ADC` | 5-Bit Resistor Ladder Input |
| D34 | GPIO 34 | `JOY_X` | Joystick X-Axis (ADC) |
| D35 | GPIO 35 | `JOY_Y` | Joystick Y-Axis (ADC) |
| D32 | GPIO 32 | `ENC2_B` | Rotary Encoder 2 Phase B |
| D33 | GPIO 33 | `ENC2_SW` | Rotary Encoder 2 Button (Active LOW, Int Pull-up) |
| D25 | GPIO 25 | `ENC1_A` | Rotary Encoder 1 Phase A |
| D26 | GPIO 26 | `ENC1_B` | Rotary Encoder 1 Phase B |
| D27 | GPIO 27 | `ENC2_A` | Rotary Encoder 2 Phase A |
| D14 | GPIO 14 | `RGB1_G` | RGB LED 1 (Green) |
| D12 | GPIO 12 | *NC* | No Connect |
| D13 | GPIO 13 | `RGB1_R` | RGB LED 1 (Red) |
| GND | GND | `GND` | System Ground |
| VIN | 5V IN | `+5V` | 5V Power Input |

### Right Header (J2)
| Pin Name | GPIO | Net Label | Function / Notes |
| :--- | :--- | :--- | :--- |
| D23 | GPIO 23 | `ENC1_SW` | Rotary Encoder 1 Button (Active LOW, Int Pull-up) |
| D22 | GPIO 22 | `TRACKPAD_SCL`| I2C Clock (Ext 4.7k Pull-up) |
| TX0 | GPIO 1 | *NC* | No Connect |
| RX0 | GPIO 3 | *NC* | No Connect |
| D21 | GPIO 21 | `TRACKPAD_SDA`| I2C Data (Ext 4.7k Pull-up) |
| D19 | GPIO 19 | `RGB2_B` | RGB LED 2 (Blue) |
| D18 | GPIO 18 | `RGB2_G` | RGB LED 2 (Green) |
| D5 | GPIO 5 | *NC* | No Connect |
| TX2 | GPIO 17 | `RGB2_R` | RGB LED 2 (Red) |
| RX2 | GPIO 16 | `RGB1_B` | RGB LED 1 (Blue) |
| D4 | GPIO 4 | `PIEZO` | Buzzer Output |
| D2 | GPIO 2 | *NC* | No Connect |
| D15 | GPIO 15 | *NC* | No Connect |
| GND | GND | `GND` | System Ground |
| 3V3 | 3.3V OUT | `+3V3` | 3.3V Power Output |

---

## 4. Module Specifications

### A. 5-Bit Resistor Ladder (ADC)
*   **Net Label:** `LADDER_ADC`
*   **Architecture:** 5 parallel switches feeding a single ADC line, utilizing a 22kΩ master pull-down to GND.
*   **Components:**
    *   **SW1 (Push):** Series resistor 10kΩ
    *   **SW2 (Push):** Series resistor 22kΩ
    *   **SW3 (Push):** Series resistor 47kΩ
    *   **SW4 (Push):** Series resistor 100kΩ
    *   **SW5 (SPDT):** Series resistor 220kΩ (Wired to COM and NO; OFF position is completely floating).
    *   **Master Pull-down:** 22kΩ to GND.

### B. Analog Joystick
*   **Type:** 5-pin analog thumbstick module.
*   **Power:** `+3V3` and `GND`.
*   **Logic:**
    *   X and Y axes output directly to ADC pins.
    *   Switch pin (`JOY_SW`) requires a **10kΩ external pull-down resistor** to `GND` to prevent floating states. Button press shorts to `+3V3`.

### C. Rotary Encoders (x2)
*   **Type:** Standard tactile mechanical rotary encoders with push-button (5 pins total).
*   **Logic:** 
    *   Common (C) and Switch Ground (S2) pins are tied directly to `GND`.
    *   A, B, and S1 pins connect directly to ESP32.
    *   *Note:* Requires internal `INPUT_PULLUP` configured in software.

### D. PS4 Trackpad (JDM-050)
*   **Type:** I2C Capacitive Touchpad.
*   **Power:** `+3V3` and `GND`.
*   **Logic:** 
    *   SDA and SCL require **4.7kΩ external pull-up resistors** to `+3V3`.

### E. RGB LEDs (x2)
*   **Type:** Common-Cathode RGB LEDs.
*   **Logic:** Common pin tied to `GND`. Colors driven HIGH via PWM.
*   **Current Limiting Resistors:**
    *   Red: 220Ω
    *   Green: 470Ω
    *   Blue: 330Ω

### F. Piezo Buzzer
*   **Logic:** Driven via PWM on `PIEZO` net.
*   **Hardware:** 100Ω series resistor on the positive terminal to limit current draw from the ESP32 GPIO. Negative terminal tied to `GND`.
