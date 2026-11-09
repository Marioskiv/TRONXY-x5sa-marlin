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

## Owner decision (2026-11-09): external power for the hotend
Plan (to confirm with owner's parts):
- External MOSFET module (heater module), switched by the board's hotend output (PG12 / HEATER_0).
  Board then only drives the module input -> no 4 A through board/ribbon.
- Module must keep the heater **OFF when its input floats** (pull-down on input): during reset, USB
  flashing (ROM bootloader) or a crash, PG12 is an unconfigured input.
- Isolated (optocoupler) module, or common GND between PSUs as the module's datasheet says.
- Inline fuse on the external supply (~6-7 A for a 100 W/24 V heater), wire >= 0.75 mm2 (18 AWG).
- Supply voltage must match the heater label (check before connecting).
- Thermistor stays on the board (PA1). Marlin thermal protection works unchanged (it controls the module).

## Board hotend output (X5SA schematic, same family as our V8)
- Net HOT-END1 (MCU PG12) goes straight to the gate of **Q6 HY1403** (N-MOSFET, low-side switch),
  with **R23 10 k pull-down** gate->GND. Connector **JP6** (KF301 2-pin): pin 1 = supply (+), pin 2 = Q6 drain.
  -> When PG12 floats (reset, USB flashing, crash), R23 keeps Q6 **off**. So a module driven from JP6
  is off too, as long as the module itself does not turn on with an open input (normal for these modules).
- Bed: HOT-BED -> 8050 transistors -> Q5 RU30120L, R27 10 k pull-down. Part fan: Q7 AO3400A.
- Rail label in the schematic is "VCC_12V" = main input rail (24 V on this printer).

## Owner's candidate module (AliExpress, 2026-11-09)
"3D Printer Hot Bed Power Expansion Board Heating Controller MOSFET High Current Load Module 25A 30A 12or24V".
Listing not readable from here. Generic assessment:
- 25-30 A rating: far above 4.2 A -> OK.
- Wiring: module IN <- board JP6 (hotend output); module POWER IN <- external PSU (with fuse);
  module OUT -> heater.
- If the listing does **not** say optocoupler/isolated: connect external PSU V- to main PSU V- (common GND).
- Check on arrival: no heater power when the board is off / unplugged from the module input.

## Module chosen (listing screenshot from owner, 2026-11-09)
`docs/hardware/images/hotend-mosfet-module-listing.jpg` - HuiSheng DongYe (HSDY) "Heated Bed Power Module":
4 screw terminals (DC IN -, DC IN +, HOT BED -, HOT BED +), 2-pin input plug from the mainboard's heater
output, large MOSFET. Seller diagram powers it from the same PSU and drives it from the board's bed output.
For us: input plug -> board **hotend** output JP6; DC IN -> **external PSU** (with fuse); HOT BED +/- -> K2 heater.
Optocoupler on input: **not visible in the picture** -> check for a 4-pin chip marked 817 (PC817/EL817)
near the input plug. Decision: connect both PSUs' V- together in any case (safe with or without opto).
Input polarity: plug + to JP6 pin 1 (supply), - to JP6 pin 2 (switched).
