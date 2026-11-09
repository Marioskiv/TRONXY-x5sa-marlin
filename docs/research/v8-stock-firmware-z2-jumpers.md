<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Stock firmware note for the black CXY-V8-190729 board (Z2 jumpers, lead screw)

## Source
Found by owner in a forum (2026-11-09). Very likely the description of Thingiverse thing 4267636
"Tronxy xy 2 Pro (FIRMWARE ONLY BLACK BOARD V8 190729)": https://www.thingiverse.com/thing:4267636

## Verbatim
> ONLY BLACK BOARD 190729....COPY ALL FILES TO SD (NO FOLDERS) PRINT GCODE BIN AND LOGO.....FINISH RESTART
>
> VERY IMPORTANT!!!! INSERT 2 JUMPERS ON THE Z2 CONNECTOR AS IN THE PHOTO...
>
> If your previous machine should be a non mute motherboard (CXY-V5-180409),
> With a T2 lead screw,
> Now you bought the mute motherboard (CXY-V8-190729),
> It should be matched with T8 lead screw.

## Interpretation (NOT verified)
1. **Firmware package**: this is a *stock Chitu* firmware for the V8 (installed by copying files
   to the SD root and "printing" the gcode). Not Marlin. But it is a possible **recovery source**
   for the stock firmware if the original update file is lost.
2. **Z2 jumpers**: on this board the Z2 socket is probably wired **in series** with Z (one Z driver
   feeds both sockets). With only one motor, the Z2 socket must be bridged by 2 jumpers to close the
   coil circuits, otherwise the Z motor does not move. This is hardware only; Marlin does not see it.
   - If true: Marlin needs **no** Z2 driver / `NUM_Z_STEPPERS 2` for this socket.
   - Check: how many Z motors the X5SA Pro has here, where each is plugged, whether the jumpers are on Z2,
     and whether Klipper's printer.cfg defines a `[stepper_z1]`.
3. **"Mute" board + T8 lead screw**: V8 = "mute" (silent drivers). The stock firmware for it
   assumes a different Z lead/steps than the V5 + "T2" setup. For Marlin, Z steps/mm must match the
   real lead screw **and** the drivers' microstepping (fixed in hardware on standalone drivers).
   - Klipper `rotation_distance` + `microsteps` in the working printer.cfg is the verified value;
     we take Z steps/mm from there, not from forum text.

## What it changes for us
- Do not remove Z2 jumpers. Ask owner for a photo of the Z/Z2 sockets.
- Z steps/mm and driver type: derive from printer.cfg and a photo of the driver chips.
- Thingiverse 4267636 files = candidate stock-firmware backup (to be downloaded by owner; not flashed).
