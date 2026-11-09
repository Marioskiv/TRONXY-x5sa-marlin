<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Tronxy official Marlin for STM32F103 (tronxy3d/F103_PIC480x320)

## Source
https://github.com/tronxy3d/F103_PIC480x320 (GPL-3.0, Tronxy's own account `tronxy3d`)
Commit c11d97c1000548b4b2c53ade3efc3e283e5b0edf (2023-03-09). Marlin-based, "Version 2.18.0".
Models: XY2-PRO, XY3_PRO(_V2), X5SA, X5SA_g, **X5SA_PRO**, X5SA_2E, X5SA400_PRO/2E, XY3SE(_2E/_V2), D01, D01_PLUS, C2.
Board file: `Marlin/src/pins/stm32f1/pins_tronxy_stm32f103.h` (BOARD_TRONXY_STM32F103).
Same account also has `F4xx-*` repos (STM32F446/GD32F4 boards, not ours).

**This is the first manufacturer-published pin map for these boards.** It does not name board
revisions (V5/V6/V8); it is one pin file for all their F103 machines.

## IMPORTANT: different bootloader
`env:tronxy_stm32f103`: `board_build.offset = 0x10000`, output `update/fmw_tronxy.bin` (+ ui/font files).
-> This targets Tronxy's **newer bootloader** (app at 0x08010000), NOT our Chitu bootloader
(app at 0x08008800, encrypted `update.cbd`). This explains the guide warning "newer Tronxy firmware
doesn't read update.cbd". **Do not use this repo's build or update files on our board.**
Only the pin information is useful to us.

## Pin map from Tronxy (compare with upstream pins_CHITU3D_*.h and our Klipper cfg)
| Function | Tronxy official | Upstream Chitu | Note |
|---|---|---|---|
| X step/dir/en | PE5/PE6/PC13 | same | |
| Y step/dir/en | PE2/PE3/PE4 | same | |
| Z step/dir/en | PB9/PE0/PE1 | same | |
| **Z2 step/dir/en** | **PF5/PF1/PF3** | V6 only | separate Z2 driver on some boards; on ours Z2 = jumpered series socket? (check) |
| E0 step/dir/en | PB4/PB5/PB8 | same | |
| E1 step/dir/en | PC7/PC6/PG8 | same | |
| X stop | PG10 | same | |
| Y stop | PA12 | same | |
| **Z min** | **PA14** | V5: PA14 / common: PG9 | our probe on PA14 (verified) |
| **Z max** | **PA13** | V5: SERVO0 PA13 | |
| **Z probe (separate)** | **PG9** | V6 | X5SA_PRO profile uses FIX_MOUNTED_PROBE on PG9 (newer boards) |
| Temp hotend / bed | PA1 / PA0 | same | |
| Heater hotend / bed | PG12 / PG11 | same | |
| Part fan / hotend fan / board fan | PG13 / PG14 / PD6 | same | |
| Beeper | PB0 (passive speaker) | same | |
| Runout 1 / 2 | PA15 / PF13 | same | |
| **Power-loss detect** | **PG2** (LM393 output, input pull-up) + **PG4** (LM393 ref, must be output HIGH) | commented | power-loss recovery possible (to verify on V8) |
| PS_ON / LED | PD3 ("LED used to simulate power switch") | LED_PIN PD3 commented | |
| **WiFi socket** | **PB11 "wifi:TX"**, **PB10 "wifi:RX"** (used by Tronxy for laser PWM / servo) | - | confirms schematic: WiFi = USART3 PB10/PB11 |
| **I2C EEPROM AT24Cxx** | **SCL PG0, SDA PG1, WP PF12** | not used (FLASH_EEPROM_EMULATION) | Chitu bootloader keeps its file key in this EEPROM (hackaday). We keep upstream flash EEPROM so Marlin never touches it. |
| SPI flash W25Qxx 2 MB | PB12/PB13/PB14/PB15 | same | |
| Touch (XPT2046, SPI1) | CS PB7, SCK PA5, MISO PA6, MOSI PA7 | same | Tronxy calibration: X -17181, Y 11434, Xoff 501, Yoff -9 |
| TFT (FSMC, ILI9488 480x320) | CS PD7, RS PD11, RESET PF11, BL PD13, DMA2 CH5 | same | |
| SD | SDIO, detect not connected | same | |
| Timers | STEP_TIMER 3, TEMP_TIMER 4 | - | |
| JTAG | DISABLE_JTAG only | same | SWD stays on -> PA13/PA14 issue (see v8-z-endstop-pa14-swd.md) |

## What this changes for us
- WiFi = USART3 confirmed by the manufacturer's own code (PB10/PB11).
- Power-loss pins PG2/PG4 are new information -> optional POWER_LOSS_RECOVERY later (needs PG4 driven high).
- Board has an AT24Cxx I2C EEPROM used by the bootloader -> keep Marlin on FLASH_EEPROM_EMULATION.
  FLASH_EEPROM sits at the end of flash, so the **512 KB chip size must be confirmed** (chip photo).
- Touch calibration values are a good starting point for TFT_COLOR_UI (Marlin also has on-screen calibration).
