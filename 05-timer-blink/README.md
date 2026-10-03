# 05 – Timer Blink

LD2 blinks at a fixed rate driven entirely by a hardware timer interrupt. Visually identical to project 01, but with no `HAL_Delay` anywhere — the CPU is free the whole time, which a responsive UART shell in the main loop demonstrates.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. LD2 is on **PA5**.

## Timer configuration

| Setting | Value |
|---|---|
| Timer | TIM2 |
| Clock source | Internal clock |
| APB1 timer clock | 84 MHz |
| Prescaler (PSC) | 8400 - 1 |
| Counter period (ARR) | 10000 - 1 |
| Update frequency | 1 Hz |
| NVIC | TIM2 global interrupt enabled |

## The arithmetic

```
update frequency = timer_clock / ((PSC + 1) × (ARR + 1))
```

Both registers take `+1` because a value of 0 means "divide by 1" — there is no division by zero, and counting 0 to N is N+1 ticks.

Choosing PSC to give a round tick rate (e.g. 10 kHz) makes ARR directly readable: at 10 kHz, ARR is simply tenths of a millisecond.

PSC is 16-bit on all timers, so a prescaler is unavoidable at megahertz clock speeds — a 16-bit counter at 16 MHz overflows in about 4 ms, nowhere near a 1 Hz blink.

## How it works

TIM2 counts prescaled ticks. On reaching ARR it resets to 0 and raises an update event, vectoring to `TIM2_IRQHandler` in `stm32f4xx_it.c`, which calls `HAL_TIM_IRQHandler` and then the weak `HAL_TIM_PeriodElapsedCallback` overridden in `main.c`.

That callback is shared by **every** timer, so it checks `htim->Instance == TIM2` before toggling LD2.

## How to run

1. Build and flash.
2. Open a serial terminal on the board's COM port at 115200 baud.
3. LD2 blinks on its own. Type in the terminal — characters echo with no
   delay while the blink stays steady.

## Notes

- CubeMX accepts expressions in the PSC and ARR fields, so `1600-1` can be written instead of `1599`, which documents the intent.
- Compared with project 01: `HAL_Delay(500)` blocked the CPU for half a second at a time. Here the timer counts in hardware and the main loop stays fully responsive.

## Future ideas

- Change the blink rate at runtime from a shell command (`rate 200`)
- Drive two LEDs from two timers at different rates
