<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# V8 Z endstop / probe on PA14 and SWD

## Sources (read 2026-11-09)
- Configurations #658 "Tronxy X5ST-500 with CXY-V8-190729" (closed, no fix posted):
  https://github.com/MarlinFirmware/Configurations/issues/658
  M119 shows X/Y/Z "open" while pressed; maintainer says use BOARD_CHITU3D_V5; then Z behaves "like #321".
- Configurations #321 "[BUG] TRONXY Chitu V8 Z Endstop" (open, labelled bug, no confirmed fix):
  https://github.com/MarlinFirmware/Configurations/issues/321
  - "Z-Endstop on Chitu V8 boards seems to remain in triggered state after pressed once and released."
  - Pull-up did not help. `chitu_v5_gpio_init` (CHITU_V5_Z_MIN_BUGFIX) did not help.
  - Maintainer rhapsodyv: "Some v8 are equal to V5 and some are equal to V6" (V6 = probe on PG9).
  - Workarounds reported: move switch to Z-max and use PA13; one user built a divider because
    "the bed-levelling (z-min) connector ... runs at 24v".
  - thinkyhead: use `PINS_DEBUGGING` + `M43` to locate pins.
- Marlin #19320 (XY-2 with V8-190729, LVGL blank screen): resolved (assets placement). Side note from
  maintainer: that board may have **1 MB** SPI flash, not 2 MB like the X5SA (matters only for LVGL/fonts).

## Code facts (read from source today)
- Marlin `env:chitu_f103` uses the STM32 HAL (`stm32_variant`). In `HAL/STM32/HAL.h` for STM32F1:
  `JTAG_DISABLE()` = SWJ_CFG_JTAGDISABLE (SWD stays on), `JTAGSWD_DISABLE()` = SWJ_CFG_DISABLE,
  `JTAGSWD_RESET()` = full JTAG+SWD.
- `MarlinCore.cpp` setup(): runs `JTAGSWD_RESET()`, then `JTAGSWD_DISABLE()` only if `DISABLE_DEBUG`,
  else `JTAG_DISABLE()` if `DISABLE_JTAG`.
- `pins_CHITU3D_common.h` defines only `DISABLE_JTAG` -> **PA13/PA14 stay SWDIO/SWCLK under Marlin**.
- Klipper `src/stm32/stm32f1.c` gpio_peripheral(): when PA13 or PA14 is configured as a GPIO it
  **automatically** sets SWJ_CFG_DISABLE ("Disable SWD to free PA13, PA14"). The Kconfig option
  "Disable SWD at startup" exists only for GigaDevice clones.

## Conclusion (hypothesis, strong; NOT yet tested on this board)
Klipper works on `^!PA14` because it turns SWD off automatically. Marlin, with only DISABLE_JTAG,
leaves PA14 as SWCLK (which the MCU holds with an internal pull-down while SWD is enabled). That
matches the #321 symptom. Fix to try: `#define DISABLE_DEBUG` in our config (frees PA13/PA14).
Side effect: no SWD debugging after boot. Harmless for us (we never use a debugger/stm32flash).

Verification plan after flashing: `M119` with probe free/triggered, several times; `M43` (PINS_DEBUGGING)
watch on PA14 if needed.

## Other points
- Our board: probe works on PA14 under Klipper -> our V8 is the "V5-like" variant for the probe pin.
- 24 V on the probe/Z-min connector: Klipper reads it fine today, so the board has its own level
  shifting/divider on that input. No hardware change planned; check with photo if in doubt.
