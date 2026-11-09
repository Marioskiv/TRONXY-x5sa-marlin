<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Tronxy X5SA Pro: Klipper -> upstream Marlin

Workspace for moving a Tronxy X5SA Pro (CoreXY, board CXY-V8-190729) from Klipper
to upstream Marlin bugfix-2.1.x running entirely on the stock board.

Note: Marlin itself is GPL-3.0. The MIT license here covers only the files
authored in this repository (notes, scripts, config deltas), not Marlin sources.

## Layout
- `backup/klipper/` – verified working Klipper setup (pin reference + rollback path)
- `docs/facts.md`   – verified facts vs. assumptions, kept up to date

## Safety rules
- Owner decision 2026-11-09: the former rule "no stm32flash / no writes to 0x08000000" is dropped.
  Choose the easiest flashing method (USB + BOOT jumper, stm32flash, or `update.cbd` on SD).
- Firmware is installed only by the owner; Claude explains the steps first.

## Work order
1. Back up Klipper files  <- current
2. Fetch Marlin + Configurations, verify V8 vs `pins_CHITU3D_V5.h`
3. Configure Marlin
4. Identify WiFi socket UART -> SERIAL_PORT_2 for ESP3D
5. Build `chitu_f103` -> `update.cbd`
6. Post-flash checks (M115, M119, M503, M401/M48, M303, M500)
