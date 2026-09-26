# 04 – Button Interrupt

Pressing the blue user button (B1) toggles LD2 and increments a press counter, printed over UART. The main loop never checks the button — the hardware interrupts the CPU when the pin changes.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. B1 is on **PC13**, LD2 on **PA5**.

## Configuration

| Setting | Value |
|---|---|
| PC13 mode | `GPIO_EXTI13`, falling edge trigger |
| Pull-up/pull-down | None (external pull-up fitted on the board) |
| NVIC | EXTI line[15:10] interrupt enabled |
| Debounce window | 50 ms |

## How it works

On a falling edge at PC13, the hardware vectors to `EXTI15_10_IRQHandler` in `stm32f4xx_it.c`. That calls `HAL_GPIO_EXTI_IRQHandler`, which clears the pending flag and then calls `HAL_GPIO_EXTI_Callback` — the weak function overridden in `main.c`.

The callback checks the pin, applies the debounce window, toggles LD2 and increments `counter`. The main loop compares `counter` against the last value it printed and prints only when it changes.

## Why the button is active-low

B1 connects PC13 to ground when pressed. An external pull-up resistor holds the pin at 3.3 V when released, so the pin reads **high when released** and **low when pressed**. The falling edge is therefore the press.

## How to run

1. Build and flash.
2. Open a serial terminal on the board's COM port at 115200 baud.
3. Press RESET, then press B1. LD2 toggles and the count prints.

## Notes

- **The onboard button did not bounce observably.** The Nucleo has an RC filter on the user button line, which smooths contact chatter in hardware before it reaches the pin. Each press incremented the count by exactly 1 even before debouncing was added. A bare switch on a breadboard, with no filtering, would be expected to register several counts per press.

## Future ideas

- An external button on a breadboard, to observe real bouncing
