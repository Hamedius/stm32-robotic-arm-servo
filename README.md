# STM32 4-DOF Robotic Arm Control

This project implements a **4‑degree‑of‑freedom robotic arm** controlled using an **STM32 microcontroller**.  
Each joint of the arm is driven by a servo motor, and the position of every servo is controlled through corresponding analog inputs (potentiometers) read via ADC channels.  
The STM32 processes these inputs and generates PWM signals that precisely position the arm.

This repository contains the firmware and structure for a clean, reproducible implementation based on standard STM32Cube HAL configuration.

---

##  Project Overview

The robotic arm has four controllable axes, each actuated by a servo motor.  
The controller:

1. Continuously reads analog inputs from potentiometers.  
2. Maps each value (0–4095) to servo angles (0°–180°).  
3. Converts the angle into a PWM compare value (CCR) that drives each servo.  
4. Updates servo positions in real time.

This setup demonstrates a full signal chain:

```
Potentiometer → STM32 ADC → Angle Mapping → PWM Output → Servo → Arm Motion
```

---

##  Hardware

- STM32 Nucleo or similar STM32 microcontroller board  
- Four standard RC servo motors  
- Four potentiometers (for manual joint control)  
- External power supply capable of powering all servos  
- Breadboard or PCB for wiring  

### Microcontroller Peripheral Mapping

- **Joint 1**  
  - ADC input: e.g., PA0  
  - PWM output: TIM channel for servo 1  

- **Joint 2**  
  - ADC input: e.g., PA4  
  - PWM output: TIM channel for servo 2  

- **Joint 3**  
  - ADC input: e.g., PA9  
  - PWM output: TIM channel for servo 3  

- **Joint 4**  
  - ADC input: board‑dependent  
  - PWM output: TIM channel for servo 4  

### PWM Parameters

All four servos use:

- **PWM frequency:** 50 Hz  
- **Prescaler:** chosen so that timer ticks match required resolution  
- **ARR:** configured for a 20 ms PWM period  

Pulse‑width mapping:

| Servo angle | Pulse width | CCR value |
|-------------|-------------|-----------|
| 0°          | 0.5 ms      | ≈ 2.5%    |
| 90°         | 1.5 ms      | ≈ 7.5%    |
| 180°        | 2.5 ms      | ≈ 12.5%   |

---

##  Repository Structure

```
stm32-robotic-arm-4dof/
├── README.md
├── docs/
│   └── Robotic arm.pdf
└── src/
    └── main.c
```

---

##  Firmware Logic

For each of the four joints:

1. Read ADC input:  
   ```c
   uint16_t adcValue = HAL_ADC_GetValue(&hadcX);    // 0–4095
   ```
2. Convert to angle:  
   ```c
   uint16_t angle = adcValue * 180 / 4096;
   ```
3. Convert angle to PWM compare value:  
   ```c
   uint16_t ccr = MIN_CCR + (MAX_CCR - MIN_CCR) * angle / 180;
   ```
4. Update PWM channel:  
   ```c
   __HAL_TIM_SET_COMPARE(&htimX, TIM_CHANNEL_Y, ccr);
   ```

This loop updates all servos continuously for real‑time movement.

---

##  How to Build & Run

1. Create a project in **STM32CubeIDE**.  
2. Configure four ADC inputs and four TIM PWM channels.  
3. Generate initialization code from CubeMX.  
4. Copy the provided `main.c` into `Core/Src/`.  
5. Compile and flash the firmware.  
6. Power the servos from a stable 5‑6V supply.  
7. Adjust potentiometers to move the robotic arm joints.

---

##  Documentation

The report included in the `docs/` folder contains diagrams, wiring references, and project details.

---

##  Authors

- **Hamed Nahvi**  
- **Yasin Shadrouh**
