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
| Galaxy A53 5G (a53x, Exynos 1280) | packages built 2026-01-28, but the port now sits in **device/archived** (see below) | downstream 5.10 fork | details in the A53 section below |
| Galaxy S6 G920F (zerofltexx, Exynos 7420) | **testing**, fixes up to 2024 | downstream (Exynos7420 tree, LineageOS 17.1 base) | Wi-Fi fix was pending in 2021; 2015 phone, old battery |
| Huawei Mate 10 Pro (Kirin 970) | no port found | - | Huawei unlock service closed; PotatoNV does not support Kirin 970; only paid/unvetted tools |
Same open questions for all: USB host mode + charging at the same time under Linux, Klipper never reported on them.

## Galaxy A53 (samsung-a53x): port files and kernel config (researched 2026-11-13)
Sources: postmarketOS hosts (wiki, pkgs, gitlab.postmarketos.org) are blocked from the cloud session, so the port
files were read from the GitHub mirror `external-mirrors/pmaports` (raw files, master), and the kernel defconfig from
the source repo the port builds: `Gabriel2392/android_kernel_samsung_a53x_xy` @ b155b4e2,
`arch/arm64/configs/s5e8825-a53xxx_defconfig` (header: Linux/arm64 5.10.177).

### Verified (read in the files)
- Port is in **`device/archived/`**, APKBUILD says `# Archived: unmaintained`, `maintainer=""`.
  (Older mirrors list it under `device/downstream/`; it was moved later.) Archived = no upstream testing, build it yourself.
- deviceinfo: flash method `heimdall-bootimg` (Samsung download mode), rootfs to `system` partition.
  kernel-cmdline.conf is only `rw`. No firmware subpackage, no extra device subpackages.
- Kernel APKBUILD comment: "Kernel config based on make defconfig" -> it uses **Samsung's Android defconfig unchanged**
  (only adds SECTION_MISMATCH_WARN_ONLY). No postmarketOS config fragment.
- In that defconfig:
  - `CONFIG_USB_SERIAL_CH341` **is not set** (the Tronxy board's USB chip is a CH340). `CONFIG_USB_SERIAL=m`,
    `CONFIG_USB_ACM=y` (ACM would only help a board with native USB, not the CH340).
  - USB host hardware is there: `CONFIG_USB_OTG=y`, `CONFIG_USB_XHCI_HCD=y`, `CONFIG_USB_DWC3=y`,
    `CONFIG_USB_DWC3_DUAL_ROLE=y`, `CONFIG_USB_HOST_NOTIFY=y`.
  - The host/device role switch and charging go through Samsung drivers built as **modules**:
    `CONFIG_PDIC_SM5714=m`, `CONFIG_MUIC_SM5714=m`, `CONFIG_CHARGER_SM5714=m`, `CONFIG_FUELGAUGE_SM5714=m`,
    `CONFIG_BATTERY_SAMSUNG=m`, `CONFIG_USB_NOTIFY_LAYER=m`, `CONFIG_USB_TYPEC_MANAGER_NOTIFIER=m`.
  - `# CONFIG_DEVTMPFS is not set`, `# CONFIG_VT is not set`, `# CONFIG_FHANDLE is not set`,
    `# CONFIG_SYSVIPC is not set`.
  - Wi-Fi is Samsung SCSC (`CONFIG_SCSC_WLAN=m`) with firmware expected in `/vendor/etc/wifi`
    (`CONFIG_SCSC_CORE_FW_LOCATION`), and the port packages no firmware.

### Inferences (not verified)
- postmarketOS requires DEVTMPFS (its initramfs and udev use it) and normally VT/FHANDLE/SYSVIPC; with the stock
  Android defconfig the port most likely reached only an early boot stage. This would also explain "unmaintained".
- Even with a fixed config: USB host mode and charging depend on the SM5714 PDIC/MUIC/charger modules loading
  correctly outside Android; nothing shows this was ever tested. Wi-Fi needs firmware copied from the phone's vendor
  partition by hand.
- Nobody found running Klipper (or any USB serial host) on an A53 under Linux.

### What it would take on the A53
1. Unlock bootloader (wipes phone, Knox tripped), TWRP or heimdall.
2. Build the kernel yourself with a fixed config: DEVTMPFS, VT, FHANDLE, SYSVIPC, USB_SERIAL_CH341 (+ pmbootstrap
   kconfig check), make sure the SM5714 modules load at boot.
3. Copy Wi-Fi firmware from `/vendor` into the rootfs.
4. Test USB host mode + charging with a powered hub, then Klipper, then long prints.
This is kernel porting work, not an install. Estimate: days to weeks, uncertain result.

### Verdict (A53)
Worse than it looked from the package list: the port is archived and its kernel config is not prepared for
postmarketOS at all. Not recommended as the printer host. The Android pre-test (charging + OTG at the same time)
is still worth doing only if the owner wants a hobby project.
