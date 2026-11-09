<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Display / UI plan (original Tronxy touchscreen)

Language: **English** (owner's choice; no Greek font -> saves flash).

## Hardware (from source, 2026-11-09)
- Marlin option `TFT_TRONXY_X5SA` = "480x320, 3.5", FSMC Stock Display from Tronxy" -> ILI9488, 480x320
  (Conditionals-2-LCD.h). Tronxy's own pin file also says ILI9488 480x320.
- FSMC: CS PD7, RS PD11, reset PF11, backlight PD13. Touch XPT2046 on SPI1 (PB7/PA5/PA6/PA7).
- Tronxy touch calibration (starting point): X -17181, Y 11434, X offset 501, Y offset -9.
  Marlin also has on-screen `TOUCH_SCREEN_CALIBRATION` (+ auto-save to EEPROM).
- Proven on a V8: Color UI and Classic UI worked (Marlin #19320). LVGL needs SD assets and is reported buggy on Chitu.

## Option A (baseline, proven): TFT_COLOR_UI
- Font NOTOSANS (anti-aliased) or HELVETICA (smaller flash). Themes: BLUE_MARLIN, BLACK_MARLIN, ANET_BLACK,
  or **our own theme** (`themes/theme_CUSTOM.h` style: background/accent/hot/cold colours).
- Own boot screen and status logo possible (SHOW_CUSTOM_BOOTSCREEN / CUSTOM_STATUS_SCREEN_IMAGE).

## Option B (experimental, nicer): NOVIN3DP_GALAXY_UI
- Merged into bugfix-2.1.x on **2026-11-05** (PR #28702): home tiles, print screen with G-code
  thumbnails (QOI 48x48 + 150x150 from the slicer), file grid, round XY jog pad, heater/fan sliders,
  menus restyled. Strings are English.
- Tested only on STM32F407 + SPI display (MKS TS35). **Never tested on STM32F103 / FSMC / Chitu.**
  Icon data is large in source (~600 KB of .cpp) -> flash size on our 512 KB chip (minus 34 KB bootloader
  and 4 KB EEPROM) must be checked by a test build. Plan: build A first, then try B as a separate build;
  going back is just another update.cbd.

## "Helpful" features (all in Marlin, chosen for this printer)
| Feature | Option |
|---|---|
| Own main-menu buttons (macros: load/unload filament, preheat, level, PID, park, etc.) | CUSTOM_MENU_MAIN |
| Filament change / load / unload wizard | ADVANCED_PAUSE_FEATURE, FILAMENT_LOAD_UNLOAD_GCODES (Bowden ~325 mm) |
| Runout sensor -> pause + wizard | FILAMENT_RUNOUT_SENSOR (PA15) |
| Bed tramming help (screws) | LCD_BED_TRAMMING, ASSISTED_TRAMMING_WIZARD (uses the probe) |
| Probe Z-offset wizard + live baby-stepping of Z-offset | PROBE_OFFSET_WIZARD, BABYSTEP_ZPROBE_OFFSET |
| Bed mesh from the screen + edit points | LCD_BED_LEVELING, MESH_EDIT_MENU |
| PID autotune from the screen | PID_AUTOTUNE_MENU |
| Input shaping frequencies from the screen | SHAPING_MENU |
| Time remaining / progress from slicer (M73) | SET_PROGRESS_MANUALLY, SET_REMAINING_TIME, SHOW_REMAINING_TIME |
| Preheat shortcuts | PREHEAT_1..n, PREHEAT_SHORTCUT_MENU_ITEM |
| Screen sleep with adjustable timeout | DISPLAY_SLEEP_MINUTES + EDITABLE_DISPLAY_TIMEOUT |
| Mute button | SOUND_MENU_ITEM |
| Info screen (firmware, stats) | LCD_INFO_MENU, PRINTCOUNTER |
| Messages from ESP3D/slicer shown on screen | M117 (default), STATUS_MESSAGE_SCROLLING |
All of this costs flash; final list is decided by the test build size.

## ESP3D side (research)
- ESP3D 3.1 (luc-github/ESP3D, branch 3.1, updated 2026-11-05): supports ESP8266/ESP8285 and ESP32
  (C3/S2/S3/C6). `DEFAULT_FW MARLIN`, `COMMUNICATION_PROTOCOL RAW_SERIAL`. Web UI = ESP3D-WEBUI 3.1.
- ESP8266 prints boot messages on its serial at start (garbage on USART3 for a moment; harmless).
  An ESP32-C3 board is a good alternative (more RAM, still 3.3 V).
- Marlin host features to enable for a good web UI (standard): EMERGENCY_PARSER, AUTO_REPORT_TEMPERATURES,
  AUTO_REPORT_POSITION, HOST_KEEPALIVE_FEATURE, EXTENDED_CAPABILITIES_REPORT, LONG_FILENAME_HOST_SUPPORT,
  AUTO_REPORT_SD_STATUS, HOST_ACTION_COMMANDS. (To confirm against ESP3D-WEBUI docs at build time.)
