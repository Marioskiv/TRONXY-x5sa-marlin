<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Phone as Klipper host + flashing plan (2026-11-09)

Read-only check done over `adb` (platform-tools, nothing changed on the phone). Owner writes Greek.

## 1. The phone (verified with adb)
| Item | Value |
|---|---|
| Model | Samsung Galaxy A51, SM-A515F (Exynos 9611, arm64), EUR |
| Android | 13 (patch 2023-08-01), 3.6 GB RAM, 111 GB storage |
| Kernel | 4.14.113 stock Samsung, `CONFIG_MODULES=y`, no `/vendor/lib/modules` |
| USB-serial drivers in kernel | FTDI, PL2303 only. **CH341 not set**, no CDC_ACM |
| Root | No (`adb shell` = uid 2000, no `su`) |
| Bootloader | Locked (`flash.locked=1`, verifiedbootstate green), SELinux enforcing |
| Battery | Li-ion 4000 mAh, health Good, **917 charge cycles** |
| Charging while OTG | **Not verified** (needs printer plugged in, test) |
| Installed apps | Termux 0.118.3, F-Droid, **BeamKlipper 2.0.0 (Klipper 0.12.0)** |

BeamKlipper runs Klipper inside Android without root and has its own userspace CH34x driver.
Logs from 2026-02-26 (`/sdcard/Download/klippy.log.txt`): MCU was reached as
`usb-1a86_USB_Serial-if00-port0`, ran about 2 h, but with `bytes_retransmit=268`, `bytes_invalid=217`
and many `Serial connection closed`. Cause not analysed (Android killing the app, cable, OTG, power?).

## 2. Verdict
- (a) Phone as host as-is: works with BeamKlipper, **but the USB link was unreliable** in the logs.
- (b) Native Linux on the A51: needs bootloader unlock (wipes the phone, Knox tripped). Not recommended;
  support for the SM-A515F not verified.
- (c) **Chosen direction:** Raspberry Pi (Zero 2 W is enough without screen/camera; Pi 3B+ for more
  headroom) on **UART (USART3, PB10/PB11)**. The phone is only the screen (Mainsail / Mobileraker over WiFi).
  KlipperScreen on Android is not recommended (needs an X server app).
- Phone battery (917 cycles) must not sit at 100 % 24/7 if left on the printer.
- Orange Pi Zero 2 / Zero 3 are supported by MainsailOS but UART/permissions are more work. Not chosen.
- Pi Zero 2 W caveats: 512 MB RAM (no camera, no KlipperScreen on it), WiFi 2.4 GHz only, some reports of
  WiFi dropouts. Print does not depend on WiFi.

## 3. Owner decision: flashing rule dropped
The old rule "no stm32flash / no writes to 0x08000000" was arbitrary and is dropped (commit 040706b).
Claude still does not flash the printer itself; the owner flashes after Claude explains the steps.

## 4. Klipper PC facts (checked over SSH, read-only)
- Linux Debian, host `debian`, Klipper in `~/klipper`.
- `.config`: `CONFIG_STM32_FLASH_START_0000=y` -> **built for offset 0x0, no bootloader**.
  So the Chitu bootloader on the board was overwritten; SD update with `update.cbd` cannot work today.
- `~/klipper/update.cbd` and `~/Desktop/update.cbd`: both 38648 B, different md5, dated 2026-03-10,
  older than the latest `out/klipper.bin` (38364 B, 2026-03-10 13:13). Probably own Klipper attempts,
  **not** the stock Chitu firmware (stock `update.cbd` is still owed, see `facts.md`).
- `stm32flash` and `dfu-util` are already installed on the Klipper PC.

## 5. Plan: make SD updates work (owner wants this; avoids opening the case for every update)
Klipper already ships `scripts/update_chitu.py` (klipper.bin -> update.cbd, app at 0x08008800).
Missing piece: the Chitu bootloader image.

Downloaded candidate (J.C. Nelson, hackaday.io project 167594, 2019, from a Tronxy X5S / Chitu board):
- File: `chitutronxybootloader.bin`, 34816 B (= 0x8800), SHA-256
  `bc2c3f90b3890095a820713f475b8613910688273008bd32e086698a207f4d18`
- Vector table looks sane: SP `0x2000700c`, reset `0x08000165`, tail padded with 0xFF.
- **NOT verified for our V8 board** (CXY-V8-190729). Two Chitu bootloader generations exist (.cbd vs .bin).

Steps (owner flashes, once, with BOOT jumper over USB):
1. On the Klipper PC: menuconfig with `STM32_FLASH_START_8800`, keep the MCU settings
   (for the Pi: serial on USART3 PB11/PB10, 250000 baud, 8 MHz crystal), `make`.
2. `./scripts/update_chitu.py ./out/klipper.bin ./out/update.cbd`
3. `stm32flash` with BOOT jumper: write the bootloader at 0x08000000 and `klipper.bin` at 0x08008800
   (exact commands to be written out before doing it).
4. Remove BOOT jumper, put `update.cbd` on the SD card, check that the board updates.
5. If SD does not work: re-flash Klipper at offset 0x0 over USB + BOOT as before. Recovery is always
   possible with the BOOT jumper (STM32 ROM bootloader).
Open: does Klipper start correctly when started by the Chitu bootloader ("leaves hardware half-initialised",
see `docs/hardware/sources.md`)? Only a test shows it.
An STM32F103 cannot be flashed over USB/serial without BOOT0 high, so the first flash needs the jumper.

## 6. Still open
- Photos / measurement of WiFi socket J1 before connecting the Pi (pin numbering not verified).
- K2 Plus thermistor, HGX extruder values, stock Chitu `update.cbd`.
- Whether the phone charges while in OTG host mode (only matters if the phone stays as a host).
- Original question to owner: what exactly was tried before with Linux on a phone (Termux was installed
  2026-04-07, BeamKlipper 2025-12-02). Not answered yet.
