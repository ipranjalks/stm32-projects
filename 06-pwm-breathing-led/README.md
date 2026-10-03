# 06 – PWM Breathing LED

LD2 fades smoothly in and out instead of blinking. A hardware timer generates the PWM signal directly on the pin, so the CPU only intervenes to change the duty cycle.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. LD2 is on **PA5**, driven by TIM2 channel 1.

## Configuration

| Setting | Value |
|---|---|
| Timer / channel | TIM2, Channel 1 (PWM Generation CH1) |
| PA5 mode | Alternate function `TIM2_CH1` |
| Prescaler (PSC) | 84 - 1 |
| Counter period (ARR) | 1000 - 1 |
| PWM frequency | 1 kHz |
| NVIC | Not needed — no interrupt used |

## How it works

An LED is only ever fully on or fully off. Switching it faster than the eye can follow makes it *appear* dimmer in proportion to the fraction of time it spends on — the **duty cycle**.

TIM2 counts up to ARR and wraps, as in project 05. The difference is that channel 1's output is wired to PA5: on each tick the hardware compares CNT against CCR1, driving the pin high below the compare value and low above it. A compare of 0 is fully off, a compare equal to ARR is fully on.

The main loop ramps the compare value up and down to animate the fade.

```c
__HAL_TIM_SET_COMPARE(&htim2, TIM_CHANNEL_1, dutyCycle);
```

## Pin multiplexing

PA5 is a single pin, but several peripherals inside the chip could drive it. The mode setting selects which one reaches the output driver:

- **GPIO_Output** — driven by a bit in the GPIO output register, which only the code changes (projects 01–05).
- **Alternate function `TIM2_CH1`** — driven by the timer hardware, continuously, with no CPU involvement.

## How to run

1. Build and flash.
2. LD2 fades in and out continuously. No terminal needed.

## Notes

- A compare value **greater than ARR** means the counter never exceeds it, so the pin stays permanently high and the LED sits at full brightness. Easy off-by-one if the ramp's upper bound and ARR disagree.
- `__HAL_TIM_SET_COMPARE` is a macro, not a function: it compiles to a single register write (`htim2.Instance->CCR1 = value`) with no bounds checking. The `__HAL_` prefix marks this whole family of direct register accessors.
- PWM frequency must be well above ~100 Hz or the flicker becomes visible. Around 1 kHz is comfortable.

## Future ideas

- Drive the fade from `HAL_GetTick()` instead of `HAL_Delay`, so the UART shell stays responsive alongside it
- Set brightness from a shell command (`led 60`)
