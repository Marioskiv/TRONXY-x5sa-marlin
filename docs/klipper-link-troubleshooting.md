<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Keep the current Klipper working: link troubleshooting (2026-11-11)

Setup (verified): Debian PC runs two Klipper instances; Tronxy instance in `~/printer_TRONXY_data`,
Moonraker port 7126, `[mcu] serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0`, 250000 baud,
MCU firmware built for USART1 (CH340 USB), offset 0x0 (no bootloader), klipper.bin dated 2026-03-10.

## Likely causes (to confirm, in this order)
1. **Same by-id name for two CH340 boards.** Cheap CH340 chips have no serial number, so two printers get the
   same `/dev/serial/by-id/...` name; only one link exists, so one printer "disappears".
   Check: `ls -l /dev/serial/by-id/ /dev/serial/by-path/` with both printers plugged in.
   Fix: use `/dev/serial/by-path/...` (tied to the physical USB port) in `printer.cfg`, or a udev rule.
2. **MCU firmware older than the host** (MCU built 2026-03-10; Moonraker `channel: dev` keeps updating Klipper).
   Check: `tail -n 200 ~/printer_TRONXY_data/logs/klippy.log` for `Version mismatch`, `Unknown command`, `Protocol error`.
   Fix: rebuild and reflash MCU from the same Klipper version (BOOT jumper + USB), or pin the host version.
3. **Service not running / Mainsail forgot the printer.** `systemctl list-units 'klipper*' 'moonraker*' --all`;
   in Mainsail add printer `IP:7126`.
4. **USB power management / cable.** `cat /sys/module/usbcore/parameters/autosuspend` (should be -1);
   short data cable, no hub, `dmesg -w` while moving the cable (look for `disconnect`, `reset`).
5. **Board power jumper**: "v source" must be on **5V** (not USB) during normal use. The USB position is only for
   flashing (drucktipps3d tutorial, Eddy Beaupre guide).
6. Phone/BeamKlipper link (not the PC): logs show `bytes_retransmit=268`, `bytes_invalid=217`, many
   `Serial connection closed` -> unreliable on that path; PC link is the reference.

## Next step
The local session already has SSH access to the Klipper PC: run checks 1-4 read-only there, then propose fixes
(owner approves each change).

## Owner report (2026-11-11): what actually happens
- Prints sometimes stop because the **USB link is interrupted**. Cable: 1 m, with ferrite.
- The host laptop **sometimes powers off**: it has no battery installed and runs from its charger only
  (a power/cable glitch kills it, and the print).
- Owner has a second small, old, slow laptop **with a working battery** (not yet checked).

### Reading
- Klipper aborts the print on any MCU link loss, so host power and USB must be rock solid.
- A laptop **with a battery acts as a built-in UPS for the host** (survives charger glitches and short power blips).
- "Slow" is fine: the Klipper host load is small (it runs on a Pi Zero). Needs ~1 GB RAM+, a normal Linux install.
- Do not mix up two failure types in `klippy.log`:
  `Lost communication with MCU` / `Serial connection closed` -> cable, EMI, ground loop, host power;
  `Timer too close` -> host too busy/slow.
  Check: `grep -nE "Lost communication|Timer too close|MCU shutdown|Serial connection closed|Unable to open serial" ~/printer_TRONXY_data/logs/klippy.log*`
- Extra for the USB: keep the USB cable away from motor/heater cables; laptop chargers with 2 pins can inject noise
  (a USB isolator can fix ground-loop problems); "v source" jumper on 5V; disable USB autosuspend.

### Plan
Dedicated old laptop (battery in good shape, not swollen) as the only host for the Tronxy: clean Debian + KIAUH,
sleep/suspend/lid-close actions disabled, one printer per host (no by-id name clash).
