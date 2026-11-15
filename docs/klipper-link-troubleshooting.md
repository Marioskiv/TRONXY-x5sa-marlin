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

## Candidate host laptop: Toshiba NB250-107 (owner photo of the label, 2026-11-15)
Label (verified): part PLL2PE-00200WEN, `AN455/1G/250G/10.1WSVT` = Intel **Atom N455**, **1 GB** RAM, **250 GB** HDD,
10.1" screen, DC 19 V 1.58 A (30 W adapter), Wi-Fi module PA3722U-1MPC, Windows 7 Starter.
General specs (Intel/Toshiba, from memory, not re-checked here): N455 = 1 core / 2 threads, 1.66 GHz, **64-bit**,
max RAM 2 GB DDR3 (one SO-DIMM slot), very low power.

Assessment (assumption, not tested on this printer):
- Klipper's step computation is C code; klippy, Moonraker and nginx are light. A 1 GB / single-core 64-bit host is in
  the same class as a Raspberry Pi 3 that many printers run on. Run it **headless** (Debian 12/13 amd64 minimal,
  no desktop), Mainsail opened from another device.
- Limits: one slow core. Avoid extras on it (timelapse encoding, webcam streaming, KlipperScreen with a desktop).
  Watch klippy.log for "Timer too close" / high `sysload` during fast prints with input shaper.
- Cheap upgrades worth doing: RAM to **2 GB DDR3 SO-DIMM** (max), old HDD -> small **SATA SSD** (15-year-old disk is the
  most likely part to fail). Check the battery holds the laptop for at least a few minutes.
- Settings needed: lid close ignored (`HandleLidSwitch=ignore` in logind), no suspend, USB autosuspend off, Wi-Fi power
  save off; BIOS "power on with AC" probably not available on this netbook (to check in its BIOS).

### Search for real reports (2026-11-15)
- **No report found** of Klipper on an Atom N450/N455 (or N270) netbook, good or bad. klipper3d.org and
  klipper.discourse.group are blocked from the cloud session, so only search-result snippets could be read.
- Indirect, from snippets: a Klipper forum member warned that the usual problem host is "an old laptop ... that has one
  core, 1GB of DDR"; an Asus Eee PC user found the netbook struggled after also running a heavy browser on it; a Hackaday
  author runs Klipper + OctoPrint on an old x86 netbook; for a Celeron NUC the advice was "do not run anything in
  parallel (apart Klipper frontends)". cpubenchmark: N455 single-thread about the same as N450 (~200-280 points, very low).
- Conclusion: plausible headless, not proven. The decisive test is cheap: Debian minimal + Klipper on the NB250, then a
  fast print without filament (or the real speeds with input shaper), and read the `Stats` lines in klippy.log
  (`sysload`, `cputime`, `memavail`) or graph them with `~/klipper/scripts/graphstats.py`. "Timer too close" = too slow.

### Deeper research (2026-11-15, owner wants certainty without tests)
Forum, Klipper docs site and openbenchmarking are blocked here (owner declined allowing them), so: Klipper repo files
(GitHub raw), GitHub issue search and search-result snippets.
- **Klipper FAQ (docs/FAQ.md, read in full section):** recommended Pi Zero 2 W, Pi 3, 4, 5; Pi 1, 2 and **Pi Zero 1
  "don't have enough processing power"** (print stalls). x86 is supported; only hard need is double-precision FP
  hardware; avoid heavy tasks (defrag, heavy swapping) during prints; disable ModemManager.
- **Klipper issue #237:** "we have been able to run klipper on old nettop. Atom CPU. Debian linux." -> works (old report,
  model not given).
- Klipper forum (snippet): Eee PC 900 host (Celeron M/Atom class) with 2 GB + SSD; the open problem there was networking;
  a reply doubted the old netbook plus graphics load. No failure due to CPU was reported in the snippet.
- **Per-core speed:** Phoronix clock-for-clock test: Cortex-A9 roughly matches Atom N450 per clock. Cortex-A53
  (Pi Zero 2 W 1.0 GHz, Pi 3 1.2 GHz) is about A9 class per clock. => one N455 core at 1.66 GHz is at least one Pi Zero 2 /
  Pi 3 core, and far above the Pi Zero 1 (ARM11 1 GHz) that the FAQ calls too weak. Openbenchmarking snippet: Atom N270
  beat the Pi 3 in Himeno (101 vs 71 MFLOPS), lost in GnuPG (61 vs 53 s).
