# Skyrim VR Mad God Overhaul on Linux
## WiVRn + OpenComposite Unleashed (OCU) + Quest controller fix

> ✅ **Confirmed working on CachyOS:** MGO 3.8.x, WiVRn 26.9-1, OpenComposite Unleashed (OCU), Quest 3, and Quest Touch Plus controllers.

This community guide documents a working way to run **Skyrim VR Mad God Overhaul (MGO)** on Linux using WiVRn and OCU. It is not an official MGO, WiVRn, OpenComposite, or Bethesda guide.

**Important update:** Downgrading WiVRn to 26.2.3 is **not required** for the working configuration documented here. The tested setup uses the current WiVRn 26.9-1 packages and OCU. The earlier downgrade procedure has been removed because it adds unnecessary steps for this fix.

---

## What this fixes

The original symptom was that MGO worked through SteamVR, but with WiVRn + xrizer only trigger and grip input worked; A/B, X/Y, and the thumbsticks did not. An older MGO-bundled OpenComposite mod also caused OpenXR session errors on the newer WiVRn setup.

Replacing that old bridge with **OpenComposite Unleashed (OCU)** made MGO launch normally and restored full Quest controller input in-game.

| Component / input | Verified result |
|---|---|
| WiVRn PC packages 26.9-1 | ✅ Working |
| Quest 3 Touch Plus controllers | ✅ Working |
| A / B buttons | ✅ Working |
| X / Y buttons | ✅ Working |
| Left and right thumbsticks | ✅ Working |
| Triggers and grips | ✅ Working |
| In-game controller input and menus | ✅ Working |
| Downgrade WiVRn to 26.2.3 | ❌ Not necessary for this setup |

The tested path is:

```text
Quest 3
  ↓
WiVRn (current tested PC packages: 26.9-1)
  ↓
WiVRn OpenXR runtime
  ↓
OpenComposite Unleashed (OCU) in the MGO profile
  ↓
Skyrim VR + SKSE + MGO
```

OCU is the game's OpenVR-to-OpenXR bridge for this setup. **Do not stack OCU with another OpenVR replacement/shim such as xrizer for the same game path.** This guide does not require downgrading WiVRn or sideloading an older WiVRn APK.

---

## Tested configuration

- **OS:** CachyOS (Arch-based Linux)
- **Game:** Skyrim VR
- **Modlist:** Mad God Overhaul (MGO 3.8.x)
- **PC streaming:** WiVRn 26.9-1 packages
- **Headset:** Meta Quest 3
- **Controllers:** Quest Touch Plus
- **OpenVR bridge used by MGO:** OpenComposite Unleashed (OCU)
- **Mod manager:** MO2 / Jackify-managed MGO setup
- **Result:** Full controller input confirmed in-game

The exact WiVRn and xrizer package versions can change over time. The key confirmed result is that the current WiVRn 26.9-1 setup works with OCU; **do not downgrade WiVRn just because the earlier workaround recommended it.**

---

# Quick Start

If Skyrim VR, Jackify/MGO, and WiVRn are already installed and WiVRn works with your headset:

1. **Install OpenComposite Unleashed (OCU)** from its Nexus page:  
   https://www.nexusmods.com/skyrimspecialedition/mods/171182

2. In MO2, enable OCU in your MGO profile.

3. Disable the old MGO mod:
   `Open Composite XR - Incompatible with VR Performance Kit`

   Do not leave the old bundled OpenComposite DLL active alongside OCU.

4. Keep **VR Performance Kit disabled** when using OCU. OCU's documentation explicitly warns that it is incompatible with VR Performance Kit and with other OpenVR replacement/shim implementations.

5. Do not enable another OpenVR bridge for the same game path. OCU is taking the place of xrizer for Skyrim VR's OpenVR-to-OpenXR translation in this configuration.

6. Keep WiVRn at the installed current version. The confirmed test used PC packages `26.9-1`; no WiVRn downgrade or old Quest APK was needed.

7. Launch MGO normally and test all controller inputs listed below.

