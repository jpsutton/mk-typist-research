# MK Typist research

Hardware and firmware notes for the **Mechanical Keyboards MK Typist**, gathered while porting [QMK](https://qmk.fm) ([archive](https://web.archive.org/web/20261010111415/https://qmk.fm/)) to it. The keyboard turned out to be a Ducky One 2 PCB with a Holtek HT32F52352 and a Cheng Yong IAP bootloader, so much of this also applies to related boards from the same platform.

Everything here was found on one unit (stock firmware V1.00.05) in October 2026. Each fact carries a confidence level and its evidence. Please open an issue if your board differs.

**QMK port:** branch [`mk-typist`](https://github.com/jpsutton/qmk_firmware/tree/mk-typist) of `jpsutton/qmk_firmware`, keyboard `mk/typist`. It runs on the stock bootloader, all 108 keys work, and the Num/Caps/Scroll indicators work. It is not in upstream QMK yet.

## Summary

| Item | Value | Confidence |
| --- | --- | --- |
| Board | Ducky One 2 PCB, silkscreen `DK11-ONE02-G458 71702053` | Confirmed (photo) |
| MCU | Holtek **HT32F52352** (Arm Cortex-M0+, 128 KB flash, 16 KB SRAM, LQFP64), marked "HT50F52352" | Confirmed (photo, ISP query) |
| Clock | 8 MHz crystal (X1) | Confirmed (USB works with an 8 MHz HSE PLL setup) |
| LED driver | Macroblock MBI5042GP (U7), driving 4 indicator LEDs | Confirmed (photo, working QMK driver) |
| SPI flash | GigaDevice 25Q40CT (U5), 4 Mbit | Part confirmed (photo); contents and wiring not checked |
| Key matrix | 16 strobes × 8 inputs, COL2ROW | Confirmed (108/108 key test under QMK) |
| Debug header | CN2: 3.3V, SWDIO, SWCLK, nRST, GND | Confirmed (continuity) |
| Boot pads | SEL2 = PA9/BOOT to GND (ROM ISP); SEL1 = PA8 to GND (purpose unknown) | Confirmed (continuity, ISP entry) |
| Flash protection | Flash security and option byte protection **on**; pages 0–11 write-protected | Confirmed (ISP query) |
| Flash layout | IAP 0x0000–0x2FFF, version page 0x3000–0x31FF, application from 0x3200 | Confirmed |
| USB IDs | Stock app 04D9:0301, stock IAP 04D9:1301, ROM ISP 04D9:8010 | Confirmed |
| Updater | `MK_Typist_V1.00.05NL.exe`, Cheng Yong `maaV102` format | Confirmed |
| Ducky One 2 full size | Same firmware platform. Ducky's updaters have byte-identical pin tables, keymap and application header, so this QMK port likely runs on it | Static analysis only; no One 2 tested |

## Contents

- [Hardware](docs/hardware.md): board, parts, MCU, photos
- [Pinout](docs/pinout.md): every MCU pin we know, with its source
- [Memory map](docs/memory-map.md): flash, SRAM, protection
- [Boot modes](docs/boot-modes.md): ROM ISP, stock IAP, application, and how to move between them
- [Stock IAP bootloader](docs/stock-iap.md): boot decision, application signature and header, version page, update protocol
- [USB](docs/usb.md): descriptors and IDs in each mode
- [Stock firmware](docs/stock-firmware.md): updater format and what the stock application does
- [Indicator LEDs](docs/indicator-leds.md): MBI5042 wiring and drive scheme
- [Debug access](docs/debug-access.md): SWD header, OpenOCD, ROM ISP, what the lock blocks
- [Porting QMK](docs/qmk-port.md): what the port needed and the traps we hit
- [Flashing and recovery](docs/flashing-and-recovery.md): update procedure and recovery options
- [Related boards](docs/related-boards.md): Ducky One 2 (firmware comparison), Night Typist, MasterKeys Pro L White, Anne Pro 2, Vortex
- [How we got here](docs/process.md): the order of work and the methods used
- [Open questions](docs/open-questions.md)
- [Sources and credits](docs/sources.md)
- [Captures](captures/): raw command output referenced by the docs
- [Photos](photos/): PCB bottom side
- [OpenOCD config](openocd/typist.cfg)

## Warnings

- **The chip ships locked.** SWD and the ROM ISP cannot read flash. Unlocking means a mass erase, which also deletes the stock IAP bootloader. The bootloader is not included in the vendor updater, so a mass erase cannot be undone from public files.
- The stock IAP is the safe update path. It refuses to start an application that lacks the expected signature, and it writes the version page last, so a failed update leaves the bootloader in control.
- Do not run the vendor updater on a board with modified firmware unless you know what it will do. We never ran it; all analysis of it was static.

## What is not here

Vendor binaries (the updater, the extracted firmware, flash dumps), decompiled vendor code and Holtek's PDFs are not redistributed. [Sources](docs/sources.md) links to where they can be obtained and gives the SHA-256 of each file we used.

## Credits

This work stands on earlier research by hansemro, the pok3r-custom project (ChaoticEnigma and contributors), re-masterkeys, OpenDucky, usedbytes, the QMK and ChibiOS-Contrib maintainers, and others. [Sources and credits](docs/sources.md) lists what each one contributed.

The research and the port were done by Jared Sutton with Claude (Anthropic) as an AI assistant. Claude did the static analysis, wrote the tools and drafted these docs. The owner did all hardware work and approved every write to the keyboard.

## License

Documentation and photos: [CC BY 4.0](LICENSE). The OpenOCD config is a few lines of configuration and is offered under the same terms.
