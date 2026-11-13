<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Galaxy A51 (SM-A515F, Exynos 9611) as a Linux Klipper host? (researched 2026-11-13)

## Found
- **ahmedsaed/pmos-samsung-a51** (GitHub, README dated 2026-10-22): postmarketOS on an SM-A515F as a **headless
  homelab server** (DNS, Docker, Tailscale). Works: boot, SSH, Wi-Fi, Docker, **battery charge limit 80 %**,
  zram swap. Needed many local patches (initramfs, 4K-sector storage, kernel panic fix, runc crash).
  Device is **archived** in pmaports (maintainer inactive): edge channel only, nothing tested upstream, packages
  must be built locally. Kernel: downstream **4.14.113** (built from source -> CH341 driver could be enabled).
  Service manager **OpenRC** ("systemd needs newer kernel features than 4.14").
  Wi-Fi: incoming latency up to ~100 ms even with power saving off (accepted by the author).
- postmarketOS package device-samsung-a51: last built Dec 2024. ztOS-a51 port lists display/touch/Wi-Fi/charging working.
- No mainline Linux support for Exynos 9611 found; all kernels are Samsung downstream.
- Owner's phone today (verified earlier over adb): Android 13, locked bootloader, no root, stock kernel without CH341.

## Not found / unverified (deal-breakers for a printer host)
- **USB host (OTG) mode under postmarketOS** on this phone: not mentioned (the USB port is used as USB network gadget).
- **Charging while acting as USB host**: not verified (needs a powered OTG hub/Y-cable; many phones refuse).
- Nobody found running Klipper on an A51 under Linux.

## What it would take
Unlock bootloader (wipes the phone, trips Knox), TWRP, build pmOS with the patches, build a kernel with
CONFIG_USB_SERIAL_CH341 and OTG host, Klipper/Moonraker without systemd (KIAUH expects systemd -> manual OpenRC
services), then long test prints. Recovery from a bad flash is possible with Odin + stock firmware.

## Verdict
Technically plausible as a project, but many unverified links; the dedicated laptop with a battery already gives a
stable x86 host. Cheap pre-test on Android (no root): does the phone charge while a USB device is attached through a
powered OTG hub/Y-cable? If not, the phone cannot be a 24/7 USB host anyway.

## Other phones checked (2026-11-13)
| Phone | postmarketOS | Kernel | Notes |
|---|---|---|---|
| Galaxy A53 5G (a53x, Exynos 1280) | device + kernel package in **main** branch, built 2026-01-28 | downstream 5.10 fork | newest kernel of the three; no feature status page found; OTG/charging unknown |
| Galaxy S6 G920F (zerofltexx, Exynos 7420) | **testing**, fixes up to 2024 | downstream (Exynos7420 tree, LineageOS 17.1 base) | Wi-Fi fix was pending in 2021; 2015 phone, old battery |
| Huawei Mate 10 Pro (Kirin 970) | no port found | - | Huawei unlock service closed; PotatoNV does not support Kirin 970; only paid/unvetted tools |
Same open questions for all: USB host mode + charging at the same time under Linux, Klipper never reported on them.
