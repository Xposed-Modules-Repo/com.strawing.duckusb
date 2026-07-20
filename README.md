# DuckUSB

An LSPosed / Xposed module that makes **scoped apps read USB debugging as OFF while
it stays really ON** on the device — and does the same for wireless debugging and
the Developer Options master toggle. It can also **hide the persistent "USB
debugging enabled" notification**.

Source, issues & builds: **https://github.com/Bouteillepleine/DuckUSB**

![DuckUSB UI](https://raw.githubusercontent.com/Xposed-Modules-Repo/com.strawing.duckusb/master/screenshot.png)

## How it works

Detection apps (banking, MDM/Intune, games, integrity checks) don't read any real
"adb" state — they query the settings provider:

```
Settings.Global.getInt(cr, "adb_enabled")                   // USB debugging
Settings.Global.getInt(cr, "adb_wifi_enabled")               // wireless / ADB-over-Wi-Fi
Settings.Global.getInt(cr, "development_settings_enabled")   // Developer Options
Settings.Secure.getInt(cr, "adb_enabled")                    // legacy (pre-4.2) location
```

DuckUSB hooks the static getters on `android.provider.Settings$Global` and
`android.provider.Settings$Secure` inside each **scoped** process. When the
requested key is one of the three above, it returns the "off" value. Every other
setting passes through untouched, and the device keeps debugging genuinely enabled
so `adb` still works for you.

The persistent *"USB debugging enabled"* notification is posted by `system_server`,
so a second, independent hook runs in the **System Framework** and **System UI**
processes and swallows the post, matched by channel and by the ROM's own localized
title strings (so any language matches).

## Features

- **Spoof USB debugging** — `adb_enabled` / `adb_wifi_enabled` / Developer Options
  read as off by your scoped detector apps.
- **Hide "USB debugging" notification** — needs System Framework + System UI scoped.
- **Two live toggles**, both on by default, read live via `XSharedPreferences` on
  every call — flipping them applies **without a reboot**.
- **Core OS is hard-skipped** for the spoof half: even if you scope the framework,
  it refuses to run in `android`, `com.android.settings`, `com.android.systemui`,
  `com.android.shell` and `com.android.phone`, so the real Settings toggle and
  `adbd` are never lied to.

## Install & scope

1. Install and enable **DuckUSB** in LSPosed.
2. Set the module **scope**:
   - Your detector apps (banking, MDM/Intune, games…) for the spoof.
   - **System Framework** (`android`) + **System UI** to also hide the notification.
3. Force-stop the scoped apps (or reboot) to apply.

## Credits

Author: **YxxX**.
