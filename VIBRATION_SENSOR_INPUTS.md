# Vibration Sensor Input System with Interrupt-Driven Buzzer Alerts

## Overview

This system implements 4 vibration sensor inputs connected to TLP281-4 optocoupler outputs. Each input triggers a unique interrupt that activates a warning siren on a buzzer directly attached to the MCU.

## GPIO Pin Selection (Current Implementation)

**Selected Pins: PA0, PA1, PA2, PA3**

| Pin | Function | EXTI | NUCLEO-F411RE connector |
|-----|----------|------|--------------------------|
| **PA0** | Vibration sensor input 1 | EXTI0 | Arduino A0 ✅ |
| **PA1** | Vibration sensor input 2 | EXTI1 | Arduino A1 ✅ |
| **PA2** | Vibration sensor input 3 | EXTI2 | **Morpho only** (not Arduino A2) |
| **PA3** | Vibration sensor input 4 | EXTI3 | **Morpho only** (not Arduino A3) |

### ⚠️ NUCLEO-F411RE connector wiring (why only one sensor may work)

On the **Arduino Uno R3** connector, the analog row is often:

- **A0 = PA0**, **A1 = PA1** → correct for sensors 1 and 2.
- **A2 = PA4** and **A3 = PB0** on many NUCLEO-64 boards — **not** PA2/PA3.

The firmware configures **PA0, PA1, PA2, PA3**. If you connect all four optocoupler outputs to Arduino pins A0, A1, A2, A3, then only the sensor on **A0 (PA0)** — and possibly A1 (PA1) — is actually driven; A2 and A3 are wired to PA4 and PB0, which are not used by this code. Result: only one (or two) sensors appear to trigger.

**Fix:** Use the **Morpho** extension headers for **PA2** and **PA3** (sensors 3 and 4). Connect:

- Sensor 1 → **PA0** (Arduino A0)
- Sensor 2 → **PA1** (Arduino A1)
- Sensor 3 → **PA2** (Morpho header, not Arduino A2)
- Sensor 4 → **PA3** (Morpho header, not Arduino A3)

**Why these pins?**
- ✅ EXTI0–EXTI3 (one interrupt vector per pin)
- ✅ All on GPIOA; single EXTICR1 setting
- ✅ No conflict with I2C (PB8/PB9), H-Bridge (PB13/PB14), etc.

**Buzzer Output: PA7**
- ✅ Available GPIO pin
- ✅ Can use TIM3 CH2 for PWM (optional upgrade)
- ✅ Currently using simple GPIO toggle

## Hardware Connection

### TLP281-4 Optocoupler Output Side (active LOW → falling edge)

Optocoupler outputs pull the MCU pin to GND when active. MCU pins use internal pull-up; trigger is **falling edge**.

```
TLP281-4 Optocoupler (Output Side)    NUCLEO-F411RE
─────────────────────────────────     ──────────────

Ch1 Collector (Pin 4) ──────────────> PA0 (Arduino A0)
Ch1 Emitter (Pin 3) ─────────────────> GND

Ch2 Collector (Pin 6) ──────────────> PA1 (Arduino A1)
Ch2 Emitter (Pin 5) ─────────────────> GND

Ch3 Collector (Pin 8) ──────────────> PA2 (Morpho header)
Ch3 Emitter (Pin 7) ─────────────────> GND

Ch4 Collector (Pin 10) ─────────────> PA3 (Morpho header)
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
- **Mode**: Input with pull-up (optocoupler pulls to GND when active)
- **Trigger**: Falling edge (pin goes from HIGH to LOW when optocoupler activates)
- **Voltage Levels**:
  - HIGH: 3.3V (pull-up when optocoupler inactive)
  - LOW: 0V (when optocoupler activates)
- **Interrupt**: EXTI (External Interrupt) on falling edge

### Buzzer Specifications
- **Type**: Active buzzer (3.3V)
- **Current**: Typically 5-30mA
- **Frequency**: Software-generated (800-1200 Hz for siren)
- **Control**: GPIO toggle (PA7)

**Note**: If buzzer draws >25mA, use a transistor driver circuit.

## Interrupt System

### EXTI Configuration

**Interrupt Lines (separate vectors):**
- PA0 → EXTI0 → `EXTI0_IRQHandler`
- PA1 → EXTI1 → `EXTI1_IRQHandler`
- PA2 → EXTI2 → `EXTI2_IRQHandler`
- PA3 → EXTI3 → `EXTI3_IRQHandler`

**Trigger Type**: Falling edge only
- Optocoupler output pulls pin to GND when vibration detected
- Pull-up ensures clean HIGH state when inactive

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
2. **Trigger sensor 1**: Pull PA0 low (or tap sensor 1) → Should trigger
3. **Trigger sensor 2**: Pull PA1 low → Should trigger
4. **Trigger sensor 3**: Pull PA2 low (Morpho) → Should trigger
5. **Trigger sensor 4**: Pull PA3 low (Morpho) → Should trigger

### Debugging
- **Only one sensor works**: Ensure PA2 and PA3 are wired from **Morpho** headers, not Arduino A2/A3 (those are PA4 and PB0).
- **No interrupt**: Check EXTI configuration, SYSCFG EXTICR[0]=0, NVIC enabled for IRQ 6–9.
- **No buzzer sound**: Check PA7 connection, buzzer power.
- **Multiple triggers**: Check for noise/bounce on input lines.

## Summary

**GPIO Pins Selected:**
- ✅ PA0, PA1, PA2, PA3 (vibration sensor inputs; PA2/PA3 from Morpho)
- ✅ PA7 (buzzer output)

**Features:**
- ✅ 4 independent EXTI lines (EXTI0–EXTI3, one handler per pin)
- ✅ Falling edge trigger (optocoupler pulls pin to GND)
- ✅ Pull-up configuration (clean HIGH when inactive)
- ✅ Buzzer / game effects on trigger; non-blocking where possible

**Hardware Requirements:**
- TLP281-4 optocoupler module (output side connected)
- Vibration sensors (connected to optocoupler input side)
- 3.3V active buzzer (connected to PA7)
- Pull-down resistors (internal, already configured)

The system is ready to detect vibrations and trigger buzzer alerts!

