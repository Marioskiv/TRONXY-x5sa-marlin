<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# K2 Plus hotend on the Tronxy (research, 2026-11-09, NOT verified)

Owner: hotend is from a Creality K2 Plus; flow is not the limiting factor.

## Found online
- sw3defy K2 Plus wiki (boards layout): hotend thermistor on the K2 toolhead board `nozzle_mcu:PA0`,
  configured as a **custom thermistor** in Creality's Klipper config (exact table not found yet).
  -> Marlin: either a standard type if it matches, or `TEMP_SENSOR_0 1000` (custom, Steinhart-Hart /
  beta values in Configuration_adv.h). Needs the K2 stock printer.cfg values or a resistance measurement.
- Creality forum: K2 Plus hotend uses a **100 W ceramic heater**. Retailer: printing up to **300 C**.

## SAFETY check needed before first heat-up
- 100 W at 24 V = **~4.2 A** on the hotend output (PG12 MOSFET, board connector, cable, fuse).
  The stock Tronxy heater is much smaller. On the V6 board the hotend current goes through the 30-pin
  ribbon (4 pins hotend + 4 pins +24 V, Marlin pins_CHITU3D_V6.h). Ribbon pins are typically ~1 A each.
  -> Need: heater voltage/power label, how it is wired (ribbon or separate cable), board fuse rating.
- Max temp in Marlin set from the real hotend limit (300 C -> HEATER_0_MAXTEMP ~315), and thermal
  runaway protection stays ON.
