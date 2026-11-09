<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Power-Loss Recovery (PLR) on the V8

## Marlin (bugfix-2.1.x Configuration_adv.h, read 2026-11-09)
- `POWER_LOSS_RECOVERY`, enabled/disabled at runtime with `M413 S1/S0` + `M500`.
- Works only for prints **from the printer's SD card** (Marlin keeps a recovery file on SD).
  ESP3D uploads to SD and starts the print there -> covered. Prints streamed line by line are not.
- Two modes:
  1. **No detect pin**: state written to SD periodically (each layer, `POWER_LOSS_MIN_Z_CHANGE 0.05`).
     Works on any board. On resume the print continues from the last saved layer.
  2. **With detect pin** (`POWER_LOSS_PIN`): state saved at the moment power drops. Needs a detection
     circuit and enough capacitor energy to finish the write.
- Options: PLR_HEAT_BED_ON_REBOOT (bed heated at once so the part does not detach),
  POWER_LOSS_PURGE_LEN, POWER_LOSS_RECOVER_ZHOME + POWER_LOSS_ZHOME_POS (home Z at a safe XY spot).

## Board facts
- Tronxy official pin file: `POWER_LOSS_PIN PG2` = output of an LM393 comparator (input pull-up),
  `POWER_LM393_PIN PG4` = comparator reference, **must be driven HIGH** by firmware.
- Upstream Marlin Chitu: `//#define POWER_LOSS_PIN PG2 // PG4 PW_DET` (commented out).
- Tronxy's own X5SA_PRO profile has `TRONXY_BACKUP_POWER 0` -> Tronxy itself did not use the
  hardware detection on this model. Whether the LM393 circuit is populated on our V8: **unknown**
  (photo of the board near PG2/PG4 traces, or test with M43 after flashing).

## Risk specific to X5SA (bed moves in Z)
With power off, the heavy 330 mm bed on T8 lead screws can sink by gravity. Then the saved Z is wrong
and the resumed print is shifted. Mitigation: `POWER_LOSS_RECOVER_ZHOME` with a safe XY point for the
probe that is outside the part (e.g. a bed corner), so Z is re-measured before resuming.

## Plan
1. First build: PLR in mode 1 (no pin) + ZHOME at a corner, PLR enabled from the menu.
2. Later: test PG2/PG4 (M43 + unplug test) -> if it works, enable mode 2.
