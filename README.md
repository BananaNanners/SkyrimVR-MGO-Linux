# Skyrim VR Mad God Overhaul on Linux
## WiVRn + OpenComposite + Quest controller workaround

> ✅ Confirmed working on CachyOS with MGO 3.8.2, WiVRn 26.2.3, OpenComposite, Quest 3, and Quest Touch Plus controllers.

> Community-tested workaround for running **Skyrim VR Mad God Overhaul (MGO)** on Linux with **WiVRn** and full Quest controller input.
>
> This is **not an official MGO, WiVRn, OpenComposite, or Bethesda guide**. It documents a working configuration discovered through testing on CachyOS/Arch Linux.

---

## What this fixes

A problematic setup looked like this:

| Setup | Result |
|---|---|
| SteamVR + MGO | ✅ Full controller input |
| WiVRn + xrizer + stock Skyrim VR | ✅ Full controller input |
| WiVRn + xrizer + MGO | ❌ Trigger/grip worked, but A/B, X/Y, and both sticks did not |
| Newer WiVRn + MGO OpenComposite | ❌ OpenComposite/OpenXR session errors |
| **WiVRn 26.2.3 + MGO OpenComposite** | ✅ **MGO runs and Quest controllers work normally** |

The working workaround is:

```text
Quest
  ↓
WiVRn 26.2.3 client
  ↓
WiVRn 26.2.3 server
  ↓
OpenXR
  ↓
MGO bundled OpenComposite
  ↓
Skyrim VR + SKSE + MGO
```

The important detail is that this guide uses **MGO's bundled Windows OpenComposite mod**, not the native Linux `/opt/opencomposite` runtime.

---

# Tested configuration

This guide was tested with:

- **CachyOS / Arch-based Linux**
- **Skyrim VR**
- **Mad God Overhaul 3.8.8.1**
- **WiVRn 26.2.3**
- **Quest 3**
- **Quest Touch Plus controllers**
- **Proton**
- **MGO's bundled `Open Composite XR - Incompatible with VR Performance Kit` mod**

The PC-side WiVRn packages used were:

```text
lib32-wivrn-server 26.2.3-1
wivrn-dashboard 26.2.3-1
wivrn-server 26.2.3-1
```

The matching upstream WiVRn tag is:

```text
v26.2.3
```

---

# Before you begin

This guide assumes:

1. Skyrim VR already launches on Linux.
2. MGO is already installed and can launch.
3. WiVRn already works for normal VR use.
4. You have `paru` or another AUR-capable workflow.
5. Your Quest has Developer Mode enabled if you plan to sideload the GitHub APK with `adb`.

If MGO itself does not launch yet, solve that first before applying this workaround.

---


# Quick Start

If you already have **Skyrim VR + MGO + Jackify + WiVRn** working and only need the controller/OpenComposite fix, this is the short version.

1. **Downgrade the PC-side WiVRn packages to 26.2.3** using the historical AUR `wivrn-server` package base:

```bash
mkdir -p ~/build/wivrn-downgrade
cd ~/build/wivrn-downgrade
git clone https://aur.archlinux.org/wivrn-server.git
cd wivrn-server
git checkout 3b44205
makepkg -Cfs
```

Install the three generated packages together:

```bash
sudo pacman -U \
  ./wivrn-server-26.2.3-1-x86_64.pkg.tar.zst \
  ./lib32-wivrn-server-26.2.3-1-x86_64.pkg.tar.zst \
  ./wivrn-dashboard-26.2.3-1-x86_64.pkg.tar.zst
```

Verify:

```bash
pacman -Q | grep -i wivrn
```

Expected:

```text
lib32-wivrn-server 26.2.3-1
wivrn-dashboard 26.2.3-1
wivrn-server 26.2.3-1
```

2. **Install the official WiVRn v26.2.3 Quest APK** from the GitHub release page:

```text
https://github.com/WiVRn/WiVRn/releases/tag/v26.2.3
```

Then install it with ADB:

```bash
adb install -r ~/Downloads/WiVRn-release.apk
```

If Android rejects the update because the existing app has a different signature:

