# Related boards

## Same platform

| Board | MCU | Relation | What carried over |
| --- | --- | --- | --- |
| **Cooler Master MasterKeys Pro L White** (hansemro's QMK port) | HT32F52352 | Same Cheng Yong platform | Same matrix pin set (only the A3/A8 order differs), same MBI5042 pins (SDI PC12, DCLK PA14, LE PA15, SDO PB0, GCLK PC10, power PC11), same LED rows (PC15, PC14, PB5, PB4, PB3, PB2, PD3, PD2), application at 0x3200, same IAP magic 0x55AAFAF5 at 0x20003FFC, watchdog reload. This independently confirmed our firmware-derived pin map |
| **Ducky One 2** | (varies) | The Typist PCB is a Ducky One 2 board (DK11-ONE02) | We found no public QMK port for an HT32-based Ducky One 2 |
| **MK Night Typist** (backlit) | HT32F1654 per TechPowerUp's teardown | Sister product | See OpenDucky below |
| **MK Frozen Llama** | Unknown | RGB sister product | Unknown |

## Same MCU, different platform

| Board | Notes |
| --- | --- |
| **Anne Pro 2 (C18)** | HT32F52352. In mainline QMK as `annepro2/c18`. It proves the toolchain and supplies the 48 MHz clock setup, and its `wear_leveling` + `spi_flash` EEPROM is the likely model for persistent settings here |
| **Cooler Master MasterKeys Pro S** | HT32F52352 |

## HT32F1654 boards (different MCU, similar bootloaders)

| Board | Notes |
| --- | --- |
| **Vortex Core, POK3R and others** (pok3r-custom) | HT32F1654 (Cortex-M3). Application at 0x3400 with a version block at 0x3000. Similar IAP protocols; pok3rtool's CYKB support comes from this work. Their wiki documents the factory lock and that unlocking means a mass erase |

## OpenDucky's MK Night Typist fork

[OpenDucky/qmk_firmware](https://github.com/OpenDucky/qmk_firmware/tree/mk_typist) ([archive](https://web.archive.org/web/20261025013858/https://github.com/OpenDucky/qmk_firmware/tree/mk_typist)), branch `mk_typist`: six commits by "codetector", June 2020, on QMK of that time. Keyboard `ducky_one/mk_night_typist`.

| Item | OpenDucky (Night Typist) | Typist (this work) |
| --- | --- | --- |
| MCU | HT32F1654, Cortex-M3, 72 MHz | HT32F52352, Cortex-M0+, 48 MHz |
| Application base | 0x3000 | 0x3200 |
| Matrix | 15 × 8, custom `matrix.c` | 16 × 8, standard QMK matrix |
| Matrix pins | Mostly different | From the stock firmware |
| Key arrangement per row | Same as stock S0–S14 | Same |
| MBI5042 pins | DATA PC2, DCLK PA14, LE PA15, GCLK PC0, disable PC1 | SDI PC12, DCLK PA14, LE PA15, GCLK PC10, power PC11 |
| LED rows | PC8, PC7, PB5, PB4, PB3, PB2, PC6, PC5 | PC15, PC14, PB5, PB4, PB3, PB2, PD3, PD2 |
| Indicator slots | Output 15 on rows 4–7 | Same |
| USB ID | 04D9:0141 | 4D4B:5459 (QMK) |
| MBI5042 config word | 0x86B0 | 0x12C (from hansemro) |

Whether the Night Typist really differs this much or the fork was only partly adapted is unknown. Its DIP switch and SPI pins were commented-out values copied from the Vortex Core board and do not fit this matrix. The fork was the right starting point: it showed the HT32 HAL path, the MBI5042 driver idea, a bootloader offset and the indicator layout.

The current QMK `lib/chibios-contrib` contains everything the fork's ChibiOS-Contrib branch had for HT32 (GPIO, SPI, timers, USB, USART), plus a watchdog driver. Neither has an embedded-flash (EFL) driver, I2C or ADC for HT32.
