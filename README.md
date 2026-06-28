
## 1. Overview

This design is a production-optimized modification of the open-source nRF9151 Feather platform, tailored for animal tracking applications.

The system integrates:

- Cellular (NTN/LTE-M/NB-IoT) + GPS via nRF9151
- LoRa communication# Animal Tracker PCB — nRF9151 + SX1262
 via SX1262
- Dual battery support (LiPo + Solar OR LiSOCl₂)
- Ultra-low power operation
- Compact form factor (≤ 50.8mm × 22.86mm)

All development-board-specific and non-essential components have been removed to improve cost, efficiency, and manufacturability.

---

## 2. Removed Components

The following features from the original Feather design are excluded:

- RP2040 co-processor
- USB-C interface
- External NOR Flash (Winbond 4MB)
- Non-essential push buttons (only Reset retained)
- QWIIC connector

---

## 3. Core Functional Blocks

### 3.1 Main Processing Unit

**nRF9151 SiP**

- Integrated LTE-M / NB-IoT / NTN modem
- Integrated GPS
- Main MCU for system control
- Interfaces with LoRa module and power control logic

### 3.2 LoRa Communication

**SX1262** (SMD 16×16mm module)

- SPI-connected to nRF9151
- Controlled via GPIO (including power gating)
- External U.FL antenna connector
- Internal RF matching (no external matching network required)
- Optimized for low-cost (~$2.8) and long-range communication

---

## 4. Power Architecture

### 4.1 Dual Battery Support

The system supports two mutually exclusive power configurations:

**LiPo Battery (Rechargeable)**
- Charged via solar input
- Managed by CN3165 solar charge controller

**LiSOCl₂ Battery (Primary)**
- Non-rechargeable
- Used for ultra-long-life deployments

### 4.2 Power Path Management

**LTC4412 Ideal Diode Controller**

- Automatically selects active power source
- Prevents reverse current
- Ensures seamless switching between battery types

### 4.3 Voltage Regulation Strategy

Instead of a PMIC (e.g., nPM1300), the design uses:

- Two buck regulators (Master + Slave)
- Controlled using RC timing + PGOOD logic
- Eliminates need for external microcontroller for sequencing

---

## 5. Power Sequencing (Critical Design)

The system uses RC-based timing ("power inertia") to meet nRF9151 strict requirements:

- VDD (core) must power ON first and OFF last
- VDD_GPIO must power OFF first

### 5.1 Master Rail (Main 3.3V – Core Supply)

**Components:**
- 100kΩ resistor
- 10µF capacitor

**Behavior:**
- Startup delay: ~700ms
- Shutdown delay: ~1000ms

**Purpose:**
- Provides stable core voltage
- Allows LTE modem network detach, safe flash memory writes, and system state preservation

### 5.2 Slave Rail (GPIO Supply – Fast Rail)

**Components:**
- 100kΩ pull-down
- 10nF capacitor

**Behavior:**
- Startup: activates after master reaches ~95%
- Shutdown: discharges in ~1ms

**Purpose:**
- Ensures GPIO power drops before core power
- Prevents back-powering through I/O pins

---

## 6. Solar Charging System

**CN3165 Solar Charge Controller**

- Handles LiPo charging from solar panel
- Efficient energy harvesting
- Supports autonomous field deployment
- Protects nRF9151 integrity
