<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Why Linear Advance can make the extruder skip on this board

## Facts
- Drivers: TMC2225/TMC2208 in **standalone** mode (no UART). Their mode is fixed by how the chip pins are
  wired on the board; standalone default is **stealthChop** (quiet mode). Marlin cannot change it.
- Forum (drucktipps3d #41, 2020, V6 board, TMC2208): with LIN_ADVANCE, a few good parts, then layer shifts
  and the extruder stopped after 5-15 mm. Darkwulf183 config: LIN_ADVANCE "Board conversion ... is mandatory".
- Marlin bugfix-2.1.x has **SMOOTH_LIN_ADVANCE** (added 2025-04, PR #27710; S-curve support 2025-06):
  "Remove limits on acceleration by gradual increase of nozzle pressure", smoothing time ADVANCE_TAU
  (default 0.02 s), update rate SMOOTH_LIN_ADV_HZ 1000. Only requirement: 32-bit CPU. The 2020-2022 forum
  reports were made **before** this existed.

## Mechanism (general TMC behaviour)
- Classic LA adds/removes extruder speed in steps at every acceleration change -> the E motor gets very
  sudden speed jumps and short reversals.
- stealthChop regulates current indirectly (voltage PWM, auto-tuned to the motor). It is made for smooth,
  slow changes. On sudden jumps the current lags behind -> torque drops -> the motor skips steps or stalls.
  spreadCycle (true current control) handles such jumps; Trinamic recommends it for dynamic motion.
- Klipper's pressure advance smooths the E speed (pressure_advance_smooth_time, default 0.040 s), which is
  why it is gentler on stealthChop drivers.

## Options for us (in order)
1. **SMOOTH_LIN_ADVANCE** (firmware only): same idea as Klipper's smoothing. Try first; tune ADVANCE_TAU
   up if the extruder skips. CPU cost on the F103 to be checked in the step-rate budget.
2. Lower E acceleration / jerk limits.
3. Hardware: switch the E driver to spreadCycle (solder change) or the UART mod (Arakon/korrado, drucktipps3d #234).
4. Leave LA off.

New extruder (HGX, 9.54647:1) means many E steps/mm at fixed 32 microsteps -> higher E step rates; include
in the test.
