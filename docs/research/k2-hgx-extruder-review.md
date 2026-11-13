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
