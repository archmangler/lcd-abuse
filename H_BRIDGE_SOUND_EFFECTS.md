# H-Bridge Sound Effects via Optocoupler LEDs (PB13, PB14)

## Pin Availability Analysis

**✅ YES, PB13 and PB14 can be used as GPIO outputs for optocoupler LEDs.**

### Current Pin Usage in Project:
- **PA5**: LED heartbeat
- **PA6**: TIM3 CH1 (Speaker/PWM audio)
- **PB8**: I2C1 SCL (LCD communication)
- **PB9**: I2C1 SDA (LCD communication)
- **PC0-PC3**: LED strobe outputs
- **PC4**: Solenoid optocoupler LED
- **PC6-PC9**: Motor optocoupler outputs

### PB13 and PB14 Status:
- **PB13**: SPI2_SCK (alternate function) - **NOT USED** ✅ Available as GPIO
- **PB14**: SPI2_MISO (alternate function) - **NOT USED** ✅ Available as GPIO
- **Port**: GPIOB (already enabled for I2C1)
- **No conflicts**: SPI2 is not used in this project

## Pin Specifications

| Pin | Alternate Function | GPIO Available | Notes |
|-----|-------------------|----------------|-------|
| **PB13** | SPI2_SCK | ✅ Yes | Can be configured as GPIO output |
| **PB14** | SPI2_MISO | ✅ Yes | Can be configured as GPIO output |

**Important**: Since SPI2 is not being used, these pins can be safely configured as standard GPIO outputs.

## Hardware Connection

### Circuit Diagram

```
NUCLEO-F411RE                  Optocoupler LEDs          H-Bridge Circuit
─────────────────              ──────────────────        ────────────────

PB13 ──[220Ω]──> Opto1 LED Anode ──> Opto1 Output ──> H-Bridge Input 1
GND ────────────> Opto1 LED Cathode    (Isolated)         (Isolated Power)

PB14 ──[220Ω]──> Opto2 LED Anode ──> Opto2 Output ──> H-Bridge Input 2
GND ────────────> Opto2 LED Cathode    (Isolated)         (Isolated Power)

H-Bridge Output ──> 8Ω Electromagnetic Speaker
```

### Component Requirements

**For each optocoupler LED:**
- **220Ω current-limiting resistor** (standard value, provides ~10mA)
- **Optocoupler** (e.g., TLP281, 4N35, PC817)
  - Forward voltage: ~1.15V
  - Forward current: 5-20mA (10mA recommended)
  - Isolation voltage: 2500-5000V typical

**H-Bridge Circuit:**
- Requires isolated power supply (separate from NUCLEO board)
- Typical H-bridge ICs: L298N, DRV8833, or discrete MOSFETs
- Must handle 8Ω speaker load
- Requires proper flyback protection diodes

## Implementation Code

### Pin Definitions

Add to your pin definitions section:

```c
// H-Bridge sound effects optocoupler LED pins (PB13, PB14)
#define HBRIDGE_OPTO_PB13   (13)   // H-Bridge optocoupler LED 1
#define HBRIDGE_OPTO_PB14   (14)   // H-Bridge optocoupler LED 2
```

### Initialization Function

```c
// Initialize H-Bridge optocoupler LED GPIO pins (PB13, PB14)
void HBridge_Opto_Init(void)
{
    // GPIOB clock should already be enabled for I2C1, but ensure it's on
    RCC->AHB1ENR |= RCC_AHB1ENR_GPIOBEN;
    
    // Configure PB13 as output
    GPIOB->MODER &= ~(3U << (HBRIDGE_OPTO_PB13 * 2));
    GPIOB->MODER |= (1U << (HBRIDGE_OPTO_PB13 * 2));      // Output mode
    GPIOB->OTYPER &= ~(1U << HBRIDGE_OPTO_PB13);           // Push-pull
    GPIOB->OSPEEDR |= (1U << (HBRIDGE_OPTO_PB13 * 2));     // Medium speed
    GPIOB->PUPDR &= ~(3U << (HBRIDGE_OPTO_PB13 * 2));      // No pull-up/pull-do    wn
    
    // Configure PB14 as output
    GPIOB->MODER &= ~(3U << (HBRIDGE_OPTO_PB14 * 2));
    GPIOB->MODER |= (1U << (HBRIDGE_OPTO_PB14 * 2));      // Output mode
    GPIOB->OTYPER &= ~(1U << HBRIDGE_OPTO_PB14);           // Push-pull
    GPIOB->OSPEEDR |= (1U << (HBRIDGE_OPTO_PB14 * 2));     // Medium speed
    GPIOB->PUPDR &= ~(3U << (HBRIDGE_OPTO_PB14 * 2));      // No pull-up/pull-down
    
    // Ensure both LEDs are OFF initially
    HBridge_Opto_Off();
}
```

