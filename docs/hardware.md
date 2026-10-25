# Hardware

## The keyboard

The MK Typist is a wired full-size ANSI keyboard sold by Mechanical Keyboards (mechanicalkeyboards.com): 104 keys plus Calculator, Mute, Volume Down and Volume Up (108 keys), 4 indicator LEDs (legends N, C, S, M), soldered plate-mounted switches, USB-C on a daughterboard, and a 4-way DIP switch on the underside. The vendor describes the DIP switch as selecting among 16 layouts.

Mechanical Keyboards stated that Ducky builds it. The PCB confirms this: it carries a Ducky logo and the silkscreen **`DK11-ONE02-G458 71702053 MADE IN TAIWAN`**, which is a Ducky One 2 board. The firmware and updater are in the Cheng Yong format used by several Ducky, Cooler Master and Vortex boards (see [Related boards](related-boards.md)).

## Parts on the PCB bottom

| Ref | Part | Notes |
| --- | --- | --- |
| U8 | Holtek HT32F52352, LQFP64, marked `HOLTEK HT50F52352 B819K003302 ARM` | Main MCU. "HT50F52352" is a known marking for the HT32F52352 on keyboard parts |
| U7 | Macroblock MBI5042GP | 16-channel constant-current LED driver; drives the 4 indicators on this board |
| U5 | GigaDevice 25Q40CT (GD25Q40C), SOP8 | 4 Mbit SPI NOR flash. What the stock firmware stores in it is unknown |
| X1 | Crystal | 8 MHz |
| U2 | Regulator | 3.3 V |
| CN2 | 5-pin 0.1" header | SWD debug header (see [Pinout](pinout.md)) |
| SEL1, SEL2 | 2-pin pads | SEL2 shorts PA9/BOOT to GND (ROM ISP entry); SEL1 shorts PA8 to GND |
| SW | 4-way DIP switch (ECE) | Left of U8. Wiring not verified |
| R295–R309 | "200" resistors around U7 | |
| Q74–Q81 | Transistors near the MCU | Probably the 8 LED row switches |

## Photos

All taken on 2026-10-24. Metadata stripped.

| Photo | Shows |
| --- | --- |
| [mcu-area-overview](../photos/2026-10-24_pcb-bottom_mcu-area-overview.jpg) | U8, U7, CN2, SEL1/SEL2, DIP switch, X1, U2, U5 |
| [mcu-u8](../photos/2026-10-24_pcb-bottom_mcu-u8.jpg) | MCU marking |
| [mbi5042-u7-cn2](../photos/2026-10-24_pcb-bottom_mbi5042-u7-cn2.jpg) | LED driver and debug header |
| [spi-flash-u5](../photos/2026-10-24_pcb-bottom_spi-flash-u5.jpg) | SPI flash |
| [ducky-logo-model](../photos/2026-10-24_pcb-bottom_ducky-logo-model.jpg) | Ducky logo and board number |

## MCU: Holtek HT32F52352

| Property | Value | Source |
| --- | --- | --- |
| Core | Arm Cortex-M0+ (ARMv6-M), SW-DP ID 0x0BC11477 | Datasheet |
| Max clock | 48 MHz | Datasheet |
| Main flash | 127.5 KB (130,560 B), 512 B pages, 255 pages | ISP `info` |
| SRAM | 16 KB at 0x20000000 | Datasheet |
| Package | LQFP64 | Photo, datasheet pin count |
| ISP | ROM ISP over USB, version v101 | ISP `info` |
| Flash security | On | ISP `info` |
| Option byte protection | On | ISP `info` |
| Page protection | 0xFFFFF000 0xFFFFFFFF 0xFFFFFFFF 0xFFFFFFFF: pages 0–11 (0x0000–0x17FF) protected | ISP `info` |

Evidence: [`captures/2026-10-24_isp_info.txt`](../captures/2026-10-24_isp_info.txt).

The same chip is used by the Anne Pro 2 (C18) and Cooler Master MasterKeys Pro S/L, which is why QMK's existing HT32F523xx support works here.

### Earlier assumptions that were wrong

Prior notes on the MK Night Typist and the Vortex Core suggested an **HT32F1654** (Cortex-M3, 64 KB). That is wrong for this unit. The chip marking, the 48-entry Cortex-M0+ vector table in the stock firmware, and the ROM ISP's model report all say HT32F52352. The two families share a peripheral map, which is why register-level analysis made under the HT32F1654 assumption still held.

### Peripheral addresses used by the stock firmware

| Peripheral | Base |
| --- | --- |
| GPIO A/B/C/D | 0x400B0000, +0x2000 per port |
| AFIO | 0x40022000 |
| SPI1 | 0x40044000 |
| GPTM1 | 0x4006F000 |
| BFTM1 | 0x40077000 |
| WDT | 0x40068000 (reload: write 0x5FA00001 to WDT_CR) |
| RSTCU GRSR (reset flags) | 0x40088100 |
| USB | 0x400A8000 |

## Datasheets

Holtek publishes the HT32F52342/52352 datasheet and user manual on its [product page](https://www.holtek.com/page/vg/HT32F52352) ([archive](https://web.archive.org/web/20250308053204/https://www.holtek.com/page/vg/HT32F52352)). We worked from datasheet v1.50 and user manual v1.30 (hashes in [Sources](sources.md)). The LQFP64 pin numbers in these docs come from datasheet Table 3. Flash protection behaviour comes from user manual Table 9.
