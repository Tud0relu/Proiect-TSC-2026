# InkTime — E-Ink Smartwatch Hardware Documentation

> **InkTime** is a custom smartwatch built around the **Nordic nRF52840** SoC featuring an e-paper (EPD) display, haptic feedback, 3-axis accelerometer, smart battery management, and USB-C charging — all packed into a compact wearable PCB.

---

## Table of Contents

1. [Block Diagram](#block-diagram)
2. [Bill of Materials (BOM)](#bill-of-materials-bom)
3. [Hardware Functionality Description](#hardware-functionality-description)
   - [Microcontroller — Nordic nRF52840](#microcontroller--nordic-nrf52840)
   - [Power Architecture](#power-architecture)
   - [E-Paper Display Interface](#e-paper-display-interface)
   - [IMU — Bosch BMA423](#imu--bosch-bma423)
   - [Haptic Driver — TI DRV2605](#haptic-driver--ti-drv2605)
   - [Fuel Gauge — MAX17048](#fuel-gauge--max17048)
   - [RF & Antenna](#rf--antenna)
   - [User Interface — Buttons](#user-interface--buttons)
   - [USB-C Connectivity](#usb-c-connectivity)
   - [Debug Interface](#debug-interface)
4. [nRF52840 Pin Assignment](#nrf52840-pin-assignment)
5. [Power Consumption Estimates](#power-consumption-estimates)
6. [PCB Design Notes](#pcb-design-notes)
7. [Enclosure](#enclosure)

---

## Block Diagram

```
                        ┌─────────────────────────────────────────────────────┐
                        │                  InkTime PCB                        │
                        │                                                     │
  USB-C (J4)            │  ┌───────────┐     I²C (SDA/SCL)   ┌───────────┐  │
 ──────────────► VBUS ──┼─►│ BQ25180   │◄───────────────────►│ BMA423    │  │
  KH-TYPE-C-16P         │  │ (Charger) │◄──P0.11 INT         │ (IMU)     │  │
                        │  │  IC1      │                      │  IC3      │  │
                        │  └─────┬─────┘  I²C (SDA/SCL)  ┌──┴─────────┐ │  │
                        │        │ SYS ──────────────────►│ DRV2605    │ │  │
  LiPo Battery          │        │           P0.12 EN     │ (Haptic)   │ │  │
 ──────────────► VBAT ──┼────────┤           OUT+/OUT─►  │   IC2      │ │  │
                        │        ▼                       └────────────┘ │  │
                        │  ┌───────────┐                  ┌────────────┐ │  │
                        │  │ RT6160    │  3V3 Rail        │ MAX17048   │ │  │
                        │  │ Buck-Boost│──────────────────► (FuelGauge)│ │  │
                        │  │  IC9      │                  │   U3       │ │  │
                        │  └───────────┘                  └────────────┘ │  │
                        │        │                              I²C ──────┘  │
                        │        │ 3V3                                        │
                        │        ▼                                            │
                        │  ┌─────────────────────────────────────────────┐   │
                        │  │         Nordic nRF52840  (U1)               │   │
                        │  │  BT5.0 + 802.15.4 + ARM Cortex-M4F 64MHz   │   │
                        │  │                                             │   │
                        │  │  SPI ──────────────────────► EPD Display   │   │
                        │  │  I²C ──────────────────────► BMA423 / DRV  │   │
                        │  │  I²C ──────────────────────► MAX17048 / DRV│   │
                        │  │  GPIO ─────────────────────► EPD DC/RST/CS │   │
                        │  │  GPIO ─────────────────────► HAPTIC EN     │   │
                        │  │  GPIO ─────────────────────► Buttons x3    │   │
                        │  │  SWDIO/SWDCLK ─────────────► TC2030 J2     │   │
                        │  │  ANT ──────────────────────► 2450AT18B100E │   │
                        │  └─────────────────────────────────────────────┘   │
                        │                                                     │
                        │  EPD Power Rail (3V3_EPD)  Q1(P-MOSFET) + Q3(N-MOSFET) │
                        │  FPC Connector J1 ──────────────────────► E-Paper Display│
                        └─────────────────────────────────────────────────────┘
```

> **Key interfaces:** SPI (EPD display), I²C (BMA423 + DRV2605 + MAX17048 + BQ25180), USB-C (charging + USB 2.0 FS), SWD (debug/programming).

---

## Bill of Materials (BOM)

| # | Qty | Designator | Component | Value / MPN | Package | JLCPCB / LCSC | Datasheet |
|---|-----|------------|-----------|-------------|---------|----------------|-----------|
| 1 | 1 | U1 | Nordic nRF52840 | NRF52840_QF | AQFN-73 7×7mm | — | [Datasheet](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.7.pdf) |
| 2 | 1 | IC1 | Li-Ion charger | BQ25180YBGR | 8-DSBGA 1.6×1.1mm | [C2678081](https://jlcpcb.com/partdetail/TexasInstruments-BQ25180YBGR/C2678081) | [Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) |
| 3 | 1 | IC2 | Haptic driver ERM/LRA | DRV2605YZFR | 9-BGA 1.44×1.44mm | [C2903721](https://jlcpcb.com/partdetail/TexasInstruments-DRV2605YZFR/C2903721) | [Datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) |
| 4 | 1 | IC3 | Accelerometer | BMA423 | LGA-12 2×2mm | [C332461](https://jlcpcb.com/partdetail/BoschSensortec-BMA423/C332461) | [Datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bma423-ds000.pdf) |
| 5 | 1 | IC9 | Buck-boost DC/DC | RT6160AWSC | WCSP-15 1.4×2.3mm | [C2844520](https://jlcpcb.com/partdetail/Richtek-RT6160AWSC/C2844520) | [Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-00.pdf) |
| 6 | 1 | U3 | 1-cell fuel gauge | MAX17048G+T10 | TDFN-8 2×2mm | — | [Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) |
| 7 | 1 | ANT1 | 2.45 GHz chip antenna | 2450AT18B100E | 3.2×1.6×1.4mm SMD | [C404959](https://jlcpcb.com/partdetail/JohansonDielectrics-2450AT18B100E/C404959) | [Datasheet](https://www.johansontechnology.com/datasheets/2450AT18B100E.pdf) |
| 8 | 1 | J1 | FPC connector 24-pin | 503480-2400 | FPC 0.5mm RA SMD | [C311220](https://jlcpcb.com/partdetail/Molex-503480_2400/C311220) | [Datasheet](https://www.molex.com/pdm_docs/sd/5034802400_sd.pdf) |
| 9 | 1 | J4 | USB Type-C 16-pin | KH-TYPE-C-16P | SMD receptacle | — | [Datasheet](https://www.kinghelm.net/product/kh-type-c-16p.html) |
| 10 | 1 | J2 | SWD debug header | TC2030-IDC | Tag-Connect | — | [Datasheet](https://www.tag-connect.com/wp-content/uploads/bsk-pdf-manager/TC2030-IDC_1.pdf) |
| 11 | 1 | Q1 | P-MOSFET EPD power switch | DMG2305UX-7 | SOT-23 | — | [Datasheet](https://www.diodes.com/assets/Datasheets/DMG2305UX.pdf) |
| 12 | 1 | Q3 | N-MOSFET EPD gate driver | SI1308EDL-T1-GE3 | SC-70 | [C240327](https://jlcpcb.com/partdetail/VishayS-SI1308EDL_T1_GE3/C240327) | [Datasheet](https://www.vishay.com/docs/63399/si1308edl.pdf) |
| 13 | 3 | D2, D4, D5 | Schottky diode 30V/0.5A | MBR0530 | SOD-123 | [C191023](https://jlcpcb.com/partdetail/OnSemiconductor-MBR0530/C191023) | [Datasheet](https://www.onsemi.com/pdf/datasheet/mbr0530-d.pdf) |
| 14 | 1 | D3 | USB 2.0 ESD protection | USBLC6-2SC6Y | SOT-363 | [C145414](https://jlcpcb.com/partdetail/STMicroelectronics-USBLC6_2SC6Y/C145414) | [Datasheet](https://www.st.com/resource/en/datasheet/usblc6-2.pdf) |
| 15 | 1 | L5 | Power inductor 68uH | Würth 744043680 | IND 4.8×2.8mm | [C516667](https://jlcpcb.com/partdetail/WurthElektronik-744043680/C516667) | [Datasheet](https://www.we-online.com/catalog/datasheet/744043680.pdf) |
| 16 | 1 | L7 | Power inductor 0.47uH | FTC252012SR47MBCA | L1608 2×1.6mm | [C5832368](https://jlcpcb.com/partdetail/6763488-FTC252012SR47MBCA/C5832368) | [Datasheet](https://product.tdk.com/info/en/documents/catalog/clq_prog_en_ds_1.pdf) |
| 17 | 1 | X1 | Crystal 32 MHz | 32MHz SMD | Crystal_2016 | — | Nordic reference crystal |
| 18 | 1 | X2 | Crystal 32.768 kHz | 32.768kHz | Crystal_3215 | — | Nordic reference crystal |
| 19 | 3 | SW_UP, SW_DN, SW_ENT | Tactile switch | EVP-AKE31A | SMD | [C2689689](https://jlcpcb.com/partdetail/Panasonic-EVP_AKE31A/C2689689) | [Datasheet](https://industrial.panasonic.com/ww/products/pt/tactile-switches/models/EVP-AKE31A) |
| 20 | 1 | L1 | RF inductor 3.9nH | 3.9nH | 0402 | — | Matching network |
| 21 | 1 | L2 | RF choke 10uH | 10uH | 0402 | — | DC feed |
| 22 | 1 | L3 | RF filter | 15nH | 0402 | — | Matching network |
| 23 | 4 | C14, C20, C21, C6 | MLCC capacitor 4.7uF | 4.7uF | 0402 | — | Decoupling |
| 24 | 7 | C10, C12, C13, C19, C5, C7, C8 | MLCC capacitor 100nF | 100nF | 0201 | — | Decoupling |
| 25 | 3 | C29, C30, C31 | MLCC capacitor 1uF | 1uF | 0402 | — | Button debounce |
| 26 | 3 | C1-EP-DR, C24, C39 | MLCC capacitor 10uF | 10uF | 0402 | — | Bulk decoupling |
| 27 | 9 | EPD_C1..C12 | MLCC capacitor 1uF/50V | 1uF/50V | 0402 | — | EPD power |
| 28 | 2 | R17, R18 | Resistor 3.3kΩ | 3K3 | 0201 | — | I²C pull-up |
| 29 | 2 | R1_USB, R2_USB | Resistor 5.1kΩ | 5K1 | 0201 | — | USB-C CC detection |
| 30 | 3 | R2, R3, R4 | Resistor 7.68kΩ | 7.68k | 0201 | — | Config resistors |
| 31 | 3 | R5, R7, R8 | Resistor 10kΩ | 10k | 0201 | — | Button pull-up |
| 32 | 3 | R2_EP_DR, R9, R_PWR_EPD | Resistor 10kΩ | 10K | 0201 | — | EPD / PMIC bias |
| 33 | 1 | SJ1 | Solder jumper | — | Solder Jumper | — | Config bridge |
| 34 | 14 | TP1–TP6, TP_* | Test points | — | TP_20R | — | Debug/test |

---

## Hardware Functionality Description

### Microcontroller — Nordic nRF52840

The **nRF52840** (U1) is the heart of InkTime. It is a single-chip multiprotocol SoC combining:

- **ARM Cortex-M4F** core @ 64 MHz with FPU
- **1 MB Flash** + **256 KB RAM**
- **Bluetooth 5.0 LE** (1 Mbps/2 Mbps/500 kbps/125 kbps) + IEEE 802.15.4 (Zigbee/Thread)
- **USB 2.0 Full-Speed** device controller (12 Mbps)
- Hardware AES-128 encryption, ARM TrustZone CryptoCell-310
- 48 GPIO pins, 3× TWI (I²C), 4× SPI, 2× UART, 12-bit ADC

The nRF52840 uses the internal DC/DC converter for lower power consumption. Supply is 3.3 V from IC9. Two crystals are used:
- **X1** — 32 MHz (RF clock, high-frequency crystal)
- **X2** — 32.768 kHz (LFCLK for RTC, sleep timekeeping)

The built-in USB 2.0 controller connects directly to the KH-TYPE-C-16P receptacle (J4) via the ESD-protected D+/D− lines, enabling USB DFU firmware updates without a dedicated USB-UART chip.

---

### Power Architecture

```
USB-C (VBUS 5V)
      │
      ▼
  BQ25180YBGR (IC1) ◄── LiPo battery (VBAT)
  Linear charger, up to 1A charge current
      │
      ▼ SYS rail (≈ VBAT or 4.2V when charging)
      │
      ▼
  RT6160AWSC (IC9)
  I²C-programmable buck-boost converter
  Input: 2.5V – 5.5V  |  Output: 3.3V @ up to 800mA
      │
      ▼ 3V3 rail (powers nRF52840, BMA423, DRV2605, MAX17048)
```

**BQ25180YBGR (IC1) — Li-Ion Battery Charger:**
- Input voltage: 3.8V–5.5V (VBUS from USB-C)
- Charge current: configurable via I²C (default ~200 mA, up to 1A)
- Integrated power path management: device runs from VBUS while simultaneously charging
- Interrupt line (!INT) → P0.11: alerts the MCU to status changes (charge complete, fault, OVP)
- I²C address: 0x6B on shared SDA/SCL bus (P0.06/P0.07)

**RT6160AWSC (IC9) — Buck-Boost Regulator:**
- Input: SYS rail (2.6V–5.5V)
- Output: fixed 3.3V ±2% at up to 800mA
- Operates in both step-up (battery near discharge) and step-down (charging) modes
- Enables reliable 3.3V operation down to VBAT ≈ 2.5V, extending battery life
- Enable controlled by VREG signal (always-on in normal operation)
- L7 (0.47uH) is the switching inductor; C25, C33 (22uF) are output caps

**MAX17048G+T10 (U3) — Battery Fuel Gauge:**
- ModelGauge algorithm: estimates State of Charge (SoC %) without coulomb counting
- I²C address: 0x36 on shared I²C bus
- ALERT pin → P0.10 (NFC2): configurable SoC threshold interrupt (e.g., alert at <5%)
- Powered from VBAT directly for always-on monitoring
- Current consumption: 3µA typical in active, 0.5µA in hibernate

---

### E-Paper Display Interface

The watch uses an e-paper (EPD) display connected via **Molex 503480-2400 FPC connector (J1)** — a 24-pin, 0.5mm pitch ZIF socket. The nRF52840 drives the EPD over **SPI** with additional GPIO control lines:

| Signal | nRF52840 Pin | Direction | Function |
|--------|-------------|-----------|----------|
| SCK | P0.02 | OUT | SPI clock |
| MOSI | P0.03 | OUT | SPI data |
| EPD_CS | P0.05 | OUT | Chip select (active low) |
| EPD_DC | P0.15 | OUT | Data/Command select |
| EPD_RST | P0.16 | OUT | Hardware reset (active low) |
| EPD_BUSY | P0.17 | IN | Busy signal from display controller |

**EPD Power Supply:**
The e-paper panel requires a higher-voltage supply (typically 20–40V internally generated by its own controller IC), but its logic/interface side runs on 3.3V. A dedicated **3V3_EPD** rail is created by an additional path controlled by:
- **Q1 (DMG2305UX-7)** — P-channel MOSFET acting as a high-side power switch for the EPD 3.3V supply, controlled by `R_PWR_EPD`
- **Q3 (SI1308EDL)** — N-channel MOSFET for EPD gate driver control
- **D2, D4, D5 (MBR0530)** — Schottky diodes for EPD voltage rail protection and PREVGH/PREVGL rail generation (required by most EPD controllers like UC8176/SSD1680)

This architecture ensures the EPD power is only active when needed, saving power during sleep.

---

### IMU — Bosch BMA423

The **BMA423** (IC3) is a triaxial, low-g accelerometer designed for wearables:

- **Range:** ±2g / ±4g / ±8g / ±16g (configurable)
- **Resolution:** 12-bit
- **Interface:** I²C (address 0x18, CSB pin pulled to VDD → secondary address disabled)
- **Features:** Built-in step counter, wrist-wear wake-up, double-tap/single-tap detection, activity recognition
- **Power:** 170µA active, 2µA in suspend, ~0.9µA in deep suspend
- **Supply:** 1.71V–3.6V (VDDIO and VDD both connected to 3.3V rail)

**Connections:**
- SDA → P0.06 (shared I²C bus)
- SCL → P0.07 (shared I²C bus)
- INT1 → P0.08: wake-up interrupt (step counter, gesture)
- INT2 → P1.08: secondary interrupt (configurable)
- CSB → 3V3 (I²C mode selection)

The BMA423 enables the watch to detect wrist rotation (raise-to-wake), count steps, and recognize sleep/wake cycles without MCU involvement.

---

### Haptic Driver — TI DRV2605

The **DRV2605YZFR** (IC2) provides haptic feedback to a motor (ERM or LRA type):

- **Interface:** I²C (address 0x5A)
- **Built-in waveform library:** 123 pre-loaded haptic waveforms (ROM)
- **Smart Loop Architecture:** auto-resonance tracking for LRA motors
- **Overdrive/brake control** for precise tactile feel
- **Supply:** 2.7V–5.5V (connected to local 3.3V rail)
- **Output current:** ±1.2A peak

**Connections:**
- SDA → P0.06 (shared I²C bus)
- SCL → P0.07 (shared I²C bus)
- EN → P0.12: enable pin (active high, controls driver power state)
- OUT+ / OUT− → motor terminals (differential drive)
- REG output decoupled by C32 (1uF)

I²C address 0x5A is hardwired (ADDR pin tied to GND internally).

---

### RF & Antenna

The **2450AT18B100E** (ANT1) from Johanson Technology is a multilayer ceramic chip antenna tuned for 2.4–2.5 GHz:

- **Gain:** 0 dBi typical
- **Impedance:** 50Ω
- **Size:** 3.2×1.6×1.4mm

RF matching network (π-filter):
- **L1** (3.9nH) — series matching inductor from ANT pin of nRF52840
- **C3, C4** (1pF each) — shunt capacitors for impedance matching
- **L2** (10uH) + **DCC** node — DC feed inductor for the internal PA supply
- **C9** (820pF), **C11** (100pF), **C3/C4** (1pF) — harmonics filtering

The antenna is placed at the corner of the PCB, away from the battery, with a keep-out zone under the radiating element per Nordic reference design guidelines.

---

### User Interface — Buttons

Three tactile switches (**Panasonic EVP-AKE31A**) provide the user interface:

| Designator | Function | nRF52840 Pin | Pull-up | Debounce Cap |
|------------|----------|-------------|---------|--------------|
| SW_UP | Scroll Up / Confirm | P0.13 | R5 (10kΩ) | C30 (1uF) |
| SW_ENT | Enter / Select | P0.14 | R8 (10kΩ) | C31 (1uF) |
| SW_DN | Scroll Down / Back | P1.02 | R7 (10kΩ) | C29 (1uF) |

All buttons are **active-low** — pressing connects the GPIO to GND through the switch. The pull-up resistors (10kΩ) are tied to the 3.3V rail. Hardware RC debouncing is achieved by the 1uF capacitors in parallel with the switches.

The nRF52840 GPIOTE module handles button interrupts with minimal power overhead.

---

### USB-C Connectivity

The **KH-TYPE-C-16P** (J4) is a 16-pin USB Type-C receptacle providing:

- **VBUS (5V)** → IC1 (BQ25180) for charging
- **D+ / D−** → nRF52840 P0.12/P0.13 equivalent USB data pins for USB 2.0 FS
- **CC1 / CC2** → R1_USB, R2_USB (5.1kΩ each to GND): configure the port as a UFP (power sink), telling the attached charger to provide 5V/standard current
- **USBLC6-2SC6Y (D3)** — bidirectional TVS diode array on D+/D− lines for IEC 61000-4-2 ESD protection (±8kV contact discharge)

The nRF52840's built-in USB Full-Speed device controller (12 Mbps) enables:
- USB DFU (Device Firmware Upgrade) — drag-and-drop firmware flashing
- USB CDC (virtual serial port) — debug logging
- USB HID — optional time sync interface

---

### Debug Interface

**Tag-Connect TC2030-IDC (J2)** is a 6-pin spring-loaded programming/debug connector with no PCB footprint pins, saving board space:

| Pin | Signal | Function |
|-----|--------|----------|
| 1 | 3V3 | Target power sense |
| 2 | SWDIO | Serial Wire Debug I/O |
| 3 | GND | Ground |
| 4 | SWDCLK | Serial Wire Debug Clock |
| 5 | GND | Ground |
| 6 | RESET | nRF52840 hardware reset |

Additional debug test points on PCB: `TP_SWDIO`, `TP_SWDCLK`, `TP_3V3`, `TP_VBAT`, `TP_VREG`, `TP_SDA`, `TP_SCL`, `TP_ON`, `TP_OP`, `TP_SWO` (Serial Wire Output for ITM tracing).

---

## nRF52840 Pin Assignment

| GPIO Pin | Net / Signal | Connected To | Function |
|----------|-------------|--------------|----------|
| P0.00 / XL1 | P0.00/XL1 | X2 pin 1, C17 | 32.768 kHz LFXO in |
| P0.01 / XL2 | P0.01/XL2 | X2 pin 2, C18 | 32.768 kHz LFXO out |
| P0.02 / AIN0 | SCK | EPD FPC (J1 via schematic) | SPI clock for e-paper |
| P0.03 / AIN1 | MOSI | EPD FPC (J1) | SPI MOSI for e-paper |
| P0.05 / AIN3 | EPD_CS | EPD FPC (J1) | EPD chip select (active low) |
| P0.06 | SDA | IC1, IC3, IC2, U3, R2, R18 | I²C data (shared bus) |
| P0.07 | SCL | IC1, IC3, IC2, U3, R4, R17 | I²C clock (shared bus) |
| P0.08 | IMU_INT1 | IC3 (BMA423) INT1 | IMU interrupt 1 (step / wake) |
| P0.10 / NFC2 | ALERT | U3 (MAX17048) ALERT | Fuel gauge low-battery alert |
| P0.11 | PMIC_INT | IC1 (BQ25180) !INT | Charger status interrupt |
| P0.12 | HAPTIC_EN | IC2 (DRV2605) EN | Haptic driver enable |
| P0.13 | P0.13 | SW_UP, R5, C30 | Button UP (active low) |
| P0.14 | P0.14 | SW_ENT, R8, C31 | Button ENTER (active low) |
| P0.15 | EPD_DC | EPD FPC (J1) | EPD data/command select |
| P0.16 | EPD_RST | EPD FPC (J1) | EPD reset (active low) |
| P0.17 | EPD_BUSY | EPD FPC (J1) | EPD busy status input |
| P0.18 / RESET | RESET | J2 pin 6, TP_RESET | Hardware reset (active low) |
| P1.00 | SWO | TP_SWO | Serial Wire Output (ITM trace) |
| P1.02 | P1.02 | SW_DN, R7, C29 | Button DOWN (active low) |
| P1.08 | IMU_INT2 | IC3 (BMA423) INT2 | IMU interrupt 2 (configurable) |
| SWDIO | SWDIO | J2 pin 2, TP_SWDIO | SWD debug data |
| SWDCLK | SWDCLK | J2 pin 4, TP_SWDCLK | SWD debug clock |
| VBUS | VBUS | IC1 IN, C38, C21 | USB VBUS sense |
| D+ | D+ | J4, D3 | USB 2.0 FS D+ |
| D− | D− | J4, D3 | USB 2.0 FS D− |
| XC1/XC2 | N$3/N$4 | X1, C1, C2 | 32 MHz HFXO crystal |
| ANT | N$7 | L1, C3 | RF antenna feed |
| DCC | N$9 | L2 | PA DC feed |
| VDD (×4) | 3V3 | C6, C7, C8, C14 + IC9 VOUT | 3.3V supply |
| VDDH | 3V3 | IC9 VOUT | High-drive GPIO supply |
| VSS / VSS_PA / VSS_PAD | GND | Ground pour | Ground |

**I²C Bus Summary (P0.06 SDA / P0.07 SCL, 400 kHz Fast Mode):**

| Device | I²C Address | Pull-up |
|--------|------------|---------|
| BQ25180 (IC1) | 0x6B | R2, R18 (3.3kΩ each) |
| BMA423 (IC3) | 0x18 | R2, R18 (3.3kΩ each) |
| DRV2605 (IC2) | 0x5A | R2, R18 (3.3kΩ each) |
| MAX17048 (U3) | 0x36 | R2, R18 (3.3kΩ each) |

All four devices share a single I²C bus with 3.3kΩ pull-up resistors (R17, R18 to 3V3). Effective pull-up for fast-mode at 400 kHz with 4 devices at ~100pF bus capacitance is within spec.

---

## Power Consumption Estimates

| State | Subsystem Active | Current Draw | Notes |
|-------|-----------------|-------------|-------|
| **Deep sleep** | nRF52840 System OFF, RTC running | ~5–10 µA | 32.768kHz RTC, BMA wake-on-tilt |
| **BLE advertising** | nRF52840 BLE adv @ 1s interval | ~60–80 µA avg | Connectable undirected advertising |
| **BLE connected** | Connection interval 50ms | ~200–400 µA avg | — |
| **EPD update** | nRF52840 + EPD refresh 2s | ~15–25 mA peak | EPD full refresh draws spike current |
| **Haptic pulse** | DRV2605 + LRA 200ms | ~50–150 mA peak | Short pulses only |
| **Charging @ 200mA** | BQ25180 charging | 200 mA from VBUS | CC/CV profile |

**Estimated average daily consumption** (light wrist-watch use):
- 22h deep sleep (5µA) + 1h BLE connected (300µA) + 48 EPD updates/day (avg 0.5mA ×2s) ≈ **~0.4–0.6 mAh/day**

With a **100 mAh LiPo**, estimated battery life: **>6 months** with e-ink always-on display and minimal Bluetooth. With active BLE and hourly notifications, expect **4–8 weeks**.

---

## PCB Design Notes

- **Layer stack:** 2-layer PCB (Top + Bottom copper), 1.0mm FR4
- **Board dimensions:** ~45mm × 38mm (designed to fit circular wristwatch case)
- **Minimum trace width:** 0.1mm signal, 0.3mm power
- **Minimum clearance:** 0.1mm
- **Minimum drill:** 0.2mm (vias), 0.3mm annular ring
- **Surface finish:** HASL or ENIG (ENIG recommended for BGA/LGA pads)
- **Solder mask:** Green (standard), openings on all SMD pads
- **Component side:** All components on Top layer (single-sided SMT assembly possible)
- **Antenna keep-out:** 5mm clearance under 2450AT18B100E antenna element
- **Decoupling strategy:** 100nF 0201 caps placed within 0.5mm of each VDD pin, 4.7uF 0402 bulk caps placed per power domain

**Design Rules (JLCPCB 2-layer standard):**
- Min trace/space: 5/5 mil
- Min hole: 0.3mm
- Board outline tolerance: ±0.2mm
- Copper to board edge: ≥0.3mm

---

## Enclosure

The InkTime watch case is designed in **Autodesk Fusion 360** to fit the PCB precisely. Key features:

- **Outer dimensions:** 46mm × 38mm × 12mm (approximately)
- **Display window:** cutout for the EPD panel, with recessed bezel for protection
- **USB-C port cutout:** access to J4 on the bottom edge
- **Button apertures:** 3 side-mounted holes aligned with SW_UP, SW_ENT, SW_DN
- **Strap lugs:** 20mm standard watch band attachment points
- **PCB mounting:** 4× M1.4 screw bosses aligned with PCB corner keepout zones
- **Battery compartment:** cavity on back for 380mAh/100mAh flat LiPo cell
- **Material:** PLA/PETG for prototyping; Aluminum CNC or SLA resin for production

The 3D model is provided as `3DmodelFull.f3z` (Fusion 360 archive) and `3D.step` (neutral STEP format for other CAD tools).

---

## Repository Structure

```
Proiect-TSC-2026/
│
├── Hardware/
│   ├── PCB.brd
│   └── Schematic.sch
│
├── Images/
│   └── PCB.png
│
├── Manufactoring/
│   ├── PCB.cpl
│   ├── Schematic.bom.txt
│   └── gerber.zip
│
├── Mechanical/
│   ├── 3D.step
│   └── PCB_3D.f3d
│
├── LICENSE
└── README.md
```

---

*Project: InkTime Smartwatch | Hardware revision 6 | April 2026*
