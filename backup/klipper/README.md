<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Klipper backup

Verified-working Klipper setup for this printer. Used as:
1. the pin reference for the Marlin port,
2. the rollback path (flash `update.cbd` built from these files).

Expected contents (uploaded by owner):
- `klipper-backup.tgz` – printer_data/config, klipper/.config, klipper/out/klipper.bin (+ update.cbd if present)
- `klipper-version.txt` – Klipper git commit + local changes
- `klippy.log` (optional) – shows the MCU build/config Klipper actually used

Files are committed unmodified; extracted copies go next to the archive.
