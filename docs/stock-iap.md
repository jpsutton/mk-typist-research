# Stock IAP bootloader

The stock bootloader (Holtek calls this "IAP", in-application programming) occupies 0x0000–0x2FFF and identifies as 04D9:1301 "USB-HID IAP". It is the Cheng Yong "CYKB" bootloader also found on Cooler Master and Vortex boards. The model string in its info block is `KB0301`.

**How this was learned.** The chip is read-protected, so the IAP could not be read over SWD or ISP. We dumped it with a patched stock application loaded through the IAP itself (method in [Flashing and recovery](flashing-and-recovery.md#how-the-flash-was-dumped)), then disassembled it. Two reads of the full 0x00000–0x1FDFF range gave the same SHA-256, `35f6e0166c282715dfe107b4c440e718093ec5ff5e32a271babccc3906c15226`. The SHA-256 of 0x0000–0x2FFF alone (this unit's IAP) is `fbb68c38129d066a05ddcd990bf566a937aef91243a7c6e89014ce3bf9429841`. If your board's IAP hashes the same, these notes apply to it directly. The dump itself is not published.

Addresses below are flash addresses in the IAP unless stated.

## Layout

| Address | Contents |
| --- | --- |
| 0x0000 | Vector table, 4 entries only: SP 0x20003FE0, Reset 0x000000B9, NMI and HardFault 0x000000C1 (`b .`) |
| 0x0010 | Scatter-load routine |
| 0x00B8 | Reset handler: calls SystemInit (0x01A4), scatter-load (0x0010), then `main` (0x1DA8) |
| 0x0400 | Information block (below) |
| 0x0434 | 8-byte RC4 key used to decrypt firmware writes |
| 0x1D40 | Application validity check |
| 0x1DA8 | `main` |
| 0x223B | End of code |

## Information block (0x400)

Returned by command `0x12/0x00` in both the IAP and the stock application. Offsets relative to 0x400:

| Offset | Type | Value on this unit | Meaning |
| --- | --- | --- | --- |
| 0x00 | u32 | 4 | ? |
| 0x04 | u32 | 0 | ? |
| 0x08 | u16 | 0x04D9 | IAP USB VID |
| 0x0A | u16 | 0x1301 | IAP USB PID |
| 0x0C | u16, u16 | 0x0000, 0x0404 | As a word: 0x04040000, the application signature value |
| 0x10 | u32 | 0xFFFFFFFF | |
| 0x14 | u32 | 1 | ? |
| 0x18 | u32 | 0xFFFFFFFF | |
| 0x1C | u32 | 0xD5 | ? |
| 0x20 | u16 | 0x0200 | Firmware offset (application at version page + 0x200) |
| 0x22 | u16 | 0x00E7 (231) | Firmware pages |
| 0x24 | u16 | 0x0200 | Page size |
| 0x26 | char[10] | `KB0301` | Model |

Field names for 0x20–0x26 follow pok3rtool's CYKB decoder. Raw reply: [`captures/2026-10-24_cykb_info.txt`](../captures/2026-10-24_cykb_info.txt).

## Boot decision

`main` (0x1DA8) runs this on every reset. The magic word is at SRAM 0x20003FFC; the reset flags come from RSTCU GRSR (0x40088100).

1. If the magic is outside 0x55AAFAF0..0x55AAFAF5, set it to 0x55AAFAF0.
2. Else, if the magic is below 0x55AAFAF5 and GRSR bit 2 (watchdog reset) is set, increment it.
3. Clear the reset flags (write 0xF to GRSR).
4. If the magic is not 0x55AAFAF5, run the application check (0x1D40), which jumps to the application if it passes.
5. Otherwise stay in the IAP's USB loop.

So:

- An application enters the IAP by writing 0x55AAFAF5 to 0x20003FFC and resetting.
- Each watchdog reset increments the counter. After five of them (0x55AAFAF0 to 0x55AAFAF5) the IAP stays resident. This is a recovery path for an application that hangs without reloading the watchdog.
- Other resets leave the counter where it is. The IAP never puts it back to 0x55AAFAF0 while it is in range, so the five watchdog resets need not be consecutive unless the application clears the word itself. A power cycle normally leaves an out-of-range value in SRAM, which step 1 replaces.
- No GPIO, key or pad is read. SEL1 (PA8) plays no part.

## Application validity check

The IAP jumps to the application at 0x3200 only if **all** of these hold:

| Check | Requirement |
| --- | --- |
| Initial SP (0x3200) | ≤ 0x20004000 (computed as `SP + 0xE0000000 < 0x4001`) |
| Reset vector (0x3204) | < 0x3200 + 0x1CC00 (= 0x1FE00) |
| **Application signature (0x3BF8)** | **== 0x04040000** |
| Version page magic (0x30B0) | low 18 bits == 0x5AA5; n = magic >> 18, and n ≤ 14 |
| n words from 0x3078 | none equal to 0xFFFFFFFF |

It then loads SP and PC from 0x3200 (`bl 0x24`). **The IAP does not set VTOR**; the application must point VTOR at 0x3200 itself. (The Cortex-M0+ in this part has VTOR. The stock application sets it.)

## Application header (0x3BF8–0x3C03)

Twelve bytes in the middle of the application image, at application offset 0x9F8. The stock application has:

| Address | Value | Used by |
| --- | --- | --- |
| 0x3BF8 | `0x04040000` | Boot check above; checksum routine |
| 0x3BFC | `0x34000001` | Checksum routine (0x1658). Byte 0x3BFC = 0x01 enables it (without it the sum and CRC commands return 0). Byte 0x3BFF = 0x34 (52) is the re-encryption chunk size |
| 0x3C00 | `0x00020004` | Returned by info command `0x12/0x01`; not validated |

Replacement firmware must carry all three words at these addresses. Without the first, the IAP never starts it. Without the second, the IAP's verify command returns 0/0 and an updater will refuse to finish. We hit both failures (see [Porting QMK](qmk-port.md)).

## Version page (0x3000–0x31FF)

The updater writes this page **last**, after the application has been written and verified. Because the application check needs the page's magic, an interrupted update leaves the IAP in control. On this unit the page holds 0xB4 bytes (0x3000–0x30B3) followed by 0xFF:

```text
0x3000: 1a000000 56003100 2e003000 30002e00 30003500 00000000 00000000 0000ffff
0x3020: ffffffff ... (0xFF)
0x3060: ffffffff ffffffff ffffffff ffffffff ffffffff ffffffff 04001000 05000100
0x3080: db000000 ffffffef 01000000 00000000 d9040103 ffffffff ffffffff ffffffff
0x30a0: ffffffff ffffffff ffffffff ffffffff a55a1c00 ffffffff ...
```

| Address | Value | Meaning |
| --- | --- | --- |
| 0x3000 | u32 0x1A, then UTF-16LE `V1.00.05` | Firmware version string. Read by the stock app through commands `0x12/0x20`–`0x27` |
| 0x3078–0x3093 | 7 words: 0x00100004, 0x00010005, 0x000000DB, 0xEFFFFFFF, 0x00000001, 0x00000000, 0x030104D9 | Checked by the IAP (none may be 0xFFFFFFFF). The last word holds the application VID/PID halves (0x04D9, 0x0301). Other fields unknown |
| 0x30B0 | 0x001C5AA5 | Magic: low 18 bits 0x5AA5, n = 7 (the 7 words above) |

Raw read: [`captures/2026-10-24_cykb_info.txt`](../captures/2026-10-24_cykb_info.txt).

We wrote these same 180 bytes back after each QMK update, and the IAP accepted them. Whether other units carry different values (for example per-layout or per-version) is unknown. Nothing in the page looks unit-specific.

Version-page writes (below 0x3200) are **not** encrypted.

## Update protocol (CYKB over HID)

The IAP and the stock application both expose a vendor HID interface (usage page 0xFF00, 64-byte reports, interrupt EP IN/OUT). Requests and replies are 64 bytes. Byte layout (little-endian):

```text
request:  [cmd u8] [sub u8] [0x0000 u16] [payload ... up to 60 bytes]
reply:    [cmd u8] [sub u8] [status u16 = 0 on success] [data ...]
error:    status / reply 0xAAFF
```

On Linux hidraw, prefix a 0x00 report ID byte when writing.

| Cmd/Sub | Name | Payload | Reply | Notes |
| --- | --- | --- | --- | --- |
| 0x11/0x00 | Reset to IAP | — | none | App: writes the magic and resets. Tested |
| 0x11/0x01 | Reset to firmware | — | none | Tested from the IAP |
| 0x12/0x00 | Read info | — | info block from 0x400 | Read-only |
| 0x12/0x01 | Read header word | — | word at 0x3C00 | Read-only |
| 0x12/0x02 | Read mode | — | byte: 0 = IAP, 1 = application | Read-only |
| 0x12/0x20–0x27 | Read version page | — | 60-byte slices of 0x3000–0x31DF | Stock app; read-only |
| 0x1D/0x00 | Erase | offset u32, size u32 | — | Erases the pages covering the range. Can take seconds |
| 0x1D/0x01 | Sum | offset u32, size u32 | u32 | Needs offset ≥ 0x200, size ≥ 0x1000, and the application header |
| 0x1D/0x02 | CRC | offset u32, size u32 | u32 | Same conditions |
| 0x1E/0x01 | Set address | offset u32 | — | |
| 0x1E/0x00 | Get address | 0xFFFFFFFF | current offset | |
| 0x1F/len | Write | `len` bytes of data (≤ 60, multiple of 4) | 0x1E/0x02 with the next offset | Data at offsets ≥ 0x200 is RC4-decrypted (below) |

Addressing:

- **All offsets are relative to 0x3000** (the version page). The application starts at offset 0x200.
- Offset and size must be 4-aligned, size non-zero, and offset + size ≤ 0x1CE00. Otherwise the reply is 0xAAFF.
- The IAP therefore cannot erase or write its own pages.

Command numbering and the basic packet shape match pok3rtool's `CYKB_Device`. The address limits, sum/CRC conditions and encryption details come from the IAP disassembly. Erase, write, sum, CRC and both resets were exercised on this unit (see [Flashing and recovery](flashing-and-recovery.md)).

### Firmware encryption

- Write handler: 0xC20. For destination offsets ≥ 0x200 (absolute ≥ 0x3200), each packet is decrypted with **RC4** (0x09EC) before programming (0x0A78).
- The key schedule (0x1618) uses an **8-byte key stored at 0x434** in the IAP. The keystream restarts for every packet.
- pok3rtool's 13-word "POK3R RGB XOR key" for this protocol is simply the first 52 bytes of that RC4 keystream. XOR-encoding the image with it is correct **as long as every write packet carries exactly 52 bytes**. Packets of other sizes would need the real keystream.
- The firmware inside the vendor updater is stored already encoded this way.

The key bytes are not reproduced here: the 52-byte XOR form is already public in pok3rtool and is all a flasher needs.

### Sum and CRC

The checksum routine (0x1658) re-encrypts flash in chunks of [0x3BFF] = 52 bytes before summing, so both results match the **encoded** image, not the plaintext:

- Sum: 32-bit sum of the encoded image as little-endian words (modulo 2³²).
- CRC: standard CRC-32 (zlib) of the encoded image.

Example from this unit with stock V1.00.05 installed: sum `ad62fb58`, CRC `f2f1dee5`, matching the values computed from the updater's image ([`captures/2026-10-24_cykb_iap_sum.txt`](../captures/2026-10-24_cykb_iap_sum.txt)). This read-only check confirmed the encoding, the relative addressing and that the installed application was byte-identical to the updater's.
