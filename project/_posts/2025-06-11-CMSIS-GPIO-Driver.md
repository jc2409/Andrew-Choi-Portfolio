---
layout:     post
title:      "Creating a GPIO and Timer Driver with CMSIS"
sitemap: false
excerpt_separator: <!--more-->
hide_last_modified: true
comments: true
---

# Creating a GPIO and Timer Driver with CMSIS

## Introduction

This project involved building a basic driver in C to control an LED using GPIO and a hardware timer on an STM32 microcontroller. By leveraging CMSIS (Cortex Microcontroller Software Interface Standard), I interacted directly with peripheral registers, gaining hands-on experience in low-level embedded development.

<!--more-->

## Project Overview

The primary goal was to:

- Toggle an LED using a GPIO pin.
- Create a delay mechanism using a hardware timer.
- Use CMSIS to access microcontroller registers instead of using HAL libraries.

## Features

- **GPIO Driver:** Initializes PA5 as an output pin to control the onboard LED.
- **Timer Driver:** Configures TIM15 to generate a precise 1 ms delay.
- **Main Loop:** Toggles the LED with a 100 ms interval using the timer-based delay function.

## Implementation Details

### GPIO Configuration

The `GPIO_Init()` function enables the clock to GPIOA and configures PA5 as a general-purpose output. Toggling is handled via the `LED_Toggle()` function, which manipulates the Output Data Register (ODR).

```c
GPIOA->MODER |= (1U<<10);
GPIOA->MODER &= ~(1U<<11);
GPIOA->ODR ^= (1U<<5);
```

### Timer Setup

The `TIM15_Init()` function configures the microcontroller's system clock to 16 MHz and sets TIM15 with a prescaler of 15 and auto-reload of 999. This setup provides a 1 kHz timer frequency—ideal for millisecond precision delays.

### Delay Function
The `delay_ms()` function waits for the update interrupt flag (UIF) to be set, providing accurate delays in a blocking loop.
```c
while((TIM15->SR & TIM15_UIF) == 0);
TIM15->SR &= ~(TIM15_UIF);
```

## Learning Process

This project was an excellent opportunity to dive into the STM32 datasheet, reference manual, and CMSIS documentation. Understanding how registers work and how different peripheral clocks interact gave me a solid foundation in embedded systems programming.

## Challenges
- Setting up the system clock correctly and confirming the correct prescaler values.
- Ensuring accurate delay timing by correctly handling the timer's update event flag.
- Interpreting the bitfields from the documentation precisely.

## Next Steps

I plan to build more drivers including:
- SPI and I2C communication drivers.
- UART interface for serial communication.
- Eventually, a Wi-Fi driver for basic networking.

## Conclusion

This project reinforced the importance of understanding hardware at the register level. Using CMSIS rather than a high-level HAL taught me how embedded systems truly function and will make future driver development more intuitive.

## GitHub Repository

You can find the full source code and setup instructions on [GitHub](https://github.com/jc2409/CMSIS-STM32L4).
