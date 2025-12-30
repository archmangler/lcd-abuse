# STM32F411RE LCD Display with Heartbeat Monitor

A bare-metal embedded system project for the STM32F411RETx microcontroller on the Nucleo-F411RE development board. This project demonstrates register-level programming to drive a 20×4 character LCD via I²C and implement a system heartbeat monitor using the onboard LED.

## Project Overview

This project implements a minimal embedded system that:
- **Monitors system liveness** via an LED heartbeat indicator (PA5)
- **Displays system status** on a 20×4 HD44780 character LCD via I²C interface
- **Operates without HAL libraries** using direct register manipulation for maximum control and minimal footprint

### Objectives

1. **Heartbeat/Liveness Indicator**: The onboard user LED (LD2) blinks continuously at approximately 1 Hz to provide visual confirmation that the program is running and the system is alive.

2. **LCD Status Display**: A 20×4 character LCD displays system information, including board identification, interface status, and operational confirmation.

3. **Educational Demonstration**: Showcase bare-metal programming techniques for STM32 microcontrollers, including:
   - System clock initialization
   - GPIO configuration
   - I²C peripheral setup and communication
   - LCD driver implementation

## Hardware Requirements

### Development Board
- **STM32 Nucleo-F411RE** (STM32F411RETx microcontroller)

### Additional Hardware
- **20×4 HD44780 Character LCD** with I²C backpack (PCF8574-based)
  - Common addresses: 0x27 or 0x3F
  - Requires 5V power supply
  - Includes contrast adjustment potentiometer

### Connections

```
LCD I²C Backpack  →  Nucleo-F411RE
─────────────────────────────────────
VCC               →  5V
GND               →  GND
SDA               →  PB9 (D14 on Arduino header)
SCL               →  PB8 (D15 on Arduino header)
```

**Note**: The Nucleo-F411RE provides 5V power from the ST-Link USB connector. The I²C lines (SDA/SCL) are 5V-tolerant on the STM32F411, allowing direct connection to the LCD module's pull-up resistors.

## System Architecture

### Hardware Architecture

```
┌─────────────────┐         I²C Bus         ┌──────────────┐
│                 │   PB8 (SCL) ────────────┤              │
│  STM32F411RETx  │                         │  PCF8574     │
│                 │   PB9 (SDA) ────────────┤  I²C         │
│                 │                         │  Backpack    │
│  PA5 (LED) ─────┼─────────────────────────┼─┐            │
└─────────────────┘                         └─┼────────────┘
                                               │
                                               │ Parallel Interface
                                               ▼
                                          ┌──────────────┐
                                          │  HD44780     │
                                          │  20×4 LCD    │
                                          └──────────────┘
```

### Software Architecture

The system is structured in layers:

1. **Hardware Abstraction Layer**
   - Register definitions for RCC, GPIO, and I²C peripherals
   - Direct memory-mapped register access structures

2. **Peripheral Drivers**
   - **System Clock**: HSI (16 MHz) initialization
   - **GPIO**: LED output configuration (PA5)
   - **I²C1**: Master mode configuration for LCD communication
   - **LCD Driver**: HD44780 controller interface via PCF8574

3. **Application Layer**
   - System initialization sequence
   - Main control loop with heartbeat generation
   - LCD status display management

## How It Works

### System Initialization Sequence

1. **SystemInit()** (called from startup code)
   - Enables HSI (High-Speed Internal) oscillator (16 MHz)
   - Configures HSI as system clock source
   - Sets up basic system timing

2. **GPIO Configuration**
   - Enables GPIOA peripheral clock
   - Configures PA5 as push-pull output for LED control
   - Sets appropriate speed and pull-up/pull-down settings

3. **I²C1 Initialization**
   - Enables GPIOB and I²C1 peripheral clocks
   - Configures PB8 (SCL) and PB9 (SDA) as alternate function 4 (I²C1)
   - Sets pins to open-drain mode with pull-up resistors
   - Configures I²C1 for 100 kHz Standard mode
   - Enables I²C1 peripheral

4. **LCD Initialization**
   - Implements HD44780 4-bit initialization sequence
   - Configures LCD for 2-line mode (20×4 display)
   - Enables display, disables cursor and blinking
   - Clears display and sets entry mode

5. **Initial Display**
   - Writes startup message to LCD showing:
     - Board identification (STM32F411RE)
     - Interface type (LCD 20×4 I²C)
     - Status indicators (Heartbeat: OK, System Ready)

### Main Loop Operation

The main program loop continuously:

1. **Toggles LED (PA5)** every 500 ms
   - Provides visual heartbeat at ~1 Hz
   - Indicates system is running and responsive
   - Can be used to detect system hangs or crashes

2. **Updates LCD Status** (every 5 seconds)
   - Refreshes heartbeat indicator on LCD
   - Confirms system is operational

3. **Maintains Responsive Timing**
   - Uses blocking delays for simplicity
   - Timing is approximate and based on system clock frequency

### I²C Communication Protocol

The system communicates with the LCD module through a PCF8574 I²C-to-parallel converter:

1. **PCF8574 Pin Mapping**
   - P0: RS (Register Select)
   - P1: RW (Read/Write - always 0 for write)
   - P2: E (Enable)
   - P3: Backlight control
   - P4-P7: Data bits D4-D7 (4-bit mode)

