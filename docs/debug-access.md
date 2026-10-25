# Debug access

## SWD header CN2

| CN2 pin | Signal | MCU pin |
| --- | --- | --- |
| 1 (square pad) | 3.3V | VDD |
| 2 | SWDIO (PA13) | 39 |
| 3 | SWCLK (PA12) | 38 |
| 4 | nRST | 20 |
| 5 | GND | VSS |

Traced with a multimeter. SWD was **never actually connected** during this project: the ROM ISP answered the lock question first, and with flash security on, SWD could not have read anything useful. The wiring and OpenOCD config below are prepared but untested on hardware.

### Probe wiring (Raspberry Pi Pico running debugprobe)

| Pico pin | Pico GPIO | CN2 |
| --- | --- | --- |
| — | — | 1 (3.3V): **leave unconnected**; power the keyboard from its own USB cable |
| 5 | GP3 | 2 (SWDIO) |
| 4 | GP2 | 3 (SWCLK) |
| 2 | GP1 | 4 (nRST) |
| 3 | GND | 5 (GND) |

Both sides are 3.3 V, so no level shifter is needed. Do not put an auto-direction shifter such as a TXS0108 on SWDIO; its direction sensing misbehaves on the SWD turnaround.

A Pico with [debugprobe](https://github.com/raspberrypi/debugprobe) ([archive](https://web.archive.org/web/20261013192043/https://github.com/raspberrypi/debugprobe)) is a CMSIS-DAP probe, which OpenOCD supports fully. An ST-Link V2 clone works only through OpenOCD's HLA layer, which the HT32 target file below does not support without changes.

### OpenOCD

Use the HT32 fork [hansemro/openocd-ht32](https://github.com/hansemro/openocd-ht32/tree/ht32f165x-dev) ([archive](https://web.archive.org/web/20221026225109/https://github.com/hansemro/openocd-ht32/tree/ht32f165x-dev)), branch `ht32f165x-dev` (we built `1f3300cc`). Its `ht32f165x` flash driver also handles the HT32F52352's flash controller (re-masterkeys used it on this chip). Things to know about its `tcl/target/ht32f165x.cfg`:

- Set `HT32_FLASH_SIZE` and `HT32_SRAM_SIZE` before sourcing it.
- It defaults to the Cortex-M3 SW-DP ID. Set `CPUTAPID 0x0bc11477` for the Cortex-M0+.
- It ends with `adapter speed 12` and `reset_config trst_only`. Override both to attach under nRST.
- `ht32f165x mass_erase` and `ht32f165x test_write` are **destructive**. `test_write` writes test data to flash.
- The `bcm2835gpio` driver is only built on ARM hosts in this version.

Config: [`openocd/typist.cfg`](../openocd/typist.cfg). Read-only first contact: `openocd -f openocd/typist.cfg -c init -c "flash info 0" -c shutdown`.

[probe-rs](https://probe.rs) ([archive](https://web.archive.org/web/20261006113830/https://probe.rs/)) ships an `HT32F523xx_Series` target with a flash algorithm, which could serve as a fallback.

## ROM ISP (USB)

Short SEL2 (PA9 to GND) while plugging in. The board enumerates as 04D9:8010. [hansemro/ht32-dfu-tool](https://github.com/hansemro/ht32-dfu-tool) ([archive](https://web.archive.org/web/20261025013353/https://github.com/hansemro/ht32-dfu-tool)) (we used `848fc07`) speaks the protocol:

- `list` shows the model (HT32F52352 here).
- `info` sends only the info command (0x03) and a 64-byte read of the option-byte page (commands 0x01/0x02). It reported the lock state recorded in [`captures/2026-10-24_isp_info.txt`](../captures/2026-10-24_isp_info.txt).
- `read` refuses to run while flash security is on.

## What the lock means

From the HT32F52342/52352 user manual (Table 9) and our tests:

| Access | With flash security on |
| --- | --- |
| Code running from main flash, no debugger attached | Full read, program and page erase |
| SWD (ICP) read | Returns 0 |
| ROM ISP read | Refused |
| ROM ISP mass erase | Allowed. Clears the lock, **and erases the stock IAP** |
| IAP (CYKB) commands | Erase, write, sum and CRC of 0x3000 and up. No read command in the stock IAP |

Practical consequences:

- There is no way to read the stock IAP from outside the chip. The only route is to run code on the chip that reads flash and sends it out. See [How the flash was dumped](flashing-and-recovery.md#how-the-flash-was-dumped).
- The vendor updater contains the application but not the IAP. Once the IAP is erased, the stock firmware cannot be fully restored unless you have your own dump.
- QMK runs fine on the locked chip. hansemro reported QMK hard faults on locked HT32 parts; we did not see any. No debugger was attached while it ran.
