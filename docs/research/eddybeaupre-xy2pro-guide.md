<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# EddyBeaupre/Tronxy-XY2-Pro guide

## Source
https://github.com/EddyBeaupre/Tronxy-XY2-Pro (BSD-3, last commit fb9b8c8, 2022-02-18). Read 2026-11-09.
Guide for a **Tronxy XY-2 Pro with board CXY-V6-191017** -> **different board from ours (V8-190729)**.
It uses a forked Marlin config (`XY2_V6_255_NO_TITAN_TMC`), not upstream Configurations.

## Useful points
1. Warning in the guide: "newer Tronxy firmware doesn't read the update.cbd file". Their workaround is
   flashing at 0x08000000 over USB with the STM32 ROM bootloader. **Not allowed by our rules.**
   For us: Klipper is already installed and updated via update.cbd on this board (verified by owner),
   so our bootloader accepts update.cbd. To confirm: Klipper was installed via update.cbd, not by USB flashing.
2. On the V6, the board jumpers are: **BOOT** (normally fitted) and a **5V / USB** power jumper
   (normally on 5V). Removing BOOT + USB position = STM32 ROM bootloader mode. **Do not touch these.**
   The V8 likely has similar jumpers (JP1 BOOT next to USB per Marlin PR #15493 comments) -> photo needed.
3. Their "firmware backup" = full flash **read** (bootloader + app) with ST Flash Loader Demonstrator in
   ROM bootloader mode. It is a read, not a write, but it needs BOOT jumper changes and is the same tool
   class as stm32flash. Out of our rules unless the owner decides otherwise. Today it would also capture
   Klipper, not stock Chitu.
4. Stock-firmware settings backup trick: `M6046` + `M8512 "currentconfig.gcode"` (Chitu firmware only,
   useless while Klipper runs).
5. Build/install for Marlin is the same as ours: `pio run -e chitu_f103` -> copy `update.cbd` to SD root.
   They switch from `chitu_v5_gpio_init` to `chitu_f103` (their V6 board has the probe on PG9, so the
   PA14 fix does not apply to them).
6. They enable `TFT_COLOR_UI`; they raise `EXTRUDE_MAXLENGTH` to 500 (Bowden load/unload ~325 mm).
   Practical hint for us: Bowden filament change length.

## Not useful / not applicable
- Board image is CXY-V6-191017. Pinout differences V6 vs V8 (probe PG9 vs PA14, Z2 on PF1/PF3/PF5).