```bash
adb uninstall org.meumeu.wivrn.github
adb install ~/Downloads/WiVRn-release.apk
```

Verify the Quest client:

```bash
adb shell dumpsys package org.meumeu.wivrn.github | grep -E 'versionName|versionCode'
```

Expected:

```text
versionName=26.2.3
```

3. **Reboot the Linux PC.**

If WiVRn reports `Incompatible protocol version` even though both sides show `26.2.3`, reboot before rebuilding or reinstalling anything.

4. In MGO/MO2, enable:

```text
Open Composite XR - Incompatible with VR Performance Kit
```

Keep these disabled:

```text
VR Performance Kit - FSR - Use with DLAA
VR Performance Kit - CAS Sharpening - Default
```

5. Leave WiVRn's native OpenVR compatibility layer on xrizer if that is your existing setup. **Do not switch WiVRn itself to `/opt/opencomposite` for this workaround.**

6. Launch MGO normally and test:

```text
A/B           ✓
X/Y           ✓
Left stick    ✓
Right stick   ✓
Trigger       ✓
Grip          ✓
```

If that works, continue using the detailed sections below only as reference/troubleshooting.

---

# Jackify / MGO installation prerequisites

Before applying the WiVRn/OpenComposite workaround, make sure your base Skyrim VR and MGO environment is healthy.

This guide assumes you are using **Jackify** to manage the Windows-based MGO setup on Linux.

## Base requirements

You should already have:

- Steam installed and working
- Skyrim VR installed through Steam
- Skyrim VR launched at least once before modding
- Proton configured for Skyrim VR
- Jackify installed
- Mad God Overhaul installed through Jackify
- MGO launching successfully before changing the VR runtime path
- WiVRn installed and able to run stock Skyrim VR
- `adb` available if you plan to sideload the GitHub WiVRn APK to a Quest headset

For Arch/CachyOS, useful base packages include:

```bash
sudo pacman -S --needed \
  git \
  base-devel \
  android-tools
```

If you use `paru` for AUR packages, make sure it is installed and working before starting the WiVRn downgrade.

## Recommended pre-checks

Launch **stock Skyrim VR** first and verify that:

```text
Head tracking       ✓
Left controller     ✓
Right controller    ✓
A/B                 ✓
X/Y                 ✓
Left stick          ✓
Right stick         ✓
Trigger             ✓
Grip                ✓
```

Then launch **MGO** through Jackify and verify that the game itself reaches the menu.

If MGO does not launch at all, fix that first. This guide is specifically for the case where MGO launches but controller input is incomplete under the WiVRn+xrizer path.

## Keep a working baseline

Before changing WiVRn versions or OpenComposite settings, record your current working state:

```bash
pacman -Q | grep -i wivrn
```

and save your current MGO profile state if you have customized it.

If you are using BTRFS/Snapper or another snapshot system, taking a snapshot before the downgrade is also a good idea.

## MGO-specific notes

For the final working configuration in this guide:

- MGO's bundled `Open Composite XR - Incompatible with VR Performance Kit` mod is enabled
- `VR Performance Kit - FSR - Use with DLAA` is disabled
- `VR Performance Kit - CAS Sharpening - Default` is disabled
- Skyrim VR's stock `openvr_api.dll` should not be manually replaced outside MGO's normal RootBuilder deployment
- WiVRn's native OpenVR compatibility layer can remain set to xrizer

This avoids stacking multiple OpenVR translation layers at the same time.

---

# 1. Downgrade WiVRn on the Linux PC to 26.2.3

The older AUR `wivrn-server` package base produced all three required packages:

```text
wivrn-server
lib32-wivrn-server
wivrn-dashboard
```

Create a working directory:

```bash
mkdir -p ~/build/wivrn-downgrade
cd ~/build/wivrn-downgrade
```

Clone the AUR package repository:

```bash
git clone https://aur.archlinux.org/wivrn-server.git
cd wivrn-server
```

Check out the historical AUR revision for WiVRn 26.2.3:

```bash
git checkout 3b44205
```

Verify:

```bash
grep -E '^(pkgver|pkgrel)=' PKGBUILD
```

Expected:

