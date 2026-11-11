<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Marlin + WiFi on Tronxy/Chitu boards: stability and speed reports (searched 2026-11-11)

Searched: GitHub issues (MarlinFirmware/Marlin, rhapsodyv/Marlin = the Tronxy Marlin fork, luc-github/ESP3D),
drucktipps3d.de thread 7796 (16 pages), general web. All reports are from **V6 boards or other printers**;
none is a measured test on a CXY-V8-190729. Nothing here is verified on our printer.

## Positive reports
- drucktipps3d #59 (Iceaac): Marlin (V5 config) "works without problems" on a V8 in an X5SA Pro.
  #86: V8 owner, Marlin running, "shocked what the printer can do now".
- Marlin #19320: Color UI works on a V8-190729.
- rhapsodyv/Marlin #28: 0.6 x 0.5 mm extrusion prints fine at ~100 mm/s on an X5SA Pro V6.

## Negative reports (important)
1. **Motor goes "torque off" mid-print** on Tronxy boards with standalone TMC drivers:
   - Marlin #21390 / rhapsodyv #28 (X5SA Pro, V6, Linear Advance OFF): extruder "just stops even trying, it's
     fully torque off", sometimes after a screech, repeatable at the same spot; one XY motor stop too. Root cause
     never confirmed; issue closed stale. Author moved to a Duet 2 board: "Zero stopping issues".
   - Same thread, Pedro8005x (XY-2 Pro, Chitu board, X axis): happened with **Tronxy stock firmware too**, new
     board and new motor did not cure it; with Marlin "better but fails again at fast accelerations".
     Workaround that worked: **max 150 mm/s and 1500 mm/s2** on X/Y.
   - Reading (hypothesis): hardware behaviour of standalone stealthChop TMC2208/2225 (driver protection trips
     on abrupt load/direction changes), not a Marlin bug. Marlin's own config notes that TMC2208 standalone needs
     a pause after a DIR change "to prevent a standstill shutdown when using StealthChop".
   - Our printer runs the same drivers under Klipper at 300 mm/s / 3000 mm/s2 limits (printer.cfg); whether the
     owner ever saw motor stops there is unknown -> ask.
2. **Linear Advance**: rhapsodyv #14 (CXY-V6): with LA the E motor stops after some hours until power cycle.
   Explanation there: LA cannot be used in stealthChop, needs spreadCycle or UART mode. drucktipps3d #41: same,
   plus layer shifts. No confirmed software fix. (SMOOTH_LIN_ADVANCE, 2025, is untested on these boards.)
3. **Input shaping + multistepping noise**: Marlin #25912 (Creality V4-class F103 board): grinding noise above
   100 mm/s with input shaping; quiet at 100 mm/s; closed "not planned". Developers: multistepping can cause
   "losing steps and strange stepper noises" on underpowered boards.
4. **Screen**: rhapsodyv #51: Color UI characters overlap; screen corrupted during a ~2 h print (print outcome not
   stated; old fork, no answer).
5. **SD prints**: Marlin #24158: freezes/reboots during SD prints traced to SD read errors (other boards).
6. **EMI**: drucktipps3d #193: strong noise from the board's 24->5 V converter falsely triggered a sensitive Z probe.

## ESP3D / WiFi
- ESP3D is only a serial web interface; the print job runs in Marlin from the SD card. So a WiFi drop should not
  stop an SD print (inference; no source states it explicitly).
- Uploads through ESP3D go over the serial line into Marlin's SD writer: slow, and corruption reports exist
  (ESP3D #251, MKS-TinyBee #75). Safe habit: carry big files on the SD card, use WiFi for control/small files.
- No report found of ESP3D on a Chitu V8 WiFi socket. Wiring quirks known (5 V on CH_PD, GPIO0 to GND).

## Conclusion
- Stand-alone Marlin removes our real failure causes (USB link, laptop power).
- Realistic stable target from the reports: **print ~100 mm/s, accel <= 1500 mm/s2, travel <= 150 mm/s**,
  Linear Advance OFF at first, input shaping tested carefully. Higher values only after long test prints.
- Not solvable in firmware: standalone driver behaviour. Real fix = UART mod or a board with UART drivers.
