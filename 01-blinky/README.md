# 01 – Blinky

Toggles the green user LED (LD2) on the Nucleo-F446RE every 500 ms. The "Hello World" of embedded systems: proves the toolchain, flashing and debugging all work end to end.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. LD2 is connected to **PA5**.

## Key code

In `Core/Src/main.c`, inside the main loop:

```c
HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
HAL_Delay(500);
```

## How to run

1. Open the project in STM32CubeIDE 1.19.
2. Build (Ctrl+B), then Run.
3. LD2 blinks once per second (500 ms on, 500 ms off).

## Notes

- `HAL_Delay` is blocking: the CPU does nothing else while waiting.
