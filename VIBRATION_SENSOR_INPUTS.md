# Vibration Sensor Input System with Interrupt-Driven Buzzer Alerts

## Overview

This system implements 4 vibration sensor inputs connected to TLP281-4 optocoupler outputs. Each input triggers a unique interrupt that activates a warning siren on a buzzer directly attached to the MCU.

## GPIO Pin Selection

**Selected Pins: PB10, PB11, PB12, PB15**

| Pin | Function | Port | Notes |
|-----|----------|------|-------|
| **PB10** | Vibration sensor input 1 | GPIOB | EXTI10 |
| **PB11** | Vibration sensor input 2 | GPIOB | EXTI11 |
| **PB12** | Vibration sensor input 3 | GPIOB | EXTI12 |
| **PB15** | Vibration sensor input 4 | GPIOB | EXTI15 |

**Why these pins?**
- ✅ All on same port (GPIOB) - easier configuration
- ✅ Support EXTI interrupts (lines 10, 11, 12, 15)
- ✅ No conflicts with current project:
  - PB8/PB9: I2C1 (LCD)
  - PB13/PB14: H-Bridge optocoupler LEDs
- ✅ Physically adjacent pins (likely on same connector)

**Buzzer Output: PA7**
- ✅ Available GPIO pin
- ✅ Can use TIM3 CH2 for PWM (optional upgrade)
- ✅ Currently using simple GPIO toggle

## Hardware Connection

### TLP281-4 Optocoupler Output Side

```
TLP281-4 Optocoupler (Output Side)    NUCLEO-F411RE
─────────────────────────────────     ──────────────

Ch1 Collector (Pin 4) ──────────────> PB10
Ch1 Emitter (Pin 3) ─────────────────> GND

Ch2 Collector (Pin 6) ──────────────> PB11
Ch2 Emitter (Pin 5) ─────────────────> GND

Ch3 Collector (Pin 8) ──────────────> PB12
Ch3 Emitter (Pin 7) ─────────────────> GND

Ch4 Collector (Pin 10) ─────────────> PB15
Ch4 Emitter (Pin 9) ─────────────────> GND

Vibration Sensor ──> TLP281-4 Input Side (isolated)
```

### Buzzer Connection

```
NUCLEO-F411RE          Buzzer
──────────────         ───────

PA7 ──────────────────> Buzzer (+) terminal
GND ──────────────────> Buzzer (-) terminal

Note: Buzzer should be rated for 3.3V operation
      Typical: 3-5V active buzzer
```

## Electrical Specifications

### Input Configuration
- **Mode**: Input with pull-down
- **Trigger**: Rising edge (optocoupler output goes HIGH)
- **Voltage Levels**:
  - LOW: 0V (pull-down active)
  - HIGH: 3.3V (when optocoupler activates)
- **Interrupt**: EXTI (External Interrupt) on rising edge

### Buzzer Specifications
- **Type**: Active buzzer (3.3V)
- **Current**: Typically 5-30mA
- **Frequency**: Software-generated (800-1200 Hz for siren)
- **Control**: GPIO toggle (PA7)

**Note**: If buzzer draws >25mA, use a transistor driver circuit.

## Interrupt System

### EXTI Configuration

**Interrupt Lines:**
- PB10 → EXTI10
- PB11 → EXTI11
- PB12 → EXTI12
- PB15 → EXTI15

**Interrupt Vector**: `EXTI15_10_IRQHandler`
- All 4 lines share the same interrupt handler
- Handler checks which line triggered and responds accordingly

**Trigger Type**: Rising edge only
- Optocoupler output goes HIGH when vibration detected
- Pull-down ensures clean LOW state when inactive

### Interrupt Priority

- **NVIC Interrupt Number**: 40 (EXTI15_10_IRQn)
- **Priority**: Default (can be configured if needed)
- **Handler**: Non-blocking (triggers buzzer, returns quickly)

## Software Implementation

### Initialization

```c
void VibrationSensor_Init(void)
{
    // Configure GPIO pins as inputs with pull-down
    // Map pins to EXTI lines via SYSCFG
    // Enable rising edge triggers
    // Enable EXTI interrupts
    // Enable NVIC interrupt
}
```

### Interrupt Handler