```text
pkgver=26.2.3
pkgrel=1
```

You can also verify that this package base builds all three packages:

```bash
git show 3b44205:.SRCINFO | grep -E '^(pkgbase|pkgname|pkgver|pkgrel)'
```

Expected package names include:

```text
wivrn-server
lib32-wivrn-server
wivrn-dashboard
```

Build the packages:

```bash
makepkg -Cfs
```

When the build finishes:

```bash
ls -lh *.pkg.tar.*
```

You should have packages similar to:

```text
wivrn-server-26.2.3-1-x86_64.pkg.tar.zst
lib32-wivrn-server-26.2.3-1-x86_64.pkg.tar.zst
wivrn-dashboard-26.2.3-1-x86_64.pkg.tar.zst
```

Install all three **together**:

```bash
sudo pacman -U \
  ./wivrn-server-26.2.3-1-x86_64.pkg.tar.zst \
  ./lib32-wivrn-server-26.2.3-1-x86_64.pkg.tar.zst \
  ./wivrn-dashboard-26.2.3-1-x86_64.pkg.tar.zst
```

Verify:

```bash
pacman -Q | grep -i wivrn
```

Expected:

```text
lib32-wivrn-server 26.2.3-1
wivrn-dashboard 26.2.3-1
wivrn-server 26.2.3-1
```

---

# 2. Install the matching WiVRn 26.2.3 Quest client

Download the **official GitHub WiVRn v26.2.3 APK** from:

https://github.com/WiVRn/WiVRn/releases/tag/v26.2.3

For the GitHub build, the Android package is normally:

```text
org.meumeu.wivrn.github
```

With the Quest connected over ADB:

```bash
adb devices
```

Try installing/updating the downloaded APK:

```bash
adb install -r ~/Downloads/WiVRn-release.apk
```

If Android refuses the update because the installed app was signed with a different certificate, uninstall the old GitHub build first:

```bash
adb uninstall org.meumeu.wivrn.github
```

Then install:

```bash
adb install ~/Downloads/WiVRn-release.apk
```

Verify:

```bash
adb shell dumpsys package org.meumeu.wivrn.github | grep -E 'versionName|versionCode'
```

Expected:

```text
versionName=26.2.3
```

---

# 3. Reboot the Linux PC

After downgrading WiVRn, **reboot the Linux PC before troubleshooting a client/server mismatch**.

This turned out to be important during testing.

If the Quest says:

```text
Incompatible protocol version
```

even though both sides show `26.2.3`, reboot the PC first.

Then restart WiVRn on the Quest and connect again.

You can watch the server log with:

```bash
journalctl --user -f | grep -iE 'wivrn|protocol|client|server'
```

Before the reboot, the server may report:

```text
Client connection failed: Incompatible protocol version
```

If both client and server really are on 26.2.3, do **not** immediately rebuild the Android app. Reboot first.

---

# 4. WiVRn OpenVR compatibility setting

For this guide, do **not** use the native Linux OpenComposite runtime as WiVRn's OpenVR compatibility layer.

During testing, native `/opt/opencomposite` could start WiVRn's OpenXR session but then crash through the Wine/Proton preloader path.

If you already use xrizer, leaving WiVRn's OpenVR compatibility layer on:

```text
/opt/xrizer
```

is fine.

The important part is that **MGO itself will use its bundled OpenComposite DLL**.

You do not need to replace Skyrim VR's stock `openvr_api.dll` manually.

---

# 5. Enable MGO's bundled OpenComposite mod

Open Mod Organizer 2 for MGO.

Enable:

```text
Open Composite XR - Incompatible with VR Performance Kit
```

Leave these disabled:

```text
VR Performance Kit - FSR - Use with DLAA
VR Performance Kit - CAS Sharpening - Default
```

Do not enable both OpenComposite and VR Performance Kit at the same time.

MGO's bundled OpenComposite mod should provide its own root files, including:

```text
openvr_api.dll
opencomposite.ini
```

through MGO/RootBuilder.

Do **not** manually overwrite Skyrim VR's stock `openvr_api.dll` unless you understand exactly how your MGO root deployment works.

---

# 6. Recommended OpenComposite configuration

