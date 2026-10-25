# Porting QMK

The port lives on branch [`mk-typist`](https://github.com/jpsutton/qmk_firmware/tree/mk-typist) of `jpsutton/qmk_firmware`, folder `keyboards/mk/typist`. It is based on QMK master `180f394b48` (2026-10-16). This page explains what it needed and the problems we hit, so the same approach can be reused on related boards.

## Starting point

- QMK already supports the HT32F523xx family through ChibiOS-Contrib (hansemro's work, [ChibiOS-Contrib PR #400](https://github.com/ChibiOS/ChibiOS-Contrib/pull/400) ([archive](https://web.archive.org/web/20261025013233/https://github.com/ChibiOS/ChibiOS-Contrib/pull/400)), merged 2024-06-01). The `annepro2/c18` keyboard uses the same HT32F52352. It is the closest mainline template, and `qmk compile -kb annepro2/c18 -km default` builds out of the box.
- No ChibiOS fork is needed.
- The stock IAP stays in place. The port is an application at 0x3200 that the stock IAP starts and updates.

## Keyboard definition

| File | What it does |
| --- | --- |
| `keyboard.json` | 108-key `LAYOUT`, 16 × 8 matrix, `COL2ROW`, bootmagic on Esc [12,0], NKRO, extrakey, LTO, `eeprom.driver = transient` |
| `rules.mk` | `MCU = cortex-m0plus`, `MCU_SERIES = HT32F523xx`, `MCU_STARTUP = ht32f523xx`, custom linker script and board, `BOOTLOADER = custom`, vector alignment and VTOR flags |
| `mcuconf.h` | 8 MHz HSE, PLL ×6 to 48 MHz for core and USB (same as Anne Pro 2 C18); USB0; GPTM1 PWM and BFTM1 GPT for the LEDs |
| `boards/MK_TYPIST/` | `pal_default_config`: every matrix pin on AF1 (GPIO), strobes as outputs idle high, inputs with pull-ups |
| `ld/HT32F52352_MK_TYPIST.ld` | Application at 0x3200, application header at 0x3BF8, last SRAM word reserved |
| `typist.c` | Application header, watchdog reload, IAP jump, indicator LEDs |
| `mbi5042.c/h` | MBI5042 driver adapted from hansemro |
| `keymaps/default` | ANSI layout. Fn (Menu position) is `MO(1)`; layer 1 has `QK_BOOT` on Esc |

Size: the `default` build is 20,640 bytes, out of 113,152 available above 0x3200.

## Traps and how we got past them

### 1. The vector table landed at 0x3400

ChibiOS's `rules_code.ld` aligns `.vectors` to 1024 bytes by default, so a flash region starting at 0x3200 put the table at 0x3400, where the IAP does not look. Setting `__vectors_align__` inside the `.ld` file had no effect. Passing it on the linker command line does:

```make
EXTRALDFLAGS = -Wl,--defsym=__vectors_align__=512
```

Check with `arm-none-eabi-objdump -h`: `.vectors` must be at 0x3200.

### 2. VTOR is not set

The IAP jumps to the application without touching VTOR. ChibiOS leaves VTOR alone on ARMv6-M by default. Without this flag, interrupts go to the IAP's vector table:

```make
OPT_DEFS += -DCRT0_VTOR_INIT=TRUE
```

### 3. Pins that are not GPIO after reset

HT32 PAL modes do not change a pin's alternate-function field. PB10, PB11 and PB12 reset to X32KIN, X32KOUT and RTCOUT, and PC13 is also a backup-domain pad. As matrix inputs they read nothing until set to AF1. The board file sets AF1 on every matrix pin through `pal_default_config` `CFG[0]`/`CFG[1]`.

### 4. The watchdog

The watchdog is running when the IAP starts the application. QMK reloads it from `matrix_scan_kb`, and during the LED driver's start-up wait:

```c
*((volatile uint32_t *)0x40068000) = 0x5FA00001; // WDT_CR reload
```

Without this the board resets in a loop, and after five watchdog resets the IAP stays resident.

### 5. The IAP would not start QMK: application signature

The first build flashed, but would not have booted: the IAP only jumps to an application with **0x04040000 at 0x3BF8**. That address is in the middle of where ChibiOS puts code. The linker script splits flash into three regions so the header has a fixed home:

```text
flash0  : org = 0x00003200, len = 0x3BF8 - 0x3200   /* vectors */
flash1  : org = 0x00003BF8, len = 12                /* application header */
flash2  : org = 0x00003C04, len = 127k + 512 - 0x3C04
```

```c
__attribute__((section(".iap_header"), used))
static const uint32_t iap_header[3] = {0x04040000, 0x34000001, 0x00020004};
```

`.text` starts right after the header in `flash2`. Only the vector table lives in `flash0`, which leaves about 2.3 KB unused there.

### 6. The IAP's verify returned 0/0: checksum control word

With only the signature in place, the update reached the verify step and the IAP's sum and CRC came back as 0. The updater therefore did not write the version page, and the keyboard stayed in the IAP (by design, see [Stock IAP](stock-iap.md)). The disassembly showed that the checksum routine needs **0x34000001 at 0x3BFC**: byte 0x01 enables it and 0x34 is the 52-byte re-encryption chunk. It also needs the image to be at least 0x1000 bytes. Adding the full 12-byte stock header fixed it.

### 7. Entering the bootloader

`bootloader_jump()` does what the stock firmware's reset-to-IAP handler does:

```c
*((volatile uint32_t *)0x20003FFC) = 0x55AAFAF5;
__disable_irq();
NVIC_SystemReset();
```

The linker script keeps the last word of SRAM (`ram0 len = 16k - 4`) out of the image so startup code does not clear it. `mcu_reset()` is a plain `NVIC_SystemReset()`.

Bootmagic (Esc held at plug-in) and `QK_BOOT` both call it, and both were tested.

## Indicator LEDs

See [Indicator LEDs](indicator-leds.md). The port uses a custom `led_update_kb` because the indicators go through the MBI5042 and a row multiplexer, not plain GPIO. The M LED (stock mouse mode) is not driven.

## Not done yet

- **Persistent settings.** EEPROM is `transient`, so settings are lost at power-off. QMK has no HT32 embedded-flash driver. The proven path is annepro2's `wear_leveling` + `spi_flash` on the SPI flash (U5), once its wiring is checked and its stock contents are dumped.
- **VIA.** A VIA keymap builds (about 23 KB when last measured), but without persistent EEPROM it is of little use.
- **DIP switches.** `keyboard.json` maps `dip_switch.matrix_grid` on [15,0..3]. Unverified.
- **Flashing.** QMK's `:flash` target does not know the CYKB protocol. See [Flashing and recovery](flashing-and-recovery.md).
- **Upstreaming.** QMK review will likely ask for a few changes: no custom board definition in a keyboard PR, a pristine `default` keymap, VIA keymaps kept in a separate repo, a registered or acceptable USB ID, and an explanation of the custom flashing path.

## Build

We built in the official QMK CLI container:

```sh
docker run --rm -v "$PWD":/qmk_firmware -w /qmk_firmware ghcr.io/qmk/qmk_cli \
  qmk compile -kb mk/typist -km default
```

`qmk lint -kb mk/typist` passes, apart from the VIA keymap, which QMK does not allow inside the keyboard folder.
