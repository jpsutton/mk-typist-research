# Captures

Raw output from tools run against the keyboard on 2026-10-24. Paths inside these files (`updater/…`, `qmk_firmware/…`, `/dev/hidrawN`) refer to the original project workspace and are not part of this repo.

| File | What |
| --- | --- |
| `2026-10-24_stock-app_lsusb-v.txt` | `lsusb -v -d 04d9:0301` with stock firmware V1.00.05 (unrelated host devices removed) |
| `2026-10-24_normal_report_descriptors.txt` | HID report descriptors of the stock application's three interfaces, hex |
| `2026-10-24_cykb_info.txt` | Stock application: replies to `0x12/0x02` (mode), `0x12/0x01`, `0x12/0x00` (IAP info block) and the version page 0x3000–0x31DF |
| `2026-10-24_cykb_app_info.txt` | Info block decoded, in the application (mode 1) |
| `2026-10-24_cykb_iap_sum.txt` | In the IAP (mode 0): sum and CRC of the installed application against the updater's image. Read-only |
| `2026-10-24_isp_info.txt` | Holtek ROM ISP `info` (`ht32-dfu-tool`): model, flash size, lock state |
| `2026-10-24_cykb_flash_dumpread.txt` | Write of the patched stock application used to dump flash |
| `2026-10-24_cykb_flash_qmk_default.txt` | First QMK write: verify failed (missing checksum control word); version page not written, IAP stayed resident |
| `2026-10-24_cykb_flash_qmk_default_2.txt` | Second QMK write: verify OK; QMK booted |
| `2026-10-24_cykb_flash_qmk_leds.txt` | QMK with indicator LEDs |
| `2026-10-24_cykb_flash_qmk_fn.txt` | QMK with the Fn layer |
| `keytest-2026-10-24.txt` | Key test under QMK: Linux evdev code → layout key → matrix position, for all 108 keys |
