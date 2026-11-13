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

## Ready-made X5SA screws_tilt_adjust configs found on GitHub (2026-11-13; community, not official Tronxy)
- justheath/klipper_config `x5sa-config/printer.cfg`: 6 screws, CW-M3 (own head/offsets).
- dung-adumbs/Klipper_BU `printer_Gemini.cfg` (X5SA): 6 screws at 60/203/335 x 30/315, CW-M3, probe offset 0;
  its [bed_screws] = 5/165/325 x 5/325 (same as the official Klipper template).
- iamSiiN/JuggernautX5SA (X5SA-400, Manta M8P), marcin-polo/klipper_tronxy (400 bed, 4 screws): other sizes.
- All use **CW-M3**. Coordinates always depend on the probe offset -> our version: `klipper-config/screws_tilt_adjust.cfg`.

## Two Z motors (researched 2026-11-13, deeper search)
How the two Z sockets are wired depends on the board version:
| Board | Z wiring | Source |
|---|---|---|
| X5SA main board schematic (Altium 2019-06, A4982 drivers, non-"mute") | **one Z driver, two sockets JP14 + JP11 in parallel** (each output wire goes to both) | x5sa-main_board-2.pdf (Marlin link), rendered today |
| Tronxy D01 (same board family) | two Z sockets **in series** on one driver; with one motor, 2 jumpers needed in the 2nd socket | RepRap forum thread 878044 (search snippet) |
| **CXY-V8-190729 ("mute", 4 x TMC2225)** | "insert 2 jumpers on the Z2 connector" when one motor -> **series on one driver** | Thingiverse 4267636 note; GearBest listing (4 drivers) |
| Chitu V6 190319 / V9 200420 (X5SA-500, TMC2209) | **separate Z2 driver** PF5/PF1/PF3 -> Klipper `[stepper_z1]` possible | Darkwulf183 Klipper configs; drucktipps #89; Tronxy pin file |
Our printer.cfg has only `[stepper_z]` -> consistent with one driver.

Consequences for us (V8): both motors get the same steps; they cannot drift electrically, but can drift mechanically
(one side binds/skips, or a lead screw is turned by hand while unpowered). Klipper cannot correct it
(`Z_TILT_ADJUST` needs one driver per motor).

Options:
1. Align once mechanically (power OFF, never unplug motors while powered): turn one lead screw until the bed is parallel
   to the frame at both sides; printable helper: Thingiverse 4297432 "Tronxy X5SA Z Axis Alignment Fixture". Then
   SCREWS_TILT_CALCULATE + mesh.
2. Mechanical sync mod: link both lead screws with GT2 pulleys + closed belt (drucktipps forum user: "beide Z-Achsen
   mit GT2 60 Scheiben synchronisiert"); keeps them aligned even when unpowered.
3. Independent motors + `Z_TILT_ADJUST`: only with a 5th driver (other board, e.g. Manta M5P).
Owner 2026-11-13: E2 driver position is empty (not soldered) -> confirmed 4 drivers; Z + Z2 on one driver.
Populating the empty footprint (fine-pitch TMC chip + passives) is not recommended. Use options 1 or 2 (or 3 with a new board).

## First SCREWS_TILT_CALCULATE (owner, 2026-11-13, steel PEI, before any adjustment)
Probe Z: front left -1.089 (base), front middle -0.613, front right +0.098, rear right +0.109, rear middle +0.192,
rear left -0.363 -> spread **1.28 mm**. Left side ~0.8 mm lower than right; front-left corner lowest.
Klipper output (base = front left): turn the other 5 CCW 00:57, 02:22, 02:24, 02:34, 01:27 (~9.7 turns total).
Alternative with fewer turns: raise the low points to the highest (rear middle), M3 = 0.5 mm/turn:
| screw | Z | turn |
|---|---|---|
| front left | -1.089 | 2:34 CW |
| front middle | -0.613 | 1:37 CW |
| front right | +0.098 | 0:11 CW |
| rear right | +0.109 | 0:10 CW |
| rear middle | +0.192 | 0:00 CW |
| rear left | -0.363 | 1:07 CW |
First check the Z lead-screw alignment (left/right tilt can come from the two Z motors on one driver), and verify the
turning direction with a small turn + re-run before turning a lot.

## Second SCREWS_TILT_CALCULATE (2026-11-13)
FL -0.534 (base), FM -0.154, FR +0.026, RR -0.205, RM -0.130, RL -0.436 -> spread 0.56 mm (was 1.28 mm).
Fewest turns: bring all to the median (-0.179); CW = smaller nozzle gap (raise that point):
| screw | Z | turn |
|---|---|---|
| front left | -0.534 | CW 0:43 |
| front middle | -0.154 | CCW 0:03 |
| front right | +0.026 | CCW 0:25 |
| rear right | -0.205 | CW 0:03 |
| rear middle | -0.130 | CCW 0:06 |
| rear left | -0.436 | CW 0:31 |

## Third SCREWS_TILT_CALCULATE (2026-11-13)
FL -0.195 (base), FM -0.049, FR -0.043, RR -0.311, RM -0.218, RL -0.348 -> spread 0.31 mm (1.28 -> 0.56 -> 0.31).
Front ~0.2 mm higher than rear. One more round to the median (-0.207): FM CCW 0:19, FR CCW 0:20, RR CW 0:13,
RL CW 0:17; FL and RM leave. Stop when spread <= ~0.1 mm (or when it is close to the PROBE_ACCURACY range).

## Fourth SCREWS_TILT_CALCULATE (2026-11-13)
FL -0.195 (base), FM -0.108, FR -0.088, RR -0.232, RM -0.171, RL -0.296 -> spread 0.21 mm.
Optional last fine round (median -0.183): FM CCW 0:09, FR CCW 0:11, RL CW 0:14; then stop and mesh.

## Fifth SCREWS_TILT_CALCULATE (2026-11-13) - screws done
FL -0.193 (base), FM -0.106, FR -0.114, RR -0.229, RM -0.160, RL -0.270 -> spread 0.16 mm (1.28 -> 0.56 -> 0.31 -> 0.21 -> 0.16).
All corrections <= 00:10. Decision: stop adjusting screws; remaining shape is left to the bed mesh.
Next: PROBE_ACCURACY (range), BED_MESH_CALIBRATE, SAVE_CONFIG, heightmap photo.
