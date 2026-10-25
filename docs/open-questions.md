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
- **Version page fields** at 0x3078–0x3093 other than the VID/PID word. Are they the same on other units and layouts?
- **IAP versions.** Do other units have the same IAP? Compare against SHA-256 `fbb68c38129d066a05ddcd990bf566a937aef91243a7c6e89014ce3bf9429841` (0x0000–0x2FFF).
- **Mouse mode.** Which Fn combination toggles it, and what do codes 0xE8/0xE9, 0xF0/0xF1 and 0xFE do?
- **Option bytes** at 0x1FF00: not read.

## Port

- Persistent settings on the SPI flash (`wear_leveling` + `spi_flash`).
- A public flashing tool for the CYKB protocol, or support for 04D9:1301 in pok3rtool.
- Upstreaming to QMK.
