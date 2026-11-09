<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# New direction (owner decision 2026-11-09): Klipper on a dedicated Raspberry Pi, serial link, touchscreen

Reason: owner likes Klipper but wants a stable, self-contained printer (no shared PC, no USB drop-outs).
The Marlin work stays in the repo (docs/plan-*.md, docs/research/) as a fallback.

## Target
- Raspberry Pi mounted inside the printer, only for the Tronxy. Klipper + Moonraker + Mainsail + KlipperScreen.
- MCU link: **UART, no USB**: Pi GPIO14/15 <-> board USART3 (WiFi socket J1, PB10/PB11), 3.3 V both sides.
- Touchscreen on the Pi (KlipperScreen). The original Tronxy screen stays unused (Klipper cannot drive it).
- Pi powered from the printer PSU via a 24 V -> 5 V buck (5 A if the screen is also fed from it).

## Hardware to buy (suggestion, owner to choose)
| Part | Suggestion | Why |
|---|---|---|
| Pi | Raspberry Pi 4 (2 GB) or Pi 3B+ | KlipperScreen runs smoothly; Zero 2 W works but is slow with a screen |
| Screen | Raspberry Pi Touch Display 2 (7", DSI) or a BTT HDMI5/HDMI7 (made for KlipperScreen) | well-supported touch drivers |
| Power | 24 V -> 5 V buck, 5 A, fixed output (5.1-5.2 V) | Pi 4 + screen need ~3-4 A |
| Storage | good A1/A2 microSD (or USB SSD on Pi 4) | SD corruption is the usual Pi failure |
| Wires | 3 x Dupont (TX, RX, GND), short, twisted | serial link |

## Wiring (from schematic + Tronxy code; J1 numbering NOT yet measured on our V8)
| Pi | Board J1 (schematic numbering) | MCU |
|---|---|---|
| GPIO14 TXD (header pin 8) | pin 1 WIFI_UART_RX | PB11 USART3_RX |
| GPIO15 RXD (header pin 10) | pin 5 WIFI_UART_TX | PB10 USART3_TX |
| GND (header pin 6) | pin 6 or 8 GND | GND |
| (do not connect) | pin 2 = 5 V, pin 4 = 3.3 V | - |
**Measure J1 before connecting** (photo + voltages), because forum numberings differ.

## Software steps (later, one at a time, owner approves each)
1. Pi OS Lite + KIAUH: Klipper, Moonraker, Mainsail, KlipperScreen.
2. Pi UART: `enable_uart=1`, `dtoverlay=disable-bt` (Pi 3/4), serial console off -> device `/dev/ttyAMA0`.
3. Klipper MCU build: STM32F103, 8 MHz crystal, **Serial (on USART3 PB11/PB10)**, 250000 baud,
   bootloader offset = whatever the board has now (see open bootloader question).
4. Flash once (owner), same method as the current Klipper (probably USB + BOOT jumper).
5. printer.cfg: `[mcu] serial: /dev/ttyAMA0` + `baud: 250000`; rest from backup/klipper/pasted/printer.cfg,
   updated for K2 Plus hotend (custom thermistor), HGX extruder, external hotend MOSFET.

## Still open (same as before)
- Bootloader offset of the current Klipper build (FLASH_START).
- K2 Plus thermistor values; HGX extruder rotation distance; J1 measurements.
