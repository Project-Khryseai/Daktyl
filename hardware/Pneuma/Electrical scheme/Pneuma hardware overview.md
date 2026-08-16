# Pneuma Power Supply - Hardware Overview

## 1. System Purpose
**Pneuma** is the dedicated power management and supply board for the Daktyl controller (ESP32-based). It provides battery charging, system safety, low-voltage indication, and a clean stepped-up voltage supply to ensure the Daktyl controller receives stable power until the battery is depleted.

## 2. Core Modules & Bill of Materials (BOM)
*   **Battery:** 3.7V Li-ion Cell.
*   **Charging/Protection:** TP4056 Module (with built-in under-voltage/over-charge protection).
*   **Boost Converter:** Generic DC-DC Step-Up Converter (tuned to 5.0V output).
*   **Transistor (Q1):** 2N3904 (or BC337) NPN Transistor.
*   **Diodes & LEDs:**
    *   1x 1N5819 Schottky Diode (Output reverse-current protection).
    *   1x Blue LED (Main power indicator).
    *   1x Red LED (Low battery warning).
*   **Resistors:** 10kΩ, 2.2kΩ, 1kΩ, 220Ω.
*   **Switches/Safety:** 1x SPST Slide/Toggle Switch, 1x Inline Fuse/Breaker.

---

## 3. Architecture & Functional Blocks
The system flows linearly from left to right: **Power Source -> Management & Safety -> Indicator Logic -> Voltage Step-Up -> Output.**

### Block A: Power Source & Management (TP4056)
*   The 3.7V Li-ion battery connects directly to the `B+` and `B-` pads of the TP4056 module.
*   The TP4056 handles safe USB charging.
*   Power is output to the rest of the board via the TP4056 `OUT+` and `OUT-` pads. This ensures the system utilizes the TP4056's built-in hard-cutoff protection (usually ~2.5V) to prevent battery destruction.

### Block B: Safety & Switching
*   **Fuse:** Placed immediately on the `OUT+` line leaving the TP4056.
*   **Main Switch (SW1):** Placed in series immediately after the Fuse.
*   *Design Note:* Placing the switch *after* the TP4056 ensures that the battery can still be charged via USB when the Daktyl system is turned off. Turning the switch OFF completely kills parasitic drain from the downstream boost converter and indicator circuit.

### Block C: Low-Battery Indicator (Analog Shunt Logic)
This custom circuit taps into the switched VCC and GND lines to monitor the raw battery voltage before it is stepped up. 

*   **Power Indicator:** A Blue LED in series with a 1kΩ resistor sits across VCC and GND. It remains lit as long as SW1 is closed.
*   **Voltage Divider:** A 10kΩ resistor (VCC) and a 2.2kΩ resistor (GND) form a voltage divider. The center node connects to the Base of the NPN Transistor (Q1).
*   **The Trapdoor (Shunt):** A Red LED and a 220Ω current-limiting resistor are connected in series across VCC and GND. However, the Collector of Q1 is tapped between the 220Ω resistor and the Red LED anode. The Q1 Emitter connects to GND.
*   **Logic States:**
    *   **High Battery (4.2V - 3.2V):** The voltage divider supplies >0.65V to the transistor base. Q1 turns ON, creating a low-resistance path to ground. Current bypasses the Red LED, keeping it OFF.
    *   **Low Battery (< 3.2V):** The voltage divider output drops below the ~0.65V base threshold. Q1 turns OFF. The current is forced through the Red LED, turning it ON to warn the user.

### Block D: Voltage Step-Up & Protection
*   The raw, switched battery voltage (now ranging from 4.2V down to 3.2V) enters the `VIN+` and `VIN-` pads of the DC-DC Boost Converter.
*   The Boost Converter forces the voltage up to a stable **5.0V** on the `VOUT+` and `VOUT-` pads.
*   **Output Diode (D1):** A 1N5819 Schottky Diode is placed in series on the `VOUT+` line pointing toward the load. This prevents the Daktyl board from back-feeding power into the boost converter if Daktyl is plugged into a PC via USB.

### Block E: Daktyl Controller Integration (Output)
*   Due to the ~0.3V forward voltage drop of the 1N5819 Schottky diode, the final output delivered to the Daktyl board is approximately **4.7V**.
*   **Critical Wiring Note:** This 4.7V output MUST be connected to the `VIN` or `5V` pin of the ESP32 Daktyl board. It must **never** be connected to the `3V3` pin. The ESP32's onboard voltage regulator (e.g., AMS1117) will take this 4.7V and safely drop it to the 3.3V required by the ESP32 chip.

---

## 4. Pin Connections Summary (For Physical Assembly)

| Pneuma Node | Connected To | Notes |
| :--- | :--- | :--- |
| `B+` / `B-` (TP4056) | Li-ion Battery | Direct connection to raw cell. |
| `OUT+` (TP4056) | Fuse -> Switch 1 (Pin 1) | Main positive feed. |
| Switch 1 (Pin 2) | VCC Rail | Feeds 10kΩ, 220Ω, 1kΩ, and Boost `VIN+`. |
| `OUT-` (TP4056) | GND Rail | Feeds Emitter, 2.2kΩ, LED Cathodes, Boost `VIN-`. |
| `VOUT+` (Boost) | 1N5819 Diode (Anode) | Boosts to 5.0V. |
| 1N5819 Diode (Cathode)| Daktyl `VIN` / `5V` Pin | Drops voltage to safe 4.7V for ESP32 regulator. |
| `VOUT-` (Boost) | Daktyl `GND` Pin | Common ground reference. |
