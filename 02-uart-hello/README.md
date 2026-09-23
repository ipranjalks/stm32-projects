# 02 – UART Hello

Two-way serial communication with a PC over the ST-LINK virtual COM port: prints messages with `printf`, echoes typed characters, and assembles them into lines.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. USART2 is wired to the onboard ST-LINK, which presents a virtual COM port to the PC.

## Serial settings

| Setting | Value |
|---|---|
| Baud rate | 115200 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |

## How it works

`printf` formats text into newlib's stdout buffer. When the buffer is flushed (on a newline), newlib calls `_write`, which is overridden here to send the bytes over USART2:

```c
int _write(int file, char *ptr, int len)
{
    HAL_UART_Transmit(&huart2, (uint8_t *)ptr, len, HAL_MAX_DELAY);
    return len;
}
```

The main loop receives one character at a time, echoes it back so typing is visible, and appends it to a buffer. On `\r` the buffer is null-terminated and printed.

## How to run

1. Build and flash.
2. Open a serial terminal on the board's COM port at 115200 baud.
3. Press RESET to see the startup banner.
4. Type anything and press Enter.

## Notes

- `HAL_UART_Receive` with `HAL_MAX_DELAY` blocks: the CPU can do nothing else while waiting for a keypress. Interrupt-driven receive solves this.
- `printf` is slow. At 115200 baud each character takes about 87 µs, so it should never be called from an interrupt handler.