```c
void EXTI15_10_IRQHandler(void)
{
    // Check which line triggered (PB10, PB11, PB12, or PB15)
    // Clear pending bit
    // Trigger buzzer warning siren (500ms)
}
```

### Buzzer Functions

```c
void Buzzer_Init(void)           // Initialize PA7 as output
void Buzzer_On(void)             // Turn buzzer on
void Buzzer_Off(void)            // Turn buzzer off
void Buzzer_GenerateTone(freq, duration)  // Generate tone
void Buzzer_WarningSiren(duration)       // Warning siren effect
```

## Buzzer Warning Siren

The buzzer warning siren uses frequency modulation:
- **Frequency Range**: 800 Hz → 1200 Hz
- **Pattern**: Smooth up/down sweeps
- **Duration**: 500ms per trigger
- **Sound**: Classic "wee-woo" siren pattern

**Implementation**:
- Uses software PWM (GPIO toggle)
- 20 steps per sweep direction
- Smooth frequency transitions

## Operation Flow

1. **Vibration Detected**: Sensor triggers optocoupler input
2. **Optocoupler Activates**: Output goes HIGH (3.3V)
3. **GPIO Pin Detects**: PB10/PB11/PB12/PB15 sees rising edge
4. **EXTI Triggers**: Interrupt fires
5. **Handler Executes**: `EXTI15_10_IRQHandler()` called
6. **Buzzer Activates**: Warning siren plays for 500ms
7. **Interrupt Returns**: System continues normal operation

## Timing Considerations

### Interrupt Response Time
- **Latency**: <1µs typical (Cortex-M4)
- **Handler Execution**: ~500ms (buzzer siren duration)
- **Non-blocking**: Interrupts can nest if multiple sensors trigger

### Debouncing
- **Hardware**: Optocoupler provides some filtering
- **Software**: None currently (can add if needed)
- **Spiky Pulses**: Brief pulses are handled correctly by EXTI

## Protection and Safety

### Input Protection
- ✅ **Pull-down resistors**: Ensure clean LOW state
- ✅ **Optocoupler isolation**: Protects MCU from sensor side
- ✅ **3.3V tolerant**: GPIO pins handle optocoupler output

### Buzzer Protection
- ⚠️ **Current limiting**: Check buzzer current rating
- ⚠️ **If >25mA**: Use transistor driver circuit
- ✅ **GPIO protection**: Standard GPIO output protection

## Example Transistor Driver (if needed)

If buzzer draws >25mA:

```
PA7 ──[1kΩ]──> 2N2222 Base
                  │
                  │ Collector ──> Buzzer (+)
                  │     │
                  │     └──> +3.3V
                  │
                  └──> Emitter ──> GND

Buzzer (-) ──> GND
```

## Testing

### Test Sequence
1. **Initialize system**: Call `VibrationSensor_Init()` and `Buzzer_Init()`
2. **Trigger sensor 1**: Apply signal to PB10 → Should hear buzzer siren
3. **Trigger sensor 2**: Apply signal to PB11 → Should hear buzzer siren
4. **Trigger sensor 3**: Apply signal to PB12 → Should hear buzzer siren
5. **Trigger sensor 4**: Apply signal to PB15 → Should hear buzzer siren

### Debugging
- **No interrupt**: Check EXTI configuration, SYSCFG mapping
- **No buzzer sound**: Check PA7 connection, buzzer power
- **Multiple triggers**: Check for noise/bounce on input lines

## Summary

**GPIO Pins Selected:**
- ✅ PB10, PB11, PB12, PB15 (vibration sensor inputs)
- ✅ PA7 (buzzer output)

**Features:**
- ✅ 4 independent interrupt-driven inputs
- ✅ Rising edge trigger (optocoupler output HIGH)
- ✅ Pull-down configuration (clean LOW state)
- ✅ Unique interrupt per pin (shared handler)
- ✅ Buzzer warning siren (500ms, frequency-modulated)
- ✅ Non-blocking operation

**Hardware Requirements:**
- TLP281-4 optocoupler module (output side connected)
- Vibration sensors (connected to optocoupler input side)
- 3.3V active buzzer (connected to PA7)
- Pull-down resistors (internal, already configured)

The system is ready to detect vibrations and trigger buzzer alerts!

