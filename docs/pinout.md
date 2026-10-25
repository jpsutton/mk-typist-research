# Pinout

Pin numbers are HT32F52352 **LQFP64** (datasheet Table 3). "Source" says how each assignment is known:

- **Continuity**: traced on the PCB with a multimeter.
- **Firmware**: read from the stock firmware's pin tables and init code (static analysis).
- **Working**: QMK uses the pin this way on the real board and the function works.
- **Datasheet**: fixed function of the chip.

Only CN2, SEL1 and SEL2 were traced with a meter. The matrix and LED pins come from the stock firmware and are confirmed by working QMK firmware (every key and three of the four indicators work). The SPI flash and DIP switch wiring are firmware-derived only.

## Debug header CN2 and boot pads

| Pad | Signal | MCU pin | Source |
| --- | --- | --- | --- |
| CN2-1 (square) | 3.3V (VDD 9, 18, 42, 55; VDDA 63; VBAT 21) | — | Continuity |
| CN2-2 | SWDIO / PA13 | 39 | Continuity |
| CN2-3 | SWCLK / PA12 | 38 | Continuity |
| CN2-4 | nRST | 20 | Continuity |
| CN2-5 | GND (VSS 10, 19, 43, 56; VSSA 64) | — | Continuity |
| SEL1-1 / SEL1-2 | PA8 / GND | 34 | Continuity |
| SEL2-1 / SEL2-2 | PA9_BOOT / GND | 35 | Continuity; shorting at power-up starts the ROM ISP (tested) |

CN2 has the same order as the Vortex Core's CN2.

PA8 is matrix strobe S15, which the firmware drives as an output. **Do not short SEL1 while the keyboard runs.** What SEL1 is for is unknown; the stock IAP does not read it (see [Stock IAP](stock-iap.md)).

## Key matrix

16 strobes (outputs, QMK rows) × 8 inputs (QMK columns). The stock firmware drives one strobe low at a time and reads the inputs active-low with pull-ups, debouncing over 3 scans. In QMK terms this is `COL2ROW`.

