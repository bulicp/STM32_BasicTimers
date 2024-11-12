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

The "+1" in the equation accounts for the fact that a PSC value of 0 corresponds to no division (a division factor of 1). This is because the prescaler’s counting starts at 0 and continues up to the value in PSC. After reaching PSC, the prescaler resets, which takes (PSC + 1) cycles in total.

By adjusting the prescaler, you can set a wide range of timer frequencies. The prescaler allows you to create slower time bases without adjusting the high-speed system clock and/or increase the duration of timer intervals, useful for creating delays or periodic events.
Setting an appropriate prescaler value is often the first step when configuring a timer for specific timing requirements. After setting the prescaler, you typically configure the Auto-Reload Register (ARR) to further control the timing period by setting the maximum count limit for the timer. 

The frequency of the Update Event (UEV), or the rate at which the timer counter resets and generates an update event, is determined by both the prescaler (PSC) and the auto-reload register (ARR):

UEV Frequency = Input Clock Frequency / [(PSC + 1) x (ARR + 1)]


### Control Register 1 (TIMx_CR1)

The **Control Register 1 (TIMx_CR1)** configures the timer’s core operating modes and settings. This register includes bits for enabling the timer, selecting the counting direction, and defining update behavior.

Important configuration bits in TIMx_CR1 include:

- **CEN (Counter Enable):** Enables the timer counter when set. Clearing this bit halts the counter.
- **UDIS (Update Disable):** Controls whether updates generate a UEV. When set, the timer will not generate UEVs, allowing updates to the ARR and PSC values without restarting the counter.
- **ARPE (Auto-Reload Preload Enable):** Enables the preload of ARR register. 


#### ARPE and ARPE Bit in CR1 (Control Register 1)

In STM32 timers, **ARPE** stands for **Auto-Reload Preload Enable**. It is a control feature that allows you to preload the auto-reload register (ARR) value into the timer, ensuring that the next value in the ARR is loaded into the timer immediately when the timer overflows (i.e., reaches its ARR value and resets the counter). This is important for ensuring smooth transitions between timer periods.

The **ARPE** bit is located in the **CR1 (Control Register 1)** of the timer. The setting of this bit controls whether the auto-reload register (ARR) value is updated immediately or only at the timer overflow event.

1. **ARPE = 0 (Disabled)**:
   - If the ARPE bit is cleared (i.e., ARPE = 0), the value in the **ARR** register is updated **only** when the timer overflows (i.e., the counter reaches the ARR value).
   - This means that the timer uses the same ARR value until the next timer overflow occurs.
   - This setting is typically used for cases where you do not need to update the ARR register during the timer's operation.

2. **ARPE = 1 (Enabled)**:
   - If the ARPE bit is set (i.e., ARPE = 1), the **ARR** register can be updated immediately without waiting for the overflow event.
   - The new ARR value is preloaded and used immediately in the next timer period, allowing for smooth transitions in time base updates.
   - This is useful when you want to change the period of the timer dynamically and have the change take effect at the next update event (i.e., when the counter overflows).

Let's say you're using a timer with a specific period set in the **ARR** register, but you want to dynamically change the period of the timer. If **ARPE** is enabled (ARPE = 1), any changes made to the ARR will immediately be used in the next update event. If **ARPE** is disabled (ARPE = 0), changes made to ARR will only take effect after the current update event.


## Configuring a Basic Time Base

Let's break down the configuration of TIM3 timer for generating a 1 kHz Update Event (UEV). We will also explain each step in detail based on the APB1 clock frequency of 64 MHz.

We use the following formula to calculate the UEV's frequency:

UEV Frequency = Input Clock Frequency / [(PSC + 1) x (ARR + 1)]

We need to configure PSC and ARR to achieve UEV frequency = 1 kHz.

To achieve a UEV frequency of 1 kHz (assuming APB1 tiimer frequency of 64 MH ), we first solve for (PSC + 1) × (ARR + 1):

(PSC+1)x(ARR+1) = 64MHz/1kHz = 64000000/1000 = 64000

We can choose ARR = 639 and PSC = 99 to satisfy this equation because:

(99+1)x(639+1) = 64000.


```c
volatile uint32_t* pTIM3_CR1 = (uint32_t*)(0x40000400);
volatile uint32_t* pTIM3_CNT = (uint32_t*)(0x40000424);
volatile uint32_t* pTIM3_PSC = (uint32_t*)(0x40000428);
volatile uint32_t* pTIM3_ARR = (uint32_t*)(0x4000042C);
__HAL_RCC_TIM3_CLK_ENABLE();

// Set ARR to 639 (this gives 640 total counts)
*pTIM3_ARR = 640-1;

// Set PSC to 99 (this divides the clock by 100, resulting in 1 kHz)
*pTIM3_PSC = 100-1;

// Set ARPE (Auto-Reload Preload Enable)
*pTIM3_CR1 = *pTIM3_CR1 | (1 << 7);

// Enable the timer counter
*pTIM3_CR1 = *pTIM3_CR1 | (1 << 0);
```


Now let's break down the code step-by-step:
```c
volatile uint32_t* pTIM3_CR1 = (uint32_t*)(0x40000400);
volatile uint32_t* pTIM3_CNT = (uint32_t*)(0x40000424);
volatile uint32_t* pTIM3_PSC = (uint32_t*)(0x40000428);
volatile uint32_t* pTIM3_ARR = (uint32_t*)(0x4000042C);
```
pTIM3_CR1, pTIM3_CNT, pTIM3_PSC, pTIM3_ARR are pointers to the relevant registers for TIM3. These registers control the timer configuration and operation.

```c
__HAL_RCC_TIM3_CLK_ENABLE();
```
This macro enables the clock for TIM3. It's necessary to initialize and use TIM3 because peripherals in STM32 are clocked by the system's RCC (Reset and Clock Control).

```c
// Set ARR to 639 (this gives 640 total counts)
*pTIM3_ARR = 640-1;
```
This line sets the ARR value to 639. The timer will count from 0 to 639, which gives a period of 640 clock cycles (since the counter starts at 0). This means the timer will reset and generate UEV every 640 counts.

```c
// Set PSC to 99 (this divides the clock by 100, resulting in 1 kHz)
*pTIM3_PSC = 100-1;
```
This line sets the PSC value to 99. The prescaler divides the input timer clock by 100. With an input frequency of 64 MHz, this results in a timer clock frequency of 640 kHz.

```c
// Set ARPE (Auto-Reload Preload Enable)
*pTIM3_CR1 = *pTIM3_CR1 | (1 << 7);
```
This line enables the Auto-Reload Preload Enable (ARPE) bit in CR1. When ARPE is set, the ARR value is updated at the next timer overflow rather than immediately. This is useful for configuring the timer without causing an immediate update that could disrupt timing.

```c
*pTIM3_CR1 = *pTIM3_CR1 | (1 << 0); // enable CNT
```
This line enables the timer by setting the CEN (Counter Enable) bit in the CR1 register. Once this bit is set, the timer starts counting.










