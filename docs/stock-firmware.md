# Stock firmware

## The updater

| Item | Value |
| --- | --- |
| File | `MK_Typist_V1.00.05NL.exe`, 3,015,112 bytes, SHA-256 `cc214797c5e9afe91bd077e447908dd633759337e733629fae8aed225e8ca440` |
| Original URL | `https://mechanicalkeyboards.com/downloads/MK_Typist_V1.00.05NL.exe` (returns 404 since at least August 2025; original Last-Modified 2018-01-02) |
| Archived copy | [Internet Archive, 2023-07-20 capture](https://web.archive.org/web/20230720170032/https://mechanicalkeyboards.com/downloads/MK_Typist_V1.00.05NL.exe). Earlier captures from 2018-07-03 and 2023-06-06 have the same archive digest |
| Format | Cheng Yong "maaV102" updater: strings "Cheng Yong FWupdate V1.00", `.maaV102` trailer signature |
| Header IDs | Application 04D9:0301, IAP 04D9:1301 |
| Payload | One image, "US Layout", 31,728 bytes, V1.00.05 |
| Decoder | `pok3rtool extract maav102` ([pok3r-custom/pok3rtool](https://github.com/pok3r-custom/pok3rtool) ([archive](https://web.archive.org/web/20251025230811/https://github.com/pok3r-custom/pok3rtool))) |

We never ran the updater; it was only analysed statically. The Ducky One updater format (52-byte XOR scheme handled by usedbytes/ducky-tools) does **not** apply. A search for that header found nothing.

After decoding, the payload is plain Cortex-M0+ code linked at 0x3200, SHA-256 `0094d2472a2e160d095387df2f05ce13f78a38d21d45bf1ac33f84264efaea9c`. This matches the firmware installed on the keyboard byte for byte (checked through the IAP's sum and CRC, see [Stock IAP](stock-iap.md#sum-and-crc)). The stock application can therefore always be restored through the IAP from the public updater, as long as the IAP survives.

Inside the updater the payload is XOR-encoded with the 52-byte key that pok3rtool knows, which is the first 52 bytes of the IAP's RC4 keystream.

## Application overview

All addresses are flash addresses with the image loaded at 0x3200. Functions are named by address.

| Item | Address | Notes |
| --- | --- | --- |
| Vector table | 0x3200 | 48 entries (0xC0 bytes); reset vector 0x32C1 |
| Application header | 0x3BF8 | `0x04040000 0x34000001 0x00020004` |
| Vendor HID command handler | 0x6CA4 | Serves `0x11`, `0x12` subcommands |
| Reset to IAP | 0x6DB0 | Writes 0x55AAFAF5 to 0x20003FFC, then resets |
| Matrix scan | 0x8D04 | 16 strobes, 3-scan debounce |
| Keymap | 0xA0D6 | 128 bytes, one HID usage (or vendor code) per slot, strobe-major |
| Matrix input pin table | 0xA156 | (port, pin) pairs |
| Matrix strobe pin table | 0xA166 | (port, pin) pairs |
| MBI5042 serial pin table | 0xA2A2 | |
| LED row table | 0xA4C8 | (register, value) pairs for GPIO set/reset |
| LED row pin table | 0xA508 | |
| USB device descriptor | 0xA6D0 | |
| GCLK pin descriptor | 0xA7A9 | PC10, AF4 |
| SPI flash descriptor | 0xA7C4 | CS PB7 then SPI1 pins |
| Indicator routine | 0x5A28 | |
| LED refresh ISR | BFTM1 (vector 0x88) | 8 rows |

Other behaviour:

- Sets VTOR to 0x3200 itself.
- Reloads the watchdog (0x5FA00001 to 0x40068000) from its main loop.
- Uses no internal flash for settings: on this unit, everything from 0xAE00 to the end of flash was 0xFF.
- Uses the SPI flash (U5) for something; contents unknown (not dumped).

## Keys with vendor codes

The keymap table mixes HID usages with internal codes ≥ 0xC0:

| Code(s) | Position | Meaning |
| --- | --- | --- |
| 0xD8 | S7 In4 (Menu position) | **Fn** |
| 0xF6, 0xF5, 0xEE, 0xF4 | S13 In4–In7 | Vol+, Vol−, Calculator, Mute (order confirmed by key test) |
| 0xC0–0xC3, 0xC9–0xCC | S15 In0–In7 | Probably the DIP switches (In0–In3); rest unknown |

## Mouse mode (the "M" LED)

The fourth indicator, legend **M**, shows a numpad mouse mode:

- Internal Fn action 0x30 toggles bit 3 of a flag byte at 0x20000AAF.
- While that bit is set, routine 0x9B08 remaps numpad usages through a table at 0x9FBB: KP7, KP8, KP9, KP6, KP3, KP2, KP1, KP4 to internal codes 0xE0–0xE7 in that order (the 8 directions, clockwise from up-left); KP− and KP+ to 0xE8/0xE9; KP0 and KP. to 0xF0/0xF1; KP5 to 0xFE.
- The indicator routine lights the M LED from the same bit.
- The mouse report on USB interface 2 (report ID 3) carries the result.

What 0xE8/0xE9, 0xF0/0xF1 and 0xFE do (probably wheel and buttons) was not traced. Which Fn combination triggers action 0x30 was not traced either.

## LED handling

The firmware shifts 48 bits per channel to the MBI5042, as if three drivers were cascaded (the RGB variants of this platform). The Typist has one driver, so only one of the three 16-bit words reaches an LED. Details in [Indicator LEDs](indicator-leds.md).
