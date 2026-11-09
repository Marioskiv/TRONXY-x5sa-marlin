<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# External sources reviewed

| Source | Status | What it gives us |
|---|---|---|
| Marlin `pins_CHITU3D_V6.h` -> `x5sa-main_board-2.pdf` (Altium, 2019-06-03) | read | Full MCU schematic; WiFi J1 = USART3 PB10/PB11 (see wifi-socket.md). Revision not printed. |
| hackaday.io project 167594 "Chitu 3D Printer Mainboard Hacking" (J.C. Nelson) | read | Bootloader/update.cbd encryption; author of Marlin `chitu_f103` env. No pinout for V8. |
| ESP3D discussion #706 | read | Users report 5 V on CH_PD and GPIO0 grounded when an ESP-01 is plugged into the socket. |
| tronxy.fandom.com "TronXY Mainboard Documentation" | **blocked** (Cloudflare challenge) | Said to host per-board pinout PDFs. Owner must save the page/PDFs manually. |

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
