<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# GitHub search: community Marlin configs mentioning the V8-190729

## Search (2026-11-09, GitHub code search)
- "CXY-V8-190729": 0 hits. "CXY-V8": none relevant.
- "190729 tronxy": 3 hits, all copies of the same community config ("Marlin-tronxy", the rhapsodyv-era
  multi-machine config): leofauree/3D-Printer-Configurations (x5sa_pro/Marlin/Marlin-tronxy),
  BiOs2k/TRONXY_X5SA_PRO.
- Other repos: Darkwulf183/Marlin-2.1.2.9-ALL-UI-... (X5SA/XY-2/XY-3, Chitu V5/V6/V9), impet14 (V6),
  Siva1432 (V6, rhapsodyv fork), marcellodum/Tronxy-XY-3-Pro-V2.

## What the community config says (NOT verified on our board)
- `D01_V8_220_TITAN_TMC_NO_ABL`: "The V8-190729 is similar to the V5" -> `BOARD_CHITU3D_V5`, `WITH_TMC 1`.
  -> so V8 boards are the "TMC" ("mute") variant.
- `WITH_TMC` -> all drivers `TMC2208_STANDALONE` (no UART, fixed microstepping).
- Steps/mm:
  | Variant | X | Y | Z | E |
  |---|---|---|---|---|
  | TMC (mute)          | 160 | 160 | 800  | 415 (Titan) / 186 (stock) / 830 (BMG) |
  | non-TMC, T8 screw   | 80  | 80  | 400  | 420 (Titan) / 93 (stock) |
  | non-TMC, T2 screw   | 80  | 80  | 1600 | 420 / 93 |
  160 = 2x80 -> TMC boards run **32 microsteps** (hardwired), vs 16 on the old boards.
  This matches the forum note "V5 non-mute + T2 / V8 mute + T8": both give Z 1600 vs 800 steps/mm.
- `SERIAL_PORT 1`, `SERIAL_PORT_2 3`, `BAUDRATE 115200`.
- Endstops inverted (true) for X/Y/Z min and probe; direction inversion depends on model
  (X5SA: X/Y inverted, Z not).

## How we use this
Only as a cross-check. Real values come from the working Klipper printer.cfg
(`microsteps`, `rotation_distance`, `endstop_pin` polarity, `dir_pin` polarity) and a photo of the drivers.