- **Commercial reference:** Creality K1 runs Klipper (input shaper, 600 mm/s class) on an Ingenic X2000E dual-core MIPS
  1.2 GHz with **256 MB** RAM (HelixScreen developer page). RAM is not the limit at 1 GB.
- Difference to a Pi 3: 1 core + Hyper-Threading instead of 4 cores, so Moonraker/nginx share the core with klippy.
  Fine headless; not fine with desktop, browser, webcam or timelapse on the same machine.
Verdict: very likely adequate for this printer as a headless host (evidence-based estimate, not a test on the NB250).
Remaining real risks are the 15-year-old parts: HDD, battery, power adapter, USB port.

### Research with the Klipper forum open (2026-11-15, owner allowed the hosts)
**Same benchmark for all (PassMark cpubenchmark.net, via search snippets, Oct 2026):**
| CPU | single-thread | all cores |
|---|---|---|
| **Atom N455** (NB250) | **266** | 188 |
| Cortex-A53 4x1.2 GHz (**Pi 3**, recommended) | ~196 | ~266 |
| Cortex-A53 4x1.0 GHz (**Pi Zero 2 W**, recommended) | ~162-169 | ~198-215 |
=> per thread the N455 is ~35 % faster than a Pi 3 and ~60 % faster than a Pi Zero 2 W; in total it is about a
Pi Zero 2 W. Klippy's time-critical work (main thread + serial thread) depends mostly on single-thread speed.

**Klipper forum, read in full:**
- Sineos (moderator), t/19298 (Eee PC 900 thread): "Klipper itself and most of its ecosystem do not require much CPU
  power or memory. Anything from the RPi 3 class and above is fully sufficient." and "The Klipper host should only do its
  host thing and nothing else." (That user switched to a Pi 3 before testing the netbook: no result either way.)
- Piezo, t/8758 (Timer too close on a Core 2 Solo SU3500 laptop with Linux Mint, CPU 10-15 %): cause was background jobs
  of the desktop distro saturating the CPU; "An old SU4100 laptop should not have any issues with Klipper if you dedicate
  it entirely to this task with a lightweight distribution like Debian." "Avoid the single core SBCs like the Raspberry
  Pi Zero (W)." Hardening tip for /etc/systemd/system/klipper.service: `[Service] Nice=-18 IOSchedulingPriority=1`.
- Sineos, t/19116 (Fujitsu laptop with Mint, disconnects): "Neither the processing power nor the RAM is relevant for
  Klipper"; mykepredko: the laptop problem is "execution consistency" of mainstream desktop OSes, not CPU power.
- Kevin O'Connor, t/2492: a 580 MHz single-core MIPS box with 128 MB (Creality Wi-Fi box) was overloaded (load >100 %)
  and swapped; "low end host cpus aren't a target for me". The N455 is far above that box.
**Conclusion:** CPU and RAM of the NB250 are sufficient (above the recommended Pi Zero 2 W per thread, equal in total).
The documented failures of laptops come from desktop distributions (background updates/jobs), disk activity and USB,
not CPU. So: Debian minimal without desktop, no other services, klipper.service with Nice=-18, ModemManager removed,
automatic updates off, SSD instead of the old HDD, USB autosuspend off.
- 2026-11-15, owner asked to swap the N455 for an AMD Turion or Athlon 64 X2 he has: **not possible.** The Atom N455 is
  soldered (BGA) on an Intel NM10 board; AMD CPUs need an AMD socket and chipset. Not needed either (see above).
  If those CPUs sit in working laptops/PCs, that machine itself could be the host instead (laptop preferred: battery).

### NB250 measured by the owner (photos 2026-11-15)
- Lubuntu (LXQt), GRUB entry "Ubuntu, with Linux 6.11.0-17-generic" -> Ubuntu 24.04 HWE or 24.10 (os-release still to read).
- `uname -m` = **x86_64**. RAM **1.9 GiB** (already 2 GB, max for this model), swap 511 MiB. Disk /dev/sda1 229 G, 7.5 G used.
- Network: `enp9s0` (Ethernet) DOWN, `wlp7s0` (Wi-Fi) DOWN at the time of the photo.
- User `toshiba`, hostname `toshiba-toshibanb250`; password reset via recovery mode (root shell, passwd).
