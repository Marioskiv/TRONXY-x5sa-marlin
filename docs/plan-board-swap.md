<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Option: full mainboard swap (researched 2026-11-11, nothing bought, nothing verified on our printer)

Owner goals: stable, no USB cable to a PC, starts with the mains switch, likes Klipper + KlipperScreen, low cost.

## Candidate A (recommended): BTT Manta M5P + CB1 (Klipper all-in-one)
- MCU board with a socket for a compute module (CB1 / CB2 / CM4): the Klipper host sits **on the board**,
  no external PC, no USB cable; boots when the printer is switched on. 5 driver sockets (we need 4: X, Y, Z, E).
- Retail listings Nov 2026: board 45-65 USD (Tiny Machines 45.50, Fabreeko 48.99, West3D 52.99, 3DJake 65.13);
  3DJake UK bundle board + CB1 101.50 GBP. Drivers (TMC2209 UART) and CB1 are separate items.
  CB2 price not found. MKS SKIPR (similar all-in-one): no price/stock data found.
- TMC2209 in UART mode removes the standalone-driver limits (pressure advance, current from config).
- Screen: HDMI/DSI touchscreen for KlipperScreen, or phone/tablet with Mainsail. Original Tronxy screen is not usable.

## Candidate B (cheapest, stand-alone Marlin): board with integrated TMC2209 UART drivers (e.g. BTT SKR Mini E3 V3)
- Not price-checked. Needs its own display (e.g. BTT TFT35, which has an ESP WiFi socket). No Klipper.

## The hard part: wiring
- X5SA Pro uses a **30-pin ribbon** from the mainboard to the toolhead (hotend heater, fans, X motor, E motor,
  probe, runout, X stop, thermistor). No ready-made adapter for SKR/Manta found.
- Community routes (Thingiverse forum 45347, thing 4591046, Tronxy wiki "Upgrading your Mainboard"):
  build/buy a 30-pin breakout near the board, or mount the new board where the old breakout was, or run new wires.
- Ribbon pinout is documented for the **V6** in Marlin `pins_CHITU3D_V6.h` (comment table); must be checked
  on our V8 with a multimeter before use.
- Owner already plans a K2 Plus hotend with external MOSFET and a new extruder -> the toolhead wiring is being
  redone anyway, so new wires (or a CAN/USB toolhead board later) fit that work.
- Probe/Z-min input carries 24 V sensor levels on the stock board; check the new board's input limits.
