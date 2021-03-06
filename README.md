## Hoverboard-Board-Hack

This repo contains open source firmware for generic Hoverboard Mainboards.
the firmware you can find here allows you to use your Hoverboard Hardware (like the Mainboard, Motors and Battery) for cool projects like driving armchairs, person-tracking transportation robots and every other application you can imagine that requires controlling the Motors.

---

#### Hardware
![otter](https://raw.githubusercontent.com/NiklasFauth/Hoverboard-Board-Hack/master/pinout.png)

The original Hardware supports two 4-pin cables that originally were connected to the two sensor boards. They break out GND, 12/15V and USART2&3 of the Hoverboard mainboard.
Both USART2 & 3 can be used for UART and I2C, PA2&3 can be used as 12bit ADCs.

---

#### Flashing
To build the firmware, just type "make". Make sure you have specified your gcc-arm-none-eabi binary location in the Makefile. Right to the STM32, there is a debugging header with GND, 3V3, SWDIO and SWCLK. Connect these to your SWD programmer, like the ST-Link found on many STM devboards.

Make sure you hold the powerbutton or connect a jumper to the power button pins while flashing the firmware, as the STM might release the power latch and switches itself off during flashing.

To flash the STM32, use the ST-Flash utility (https://github.com/texane/stlink).

If you never flashed your mainboard before, the STM is probably locked. To unlock the flash, use the following OpenOCD command:

```
openocd -f interface/stlink-v2.cfg -f target/stm32f1x.cfg -c init -c "stm32f1x unlock 0"
```
Then you can simply flash the firmware:
```
st-flash write build/hoverboard.bin 0x8000000
```

---
#### Troubleshooting
First, check that power is connected and voltage is > 36V.
If the board draws more than 100mA in idle, it's probably broken.

If the motors do something, but don't rotate smooth and quietly, try to use the alternative phase mapping in the picture above. Some hoverboards have a different layout then others, and this might be the reason your motor isn't spinning.

---


#### Examples

There are currently 3 Branches in this repo: master, PPM and GameTrak.

##### master

Master uses UART DMA to receive two floats on USART2 with 115200 baud, and uses these floats to control the motor current of each motor. It also counts the hall ticks and sends back the motor positions to use them for odometry data, for example combined with ROS.

Simply connect a USB-USART adapter to GND; PA2 and PA3 and give it a try.

##### PPM

PPM allows you to connect a radio control with PPM sum signal output and use it to directly control and steer the motors. Channel 0 is used for motor current scaling, Ch 1&2 for speed / direction. You can change that as you want in the main.c fiile.

Connect your RC receiver to GND and 12V (if is supports 12V input voltage, else use a separate voltage regulator or use the hall sensors rail) and the PPM sum signal to PA3. The firmware also streams out the received channel values on USART3 / 115200 baud for debugging reasons.

A 510 Ohm resistor and a 10pF ceramic cap in parallel with the PPM input (directly soldered between PA3 and GND on the pcb) improves noise rejection significantly.

This firmware also features some bare minimum protection functions like undervoltage protection (32V) and overcurrent limit (40A / motor).



##### GameTrak

This firmware allows you to connect the sensor unit of a GameTrak PS2 controller to the ADC channels PA2 & 3. Use the potentiometers for x and z as voltage dividers (3V3 VREF) and connect them to PA2 & PA3. Also a great template for your own projects that require USART & ADC readout of some type of analog sensor (like a joystick). Same protection features as PPM. I2C LCD support to show data like battery gauge is WIP.