> **Runtime note:** OCU needs a working OpenXR runtime. For this setup, WiVRn is the OpenXR runtime. If OCU cannot see the runtime inside Steam/Proton's container, first check runtime visibility rather than downgrading WiVRn.

---

# 1. Prerequisites

This guide assumes:

- Steam and Skyrim VR are installed and working on Linux.
- Jackify/MGO is installed and can be launched.
- WiVRn already connects to the Quest and works for VR streaming.
- MGO is launched through its normal MO2/Jackify workflow.
- You can enable and disable mods in the MGO MO2 profile.

If stock Skyrim VR or WiVRn itself does not work yet, get that baseline working first. This guide targets the case where WiVRn works but MGO's controller input or old OpenComposite startup is broken.

---

# 2. Install and enable OCU

Download **OpenComposite Unleashed for Skyrim VR** from Nexus:

https://www.nexusmods.com/skyrimspecialedition/mods/171182

Install it through MO2 into the MGO profile, then enable it.

OCU includes a replacement `openvr_api.dll`, its own OpenXR handling, and an optional/configurable controller-binding system. Use OCU's Configurator to review or choose a controller-binding preset if you need to change the controls.

Follow the mod's own documentation for the exact files and configurator options.

## Disable the old bundled OpenComposite mod

In MO2, disable:

```text
Open Composite XR - Incompatible with VR Performance Kit
```

That older mod was associated with the startup errors seen in the original setup, including an OpenXR view-configuration mismatch. With the old mod disabled, MGO launched again; installing OCU then restored full controller input.

**Only one OpenVR replacement should provide the game's bridge.** Do not allow the old mod's `openvr_api.dll` to override OCU's DLL.

## Keep VR Performance Kit disabled

OCU's documentation says not to use it with VR Performance Kit because they are incompatible and can crash. Keep the conflicting VR Performance Kit mod(s) disabled while testing OCU.

Also avoid stacking OCU with another OpenVR replacement/shim, including xrizer, for the same Skyrim VR game path. OCU is the bridge being tested here.

---

# 3. Keep WiVRn current

The working configuration tested the following PC packages:

```text
wivrn-server 26.9-1
lib32-wivrn-server 26.9-1
wivrn-dashboard 26.9-1
```

Check your installed package versions with:

```bash
pacman -Q | grep -i wivrn
```

**Do not follow the old WiVRn 26.2.3 downgrade procedure for this fix.** Downgrading the PC packages, installing a matching old Quest APK, handling ADB signatures, and rebooting solely for a version change are unnecessary when using the confirmed OCU configuration.

Keep the Quest WiVRn client and PC server compatible with one another as required by the installed release. If WiVRn reports a protocol mismatch, troubleshoot that mismatch separately rather than assuming an old version is needed for controller input.

---

# 4. Steam / Proton OpenXR runtime visibility

If OCU cannot find the WiVRn OpenXR runtime from inside Steam/Proton, the following launch option was used in the tested setup:

```bash
PRESSURE_VESSEL_IMPORT_OPENXR_1_RUNTIMES=1 %command%
```

Keep any additional launch options that your existing Jackify/MGO setup requires. Do not replace your full launch command with this one-variable example if it already contains required settings.

The full setup tested in this environment included the OpenXR runtime import and existing MGO-specific mount/capture variables. Those extra options are installation-specific and should not be copied blindly.

The runtime manifest installed by WiVRn is normally under:

```text
/usr/share/openxr/1/openxr_wivrn.json
```

Check that the file exists and points to the WiVRn OpenXR runtime if runtime discovery fails. Do not start by downgrading WiVRn.

---

# 5. Test every controller input

Launch MGO and verify the following at the menu and in-game:

```text
A / B buttons          ✓
X / Y buttons          ✓
Left thumbstick        ✓
Right thumbstick       ✓
Left and right trigger ✓
Left and right grip    ✓
In-game menus          ✓
```

The reported working setup passed all of these checks, including face buttons, both thumbsticks, triggers, grips, and in-game menus.

