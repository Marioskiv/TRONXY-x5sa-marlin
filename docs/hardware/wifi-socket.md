<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# WiFi socket (J1 "UART_WIFI")

## Source
Tronxy/Chitu mainboard schematic `x5sa-main_board-2.pdf` (Altium, dated 2019-06-03),
linked from Marlin `pins_CHITU3D_V6.h`:
https://github.com/MarlinFirmware/Marlin/files/3401484/x5sa-main_board-2.pdf
sha256 `31432cb46378c3d04c7b8069ec453945baaba9dce9221df038b06daef46bc979`.

The title block is empty, so the schematic does **not** say which board revision it is.
It matches this printer on these points:
- PA14 = `Z-MINUS-STOP`: your probe runs on PA14 under Klipper (verified)
- PA13 = `Z-PLUS-STOP`, PA15 = filament runout, PG10 = X-, PA9/PA10 -> CH340G (USB)

**Status: schematic = strong evidence, NOT yet verified on the CXY-V8-190729.**

## What the schematic shows
There is **no ESP8266 chip on the board**. There is an 8-pin header (2x4) for an
optional plug-in module (Tronxy sold one). It is wired to **USART3**:

| J1 pin | Net (board side) | MCU |
|---|---|---|
| 1 | WIFI_UART_RX (module TX -> board) | PB11 = USART3_RX |
| 2 | VCC_5V | - |
| 3 | PD12_EXT_CS | PD12 (not used by Marlin) |
| 4 | VCC_3.3V (10 uF to GND) | - |
| 5 | WIFI_UART_TX (board -> module RX) | PB10 = USART3_TX |
| 6 | GND (tied to pin 8) | - |
| 7 | not connected | - |
| 8 | GND | - |

Chinese notes on the sheet: "Serial WIFI/ESP8266, reserved SPI interface, RX to TX, TX to RX"
and "the 5V pull-up is for future 5V expansion modules".

In Marlin this means `SERIAL_PORT_2 3`, which the `Tronxy/X5SA` example config already sets.
Nothing else on the board needs to be switched on.

## Why a plain ESP-01 must NOT be plugged straight in
Standard ESP-01 header: GND, GPIO2, GPIO0, RXD / TXD, CH_PD(EN), RST, VCC.
If an ESP-01 is plugged into J1 in the matching orientation:
- TXD -> pin 1 (OK), RXD -> pin 5 (OK), VCC -> pin 4 3.3 V (OK), GND -> pin 8 (OK)
- **CH_PD/EN -> pin 2 = 5 V** (over the ESP8266 limit)
- **GPIO0 -> pin 6 = GND**, so the module boots into flash mode and ESP3D never starts
- RST -> pin 3 (PD12), GPIO2 -> pin 7 (NC)

This matches what users report in ESP3D discussion #706 (5 V on CH_PD, GPIO0 grounded).
Safe wiring: use jumper wires (TX, RX, 3.3 V, GND only), tie EN to 3.3 V on the module,
leave GPIO0/GPIO2 unconnected (internal pull-ups), and do not connect pin 2 (5 V).

## Still to verify on the real board
- [ ] Photo of J1 front and back: is the header populated, pin-1 marker, which row is which
- [ ] Multimeter vs GND (printer on, nothing plugged in): voltage on each of the 8 pins
      (expected: 2 = 5 V, 4 = 3.3 V, 6 and 8 = 0 V / continuity to GND)
- [ ] Continuity J1 pin 1 -> MCU pin 70 (PB11), J1 pin 5 -> MCU pin 69 (PB10), if you are comfortable probing the LQFP144
- [ ] Do you have the original Tronxy WiFi module, or a plain ESP-01 / other ESP8266 board?
