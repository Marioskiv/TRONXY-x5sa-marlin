<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Klipper on the Tronxy: leveling, tuning order, extras (researched 2026-11-13)

Sources: Klipper docs (Manual_Level.md, Bed_Mesh.md, Config_Reference.md, TMC_Drivers.md, Resonance_Compensation.md,
Pressure_Advance.md) and Moonraker docs (configuration.md, [power]), read from GitHub master today.

## 1. Bed screws with the probe: SCREWS_TILT_CALCULATE
Klipper's equivalent of Tronxy's leveling menu, but measured by the probe: it prints how far to turn each screw
("01:20" = 1 turn 20 min). Coordinates are **nozzle positions so that the probe is above the screw** (or as close as
possible). Screw 1 is the reference.

Template (from the old `[bed_screws]` positions + `x_offset: -35`; **screw positions, count and thread must be
checked on the real bed**; right-hand screws limited by X max 350 -> probe 10 mm short of them):
```
[screws_tilt_adjust]
screw1: 40, 5
screw1_name: front left
screw2: 200, 5
screw2_name: front middle
screw3: 350, 5
screw3_name: front right
screw4: 350, 325
screw4_name: rear right
screw5: 200, 325
screw5_name: rear middle
screw6: 40, 325
screw6_name: rear left
horizontal_move_z: 10
speed: 100
screw_thread: CW-M3   # check: M3/M4/M5 and turning direction
```
Owner (2026-11-13): bed has **6 screws, 3 front + 3 rear** (matches the template layout; exact X/Y and thread still to measure).
With 6 screws: set the 4 corners first, then the 2 middle screws (the plate can bow if the middles are too tight).
Use: hot bed, `G28`, `SCREWS_TILT_CALCULATE`, turn screws, repeat until all within ~00:03. Then `BED_MESH_CALIBRATE`.

## 2. Adaptive mesh (built into Klipper, no KAMP needed)
- `BED_MESH_CALIBRATE ADAPTIVE=1 ADAPTIVE_MARGIN=5` in the start macro; needs `[exclude_object]` in printer.cfg
  and "Label objects" enabled in the slicer. Docs: best when the full-bed variance is <= 1 layer height
  (so level the screws first). Adaptive meshes are per print, not reused.
- Check Klipper version on the PC first (`cd ~/klipper && git describe`); adaptive meshing exists in current master.

## 3. Tuning order
1. Mechanics: belt tension, rails, frame; driver current is set by the **potentiometers** (standalone TMC, not in Klipper).
2. Extruder: PID, direction, rotation_distance (see research/k2-hgx-extruder-review.md).
3. Probe: z_offset (hot bed), PROBE_ACCURACY.
4. Screws (section 1), then mesh.
5. Pressure advance (Klipper TUNING_TOWER method). Klipper docs: many use stealthChop with Klipper's **smooth**
   pressure advance (no instantaneous velocity changes); stealthChop may give less torque / more heat.
   -> The Marlin LA problem does not apply the same way here; still watch for E skips.
6. Input shaper without accelerometer: ringing tower test (Resonance_Compensation.md), then max_accel.
7. Optional: skew correction, firmware retraction.

## 4. Power loss recovery (Klipper has none built in; community plugins, none tested on this printer)
- mikeoverbay/klipper-power-loss-recovery: saves almost nothing during print (uses klippy.log sd_pos + small json),
  resume file + Mainsail button; tested on one Ender 3 Pro, Klipper v0.13. Needs lead-screw Z (ok), **warns against
  using the bed probe as Z endstop with safe_z_home at bed centre** (our setup) -> Z must home outside the part.
- bigtreetech/KlipperPLR (fork of YUMI_PLR): logs Z each layer via slicer G-code (`LOG_Z`), `RESUME_INTERRUPTED`;
  "no guarantee that it will work in 100% of cases"; Z must not move after the power cut.
- ankurv2k6/klipper-plr, T9Air/Klipper_Power_Resume, Royal3DP-Klipper-PLR: similar ideas.
- Prefer the **same plugin the owner already uses on the Kobra Plus** (ask for its config).
- X5SA bed is heavy and may sink when unpowered -> resume needs a Z re-home outside the part.

## 5. Power off after print
- The X5SA has no PSU relay -> needs a switchable device: smart plug (Shelly, Tasmota, Home Assistant, TP-Link
  [local API may be blocked]) or a relay. Moonraker `[power printer]` (`restart_klipper_when_powered: True`,
  `locked_while_printing: True`).
- Klipper macro from Moonraker docs: `POWER_OFF_PRINTER` with `action_call_remote_method("set_device_power",
  device="printer", state="off")`, triggered by `[delayed_gcode]` from `idle_timeout`, or at print end after the
  hotend has cooled (e.g. `TEMPERATURE_WAIT SENSOR=extruder MAXIMUM=50`).
- The laptop host stays on (battery), so Klipper reconnects when the printer is powered again.