| Strobe | Pin | In0 PC4 (11) | In1 PA5 (6) | In2 PA6 (7) | In3 PA7 (8) | In4 PC8 (13) | In5 PB10 (22) | In6 PB11 (23) | In7 PB12 (24) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S0 | PD1 (46) | \` | 1 | Tab | Q | Caps | A | LShift | Z |
| S1 | PC1 (57) | 2 | 3 | W | E | S | D | X | C |
| S2 | PD0 (25) | 4 | 5 | R | T | F | G | V | B |
| S3 | PB8 (62) | 6 | 7 | Y | U | H | J | N | M |
| S4 | PA11 (37) | 8 | 9 | I | O | K | L | , | . |
| S5 | PA0 (1) | 0 | - | P | [ | ; | ' | / | RShift |
| S6 | PA1 (2) | = | Backspace | ] | \\ | Enter | — | RCtrl | — |
| S7 | PA2 (3) | Insert | Home | Delete | End | **Fn** | Up | Down | Left |
| S8 | PA3 (4) | PgUp | — | PgDn | F10 | F11 | F12 | Right | KP0 |
| S9 | PA4 (5) | NumLock | KP/ | KP7 | KP8 | KP4 | KP5 | KP1 | KP2 |
| S10 | PC5 (12) | KP\* | KP- | KP9 | KP+ | KP6 | KPEnter | KP3 | KP. |
| S11 | PC9 (14) | — | — | LGUI | LCtrl | LAlt | Space | RAlt | RGUI |
| S12 | PC13 (33) | Esc | F1 | F2 | F3 | (ISO \\) | — | PrtSc | ScrLk |
| S13 | PA10 (36) | F4 | F5 | F6 | Pause | Vol+ | Vol- | Calc | Mute |
| S14 | PB1 (45) | F7 | F8 | F9 | (JIS) | (JIS) | (JIS) | (JIS) | (JIS) |
| S15 | PA8 (34) | DIP? | DIP? | DIP? | DIP? | ? | ? | ? | ? |

Source: firmware (pin tables at 0xA156 inputs and 0xA166 strobes, keymap at 0xA0D6, scan loop at 0x8D04). Confirmed working: all 108 keys, each at a distinct matrix position, in a key test under QMK ([`captures/keytest-2026-10-24.txt`](../captures/keytest-2026-10-24.txt)).

Notes:

- **Fn** (S7 In4) sits where a Menu key would be. The stock firmware sends vendor code 0xD8 there and treats it as Fn.
- The media keys were checked by pressing them in physical order left to right: Calc [13,6], Mute [13,7], Vol− [13,5], Vol+ [13,4]. Stock codes at S13 In4–In7 are 0xF6, 0xF5, 0xEE, 0xF4.
- Positions marked (ISO) and (JIS) are in the stock keymap but have no switch on the ANSI board.
- **S15** holds vendor codes 0xC0–0xC3 and 0xC9–0xCC in the stock keymap. These are probably the 4 DIP switches on In0–In3. Not verified: toggling the switches under QMK produced nothing, but QMK's `dip_switch` feature consumes those positions without sending keys, so the test was inconclusive. The owner does not use the switches, so this was not pursued.
- **PB10, PB11, PB12** are backup-domain pads. Their reset function (AF0) is X32KIN, X32KOUT and RTCOUT, not GPIO. Firmware must select AF1 (GPIO) before using them as matrix inputs. The QMK board file sets AF1 on every matrix pin. PC13 is in the same situation.

## Indicator LEDs (MBI5042, U7)

| Signal | Pin | Source |
| --- | --- | --- |
| MBI5042 SDI (MCU data out) | PC12 (32) | Firmware; working |
| MBI5042 DCLK | PA14 (40) | Firmware; working |
| MBI5042 LE | PA15 (41) | Firmware; working |
| MBI5042 SDO (read back) | PB0 (44) | Firmware; working (QMK reads the shift-register length) |
| MBI5042 GCLK | PC10 (30), AF4 = GPTM1 CH0 | Firmware; working |
| LED driver power | PC11 (31), active low | Firmware; working |
| LED row switches 0–7 | PC15 (54), PC14 (53), PB5 (52), PB4 (51), PB3 (50), PB2 (49), PD3 (48), PD2 (47) | Firmware (row table 0xA4C8, pin table 0xA508); rows 5–7 working |

The 4 indicators are all on MBI5042 output 15, one per row: **Num = row 7 (PD2), Caps = row 6 (PD3), Scroll = row 5 (PB2), M = row 4 (PB3)**. Num, Caps and Scroll are confirmed lit under QMK. See [Indicator LEDs](indicator-leds.md).

## SPI flash (U5, GD25Q40C)

| Signal | MCU pin | Flash pin | Source |
| --- | --- | --- | --- |
| CS# (GPIO) | PB7 (61) | 1 | Firmware (descriptor at 0xA7C4) |
| SCK (SPI1_SCK, AF5) | PC2 (58) | 6 | Firmware |
| MOSI (SPI1_MOSI, AF5) | PC3 (59) | 5 (SI) | Firmware |
| MISO (SPI1_MISO, AF5) | PB6 (60) | 2 (SO) | Firmware |

Not traced and not exercised by QMK yet. PC1 (pin 57) would be the hardware SPI1_SEL, but it is matrix strobe S1, which explains why chip select is a GPIO. WP# (3) and HOLD# (7) are probably pulled high through R98–R103; not checked.

## USB, clock and other fixed pins

| Signal | Pin | Source |
| --- | --- | --- |
| USB D− / D+ | 15 / 16 | Datasheet |
| HSE crystal (8 MHz) | PB13 (26) / PB14 (27) | Datasheet; QMK's 8 MHz PLL setup gives working USB |
| BOOT | PA9 (35), internal pull-up | Datasheet. The only boot pin on this part. Unused by the stock firmware |

## Free or unknown pins

Every pin not listed above is unknown. The stock firmware's init code did not show any other used pins, but this was not checked exhaustively.
