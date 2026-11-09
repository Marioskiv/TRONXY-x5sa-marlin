<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Facts vs. assumptions

## Verified (by owner, on the hardware)
| Item | Value |
|---|---|
| Board | CXY-V8-190729 |
| MCU | STM32F103 high density, stm32flash ID 0x0414, 8 MHz crystal |
| Bootloader | Chitu, app at 0x08008800, updates via `update.cbd` on SD |
| Klipper | currently works; its `printer.cfg` is the pin reference |
| Z probe | works on `^!PA14` (pull-up, inverted) |
| Stock firmware | Chitu V1.4.1RC26m4, UI Colorful 0.8 |

## Read from Marlin source in an earlier session (to be re-checked in step 2)
- `env:chitu_f103`: genericSTM32F103ZE, offset 0x8800, outputs `update.cbd`.
- `Configurations/Tronxy/X5SA Pro` uses BOARD_CHITU3D_V6 (probe PG9) -> not a direct match.
- `pins_CHITU3D_common.h` only DISABLE_JTAG -> PA13/PA14 remain SWD; probe on PA14 may need DISABLE_DEBUG.

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

## Open items owed by owner
- [ ] Klipper backup (step 1)
- [ ] Stock Chitu `update.cbd` (recovery)
- [ ] Hotend thermistor type, extruder steps/mm
- [ ] Photos: MCU marking, WiFi socket front/back, stepper drivers, display ribbon/label, jumpers
- [ ] Voltages on each WiFi socket pin vs GND
