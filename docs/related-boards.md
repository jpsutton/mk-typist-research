# Related boards

## Same platform

| Board | MCU | Relation | What carried over |
| --- | --- | --- | --- |
| **Cooler Master MasterKeys Pro L White** (hansemro's QMK port) | HT32F52352 | Same Cheng Yong platform | Same matrix pin set (only the A3/A8 order differs), same MBI5042 pins (SDI PC12, DCLK PA14, LE PA15, SDO PB0, GCLK PC10, power PC11), same LED rows (PC15, PC14, PB5, PB4, PB3, PB2, PD3, PD2), application at 0x3200, same IAP magic 0x55AAFAF5 at 0x20003FFC, watchdog reload. This independently confirmed our firmware-derived pin map |
| **Ducky One 2, full size** (DKON1808 non-backlit, DKON1808S backlit) | Cortex-M0+ image, same as the Typist (MCU not seen) | The Typist PCB is a Ducky One 2 board (DK11-ONE02). Ducky's own updaters use the same Cheng Yong platform | Every pin table and the keymap are byte-identical. See [below](#ducky-one-2-full-size). We found no public QMK port for it |
| **MK Night Typist** (backlit) | HT32F1654 per TechPowerUp's teardown | Sister product | See OpenDucky below |
| **MK Frozen Llama** | Unknown | RGB sister product | Unknown |

## Ducky One 2 full size

Static comparison of Ducky's official One 2 updaters with the Typist's. Neither updater was run, and no Ducky hardware was tested. Sources and hashes are in [Sources](sources.md#vendor-files-not-redistributed).

Both updaters (non-backlit and single-colour backlit, V1.01.01) decode with `pok3rtool extract maav102`, and pok3rtool's re-encode check passes. The same tool reproduces the known hash of the Typist image, so the method is sound. Each updater holds a US and an EU (ISO) image. Ducky offers no firmware for the RGB model (DKON1808ST).

**Same as the Typist:**

- Updater program: the same Cheng Yong `FWUpdate` build, the `maaV102` package, "IAP Version V1.0.0" and the same XOR encoding. The encoding is the first 52 bytes of the IAP's RC4 keystream, so the One 2's IAP very likely uses the same key.
- Firmware family: USB model string `CYRM02p0296h00E0r0101` against the Typist's `CYRM02p0301h00DBr0005`.
- Image layout: linked at 0x3200, initial SP 0x20003FE0, reset vector 0x32C1, a 48-entry Cortex-M0+ vector table, and the same 12-byte application header at 0x3BF8 (`04040000 34000001 00020004`).
- Byte-identical in all four images:
  - keymap (128 bytes, including Fn, the Calc, Mute and volume codes, and the DIP-switch codes)
  - matrix input and strobe pin tables
  - MBI5042 serial pins, LED row table and LED row pins
  - GCLK pin descriptor (PC10, AF4)
  - SPI flash descriptor (CS PB7, SPI1)
- Constants: the IAP magic 0x55AAFAF5, and the watchdog reload (0x5FA00001 to 0x40068000).
- About half of the Typist image's 16-byte blocks appear verbatim in each One 2 image. That fits one code base built for two products.

**Different:**

| Item | Typist | Ducky One 2 |
| --- | --- | --- |
| Application USB ID | 04D9:0301 | 04D9:0296 |
| IAP USB ID | 04D9:1301 | 04D9:1296 |
| bcdDevice | 1.05 | 11.01 (US), 21.01 (EU) |
| Version page words at 0x3078 | 0x00100004, 0x00010005, 0xDB | 0x00800004, 0x00010101, 0xE0 |
| Layouts in the updater | US | US, EU (ISO) |
| LEDs | 4 indicators | No backlight (DKON1808; indicators not checked) or a full single-colour backlight (DKON1808S) |

**What this means for the QMK port.** On a non-backlit full-size One 2, the `mk/typist` firmware should boot under the stock IAP and scan every key correctly with no changes. Its indicator code drives MBI5042 output 15 on rows 4–7, which is harmless if nothing is fitted there. Needed for real support:

- A flasher that also accepts IAP 04D9:1296, and the One 2's version page (built from its updater template as described in [Stock IAP](stock-iap.md#version-page-0x30000x31ff)).
- An ISO `LAYOUT` for EU boards (the ISO positions are already in the matrix).
- QMK's LED matrix feature for the backlit model.

**Not known:**

- The One 2's IAP binary. It is not in the updater. Same header, magic, encoding and version string point to the same IAP, but it has not been read.
- Whether every One 2 production run uses this platform.
- The RGB model.

To confirm on a real One 2 without writing anything:

1. `lsusb` shows 04D9:0296.
2. In the IAP, `0x12/0x00` reports the model (expected `KB0296`), firmware offset 0x200 and page size 0x200.
3. The IAP's sum and CRC of the installed application match the image from the updater (see [Stock IAP](stock-iap.md#sum-and-crc)).

Before flashing QMK on any board other than the Typist, add a way to reach the IAP that does not depend on the matrix, such as a raw HID command. If the matrix were wrong, Esc bootmagic and Fn+Esc would not work, and only the ROM ISP mass erase, which deletes the stock IAP, would remain.

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
