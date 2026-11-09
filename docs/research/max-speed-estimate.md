<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# How fast can Marlin drive this printer? (estimate, 2026-11-09)

## Method
Marlin's own ISR cost model, `Marlin/src/module/stepper/cycles.h` (bugfix-2.1.x): 32-bit values
(base ISR 770 cycles, 100 cycles per stepper, input shaping +180 base, LA +64), F_CPU = 72 MHz,
4 logical axes (XYZE), min pulse ~100 ns. Marlin says these numbers are "rough estimates" and were
measured on an LPC1768; an STM32F103 with flash wait states is probably **slower**, so treat as an upper bound.

## Max step rate per motor (theoretical ceiling)
| Build | 1x (smoothest) | 2x | 4x | 8x | 128x multi-stepping |
|---|---|---|---|---|---|
| No input shaping | 61 k/s | 91 k | 120 k | 142 k | 173 k |
| Input shaping X+Y | 46 k/s | 66 k | 84 k | 98 k | 115 k |
| Input shaping + Linear Advance | 41 k/s | 61 k | 80 k | 95 k | 115 k |

## What the printer needs (160 steps/mm, CoreXY, 32 microsteps fixed in hardware)
| Speed | Pure X or Y move (per motor) | 45 deg move (worst motor) |
|---|---|---|
| 100 mm/s | 16 k/s | 23 k/s |
| 150 mm/s | 24 k/s | 34 k/s |
| 200 mm/s | 32 k/s | 45 k/s |
| 250 mm/s | 40 k/s | 57 k/s |
| 300 mm/s | 48 k/s | 68 k/s |

## Reading
- Up to ~150 mm/s: inside the smoothest (1x) range even with input shaping. **Comfortable.**
- ~200 mm/s: at the 1x limit with input shaping; Marlin switches to 2x multi-stepping. Fine for travel.
- 250-300 mm/s: needs 2x-4x multi-stepping (coarser timing) and the real F103 may be slower than the model.
  Possible for **travel moves**, to be tested; not a target for print moves.
- The display (FSMC), touch, SD and the ESP3D serial also use CPU time.
- Other real limits: hotend melt rate (mm3/s), acceleration/ringing, belts/frame.
- Klipper was faster here because the PC does the motion planning.

## Plan
Start with print 120-150 mm/s, travel 200 mm/s, accel 3000. Then raise step by step with test moves
(listen for stutter, check for layer shifts) and record the safe limits in this file.
