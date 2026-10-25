# How we got here

The whole project, from first look to QMK with working indicators, ran on 2026-10-24. This is the order of work and the methods used, for anyone repeating it on a related board.

## 1. Desk research and tooling (no hardware)

- Gathered prior work: OpenDucky's Night Typist fork, the pok3r-custom tools and wikis, hansemro's HT32 tools and QMK branches, re-masterkeys, ducky-tools.
- Checked upstream: HT32 support was already in QMK's ChibiOS-Contrib, so no HAL work was needed.
- Ran host tools that were missing in Docker: the QMK CLI image, hansemro's OpenOCD fork, pok3rtool, ht32-dfu-tool.
- Mined OpenDucky for a candidate pin map. It later turned out to be mostly wrong for this board.

## 2. The vendor updater (static analysis only)

- The vendor download link was dead. The Internet Archive had a copy.
- The updater turned out to be the Cheng Yong `maaV102` format, not Ducky's. `pok3rtool extract maav102` produced a plaintext application.
- The load address came from the image itself. The reset vector and the density of valid function pointers pointed to 0x3200.
- Ghidra headless decompilation recovered the matrix pins and keymap, the LED driver and row pins, the SPI flash pins, the USB descriptors and the vendor command handler.

## 3. First contact with the live keyboard (read-only)

- Captured `lsusb -v` and the HID report descriptors.
- Sent only `0x12` read subcommands to the vendor interface, after checking in the disassembly that they have no side effects. They returned the IAP info block (04D9:1301, model `KB0301`) and the version page. The installed firmware was V1.00.05, the same as the updater.

## 4. Bench work

- PCB photos identified the MCU as an **HT32F52352**, not the HT32F1654 everyone had assumed, and the board as a Ducky One 2 PCB.
- The firmware analysis was re-checked against the HT32F52352 datasheet. The peripheral map is the same, so the findings stood, and the SPI and GCLK pins now resolved cleanly to valid alternate functions.
- Continuity checks: CN2 is SWD, SEL2 is PA9/BOOT to GND, SEL1 is PA8 to GND.

## 5. The lock

- SEL2 at plug-in brought up the ROM ISP. `ht32-dfu-tool info` showed flash security on.
- The lock meant no outside read of the IAP. The owner chose to dump it through a patched stock application instead of unlocking.

## 6. Proving the update path without writing

- In the IAP, the sum and CRC of the installed application matched the updater's image under pok3rtool's XOR encoding. That confirmed the encoding and the relative addressing with zero risk.

## 7. Dump

- Wrote a 10-byte-patched stock application through the IAP, then read all of flash twice through the new read command. The hashes matched. The stock IAP was now backed up.
- Disassembled the IAP: boot decision, watchdog counter, application check, application header, RC4 write decryption, checksum rules.

## 8. QMK

- Built the keyboard definition from the firmware-derived pin map.
- The IAP disassembly showed the first build lacked the 0x04040000 signature. Fixed before flashing.
- First flash: the verify failed (sum/CRC 0/0) because the checksum control word was missing. The IAP stayed resident as designed. Added the full 12-byte header.
- Second flash: verify OK, QMK enumerated, and it stayed up with no watchdog resets.
- Key tests: first by typing, then with a terminal tester that reads evdev directly and checks each key against `keyboard.json`. 108/108 keys passed.
- Bootmagic (Esc) into the IAP worked.

## 9. Indicator LEDs

- Traced the stock indicator routine to MBI5042 output 15 on rows 4–7, with active-high row switches multiplexed by a BFTM1 interrupt.
- Ported hansemro's MBI5042 driver. Num, Caps and Scroll work. The fourth LED (M) turned out to be the stock numpad mouse mode.

## 10. Ducky One 2 comparison (static)

- Downloaded Ducky's own One 2 full-size updaters and decoded them with the same pok3rtool extractor. It reproduced the Typist image's hash, which validated the method.
- Byte-searched the One 2 images for the Typist's pin tables, keymap and application header. All were found unchanged.
- Captured each updater's version template. This showed the version page is per product and can be built from the updater.

## Safety practices that paid off

- Every write to the MCU was approved by the owner, with the exact command shown first.
- Read-only checks came first whenever possible: info reads, then sum/CRC comparison, then the first write.
- Every dump was read twice and the hashes compared.
- Updates always wrote the version page last, after verifying. The one failed flash cost nothing.
- The vendor updater was never run.
