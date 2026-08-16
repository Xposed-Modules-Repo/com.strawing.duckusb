# DuckUSB

Makes apps read **USB debugging as OFF while it stays really ON** — same for wireless debugging and Developer Options. Also spoofs the raw `sys.usb.*` properties and can hide the *"USB debugging enabled"* notification.

Source, issues & builds: **https://github.com/Bouteillepleine/DuckUSB**

![DuckUSB UI](https://raw.githubusercontent.com/Xposed-Modules-Repo/com.strawing.duckusb/master/screenshot.png)

## Scope

Tick the entry whose package is **`system`** — that is the one that injects into `system_server`, and framework mode needs it. **Not** the one whose package is `android`; that does not inject there, and picking it gives you a module that looks enabled and does nothing.

Add **System UI** for the notification hider, and individual apps only if you want the property spoof in them. **Reboot after scoping.**

## How it works

Detection apps don't read any real adb state — they query the settings provider for `adb_enabled`, `adb_wifi_enabled` and `development_settings_enabled`.

**Framework mode (recommended)** hooks the settings provider inside `system_server`, covering every app at once with no per-app scope. Callers at uid < 10000 (root/system/shell) always see the truth, so `adb` keeps working, and OS file-transfer components are spared so MTP is unaffected.

**Per-app mode** is the older client-side hook, for when you don't want `system_server` touched. The two are mutually exclusive; the UI enforces it.

A diagnostics card lists **every caller that was lied to since boot**, so a mis-scoped module can't masquerade as a working one.

## Toggles

Pause (live, stops everything) · Spoof USB debugging · Framework mode · Per-app spoof · Hide notification · Verbose logging.

Property spoofing is automatic in scoped apps — property reads are process-local, so it only works in apps you scope.

## Tested on

OnePlus 15 / OxygenOS / Android 16 with LSPosed + KernelSU. Framework mode is verified there and nowhere else.
