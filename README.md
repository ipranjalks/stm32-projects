# STM32 Projects

Projects built on the STM32 Nucleo-F446RE board (https://www.st.com/en/evaluation-tools/nucleo-f446re.html).

| # | Project | Concepts |
|---|---------|----------|
| 01 | [Blinky](01-blinky/) | GPIO output, HAL, flashing, debugging |
| 02 | [UART Hello](02-uart-hello/) | USART2, `printf` retargeting, blocking receive |
| 03 | [UART Command Shell](03-uart-command-shell/) | String parsing, command dispatch, GPIO output |
| 04 | [Button Interrupt](04-button-interrupt/) | EXTI, NVIC, weak callbacks, volatile, debouncing |

## Building any project

1. Open STM32CubeIDE and select this folder as the workspace.
2. Import the project if it isn't already listed.
3. Build with Ctrl+B, then Run to flash the board.
