<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Klipper power-loss recovery (PLR): which one for the X5SA? (researched 2026-11-14)

Owner's question: which PLR is the most reliable and accurate; the one used on the Kobra Plus may lose a layer.
Sources read: the READMEs and scripts of each project (raw files on GitHub, 2026-11-14).

## Our printer's facts that decide it (from backup/klipper/pasted/printer-2026-11-13.cfg)
- Z endstop = **inductive probe** (`endstop_pin: probe:z_virtual_endstop`), `[safe_z_home]` at **165,165** (bed centre).
- Z = 2 motors on one driver, T8 lead screws, `rotation_distance: 8`. The bed is heavy and moves in Z.
- Host = laptop (Debian, user `marios`, instance `printer_TRONXY_data`), not a Pi.
- **Unknown, must be measured:** does the bed sink (or tilt) when power is cut? (power-loss-recovery.md)

## The candidates
| Project | How it records | How it resumes | Resume accuracy | Fit for X5SA |
|---|---|---|---|---|
| **Yumi-Lab/YUMI_PLR** (original) and forks **bigtreetech/KlipperPLR**, **Kobra-S1/KlipperPLR** | `LOG_Z` in the slicer's "before layer change" G-code writes Z (+temps in the Kobra-S1 fork) to `variables.cfg` every layer; `save_last_file` at start | `plr.sh` cuts the G-code with `sed` at the logged Z, adds `SET_KINEMATIC_POSITION Z=...`, lifts 10 mm, homes **only X/Y**, replays from the start of that layer | Layer level. Kobra-S1 README: "No X/Y position recovery", "interrupted layer bonds weakly", "Z must not move during power loss" | Works with our probe (no Z homing), **only if the bed does not sink**. Paths are hard-coded to `/home/pi/...` -> must be edited for `marios` / `printer_TRONXY_data` |
| **The--Captain/plr-klipper** (older origin of the same idea) | same LOG_Z | same, plus manual "touch the nozzle on the part" option | Layer level | Older, needs gcode_shell_command; no advantage |
| **mikeoverbay/klipper-power-loss-recovery** | nothing per layer; reads `sd_pos` from klippy.log + small json for babystep/mesh | heats nozzle in place, lifts, homes X/Y **and Z at the X/Y home corner**, counts steps to know where it stopped, finds the **exact move** | Best: Z within ±0.005 mm, resume move 0.06-0.07 mm from where it stopped (author's tests) | **Not supported as is**: README says a bed probe as Z endstop with safe_z_home at the centre would drive into the part. Would need a mechanical Z endstop (Z-max, untested by the author). Tested on one machine only |
| Royal3DP-Klipper-PLR | last layer height | user raises Z by hand | Layer level | no advantage |
| OMEGA_PLR (Sovol SV07 Plus) | - | - | - | could not read (forum host blocked) |

## Weak points found in the YUMI/KlipperPLR script (read in plr.sh, verified)
- The resume point is found by **text search** of ` Z<value>` in the G-code (`sed '1,/Z'${1}'/ d'`). The first pattern is not
  anchored, so `Z0.4` also matches `Z0.45`; and PrusaSlicer/Orca often write `Z.4` (no leading zero) for heights under 1 mm.
- `SET_KINEMATIC_POSITION` takes the **first ` Z` line after the match**; with slicer **Z-hop** that can be a hop
  move, so Z would be set wrong by the hop height. -> Plausible cause of "it loses/doubles a layer".
- `LOG_Z` runs **before** the layer change, so it logs the layer just finished; whether the script replays that layer
  or the next depends on the text match above. Kobra-S1 README: "A failure between two LOG_Z calls replays from the earlier one."
- Everything relies on the bed not having moved.

## Recommendation for the X5SA (proposal, not applied)
1. **First measure** whether the bed sinks/tilts with power off (mark both sides, 10 min off).
2. If it **does not sink**: KlipperPLR (bigtreetech fork, generic) with paths fixed for our install, **Z-hop off or checked**
   in the slicer, and a dry test: start a print, cut the mains on a layer above 1 mm, resume, inspect.
3. If it **sinks**: neither works unchanged. Option: our own resume macro on top of KlipperPLR's file cutting:
   heat nozzle in place -> lower bed a few mm with Z forced -> home X/Y -> **probe Z at a bed corner outside the part**
   (like Marlin's POWER_LOSS_RECOVER_ZHOME) -> continue from the logged layer. Only works if both Z screws sank equally
   (one driver in series, so they should move together, but a tilt must be ruled out by the measurement).
4. mikeoverbay only if we later add a mechanical Z endstop; it is the most accurate design.
In all cases the host stays alive on the laptop battery, so klippy.log keeps the last file position.

## Power off after the print (owner: take it from the Kobra Plus)
- The Kobra Plus config files were **not found**: the only private repo this session can see is `Marioskiv/cnc-manuals`
  (CNC only, all 25 branches searched). A repo named "manual" is not visible to the session's GitHub access.
- Klipper's "sleep" is `[idle_timeout]` (default 600 s): it turns off **heaters and motors only**. Board, fans, PSU, LEDs
  stay on. A real power off needs a smart plug/relay controlled by Moonraker `[power]` + a macro at print end.
