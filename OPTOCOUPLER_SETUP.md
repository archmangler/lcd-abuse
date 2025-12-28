# TLP281 4-Channel Optocoupler Connection Guide

## 1. Recommended GPIO Pin Selection

Based on your current project configuration (PA5=LED, PB8/PB9=I2C1 for LCD), here are the recommended GPIO pins for the 4 TLP281 channels:

### Option 1: PB12, PB13, PB14, PB15 (Recommended - Physically Adjacent)

**Four consecutive GPIO pins on Port B - likely physically adjacent on the board:**

| Channel | GPIO Pin | Arduino Header | Port | Notes |
|---------|----------|----------------|------|-------|
| Channel 1 | **PB12** | Not on Arduino | GPIOB | ✅ Consecutive pins |
| Channel 2 | **PB13** | Not on Arduino | GPIOB | ✅ Consecutive pins |
| Channel 3 | **PB14** | Not on Arduino | GPIOB | ✅ Consecutive pins |
| Channel 4 | **PB15** | Not on Arduino | GPIOB | ✅ Consecutive pins |

**Why these pins?**
- ✅ **Four consecutive pins on GPIOB (PB12-PB15)**
- ✅ **Same port** - easier configuration
- ✅ **No conflicts** with your current setup (PA5=LED, PB8/PB9=I2C)
- ✅ **Physically adjacent** on the Nucleo board (check your board's connector layout)

**Note:** These pins may have SPI2 alternate functions, but they work fine as GPIO if not using SPI2.

### Option 2: PC0, PC1, PC2, PC3 (Alternative - Physically Adjacent)

**Four consecutive GPIO pins on Port C:**

| Channel | GPIO Pin | Arduino Header | Port | Notes |
|---------|----------|----------------|------|-------|
| Channel 1 | **PC0** | Not on Arduino | GPIOC | ✅ Consecutive pins |
| Channel 2 | **PC1** | Not on Arduino | GPIOC | ✅ Consecutive pins |
| Channel 3 | **PC2** | Not on Arduino | GPIOC | ✅ Consecutive pins |
| Channel 4 | **PC3** | Not on Arduino | GPIOC | ✅ Consecutive pins |

**Why these pins?**
- ✅ **Four consecutive pins on GPIOC (PC0-PC3)**
- ✅ **Same port** - easier configuration
- ✅ **No alternate function conflicts** - clean GPIO pins
- ✅ **Physically adjacent** on the Nucleo board

### Option 3: PB6, PB7, PB8, PB9 (If not using all I2C features)

**Note:** PB8/PB9 are currently used for I2C. Only use this if you can reconfigure.

| Channel | GPIO Pin | Arduino Header | Port | Notes |
|---------|----------|----------------|------|-------|
| Channel 1 | **PB6** | D10 | GPIOB | ⚠️ May conflict with I2C1 timing |
| Channel 2 | **PB7** | Not on Arduino | GPIOB | ⚠️ May conflict with I2C1 timing |
| Channel 3 | **PB8** | D15 (I2C1 SCL) | GPIOB | ❌ Currently used for I2C |
| Channel 4 | **PB9** | D14 (I2C1 SDA) | GPIOB | ❌ Currently used for I2C |

**Not recommended** due to I2C conflict, but listed for reference.

### Finding Physically Adjacent Pins

To find the best physically adjacent pins for your specific board:

1. **Refer to your Nucleo-F411RE board's pinout diagram** (usually in the user manual)
2. **Look for consecutive GPIO pins on the same connector block** (CN5-CN10)
3. **Check that they're on the same port** (GPIOA, GPIOB, GPIOC, etc.) for easier configuration
4. **Avoid pins used by your application:**
   - ❌ PA5 (D13) - Used for LED heartbeat
   - ❌ PB8 (D15) - Used for I2C1 SCL
   - ❌ PB9 (D14) - Used for I2C1 SDA

**Recommended approach:** Check your board manual to confirm which of the following pin groups are physically adjacent on the same connector:
- PB12, PB13, PB14, PB15 (Option 1 - recommended)
- PC0, PC1, PC2, PC3 (Option 2 - alternative)
- Other consecutive pins you identify

## 2. Current-Limiting Resistors (Required!)

**YES, you MUST use external current-limiting resistors** for the optocoupler LEDs.

### Resistor Calculation

For TLP281 optocoupler:
- **Forward Voltage (V_F)**: ~1.15V to 1.5V (typically 1.15V at 10mA)
- **Recommended Forward Current (I_F)**: 5-20mA (10mA is ideal)
- **STM32 GPIO Output Voltage**: 3.3V (HIGH state)

**Calculation:**
```
R = (V_GPIO - V_F) / I_F
R = (3.3V - 1.15V) / 0.010A
R = 215Ω
```

### Recommended Resistor Value

**Use 220Ω (standard value) for each channel**

This provides:
- **Current**: ~9.8mA at 3.3V (safe and sufficient)
- **Power Dissipation**: ~0.02W per resistor (well within 1/4W rating)
- **Reliability**: Standard value, readily available

### Alternative Resistor Values

| Resistor Value | Forward Current | Notes |
|----------------|-----------------|-------|
| 150Ω | ~14mA | Higher current, brighter LED, faster switching |
| **220Ω** | **~10mA** | **✅ Recommended - balanced** |
| 330Ω | ~6.5mA | Lower current, still reliable |

### Circuit Configuration

```
STM32 GPIO (3.3V) ──[220Ω]──> Anode (Pin 1 of TLP281)
                              │
                              └──> Cathode (Pin 2) ──> GND
```

**Important:**
- Connect one 220Ω resistor **per channel** (4 resistors total)
- Resistor goes between GPIO pin and optocoupler anode
- Cathode connects directly to GND
- All channels share the same GND reference

## 3. Complete Wiring Diagram

### Using Option 1: PB12-PB15 (Recommended)

```
NUCLEO-F411RE                  TLP281 (4-Channel)
(Morpho Connector)             ────────────────────
                               
PB12 ──[220Ω]──> Ch1 Anode (Pin 1)
                    │
                    └──> Ch1 Cathode (Pin 2) ──> GND
                               
PB13 ──[220Ω]──> Ch2 Anode (Pin 3)
                    │
                    └──> Ch2 Cathode (Pin 4) ──> GND
                               
PB14 ──[220Ω]──> Ch3 Anode (Pin 5)
                    │
                    └──> Ch3 Cathode (Pin 6) ──> GND
                               
PB15 ──[220Ω]──> Ch4 Anode (Pin 7)
                    │
                    └──> Ch4 Cathode (Pin 8) ──> GND
                               
GND ─────────────────────────> Common GND
```

### Using Option 2: PC0-PC3 (Alternative)

```
NUCLEO-F411RE                  TLP281 (4-Channel)
(Morpho Connector)             ────────────────────
                               
PC0 ──[220Ω]──> Ch1 Anode (Pin 1)
                   │
                   └──> Ch1 Cathode (Pin 2) ──> GND
                               
PC1 ──[220Ω]──> Ch2 Anode (Pin 3)
                   │
                   └──> Ch2 Cathode (Pin 4) ──> GND
                               
PC2 ──[220Ω]──> Ch3 Anode (Pin 5)
                   │
                   └──> Ch3 Cathode (Pin 6) ──> GND
                               
PC3 ──[220Ω]──> Ch4 Anode (Pin 7)
                   │
                   └──> Ch4 Cathode (Pin 8) ──> GND
                               
GND ─────────────────────────> Common GND
```

### TLP281 Pin Configuration

**Input Side (LED Side - Connected to STM32):**
- Pin 1: Channel 1 Anode
- Pin 2: Channel 1 Cathode
- Pin 3: Channel 2 Anode
- Pin 4: Channel 2 Cathode
- Pin 5: Channel 3 Anode
- Pin 6: Channel 3 Cathode
- Pin 7: Channel 4 Anode
- Pin 8: Channel 4 Cathode

**Output Side (Phototransistor Side - Connected to Solenoids):**
- Collector pins: Connect to your solenoid drive circuit
- Emitter pins: Connect to isolated GND
- **Note:** Output side requires separate power supply and pull-up resistors (not covered here as you mentioned simple solenoids)

## 4. Electrical Specifications

### STM32F411 GPIO Capabilities
- **Output Voltage (HIGH)**: 3.3V
- **Output Voltage (LOW)**: 0V
- **Max Sink Current**: 25mA (per pin, absolute max)
- **Recommended Current**: 5-10mA per pin
- **Our Design**: ~10mA per pin ✅ Safe

### TLP281 Optocoupler
- **Isolation Voltage**: 3750Vrms
- **Forward Voltage**: 1.15V (typical at 10mA)
- **Forward Current**: 5-20mA (recommended range)
- **CTR (Current Transfer Ratio)**: 50-600% (typ 100%)

## 5. Protection Considerations

### Do You Need Additional Protection?

**For the GPIO pins:**
- ✅ **220Ω resistor is sufficient** - provides current limiting
- ✅ **No additional pull-up/down needed** - GPIO is push-pull output
- ✅ **No reverse protection needed** - LED in optocoupler handles direction
- ✅ **No ESD protection needed** - optocoupler provides isolation

**For the optocoupler:**
- ✅ **Input side is protected** by the current-limiting resistor
- ⚠️ **Output side** - ensure solenoids have proper flyback diodes (separate concern)

## 6. Implementation in Code

**Example GPIO initialization for Option 1: PB12-PB15 (Adjacent pins on same port):**

```c
// Enable GPIOB clock
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOBEN;

// Configure PB12-PB15 as outputs (all in one go for efficiency)
// PB12 (Channel 1)
GPIOB->MODER &= ~(3U << (12 * 2));
GPIOB->MODER |= (1U << (12 * 2));     // Output mode
GPIOB->OTYPER &= ~(1U << 12);         // Push-pull
GPIOB->OSPEEDR |= (1U << (12 * 2));   // Medium speed

// PB13 (Channel 2)
GPIOB->MODER &= ~(3U << (13 * 2));
GPIOB->MODER |= (1U << (13 * 2));     // Output mode
GPIOB->OTYPER &= ~(1U << 13);         // Push-pull
GPIOB->OSPEEDR |= (1U << (13 * 2));   // Medium speed

// PB14 (Channel 3)
GPIOB->MODER &= ~(3U << (14 * 2));
GPIOB->MODER |= (1U << (14 * 2));     // Output mode
GPIOB->OTYPER &= ~(1U << 14);         // Push-pull
GPIOB->OSPEEDR |= (1U << (14 * 2));   // Medium speed

// PB15 (Channel 4)
GPIOB->MODER &= ~(3U << (15 * 2));
GPIOB->MODER |= (1U << (15 * 2));     // Output mode
GPIOB->OTYPER &= ~(1U << 15);         // Push-pull
GPIOB->OSPEEDR |= (1U << (15 * 2));   // Medium speed
```

**Example GPIO initialization for Option 2: PC0-PC3 (Alternative adjacent pins):**

```c
// Enable GPIOC clock
RCC->AHB1ENR |= RCC_AHB1ENR_GPIOCEN;

// Configure PC0-PC3 as outputs
// PC0 (Channel 1)
GPIOC->MODER &= ~(3U << (0 * 2));
GPIOC->MODER |= (1U << (0 * 2));      // Output mode
GPIOC->OTYPER &= ~(1U << 0);          // Push-pull
GPIOC->OSPEEDR |= (1U << (0 * 2));    // Medium speed

// PC1 (Channel 2)
GPIOC->MODER &= ~(3U << (1 * 2));
GPIOC->MODER |= (1U << (1 * 2));      // Output mode
GPIOC->OTYPER &= ~(1U << 1);          // Push-pull
GPIOC->OSPEEDR |= (1U << (1 * 2));    // Medium speed

// PC2 (Channel 3)
GPIOC->MODER &= ~(3U << (2 * 2));
GPIOC->MODER |= (1U << (2 * 2));      // Output mode
GPIOC->OTYPER &= ~(1U << 2);          // Push-pull
GPIOC->OSPEEDR |= (1U << (2 * 2));    // Medium speed

// PC3 (Channel 4)
GPIOC->MODER &= ~(3U << (3 * 2));
GPIOC->MODER |= (1U << (3 * 2));      // Output mode
GPIOC->OTYPER &= ~(1U << 3);          // Push-pull
GPIOC->OSPEEDR |= (1U << (3 * 2));    // Medium speed
```

**Example functions to control channels (for Option 1: PB12-PB15):**

```c
// Turn on channel (LED lit = optocoupler active)
void opto_channel_set(uint8_t channel, uint8_t state)
{
    uint8_t pin = 11 + channel;  // Convert 1-4 to 12-15 for PB12-PB15
    
    if (state) 
        GPIOB->BSRR = (1U << pin);           // Set (HIGH)
    else       
        GPIOB->BSRR = (1U << (pin + 16));    // Reset (LOW)
}

// Alternative: Set all channels at once using bit manipulation
void opto_set_all(uint8_t state)
{
    if (state)
        GPIOB->BSRR = (0xF << 12);           // Set PB12-PB15 all HIGH (bits 12-15)
    else
        GPIOB->BSRR = (0xF << (12 + 16));    // Set PB12-PB15 all LOW
}
```

## Summary

**1. Recommended GPIO Pins (Adjacent - Option 1):**
   - Channel 1: **PB12**
   - Channel 2: **PB13**
   - Channel 3: **PB14**
   - Channel 4: **PB15**
   - ✅ **Four consecutive pins on same port (GPIOB) - check board manual for physical adjacency**

**Alternative Adjacent Pins (Option 2):**
   - Channel 1: **PC0**
   - Channel 2: **PC1**
   - Channel 3: **PC2**
   - Channel 4: **PC3**
   - ✅ **Four consecutive pins on same port (GPIOC) - check board manual for physical adjacency**

**2. Required Resistors:**
   - ✅ **Yes, 220Ω current-limiting resistors are REQUIRED**
   - One resistor per channel (4 total)
   - Connect between GPIO pin and optocoupler anode
   - No pull-up/down resistors needed

**3. Protection:**
   - 220Ω resistors provide sufficient current limiting
   - Optocoupler provides electrical isolation
   - No additional protection circuits needed for input side

**4. Power:**
   - Input side: Powered from STM32 3.3V GPIO
   - Output side: Requires separate isolated power supply for solenoids

**⚠️ Important:** Please verify in your Nucleo-F411RE board manual which pins (PB12-PB15 or PC0-PC3) are physically adjacent on the same connector block before finalizing your design.