If one control is still missing, check OCU's controller-binding preset/configuration first and confirm that the old OpenComposite mod and conflicting VR Performance Kit are disabled. Change one thing at a time.

---

# Troubleshooting

## MGO crashes or shows an OpenXR view-configuration error

An older bundled OpenComposite build produced errors such as a view-configuration mismatch and a `viewCount == XruEyeCount` assertion in the previous setup.

1. Confirm `Open Composite XR - Incompatible with VR Performance Kit` is disabled.
2. Confirm OCU is enabled and its `openvr_api.dll` is the one being deployed.
3. Keep VR Performance Kit disabled with OCU.
4. Confirm the WiVRn OpenXR runtime is visible inside the Steam/Proton environment.

The tested fix was to disable the old bundled mod and use OCU. Downgrading WiVRn was not required.

## Triggers and grips work, but A/B, X/Y, or thumbsticks do not

This was the original controller symptom with MGO using WiVRn + xrizer. Stock Skyrim VR had working controllers, and MGO worked through SteamVR, but MGO input was incomplete through the old path.

The confirmed working change was to use OCU for the Skyrim VR bridge, with the old bundled OpenComposite mod disabled. Verify OCU's binding preset if controls still behave incorrectly.

Do not enable OCU and xrizer as two competing OpenVR replacements for the same game path.

## OCU cannot find an OpenXR runtime

Check that WiVRn is installed and its runtime manifest exists:

```bash
ls -l /usr/share/openxr/1/openxr_wivrn.json
ls -l /usr/lib/wivrn/libopenxr_wivrn.so
```

If you use Steam/Proton, try the launch option shown in section 4 to import OpenXR runtime manifests into Pressure Vessel. Keep the rest of your launch options intact.

## WiVRn reports “Incompatible protocol version”

This is a PC/Quest client-server version mismatch issue, not evidence that MGO needs WiVRn 26.2.3.

Check the PC package versions:

```bash
pacman -Q | grep -i wivrn
```

Then verify that the Quest app is compatible with the installed server release. Follow the current WiVRn release's instructions to update the mismatched side. Do not use the old 26.2.3 downgrade steps unless you are deliberately testing that historical version.

---

# Known-good result

```text
CachyOS                         ✓
WiVRn PC packages 26.9-1        ✓
Quest 3 streaming               ✓
MGO launches normally           ✓
OpenComposite Unleashed (OCU)   ✓
Old bundled OpenComposite       disabled
VR Performance Kit              disabled with OCU
A / B                           ✓
X / Y                           ✓
Left and right thumbsticks      ✓
Triggers and grips              ✓
In-game controller input        ✓
WiVRn downgrade required        no
```

This is a tested configuration, not a claim that every WiVRn, Proton, headset, or MGO version combination will behave identically. If you reproduce it on another system, report your distro, WiVRn client/server versions, headset, MGO version, OCU version, and controller results.

---

# Useful links

- **OpenComposite Unleashed for Skyrim VR (OCU):** https://www.nexusmods.com/skyrimspecialedition/mods/171182
- **WiVRn:** https://github.com/WiVRn/WiVRn
- **WiVRn releases:** https://github.com/WiVRn/WiVRn/releases
- **WiVRn AUR package:** https://aur.archlinux.org/packages/wivrn-server

---

## Credits / testing notes

This guide was updated after testing the original MGO controller issue on CachyOS with a Quest 3 and Quest Touch Plus controllers.

The key findings are:

- Stock Skyrim VR worked through WiVRn + xrizer.
- MGO initially had incomplete face-button/thumbstick input through that path.
- The old MGO-bundled OpenComposite implementation caused launch errors on the current WiVRn setup.
- Disabling the old mod allowed MGO to launch again.
- Installing OCU and keeping the old bundled mod disabled restored full controller input in MGO.
- WiVRn 26.9-1 worked in this configuration; downgrading WiVRn to 26.2.3 was unnecessary.

The purpose of this guide is to document the simpler working path so other Linux VR users do not have to repeat the unnecessary downgrade steps.
