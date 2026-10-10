# 07 – ADC Temperature

Reads the STM32F446's internal temperature sensor over ADC1 and prints the raw count, the converted voltage and the temperature in °C once a second. No external components — the sensor is on the die.

## Hardware

- Nucleo-F446RE
- USB Mini-B cable

No external components. The temperature sensor is internal to the MCU and wired to ADC1 channel 18.

## Configuration

| Setting | Value |
|---|---|
| ADC | ADC1 |
| Channel | Temperature Sensor Channel (IN18) |
| Resolution | 12-bit (0–4095) |
| Clock prescaler | PCLK2 / 4 |
| PCLK2 | 84 MHz |
| ADCCLK | 21 MHz |
| Sampling time | 480 cycles = 22.9 µs |
| Conversion mode | Single conversion, polled |

### Sampling time arithmetic

The datasheet specifies a **minimum 10 µs** sampling time for the temperature sensor — far longer than an ordinary pin, because the internal source has high impedance and the ADC's sample-and-hold capacitor needs time to charge.

```
sampling time (µs) = cycles / ADCCLK (MHz)

144 cycles / 21 MHz =  6.9 µs   ✗ below minimum
480 cycles / 21 MHz = 22.9 µs   ✓
```

Too short a sampling time does not fail loudly — it produces readings that look plausible but sit low.

## The RM0390 procedure, mapped to HAL

Reference manual page 376 gives the register-level steps. They map onto CubeMX and the HAL as follows:

| RM0390 step | Here |
|---|---|
| Select ADC1_IN18 | Temperature Sensor Channel ticked in the .ioc |
| Sampling time ≥ minimum | Sampling Time set to 480 cycles on the rank |
| Set TSVREFE in ADC_CCR | Set by the same tick box, written in `MX_ADC1_Init` |
| Set SWSTART | `HAL_ADC_Start(&hadc1)` |
| Read the data register | `HAL_ADC_PollForConversion` then `HAL_ADC_GetValue` |
| Apply the formula | Own arithmetic, below |

## The conversion

Two steps, in order:

```c
v_sense = (adc_value * 3.3f) / 4095.0f;          // raw count → volts
temp    = (v_sense - 0.76f) / 0.0025f + 25.0f;   // volts → °C
```

`V25 = 0.76 V` (sensor output at 25 °C) and `Avg_Slope = 2.5 mV/°C` come from the datasheet's electrical characteristics, page 145.

**The units trap:** VSENSE in the reference manual's formula is a **voltage**, not the raw count. Feeding the 12-bit value straight in gives a temperature in the thousands.

Full scale is **4095**, not 4096 — 12 bits counts 0 to 4095.

## How to run

1. Build and flash.
2. Open a serial terminal on the board's COM port at 115200 baud.
3. Readings print once a second.
4. Hold a finger on the MCU (the large chip below the headers, not the ST-LINK processor above the break line) for 10–15 seconds and watch the value climb, then drift back.

## Notes

- **`HAL_ADC_Start` and `HAL_ADC_Stop` must be paired** in single conversion mode. `Stop` disables the ADC, so leaving it in the loop without a matching `Start` gives one good reading and then timeouts. Moving `Start` outside the loop only works with continuous conversion mode enabled.
- **This measures die temperature, not room temperature.** The silicon runs above ambient by however much the chip is dissipating, and that offset changes with workload. Combined with ±1.5 °C sensor precision, several degrees of absolute error is normal. The sensor is designed for detecting thermal *change*, which it does well.
- **Printing floats** requires enabling it at Project → Properties → C/C++ Build → Settings → MCU Settings → "Use float with printf from newlib-nano".
- Channel 18 is **shared with VBAT**. The VBATE bit in ADC_CCR takes precedence if set, which would give a stable but wrong reading.
- The ADC has a **maximum input frequency** (~36 MHz here). The prescaler exists to stay under it; exceeding it corrupts conversions subtly rather than obviously.

## Future ideas

- Average 16 samples to smooth the reading
- Log overnight to see the daily ambient curve, despite the fixed offset
- Add a `temp` command to the UART shell from project 03