### Control Functions

```c
// Turn off both H-Bridge optocoupler LEDs
void HBridge_Opto_Off(void)
{
    GPIOB->BSRR = (1U << (HBRIDGE_OPTO_PB13 + 16));  // Reset PB13 (LOW)
    GPIOB->BSRR = (1U << (HBRIDGE_OPTO_PB14 + 16));  // Reset PB14 (LOW)
}

// Turn on PB13 optocoupler LED
void HBridge_Opto1_On(void)
{
    GPIOB->BSRR = (1U << HBRIDGE_OPTO_PB13);  // Set PB13 HIGH
}

// Turn off PB13 optocoupler LED
void HBridge_Opto1_Off(void)
{
    GPIOB->BSRR = (1U << (HBRIDGE_OPTO_PB13 + 16));  // Reset PB13 (LOW)
}

// Turn on PB14 optocoupler LED
void HBridge_Opto2_On(void)
{
    GPIOB->BSRR = (1U << HBRIDGE_OPTO_PB14);  // Set PB14 HIGH
}

// Turn off PB14 optocoupler LED
void HBridge_Opto2_Off(void)
{
    GPIOB->BSRR = (1U << (HBRIDGE_OPTO_PB14 + 16));  // Reset PB14 (LOW)
}

// Pulse PB13 optocoupler LED
void HBridge_Opto1_Pulse(uint32_t duration_ms)
{
    HBridge_Opto1_On();
    delay_ms(duration_ms);
    HBridge_Opto1_Off();
}

// Pulse PB14 optocoupler LED
void HBridge_Opto2_Pulse(uint32_t duration_ms)
{
    HBridge_Opto2_On();
    delay_ms(duration_ms);
    HBridge_Opto2_Off();
}
```

## Sound Effect Examples

### Explosion Sound Effect

```c
// Explosion sound: rapid pulses with varying duration
void SoundEffect_Explosion(void)
{
    // Rapid pulses to create explosion effect
    for (uint8_t i = 0; i < 5; i++)
    {
        HBridge_Opto1_Pulse(10);   // 10ms pulse
        delay_ms(5);
        HBridge_Opto2_Pulse(10);   // Alternate between channels
        delay_ms(5);
    }
    
    // Longer pulse for "boom"
    HBridge_Opto1_Pulse(50);
    HBridge_Opto2_Pulse(50);
    delay_ms(20);
    
    HBridge_Opto_Off();
}
```

### Warning Siren Sound Effect

```c
// Warning siren: alternating pulses to create siren effect
void SoundEffect_WarningSiren(uint32_t duration_ms)
{
    uint32_t end_time = duration_ms;
    uint32_t elapsed = 0;
    uint8_t state = 0;
    
    while (elapsed < end_time)
    {
        if (state == 0)
        {
            HBridge_Opto1_On();
            HBridge_Opto2_Off();
        }
        else
        {
            HBridge_Opto1_Off();
            HBridge_Opto2_On();
        }
        
        delay_ms(50);  // 50ms per cycle (20 Hz siren rate)
        elapsed += 50;
        state = !state;  // Toggle state
    }
    
    HBridge_Opto_Off();
}
```

### Alert Beep Pattern

```c
// Alert beep: short beeps
void SoundEffect_AlertBeep(uint8_t count)
{
    for (uint8_t i = 0; i < count; i++)
    {
        HBridge_Opto1_Pulse(20);   // 20ms beep
        delay_ms(100);              // 100ms pause between beeps
    }
    HBridge_Opto_Off();
}
```

### Low-Frequency Rumble

