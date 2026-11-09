<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Search for CXY-V8-190729 documentation (2026-11-09)

## Where we searched
- GitHub code search: "CXY-V8-190729", "CXY-V8", "190729 tronxy", "chitu v8 tronxy".
- GitHub issues/PRs: "190729", "CXY-V8", "chitu v8", "tronxy v8", "chitu3d v8".
- Klipper config directory (all Tronxy configs), gists, web (Tronxy wiki, forums, shops, Thingiverse).

## Result: there is no public V8 schematic or full pinout
- **ChiTu3D/ChiTu-3D-Printer-Controller#4** (opened 2022-01-12 by ellensp, a Marlin contributor):
  "Please release CHITU3D printer board documentation. In particular for me CXY-V8-190729".
  **No answer from ChiTu.** The manufacturer has never published it.
- Klipper has **no** V8 config. All Tronxy STM32 configs say "CXY-V6" (x5sa-v6-2019, x5sa-pro-2020,
  xy-2-Pro-2020). Their pins match the common Chitu pins; the probe is on PG9 (V6), not PA14.

## Every V8-specific data point found
| Source | Data | Status |
|---|---|---|
| Marlin PR #15493, JHaney (2020): "my board is a CXY-V8-190729", X5SA Pro | stm32flash: ID 0x0414, **Flash 512 KiB** | Another V8 had a 512 KB chip. Supports ZET6 for ours too; our chip photo still needed |
| same, JHaney | "Put a jumper right of the USB port on the BOOT pins (**JP1**)... reset button (**SW1**) near the BOOT jumper" | Location of BOOT jumper on V8. **We do not touch it.** |
| same, 1lug1998 | "The board has a STM32F103ZET6 processor." (board version not stated) | - |
| same, xC0000005 | USB = **CH340** serial bridge | matches schematic (PA9/PA10) |
| Configurations #321, #658, #659 | PA14 Z endstop stuck in Marlin on V8; X/Y "open" on one X5ST-500 V8 | see v8-z-endstop-pa14-swd.md |
| rhapsodyv in #321 | "Some v8 are equal to V5 and some are equal to V6" | ours = V5-like (probe works on PA14) |
| Marlin #19320 (XY-2, V8-190729) | Colour UI and Classic UI work; LVGL needs assets; SPI flash maybe 1 MB | TFT_COLOR_UI is known to work on a V8 |
| Community Marlin config (rhapsodyv era) | "The V8-190729 is similar to the V5", TMC2208 standalone, 160/160/800 | cross-check only |
| GearBest listing "TMC Series Silent Motherboard" cxy-v8-190729 | **4x TMC2225** drivers | driver type to be confirmed by photo |
| Thingiverse 4267636 | V8 = "mute", pair with T8 screw; 2 jumpers on Z2 | see v8-stock-firmware-z2-jumpers.md |
| matiaspl gist, stock Chitu configs XY-2 Pro | basic: 0.0125 / 0.000625 mm/step (80 / 1600); TITAN: 0.00625 / 0.00125 (160 / 800) | 160/800 = 32 microsteps + T8, consistent with the TMC (mute) board |
| Tronxy shop: WiFi module | module sold as an option; listing says it does not work with the "#446" board | no V8 statement |

## Conclusion
No public V8 pinout exists. The best references are, in order:
1. our own working Klipper `printer.cfg` (verified on this exact board),
2. the X5SA schematic (strong match: PA14 = Z-MINUS-STOP, WiFi = USART3),
3. our own measurements/photos and, after flashing, `M43` pin debugging.
