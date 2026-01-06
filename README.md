# Microstepping Stepper Motor Control on STM32 (STM32H7)

Course project (**Microcontroller / IMEC, SS25**) implementing a **stepper motor controller** with **full-step and microstepping** on an **STM32H7 (STM32H745)** platform.

The firmware provides a small on-device UI (buttons + rotary encoder + LEDs + 7‑segment display via DAC), supports **Rapid parameter tuning** (amplitude / target speed / target angle), and logs/derives **current, speed, and angle** from ADC and encoder signals.

---

## Key features

- **Full-step + microstepping**
  - Toggle **Full ↔ Micro** during runtime
  - Microstepping based on **sine/cosine lookup tables** (scaled to ±10000) for smooth phase actuation
  - Step resolution constants:
    - `VOLL_SCHRITTE_UMDR = 200`
    - `MIKRO_SCHRITTE_UMDR = 6400`
- **Motor actuation via two PWM bridges (phase A/B)**
  - Per-bridge enable/disable and error indication
- **User interface**
  - Buttons (S1/S2/S3 + BLUE key), rotary encoder, LEDs 1–7
  - 7‑segment display driven through a **calibrated DAC mapping (0…10 V in 1 mV steps)**
- **Sensing & derived signals**
  - **ADC**: potentiometer (speed), phase current A/B (offset corrected)
  - **Encoder**: speed (averaged over cycles) and angle (single-turn + multi-turn)

---

## Operating modes & controls

The application is structured into three modes:

- `RUN_MODE` – normal operation (motor may run)
- `CONFIG_MODE` – browse/select parameters
- `CHANGE_CONFIG_MODE` – change the selected parameter value

### Button mapping

| Control | RUN_MODE | CONFIG_MODE | CHANGE_CONFIG_MODE |
|---|---|---|---|
| **S1** | Start motor **CW** | – | Abort change |
| **S2** | Start motor **CCW** | Enter CHANGE_CONFIG_MODE | – |
| **S3** | Open CONFIG_MODE | Back to RUN_MODE | – |
| **BLUE** | Toggle **Full ↔ Micro** | Set both PWM bridges to **0%** | Save change |
| **Rotary encoder** | Browse displayed values | Browse displayed values | Change parameter value |
| **Potentiometer** | Set target speed | Set target speed | – |

> Note: In microstepping mode, the target angle is quantized to the microstep grid, so the exact requested angle may be approximated.

---

## Display & LEDs

### Menu parameters (LED1–LED4 binary code)

The menu cycles through these values:

1. Target amplitude  
2. Target speed  
3. Target angle  
4. Current A  
5. Current B  
6. Angle (single turn)  
7. Angle (multi turn)  
8. Actual speed  

### Status LEDs

- **LED5**: on if an H‑bridge driver error is present
- **LED6**: mode indicator  
  - off: RUN_MODE  
  - on: CONFIG_MODE  
  - blinking: CHANGE_CONFIG_MODE
- **LED7**: step mode indicator (Micro-step vs Full-step)

---

## Timing

- Main loop cycle time: **0.75 ms** (`ZYKLUS_ZEIT_MAIN = 750 µs`)
- PWM update and motor control are driven via timers (see `tim.c`)

---

## Project structure

This repository is a STM32CubeIDE project:

```
Projektarbeit_SS25/
├─ CM7/
│  ├─ Core/Inc/     (adc.h, dac.h, gpio.h, tim.h, general.h, ...)
│  └─ Core/Src/     (main.c, adc.c, dac.c, gpio.c, tim.c, ...)
└─ CM4/             (auxiliary core project)
```

The application logic lives mainly in:
- `CM7/Core/Src/main.c` – mode state machine, UI handling, main loop
- `CM7/Core/Src/tim.c` – PWM generation, motor stepping (full/micro), encoder handling
- `CM7/Core/Src/adc.c` / `dac.c` / `gpio.c` – peripherals + helpers

---

## Build & flash

1. Open the project in **STM32CubeIDE**.
2. Build the **CM7** target.
3. Flash via **ST‑LINK** to the STM32H745 board.
4. Connect the lab hardware (stepper motor + bridges, encoder, UI elements) as defined in the CubeMX configuration (`main.h` pin mapping).

---

## Notes

- Phase current measurement is offset-corrected and used as absolute magnitude for display.
- The BLUE key in CONFIG_MODE forces PWM outputs to 0% (safety / quick stop).
- Microstepping uses lookup-based phase values (sin/cos) to reduce torque ripple compared to full-step.

---

## Authors

Kadri Bajrami • Robin Bruns
