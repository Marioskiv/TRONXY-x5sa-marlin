<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Project status (updated 2026-11-09)

## Done
- Repo skeleton, facts/assumptions file, research folder (`docs/research/`).
- Step 1 (partial): Tronxy `printer.cfg` + `moonraker.conf` saved (`backup/klipper/pasted/`).
- Step 2: pin comparison -> **no pin differs** from Marlin BOARD_CHITU3D_V5 (`docs/pin-comparison.md`).
- WiFi socket = USART3 PB10/PB11 (schematic + Tronxy official code) (`docs/hardware/wifi-socket.md`).
- PA14 probe needs DISABLE_DEBUG in Marlin (`docs/research/v8-z-endstop-pa14-swd.md`).
- Display plan: Color UI first, Galaxy UI as experiment; English only (`docs/plan-display-ui.md`).
- Runtime config / WiFi / debug plan (`docs/plan-runtime-config-wifi-debug.md`).
- Klipper vs Marlin feature list incl. adaptive mesh (`docs/klipper-vs-marlin.md`).

## DIRECTION CHANGE #2 (2026-11-11)
Raspberry Pi Zero 2 W not available anywhere (list price 15 USD, street prices up to 188 EUR); owner does not want a Raspberry.
-> **Back to the Marlin plan** (stand-alone on the board, original screen, ESP3D WiFi). Klipper-on-Pi plan kept as a later option.
Open: owner's typical Klipper print speed; `grep FLASH_START` result; how Klipper was flashed (SD or USB).

## DIRECTION CHANGE #1 (2026-11-09)
Owner chose **Klipper on a dedicated Raspberry Pi** with a **serial (UART) link** and a **touchscreen**
-> see `docs/plan-klipper-pi.md`. Marlin plan kept as fallback.

## Decisions taken by owner
- UI language English. Screen should be nice and helpful.
- More research welcome; all findings stored in `docs/research/`.
- I (Claude) never flash; owner flashes.

## OPEN - blocking
1. **Bootloader**: printer.cfg note says Klipper was flashed by USB with the BOOT jumper and "No bootloader"
   -> Chitu bootloader probably overwritten -> update.cbd may not work.
   Next: owner runs `grep FLASH_START ~/klipper/.config`, `ls ~/klipper/out/`, `find ~ -name "*.cbd"`.
   Then owner chooses: A) Marlin by USB each time, B) restore a bootloader once by USB then SD updates,
   (owner dropped the 0x08000000 / stm32flash rule on 2026-11-09; pick whichever is easiest).
2. Hotend upgrade unfinished -> thermistor type.
3. Extruder -> "HGX pulse 562", ratio 9.54647:1 -> need maker's steps/rotation distance + motor step angle.

## OPEN - not blocking
- `.config` / `klipper.bin` of the Tronxy build, `klippy.log`.
- Photos: MCU marking (512 KB?), drivers (TMC2225?), WiFi socket front/back, Z/Z2 + jumpers, BOOT/5V jumpers.
- Voltages on WiFi socket pins. Which ESP module (suggest ESP32-C3 or D1 mini + 24->5 V buck).
- Klipper instance on PC: Moonraker port 7126 (`printer_TRONXY_data`), serial `usb-1a86_USB2.0-Serial-if00-port0`.
