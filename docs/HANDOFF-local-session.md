<!-- Copyright (c) 2026 marioskiv. SPDX-License-Identifier: MIT -->
# Handoff: local session with the old smartphone (2026-11-09)

(The owner writes Greek; answer in Greek or English.)

## Context in 10 lines
- Project: Tronxy X5SA Pro, board CXY-V8-190729 (STM32F103, USART1 via CH340 USB, WiFi socket = USART3).
  Read `docs/STATUS.md`, `docs/facts.md`, `docs/plan-klipper-pi.md` first.
- Direction chosen by owner: **keep Klipper** (Marlin plan is only a fallback), but with a dedicated,
  stable host instead of the shared PC. No Raspberry Pi owned.
- Idea under test: use an **old smartphone (and/or old Samsung tablet) as the Klipper host / screen**.
- Current Klipper MCU firmware is built for **USART1 (USB/CH340)** -> a host connected by USB needs **no new flash**.
- Owner tried installing Linux on this phone before and failed (ask what exactly was tried).
- Known issues from research: stock Android kernels often lack the CH340 driver; many phones cannot charge
  while in USB host (OTG) mode; battery at 100 % 24/7 can swell.

## Rules (from the owner, keep them)
- Owner decision 2026-11-09: the old rule "no stm32flash / no writes to 0x08000000" was arbitrary and is **dropped**.
  Goal: make flashing as easy as possible (USB with BOOT jumper, stm32flash or SD, whichever is simplest).
  Claude still does not flash the printer itself; the owner flashes. Explain each step first.
- Explain every change first and wait for OK. Do not root, unlock the bootloader, flash a ROM or wipe the
  phone without explicit OK (unlocking the bootloader erases the phone).
- Separate verified facts from assumptions. Do not invent values. Say exactly what measurement/photo is needed.
- Commit findings to branch `claude/hopeful-hypatia-1agoqs` (docs/ folder). Every source file gets an MIT header.

## Task now: check the phone (read-only first)
Owner has the phone connected by USB to this computer. Needs: `adb` (platform-tools) and USB debugging on
(accept the "Allow USB debugging?" prompt on the phone).
1. `adb devices`; model, Android version, CPU ABI, RAM, storage, battery health/state.
2. Kernel: `uname -a`; `zcat /proc/config.gz | grep -E 'USB_SERIAL|CH341|CDC_ACM|USB_OTG|USB_HOST'`
   (file may not exist -> then check `lsmod`, `ls /lib/modules`, `/vendor/lib/modules`).
3. Root: `adb shell id`, `adb shell su -c id`. Bootloader: `getprop ro.boot.verifiedbootstate`,
   `ro.boot.flash.locked`, OEM unlock setting. SELinux mode.
4. Host mode: does the phone support OTG, and can it **charge while in host mode** (check
   `/sys/class/power_supply`, vendor docs, or test with a powered hub). Which USB-C/micro-USB port type.
5. With the **printer board plugged in** (ask owner; printer powered): look for the CH340 (vendor id 1a86)
   in `adb shell dmesg`, `/sys/bus/usb/devices`, `lsusb` if available. Report whether a `/dev/ttyUSB*` appears.
6. Search what Linux options exist for this exact model (postmarketOS, LineageOS+Termux/chroot, UserLAnd,
   octo4a-style apps with a CH34x driver). Report support level and risks.
7. Ask the owner what was tried before and where it failed.

## Deliver
- A clear verdict: (a) phone can be the Klipper host as is, (b) needs root/custom kernel/Linux install
  (list steps + risks + what gets wiped), or (c) not worth it -> buy a Raspberry Pi 3B+/4 (~30-60 EUR).
- Same for the old Samsung tablet if the owner connects it (as host or just as a Mainsail/KlipperScreen display).
- Write results to `docs/research/phone-host-check.md`, commit, push.

## Still open (not for this task, but keep in mind)
Bootloader offset of the current Klipper build (`grep FLASH_START ~/klipper/.config` on the Klipper PC),
K2 Plus thermistor values, HGX extruder data, WiFi-socket measurements. Hotend will use an external MOSFET module.
