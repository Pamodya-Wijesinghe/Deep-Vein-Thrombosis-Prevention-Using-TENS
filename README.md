# Professional Wearable IoT TENS Device

An industry-standard, mixed-signal, 4-layer PCB architecture designed for a calf-worn Transcutaneous Electrical Nerve Stimulation (TENS) medical-grade device. This system balances precise closed-loop low-side current regulation, robust galvanic safety isolation barriers, multi-point visual user dashboards, and cloud database telemetry sync utilizing the **ESP32-S3-MINI-1-N8** microcontroller module.

---

## 🛠 System Architecture & Subsystems

The hardware is structurally split into two primary electrical domains separated by a continuous **3mm physical isolation gap** across all layers to ensure absolute patient safety and satisfy stringent high-voltage compliance requirements.

### 1. Digital Control & Power Management Domain (Logic Side)
* **Main Controller:** ESP32-S3-MINI-1-N8 (Dual-core Xtensa LX7 running up to 240MHz, 2.4GHz Wi-Fi, BLE 5.0, internal USB-JTAG engine).
* **Power Infrastructure:** Linear Lithium-Polymer battery charging system (`MCP73831` / `TP4056`) combined with a precise `MAX17048` Fuel Gauge IC via local I2C telemetry tracking. Raw battery voltage is stepped down cleanly via a primary high-efficiency `MP1584` buck regulator before being smoothed into an ultra-quiet `+3V3_LOGIC` rail by a dedicated `AMS1117-3.3` LDO layout.
* **Firmware Deployment:** Direct code flashing via the module's native USB hardware pins (`USB_D+` / `USB_D-`), safely passing through a dedicated low-capacitance `USBLC6-2SC6` ESD protection array. Requires no external USB-to-UART bridge silicon or separate programming debuggers.
* **User Dashboard Interface:** Localized I2C monochromatic OLED screen assembly combined with three low-noise vertical control potentiometers mapped to safe, non-strapping `ADC1` channels to handle real-time pulse adjustment seamlessly.

### 2. High-Voltage Output & Stimulation Domain (Isolated Side)
* **Isolated Compliance Rail:** High-isolation medical-grade Traco Power converter configuration (`TIM 2-1224`) supplying a true **24V compliance ceiling** to overcome high skin-contact impedances up to 30mA. Features an active hardware interlock loop: the converter's `REMOTE` shutdown pin is tied via a small-signal N-channel MOSFET gate switch to a dedicated microcontroller line, granting immediate firmware override shutdown authority.
* **Biphasic Waveform Generation:** Fully integrated H-Bridge driver (`DRV8876PWPR`) hardware-strapped into **IN1/IN2 PWM control logic** (`PMODE` pulled directly to `ISO 3.3V`) and manual external loop tracking (`IMODE` grounded). 
* **Precision Closed-Loop Sink:** High-performance, low-offset precision operational amplifier configuration (`OPA2192IDR` e-trim series) monitoring dynamic current feedback across a premium $5\Omega$ $\pm$0.1% low-drift sense resistor. It accurately shapes the stimulation profile by matching the op-amp command voltage to a 16-bit precision voltage DAC (`DAC80501ZDGSR`) over an isolated bidirectional I2C bridge (`ISO1640DWR`).

---

## 🗺 Pin Mapping & Hardware Wiring Matrix

### Core Infrastructure & Analog Inputs
| ESP32-S3 Pin | Net Name | Functional Connection Topography & Filtering Passives |
| :--- | :--- | :--- |
| **Pin 3 (EN)** | `MCU_EN` | Direct trace route to manual momentary reset push-button (`SW_RESET`). No external $RC$ delay stack (module contains internal pre-shielded parameters). |
| **Pad 4 (IO1)** | `KNOB_PULSE_WIDTH` | Potentiometer 1 Wiper $\rightarrow$ $1\text{ k}\Omega$ series resistor $\rightarrow$ $100\text{ nF}$ ceramic shunt $\rightarrow$ Logic Ground. |
| **Pad 5 (IO2)** | `KNOB_FREQUENCY` | Potentiometer 2 Wiper $\rightarrow$ $1\text{ k}\Omega$ series resistor $\rightarrow$ $100\text{ nF}$ ceramic shunt $\rightarrow$ Logic Ground. |
| **Pad 6 (IO3)** | `KNOB_AMPLITUDE` | Potentiometer 3 Wiper $\rightarrow$ $1\text{ k}\Omega$ series resistor $\rightarrow$ $100\text{ nF}$ ceramic shunt $\rightarrow$ Logic Ground. |
| **Pad 7 (IO4)** | `I2C_LOGIC_SDA` | Shared Local I2C Data bus connecting local OLED display, `MAX17048` gauge, and `ISO1640` Pin 3. Pulled up via **1x $4.7\text{ k}\Omega$ resistor** to `+3V3_LOGIC`. |
| **Pad 8 (IO5)** | `I2C_LOGIC_SCL` | Shared Local I2C Clock bus connecting local OLED display, `MAX17048` gauge, and `ISO1640` Pin 2. Pulled up via **1x $4.7\text{ k}\Omega$ resistor** to `+3V3_LOGIC`. |

### Flashing, Safety Loops, & Stimulation Control
| ESP32-S3 Pin | Net Name | Functional Connection Topography & Filtering Passives |
| :--- | :--- | :--- |
| **Pad 12 (IO19)** | `USB_D-` | Native USB Data- link routing directly through Channel 2 of the `USBLC6-2SC6` ESD array to the USB-C Connector pins A7/B7. |
| **Pad 13 (IO20)** | `USB_D+` | Native USB Data+ link routing directly through Channel 1 of the `USBLC6-2SC6` ESD array to the USB-C Connector pins A6/B6. |
| **Pad 15 (IO14)** | `HV_KILL_SWITCH` | Active active safety line running to the gate of an N-channel MOSFET that clamps the `TIM 2-1224` REMOTE line to disable the high-voltage rail. |
| **Pad 9 (IO6)** | `MCU_PWM_IN1` | High-speed stimulation timing PWM line A running directly into Pin 3 (`INA`) of the `ISO7741` quad digital isolator. |
| **Pad 10 (IO7)** | `MCU_PWM_IN2` | High-speed stimulation timing PWM line B running directly into Pin 4 (`INB`) of the `ISO7741` quad digital isolator. |

---

## 📐 4-Layer PCB Layout Rules & Fabrication Constraints

To preserve extreme analog accuracy and safely contain high-voltage transients on a compact wearable profile, your Altium Designer layout constraints must implement this layered controlled stackup:
