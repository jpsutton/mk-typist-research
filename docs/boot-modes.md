# Boot modes

The board has three pieces of code that can run after reset, each with its own USB identity.

| Mode | Code | USB ID | Product string |
| --- | --- | --- | --- |
| ROM ISP | Holtek boot ROM | 04D9:8010 | (Holtek ISP) |
| Stock IAP | Cheng Yong bootloader at 0x0000 | 04D9:1301 | "USB-HID IAP" |
| Application | Stock firmware at 0x3200 | 04D9:0301 | "USB-HID Keyboard" |
| Application | QMK port at 0x3200 | 4D4B:5459 | "Mechanical Keyboards" / "MK Typist" |

## Boot flow

```text
power-on / reset
  │
  ├─ PA9 (BOOT) low at reset (SEL2 shorted) ──► ROM ISP, 04D9:8010
  │
  └─ PA9 high (default, internal pull-up) ──► main flash 0x0: stock IAP
        │
        ├─ SRAM magic 0x20003FFC == 0x55AAFAF5 ─────────────► stay in IAP, 04D9:1301
        ├─ application check fails (no signature, no
        │  version-page magic, bad vectors) ────────────────► stay in IAP, 04D9:1301
        └─ otherwise ──► jump to 0x3200 (application)
```

The IAP's exact boot decision is in [Stock IAP](stock-iap.md#boot-decision).

## Entering the stock IAP

| Method | Works from | Status |
| --- | --- | --- |
| Vendor HID command `0x11/0x00` on the vendor interface | Stock application | Tested |
| Hold **Esc** while plugging in (QMK bootmagic, matrix [12,0]) | QMK port | Tested |
| **Fn + Esc** (`QK_BOOT` on layer 1) | QMK port | Tested |
| Any code that writes 0x55AAFAF5 to 0x20003FFC and resets | Any application | Tested (this is what the stock app and QMK both do) |
| Five watchdog resets without a power cycle | An application that hangs without reloading the watchdog | From disassembly; not tested |
| Erased or invalid application | — | From disassembly. This is why a failed update leaves the IAP in control |

There is **no pin or key the IAP itself checks**. Key-combo entry exists only if the running application implements it. If an application hangs before USB comes up and keeps reloading the watchdog, only the ROM ISP remains (see [Flashing and recovery](flashing-and-recovery.md)).

## Leaving the stock IAP

- Vendor HID command `0x11/0x01` (reboot to firmware). Tested; the application comes back within about 1 s.
- A power cycle should also start the application, because SRAM loses the magic word. Inferred from the disassembly; not tested on this unit.

## ROM ISP

Short the SEL2 pads (PA9 to GND) while plugging in the USB cable, then release. The board enumerates as 04D9:8010 and [hansemro/ht32-dfu-tool](https://github.com/hansemro/ht32-dfu-tool) ([archive](https://web.archive.org/web/20261025013353/https://github.com/hansemro/ht32-dfu-tool)) talks to it.

With flash security on, the ROM ISP can report chip info and the option-byte page, and it can mass-erase. It cannot read flash. A mass erase removes the stock IAP as well as the application and clears the lock. See [Debug access](debug-access.md).

## Watchdog

The watchdog is running when the IAP jumps to the application (whether the IAP enables it or leaves a reset default in place was not pinned down). Any application must reload it (write 0x5FA00001 to WDT_CR at 0x40068000) or it is reset. The stock firmware reloads it from its main loop. The QMK port does it in `matrix_scan_kb`.
