<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Bring-up checklist (Klipper on the laptop, K2 Plus hotend, HGX extruder, magnetic PEI) - 2026-11-13

A. Extruder / hotend (hotend heater only through the external MOSFET module)
1. printer.cfg [extruder]: replace `rotation_distance: 22.678` + `gear_ratio: 9.54647:1` with `rotation_distance: 5.694`. RESTART.
2. Heat to ~200 C: smooth rise, no jumps.
3. `PID_CALIBRATE HEATER=extruder TARGET=250`, `SAVE_CONFIG`.
4. Hot: `M83`, `G1 E10 F60` -> filament goes in; else drop `!` from dir_pin.
5. Hot: mark 120 mm, `M83`, `G1 E100 F60`, measure; new rotation_distance = 5.694 * actual / 100.

B. Z mechanics (both Z motors on one driver)
6. Printer OFF: turn one lead screw until the bed is parallel to the frame on both sides (fixture: Thingiverse 4297432).

C. Probe and bed (sensor set to trigger with nozzle ~2 mm above the steel PEI)
7. Add klipper-config/screws_tilt_adjust.cfg content to printer.cfg. RESTART.
8. Heat the bed to print temperature, wait a few minutes.
9. `G28`, `PROBE_CALIBRATE`, paper test, `ACCEPT`, `SAVE_CONFIG`.
10. `G28`, `SCREWS_TILT_CALCULATE` (first time: check the probe stops over each screw); turn corners first, then middles;
    repeat until all ~00:03.
11. `PROBE_ACCURACY` at the centre and at `G1 X300 Y300 F6000`; range ideally < ~0.02 mm.
12. `BED_MESH_CALIBRATE`, `SAVE_CONFIG`.

D. Later
13. `[exclude_object]` + slicer "Label objects" + `BED_MESH_CALIBRATE ADAPTIVE=1 ADAPTIVE_MARGIN=5` in the start macro.
14. Pressure advance (TUNING_TOWER), input shaper (ringing tower), max_accel.
15. PLR and power-off: copy from the Kobra Plus config (owner to share), smart plug for the X5SA.
