<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Tronxy leveling sensor: height and X/Y offset (researched 2026-11-13)

## Sensor facts found (retailer listings, not a Tronxy datasheet)
- **Blue sensor**: model XY-08M, inductive (detects **metal**), DC 6-36 V, Sn = **8 mm** (one listing: 8 mm +-15 %).
- **Black TR sensor**: detection distance 8 mm, "can sense all non-transparent objects", 35x19x19 mm, 120 cm cable.
- Owner's Klipper cfg comment: "Tronxy blue self-levelling plate sensor"; pin `!PA14`, `z_offset: 1`, `x_offset: -35`, `y_offset: 0`.
- No official mounting height above the nozzle found (Tronxy manual only says: set Z offset first, then level).

## Height above the nozzle tip (general rule, not a Tronxy spec)
- The sensor must trigger while the nozzle is still above the bed, and its body must sit above the nozzle tip so it
  does not drag on prints.
- Rated Sn (8 mm) is for steel. On aluminium an inductive sensor triggers closer (general correction factors
  roughly 0.3-0.5 -> about 2.5-4 mm), and the X5SA bed has glass on aluminium. So a fixed number is not reliable.
- Practical setting: nozzle touching paper at Z=0, raise to Z=2 (or 3), lower the sensor until its LED just turns on,
  tighten; check that the sensor body is >= ~1.5 mm above the nozzle tip. Then `PROBE_CALIBRATE` and
  `PROBE_ACCURACY` (repeatability) in Klipper.

## X/Y offset
- Not a fixed value: it depends on the new K2 hotend / HGX toolhead mount. Measure it.
- Stock references: Klipper template X5SA Pro 2020: x_offset -35, y 0; X5SA v6 2019: -40; owner's old cfg: -35, 0.
- Sign: `x_offset = probe X - nozzle X` (probe left of nozzle = negative), `y_offset` likewise (probe behind = positive).
- Method (Klipper docs, "Calibrating probe X and Y offsets"): mark a spot on the bed (tape + pen), put the nozzle
  exactly over it, note X/Y; move until the sensor centre is over the mark, note X/Y; offset = difference.
- After changing offsets, re-check `mesh_min`/`mesh_max` (probe coordinates) and `safe_z_home`.
  Smaller offsets = more of the bed can be probed.

## Owner measurement 2026-11-13
- Nozzle touching the bed, sensor lowered until its LED just lit: sensor face to bed = **~3.5 mm**.
  -> Effective trigger distance on this bed ~3.5 mm (fits the 2.5-4 mm estimate for aluminium).
- In that position the sensor triggers exactly when the nozzle touches -> z_offset ~0 -> **no safety margin**
  (any small variation and the nozzle hits the bed while probing).
- Fix: sensor face must sit LOWER relative to the nozzle tip: target 1.5-2 mm above the tip
  -> trigger with the nozzle 2.0-1.5 mm above the bed. Procedure: nozzle at Z=2 (or a 2 mm spacer), lower the sensor
  until the LED just lights, tighten. Then PROBE_CALIBRATE, PROBE_ACCURACY.
- Owner re-set the sensor with the nozzle at Z=2: LED lights -> sensor now ~1.5 mm above the nozzle tip.

## Planned bed change: PEI plate (owner, 2026-11-13)
- Inductive sensor rated distance (8 mm) refers to a steel target; non-ferrous metal (aluminium) gives less (~3.5 mm measured).
- **PEI on spring steel (magnetic)**: the steel sheet becomes the target -> sensor will trigger noticeably farther
  away than 3.5 mm (exact value unknown, thin sheet). Sensor height and z_offset must be set again.
- **PEI film on glass/aluminium (no steel)**: target stays the aluminium; only the surface height changes.
- Either way: after the plate is fitted, repeat the Z=2 sensor setting, `PROBE_CALIBRATE` with a hot bed,
  `PROBE_ACCURACY`, new bed mesh. A calibration done now on glass will not be valid afterwards.
