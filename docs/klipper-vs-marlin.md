<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# What we keep from Klipper in Marlin (bugfix-2.1.x, checked 2026-11-09)

| Klipper feature we use / want | Marlin equivalent | Status |
|---|---|---|
| Adaptive bed mesh (only the print area) | `G29 L<x0> R<x1> F<y0> B<y1>` (bilinear ABL; limits documented in G29.cpp), called from the slicer start G-code with the first-layer bounding box | YES |
| Mesh fade / reuse saved mesh | `M420 S1 Z<fade>`, RESTORE_LEVELING_AFTER_G28, mesh saved with M500 | YES |
| Probe + safe Z home | Z_SAFE_HOMING at 165,165 | YES |
| BED_SCREWS_ADJUST | LCD_BED_TRAMMING (6 points) + ASSISTED_TRAMMING (G35, uses the probe) | YES (better: on screen) |
| PAUSE / RESUME / CANCEL / M600 | ADVANCED_PAUSE_FEATURE (park, unload, load, purge) + runout sensor | YES |
| EXCLUDE_OBJECT | CANCEL_OBJECTS (M486), slicer must label objects | YES |
| SAVE_CONFIG | M500 (EEPROM) | YES |
| PID_CALIBRATE | M303 (+ menu) | YES |
| Input shaper (frequencies) | INPUT_SHAPING_X/Y, M593 | YES, but no accelerometer measurement -> tune with a ringing test print |
| Pressure advance | LIN_ADVANCE, M900 K | LIMITED: standalone TMC2208/2225 in stealthChop lose E steps (forum reports) -> needs driver mod or spreadCycle |
| Firmware retraction | FWRETRACT (G10/G11) | YES |
| Skew correction | SKEW_CORRECTION | YES |
| Macros with logic (Jinja) | GCODE_MACROS (M810-M819, fixed text) + CUSTOM_MENU_MAIN buttons | PARTIAL: no variables/conditions |
| Mainsail web UI | ESP3D web UI over WiFi | SIMPLER (terminal, files, temps, control) |
| File upload over network | ESP3D upload to SD via serial | SLOWER |
| Thumbnails on screen | Galaxy UI (QOI thumbnails) | EXPERIMENTAL on our MCU |
| Print history / statistics | PRINTCOUNTER | BASIC |
| Webcam, timelapse, Spoolman | - | NO |
| Accelerometer resonance test | - | NO |

## Things to watch
- **Step rate**: 32 microsteps are fixed in hardware -> X/Y 160 steps/mm. At 300 mm/s that is 48 k steps/s
  per motor. Marlin on a 72 MHz STM32F103 may not reach Klipper's speeds; we test real limits after flashing.
- **Adaptive mesh slicer line** (example, OrcaSlicer/PrusaSlicer placeholders):
  `G29 L{first_layer_print_min[0]} R{first_layer_print_max[0]} F{first_layer_print_min[1]} B{first_layer_print_max[1]}`
  (exact syntax checked against the slicer when we write the start G-code).
- **Before leaving Klipper (optional)**: if an ADXL345 is available, measure X/Y resonances once in Klipper
  and keep the numbers for Marlin's M593.
