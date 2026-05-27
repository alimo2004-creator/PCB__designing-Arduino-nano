# Custom Arduino Nano (ATmega328P) Development Board

An open-source, high-reliability development board following the **Arduino Nano** form factor. This design replaces standard low-cost USB-to-UART clones with a dedicated **FTDI FT232RL** IC and introduces an optimized dual-input power auto-selection circuit, making it ideal for robust embedded systems prototyping and hardware-in-the-loop validation.

---

## 🛠️ System Architecture

The hardware design is modularized into distinct functional blocks to ensure clean signal paths and simplified debugging:

### 1. Core Processing Unit
* **Microcontroller:** ATmega328P-AU (TQFP-32 package).
* **Clock Configuration:** 16 MHz external crystal oscillator (`X1`) stabilized with dual 18 pF load capacitors (`C2`, `C3`).
* **Hardware Reset:** Manual tactile button (`S1`) tied to a $1\text{ k}\Omega$ pull-up resistor (`R2`) and filtered for automatic IDE code deployment.
* **Status Indicator:** Programmatic user LED (`D2`, Red) mapped to digital pin `D13/SCK` via a $1\text{ k}\Omega$ current-limiting resistor (`R5`).

### 2. FTDI USB-to-UART Bridge
* **Primary IC:** FT232RL (`U2`) handles high-speed asynchronous serial communication.
* **Automatic DTR Reset:** A 100 nF series capacitor (`C7`) links the FTDI `DTR#` pin to the MCU hardware `RESET` line to enable seamless serial bootloading.
* **Bus Monitoring:** * **TX LED (`D3`, Red):** Driven by `CBUS0` with a $330\ \Omega$ resistor to signal outbound data.
    * **RX LED (`D4`, Green):** Driven by `CBUS1` with a $330\ \Omega$ resistor to signal inbound data.

### 3. Power Distribution Network (PDN)
* **Voltage Regulation:** Onboard 5V linear regulator (`REG1`) drops external raw inputs (`VCC`/`VIN`) to a stable $+5	ext{V}$ logic supply, monitored by a Blue Power LED (`D1`).
* **Auto-Select Switch:** A Schottky diode (`D5`) isolates the $+5	ext{V}$ system rail from the USB power line (`VUSB`), safely allowing simultaneous connection of USB debugging and external supplies without cross-powering backflows.
* **Decoupling Matrix:** Multi-stage filtering utilizes parallel $4.7\ \mu	ext{F}$ tantalum bulk capacitors and 100 nF ceramic bypass capacitors on both the main $+5	ext{V}$ rail (`C4`, `C5`) and the `VUSB` input (`C8`, `C9`).

---

## 📌 Board Pinout & Interface Mapping

The interface is exposed via two 15-pin header blocks (`S4` and `S5`) alongside a standard 6-pin ICSP header (`S3`) for ISP programmer attachment.

### Left Port (S4)
| Pin | Net Name | Primary Description | Alternative Function |
| :---: | :--- | :--- | :--- |
| 1 | `D1/Tx` | Digital I/O 1 | Hardware UART TXD |
| 2 | `D0/Rx` | Digital I/O 0 | Hardware UART RXD |
| 3 | `RESET` | Master System Reset | Active-Low Input |
| 4 | `GND` | Ground Reference | System Low Rail |
| 5 | `D2` | Digital I/O 2 | External Interrupt `INT0` |
| 6 | `D3` | Digital I/O 3 | PWM Output / Interrupt `INT1` |
| 7 | `D4` | Digital I/O 4 | GPIO |
| 8 | `D5` | Digital I/O 5 | PWM Output |
| 9 | `D6` | Digital I/O 6 | PWM Output |
| 10 | `D7` | Digital I/O 7 | GPIO |
| 11 | `D8` | Digital I/O 8 | GPIO |
| 12 | `D9` | Digital I/O 9 | PWM Output |
| 13 | `D10` | Digital I/O 10 | PWM Output / SPI Slave Select (`SS`) |
| 14 | `D11/MOSI` | Digital I/O 11 | PWM Output / SPI Controller Out (`MOSI`) |
| 15 | `D12/MISO` | Digital I/O 12 | SPI Controller In (`MISO`) |

### Right Port (S5)
| Pin | Net Name | Primary Description | Alternative Function |
| :---: | :--- | :--- | :--- |
| 1 | `VCC` | Raw External Input Voltage | Unregulated Supply Input (VIN) |
| 2 | `GND` | Ground Reference | System Low Rail |
| 3 | `RESET` | Master System Reset | Parallel Reset Common |
| 4 | `+5V` | Regulated 5V Rail | Output / Regulated Bus Input |
| 5 | `A7` | Pure Analog Input 7 | ADC Channel 7 (Input Only) |
| 6 | `A6` | Pure Analog Input 6 | ADC Channel 6 (Input Only) |
| 7 | `A5` | Analog Input 5 | $I^2C$ Serial Clock (`SCL`) |
| 8 | `A4` | Analog Input 4 | $I^2C$ Serial Data (`SDA`) |
| 9 | `A3` | Analog Input 3 | ADC Channel 3 / GPIO |
| 10 | `A2` | Analog Input 2 | ADC Channel 2 / GPIO |
| 11 | `A1` | Analog Input 1 | ADC Channel 1 / GPIO |
| 12 | `A0` | Analog Input 0 | ADC Channel 0 / GPIO |
| 13 | `AREF` | Analog Reference | External ADC Reference Voltage |
| 14 | `3V3` | Regulated 3.3V Output | Low-current supply from internal FTDI LDO |
| 15 | `D13/SCK` | Digital I/O 13 | System Clock Input (`SCK`) / Built-in LED |

---

## 📐 PCB Layout & Manufacturing Data

* **Layer Stackup:** 2 Layers (Top Layer: Signal Routing & Local Power Planes; Bottom Layer: Continuous Low-Impedance Ground Plane).
* **SMD Footprints:** High-yield surface mount footprints utilizing 0805 passives, TQFP-32 for the ATmega328P, and SSOP-28 for the FT232RL to accommodate automated reflow profiles or manual soldering sweeps.
* **EMI Optimization:** Ground planes pour completely across empty layout areas, linking together through dense via stitching under high-frequency crystal nodes to prevent parasitic loop antennas.

---
##
![image1](images/1)
![image2](images/2)
![image1](images/pcb)
![image3](images/FTDI)
![image4](images/autoselector)
![image5](images/connection)
![image6](images/main_chip)
![image7](images/pin_header)
![image1](images/all)



---
## 🚀 Deployment & Usage

### 1. Driver Installation
If the operating system does not automatically bind a virtual COM port (VCP) to the hardware upon USB insertion, download and compile the official software drivers directly from the [FTDI Chip Drivers Directory](https://ftdichip.com/drivers/vcp-drivers/).

### 2. Development Setup
Within your preferred microcontroller IDE (e.g., Arduino IDE, PlatformIO, or VS Code):
1. Navigate to your target device manager.
2. Select **Board:** `Arduino Nano`
3. Select **Processor:** `ATmega328P` *(Note: Depending on how your fuses were initialised during compilation, select `ATmega328P (Old Bootloader)` if running factory standard 57600 baud parameters).*
4. Select the matching active communication port and deploy your binary firmware layout.

---

### License
This open-source hardware design repository is free to distribute, fork, modify, and manufacture under commercial or non-commercial constraints.