The tested MGO OpenComposite configuration was:

```ini
supersampleRatio=1.0
renderCustomHands=true
enableInputSmoothing=true
inputWindowSize=3
disableTriggerTouch=false
```

Do not add this unless you specifically need it:

```ini
initUsingVulkan=false
```

It did not solve the session problem during testing.

The source file in MGO may be located under a path similar to:

```text
mods/Open Composite XR - Incompatible with VR Performance Kit/root/opencomposite.ini
```

RootBuilder may deploy this file only while MGO is running, so it can appear in the Skyrim VR root during launch and disappear afterward.

That is normal.

---

## 7. Launch MGO

Launch MGO with a Pressure Vessel override that exposes the host OpenXR runtime to the Steam container.

The preferred option is:

```bash
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 %command%

In the tested setup, launching without an OpenXR/Pressure Vessel override caused OpenComposite to fail during startup with:
xrEnumerateInstanceExtensionProperties(nullptr, 0, &availableExtensionsCount, nullptr)
Error code: -13

Adding:
```bash
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 %command%

allowed MGO to launch normally.

An alternative that also worked was exposing the WiVRn runtime socket directly:

```bash
PRESSURE_VESSEL_FILESYSTEMS_RW=/run/user/1000/wivrn %command%

Both variables together also worked:

```bash
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 PRESSURE_VESSEL_FILESYSTEMS_RW=/run/user/1000/wivrn %command%

For the cleanest setup, use PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 first.

If your normal Jackify/MGO setup already requires additional launch options, keep those as well.

Example:

PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 STEAM_COMPAT_MOUNTS="/home/bananananers/SkyrimVRMODMGO3.8/Modlist_Downloads" OBS_VKCAPTURE=1 %command%

Note: the exact STEAM_COMPAT_MOUNTS path depends on where your MGO downloads folder is located. Adjust it to match your own installation.

---

# 8. Test controller input

At the Skyrim VR menu and in-game, verify:

```text
A/B           ✓
X/Y           ✓
Left stick    ✓
Right stick   ✓
Trigger       ✓
Grip          ✓
```

If all six work, the workaround is working.

---

# Troubleshooting

## Quest says "Incompatible protocol version"

First verify the PC:

```bash
pacman -Q | grep -i wivrn
```

All three should show:

```text
26.2.3-1
```

Verify the Quest:

```bash
adb shell dumpsys package org.meumeu.wivrn.github | grep -E 'versionName|versionCode'
```

It should show:

```text
versionName=26.2.3
```

If both are correct:

1. Reboot the Linux PC.
2. Restart the WiVRn app on the Quest.
3. Try again.

This fixed the apparent protocol mismatch in the tested setup.

---

## MGO works with SteamVR but controller input is broken with WiVRn + xrizer

This was the original symptom.

Typical behavior:

```text
Trigger       works
Grip          works
A/B           broken
X/Y           broken
Left stick    broken
Right stick   broken
```

Stock Skyrim VR may still work perfectly through the same WiVRn+xrizer setup.

Use the WiVRn 26.2.3 + MGO OpenComposite configuration in this guide instead.

---

## OpenComposite popup: `changed->session == xr_session.get()`

A failing newer WiVRn/OpenComposite combination may produce an assertion similar to:

```text
Expression is false unexpectedly:
changed->session == xr_session.get()
```

Another related error observed was:

```text
viewCount == XruEyeCount
```

If you see these:

1. Verify WiVRn really is 26.2.3 on both PC and Quest.
2. Reboot the PC.
3. Confirm MGO's bundled OpenComposite mod is enabled.
4. Confirm you are not using native `/opt/opencomposite` as the game path.

---

## OpenComposite error `-13`

An OpenComposite build may fail very early with something similar to:

```text
xrEnumerateInstanceExtensionProperties(...)
Error code: -13
```

Check:

```bash
cat ~/.config/openxr/1/active_runtime.json
```

For WiVRn it should resolve to the WiVRn OpenXR runtime.

Also check:

```bash
ls -l /run/user/$(id -u)/wivrn
```

You should normally see WiVRn's IPC socket while the server is running.

