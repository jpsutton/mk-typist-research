# Memory map

## Flash (main array, 0x00000–0x1FDFF)

| Range | Size | Contents | Source |
| --- | --- | --- | --- |
| 0x00000–0x0223B | 8,764 B | Stock IAP code and data (4-entry vector table at 0x0) | Dump of this unit |
| 0x00400 | 0x34 B | IAP information block (served over USB by command `0x12/0x00`) | Live read, dump |
| 0x0223C–0x02FFF | | Erased (0xFF) | Dump |
| 0x03000–0x031FF | 512 B (1 page) | **Version page**: version string, ID words, magic. Written by the updater after the application | Live read |
| 0x03200–0x1FDFF | 113,152 B | **Application**. Vector table at 0x3200; 12-byte application header at 0x3BF8 | IAP disassembly, stock image |
| 0x0AE00–0x1FDFF | | Erased on this unit with stock V1.00.05 installed. The stock firmware keeps no settings in internal flash | Dump |
| 0x1FF00 | | Option bytes (not read) | Datasheet |

Page size is 512 B. The IAP describes its writable area as 231 pages of 512 B starting at 0x3000 (0x3000–0x1FDFF), with the application at offset 0x200.

The stock V1.00.05 application is 31,728 bytes (0x3200–0xACEF). The updater erases 0x3000–0xADFF.

### Protection

- Flash security and option byte protection are on (ISP query).
- Page protection covers **pages 0–11 only (0x0000–0x17FF)**. The IAP code runs to 0x223B, so its last 2.6 KB (0x1800–0x223B) sit in unprotected pages. The IAP's own erase and write commands cannot reach them (all offsets are relative to 0x3000), but application code with flash-write routines could. Keep that in mind before adding EEPROM-in-flash code to an application.
- With security on, debug (SWD) and ROM ISP reads return nothing useful. Code running from main flash can still read, program and page-erase it (user manual Table 9). That is how this unit's flash was dumped (see [Flashing and recovery](flashing-and-recovery.md)).

## SRAM (0x20000000–0x20003FFF, 16 KB)

| Address | Use | Source |
| --- | --- | --- |
| 0x20003FE0 | Initial SP of the IAP | IAP vector table |
| **0x20003FFC** | **IAP magic word**. 0x55AAFAF5 keeps the IAP resident after reset; 0x55AAFAF0–0x55AAFAF4 is the watchdog-reset counter | IAP disassembly; stock app reset handler |

Applications must keep the last word of SRAM out of their image so it survives a reset. The QMK port's linker script uses `ram0 len = 16k - 4`.

## Application layout (stock and QMK)

| Address | Contents |
| --- | --- |
| 0x3200 | Vector table (48 entries, 0xC0 bytes, Cortex-M0+). Initial SP must be ≤ 0x20004000 |
| 0x3204 | Reset vector, must be < 0x1FE00 |
| 0x3BF8 | `0x04040000` application signature (required to boot) |
| 0x3BFC | `0x34000001` checksum control (required for the IAP's verify command) |
| 0x3C00 | `0x00020004` (returned by info command `0x12/0x01`) |

Stock V1.00.05: reset vector 0x32C1. The header sits at stock app offset 0x9F8, in the middle of the code, so any replacement application must reserve those 12 bytes. Details in [Stock IAP](stock-iap.md).

## Comparison with other memory maps

| Source | Application base | Notes |
| --- | --- | --- |
| **MK Typist (this unit)** | **0x3200** | Version page 0x3000 |
| OpenDucky (MK Night Typist QMK fork, 2020) | 0x3000 | Probably flashed after erasing; not usable with the stock IAP here |
| Vortex Core (HT32F1654) | 0x3400 | Version block at 0x3000 |
| Cooler Master MasterKeys Pro L White (hansemro) | 0x3200 | Same platform as the Typist |
| hansemro `ht32_usb_dfu` | 0x3400 | Replacement DFU bootloader |

How 0x3200 was found before the IAP was dumped: the stock image's reset vector (0x32C1) lands right after a 0xC0-byte vector table based at 0x3200, and 37 of 102 Thumb pointers in the image hit function prologues at base 0x3200, against 1 at 0x3000 or 0x3400. The live keyboard then served its version page from 0x3000–0x31FF.