```c
// Low-frequency rumble: slow alternating pulses
void SoundEffect_Rumble(uint32_t duration_ms)
{
    uint32_t end_time = duration_ms;
    uint32_t elapsed = 0;
    uint8_t state = 0;
    
    while (elapsed < end_time)
    {
        if (state == 0)
        {
            HBridge_Opto1_On();
            HBridge_Opto2_Off();
        }
        else
        {
            HBridge_Opto1_Off();
            HBridge_Opto2_On();
        }
        
        delay_ms(100);  // 100ms per cycle (10 Hz rumble)
        elapsed += 100;
        state = !state;
    }
    
    HBridge_Opto_Off();
}
```

## H-Bridge Control Patterns

### Forward/Reverse Control

The two optocouplers can control H-bridge direction:

```c
// Drive speaker forward (positive direction)
void HBridge_Forward(void)
{
    HBridge_Opto1_On();
    HBridge_Opto2_Off();
}

// Drive speaker reverse (negative direction)
void HBridge_Reverse(void)
{
    HBridge_Opto1_Off();
    HBridge_Opto2_On();
}

// Brake/Stop (both off or both on, depending on H-bridge design)
void HBridge_Stop(void)
{
    HBridge_Opto_Off();
}
```

### PWM-like Effects

For more complex sound effects, you can create PWM-like patterns:

```c
// Create PWM-like effect for volume control
void HBridge_PWM_Effect(uint32_t duration_ms, uint8_t duty_cycle_percent)
{
    uint32_t end_time = duration_ms;
    uint32_t elapsed = 0;
    uint32_t on_time = (duration_ms * duty_cycle_percent) / 100;
    uint32_t off_time = duration_ms - on_time;
    
    while (elapsed < end_time)
    {
        HBridge_Opto1_On();
        delay_ms(on_time);
        HBridge_Opto1_Off();
        delay_ms(off_time);
        elapsed += duration_ms;
    }
}
```

## Integration into Main Code

Add initialization in `main()`:

```c
int main(void)
{
    // ... existing initialization code ...
    
    // Initialize H-Bridge optocoupler LEDs
    HBridge_Opto_Init();
    
    // ... rest of code ...
}
```

## Electrical Specifications

### GPIO Output Capabilities
- **Output Voltage (HIGH)**: 3.3V
- **Output Voltage (LOW)**: 0V
- **Max Sink Current**: 25mA per pin (absolute max)
- **Recommended Current**: 5-10mA per pin
- **Our Design**: ~10mA per pin ✅ Safe

### Optocoupler Requirements
- **Forward Voltage (V_F)**: ~1.15V to 1.5V
- **Forward Current (I_F)**: 5-20mA (10mA recommended)
- **Resistor Calculation**: R = (3.3V - 1.15V) / 0.010A = 215Ω
- **Standard Value**: 220Ω ✅

### Power Considerations
- **GPIO Current**: ~10mA per pin (well within 25mA limit)
- **Total GPIOB Current**: ~20mA (PB13 + PB14) ✅ Safe
- **H-Bridge Power**: Requires separate isolated power supply

## Safety and Protection

### GPIO Protection
- ✅ **220Ω resistors** provide current limiting
- ✅ **Push-pull output mode** provides clean signals
- ✅ **No additional protection needed** for GPIO side

### H-Bridge Protection
- ⚠️ **Flyback diodes** required on H-bridge outputs
- ⚠️ **Isolated power supply** required for H-bridge
- ⚠️ **Current limiting** in H-bridge circuit for speaker protection

## Summary

**✅ PB13 and PB14 are available and suitable for optocoupler LED outputs**

**Advantages:**
- ✅ No conflicts with current project
- ✅ Same port (GPIOB) - easier configuration
- ✅ Physically adjacent pins (likely on same connector)
- ✅ Standard GPIO output capability
- ✅ Can create various sound effects through H-bridge control

**Implementation:**
1. Add pin definitions for PB13 and PB14
2. Initialize as GPIO outputs (push-pull, medium speed)
3. Use 220Ω current-limiting resistors
4. Connect to optocouplers
5. Drive H-bridge circuit for speaker sound effects

**Sound Effects Possible:**
- Explosions (rapid pulses)
- Warning sirens (alternating tones)
- Alert beeps (patterned pulses)
- Low-frequency rumbles (slow alternation)
- PWM-like volume effects
- Directional sound effects (forward/reverse)

The pins are ready to use for your sound effects system!

