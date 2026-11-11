# GiveMiiFont

Temporarily replace the Wii U system font (RAM only, NAND is never touched).

Two parts:

- **Plugin** (`plugin/`, Aroma `.wps`): hooks `OSGetSharedData` and serves the chosen TTF
  instead of the Standard system font. Has a toggle in the Aroma plugin menu.
- **App** (`app/`, `.wuhb`): pick, preview and equip fonts.

## Status

**Untested.** Written without a Wii U toolchain or hardware available, so it has not been
compiled or run. Expect to fix small API/build mismatches (see "Things to verify").

## Install

1. Copy `givemiifont.wps` to `SD:/wiiu/environments/aroma/plugins/`.
2. Copy `givemiifont.wuhb` to `SD:/wiiu/apps/`.
3. Put `.ttf` files in `SD:/wiiu/fonts/`.

## Use

- Run GiveMiiFont. A notice appears first (text may glitch, restart the Wii U to fix it,
  nothing is written to the NAND); press **+** to dismiss it.
- **Up/Down** select, **Y** preview, **X** equip/unequip. A and B do nothing.
- Equipping writes the file name to `SD:/wiiu/fonts/givemiifont.cfg`.
- The plugin reloads that file **every time an application starts**, so the font changes the
  next time the Wii U Menu or a game launches. No reboot needed.
- Aroma plugin menu (**L + Down + SELECT**) -> GiveMiiFont:
  - **Enabled**: master toggle.
  - **Wii U Menu only** (default: yes): limits the replacement to the Wii U Menu, since some
    software (on-screen keyboard, System Settings, ...) can crash with unusual fonts.
  - **Reset font on reboot (safety)** (default: yes): see "Safety net" below.

## Build

Plugin (needs devkitPro with wut, wups, wums):

    cd plugin && make

App (needs `wiiu-sdl2`, `wiiu-sdl2_ttf` from devkitPro pacman):

    cd app
    mkdir build && cd build
    $DEVKITPRO/portlibs/wiiu/bin/powerpc-eabi-cmake ..
    make

## Safety net: the font resets on every restart

With **Reset font on reboot** on (the default), an equipped font never survives a restart.
If a font crashes something and you can't get to the plugin menu, just power the console
off and on (or restart it) and the system font is back.

How it works: plugin memory is cleared on every boot. On the first application start after
boot, the plugin deletes the leftover `givemiifont.cfg` and applies nothing. You have to
press X in the app again to re-equip. If the SD card can't be checked yet at that moment,
the plugin applies no font at all (it fails closed).

Notes:
- This is "restart" in the sense of the plugin being reloaded, which happens on a full
  power cycle or restart. I have not verified whether sleep/standby counts; assume it
  doesn't reset anything.
- If you turn this option off, the equipped font persists across restarts until you
  unequip it, so keep another recovery path in mind (below).

## Other recovery options

Turn **Enabled** off in the plugin menu, or delete `SD:/wiiu/fonts/givemiifont.cfg`
from a PC, or remove `givemiifont.wps` from the plugins folder. The original system
font is always intact in the console.

## Limits

- Fonts must be plain TrueType and at most 2.5 MiB.
- Fonts without the Private Use Area button glyphs (U+E000...) may show blanks where apps
  draw controller icons. The preview warns about this.
- GamePad only for input.

## Things to verify first

- WUPS config API names in `plugin/src/main.cpp` (`WUPSConfigItemBoolean_AddToCategory`,
  callback result constants, `WUPSStorageAPI`) against your installed WUPS version.
- `plugin/Makefile` against the official WUPS example plugin Makefile.
- That `fs:/vol/external01/...` is readable from the plugin at application start.
- `wut_create_wuhb` options and SDL2_ttf static link flags in `app/CMakeLists.txt`.
- On hardware: whether the Wii U Menu picks up the font as expected.
