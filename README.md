# XCent

Circuit-faithful emulation of the Yamaha DX100 4-operator FM synthesizer as a cross-platform audio plugin.

![Version](https://img.shields.io/badge/version-0.14.0--rc1-blue)
![License](https://img.shields.io/badge/license-Commercial-orange)
![Tests](https://img.shields.io/badge/tests-551%20passing-brightgreen)

## What is XCent?

XCent recreates the complete DX100 hardware signal chain — not a clean, idealized FM synth, but the *specific* sound of the hardware including all its character-defining "imperfections":

- **FM aliasing** — Native 55,930 Hz sample rate means sidebands fold back into the audible range
- **DAC graininess** — YM3014 floating-point DAC (10-bit mantissa + 3-bit exponent) quantization artifacts
- **Warm rolloff** — Analog reconstruction filter (-3 dB at ~10,600 Hz) with soft treble
- **Integer signal path** — All DSP computed at hardware bit widths (20-bit phase, 14-bit operators, 13-bit EG)

The FM engine runs at the native sample rate and resamples to the host rate — deliberately preserving the hardware's aliasing character.

## Features

### Core Synthesis
- **8 FM algorithms** — DX100's complete algorithm set with OP4 self-feedback
- **8-voice polyphony** — Oldest-voice stealing per DX100 spec
- **Cycle-accurate YM2164 emulation** — Nuked-OPM core for bit-perfect FM engine
- **HD6303 CPU emulator** — Runs actual DX100 ROM firmware for 100% accurate voice allocation and parameter handling
- **Authentic analog stage** — YM3014 DAC + NJM072D reconstruction filter emulation

### Interface
- **React + Tailwind UI** — Modern, responsive interface via JUCE WebView
- **3 editing modes:**
  - **EZ Edit** — Visual knobs and sliders for fast sound design
  - **Parameter Grid** — Full DX100 parameter set (63 params per voice)
  - **Function Mode** — System settings, MIDI config, portamento
- **16-character dot-matrix LCD** — Authentic DX100 display with Teal or OLED Blue styles
- **On-screen keyboard** — 25-key virtual keyboard with velocity
- **Live CPU and VU meters**

### Sound Management
- **4 banks × 8 patches** — Per-bank storage (A-D factory, U1-U4 user)
- **Favorites & Recents** — Quick access to frequently-used patches
- **SysEx Import** — Load DX100 VCED (single) and VMEM (32-voice bank) files
- **User patch library** — Store patches on disk, drag-and-drop patch organization

### MIDI & Performance
- **Full MIDI CC support** — Mod wheel, breath control, sustain, pitch bend
- **MIDI channel select** — Omni or channels 1-16
- **Vintage/Modern engine toggle** — Bypass analog output stage for cleaner sound
- **Portamento** — Full-time and fingered modes with adjustable time
- **Pitch bend range** — Per-patch configuration (1-12 semitones)

### Accessibility
- **WCAG 2.1 Level AA compliant** — Screen reader support, keyboard navigation, focus management
- **Dark/Light themes** — Full theme system with instant switching

## Formats

- **VST3** (Windows, macOS, Linux)
- **CLAP** (Windows, macOS, Linux)
- **Standalone** (Windows, macOS, Linux)

## DAW Compatibility

Results from hands-on testing toward v1. "Known-good" means all ten core
host-integration checks pass (plugin scan, instantiate, MIDI→audio,
automation, save/reload, multi-instance, offline render, window
close/reopen, resize, CPU idle). See
[`Docs/daw-compatibility.md`](Docs/daw-compatibility.md) for the full
test tracker.

Legend: ✅ known-good · ⚠️ works with caveats · 🚧 in progress · ⬜ not yet tested · ❌ not working · — not applicable

| DAW               | Platform | VST3 | CLAP | Version tested | Notes |
|-------------------|----------|:----:|:----:|----------------|-------|
| Ableton Live      | Windows  | 🚧   | —    | —              | In progress |
| FL Studio         | Windows  | ⬜   | —    | —              | Planned next |
| Reaper            | Windows  | ⬜   | ⬜   | —              | Planned |
| Bitwig Studio     | Windows  | ⬜   | ⬜   | —              | If installed |
| Cubase            | Windows  | ⬜   | —    | —              | If installed |
| Studio One        | Windows  | ⬜   | —    | —              | If installed |
| Standalone        | Windows  |  —   | —    | —              | Quick pass |
| Logic Pro         | macOS    | —    | —    | —              | Deferred (Mac access required) |
| Cubasis / AUM     | iOS      | —    | —    | —              | AUv3 — real-device pass pending |

## Building

### Requirements

- **CMake 3.22+**
- **C++17 compiler** (MSVC 2019+, GCC 9+, or Clang 10+)
- **JUCE 8.0+** — Place at `../JUCE` or set `JUCE_DIR` environment variable
- **Node.js 18+** — For UI builds

### Quick Build

```bash
# Configure
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build all formats
cmake --build build --config Release --parallel

# Artifacts are in:
# build/XCent_artefacts/Release/VST3/XCent.vst3
# build/XCent_artefacts/Release/CLAP/XCent.clap
# build/XCent_artefacts/Release/Standalone/XCent.exe
```

### UI Development

```bash
# Build UI only (React/Tailwind)
cd src/ui
npm install
npm run build
```

### Tests

```bash
# Run full test suite (511 cases, 16,602 assertions)
build/Release/XCent_Tests.exe    # Windows
./build/XCent_Tests              # macOS/Linux
```

All DSP verified bit-exact against [Nuked-OPM](https://github.com/nukeykt/Nuked-OPM).

## Installation

### Windows

```powershell
# VST3
Copy-Item "build/XCent_artefacts/Release/VST3/XCent.vst3" -Destination "$env:ProgramFiles\Common Files\VST3" -Recurse

# CLAP
Copy-Item "build/XCent_artefacts/Release/CLAP/XCent.clap" -Destination "$env:ProgramFiles\Common Files\CLAP"
```

### macOS

```bash
# VST3
cp -r "build/XCent_artefacts/Release/VST3/XCent.vst3" ~/Library/Audio/Plug-Ins/VST3/

# CLAP
cp "build/XCent_artefacts/Release/CLAP/XCent.clap" ~/Library/Audio/Plug-Ins/CLAP/

# AU (if built)
cp -r "build/XCent_artefacts/Release/AU/XCent.component" ~/Library/Audio/Plug-Ins/Components/
```

### Linux

```bash
# VST3
cp -r "build/XCent_artefacts/Release/VST3/XCent.vst3" ~/.vst3/

# CLAP
cp "build/XCent_artefacts/Release/CLAP/XCent.clap" ~/.clap/
```

## Architecture

```
MIDI → HD6303 CPU (ROM) → YM2164 (FM) → YM3014 (DAC) → NJM072D (Filter) → Resampler → Host
```

### Source Layout

```
src/
  firmware/       HD6303 CPU emulator + ROM execution
  fm_engine/      NukedEngine wrapper (production) + TDD stubs (deprecated)
  analog/         Reconstruction filter + DC blocking
  audio/          Resampling (55930 Hz → host sample rate)
  patch/          SysEx parser + patch management
  ui/             React + Tailwind WebView UI
tests/            Catch2 test suite (511 cases)
third_party/      Nuked-OPM + dependencies
Docs/             Hardware specs + reverse engineering notes
```

## Documentation

- **[User Manual](Docs/manual/MANUAL.md)** — Complete user guide with parameter reference and algorithm diagrams
- **`CLAUDE.md`** — Developer guide for AI assistants
- **`CHANGELOG.md`** — Version history
- **`Docs/specs/`** — Hardware specification documents (clock tree, register maps, envelope timing)
- **`Docs/reverse-engineering/`** — ROM analysis and KC register investigation
- **`accessibility-audit.md`** — WCAG 2.1 accessibility audit results
- **`bugs.md`** — Known issues

## Status

**Current version:** v0.15.1-rc2 (May 2026)

**Feature complete for v1** — in DAW compatibility testing and beta. Hardware-validation pass complete (S/H LFO, saw/square/triangle LFO, KVS velocity, full-keyboard chromatic, portamento curve all measured against a real DX100 and closed).

**Recent updates (v0.15.1-rc2 — portamento calibration + hardware validation):**
- ✅ DEFER18 — Portamento glide rate calibrated to hardware (mean dGlide 21 ms); mono+legato triggering matches DX100
- ✅ S/H LFO, KVS velocity sensitivity, and Portamento Curve entries in KNOWN-LIMITATIONS promoted to RESOLVED
- ✅ Wider-chromatic DSP18 closed — 61/61 notes within ±5 cents vs hardware across C2..C7
- ✅ Saw / square / triangle LFO match hardware on rate + amplitude
- ✅ Installer pipeline supports suffixed versions (rc/beta) — `XCent-Setup-{version}.exe` matches `src/ui/package.json` exactly

**Recent updates (v0.15.0-rc2 — TEL1 + macOS installer + output-stage Phase A):**
- ✅ Anonymous opt-out telemetry + GitHub-Issue report form (gated behind `XCENT_NETWORK_ENABLED`)
- ✅ HMAC-SHA256 URL signing for the report form and kos-worker API
- ✅ MIDI CC11 (Expression) as a multiplicative scale on top of CC7
- ✅ macOS DIST2 installer — signed `.pkg` wrapped in `.dmg` with Welcome / licence screens
- ✅ TX81Z / FB-01 / DX21 output-stage classes implemented (Phase A; hardware calibration deferred)
- ✅ UI39 bank/slot vs VCED state desync detection on project reload

**Recent updates (v0.14.0-rc1 — onboarding & UI polish):**
- ✅ First-run onboarding tour (UI43) — spotlight walkthrough, re-runnable
- ✅ Parameter button hover tooltips — Edit/Func grid + Modern view knobs
- ✅ "Default to Modern view" setting
- ✅ Modern view (formerly "EZ mode") terminology in user-facing copy
- ✅ Floating UI (tooltips, context menus, copy-to menus, tag picker) — root-cause
  fix for off-screen drift at non-100% UI scale (portal to document.body)
- ✅ Redesigned standalone app icon — algorithm tree fills the canvas, higher contrast

**Recent updates (v0.12.x — DSP & DAW compatibility):**
- ✅ DSP12/DSP13 Pitch EG — ROM-extracted rate stepping + level scaling
- ✅ DSP21 +3 dB analog calibration with semantic split (passband + output cal)
- ✅ DSP19a ROM-extracted PMD/AMD scaling tables
- ✅ DSP26/DSP27/DSP28 Breath Controller EG bias formula corrected
- ✅ Code-standards audit (CODE11) — 7 RT-safety + input-validation fixes
- ✅ Nuendo / VST3 plugin-guard WebView resource URL fix
- ✅ 551/554 tests passing (2 documented pre-existing fixture-isolation failures)

**Recent updates (v0.11.x):**
- ✅ Mouse wheel editing on Modern-view knobs and sliders (UI36)
- ✅ SysEx export — six destination formats (DX100/DX21/TX81Z × VCED/VMEM)
- ✅ Auto-tagger v2 — 12 FM features + algorithm allowlists
- ✅ NSIS Windows installer (VST3 + CLAP + Standalone + manual)
- ✅ WebView2 runtime detection + native fallback panel
- ✅ Standalone settings land under `Documents/KnivesOnStrings/XCent/`

See `CHANGELOG.md` for detailed version history.

## License

XCent is a **commercial product** sold via [Gumroad](https://knivesonstrings.com/xcent). The compiled binary (VST3, CLAP, AU, AUv3, Standalone) is licensed to end users under the XCent End User Licence Agreement included with the installer (`LICENSE.txt` / `EULA.txt`). All rights reserved.

This is the private development repository. The XCent product source is **not** open source; this repository is mirrored only to internal collaborators. See [LGPL Compliance](#lgpl-compliance) for how end-user rights to the LGPL-covered third-party components are satisfied.

### LGPL Compliance

The XCent binary statically links **Nuked-OPP** (in `third_party/nuked-opp/`), a *modified* fork of Nuke.YKT's Nuked-OPM with DX100/YM2164-specific behavioural tweaks and renamed symbols. It is licensed under the GNU Lesser General Public License v2.1 (LGPL-2.1-or-later).

Because Nuked-OPP is modified, end-user LGPL §6 obligations cannot be satisfied by linking only to the unmodified upstream — the modified source must be made available. The complete `third_party/nuked-opp/` source for each released XCent version is mirrored at the public Knives On Strings org on GitHub (see Acknowledgments below for the link); end users may also obtain the source on request via [knivesonstrings@gmail.com](mailto:knivesonstrings@gmail.com).

`third_party/nuked-opm/` (Nuke.YKT's *unmodified* Nuked-OPM) is built and used only by the test target `XCent_Tests` as an upstream reference. It is not linked into any shipped binary and therefore does not impose end-user LGPL obligations on the XCent product, but its copyright notice is preserved in the source headers and credited in the manual.

## Acknowledgments

- **Nuked-OPP** — Modified fork of [Nuked-OPM](https://github.com/nukeykt/Nuked-OPM) with DX100/YM2164-specific behavioural tweaks. Original by Nuke.YKT, modifications © Knives On Strings (LGPL 2.1). Production engine. Modified source mirrored at the Knives-On-Strings public org on GitHub (link in installer NOTICES.txt).
- **[Nuked-OPM](https://github.com/nukeykt/Nuked-OPM)** by Nuke.YKT — Die-shot-verified YM2151 reference emulator (LGPL 2.1). Used only as an upstream test-comparison reference; not linked into the shipped product.
- **[JUCE](https://juce.com/)** — Cross-platform audio framework
- **[clap-juce-extensions](https://github.com/free-audio/clap-juce-extensions)** by Paul Walker — CLAP format support
- **[Catch2](https://github.com/catchorg/Catch2)** — C++ testing framework
- **[React](https://react.dev/)** + **[Tailwind CSS](https://tailwindcss.com/)** — UI framework
- **[Inter](https://rsms.me/inter/)** by Rasmus Andersson — UI typeface (SIL Open Font License 1.1)

Built with [Claude Code](https://claude.ai/code).

## Trademarks

"Yamaha" and "DX100" are trademarks of Yamaha Corporation. XCent is an
independent emulation, not affiliated with or endorsed by Yamaha.

ASIO is a trademark and software of Steinberg Media Technologies GmbH.

VST is a trademark of Steinberg Media Technologies GmbH.

## Links

- **Website:** https://knivesonstrings.com/xcent
- **GitHub:** https://github.com/Knives-On-Strings/xcent
- **Manual:** [User Manual](Docs/manual/MANUAL.md)
- **Issues:** https://github.com/Knives-On-Strings/xcent/issues
