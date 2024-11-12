# Introduction to General-Purpose Timers in STM32H7

The STM32H7 features several types of timers, including basic, general-purpose, and advanced timers. 

Basic timers in the STM32H7 series are fundamental hardware components designed to provide precise time base generation, periodic interrupts, and event triggering. Unlike advanced timers, basic timers lack features for PWM generation or input capture, focusing instead on timing functions. Understanding how to configure these timers is essential for applications requiring periodic tasks, such as LED blinking, sampling signals at fixed intervals, or maintaining accurate time references. Basic timers, such as TIM6 and TIM7, are dedicated to generating a stable time base. They operate independently and are simple to configure, which makes them ideal for applications needing time-keeping without any additional complexity.

The general-purpose timers in the STM32H7 series are versatile hardware modules designed for a range of timing functions, including precise time base generation, pulse-width modulation (PWM), input capture, and output compare functions. These timers are commonly used in applications that require tasks such as generating periodic interrupts, measuring input signals, or controlling motor speed. 
General-purpose timers, like TIM3, TIM4, and TIM5, are highly configurable and allow users to perform both basic and complex timing tasks. Unlike basic timers, general-purpose timers can operate in input capture and output compare modes, in addition to creating a time base. Understanding the time base functionality is essential for periodic task scheduling, event counting, and precise timing requirements.


## Time Base

Here, we'll focus on configuring a time base with a general-purpose timer, such as TIM3, by exploring key registers and settings. Key registers used for configuring the time base in general-purpose timers include:

- **Auto-reload register (ARR)**
- **Counter register (TIM_CNT)**
- **Prescaler register (PSC)**
- **Control register 1 (TIMx_CR1)**

Let’s explore each of these components and understand how they interact to provide a precise time base.

### Auto-Reload Register (ARR)

The **Auto-Reload Register (ARR)** sets the upper limit for the timer counter. When the counter (TIM_CNT) reaches the ARR value, an **update event (UEV)** is generated, and the counter resets to zero, beginning a new counting cycle. The ARR value effectively defines the timer’s period and, combined with the prescaler, determines the output frequency.
For example, if the timer’s frequency is set to 1 MHz and ARR is configured to 999, the counter will reach ARR every 1 ms (yielding a 1 kHz frequency).



### Counter Register (TIM_CNT)

The **Counter Register (TIM_CNT)** holds the current count value of the timer. This value is incremented or decremented at each timer tick, depending on the configured counting direction. When the TIM_CNT reaches the ARR value, an update event (UEV) occurs, resetting the counter to zero. 

The **Update Event (UEV)** occurs when the timer counter (TIM_CNT) reaches the ARR value. Each UEV can trigger an interrupt, which is useful for executing code at consistent intervals. UEVs are essential in periodic tasks, such as toggling an LED or sampling sensor data at fixed rates.
The generation of a UEV can be controlled through the TIMx_CR1 register, as we will explore in the next section.

By reading the TIM_CNT register, the software can monitor the timer’s current count, which can be useful for implementing delays, tracking elapsed time, or triggering actions based on specific intervals.

### Prescaler Register (PSC)

The **Prescaler Register (PSC)** divides the input clock frequency to control the timer’s tick frequency. Setting the PSC value effectively scales down the timer’s clock input, providing a slower counting rate. This division allows for lower-frequency time bases without modifying the system clock.

The timer clock frequency after applying the prescaler is given by the equation:


Timer Clock Frequency = Input Clock Frequency / (PSC + 1)

where:
- **Input Clock Frequency** is the frequency of the clock driving the timer (i.e., the APB1 or APB2 timer clock).
- **PSC** is the prescaler value loaded into the prescaler register.

For example, if the APB1 timer clock is 100 MHz and PSC is set to 99, the timer TIM3 counter clock is reduced to 1 MHz (100 MHz / (99 + 1)).

### Control Register 1 (TIMx_CR1)

The **Control Register 1 (TIMx_CR1)** configures the timer’s core operating modes and settings. This register includes bits for enabling the timer, selecting the counting direction, and defining update behavior.

Important configuration bits in TIMx_CR1 include:

- **CEN (Counter Enable):** Enables the timer counter when set. Clearing this bit halts the counter.
- **UDIS (Update Disable):** Controls whether updates generate a UEV. When set, the timer will not generate UEVs, allowing updates to the ARR and PSC values without restarting the counter.
- **URS (Update Request Source):** Restricts UEV generation to counter overflow events (TIM_CNT reaching ARR) when set to 1, filtering out other sources of UEVs.


