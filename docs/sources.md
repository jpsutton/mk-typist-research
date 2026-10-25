# Sources and credits

Every external link in these docs, except links to this project's own repositories, is followed by an "archive" link to an Internet Archive copy, in case the original moves or disappears. Snapshots were taken or looked up on 2026-10-24; older snapshots may predate later changes to a page.

## Prior research this work builds on

Most of what made this port quick was already worked out by others on related boards. Thank you all.

| Who | Work | What it gave this project |
| --- | --- | --- |
| **hansemro** (Hansem Ro) | [ChibiOS-Contrib PR #400](https://github.com/ChibiOS/ChibiOS-Contrib/pull/400) ([archive](https://web.archive.org/web/20261025013233/https://github.com/ChibiOS/ChibiOS-Contrib/pull/400)) and earlier HT32 HAL work | HT32F165x/HT32F523xx support in QMK's ChibiOS-Contrib, so the port needed no HAL work |
| | [openocd-ht32](https://github.com/hansemro/openocd-ht32/tree/ht32f165x-dev) ([archive](https://web.archive.org/web/20221026225109/https://github.com/hansemro/openocd-ht32/tree/ht32f165x-dev)), branch `ht32f165x-dev` | OpenOCD with an HT32 flash driver |
| | [ht32-dfu-tool](https://github.com/hansemro/ht32-dfu-tool) ([archive](https://web.archive.org/web/20261025013353/https://github.com/hansemro/ht32-dfu-tool)) | Talking to the Holtek ROM ISP; how we read the lock state |
| | [ht32_usb_dfu](https://github.com/hansemro/ht32_usb_dfu) ([archive](https://web.archive.org/web/20261025013420/https://github.com/hansemro/ht32_usb_dfu)) | Replacement DFU bootloader (recovery option) |
| | [qmk_firmware, branch `prolwhite_mbi5042_test`](https://github.com/hansemro/qmk_firmware/tree/prolwhite_mbi5042_test) ([archive](https://web.archive.org/web/20261025013600/https://github.com/hansemro/qmk_firmware/tree/prolwhite_mbi5042_test)) (Cooler Master MasterKeys Pro L White port) | Independent confirmation of our pin map, 0x3200 base, IAP magic and watchdog behaviour on the same platform; the MBI5042 configuration word |
| | [qmk_firmware, branch `cmmkproswhite_mbi_preview`](https://github.com/hansemro/qmk_firmware/tree/cmmkproswhite_mbi_preview) ([archive](https://web.archive.org/web/20261025013533/https://github.com/hansemro/qmk_firmware/tree/cmmkproswhite_mbi_preview)) | Generic MBI driver; the port's `mbi5042.c` is adapted from his code |
| **pok3r-custom** (ChaoticEnigma, jakllsch, hansemro, theyy29 and others) | [pok3rtool](https://github.com/pok3r-custom/pok3rtool) ([archive](https://web.archive.org/web/20251025230811/https://github.com/pok3r-custom/pok3rtool)) | The CYKB update protocol, the `maaV102` updater extractor and the XOR key; the decoder we used on the vendor updater |
| | [pok3r_re_firmware](https://github.com/pok3r-custom/pok3r_re_firmware) ([archive](https://web.archive.org/web/20260522024320/https://github.com/pok3r-custom/pok3r_re_firmware)) and its [wiki](https://github.com/pok3r-custom/pok3r_re_firmware/wiki) ([archive](https://web.archive.org/web/20200928071747/https://github.com/pok3r-custom/pok3r_re_firmware/wiki)) | Holtek keyboard memory maps ([Vortex CORE notes](https://github.com/pok3r-custom/pok3r_re_firmware/wiki/Vortex-CORE-Notes) ([archive](https://web.archive.org/web/20230127090445/https://github.com/pok3r-custom/pok3r_re_firmware/wiki/Vortex-CORE-Notes))), the factory flash lock and what unlocking costs ([HT32 Unlocking](https://github.com/pok3r-custom/pok3r_re_firmware/wiki/HT32-Unlocking) ([archive](https://web.archive.org/web/20250818113453/https://github.com/pok3r-custom/pok3r_re_firmware/wiki/HT32-Unlocking))) |
| | [pok3r_re_firmware issue #32](https://github.com/pok3r-custom/pok3r_re_firmware/issues/32) ([archive](https://web.archive.org/web/20200928071834/https://github.com/pok3r-custom/pok3r_re_firmware/issues/32)) | The "HT50F52352" marking on HT32F52352 parts |
| | [pok3rtool issue #6](https://github.com/pok3r-custom/pok3rtool/issues/6) ([archive](https://web.archive.org/web/20221026225025/https://github.com/pok3r-custom/pok3rtool/issues/6)) | Cooler Master boards on this platform; dumping through patched firmware |
| | [qmk_pok3r](https://github.com/pok3r-custom/qmk_pok3r) ([archive](https://web.archive.org/web/20250527144238/https://github.com/pok3r-custom/qmk_pok3r)) | Earlier QMK on Holtek keyboards |
| **ChaoticEnigma** | [openocd-ht32](https://github.com/ChaoticEnigma/openocd-ht32) ([archive](https://web.archive.org/web/20250824220005/https://github.com/ChaoticEnigma/openocd-ht32)) | The original HT32F165x OpenOCD fork |
| **mateuszradomski** | [re-masterkeys](https://github.com/mateuszradomski/re-masterkeys) ([archive](https://web.archive.org/web/20251024001256/https://github.com/mateuszradomski/re-masterkeys)) and [issue #1](https://github.com/mateuszradomski/re-masterkeys/issues/1) ([archive](https://web.archive.org/web/20221026224916/https://github.com/mateuszradomski/re-masterkeys/issues/1)) | HT32F52352 + MBI reverse engineering on Cooler Master boards; the patched-firmware dump approach; discussion of QMK on locked chips |
| **OpenDucky** ("codetector") | [qmk_firmware, branch `mk_typist`](https://github.com/OpenDucky/qmk_firmware/tree/mk_typist) ([archive](https://web.archive.org/web/20261025013858/https://github.com/OpenDucky/qmk_firmware/tree/mk_typist)), [ChibiOS-Contrib](https://github.com/OpenDucky/ChibiOS-Contrib) ([archive](https://web.archive.org/web/20251102141223/https://github.com/OpenDucky/ChibiOS-Contrib)), [ht32-usb-dfu](https://github.com/OpenDucky/ht32-usb-dfu) ([archive](https://web.archive.org/web/20261025013728/https://github.com/OpenDucky/ht32-usb-dfu)) | The 2020 MK Night Typist QMK port: the starting point, the key arrangement, the MBI5042 indicator layout, the idea of a bootloader offset |
| **usedbytes** | [ducky-tools](https://github.com/usedbytes/ducky-tools) ([archive](https://web.archive.org/web/20251029092032/https://github.com/usedbytes/ducky-tools)) | Ducky updater formats. It did not apply here, which helped identify the Cheng Yong format |
| **Akkamir** | [ducky-rgb](https://github.com/Akkamir/ducky-rgb) ([archive](https://web.archive.org/web/20261025013204/https://github.com/Akkamir/ducky-rgb)) | Ducky firmware extraction background |
| **QMK contributors** | [annepro2](https://github.com/qmk/qmk_firmware/tree/master/keyboards/annepro2) ([archive](https://web.archive.org/web/20250214035358/https://github.com/qmk/qmk_firmware/tree/master/keyboards/annepro2)) | Mainline QMK on the same HT32F52352: toolchain, clock setup, SPI-flash EEPROM model |
| | [ducky/one2mini PR #11112](https://github.com/qmk/qmk_firmware/pull/11112) ([archive](https://web.archive.org/web/20250311013048/https://github.com/qmk/qmk_firmware/pull/11112)) | MBI5042 driver code on a Ducky board |
| **Evan Pratten** | [QMK on the Vortex Core](https://ewpratten.com/blog/qmk-vortex-core/) ([archive](https://web.archive.org/web/20250819010752/https://ewpratten.com/blog/qmk-vortex-core)) | End-to-end walkthrough on a Holtek keyboard |
| **probe-rs** | [Holtek support](https://docs.rs/crate/probe-rs/latest/source/src/vendor/holtek/mod.rs) ([archive](https://web.archive.org/web/20261025013051/https://docs.rs/crate/probe-rs/latest/source/src/vendor/holtek/mod.rs)) | HT32 detection and an HT32F523xx flash algorithm (fallback, not used) |
| **Raspberry Pi** | [debugprobe](https://github.com/raspberrypi/debugprobe) ([archive](https://web.archive.org/web/20261013192043/https://github.com/raspberrypi/debugprobe)) | SWD probe firmware for a Pico |
| **Internet Archive** | [Wayback Machine](https://web.archive.org/) | The only surviving copy of the vendor updater |

Reviews and community posts that identified the hardware:

- [Keychatter: The MK Typist review](https://www.keychatter.com/2019/09/05/the-mk-typist-review-boring-in-the-best-way-possible/) ([archive](https://web.archive.org/web/20220524081445/https://www.keychatter.com/2019/09/05/the-mk-typist-review-boring-in-the-best-way-possible/)): Ducky as the maker, MBI5042 fitted on the unlit Typist.
- [Geekhack: MK Typist thread](https://geekhack.org/index.php?topic=95763.0) ([archive](https://web.archive.org/web/20250818180035/https://geekhack.org/index.php?topic=95763.0)): Ducky as OEM, the updater name.
- [TechPowerUp: MK Night Typist review, page 4](https://www.techpowerup.com/review/mechanical-keyboards-mk-night-typist-keyboard/4.html) ([archive](https://web.archive.org/web/20261025014152/https://www.techpowerup.com/review/mechanical-keyboards-mk-night-typist-keyboard/4.html)): Night Typist teardown, HT32F1654 and 25Q40 markings.

## MCU documentation

| Document | Link | Notes |
| --- | --- | --- |
| HT32F52342/52352 product page | [holtek.com](https://www.holtek.com/page/vg/HT32F52352) ([archive](https://web.archive.org/web/20250308053204/https://www.holtek.com/page/vg/HT32F52352)) | Datasheet, user manual, firmware library, application notes |
| HT32F52342/52352 datasheet v1.60 | [PDF](https://www.holtek.com/webapi/116711/HT32F52342_52_Datasheetv160.pdf) ([archive](https://web.archive.org/web/20250418094327/https://www.holtek.com/webapi/116711/HT32F52342_52_Datasheetv160.pdf)) | Current revision. Pin assignment (Table 3), alternate functions, memory map |
| HT32F52342/52352 user manual v1.60 | [PDF](https://www.holtek.com/webapi/187531/HT32F52342_52_UserManual_v160.pdf) ([archive](https://web.archive.org/web/20261025014137/https://www.holtek.com/webapi/187531/HT32F52342_52_UserManual_v160.pdf)) | Current revision. Flash controller, protection (Table 9), GPIO/AFIO, timers, watchdog, reset |
| HT32F1653/1654 product page | [holtek.com](https://www.holtek.com/page/vg/HT32F1654) ([archive](https://web.archive.org/web/20261025014041/https://www.holtek.com/page/vg/HT32F1654)) | The MCU earlier notes assumed. Datasheet and user manual for the Vortex boards |

The revisions we actually worked from (not redistributed):

| File | SHA-256 |
| --- | --- |
| `HT32F52342_52_Datasheetv150.pdf` | `44c2ee213e8df3cb895a3a3fb35cfb12421e2a16e74671e31a0d516908e45ca3` |
| `HT32F52342-52_UserManualv130.pdf` | `188d4066a1b852e2292337a2dc51d3f502e2c5821daae4183178c42997e24abc` |
| `HT32F1653-1654_Datasheet_v100.pdf` | `ce5e278dfa8c333c7d36b9b1bb8554a7c87a8acb7db9c8f58961cc206e337c28` |
| `HT32F1653-1654_UserManual_v100.pdf` | `f456a39751ca3b7fca4eb81b1695c16ffc0fe7d23c759f7042748768f8acaddc` |

## Vendor files (not redistributed)

| File | Where to get it | SHA-256 |
| --- | --- | --- |
| `MK_Typist_V1.00.05NL.exe` (3,015,112 B) | Original, now 404: `https://mechanicalkeyboards.com/downloads/MK_Typist_V1.00.05NL.exe`. Archived: [Wayback Machine, 2023-07-20](https://web.archive.org/web/20230720170032/https://mechanicalkeyboards.com/downloads/MK_Typist_V1.00.05NL.exe) | `cc214797c5e9afe91bd077e447908dd633759337e733629fae8aed225e8ca440` |
| Application extracted with `pok3rtool extract maav102` (31,728 B, plaintext, "US Layout" V1.00.05) | Derived from the updater | `0094d2472a2e160d095387df2f05ce13f78a38d21d45bf1ac33f84264efaea9c` |
| Version template inside the updater (180 B) | Derived from the updater | `865a809a163579e8d1f2bc68c0b6c3b742e4cc661be220e7d085d3098ac25e4b` |
| Version page, first 0xB4 bytes of 0x3000 | Hex in [Stock IAP](stock-iap.md#version-page-0x30000x31ff); also the template above with one byte moved | `99452e81b7674c8f2473cf2e83a83dc85fce5039972c9a63506a45290987d373` |
| Full flash dump of this unit, 0x00000–0x1FDFF (two reads) | Not published | `35f6e0166c282715dfe107b4c440e718093ec5ff5e32a271babccc3906c15226` |
| Stock IAP region of that dump, 0x0000–0x2FFF | Not published | `fbb68c38129d066a05ddcd990bf566a937aef91243a7c6e89014ce3bf9429841` |

Ducky One 2 full-size updaters, used only for the [comparison in Related boards](related-boards.md#ducky-one-2-full-size). Both are listed on Ducky's [downloads page](https://ducky.global/pages/downloads) ([archive](https://web.archive.org/web/20261025015638/https://ducky.global/pages/downloads)) under "One 2 100% Full Size".

| File | Where to get it | SHA-256 |
| --- | --- | --- |
| `One2_NonBacklit_V1.01.01.exe` (3,045,060 B), DKON1808 | [Ducky CDN](https://cdn.shopify.com/s/files/1/0728/4382/1295/files/One2_NonBacklit_V1.01.01.exe) ([archive](https://web.archive.org/web/20261025015659/https://cdn.shopify.com/s/files/1/0728/4382/1295/files/One2_NonBacklit_V1.01.01.exe)) | `7445d7e89af0d6f0ea536460df21479e4e3d97fbfda6a35097c24580339c7446` |
| `One2_Backlit_V1.01.01.exe` (3,046,892 B), DKON1808S | [Ducky CDN](https://cdn.shopify.com/s/files/1/0728/4382/1295/files/One2_Backlit_V1.01.01.exe) ([archive](https://web.archive.org/web/20261025015711/https://cdn.shopify.com/s/files/1/0728/4382/1295/files/One2_Backlit_V1.01.01.exe)) | `9c7e420ae4205211e635e6204b410dee35c51269d6497dadb53c3599ec205b03` |
| Non-backlit "US Layout" image (30,744 B) | Derived | `3404fc82adc3c1891ca445dbbc79b8f54940e443eae26e1f63466fd2da54b358` |
| Non-backlit "EU Layout" image (30,752 B) | Derived | `6b5617444634fab11e01b8022ba0d40d861b8ed4c5e2154fe8d62dda4aafe74f` |
| Backlit "US Layout" image (31,660 B) | Derived | `fdf05a2b4896534ce139dcea06a18d597f2c457237ebfb369cc853c4c0d12f12` |
| Backlit "EU Layout" image (31,668 B) | Derived | `4369ea63a574a4efc9212af5e43a9ee9c2fc32665bb673c05f63ea761c68990a` |
| Version template, identical in all four | Derived | `88d8f4b2ed40b62f48d6a51819b8cfd40ac85703edb9065c57908e1a365a0820` |

Product page: [mechanicalkeyboards.com/products/mk-typist](https://mechanicalkeyboards.com/products/mk-typist) ([archive](https://web.archive.org/web/20251118105630/https://mechanicalkeyboards.com/products/mk-typist)).

## Tools used

QMK CLI ([`ghcr.io/qmk/qmk_cli`](https://github.com/qmk/qmk_cli) ([archive](https://web.archive.org/web/20260911215028/https://github.com/qmk/qmk_cli))), [Ghidra](https://ghidra-sre.org/) ([archive](https://web.archive.org/web/20250512185340/https://ghidra-sre.org/)) (headless decompilation), GNU Arm toolchain `objdump`, pok3rtool, ht32-dfu-tool, hansemro's OpenOCD fork (built, not used on hardware), a multimeter, and Python on Linux hidraw for the IAP client and the key tester (published in [mk-typist-tools](https://github.com/jpsutton/mk-typist-tools)).
