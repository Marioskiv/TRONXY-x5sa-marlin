<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Plan: Klipper-like runtime configuration, WiFi access, USB debugging

## 1. Change parameters without reflashing (Marlin EEPROM + G-code)
Marlin stores most tuning values in EEPROM (`EEPROM_SETTINGS`), changed with G-code and saved with `M500`.
These can be sent from the touchscreen, from ESP3D over WiFi, or over USB:

| Klipper | Marlin G-code (runtime, saved with M500) |
|---|---|
| rotation_distance / steps | `M92` steps/mm |
| max_velocity / max_accel | `M203`, `M201`, `M204` |
| square_corner_velocity | `M205 J` (junction deviation) |
| pressure_advance | `M900 K` (LIN_ADVANCE) |
| input_shaper | `M593 X/Y F D` (INPUT_SHAPING_X/Y) |
| PID_CALIBRATE | `M303` then `M301` / `M304` |
| probe offsets | `M851` |
| bed mesh | `G29`, `M420` |
| stepper direction/current | NOT runtime with standalone TMC2208 (hardware) |
| macros | `GCODE_MACROS` (M810-M819), `CUSTOM_MENU_MAIN` on the screen |
| GET config | `M503`, `CONFIGURATION_EMBEDDING` (M503 C) |
| pin debugging | `PINS_DEBUGGING` -> `M43` |

**Needs a rebuild** (compile-time in Marlin, unlike Klipper): pins, thermistor type, kinematics,
enabled features, driver types, UI. Strategy: one "maximal" build with all needed features
enabled, so day-to-day changes are G-code + M500 only.

## 2. Access over WiFi (ESP3D on USART3)
- ESP3D web UI: terminal for any G-code (incl. all of the above), printer status, SD file list.
- Firmware update over WiFi (to test, NOT verified):
  1. upload `update.cbd` to the printer SD through ESP3D (`BINARY_FILE_TRANSFER` or M28/M29 over serial),
  2. `M997` (Marlin reboots via flashFirmware/NVIC reset),
  3. Chitu bootloader finds `update.cbd` on SD and installs it.
  Risks: slow serial upload (~250 KB), a broken upload = a broken file. The bootloader checks a CRC,
  so a corrupt file should be rejected (to verify). Fallback is always the SD card by hand.
  Marlin's `chitu_crypt.py` uses a random file key per build, so a new build is never ignored as a duplicate.

## 3. USB debugging by Claude
The cloud session cannot reach the owner's USB port. Options:
- Claude Code (desktop app or `claude remote-control`) on the computer the printer is plugged into,
  with a project folder shared through Remote Control. Then a session can use pyserial on the printer port
  (CH340, USART1, 115200 or 250000 baud).
- Or the owner runs commands and pastes the output here.