Before adding more launch variables, confirm that your WiVRn client/server versions match and reboot the PC.

---

## MGO launches and immediately closes when WiVRn uses `/opt/opencomposite`

Do not use the native Linux OpenComposite runtime for this workaround.

The tested native runtime path:

```text
/opt/opencomposite
```

could initialize WiVRn's OpenXR runtime but then crash through the Wine/Proton preloader path.

Use:

```text
WiVRn
+ MGO bundled OpenComposite
```

instead.

---

## Check OpenComposite logs

Native OpenComposite logs may appear under:

```text
~/.local/state/OpenComposite/logs/opencomposite.log
```

A Proton-prefix copy may also appear under:

```text
.../pfx/drive_c/users/steamuser/AppData/Local/OpenComposite/logs/opencomposite.log
```

To find recent logs:

```bash
find ~ /tmp \
  -type f \
  \( -iname '*opencomposite*.log' -o -iname 'opencomposite.log' -o -iname '*openxr*.log' \) \
  -printf '%T@ %p\n' 2>/dev/null | sort -nr | head -30
```

---

# Prevent WiVRn from immediately updating

Arch/AUR helpers may offer WiVRn 26.9 or newer again.

Until you have personally confirmed that a newer WiVRn release works with MGO + OpenComposite, avoid updating these three packages:

```text
wivrn-server
lib32-wivrn-server
wivrn-dashboard
```

An optional temporary pacman pin can be added to `/etc/pacman.conf`:

```ini
IgnorePkg = wivrn-server lib32-wivrn-server wivrn-dashboard
```

Remember to remove the pin when you want to test a newer WiVRn release.

---

# Returning to current WiVRn later

When you want to leave the workaround and return to the current AUR packages:

```bash
paru -S wivrn-server lib32-wivrn-server wivrn-dashboard
```

Then install the matching current Quest client.

Do not mix a current Quest client with an old server or vice versa.

---

# Notes about MGO and Linux

This guide does not claim every MGO component is officially supported under Linux.

MGO includes many native SKSE VR plugins, graphics modifications, and VR-specific components. Some may behave differently under Proton.

The specific issue addressed here is:

> MGO launches and works, but Quest controller input is incomplete through WiVRn+xrizer, while SteamVR works correctly.

For the tested setup, switching to a matched WiVRn 26.2.3 client/server pair and using MGO's bundled OpenComposite restored normal controller input.

---

# Known-good result

Final tested result:

```text
WiVRn 26.2.3 PC server     ✓
WiVRn 26.2.3 Quest client  ✓
MGO 3.8.2                  ✓
MGO bundled OpenComposite  ✓
Quest Touch Plus tracking  ✓
A/B                        ✓
X/Y                        ✓
Left stick                 ✓
Right stick                ✓
Trigger                    ✓
Grip                       ✓
```

---

# Useful links

WiVRn:

https://github.com/WiVRn/WiVRn

WiVRn v26.2.3 release:

https://github.com/WiVRn/WiVRn/releases/tag/v26.2.3

WiVRn AUR package:

https://aur.archlinux.org/packages/wivrn-server

OpenComposite:

https://gitlab.com/znixian/OpenOVR

---

# Credits / testing notes

This workaround was developed through repeated A/B testing of:

- SteamVR vs WiVRn
- xrizer vs OpenComposite
- stock Skyrim VR vs MGO
- multiple WiVRn versions
- Quest controller behavior
- OpenComposite/OpenXR logs
- SKSE startup logs

The key observation was that **MGO itself was not fundamentally breaking controller bindings**, because the exact same MGO setup had full controller input under SteamVR.

The failure was specific to the WiVRn/xrizer/OpenComposite compatibility path, and a matched WiVRn 26.2.3 client/server pair resolved it.

---

## Contributions

If you test this on another distro, headset, GPU, Proton version, or newer WiVRn release, please report:

```text
Distro:
Kernel:
GPU:
Driver:
Proton:
WiVRn server version:
WiVRn Quest client version:
Headset:
MGO version:
OpenComposite enabled:
Controller result:
```

That will help determine how broadly reproducible this workaround is.
