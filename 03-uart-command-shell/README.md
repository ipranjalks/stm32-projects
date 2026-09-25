# 03 – UART Command Shell

A minimal command-line interface over the ST-LINK virtual COM port. Type a command, press Enter, and the board acts on it.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. Commands control the onboard LED, LD2 (PA5).

## Serial settings

| Setting | Value |
|---|---|
| Baud rate | 115200 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |

## Commands

| Command | Effect |
|---|---|
| `help` | Lists available commands |
| `led on` | Turns LD2 on |
| `led off` | Turns LD2 off |
| `led toggle` | Inverts the current LD2 state |

Anything else prints an error naming what was typed.

## How it works

Characters arrive one at a time via `HAL_UART_Receive` and are echoed back so typing is visible. Each is appended to a 64-byte buffer, guarded by `idx < sizeof(line) - 1` so the last slot stays free for the terminator.

On `\r` the buffer is null-terminated and compared against each known command with `strcmp`. After the command runs, the prompt is reprinted and `idx` is reset to 0 to start a new line.

The prompt uses `printf("> ")` with no newline, so `fflush(stdout)` is required — stdout is line-buffered, and without a flush the prompt would sit in newlib's buffer until the next newline was printed.

## How to run

1. Build and flash.
2. Open a serial terminal on the board's COM port at 115200 baud.
3. Press RESET to see the prompt.
4. Type `help` and press Enter.

## Notes

- Commands are case-sensitive: `LED ON` is not recognised.
- The shell blocks in `HAL_UART_Receive` while waiting for input, so it cannot do anything else meanwhile.

## Future ideas

Deliberately left out to keep the project small:

- Backspace handling (`\b` and DEL, 127)
- Arguments via `strtok`, e.g. `blink 200`
- A command table of name/handler/help entries instead of an `if` chain, so `help` can generate itself
- `temp`, `vdd`, `uptime`, `reset`
- Command history with the up arrow