2. **Communication Sequence**
   - I²C Start condition
   - Send device address (0x27 << 1 for write)
   - Send control/data byte
   - Pulse Enable pin to latch data
   - I²C Stop condition

3. **LCD Data Format**
   - Commands: RS=0, data byte sent as two 4-bit nibbles
   - Data: RS=1, character byte sent as two 4-bit nibbles

### Heartbeat Mechanism

The heartbeat serves as a **liveness check**:

- **Purpose**: Provides immediate visual feedback that the microcontroller is executing code
- **Frequency**: Approximately 1 Hz (500 ms on, 500 ms off)
- **Benefits**:
  - Quick visual confirmation of system operation
  - Easy detection of system freezes or crashes
  - Simple debugging aid during development
  - Professional system status indicator in embedded applications

## Code Structure

### Key Functions

- `SystemInit()`: System clock configuration (called from startup)
- `delay_ms()` / `delay_us()`: Simple blocking delay functions
- `I2C1_Init()`: I²C1 peripheral initialization
- `I2C_Start()` / `I2C_Stop()`: I²C bus control
- `I2C_SendAddress()` / `I2C_SendByte()`: I²C data transmission
- `pcf8574_write()`: Low-level PCF8574 I²C write
- `lcd_init()`: LCD initialization sequence
- `lcd_cmd()` / `lcd_data()`: LCD command and data sending
- `lcd_set_cursor()`: Position cursor at specific row/column
- `lcd_print()`: Print null-terminated string

## Building the Project

This project is designed for **STM32CubeIDE** or compatible GCC-based toolchains.

### Prerequisites
- STM32CubeIDE (or arm-none-eabi-gcc toolchain)
- STM32F411RETx target configuration

### Build Steps

1. **Open Project**
   - Import project into STM32CubeIDE
   - Ensure correct target device is selected (STM32F411RETx)

2. **Configure Build Settings**
   - Linker script: `STM32F411RETX_FLASH.ld`
   - Target: ARM Cortex-M4
   - FPU: Software FP (or configure FPU if needed)

3. **Build**
   - Clean and build project
   - Verify no compilation errors

4. **Flash**
   - Connect Nucleo board via USB
   - Flash program to target
   - Reset board to start execution

## Usage

### Initial Setup

1. **Hardware Connection**
   - Connect LCD module to Nucleo board as per wiring diagram
   - Ensure 5V power is available (from USB connector)

2. **LCD Contrast Adjustment**
   - Locate blue potentiometer on I²C backpack
   - Power on system
   - Adjust potentiometer until text is clearly visible
   - If display shows solid blocks: contrast too high
   - If display shows nothing: contrast too low

3. **I²C Address Configuration**
   - Default address in code: `0x27`
   - If LCD doesn't respond, try changing to `0x3F` in code:
     ```c
     #define LCD_I2C_ADDR 0x3F  // Change from 0x27
     ```

### Expected Behavior

Upon startup:
- **LED (LD2)**: Begins blinking at ~1 Hz
- **LCD Display**:
  ```
  Line 1: STM32F411RE
  Line 2: LCD 20x4 I2C
  Line 3: Heartbeat: OK
  Line 4: System Ready
  ```

### Troubleshooting

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| LED doesn't blink | Code not running | Check programming, verify reset |
| LCD shows nothing | Contrast not set | Adjust potentiometer on backpack |
| LCD shows blocks | Contrast too high | Reduce contrast |
| LCD shows garbage | Wrong I²C address | Try 0x3F instead of 0x27 |
| LCD doesn't respond | Wiring issue | Check SDA/SCL connections, verify 5V power |
| Compilation errors | Missing definitions | Verify register structure definitions |

## Technical Details

### Clock Configuration
- **System Clock**: 16 MHz (HSI)
- **I²C Clock**: 100 kHz (Standard mode)
- **APB1 Clock**: 16 MHz (for I²C timing)

### Memory Map
- **Flash**: 512 KB (0x08000000)
- **RAM**: 128 KB (0x20000000)
- **Stack Size**: 1 KB (0x400)
- **Heap Size**: 512 bytes (0x200)

### Peripheral Usage
- **GPIOA**: PA5 (LED output)
- **GPIOB**: PB8 (I²C1 SCL), PB9 (I²C1 SDA)
- **I²C1**: Master mode, 100 kHz
- **RCC**: System clock control

## Design Philosophy

This project emphasizes:

1. **Bare-Metal Programming**: Direct register manipulation without HAL/LL abstractions for educational purposes and maximum control

2. **Simplicity**: Minimal code footprint, clear structure, and straightforward implementation

3. **Reliability**: Simple heartbeat mechanism provides immediate visual feedback of system status

4. **Modularity**: Well-organized code structure allows easy extension and modification

## Future Enhancements

Potential improvements:
- Hardware timer-based delays for precise timing
- Non-blocking I²C communication
- Additional LCD display features (scrolling, custom characters)
- System monitoring (temperature, voltage)
- Remote status reporting via UART
- Low-power modes with periodic wake-up

## License

Copyright (c) 2025 T.G Welcome traiano@gmail.com
This software is provided AS-IS per the author's licensing terms.

## References

- STM32F411RE Reference Manual (RM0383)
- HD44780 LCD Controller Datasheet
- PCF8574 I²C-to-Parallel Converter Datasheet
- STM32 Nucleo-F411RE User Manual

---

**Project Status**: ✅ Tested and Working

The system has been verified to work correctly with the hardware configuration described above.

