<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# drucktipps3d.de "[Tutorial] Tronxy (X5SA/X5SA-Pro) von Chitu auf Marlin" (16 pages, 2020-2023)

https://forum.drucktipps3d.de/forum/thread/7796-tutorial-tronxy-x5sa-x5sa-pro-von-chitu-auf-marlin/
Tutorial itself covers V5/V6. Relevant posts (German, paraphrased; NOT verified by us):

- #9-#12 (Jul 2020): V8 in X5SA Pro -> "compile Marlin for V5 with the matching settings, the boards are the same".
  Only known difference V6/V8: stock Chitu firmware on V8 limits acceleration and crashes at high values.
- #58-#59 (Oct 2020): XY-2 Pro with V8 -> "V5 should work without problems ... on my V8 in the X5SA Pro it works".
- #86 (Nov 2020): user with **V8** has Marlin running, runout sensor works; asks about the LED output.
- Tutorial: "TMC2225 = TMC2208 in another package". Board jumpers: **BOOT** and **"v source" (5V/USB)**.
  Their backup procedure (ROM bootloader read) expects a **512 kB** dump (524288 bytes).
- #128-#131 (Jan 2021): the Chitu bootloader installs `update.cbd` **only if it is newer/different** than the
  installed one; during install the screen is black and it beeps for a few seconds. `chitu_initial_file_autorun`
  is a G-code file the *stock* firmware runs at every boot (like Marlin's auto0.g).
  Links a dedicated thread "chitu-firmware-update-cbd-fuer-tronxy-v8-board" (old URL, no longer reachable).
- #216 (Sep 2021): bad update.cbd on V6 -> 5 beeps + continuous tone, black screen; fixed by flashing the
  original Tronxy update file. -> recovery by SD works if we keep a known-good update.cbd.
- #41 (Sep 2020): **Linear Advance** on V6 with TMC2208 standalone: a few good cubes, then layer shifts and
  the extruder stopped after 5-15 mm. -> confirms the LA + standalone TMC problem.
- #234 (May 2022, Arakon): **UART mod** for X, Y, E on the usual Chitu V6 board (thin AWG30 wires;
  "20k resistors only needed for Marlin, Klipper works without"), based on a plan by korrado (RepRap forum).
  Enables Linear Advance and current setting in firmware. Image tronxy-uart-mod.png (attachment, not fetched).
- BLTouch posts: several people used the WiFi socket pins for the BLTouch (as Darkwulf183 does with PD12).

bit.ly/3zCrela (Darkwulf183 "Board conversion" for Linear Advance) -> YouTube video DLwTD4qMoG8 (not viewable here).
