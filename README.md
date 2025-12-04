# Robotic Arm Servo Motor Controlling

This project implements a **three-servo robotic arm** controlled by an **STM32 microcontroller**.  
Each joint of the arm is driven by a 9g servo motor, and its position is adjusted using a dedicated potentiometer.  
The microcontroller reads the potentiometers through ADC channels and generates PWM signals for the servos.

The design and configuration follow the specifications in *Robotic arm.pdf*.

---

## Project Overview

The system controls three servo motors corresponding to three joints of a robotic arm.  
For each joint:

1. A potentiometer provides an analog voltage proportional to the desired angle.
2. The STM32 reads this value via an ADC channel (0–4096).
3. The value is mapped linearly to an angle in the range 0–180°.
4. The angle is converted into a PWM duty cycle for driving the servo.

The PWM is generated at **50 Hz** with a **20 ms period**, suitable for standard RC servos.

---

## Hardware

- STM32 Nucleo-G474RE (or similar STM32 microcontroller board)
- Three 9g servo motors
- Three 5 kΩ potentiometers
- Power supply suitable for servos and MCU
- Breadboard or PCB wiring

### Pin and Peripheral Mapping

- Potentiometer 1 → **PA0** → `ADC1_IN1` → controls Servo 1 via `TIM2_CH2` (PA1)  
- Potentiometer 2 → **PA4** → `ADC2_IN1` → controls Servo 2 via `TIM3_CH3` (PB0)  
- Potentiometer 3 → **PA9** → `ADC5_IN1` → controls Servo 3 via `TIM1_CH1` (PC0)  

Clock configuration: **HCLK = 45 MHz**  

All timers use:

- Prescaler `PSC = 900 - 1`
- Auto-reload `ARR = 1000 - 1`

Resulting in a **50 Hz PWM** frequency.

---

## Repository Structure

```text
stm32-robotic-arm-servo/
├── README.md
├── docs/
│   └── Robotic arm.pdf
└── src/
    └── main.c
```

---

## Firmware Logic

The main control loop:

1. Starts ADC conversion for each potentiometer.  
2. Polls until the value is ready.  
3. Maps ADC 0–4096 → 0–180°.  
4. Converts angle → PWM compare value (CCR):  
   ```c
   Angle = 25 + (100 * Degree) / 180;
   ```
5. Writes CCR value to each servo timer channel.

Pulse-width interpretation:

- `CCR = 25` → 0.5 ms → 0°  
- `CCR = 75` → 1.5 ms → 90°  
- `CCR = 125` → 2.5 ms → 180°  

for a 20 ms PWM period.

---

## Usage

1. Create a new STM32 project in STM32CubeMX / STM32CubeIDE.  
2. Configure:
   - ADC1 on PA0  
   - ADC2 on PA4  
   - ADC5 on PA9  
   - TIM2_CH2 on PA1  
   - TIM3_CH3 on PB0  
   - TIM1_CH1 on PC0  
   - System clock at HCLK = 45 MHz  
3. Let CubeMX generate initialization code.  
4. Replace the generated `main.c` with `src/main.c`.  
5. Build and flash to the MCU.  
6. Adjust potentiometers to move each joint of the arm.

---

## Authors

- **Hamed Nahvi**  
- **Yasin Shadrouh**

