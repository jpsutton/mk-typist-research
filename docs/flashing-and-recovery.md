# Flashing and recovery

## Update through the stock IAP

This is how every image (patched stock, QMK, and stock again) was written to this unit. It keeps the stock IAP. Our client was a small Python script on Linux hidraw. It is not published yet, but the steps are fully described here and in [Stock IAP](stock-iap.md#update-protocol-cykb-over-hid).

1. **Enter the IAP** (04D9:1301): stock app command `0x11/0x00`, or Esc at plug-in or Fn+Esc on QMK.
2. **Check** `0x12/0x00` (info) reports firmware offset 0x200 and page size 0x200, and `0x12/0x02` (mode) reports 0 (IAP).
3. **Erase** with `0x1D/0x00`, offset 0, size = version page plus image rounded up to whole pages. We always erased at least 0x7E00 (0x3000–0xADFF, the stock application's span) so no stale stock code remains after a smaller image.
4. **Encode** the image: XOR it with pok3rtool's 13-word key, word by word, the key repeating every 52 bytes.
5. **Write** the encoded image at offset 0x200: `0x1E/0x01` set address, then `0x1F/52` packets of exactly 52 bytes. Each reply is `0x1E/0x02` with the next offset; check it.
6. **Verify** with `0x1D/0x01` (sum) and `0x1D/0x02` (CRC) over offset 0x200, image length. Compare with the sum and CRC-32 of the encoded image. **If they differ, stop.** The version page is still erased, so the IAP stays in control and you can retry.
7. **Write the version page** at offset 0, unencoded (180 bytes on this unit; see [Stock IAP](stock-iap.md#version-page-0x30000x31ff)).
8. **Start the application** with `0x11/0x01`.

Requirements on the image: built for 0x3200, the 12-byte application header at 0x3BF8, at least 0x1000 bytes long, and a multiple of 4 bytes (pad with 0xFF).

Logs of each write: [`captures/`](../captures/) (`*_cykb_flash_*.txt`). The first QMK attempt (`2026-10-24_cykb_flash_qmk_default.txt`) shows the verify failure caused by a missing checksum control word, and that the IAP survived it.

The vendor updater does the same over the same protocol. pok3rtool implements the CYKB protocol for other PIDs. At the time of writing (commit `7da3d51`) it did not list 04D9:0301/1301.

## Restoring stock firmware

Decode the application from the vendor updater (`pok3rtool extract maav102`, see [Stock firmware](stock-firmware.md)). The extracted image is plaintext. Write it with the procedure above, encoding it in step 4 like any other image, and then write the version page.

## How the flash was dumped

The chip is read-protected and the stock IAP has no read command, but code running from flash can read all of flash. The stock application already had a command that copies flash to a HID reply: `0x12` subcommands 0x20–0x27 read 60-byte slices of the version page. We patched its bounds check so that subcommand **0xFF** reads 60 bytes from any address given in request bytes 4–7. This is the same convention pok3rtool's `read_flash` uses, and the same idea as re-masterkeys' patched-firmware dump for Cooler Master boards.

The patch, in the stock V1.00.05 application (SHA-256 `0094d247…`), at flash address 0x6CEC in the vendor command handler (0x6CA4), 10 bytes:

```text
original: 20 28 0e d3 01 46 1f 39 3c 22
patched:  ff 28 0e d1 68 68 3c 21 32 e0

0x6cec  28ff  cmp  r0, #255     ; subcommand == 0xFF ?
0x6cee  d10e  bne  0x6d0e       ; no: continue with the 0/1/2 checks
0x6cf0  6868  ldr  r0, [r5, #4] ; address from the request
0x6cf2  213c  movs r1, #60      ; length
0x6cf4  e032  b    0x6d5c       ; existing "copy r1 bytes from r0 and reply" path
```

Side effect: the version-page reads (0x20–0x27) stop working in the patched application.

Steps on this unit:

1. Read-only check: in the IAP, `0x1D/0x01` and `0x1D/0x02` over the installed application matched the updater's image. That confirmed the protocol and encoding before any write.
2. Wrote the patched application and the version page through the IAP.
3. Read 0x00000–0x1FDFF twice with `0x12/0xFF`. Both reads had the same SHA-256.
4. Then flashed QMK over it. The patched stock app was never restored, because the updater's image can restore stock at any time.

Reading option bytes (0x1FF00) was not attempted.

## Recovery options, in order

| # | Situation | Action | Cost | Tested |
| --- | --- | --- | --- | --- |
| 1 | Application running | Its own bootloader entry: QMK Esc at plug-in or Fn+Esc; stock app `0x11/0x00` | None | Yes |
| 2 | Update failed before the version page was written | Nothing: the IAP stays resident. Retry the update | None | Yes |
| 3 | Application hangs without reloading the watchdog | Five watchdog resets leave the IAP resident | None | No (from disassembly) |
| 4 | Application hangs and reloads the watchdog, or breaks USB | ROM ISP (SEL2 at plug-in), **mass erase**, then write a full flash dump back at 0x0 | Erases everything, clears the lock. Needs **your own** dump of the IAP | No |
| 5 | No dump of the IAP | ROM ISP mass erase, then install a different bootloader (for example [hansemro/ht32_usb_dfu](https://github.com/hansemro/ht32_usb_dfu) ([archive](https://web.archive.org/web/20261025013420/https://github.com/hansemro/ht32_usb_dfu)), application at 0x3400) or flash over SWD | Stock IAP gone for good | No |

Before experimenting with application code that writes to internal flash (for example a wear-levelling EEPROM driver), remember that pages 0x1800–0x2FFF of the IAP are not write-protected. A bug there can cost you option 1 to 3.

**Make your own dump** with the patched-app method above before doing anything risky. The dump of this unit is not published; it contains Cheng Yong's bootloader.
