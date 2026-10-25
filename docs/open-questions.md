# Open questions

Contributions welcome, especially from owners of other units or related boards.

## Hardware

- **SPI flash (U5).** Wiring is from firmware only. Contents are unknown; the stock firmware keeps no settings in internal flash, so U5 probably holds profiles or macros. Dump it with a clip (keyboard unplugged, MCU held in reset), toggle a setting under stock firmware, dump again and diff.
- **DIP switches.** Probably matrix strobe S15 (PA8) on inputs In0–In3. Confirm by continuity or with a test keymap that reports row 15 directly. Also, what does each of the stock firmware's 16 layouts do?
- **SEL1 pads** (PA8 to GND). The stock IAP ignores PA8. Factory test? Something in the stock application?
- **MBI5042 configuration word** used by the stock firmware (the port uses hansemro's 0x12C).
- **Night Typist and Frozen Llama.** Same PCB with more LEDs populated, or different boards? The pin maps in OpenDucky's fork disagree with ours.

## Firmware and bootloader

- **Watchdog-counter recovery** (five watchdog resets leave the IAP resident) is from disassembly and untested.
- **Power-cycle from the IAP** should start the application; untested.
- **Version page fields.** Answered in part: the page is per product and comes from the updater's template (see [Stock IAP](stock-iap.md#version-page-0x30000x31ff)). The first word (0x00100004 on the Typist, 0x00800004 on the One 2) and the 0xEFFFFFFF, 1 and 0 words are still unexplained.
- **IAP versions.** Do other units have the same IAP? Compare against SHA-256 `fbb68c38129d066a05ddcd990bf566a937aef91243a7c6e89014ce3bf9429841` (0x0000–0x2FFF).
- **Ducky One 2 full size.** Its updaters match the Typist's pin tables, keymap and application header byte for byte (see [Related boards](related-boards.md#ducky-one-2-full-size)). Needs a real board: the read-only checks listed there, then a QMK test once the port has a matrix-independent way into the IAP. The RGB model's firmware is not public.
- **Mouse mode.** Which Fn combination toggles it, and what do codes 0xE8/0xE9, 0xF0/0xF1 and 0xFE do?
- **Option bytes** at 0x1FF00: not read.

## Port

- Persistent settings on the SPI flash (`wear_leveling` + `spi_flash`).
- Support for 04D9:1301 (and 04D9:1296 for the One 2) in pok3rtool, so the port does not depend on our own flasher ([mk-typist-tools](https://github.com/jpsutton/mk-typist-tools)).
- A host-triggered IAP entry (for example a raw HID command), so a board with a wrong matrix can still reach the bootloader.
- Upstreaming to QMK.
