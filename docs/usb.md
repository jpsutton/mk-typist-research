# USB

## Identities

| Mode | VID:PID | bcdDevice | Strings | Source |
| --- | --- | --- | --- | --- |
| Stock application V1.00.05 | 04D9:0301 | 1.05 | Product "USB-HID Keyboard"; no manufacturer or serial string | `lsusb -v` |
| Stock IAP | 04D9:1301 | — | Product "USB-HID IAP" | `lsusb`, IAP info block, updater header |
| Holtek ROM ISP | 04D9:8010 | — | — | `ht32-dfu-tool list` |
| QMK port | 4D4B:5459 | 0.0.1 | "Mechanical Keyboards" / "MK Typist" | `lsusb` |

0x04D9 is Holtek's vendor ID. The QMK port's 4D4B:5459 is ASCII "MK" / "TY", picked for the port. It is **not registered** with anyone and may need to change before upstreaming.

The stock firmware also carries a model string `CYRM02p0301h00DBr0005` (descriptor area at 0xA6D0 in the stock image).

## Stock application descriptors

Full speed, bus-powered, remote wakeup, 100 mA, one configuration with three HID interfaces:

| Interface | Class | Endpoints | Report descriptor | Use |
| --- | --- | --- | --- | --- |
| 0 | HID boot keyboard | EP1 IN, 8 B, 1 ms | 64 B: standard 6KRO boot keyboard with 3 LED outputs (Num, Caps, Scroll) | Keyboard (boot protocol) |
| 1 | HID, vendor | EP3 IN + EP4 OUT, 64 B, 1 ms | 34 B: usage page 0xFF00, 64-byte input and output reports, no report IDs | CYKB command / update channel (see [Stock IAP](stock-iap.md#update-protocol-cykb-over-hid)) |
| 2 | HID | EP2 IN, 64 B, 1 ms | 203 B, report IDs 1–4 | 1 = System Control (power, sleep, wake); 2 = Consumer Control (18 usages, including Calculator, volume, mute, media); 3 = mouse (8 buttons, 16-bit X/Y, wheel, AC Pan); 4 = NKRO keyboard bitmap |

The mouse report serves the stock firmware's numpad "mouse mode" (see [Stock firmware](stock-firmware.md)).

Raw output: [`captures/2026-10-24_stock-app_lsusb-v.txt`](../captures/2026-10-24_stock-app_lsusb-v.txt) and [`captures/2026-10-24_normal_report_descriptors.txt`](../captures/2026-10-24_normal_report_descriptors.txt) (hex, one line per interface).

## Stock IAP

The IAP enumerates with three hidraw nodes on Linux. Its vendor interface uses the same usage page (0xFF00) and 64-byte reports as the application's, so one tool can talk to both: find the hidraw node whose report descriptor starts `06 00 ff`. Full descriptors for the IAP were not captured.

## Linux permissions

hidraw nodes are root-only by default. A udev rule such as this lets a logged-in user reach the stock application, the IAP and the ROM ISP:

```text
# /etc/udev/rules.d/50-mk-typist.rules
SUBSYSTEM=="hidraw", ATTRS{idVendor}=="04d9", ATTRS{idProduct}=="0301|1301", TAG+="uaccess"
SUBSYSTEM=="usb", ATTRS{idVendor}=="04d9", ATTRS{idProduct}=="8010", TAG+="uaccess"
```

Then run `sudo udevadm control --reload && sudo udevadm trigger` and replug. The ROM ISP tool uses libusb, hence the `usb` subsystem line.
