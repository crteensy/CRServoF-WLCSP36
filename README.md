# CRServoF-WLCSP36
CRSF to Servo PWM converter, using an STM32F051 in WLCSP-36

This is a different take on https://github.com/CapnBry/CRServoF, but with major differences - so no fork.

## Purpose and features
Many modern RC receivers have serial output and a number of different serial protocols. A popular one is Crossfire (or CSRF). However, in anything other than multicopters that need a flight controller, these receivers are not very useful because they don't have per-channel PWM output for servos and ESCs.

Theses RC systems also feature telemetry to get data about battery state and many other things on the ground during flight. There are a number of serial protocols for attaching sensors to a receiver for anything that the receiver can't measure itself.

Features:
- STM32F051 in WLCSP-36 *
- CRSF compatible header to attach a receiver to
- 6 Servo PWM outputs
- extra UART header for sensor communication
- Supply for external sensors, either 3V3 or 5V (solder jumper)
- VBat sensing input to an ADC channel, with an on-board divider and filter cap
- LEDs on BEC and 3V3 rails
- SWD header

It is powered by an external BEC (usually the ESC should bring one). There is no regulator on board that will handle anything above 5V. Of course it can run off a 1S LiIon, LiPo or similar battery.

\*) You might ask why I used this package. It's simple: I didn't have any other at hand, and couldn't order any similar STM32 in a more friendly package. It can still be soldered at home, though!

## Layout

The layout is simple and doesn't require special PCB fabrication technology:
- min track width / clearance: 0.1 mm
- standard vias, 0.2 mm drill / 0.5 mm outer diameter, no tenting or similar extras
- 4 layers
- single sided assembly, 0402 or larger passives

In order to route signals that do not come from pins along the perimeter, multiple pins are shorted in order to reach the perimeter. Only one of these pins may drive it at any given time.

MCU Pinout and extra pins used to route the design:

| Function | Pin (GPIO) | Extra Pins (GPIOs) | Comment
|---|---|---|---|
| NRST     | D5 | C6 (PC15) |
| BOOT0    | B4 | B5 (PF0), A6 (PC13) | 10k Pull-down
| PWM x    | F3 (PB0) |   | TIM3_CH3, 1k series resistor
| PWM x    | F2 (PB1) |   | TIM3_CH4, 1k series resistor
| PWM x    | B3 (PB3) | A4 (PB7) | TIM2_CH2, 1k series resistor
| PWM x    | A3 (PB4) |   |  TIM3_CH1, 1k series resistor
| PWM x    | A2 (PA15) |   | TIM2_CH1, 1k series resistor (also boot loader UART Rx)
| PWM x    | D1 (PA9) |   | TIM1_CH2, 1k series resistor
| SWCLK    | B2 (PA14) | A1 (PA12) | (also boot loader Tx, 1k series resistor)
| SWDIO    | B1 (PA13) |   |
| UART1 Tx | C4 (PB6) | C5 (PF1), B6 (PC14)  | 1k series resistor
| UART1 Rx | C1 (PA10) |   |
| UART2 Tx | E4 (PA2) | F4 (PA7) | CRSF header, 1k series resistor
| UART2 Rx | F5 (PA3) |   | CRSF header
| ADC      | F6 (PA0) |   | ADC_IN0, 10k/1k divider and filter cap
| VDD,VDDA | A5, E1, E5 | E6 (PB5) | supply
| VSS, VSSA | F1, D6 |   | GND

The boot loader UART (UART2) on PA14/PA15 can be accessed through a dedicated set of pads on the bottom. One of them (UART2_Tx) is shared with a servo output, but that doesn't hurt during programming.

If the battery voltage (to be measured with the ADC) is accidentally connected to a servo output, this will probably not cause damage because all PWM outputs have a 1k series resistor. Unless it's connected in reverse. That would be bad, even on the VBat header.
