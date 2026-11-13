<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Review of the K2 Plus hotend + HGX extruder Klipper section (2026-11-13)

Config: `backup/klipper/pasted/extruder-k2-hgx.cfg`. Owner continues with Klipper on the PC/laptop.

## Numbers derived from the config
- E steps/mm = 200 * 32 * 9.54647 / 22.678 = **2694** (assumes a 1.8 deg motor).
- Step rate: 10 mm/s filament = 27 k steps/s; 40 mm/s retraction = **108 k steps/s**; 60 mm/s = 162 k steps/s.
- Thermistor points give beta 4422 (25-220 C) and 4773 (220-350 C): plausible NTC shape.
  With the board's 4.7 k pull-up: 25 C -> ADC ~4022/4095, 220 C -> ~556, 350 C -> ~84 (coarse at the top end).

## Findings
1. `control: pid` without values: Klipper falls back to the **old SAVE_CONFIG PID** of the stock hotend
   (21.789 / 1.045 / 113.578) -> wrong for a 100 W heater. Run `PID_CALIBRATE HEATER=extruder TARGET=250`,
   then `SAVE_CONFIG`, before printing.
2. `max_temp: 360`: only one retailer source (300 C) found for this hotend; 350 C not confirmed. Set to the
   real rating + ~10 C until confirmed. Hotend heater runs through the external MOSFET module.
3. Thermistor table: source of the three points unknown -> check at room temperature (should read room temp)
   and at ~200 C against an external thermocouple.
4. High E step rate: watch `klippy.log` for `Timer too close` during fast retractions; keep retract speed
   moderate (e.g. 30-35 mm/s) until tested. Microsteps must stay 32 (fixed by the board hardware).
5. Calibrate `rotation_distance` (mark 120 mm, extrude 100 mm slowly when hot, new = old * actual / 100);
   leave `gear_ratio` as given.
6. Check extrusion direction with a short `G1 E10 F60` (invert `dir_pin` if wrong; new gearing may reverse it).
7. `max_extrude_only_distance: 100`: fine for direct drive; too short for a long Bowden load.
8. Hotend/part fans on PG14/PG13 switch the board's supply voltage (24 V); K2 fans must match.

## Cross-check against Creality's own Klipper configs (2026-11-13)
Source: CrealityOfficial/K2_Series_Klipper, `config/F008/printer.cfg`, `config/F008_CR0CN240319C13/printer.cfg`,
`config/F021_CR0CN200400C10/printer.cfg` (raw files read today).

### Thermistor
- F008 configs: `[thermistor my_thermistor]` 25 C / 260000, 220 C / 738, 350 C / 98 -> **identical to the owner's table**
  (so ChatGPT copied it from Creality; correct for that hotend version).
- **F021 config uses a different thermistor**: 25 C / 98633, 220 C / 377, 320 C / 73 (the 260 k table is commented out).
  -> Two K2 hotend thermistor versions exist. Test: measure the thermistor at room temperature with a multimeter
  (disconnected): ~260 k = F008 table, ~100 k = F021 table.
- Community guide (Tombraider2006/Ender5Max, `mans/k2_hotend.md`) uses the 260 k table for a K2 hotend.
- Creality reads it on the K2 toolhead board (nozzle_mcu PA0); no pullup_resistor override in the config, so the
  table is plain resistance and works with our board's 4.7 k pull-up.
- Creality max_temp: 370 (F008) / 390 (F008_CR0CN240319C13, F021). So max_temp 360 is within Creality's own limits.
- Creality PID values (e.g. 19.219 / 2.136 / 43.243) are for their board; `pid_*_high_temp` keys are Creality-fork only.
  Our setup (other board, external MOSFET) -> PID_CALIBRATE anyway.

### Extruder (HGX manual, photo docs/hardware/images/hgx-extruder-manual-parameters.jpg)
Manual: drive ratio 9.54647:1, **"Pluse" 562**, driving gear 10 T, reduction gear 44 T, extrusion wheel 37 T, 1.75 mm.
- Owner's config (rotation_distance 22.678 + gear_ratio 9.54647:1) = 1347 steps/mm at 16 microsteps,
  2694 at 32 -> **2.4x the manual's 562**. 22.678 is the generic Bondtech/Voron drive-gear value, not from this manual.
- Reading "562" as steps/mm at 16 microsteps with a 1.8 deg motor (assumption, the manual does not say):
  Klipper `rotation_distance: 5.694` **without** `gear_ratio` (same as 54.357 with gear_ratio 9.54647:1).
  At our 32 microsteps that is 1124 steps/mm; a 40 mm/s retraction = 45 k steps/s (instead of 108 k).
- Safer start: the manual's value. If it is wrong it under-extrudes (safe); the 22.678 value would over-extrude 2.4x
  if the manual is right (grinding/clog). Then calibrate with the 100 mm test.
- Creality's own K2 extruder (not the HGX): rotation_distance 6.9, microsteps 16, no gear_ratio.

## Owner result 2026-11-13: thermistor ~260 k -> F008 table confirmed.

## Next steps (to do, in this order; owner paused here)
1. Replace `rotation_distance: 22.678` + `gear_ratio: 9.54647:1` with `rotation_distance: 5.694` (no gear_ratio). RESTART.
2. First heat to ~200 C: smooth rise, no jumps; compare with an external thermometer if available.
3. `PID_CALIBRATE HEATER=extruder TARGET=250`, then `SAVE_CONFIG`.
4. Direction (hot): `M83` + `G1 E10 F60` -> filament must go in; else drop `!` from `dir_pin`.
5. Calibration (hot): mark 120 mm, `M83` + `G1 E100 F60`, measure; new = 5.694 * actual / 100.
