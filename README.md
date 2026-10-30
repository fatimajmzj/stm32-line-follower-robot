# Line Follower Robot (STM32F411RE)

Line follower robot built on an **STM32F411RE**. Two **CNY70 reflectance sensors** are read through the ADC with **DMA**, and the robot decides how to steer two DC motors driven by an H-bridge with **PWM speed control**. LEDs show what each sensor detects and a buzzer signals that the robot is ready to start.

<img width="1198" height="1017" alt="image" src="https://github.com/user-attachments/assets/be27c8f8-094a-4d11-9ca0-0230e9acf4e9" />
<img width="1199" height="868" alt="image" src="https://github.com/user-attachments/assets/b91efed3-fc03-49e1-a741-c71723251ca7" />
<img width="1199" height="985" alt="image" src="https://github.com/user-attachments/assets/16fbec29-bdd7-4d1d-90de-f5d917d392b6" />



## Features

- Two reflectance sensors (CNY70) sampled by **ADC1 in continuous scan mode with DMA** (12 bits), so reading the sensors needs no CPU time.
- Simple, fast **steering logic** based on a threshold on each sensor (forward, turn left, turn right, stop).
- **PWM speed control** (1 kHz, TIM2) on both motors, with direction set through the H-bridge input pins.
- **Visual feedback**: one yellow and one red LED per side, showing the state of each sensor.
- **Buzzer** that sounds at start-up.
- Serial debugging at 115200 baud (`printf` over USART2) with the raw sensor readings.

## Hardware

| Component | Details |
|---|---|
| Microcontroller | STM32F411RE, 84 MHz (external oscillator + PLL) |
| Sensors | 2 x CNY70 reflectance sensors (analog output) |
| Motor driver | Dual H-bridge *(add the model, e.g. L298N / TB6612)* |
| Motors | 2 DC gear motors *(add model and rated RPM)* |
| Indicators | 2 yellow LEDs, 2 red LEDs, buzzer |
| Power | *(add battery and converters)* |

### Pinout

| Function | Pin | Peripheral |
|---|---|---|
| Left sensor | PA0 | ADC1 channel 0 |
| Right sensor | PA1 | ADC1 channel 1 |
| Motor A PWM | PA15 | TIM2 CH1 |
| Motor B PWM | PB3 | TIM2 CH2 |
| Motor A direction (IN1 / IN2) | PB0 / PB1 | GPIO |
| Motor B direction (IN3 / IN4) | PB2 / PB10 | GPIO |
| Left LEDs (yellow / red) | PA10 / PB8 | GPIO |
| Right LEDs (yellow / red) | PA11 / PB9 | GPIO |
| Buzzer | PA7 | GPIO |
| Serial TX / RX | PA2 / PA3 | USART2 (115200 baud) |

## How it works

### Sensing

Both sensors are converted continuously by ADC1 and the results are stored in memory by DMA. When a full conversion cycle completes, a flag is raised and the main loop reads the latest values. A reading above **2000** (out of 4095) on a sensor is treated as "active" for that side, which also turns on its yellow LED (otherwise the red LED is on).

### Steering logic

The decision is taken from the state of both sensors every time new data arrives. Speeds go from 0 to 100 (%), which maps to the PWM duty cycle.

| Left sensor | Right sensor | Action | Motor A | Motor B |
|---|---|---|---|---|
| below 2000 | below 2000 | Forward | 50 | 50 |
| above 2000 | below 2000 | Turn right | 60 | 30 |
| below 2000 | above 2000 | Turn left | 30 | 60 |
| above 2000 | above 2000 | Stop | 0 | 0 |

### Motor control

`MotorA_Set()` and `MotorB_Set()` take a signed speed: the sign selects the direction (IN pins) and the magnitude sets the PWM duty cycle on TIM2 (1 kHz, 1000 steps). A speed of 0 turns both inputs off and sets the duty cycle to zero.

## Project structure

```
Core/
  Inc/        Headers (main.h, HAL configuration)
  Src/
    main.c      Peripheral init, sensor reading and steering logic
Drivers/      STM32 HAL / CMSIS
Proyecto_seguidorL.ioc   STM32CubeMX configuration
```

## Build and flash

1. Install [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html).
2. Clone the repository:
   ```bash
   git clone https://github.com/fatimajmzj/stm32-line-follower-robot.git
   ```
3. In STM32CubeIDE: **File → Import → Existing Projects into Workspace** and select the cloned folder.
4. Build the project and flash it with an ST-LINK (**Run → Debug / Run**).
5. Open a serial terminal at 115200 baud to see the sensor readings.

## Results

*Add here a video/GIF of the robot following a line.*

## Possible improvements

- Replace the threshold logic with a **PID controller** using a weighted sensor array, for smoother and faster tracking.
- Add more sensors to detect sharp curves and intersections.
- Calibrate the threshold automatically at start-up.

## License

See the [LICENSE](LICENSE) file.
