<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# ESP3D wiring reports (community)

## ESP3D discussion #706 (Dec 2021), board: Tronxy **V6** (X5SA-2E), not V8
https://github.com/luc-github/ESP3D/discussions/706
- Stock Tronxy WiFi module = "basically an ESP01 with 1MB" flash.
- Their socket numbering: 1 GND, 2 WIFI_TX, 4 CH_PD (**5 V**), 5 GPIO0 (**tied to GND**), 7 WIFI_RX, 8 3V3.
- Maintainer luc-github: GPIO0 to GND at start "is not normal"; 5 V on ESP pins "not supported".
- Working fixes: (a) bend GPIO0 pin out, plug module in -> ESP3D boots (5 V still on CH_PD!);
  (b) Dupont wires GND/TX/RX/3V3 only, short 3V3-EN on the ESP-01.
- No baud / Marlin settings given.
- NOTE: their pin numbers differ from the J1 numbering in our schematic (docs/hardware/wifi-socket.md),
  but the functions agree: one position carries 5 V where an ESP-01 has CH_PD, one position is GND
  where an ESP-01 has GPIO0. -> **Pin numbers from forums cannot be trusted; measure on our board.**

## Nathan Tsoi, "ESP3D WiFi Enable your 3D Printer" (Jan 2019), printer: Tronxy **X5S** (old board)
https://www.nathantsoi.com/builds/esp3d-wifi-enable-your-3d-printer/
- Module: LoLin V3 NodeMCU (ESP-12E, 4 MB), ESP3D **1.0** built with Arduino IDE (outdated today).
- Wired to the TX/RX of the **CH340G** (= USB serial) because that old board had no WiFi socket.
  Not for us: we use J1/USART3 so USB stays free.
- Power: "pulling 5V from the TronXY ... the ESP8266 was drawing too much current" -> used a separate
  regulator from the 12 V rail. -> **Board 5 V/3.3 V capacity is a real concern.**
- Baud 115200 in ESP3D to match Marlin.

## Resulting recommendation for us (to be confirmed after measurements)
- Use a dev board with its own regulator and USB (Wemos D1 mini / NodeMCU, 4 MB flash) rather than a bare
  ESP-01: easy flashing, GPIO0/EN handled on-board, enough flash for current ESP3D.
- Wire only **TX, RX, GND** to J1 (3.3 V logic on both sides). Do **not** use J1 5 V or 3.3 V
  for power until we know the board regulator's capacity; prefer a small 24 V -> 5 V buck into the
  dev board's 5 V pin.
- Marlin: `SERIAL_PORT_2 3`, `BAUDRATE_2` = ESP3D baud (115200 is safe; higher to test).
