<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Facts vs. assumptions

## Verified (by owner, on the hardware)
| Item | Value |
|---|---|
| Board | CXY-V8-190729 |
| MCU | STM32F103 high density, stm32flash ID 0x0414, 8 MHz crystal |
| Bootloader | Chitu, app at 0x08008800, updates via `update.cbd` on SD |
| Klipper | currently works; its `printer.cfg` is the pin reference |
| Z probe | works on `!PA14` (inverted; cfg has no Klipper pull-up) |
| Klipper cfg | `backup/klipper/pasted/printer.cfg`: all pins = Marlin CHITU3D_V5 (docs/pin-comparison.md) |
| Steppers | 32 microsteps all axes; X/Y rd 40, Z rd 8, E Titan clone rd 22.478 66:22 |
| Thermistors (Klipper) | hotend ATC Semitec 104GT-2, bed EPCOS 100K |
| Stock firmware | Chitu V1.4.1RC26m4, UI Colorful 0.8 |

## Read from Marlin source in an earlier session (to be re-checked in step 2)
- `env:chitu_f103`: genericSTM32F103ZE, offset 0x8800, outputs `update.cbd`.
- `Configurations/Tronxy/X5SA Pro` uses BOARD_CHITU3D_V6 (probe PG9) -> not a direct match.
- `pins_CHITU3D_common.h` only DISABLE_JTAG -> PA13/PA14 remain SWD; probe on PA14 may need DISABLE_DEBUG.

## RESOLVED: the Chitu bootloader is gone (verified 2026-11-09 over SSH, docs/research/phone-host-check.md)
- Klipper PC `.config`: `CONFIG_STM32_FLASH_START_0000=y` -> Klipper was built for offset 0x0 and flashed
  at 0x08000000, **overwriting the Chitu bootloader**. `update.cbd` from SD cannot work today.
- Recovery is always possible: BOOT jumper + USB (STM32 ROM bootloader), `stm32flash` is on the Klipper PC.
- Candidate bootloader image (hackaday 167594, X5S, 34816 B) is **not verified for the V8**.

## Assumptions (NOT verified)
- "V8 board = Chitu V5 pinout" (community claim, Configurations README).
- WiFi socket J1 = USART3 (PB10 TX / PB11 RX), pin 2 = 5 V, pin 4 = 3.3 V. Confirmed by the X5SA schematic
  (docs/hardware/wifi-socket.md) AND by Tronxy's own Marlin pin file (PB11 "wifi:TX", PB10 "wifi:RX").
  Two independent sources; still to measure on our V8.
- MCU is 512 KB (ZET6) – ID 0x0414 also covers 256/384 KB parts. Needs chip photo.
- WiFi socket supply/logic voltage – needs measurement (ESP8266 is 3.3 V only).

## Strong evidence (manufacturer code, not yet measured)
- Power-loss detect PG2 + PG4, I2C EEPROM AT24Cxx on PG0/PG1 (WP PF12), Z max PA13, Z2 driver PF5/PF1/PF3
  (docs/research/tronxy-official-marlin-f103.md).
- Linear Advance with standalone TMC2208/2225 is unreliable (forum reports); needs UART mod or spreadCycle.

## Hardware changes in progress (owner, 2026-11-09)
- Hotend: **Creality K2 Plus hotend** (owner). Flow not a limit. Thermistor = Creality custom (TBD);
  heater reportedly 100 W ceramic -> current check needed (docs/research/k2-plus-hotend.md).
- Extruder: will be "HGX pulse 562", drive ratio 9.54647:1 -> E steps/mm TBD (need motor step angle
  and the maker's rotation distance / steps value).

## Open items owed by owner
- [ ] Klipper backup (step 1)
- [ ] Stock Chitu `update.cbd` (recovery)
- [ ] Hotend thermistor type, extruder steps/mm
- [ ] Photos: MCU marking, WiFi socket front/back, stepper drivers, display ribbon/label, jumpers
- [ ] Voltages on each WiFi socket pin vs GND
