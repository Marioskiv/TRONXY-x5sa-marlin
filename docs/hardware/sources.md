<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# External sources reviewed

| Source | Status | What it gives us |
|---|---|---|
| Marlin `pins_CHITU3D_V6.h` -> `x5sa-main_board-2.pdf` (Altium, 2019-06-03) | read | Full MCU schematic; WiFi J1 = USART3 PB10/PB11 (see wifi-socket.md). Revision not printed. |
| hackaday.io project 167594 "Chitu 3D Printer Mainboard Hacking" (J.C. Nelson) | read | Bootloader/update.cbd encryption; author of Marlin `chitu_f103` env. No pinout for V8. |
| ESP3D discussion #706 | read | Users report 5 V on CH_PD and GPIO0 grounded when an ESP-01 is plugged into the socket. |
| Configurations #658, #321; Marlin #19320 | read | V8 Z endstop on PA14 problem in Marlin; likely SWD left on (see docs/research/v8-z-endstop-pa14-swd.md). |
| EddyBeaupre/Tronxy-XY2-Pro | read | XY-2 Pro guide for V6 board; jumpers BOOT + 5V/USB; warning about update.cbd on newer firmware (docs/research/eddybeaupre-xy2pro-guide.md). |
| nathantsoi.com ESP3D build (X5S, 2019) | read | NodeMCU on CH340 lines; board 5 V too weak -> external regulator (docs/research/esp3d-wiring-reports.md). |
| Full V8 search (GitHub, Klipper, ChiTu repo, shops) | read | No public V8 schematic; ChiTu never answered a docs request (docs/research/v8-search-summary.md). |
| **tronxy3d/F103_PIC480x320 (Tronxy official Marlin)** | read | Manufacturer pin map for F103 boards; WiFi PB10/PB11, power-loss PG2/PG4, I2C EEPROM PG0/PG1. Newer bootloader (0x10000) - build NOT for us (docs/research/tronxy-official-marlin-f103.md). |
| forum.drucktipps3d.de thread 7796 (16 pages) | read | see docs/research/drucktipps3d-thread.md |
| tronxy.fandom.com "TronXY Mainboard Documentation" | **blocked** (Cloudflare challenge); text copied by owner | Section "TronXY CXY-V8-20190729": "The V8 is the third of Tronxys modern mainboards. It is commonly found on X5SA/D01/XY-2Pro printers and can be configured in Marlin as a V5 due to it's similarities. Some models were manufactured in 2019." No pinout in the text; pinout files (if any) still to be fetched by owner. |

## Notes from hackaday 167594 (relevant to us)
- Chitu bootloader decrypts `update.cbd`; Marlin's `chitu_f103` env produces that file (merged in Marlin 2020).
- **The bootloader stores the file key in the I2C EEPROM, so flashing the exact same
  `update.cbd` again may be silently ignored.** If a re-flash "does nothing", rebuild
  (new key) instead of assuming the board is broken. (Author's observation, unverified by us.)
- Bootloader leaves hardware half-initialised; Marlin has custom init code for this
  (`chitu_v5_gpio_init` / CHITU_V5_Z_MIN_BUGFIX env exists for related issues).
- Files: `chitucrypt.py` (encrypt/decrypt update.cbd), `chitutronxybootloader.bin`
  (bootloader dump; **never** to be written by us, rule: no writes to 0x08000000).
  `chitucrypt.py` could later turn a stock Chitu firmware back into an update.cbd for recovery.
