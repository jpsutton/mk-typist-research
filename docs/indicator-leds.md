# Indicator LEDs

The 4 indicators (legends **N, C, S, M**, left to right) are driven by a Macroblock **MBI5042GP** (U7), a 16-channel constant-current sink with 16-bit PWM and a serial interface. The driver sinks current on its outputs; 8 row switches on the MCU side feed the anodes. This is the scheme the RGB boards of this platform use for a full LED matrix. The Typist only populates 4 LEDs.

## Wiring

| Signal | MCU pin |
| --- | --- |
| SDI (data to driver) | PC12 |
| DCLK | PA14 |
| LE (latch) | PA15 |
| SDO (data from driver) | PB0 |
| GCLK (grayscale clock) | PC10, AF4 = GPTM1 channel 0 |
| Driver power | PC11, driven **low** to power the driver |
| Row switches 0–7 | PC15, PC14, PB5, PB4, PB3, PB2, PD3, PD2, active **high** |

| LED | MBI5042 output | Row | Row pin | Host LED bit |
| --- | --- | --- | --- | --- |
| N (Num Lock) | OUT15 | 7 | PD2 | 0 |
| C (Caps Lock) | OUT15 | 6 | PD3 | 1 |
| S (Scroll Lock) | OUT15 | 5 | PB2 | 2 |
| M (mouse mode) | OUT15 | 4 | PB3 | — (internal flag) |

## How this was found

- The stock indicator routine (0x5A28) sets LED plane slots 0x7F (Num), 0x7E (Caps), 0x7D (Scroll) and 0x7C (M).
- The slot-to-hardware function (0x70D0) computes slot = channel × 8 + row, so all four are channel 15 on rows 7, 6, 5 and 4.
- The row table at 0xA4C8 lists (register, value) pairs. Each row is enabled by writing the pin to the port's set register and disabled through the reset register, so the rows are active high.
- The refresh interrupt is BFTM1 (vector 0x88). It steps through 8 rows, so each LED is lit 1/8 of the time.

The QMK port lights N, C and S correctly. Forcing the kernel's LED state lit the first three LEDs from the left.

## Driving it

The MBI5042 interface is bit-banged. Instructions are given as the number of DCLK pulses while LE is high:

| Instruction | DCLK pulses with LE high |
| --- | --- |
| Data latch | 1 |
| Global latch | 3 |
| Read configuration | 5 |
| Write configuration | 11 |
| Enable configuration write | 15 |

The QMK port's sequence:

1. Pull PC11 low to power the driver.
2. Start a ~4 MHz GCLK on PC10 (GPTM1 CH0 PWM, 50 % duty).
3. Probe the shift-register length: clock a 1 through and count pulses until it appears on SDO. Expect 16 for one chip. Wait up to 100 ms after power-on for the driver to answer.
4. Write the configuration word. The port uses **0x12C**, taken from hansemro's MasterKeys Pro L White port. The stock firmware's configuration word was not recovered (it lives in compressed initialised data).
5. Load grayscale: 0xFFFF on channel 15, 0 elsewhere. Global latch.
6. Start BFTM1 at 4 kHz. Each tick turns off the current row and turns on the next one if its LED should be lit. Each LED refreshes at 500 Hz with 1/8 duty, like the stock firmware.

Because only output 15 is used, the grayscale data never changes after start-up; only the row switches change.

The stock firmware shifts 48 bits per channel, as if three drivers were cascaded (R, G, B on the RGB boards). The Typist has a single driver.

## Prior work

- hansemro's MBI5042 driver and his Cooler Master MasterKeys Pro L White QMK port, which uses the same pins on the same platform. The QMK port's `mbi5042.c` is adapted from it.
- OpenDucky's MK Night Typist fork had an MBI5042 driver too, but for different pins (it assumed an HT32F1654). Its indicator slots, `led_matrix_data[7*16+15]` for Num and so on, agree with what we found: output 15 on rows 4–7.
- The `ducky/one2mini` QMK PR (#11112) has MBI5042 code for a Nuvoton-based board.
