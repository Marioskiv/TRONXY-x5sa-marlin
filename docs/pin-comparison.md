<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Pin comparison: our Klipper printer.cfg vs Marlin BOARD_CHITU3D_V5

Sources: `backup/klipper/pasted/printer.cfg` (works on this board = verified),
Marlin bugfix-2.1.x `pins_CHITU3D_V5.h` + `pins_CHITU3D_common.h` (commit c49ded2, 2026-11-08),
Tronxy official `pins_tronxy_stm32f103.h` (cross-check).

| Function | Klipper (ours) | Marlin V5 | Match |
|---|---|---|---|
| X step / dir / enable | PE5 / !PE6 / !PC13 | PE5 / PE6 / PC13 | yes (inversions = config) |
| Y step / dir / enable | PE2 / !PE3 / !PE4 | PE2 / PE3 / PE4 | yes |
| Z step / dir / enable | PB9 / PE0 / !PE1 | PB9 / PE0 / PE1 | yes |
| E0 step / dir / enable | PB4 / !PB5 / !PB8 | PB4 / PB5 / PB8 | yes |
| X endstop | !PG10 | X_STOP PG10 | yes |
| Y endstop | !PA12 | Y_STOP PA12 | yes |
| Z probe | !PA14 (no Klipper pull-up) | Z_STOP PA14 (V5) | yes |
| Hotend thermistor | PA1, ATC Semitec 104GT-2 | TEMP_0 PA1 | yes -> Marlin type **5** |
| Bed thermistor | PA0, EPCOS 100K B57560G104F | TEMP_BED PA0 | yes -> Marlin type **1** |
| Hotend heater | PG12 | HEATER_0 PG12 | yes |
| Bed heater | PG11 | HEATER_BED PG11 | yes |
| Part fan | PG13 | FAN0 PG13 | yes |
| Hotend fan | PG14 | FAN2 PG14 | yes -> E0_AUTO_FAN_PIN PG14 |
| Controller fan | PD6 | CONTROLLER_FAN PD6 | yes |
| Runout sensor | !PA15 | FIL_RUNOUT PA15 | yes (inverted) |
| Beeper | PB0 | BEEPER PB0 | yes |
| MCU serial | USART1 (CH340), 250000 | SERIAL_PORT 1 | yes |

**Result: no pin differs.** Our V8 behaves exactly like Marlin's BOARD_CHITU3D_V5.
The only V5/V6 difference that matters (probe PA14 vs PG9) is on the V5 side for us.

Notes:
- Owner's earlier note said `^!PA14`; the working cfg has `!PA14` (no pull-up). The board has its own
  pull-up on that input, so both work. Marlin: probe inverted; pull-up optional.
- Klipper frees PA14 from SWD automatically; Marlin needs `DISABLE_DEBUG` (see research/v8-z-endstop-pa14-swd.md).
- Direction inversions (`!`) map to Marlin INVERT_*_DIR, but on CoreXY the final signs are verified after
  flashing with small G-code moves.

## Values carried over to Marlin
| Item | Klipper | Marlin |
|---|---|---|
| Kinematics | corexy | COREXY |
| Steps/mm X, Y | 32 usteps, rd 40 | **160, 160** |
| Steps/mm Z | 32 usteps, rd 8 | **800** |
| Steps/mm E | 32 usteps, rd 22.478, gear 66:22 | **854.17** (Titan clone) |
| Bed / travel | X -1..350 (mesh), Y 0..330, Z -2..400 | X/Y bed 330, Z_MAX 400 |
| Max velocity / accel | 300 mm/s / 3000 | 300 / 3000 (start) |
| Z velocity / accel | 25 / 30 | 25 / 30 |
| Probe offset | X -35, Y 0, z_offset 1, 3 samples | NOZZLE_TO_PROBE_OFFSET {-35, 0, -1} (Z re-measured) |
| Safe home | 165,165 | Z_SAFE_HOMING 165,165 |
| Mesh | 5x5, 20..310 | AUTO_BED_LEVELING_BILINEAR 5x5 |
| Bed screws | 6 screws | LCD_BED_TRAMMING (6 points) |
| Hotend PID | 21.789 / 1.045 / 113.578 | starting values, re-tune with M303 |
| Bed PID | 68.017 / 1.354 / 854.466 | starting values, re-tune with M303 |
| Max temp | hotend 260, bed 130 | HEATER_0_MAXTEMP 275 (Marlin adds margin), BED 140 |
| Extrude-only | 350 mm | EXTRUDE_MAXLENGTH 400 |

Note on PID units: Klipper and Marlin use different PID scaling; values above are only a hint.
Marlin will be tuned with M303 after flashing.
