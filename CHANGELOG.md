# XCent Changelog

## v0.15.1-rc2 — 2026-05-14 — Portamento calibrated; hardware-validation pass; installer pipeline supports rc/beta versions

### Fixes

- **Portamento now matches DX100 hardware (DEFER18)** — Plugin glide
  was previously ~2.2× slower than hardware at every portaTime, and
  engaged under more conditions than the real device. Same-day fix
  after the 2026-05-14 hardware sweep: rate scaled by 22/10 in
  `FirmwareLogic::computePortaRate` and triggering tightened to
  mono + legato. Post-fix sweep matches hardware to a mean |dGlide|
  of 21 ms across portaTime 10..75. Trigger behaviour now lines up
  with hardware: portamento only engages when `polyMono=1` AND a
  previous voice is still held when the new note arrives. `portaMode`
  (full-time vs fingered) is preserved for SysEx round-tripping but
  no longer changes triggering (DX100 treats both modes identically).

### Build / distribution

- **Installer pipeline supports suffixed versions** — `stage_release.py
  --platform windows --build` now produces `XCent-Setup-{version}.exe`
  matching `src/ui/package.json` exactly, including `-rc` / `-beta`
  suffixes. Previously the installer fell back to a synthesised
  numeric string (`MAJOR.MINOR.BUILD_NUMBER`) and the stager couldn't
  find the artifact at the expected filename.

### Verified

- **Portamento curve verified against hardware (DEFER18-validation).**
  18 captures total (9 portaTime values × HW + plugin). Curve shape
  already matched hardware before the fix (both quasi-exponential,
  glide doubling every +23 CC5 units); post-fix the rate constant
  matches too. See `Docs/measurements/2026-05-14-portamento-curve.md`.
- **KVS velocity sensitivity matches hardware exactly** — 36 plugin
  captures vs 36 hardware captures across KVS ∈ {0, 3, 5, 7} × velocity
  ∈ {1..127, 9 points}. Mean |peak delta| 0.37 dB; mean |sustain-RMS
  delta| 0.12 dB. 36/36 |dRMS| < 3 dB. Velocity → TL routing through
  `kVelocityCurves` is hardware-exact at every KVS value (closed in
  `Docs/KNOWN-LIMITATIONS.md`).
- **Chromatic pitch accuracy verified end to end (DSP18 wider range)**
  — 61-note sweep MIDI 36..96 (5 octaves). Hardware and plugin both
  track equal-temperament with mean error inside ±0.5 cents; plugin
  vs hardware mean delta −0.13 cents, max 1.44 cents. 61/61 captures
  within ±5 cents. DSP18 promoted to `[x]`.
- **Saw / square / triangle LFO match hardware on amplitude + rate**
  — 30 plugin vs 30 hardware captures across the three deterministic
  LFO waveforms × 5 LFS values × 2 isolation patches. Amplitude path
  matches everywhere (14/15 LSD < 0.3); pitch path matches fundamental
  rate and total mod power on every waveform but shows harmonic-shape
  divergence on triangle/saw (filed as DEFER17, the OPP/OPM PMS-curve
  fingerprint).
## v0.15.0-rc2 — 2026-05-12 — TEL1 first soft-launch end-to-end on macOS; output-stage Phase A

### Fixes

- **TEL1 host_version on Standalone (macOS)** — `/api/issues` was rejecting
  every form submission with `host_version is required`. Plugin now sends
  the OS version (e.g. `"macOS 14.5"`, `"Windows 11"`) via
  `juce::SystemStats::getOperatingSystemName()` for both Standalone and
  DAW contexts — JUCE doesn't expose the host application's version
  separately, but the OS string is a useful triage signal that is always
  non-empty.
- **TEL1 worker-thread crash on quit (macOS)** — `XCent quit unexpectedly`
  SIGSEGV inside `objc_autoreleasePoolPop` during process teardown. The
  telemetry worker accumulated Cocoa autorelease objects from each POST
  into its outer JUCE-managed pool; on shutdown that pool teardown raced
  with Cocoa finalisation and dereferenced freed memory. Fix wraps each
  drain + each POST in a scoped `JUCE_AUTORELEASEPOOL` (releases
  transient objects per-iteration) and tightens the POST timeout from
  5s to 1.5s so the worker can finish in-flight requests before being
  abandoned by the 2s join window. Same fix applied to
  `IssueReporter::ping()`.
- **TEL1 `session_end` lost on clean quit** — The telemetry singleton's
  destructor only ran during C++ static finalisation, by which point the
  UserPreferences singleton holding `install_id` had already been torn
  down. The worker's final drain read an empty install_id and the event
  silently dropped. Fix adds a `PluginProcessor`-level instance counter
  that deterministically calls `TelemetryClient::stop()` from the
  last-live-instance destructor — while prefs are still alive — so
  session_end actually POSTs.
- **Standalone state persistence on macOS** — Settings made during a
  session (theme, scale, displayStyle) were lost on close. Worked on
  Windows. Two compounding bugs:
  - **`shutdown()` ordering** — Our custom `XCentStandaloneApp::shutdown()`
    destroyed `mainWindow` before ever calling `savePluginState()`, so
    `pluginProperties` retained whatever stale `filterState` was last
    cached. `savePluginState()` is only triggered automatically by
    `systemRequestedQuit()`, which on macOS doesn't fire when the user
    clicks the window's red close button (only Cmd+Q routes that way).
    Windows routes window close through systemRequestedQuit, which is
    why this never surfaced there. Fix calls `savePluginState()`
    explicitly in `shutdown()` before destruction.
  - **Foreign children accumulating in filterState** — `setStateInformation`
    calls `apvts.replaceState(ValueTree::fromXml(...))`, which adopted
    our `XCentVoice` / `XCentRom` / `Scope` / `XCentPrefs` nodes into
    the APVTS state tree. Each subsequent save reproduced them via
    `copyState()`, then we *appended* a fresh copy on top — duplicates
    compounded every cycle, and `getChildByName` returned the oldest
    stale match on load. Fix scrubs known foreign-child tags from the
    APVTS XML before re-appending fresh ones; one user's `filterState`
    went 8705 bytes → 664 bytes after the fix.

### Features

- **HMAC-signed report-form & API URLs** — The hosted issue-report form
  (`report.knivesonstrings.com`) and the kos-worker API endpoints
  (`/api/ping`, `/api/telemetry`, `/api/issues`) now require an
  HMAC-SHA256 signature on every request. The plugin signs each URL
  with a shared secret compiled in via CMake
  (`XCENT_FEEDBACK_FORM_SECRET`); the server recomputes the signature
  and 302-redirects to `www.knivesonstrings.com` (form) or returns 401
  (API) when it doesn't match. Drive-by visits to the form URL or
  random `curl` against the API are no longer enough to reach the
  endpoints — you have to know the secret. Replay window is ±300 s
  via a signed `ts` param. The secret is a spam moat, not a real
  credential — rotate by bumping the CMake var + worker env in lockstep.

- **MIDI CC11 (Expression)** — XCent now responds to CC11 as a
  multiplicative volume scale on top of CC7. CC11=127 (default) is
  unity — no effect — and lower values attenuate alongside CC7. Lets
  you ride expression as a separate automation lane from master volume,
  the way string libraries and wind controllers expect. Not on the
  original DX100; this is a plugin extension. The routing target will
  become configurable in a future release (DEFER13: volume / TL bias /
  off).
- **macOS installer (DIST2)** — Signed `.pkg` installer wrapped in a `.dmg`
  with embedded README, licence, and auto-generated manual PDF.
  Welcome / licence screens included; installs VST3, CLAP, AU, and the
  Standalone app to the standard `~/Library/Audio/Plug-Ins/` paths.
  `clear-quarantine.sh` helper ships alongside for users who download the
  raw plugin folders rather than running the installer — strips Gatekeeper
  quarantine attributes and auto-refreshes the AU cache.
- **TX81Z output stage (DEFER6, Phase A)** — Added a `TX81ZOutputStage`
  implementing the YM3012 / DX21–DX27–TX81Z analog output path alongside
  the existing DX100 stage. Output-stage selector lives in Settings → Sound,
  feature-flagged off until hardware calibration completes.

### Verified

- **S/H LFO matches hardware across the LFS range** — 60-second
  spectral captures at 9 LFS values × 2 isolation patches confirmed
  the plugin's S/H LFO is statistically equivalent to the DX100 in
  modulation-spectrum shape (17/18 readings below the "audibly
  different" log-spectral-distance threshold). Step rates match
  exactly at lfs=20/30 and within 5–15 % across the rest of the
  range. Closes the `[~]` S/H LFO entries in v1-requirements. Residual
  mid-LFS step-rate slope gap filed as DEFER14 — not user-audible.
- **KVS velocity sensitivity matches hardware exactly** — 36 plugin
  captures vs 36 hardware captures across KVS ∈ {0, 3, 5, 7} ×
  velocity ∈ {1..127, 9 points}. Mean |peak delta| 0.37 dB; mean
  |sustain-RMS delta| 0.12 dB. 36/36 |dRMS| < 3 dB. Velocity → TL
  routing through `kVelocityCurves` is hardware-exact at every KVS
  value, including the no-sensitivity baseline (KVS=0: ±0.04 dB).
- **Chromatic pitch accuracy verified end to end (DSP18 wider range)**
  — 61-note sweep MIDI 36..96 (5 octaves) at v=100 on a stable
  all-carrier patch. Hardware and plugin both track equal-temperament
  with mean error inside ±0.5 cents; plugin vs hardware mean delta
  −0.13 cents, max 1.44 cents. **61/61 captures within ±5 cents.** No
  KC discontinuity anywhere in the playable range.
- **Saw / square / triangle LFO match hardware on amplitude and
  fundamental rate** — Same harness extended to the deterministic LFO
  waveforms (`lw=0/1/2`) at LFS = {10, 20, 40, 60, 80} on both
  isolation patches (30 hardware captures + 30 plugin captures).
  Amplitude-routed LFO matches across all three waveforms (14/15
  LSD < 0.3, with the only marginal capture — triangle/lfs=60 at
  0.361 — still showing matched first-null). Pitch-routed LFO matches
  on fundamental rate (centroid agreement within ~1 %) and total
  modulation power (dPwr within 0.34 dB everywhere) on every
  waveform; the residual harmonic-shape divergence on triangle/saw
  pitch is the known OPP/OPM PMS-curve issue (filed as DEFER15) and
  not user-audible as a rate or depth mismatch.

### Fixes

- **Bank/slot vs VCED state desync (UI39)** — When you save a project,
  edit the patch bank externally (e.g. SysEx import that overwrites a
  slot), then reopen the project, the LCD used to claim the slot was
  unchanged even though the actual sound came from the saved snapshot.
  Restore now compares the saved voice against the bank's current
  contents at the same slot and shows the modified-indicator when they
  differ — so you can see at a glance that what you're hearing isn't
  what the slot label says, and use Recall to reload the slot's current
  contents if you want to.

## v0.14.0-rc1 — 2026-05-09 — Onboarding tour, parameter tooltips, WebView floating-UI fix

> Note: skipping 0.13.0 — for luck. 🍀

### Features

- **First-run onboarding tour (UI43)** — Spotlight overlay walks new users through
  Modes, Operators, parameter editing, Modern view, Func mode, and the Patch
  browser. Dismissed permanently after first run; re-runnable any time from
  Settings → Appearance → Onboarding.
- **Parameter button hover tooltips** — Edit/Func grid buttons show the full
  parameter name + range after a 500 ms hover. Toggle in Settings →
  Appearance → Input. Works for the EZ panel knobs and sliders too.
- **Default to Modern view** — New setting (Settings → Appearance → Input)
  makes Edit mode automatically open Modern view (formerly "EZ mode").
- **Settings layout reflow** — Appearance tab uses a two-column layout; window
  scale, skin, and display colour on the left, scope/indicator/input/onboarding
  on the right.
- **Terminology** — UI labels now refer to "Modern view" instead of "EZ mode".

### Fixes

- **Floating UI off-screen at non-100% UI scale (root-cause fix)** — Tooltips,
  right-click context menus, copy-to-operator menus, and the tag picker were
  drifting / clipping off-screen at any UI scale other than 100%. The plugin
  root has `transform: scale()` for the user UI-scale setting, which makes
  it the containing block for `position: fixed` descendants — viewport-space
  coords from `getBoundingClientRect()` and mouse events were being re-scaled
  by the transform. Symptom previously patched twice with viewport clamping; this release fixes the underlying cause by
  portaling all floating elements to `document.body`.
- **LCD modified-indicator case flip on first edit (UI42)** — Two bugs:
  `markEditBufferDirty()` was inside the `if (msg.param)` guard in `PARAM_UPDATE`
  handler, skipping the call when no param selected; and `padString` in
  `DotMatrixLcd` called `.toUpperCase()` on every string, hiding E→e.
- **AMS ON LCD display** — Renamed AMS ON column to `AM=`, fixed `ampModSens`
  flag wiring.
- **Inter font + contrast pass on patch browser.**
- **Manual link** — `/xcent/manual` redirect target.

### Known issues / RC scope

- v1 launch blockers UI39 (bank/slot vs VCED desync), DIST2 (macOS installer),
  DIST5 (Windows EV signing), and BUG1 (rapid-note clicking, deferred) remain
  open and are tracked toward v1.0 (see `Docs/v1-requirements.md`).

## v0.12.7 — 2026-05-07 — BC EG bias fix + logging + Nuendo WebView fix

### Fixes

- **DSP28 — Breath Controller EG bias formula corrected.** The BC EG bias
  (breath controller → operator volume) was inverted and missing from the
  note-on path, so patches using breath control (`B10 BC1Trumpet`,
  `C18 <BC1> Sax`, `C19 BC1 Hrmnca`) produced flat volume regardless of CC2.
  Formula polarity fixed: `biasAmount = (127 − CC2) × egBiasSens × bcEgBias / 2540`.
  Low CC2 now adds attenuation (as on hardware); CC2 = 127 yields no extra
  attenuation. Bias now applied at note-on time (`applyVoiceToChannel`) in
  addition to the live CC2 handler, so notes triggered before CC2 arrives are
  weighted correctly. Divisor 2540 calibrated to hardware B10 measurement
  (20 TL units range for egBiasSens = 4, bcEgBias = 99). Plugin BC curve shape
  matches hardware; a ~3 dB systematic offset across all CC2 values is the
  same calibration gap seen in non-BC patches and is not BC-specific.

- **Garbled log output on Windows.** Debug log lines were garbled because
  `juce::String::formatted` routes through `_vsnwprintf` and interprets
  `%s` arguments as `wchar_t*`. Each pair of ASCII bytes was encoded as a
  UTF-16LE codepoint and then re-encoded as 3-byte UTF-8, producing
  unreadable log files. Fixed by replacing all macro-level string formatting
  with `snprintf` (narrow printf), which handles `%s` correctly on all
  platforms. Log files from v0.12.7 onward are plain readable ASCII/UTF-8.

- **WebView resource URL — Nuendo / plugin-guard environments.** In
  Steinberg's VST3 plugin-guard subprocess (Nuendo 15, Cubase 15), JUCE
  passes the full URL including scheme and host
  (`xcent://xcent.localhost/index.html`) to the resource provider callback
  rather than just the path. The resource table lookup failed to find any
  match, producing a blank white screen. Fixed by stripping the scheme +
  host prefix before the table lookup. The UI now loads correctly in
  plugin-guard hosts.

## v0.12.6 — 2026-05-05 — Sandbox & stripped-runtime diagnostics

Stacks on top of v0.12.5's white-screen recovery. v0.12.5 added the watchdog +
fallback panel; v0.12.6 adds the *causes* layer — pre-flight checks that
distinguish "host sandbox blocking writes" from "WebView2 runtime files were
stripped after install" from "page renderer hung", each with a distinct log
signature so a beta-tester report is actionable in one round.

### Features

- **Configurable WebView2 cache location.** New environment variable
  **`XCENT_WEBVIEW2_DATA`**. If set to an absolute path, the plugin uses that
  directory as the WebView2 user-data folder instead of the default
  `%LocalAppData%\KnivesOnStrings\XCent\WebView2`. Workaround for sandboxed
  hosts that don't permit LOCALAPPDATA writes — set the env var to a path
  the host *can* write to (the plugin install directory, a Documents
  subfolder, etc.) and reopen.

### Fixes

- **Stripped-runtime detection.** `WebView2Detector::isWebView2AvailableForJuce()`
  now cross-checks the registry version against the actual presence of
  `msedgewebview2.exe` on disk. Some "audio optimization" scripts (Atlas OS,
  BlackViper variants, Win11Debloat) strip the WebView2 runtime files but
  leave the EdgeUpdate registry key behind. Previously we trusted the
  registry, constructed the WebView, and got a white screen when the loader
  couldn't find the binary. The new check searches the two standard install
  bases (`%ProgramFiles(x86)%\Microsoft\EdgeWebView` and
  `%LocalAppData%\Microsoft\EdgeWebView`) for the version the registry
  reports; if it's not there, the runtime is treated as missing and the
  WebView2 missing panel is shown — same as a fresh install with no runtime
  at all. Either way the user gets the "install WebView2" prompt with a
  working button.

- **Host process diagnostics + write probe.** `HostDiagnostics.cpp` now runs
  three checks in the editor constructor:
  - **Host process info.** Logs the host EXE name, full path, and Windows
    token integrity level (Untrusted/Low/Medium/High/System). Detects
    AppContainer sandboxes used by FL Studio and some Cubase configurations.
  - **WebView2 runtime cross-check.** Logs the registry-reported version
    *and* the resolved on-disk path to `msedgewebview2.exe`. If the path is
    empty while the version is non-empty, an explicit `STALE REGISTRY` error
    line is emitted naming the likely culprit scripts.
  - **User-data-folder write probe.** Before the WebView is even constructed,
    attempt to create + delete a probe file in the cache folder. If the host
    sandbox is blocking writes (Cubase VST3 sandbox, restricted ACL on
    LOCALAPPDATA), the editor skips the WebView entirely and shows the
    failure panel with the specific Win32 error string and an
    environment-variable workaround.

## v0.12.5 — 2026-05-05 — WebView white-screen recovery

### Features

- **Native fallback panel when the UI fails to load.** Beta-tester report:
  a fresh v0.12.1 install came up to a blank white screen inside the VST3
  host. Previous behaviour was diagnostic-blind — if the WebView2 runtime
  initialised but the page failed to render (cache corruption, JS module
  load failure, sandbox/AV interference), the editor showed an empty white
  component with no signal that anything had gone wrong. This release adds
  two layers of detection on top of the existing `WebView2MissingPanel`
  (which still catches "runtime not installed at all"):
  - **`pageLoadHadNetworkError` hook.** `XCentWebBrowser` is now a
    `WebBrowserComponent` subclass that overrides `pageLoadHadNetworkError`
    and `pageFinishedLoading`, both of which log to `xcent.log` and feed
    back to the editor. A network-level error swaps the WebView for a native
    diagnostic panel immediately with the platform-specific error string.
  - **8-second `UI_READY` watchdog.** A one-shot `juce::Timer` arms after
    `goToURL`. If React's `UI_READY` message hasn't arrived by the time it
    fires (typical legitimate load is 1–3 s), the editor falls back to the
    same diagnostic panel. UI_READY arriving first cancels the watchdog.
    Network errors arriving first also cancel it.

- **Diagnostic panel content.** The new `WebViewLoadFailedPanel` shows
  failure reason (network error string, or "did not signal ready within Ns"),
  detected WebView2 runtime version, path to `xcent.log`, path to the
  WebView2 cache folder, an "Open log folder" button (shells to the logs
  directory so testers can grab the file), and a "Clear WebView2 cache"
  button (deletes the user-data folder so the next plugin instantiation
  rebuilds it from scratch — fixes ~half of stale-state white-screen
  reports on similar JUCE WebView plugins).

- **Installer: WebView2 cache wipe on install and uninstall.**
  `xcent-installer.nsi` now removes
  `%LOCALAPPDATA%\KnivesOnStrings\XCent\WebView2` in the Finalize section
  of the install and at the end of the uninstall. Reinstalling a new
  version always starts with a fresh WebView2 cache, which eliminates "old
  cache poisons new build" as a class of failure mode without the user
  having to do anything manually. User data at
  `Documents\KnivesOnStrings\XCent` (patches, prefs, logs) is still
  preserved across install/uninstall as before.

## v0.12.4 — 2026-05-03 — CPU meter idle fix

### Fixes

- **CPU meter now shows 0% at idle.** The CPU meter was showing 30–50% with
  no notes playing. Root cause: any MIDI input (active sensing, clock, or
  other real-time messages from a connected keyboard) kept `midiMessages`
  non-empty every block, which prevented the idle flag from engaging and
  left Nuked-OPM running on silence. Fixed with an early fast-path exit in
  `processBlock`: if the engine was idle last block, the FIFO is empty,
  and there is no pending voice change, we store `0.0f` to the CPU meter
  and return immediately — skipping APVTS reads, MIDI iteration, SysEx
  handling, `renderBlock`, gain apply, and peak metering. The meter now
  reads 0% at idle and only lights up during active synthesis.

## v0.12.3 — 2026-05-03 — undo/redo fixed, wheel speed, LCD prefix cleanup

### Features

- **Mouse wheel parameter speed +50%.** `useWheelControl` previously stepped
  1 unit per tick. Now uses a fractional accumulator with a 1.5 step/tick
  base: the accumulator alternates 1–2–1–2 (average 1.5), giving exactly
  50% faster scrolling without changing coarse-mode (`Ctrl+wheel`).
  Accumulator resets on direction reversal and on gesture end.

### Fixes

- **Undo/redo depth fixed.** `undo()` and `redo()` both send a
  `VOICE_RESTORE` to C++, which echoes back `VOICE_LOADED`. The echo called
  `setVoiceParams()` which clears both the undo and redo stacks as a side
  effect — so after the first undo the remaining history and all redo state
  was wiped. Fixed by adding `setVoiceParamsKeepHistory()` to the store
  (syncs `voiceParams` only, no stack side-effect) and using it in the
  VOICE_RESTORE echo path in `App.jsx`. Undo stack depth is now the full
  50-entry limit; redo works correctly across multiple undo steps.

- **LCD prefix: patch-level params no longer show operator mask.** In EDIT
  mode, operator-scoped params (AR, D1R, OUT, etc.) still display
  `E1111 OUT=23 OP1` — the mask shows which operators are active.
  Patch-level params (ALG, FBL, PMD, pitchBend, transpose, etc.) now
  display `E ALG=1` instead of `E1111 ALG=1`. The 4-char operator mask was
  unused for these params and caused long names like `PBend Range=12` to
  be truncated.
## v0.12.2 — 2026-05-03 — KVS velocity table, mouse wheel wheels, UI39+UI42 partial

### Features

- **UI37: mouse wheel for pitch-bend and mod-wheel controls.** Both
  on-screen wheels now respond to mouse scroll when the global Settings →
  Mouse Wheel toggle is on.
  - **BEND**: ±5% of range per tick (Ctrl/⌘ = ±10%), springs back to center
    (8192) automatically after 350 ms of quiet — matching the hardware
    pitch-wheel spring-return behaviour.
  - **MOD**: ±1 per tick (Ctrl/⌘ = coarse), holds the scrolled value.

  Implementation uses a non-passive wheel listener with a 25 ms throttle
  and the `latest.current` closure pattern (no re-attach per render).
  Closes UI37 v1 launch blocker.

### Fixes

- **DSP24-KVS: ROM-extracted KVS velocity → TL offset table.**
  The per-operator key-velocity-sensitivity (KVS) attenuation
  previously used a quadratic approximation. At KVS=5, vel=16, the
  approximation over-attenuated by ~9 dB vs hardware. The error grew
  at low velocities with high KVS settings (up to −10 dB at vel=16).
  Velocity → TL is now a direct ROM-extracted lookup table sweeping
  all 128 MIDI velocities × 8 KVS levels through the actual DX100 ROM
  firmware. ROM values are monotonically non-increasing with velocity;
  KVS=0 is all zeros; KVS=7 at vel=1 = 78 TL units (≈46 dB
  attenuation). Zero regressions on 552-case test suite.

- **UI42 partial — two bugs fixed + diagnostic added.**
  Two confirmed bugs causing erroneous dirty-flag state were fixed:
  1. **VOICE_RESTORE echo reset dirty after undo.** The undo path sent
     VOICE_RESTORE to C++, which echoed back `VOICE_LOADED` with no
     `bank`/`slot`/`name`. App.jsx called `setPatchLoaded(undefined, …)`
     which set `editBufferDirty=false` — immediately undoing the dirty
     flag set by the undo action itself. Fix: `VOICE_LOADED` handler now
     guards `setPatchLoaded` behind `msg.bank != null`.
  2. **masterTune PARAM_UPDATE marked dirty on every patch load.** After
     each `notifyVoiceLoaded`, C++ sent a separate `PARAM_UPDATE` for
     masterTune, which triggered `markEditBufferDirty()` → dirty=true
     immediately after loading any patch or navigating prev/next. Fix:
     masterTune now travels as a field in the `VOICE_LOADED` payload and
     is applied via `setVoiceParam` without calling `markEditBufferDirty`.

  A visible diagnostic was added to the LCD EDIT MODE fallback:
  `E1111 EDIT MODE C` (dirty=false) / `e1111 EDIT MODE D` (dirty=true).
  The primary UI42 symptom (prefix not flipping on normal param edits)
  may be fixed by these two changes — requires user testing to confirm.
  Remove the `[UI42 DIAG]` comment in `DotMatrixLcd.jsx` once verified.

- **UI39: SysEx-loaded voice display fix (partial).** Two parts of the
  bank/slot desync were addressed:
  1. **Display fallback for slot-less voices.** When a voice is loaded
     with no bank/slot origin (SysEx receive, Init Voice), the LCD PLAY
     mode now shows `P<voice-name>` instead of the confusing
     `P<bank>:<voice-name>` format (e.g. `PSysEx:BRASSCOMP`).
  2. **State preservation on SysEx receive.** When a single-voice SysEx
     is received via MIDI, `currentBank_`/`currentSlot_` are now set to
     `"SysEx"/-1` so `getStateInformation` saves the correct origin.
     Previously, the saved bank/slot pointed to whatever was loaded
     before the SysEx arrived, causing the wrong patch to be shown on
     project restore.

  The underlying cause (C++ bank/slot and JS `loadedSlot` diverge when
  SysEx or Init Voice is applied) is now closed for these two cases.

- **CPU display.** VuMeter rAF loop now self-terminates when the meter
  fully decays to zero (was running at 60 fps continuously even at idle).
  CPU meter EMA decay is faster (α=0.06 vs 0.033) — stale high readings
  clear in ~0.5 s instead of ~1 s. Attack still fast (α=0.3) for spikes.
## v0.12.0 — 2026-04-29 — Pitch EG (DSP12 + DSP13)

Firmware-accurate Pitch EG closing the last two DSP v1 launch blockers.

### Fixes

- **Pitch EG level scaling (DSP13).** Level-to-pitch conversion now uses the
  ROM-extracted `kPegLevelScale[100]` table (DX21 ROM `$DCDD`) instead of the
  previous `(level-50)*5*3` approximation. Internally represented as a 16-bit
  value centered at `0x3000` (±48 semitones full swing). Level 50 = zero
  offset; Level 99 = +48 semitones; Level 0 = −48 semitones.

- **Pitch EG rate stepping (DSP12).** Per-firmware-model PEG:
  - **DX100 / FB01**: instant snap to L1 at note-on, no subsequent movement.
  - **DX21 / TX81Z / DX11**: ROM rate-accumulator algorithm —
    step = `lfoDelayRate[99−R]` per CMI tick (~13.7 Hz). Phase machine:
    attack1 (→L1) → attack2 (→L2) → sustain-hold → release (→L3) → idle.
    Note-off triggers release. Instant sentinel (R=99) snaps and advances
    phase in a single tick.

  PEG offset is wired into KC/KF register writes via `applyPitchEgToChannel`,
  so pitch is correctly shifted for both instant (note-on) and rate-based
  (per-tick) cases. 14 new test cases (`test_peg_stepping.cpp`), zero
  regressions.

## v0.11.10 — 2026-04-27 — DSP Accuracy Push (PMD/AMD + Calibration + KC Verification)

A focused pass on patch-level loudness and FM modulation depth, driven
by a new hardware-comparison pipeline against 96 stock factory patches
(banks 101/201/301/401, audio + MIDI captured from real DX100 hardware).
User AB-confirmed audibly meaningful improvement over v0.11.9.

### Features

- **Hardware-comparison pipeline (new).** Tooling shipped this version for
  the DSP17 family of investigations (reusable for DSP12/DSP13 PEG
  extraction next).

### Fixes

- **+3 dB analog-stage calibration (DSP17 / DSP21).** Hardware-comparison
  data showed the plugin was systematically ~3 dB quieter than the DX100
  line-out across all 8 algorithms after segmentation contamination was
  ruled out. Adopted `kPassbandGainDb = +2.5 dB` net, then refactored the constant into two
  semantically-distinct components:
  - `kFilterPassbandGainDb = -0.5 dB` — IC12 active LPF passband loss,
    rigorously derived from the dual-input differential drive topology:
    `Vout/Vnode_A = (Z_C3 - Z_C2) / (R + Z_C3)` with R = R2 = R3 = 2.2 kΩ.
    Yields ≈-0.6 to -2.7 dB across audio (low-frequency limit modeled as
    flat -0.5 dB). The previous "R1/(R2∥R3) divider" framing in the spec
    doc was a hand-wave (op-amp inputs are high-Z so they don't load that
    way) — magnitude was right by coincidence.
  - `kOutputCalibrationDb = +3.0 dB` — empirical calibration to match
    the absolute level of the hardware reference recordings. **Not
    attributable to a single schematic stage**: IC21 output driver
    (+14.7 dB inverting amp), VR3 volume pot, and audio-interface input
    gain during reference capture are all unmodeled and collapsed into the
    implicit "DAC max → host ±1.0" mapping. Documented as such.

  `Docs/specs/04-ANALOG-OUTPUT.md` was rewritten with the rigorous IC12
  transfer function and the calibration-constant rationale. After the
  calibration, per-algorithm median peak delta is within ±1 dB of zero
  across ALG 0–6.

- **ROM-extracted PMD/AMD scaling tables (DSP19a).** A diagnostic test revealed that
  the LFO-depth path was writing voice PMD/AMD (VCED range
  0–99) directly to the YM2164 register. The actual DX100 firmware
  applies a non-trivial ROM-extracted scaling. Tables extracted by
  sweeping the actual DX100 v1.1 ROM in the HD6303 emulator:
  - **PMD: approximately linear** (~127/99 expansion).
    voice 80 → reg 102. voice 99 → reg 127 (full scale).
  - **AMD: dramatically non-linear** with an exponential top end.
    voice 50 → reg 16 (32% of register range); voice 80 → reg 37 (29%);
    voice 90 → reg 54 (60%); voice 99 → reg 127 (full scale).

  `writeLfoDepth` now applies these tables before writing to the chip;
  `[pmd_scaling]` confirms FirmwareLogic matches the ROM register-by-
  register on all tested patches. Audible improvements verified by user
  A/B in Ableton on D06 Good Vibes (peak delta −11.5 → −6.3 dB),
  D08 Helicopter (sustain correlation 0.87 → 0.92), D12 Birds (0.14 →
  0.23 — moves out of the worst-residuals list), D18 LFO NOISE (0.81 →
  0.85), D20 Windbells (attack 0.79 → 0.87).

- **DSP18 patch 102 KC — verified, no regression.** User-supplied hardware
  capture for patch 102 (codebase A02 Uprt Piano) across MIDI 57–63
  (A3..D#4). Per-note H1/H2/H3/centroid + pitch analysis:
  plugin tracks hardware within ±1 dB on every harmonic and ±0.04 cents
  on pitch across the captured range. The KC break the user observed in
  v0.11.6 is not present in current code — almost certainly closed by
  DSP19a + the +3 dB calibration. Status: verified in narrow range; wider
  chromatic capture would formally close.
### Known issues (carried over from v0.11.9)

- **UI42** P1 v1 blocker — modified-indicator case flip never fires
  on user edit. See v0.11.9 entry below and `Docs/TODO.md` UI42 for
  full diagnostic context.

---

## v0.11.9 — 2026-04-25 — UI41 Mode-Letter Prefix (partial — see Known issues)

> ⚠ **P1 known issue (UI42):** the modified-indicator case flip
> (E→e, P→p) does not visibly fire when the user edits a voice
> parameter, even though the backend atomic flips and the frontend
> store setter is called. Clean-state prefix and the Modified
> Indicator setting both render correctly; the dirty transition
> itself is broken. Tracked in `Docs/TODO.md` UI42 with full
> diagnostic context. **Blocks v1.**

### Features

- **Play-mode LCD mode letter (clean state only — see UI42 for dirty).**
  The LCD now prepends `P` to the patch identifier in PLAY mode, matching
  the DX100 hardware convention (was previously no leading letter).
- **Modified Indicator setting.** Settings → Appearance → Modified
  Indicator. Two styles, both visible and persistable:
  - **DX Mode** (default) — hardware-style: when the dirty bit fires, the
    mode letter is intended to lowercase (`P`/`p`, `E`/`e`, `F` static).
  - **Standard (\*)** — replaces the would-be-lowercase letter with `*`.
    Clean-state display is identical to DX Mode.
- **`voiceModified` persistence.** The voice-modified flag now survives
  DAW project save/reload and Standalone restart via an atomic on
  PluginProcessor serialised alongside the voice blob, with bidirectional
  WebView event sync.

### Fixes

- Edit-mode patch-level params (ALG, FBL, LFO\*, PMD, AMD, pitchBend,
  transpose) were incorrectly showing an `F` prefix on the LCD — they're
  voice params, not system params, and now correctly get `E1111`
  (edit-mode prefix with op-mask).
- FUNC mode fallback (no param selected) was rendering `E1111 FUNC MODE`.
  Now correctly shows a static `F FUNC MODE`.
## v0.11.8 — 2026-04-21 — Mouse-wheel Editing, ROM Bank Hardening, Smarter Auto-tagger

### Features

- **UI36 — Mouse wheel adjusts EZ-mode parameters.** Scroll over any
  EZ-mode knob, slider, or the main output knob to step its value.
  Modifier keys scale the step:
  - **Default wheel**: one integer step (or 0.02 for the normalized output knob).
  - **Ctrl / ⌘ + wheel**: coarse — ~10% of full range per tick.
  - **Shift + wheel**: fine — 1/10 of a standard step (for ranges that allow
    it; integer params stay at 1).

  Under the hood, `useWheelControl` attaches a non-passive `wheel` listener
  so `preventDefault()` actually stops the page from scrolling (React 18's
  built-in `onWheel` is passive). Events are throttled to 25 ms and grouped
  into a single undo entry per gesture (350 ms quiet window). Turn the
  whole feature off in **Settings → Appearance → Input** if trackpad
  inertia keeps grabbing controls — the toggle persists with the rest of
  the appearance preferences.

- **TOOL1 — Auto-tagger feature upgrade (v2 rules).** The corpus-derived
  tag rules now use twelve features instead of four. Parameter-rule yield
  jumped from **0 firing rules** in v1 to **5**:
  - `lead` gated to algorithm 2 (conf 0.78 — fires alone)
  - `brass` gated to algorithm 5 (conf 0.78 — fires alone)
  - `organ` gated to algorithms 1/2/3 (conf 0.75 — fires alone)
  - `perc` on algorithms 3/4/5/7 and `strings` on 2/3/5/6/7 (both fire as
    secondary signals combined with a name hint)

  New feature set: carrier-vs-modulator split (using `kIsCarrier[alg]`)
  for output level, attack rate, and sustain level; LFO depth (PMD) and
  speed; feedback; all-op averages kept for backward compatibility; plus
  a categorical **algorithm allowlist** keyed to each tag. `parse.py` now
  captures `op_frequencies` and `op_detunes` too — ready to unlock an
  inharmonic-ratio feature in a future corpus rebuild.

  C++ side: `AutoTagger::paramThresholds` was promoted from a flat
  `map<string, float>` to a `ParamConditions` struct that carries both
  numeric thresholds and an algorithm allowlist. `matchesParamThreshold`
  now evaluates eighteen condition keys covering the v1 aggregates and all
  v2 splits. All eight autotagger tests pass.

- **`mouseWheelEnabled` persisted setting.** Joins `theme`, `uiScale`,
  `displayStyle`, and `fb01VelocityMode` in the state XML. `THEME_STATE`
  emitted on `UI_READY` now carries both `fb01VelocityMode` and
  `mouseWheelEnabled` so the UI reflects the saved values at load
  (previously the former was computed but never consumed by the
  front-end).

### Fixes

- **UI35 (final pieces) — ROM banks are read-only, MIDI bulk dumps
  persist.** Two safety gaps in the v0.11 patch model are now closed:
  - **ROM read-only everywhere.** Backend `storePatch`, `renamePatch`,
    and `deletePatch` refuse to write bank 100/200/300/400 and log a
    warning. The browser modal disables the **Store** button on ROM
    banks (and on Favorites/Recents), with a tooltip explaining *why*
    it's disabled. ROM bank rows in the tree carry a 🔒 glyph and
    `"read-only ROM bank"` hover tooltip.
  - **MIDI VMEM bulk dumps now stick.** When a 32-voice bulk dump
    arrives over MIDI, the audio thread writes it into bank A (as
    before) *and* sets a deferred pending flag. The editor timer
    consumes the flag on the message thread, calls
    `persistUserBankToDisk("A")` to write the bank out to disk, and
    pushes a full bank refresh to the UI so the browser's other 23 slot
    names update, not just slot 0. A log line records the event for
    diagnostics.
## v0.11.7 — 2026-04-19 — Windows Installer (NSIS)

### Features

- **Single-click Windows installer** built with NSIS. One executable drops
  VST3, CLAP, the Standalone, the PDF manual, and `CHANGELOG.md` into place
  with a bundled Microsoft Edge WebView2 Runtime fallback for clean
  machines. Output: `build/XCent-Setup-<version>.exe` (~6 MB with LZMA
  /SOLID compression).

- **Installer behaviour.**
  - **Welcome → License → Install Scope → Components → Install → Finish**
    wizard flow.
  - **Per-machine** (default) or **per-user** radio choice on a dedicated
    page after the license:
    - Per-machine: `%CommonProgramW6432%\VST3`, `%CommonProgramW6432%\CLAP`,
      `%ProgramFiles%\Knives On Strings\XCent\`
    - Per-user: `%LOCALAPPDATA%\Programs\Common\VST3`,
      `%LOCALAPPDATA%\Programs\Common\CLAP`,
      `%LOCALAPPDATA%\Programs\Knives On Strings\XCent\`
  - **WebView2 detection** mirrors the in-plugin detection logic — same
    GUID, same registry key, same minimum major version (109). Bundled
    Evergreen Bootstrapper only runs when the runtime is missing or too
    old; already-installed users see a no-op.
  - **Start Menu shortcuts** for the Standalone, the PDF manual, and the
    uninstaller (when the relevant components are selected).
  - **Uninstaller** registered at the correct HKLM/HKCU Add/Remove
    Programs key based on install scope. User data at
    `Documents/KnivesOnStrings/XCent/` is intentionally preserved on
    uninstall.

- **Documentation regeneration.** The user manual is regenerated as
  `XCent-Manual.pdf` at installer build time via `marked` + Microsoft
  Edge headless (`--print-to-pdf`). No pandoc or Puppeteer required.
  The changelog is copied from the repo at build time so it always
  tracks the released version.
## v0.11.6 — 2026-04-21 — Graceful Fallback When WebView2 Runtime Is Missing

### Fixes

- **Blank-UI problem on clean Windows installs.** When Microsoft Edge
  WebView2 Runtime is missing or older than Chromium 109 (required for
  JUCE 8's resource provider interception), the UI would either show
  "Can't reach this page https://juce.backend" or just stay blank-white.
  No more — the plugin now detects this at editor construction time and
  shows a proper fallback panel explaining what's needed, with a
  one-click button that opens Microsoft's official installer page.

  On Windows, during `XCentAudioProcessorEditor` construction:
  1. Read `HKLM\SOFTWARE\WOW6432Node\Microsoft\EdgeUpdate\Clients\{F3017226-FE2A-4295-8BDF-00C3A9A7E4C5}` `pv` value (with `HKCU` fallback for per-user installs), both via `KEY_WOW64_64KEY` so 32-bit plugins read the 64-bit registry view
  2. Parse the major version; if empty or `< 109`, skip `webView->goToURL()` and show `WebView2MissingPanel` instead
  3. Panel shows detected version (or "not installed"), required version, and an "Install WebView2 Runtime" button opening `https://go.microsoft.com/fwlink/p/?LinkId=2124703` in the user's default browser
  4. All detection results are logged via `XCENT_LOG_INFO` so the beta log captures exactly what was found

  macOS, Linux, iOS use JUCE's `defaultBackend` (WKWebView / WebKitGTK)
  which is always present — no detection, no panel.
## v0.11.5 — 2026-04-20 — SysEx Export with Format Picker + Loss Warnings

### Features

- **Format picker dialog for SysEx export.** The browser's new "Save as
  SysEx…" action opens a dialog with six target formats:
  - DX100 VCED (single voice) / DX100 VMEM (32-voice bank)
  - DX21 VCED / VMEM (byte-identical to DX100 but labeled as DX21 for
    clarity — chorusSwitch is meaningful here)
  - TX81Z VCED+ACED (two-message export: ACED first with `LM__8976AE`
    header + per-op waveform/fixed-freq/EG-shift fields, then VCED)
  - TX81Z VMEM (32-voice bank with ACED data packed into bytes 73–96 per
    voice)

- **Least-lossy default.** The dialog pre-selects the format that
  preserves the most data for the voice's `hardwareContext.firmware`:
  DX100 voice → DX100 VCED, DX21 → DX21 VCED (keeps chorusSwitch),
  TX81Z → TX81Z VCED+ACED, DX11 → TX81Z VCED+ACED (lossy, drops PEG),
  FB01 → DX100 VCED (lossy, drops arVelocitySensitivity).

- **Field-by-field loss warnings.** The dialog shows an orange "DATA LOSS
  WARNING" section listing exactly which fields will be dropped and their
  current values, or a green "No data loss" confirmation. Detected losses
  cover: OPZ per-op fields (waveformSelect, fixedFreqEnabled/Range/Fine,
  egShift) + TX81Z global fields (reverbRate, fcPitch, fcAmplitude,
  operatorOnOff) when exporting to OPP formats; DX21 fields (chorusSwitch,
  footVolumeRange) when exporting to TX81Z; arVelocitySensitivity in all
  hardware formats (FB-01 export not supported, spec doc 10 §Export
  Matrix).
### Known issues

- **TX81Z VCED edge case.** The exporter appends `operatorOnOff` as byte
  index 93 of the TX81Z VCED payload, making the data 94 bytes rather
  than the stated 93. Matches observed TX81Z hardware transmit behaviour
  but may fail checksum validation against strict 93-byte-only parsers.
  If this breaks hardware round-trip in beta testing, we'll move
  `operatorOnOff` to travel via ACED or VMEM exclusively.

---

## v0.11.4 — 2026-04-20 — Library Storage Migrated to .xcb

### Features

- **Internal library is now `.xcb` instead of `.syx`.**
  `Documents/KnivesOnStrings/XCent/Library/` contains
  `factory-a.xcb`…`factory-d.xcb` and `my-sounds.xcb`. Preserves all
  `VoiceParams` fields (OPZ waveforms, fixed-freq, `arVelocitySensitivity`,
  `chorusSwitch`, `hardwareContext`) across save/load — the `.syx` format
  silently dropped all of these.

- **`.syx` remains an IMPORT format.** `SysExLibrary::importFile(path.syx)`
  parses external DX100/DX21/FB-01/TX81Z dumps and stores them internally
  as `.xcb`. Hardware interchange still works both directions via the
  dedicated import/export paths; `.syx` just isn't used for our own
  storage anymore.
## v0.11.3 — 2026-04-20 — SysEx Import for All 5 Hardware Formats

### Features

- **TX81Z VCED+ACED import.** Single-voice TX81Z dumps (ACED supplementary
  + VCED base, transmitted together) are now parsed into a unified
  `VoiceParams` with `hardwareContext = OPZ/TX81Z/TX81Z`. The ACED data
  fills in per-operator `waveformSelect`, `fixedFreqEnabled`,
  `fixedFreqRange`, `freqRangeFine`, `egShift`, plus voice-global
  `reverbRate`, `fcPitch`, `fcAmplitude`.

- **TX81Z VMEM bank import.** 4096-byte 32-voice bulk dumps (format
  `04 10 00`, distinct from DX100's `04 20 00`) parse all voices with
  both VCED and packed-ACED fields unpacked per voice. Context auto-set
  to OPZ/TX81Z/TX81Z.

- **DX11 auto-detection.** DX11 uses the same ACED+VCED wire format as
  TX81Z but actively uses the PEG bytes (87–92). The importer now
  inspects those bytes: if they're the fixed TX81Z pattern
  (`PR=99, PL=50`) or all-zero, context is `OPZ/TX81Z/TX81Z`; if they
  look "active" (anything else), context is `OPZ/DX11/TX81Z` so the
  firmware-aware PEG behavior kicks in.

- **DX21 auto-detection.** DX100 and DX21 share byte-identical VCED
  (format `03`). On import, the parser defaults to DX100 context; if
  `chorusSwitch=1` (VCED byte 64, a DX21-specific feature the DX100
  firmware never sets), it promotes to `OPP/DX21/DX21`.
### Known issues

- **TX81Z VMEM byte 73–83 ACED packing** is inferred from analysis rather
  than service-manual page 24 directly.
  `test_tx81z_vmem_aced_fields_extracted` verifies self-consistency, but
  a real-world TX81Z VMEM file from hardware would be the definitive
  validation.

---

## v0.11.2 — 2026-04-20 — Debug Logging for Dev/Beta Builds

### Features

- **Compile-time file logger.** Gated by the
  CMake option `XCENT_DEBUG_LOGGING` (default ON for now; flip to OFF for
  public shipping). When disabled the macros expand to nothing — zero
  runtime cost.
- **Log file** at
  `%USERPROFILE%\Documents\KnivesOnStrings\XCent\logs\xcent.log`.
  Overwritten on each launch (no rotation yet). Thread-safe via
  `std::mutex`. Auto-flushes so a crash doesn't lose recent entries.
  Silent fallback if the file can't be opened (no plugin crash).
## v0.11.1 — 2026-04-20 — Patch Level Normalization, RAM Bank Tags, State Robustness

### Fixes

- **Algorithm carrier normalization re-enabled.** The DX100 firmware's
  per-algorithm TL offsets (ALG 4: +8 TL per carrier, ALG 7: +16 TL per
  carrier, etc., all extracted from ROM) are now applied in all three TL
  write paths (note-on, live parameter update, CC7 volume change).
  Factory patches play at much more consistent levels — previously a
  4-carrier ALG 7 patch could be ~12 dB louder than a 1-carrier ALG 0
  patch. Re-enables a feature the firmware was always trying to implement
  (192 kHz hardware recordings showed the real DX100 chip doesn't reflect
  the offsets at its DAC, but the intent was clearly level compensation
  and consistency wins over strict matching of that quirk).

- **RAM bank tags.** Factory tags for patches loaded into RAM banks
  A/B/C/D now show in the browser. The `factory-tags.json` had tags for
  factory voices 96–191 keyed as
  `LIB:factory-a:0`..`LIB:factory-d:23` (library-view keys), but RAM
  banks use `A:0`..`D:23`. Added 96 alias entries so both key formats
  resolve to the same tags.

- **INIT VOICE state safety net.** If the DAW-saved voice state has the
  literal name "INIT VOICE" (e.g., the user clicked Init then closed),
  the plugin now falls back to reloading the saved bank/slot on next
  launch instead of self-perpetuating the init state. User-edited factory
  patches keep their original names and are unaffected.

- **Default patch load robustness.** `setStateInformation` now explicitly
  falls back to bank 100 slot 0 (IvoryEbony) when either attribute is
  missing from state, and `getIntAttribute("slot", 0)` (was `-1`). Covers
  fresh installs, decode failures, and partial state files.
## v0.11.0 — 2026-04-20 — Unified Patch Format, DX21 & FB-01 Support

### Features

- **Unified Voice Format & Per-Voice Hardware Context.** XCent's internal
  voice representation is now the union of all supported 4-op Yamaha
  synths, not the DX100 format with extras. Every voice carries its own
  `VoiceHardwareContext` (engine / firmware / output stage) — so a voice
  loaded from a TX81Z dump can be run through a DX21 firmware table, a
  FB-01 output stage, or any mix. Defaults produce original-hardware
  behavior; model-specific fields sit at inactive defaults on DX100
  voices.

- **DX21 Support.** Full ROM analysis (tables confirmed byte-identical to
  DX100), BBD chorus modeled from hardware measurements (0.331 Hz sine
  LFO, 2.65 ms center delay, ±1.64 ms depth, pseudo-stereo topology),
  enabled via the `chorusSwitch` bit in the voice data. New CHORUS toggle
  button in the UI below the INC/DEC row — click to enable on any patch,
  red LED indicator.

- **FB-01 Support.** SYX bank import extracts `arVelocitySensitivity`
  from the D1R byte (bits 6:5) per operator. The exact AR velocity offset
  formula is reverse-engineered from FB-01 ROM `$0F83` — a 4×16 signed
  offset table applied at note-on to the AR register, preserving KS bits
  and clamping to [2, 31]. Available on all firmware models, not just
  FB-01 voices.

- **Native .xcb Bank Format.** Binary voice storage with CRC-32
  verification and forward compatibility (`voice_data_size` header field
  lets old readers skip unknown trailing bytes). Replaces lossy .syx for
  internal storage — OPZ waveforms, fixed-freq, EG shift,
  `arVelocitySensitivity`, hardware context, and chorus state all
  round-trip cleanly.

- **Firmware Model Extension.**
  - `FirmwareLogic::setModel()` dispatches `olToTl`,
    `velocityTlOffset`, and `applyArVelocitySensitivity` by model
  - Per-model tables in `RomTables.h` (`dx21::`, `fb01::` namespaces)
  - DX21 aliases DX100 tables (byte-identical per ROM analysis)
  - TX81Z/DX11 fall back to DX100 until their ROM analysis lands
  - FB-01 AR velocity offset LUT bundled as
    `rom::fb01::arVelocityOffset[4][16]`

- **UI.**
  - **CHORUS toggle button** — new engraved box below INC/DEC in the left
    column, left-aligned. Red LED lights when enabled. Matches the
    hardware look of the existing OP buttons.
  - **ROM patch numbers** — Bank 100 slots show 101–124, Bank 200 shows
    201–224, etc. User banks (A/B/C/D) keep 1–24.
  - **About modal** — Replaced the styled "DCENT" text with the XCent SVG
    logo (same asset as the header bar).
  - **Volume knob** — Double-click resets to unity gain (0 dB).
  - **WebView2 data folder** — Moved from Documents to AppData/Local to
    avoid OneDrive-sync interactions that can block WebView2 init on some
    Windows setups. Standalone was never affected because it runs as its
    own process.

### Fixes

- **FB-01 bank parser** — Corrected test expectation from 48 voices to 24
  per SYX dump. Byte math confirms: 6363-byte SYX → 32-byte header + 24 ×
  131-byte voice records.
- **FB-01 D1R mask** — Was `& 0x0F` (4-bit), now `& 0x1F` (5-bit). D1R is
  a 5-bit field; the old mask silently capped it at 15.
- **Tag seeding** — `TagStore::load()` stored empty tag arrays, blocking
  `seedFromFactory` from re-seeding. Factory tags now seed correctly on
  fresh installs AND recover on upgrades from older versions that wrote
  empty data.
- **VoiceConvert `toVar` duplicate** — Per-operator `"ampModSens"` key
  was a dead write duplicating `"amsOnOff"`. Removed.
## v0.10.0 — 2026-04-19 — Patch Browser & Auto-Tagging

### Features

- **Patch Browser.** Modal with tag-filtered search across all 192
  factory voices plus user banks. Double-click to load, right-click for
  tag picker. Modal focus trap, ARIA-labeled, keyboard accessible
  (WCAG 2.1 AA).

- **Auto-Tagging.** Every factory voice is auto-tagged via
  `auto-tag-rules.json` derived from a 6,312-voice FB-01 corpus. Rules
  score voices by name patterns and parameter thresholds; suggestions
  cover categories like bass, pad, keys, lead, brass, bell, pluck,
  percussion, SFX. Users can override any tag via the right-click tag
  picker.

- **Factory Tag Seeding.** Bundled `factory-tags.json` covers all 192
  voices with curated initial tags. Seeds on first run; custom user tags
  persist in `Documents/KnivesOnStrings/XCent/Library/tags.json`.

- **UI.**
  - **TagFilterBar** above the browser — chip-style tag filters with
    AND/OR toggle
  - **TagPicker context menu** — right-click any patch to edit tags
    inline
  - **Custom tags** — users can create their own tags in addition to the
    factory set
## v0.9.14 — 2026-04-06 — iOS Testing & On-Screen Controls

### Features

- **Pitch Bend & Mod Wheel** — On-screen pitch bend (spring-loaded, snaps
  to center) and mod wheel controls, positioned to the left of the
  on-screen keyboard. Available on all platforms. Pitch bend sends 14-bit
  MIDI pitch wheel; mod wheel sends CC1.
- **iOS/iPadOS Build Pipeline** — Full code signing with Apple Developer
  Team ID, automatic provisioning, and one-command deploy to device via
  `xcrun devicectl`.
- **iOS adaptations.**
  - **Full-screen auto-scaling** — UI automatically scales to fill the
    iPad/iPhone screen while maintaining aspect ratio. Accounts for
    keyboard, EZ mode, and orientation changes.
  - **Keyboard shown by default** — On-screen keyboard visible on launch
    (no external MIDI keyboard assumed on iOS).
  - **Landscape-only orientation** — Locked to landscape for optimal
    synth layout.
  - **Background audio** — Sound continues when app is backgrounded (via
    `JUCE_BACKGROUND_AUDIO_ENABLED`).
  - **Custom app icon** — XCent FM algorithm logo at all required iOS
    sizes via custom xcassets.
  - **Scale setting hidden** — Window scale option removed from iOS
    Settings (not applicable).

### Fixes

- **VU Meter (iOS)** — Routed VU level through the central message store
  instead of a separate event listener, fixing iOS WebKit not dispatching
  to duplicate `__juce__nativeEvent` listeners.
## v0.9.1 — 2026-04-02 — FB-01 Velocity Mode

### Features

- **FB-01 Velocity Mode** — Authentic per-operator velocity sensitivity
  for all 192 DX100 factory voices, extracted from FB-01 ROM. Replaces
  the artificial global velocity floor with real FB-01 data. **86% of
  factory voices now have velocity response** (165/192 patches),
  including classics like IvoryEbony [2,7,0,0], Solid Bass [4,4,4,4],
  and Elec Grand [7,1,1,0].
- **Velocity Mode setting** (Settings → Sound) — Choose between
  **FB-01 (Recommended)** for authentic velocity or **DX100 Classic** for
  no velocity (original hardware behavior). Default: FB-01 mode enabled.
  Removed artificial "Global Velocity Sensitivity" (Off/Low/Med/High/Max)
  — replaced with authentic FB-01 data.

### Migration from v0.9.0

- Old `globalVelocitySens` setting is discarded
- New `fb01VelocityMode` defaults to `true` (recommended)
- Factory patches now use authentic FB-01 per-operator KVS when mode is
  enabled
- User patches remain unaffected (KVS values preserved as-is)
## v0.9.0 — 2026-04-01 — Rebrand, Audio Clicking Fix, Velocity Sensitivity

### Breaking Changes

- **DCent → XCent Rebrand** — Plugin renamed to XCent. New logo: striped
  X with pointed C. All code, UI, docs, and build system updated. DAW
  projects using DCent plugin will need to reload/replace instances.

### Features

- **Global Velocity Sensitivity** — Settings → Sound → Velocity
  Sensitivity (Off/Low/Med/High/Max). Acts as KVS floor for patches
  lacking per-operator velocity (all DX100 factory patches have KVS=0).
  Default: Medium (KVS=3). Persists in settings.xml.
- **Display Style Persistence** — Matrix Display setting (Stock LCD/OLED/
  FB-01 Amber) now persists across restarts via settings.xml (was
  previously lost).
- **Improved Settings Layout** — Theme and Window Scale moved to
  settings.xml only (removed from DAW state to prevent conflicts).
  Appearance settings now use two-column layout with no scrolling.
- **UI polish.**
  - **Stock LCD Display Mode** — Default display style is now "Stock LCD"
    (light gray on dark). OLED and FB-01 Amber modes available in
    settings.
  - **Algorithm/Envelope Display** — Algorithm display scaled to 95%,
    envelope/algo text 150% larger for better readability.
  - **Grid Slot Numbers** — Match mode button styling (9px bold white).
  - **VU Meter Border** — Aligned with CPU meter for visual consistency.
  - **Default Scale** — Changed from 100% to 125%. Added 125% option to
    settings dropdown.
  - **Light Mode Button Text** — Uses panel cream color instead of black.
    Algorithm SVG inverted for proper contrast.
  - **Factory Patch on Fresh Start** — Plugin now loads factory patch A:0
    on first launch instead of INIT VOICE.

### Fixes

- **Rapid Note Clicking Eliminated** — Fixed phase discontinuities during
  register writes that caused audible clicks on rapid note-ons (40% of
  notes in fast arpeggios). NukedEngine now buffers all audio produced
  during register write clocks (88 clocks per register = ~80 samples per
  note-on) into a ring buffer. `renderOneSample()` drains buffered audio
  first before clocking fresh samples. Matches real hardware behavior
  where YM2164 produces continuous audio independent of CPU register
  writes. See `Docs/comparison-report-2026-04-01-fast-clicks.md` for
  analysis.
## v0.8.4 — 2026-03-29 — Display Fixes, CPU Optimization, Accessibility

### Bug Fixes
- **Master Tune Display** — Parameters like Master Tune now display their actual values (e.g., `F M. Tune=0`) instead of `--`. Added `getMasterTune()` accessor and send Master Tune as `PARAM_UPDATE` when patches load.
- **EZ Mode Double-Click Reset** — Double-clicking knobs/sliders in EZ edit mode now resets to the **value from the loaded patch**, not init defaults (0). Added `loadPointVoiceParams` snapshot system.
- **Display Format for Patch-Level Parameters** — Patch-level parameters (Master Tune, Pitch Bend Range, etc.) now show `F` prefix instead of full `E1111` operator status. Operator-scoped parameters still show full status (e.g., `E1111 F=18.37OP1` vs `F PBend Range=12`).

### Accessibility (WCAG 2.1 AA)
- **Icon Buttons** — Settings and About buttons now have `aria-label` for screen reader accessibility
- **Decorative SVGs** — All non-interactive SVG icons marked with `aria-hidden="true"`
- **Interactive Controls** — EZ mode knobs/sliders have proper `role="slider"` with ARIA value attributes and keyboard focus (`tabIndex={0}`)
- **Modal Focus Traps** — All modals (Browser, Settings, About) now trap keyboard focus and cycle Tab navigation correctly. Created `useFocusTrap` hook.
- **Form Labels** — All input fields have associated labels or `aria-label` attributes. Added `.visually-hidden` utility class for screen-reader-only labels.
- **Modal Semantics** — All modals have `role="dialog"`, `aria-modal="true"`, and proper labeling

### Performance
- **CPU Idle Optimization** — Plugin now detects when no notes are playing and skips expensive FM engine rendering. CPU usage drops from ~17% idle to <1% idle. Uses 3-second tail period to ensure release envelopes complete naturally. Tail duration matches `getTailLengthSeconds()` and covers worst-case RR=0 decay (2.57s).

### UI Polish
- **About Modal Links** — Website link now goes directly to https://knivesonstrings.com/xcent. GitHub link updated to https://github.com/Knives-On-Strings/xcent. "Docs" renamed to "Manual".
- **Version Display** — Now correctly shows v0.8.4 (fixed UI package.json from 0.7 to 0.8).
## v0.8.0 — 2026-03-28 — UI Overhaul, MIDI Completion, DSP Accuracy

### New Features
- **Light Mode** — Full dark/light theme system via CSS custom properties (229 var() usages across 16 components). Toggle in Settings → Appearance → Skin. Persists across sessions.
- **OLED Display Mode** — Blue OLED on Black matrix display style. Toggle in Settings → Appearance → Matrix Display. Instant switch, no restart needed.
- **Tabbed Settings Modal** — Reorganized into Sound, Appearance, and MIDI/Audio tabs.
- **MIDI Receive Channel** — Omni (all) or specific channel 1-16. Persists in DAW session state.
- **Vintage/Modern Engine Toggle** — Bypass authentic DX100 analog output stage for a cleaner sound. Settings → Sound → Engine.
- **Recall Edit** — FUNC grid button restores voice to its state at last patch load.
- **Init Voice** — FUNC grid button resets to init voice (also available in patch browser).
- **CPU Meter** — Live CPU load from processBlock timing with EMA smoothing, green/yellow/red color coding, percentage overlay.
- **Window Scale** — 50%, 75%, 100%, 150%, 200% options. Persists across sessions.
- **On-Screen Keyboard** — Brighter ivory white keys, scaled to fit window (no scroll), shorter profile.

### MIDI Implementation (Complete)
- **CC5** — Portamento Time (overrides voice patch value)
- **CC7** — Volume (attenuates carrier operators via TL offset, preserves timbre)
- **CC65** — Portamento Switch (on/off)
- **CC126** — Mono Mode On
- **CC127** — Poly Mode On
- Full MIDI receive chart now matches DX100 hardware specification.

### DSP Accuracy
- **LFO Delay Fix (DSP14)** — Completely rewritten two-phase delay model verified against ROM firmware register dumps. Phase 1: countdown of LFD×3 ticks. Phase 2: linear ramp at rate/64 PMD per tick. Previous implementation was ~20x too fast. Matches ROM within ±1 PMD at every sampled tick.
- **Master Tune Fix** — Data entry knob and INC/DEC buttons now work correctly with M.TUNE parameter.

### Code Quality
- **PluginEditor refactored** — 39 event handlers split from one 588-line block into 5 logical methods.
- **PluginProcessor cleanup** — ROM firmware state extracted into RomState sub-struct.
- **pluginval** — Passes at strictness level 10 (maximum). Factory preset name fix (getProgramName).
- **Build system** — `scripts/build.sh` auto-increments build number. Binary data refresh on every build.
## v0.7.1 — 2026-03-26 — Frequency Table Fix & DSP Accuracy Overhaul

### Critical Fix
- **Operator Frequency Lookup Table** — The VCED `freq` parameter (0-63) was incorrectly decoded using `MUL=freq>>2, DT2=freq&3`. The DX100 actually uses a **sorted ratio lookup table** where all 64 MUL×DT2 combinations are ordered by ascending frequency ratio. **53 of 64 freq values mapped to the wrong MUL/DT2 pair**, affecting every factory voice that uses freq ≥ 10. Verified against DX100 owner's manual page 34 and hardware front-panel readout. Example: D01 Glocken OP2 was incorrectly at MUL=4 (ratio 4.0) instead of MUL=5 (ratio 5.0), producing wrong odd harmonics (3rd/5th) instead of correct even harmonics (4th/6th).

### DSP Accuracy Improvements
- **Output Filter Raised (fc 10600→20000 Hz)** — 192 kHz hardware reference recordings show the real DX100 analog filter is nearly transparent below 10 kHz. The previous fc=10600 Hz over-attenuated high harmonics by ~2.5 dB at 8 kHz. At fc=20000 Hz, attenuation is only -1.1 dB at 8 kHz, matching hardware.
- **Carrier Normalization Disabled** — The per-algorithm TL offsets (ALG 4: +8, ALG 7: +16) extracted from ROM firmware register dumps were making carrier operators 6-12 dB too quiet. Hardware recordings prove these offsets are NOT applied during normal playback. Levels now match hardware within ±0.3 dB.

### Accuracy Results (192 kHz references)
- D01 Glocken (ALG=4): harmonics within **±0.9 dB**, level within **±0.3 dB**, decay rate within **±0.5 dB/s**
- D06 Good Vibes (ALG=7): harmonics within **±0.2 dB**, level within **±0.1 dB**
- D15 FM SQUARE: harmonics within **±0.2 dB**
- D17 FMSAWTOOTH: harmonics within **±0.3 dB**
- B01 Solid Bass: harmonics within **±1.0 dB**
- No ultrasonic energy above 10 kHz in any hardware recording (aliasing is not a factor)
## v0.7.0 — 2026-03-25 — ROM Firmware & DSP Accuracy

### New Features
- **ROM Firmware Audio Path (FW2)** — The actual DX100 ROM firmware now runs on an HD6303 CPU emulator inside `processBlock`. The firmware handles all MIDI→YM2164 register translation with 100% firmware accuracy. Runtime toggle between FirmwareLogic (default, no ROM needed) and ROM firmware (authentic, requires ROM file). CPU load ~19% at 48 kHz.
- **ROM Settings UI** — Settings modal now includes ROM Firmware section with file browser, status indicator (loaded/missing/invalid), and mode toggle.
- **SysEx Export** — Save current voice as single VCED (.syx, 101 bytes) or current bank as 32-voice VMEM (.syx, 4104 bytes). File save dialog via UI.
- **SysEx Import in ROM Mode** — Loading a .syx file in ROM firmware mode writes VMEM voice data directly to the emulated NVRAM.
- **Plugin Scanner Detection (COMPAT1)** — Skips heavy initialization when loaded by DAW plugin scanners/validators, preventing scan timeouts.

### DSP Accuracy Improvements
- **VMEM Byte 45 Packing Fix** — Traced firmware's unpack routine at $974B. The actual bit layout is `PMS(7:4)|AMS(3:2)|LW(1:0)`, not what was previously assumed. **All 192 factory voices** had incorrect PMS and AMS values. Regenerated from SysEx dumps with corrected bit masks. Most voices had PMS too low (e.g., PMS=3 instead of PMS=6).
- **OL→TL Formula Fix** — The firmware uses `99 - OL` (for OL ≥ 20) plus a 20-entry prefix table (ROM $8C75), NOT the 100-entry lookup table at $B77E. The $B77E table is used only for KS Level combined lookups. This changes the TL value for every note-on.
- **KS Level Formula Fix** — Replaced linear `(LS * KC) >> 9` with octave-based exponential scaling using per-octave factors `[0, 1, 3, 7, 16, 33, 66, 127]` extracted from ROM firmware. The adjustment now correctly doubles per octave.
- **ALG Carrier Normalization** — The firmware adds a TL offset to carrier operators based on algorithm carrier count: ALG 0-3 (1 carrier) +0, ALG 4 (2 carriers) +8, ALG 5-6 (3 carriers) +13, ALG 7 (4 carriers) +16. Prevents clipping when multiple carriers sum to output.
- **Velocity Sensitivity Curve** — Replaced linear approximation with ROM-extracted nonlinear curve. Per-KVS maximum attenuation table `[0, 18, 29, 40, 51, 62, 73, 78]`. Floor offset `max(0, 7-KVS)` at maximum velocity. Concave curve shape.
- **ADC Pitch Wheel Centering** — Fixed firmware boot leaving pitch wheel at minimum instead of center, causing notes to play 7 semitones low on non-center-transpose voices.
- **Mod Wheel Position Fix** — Fixed firmware inverting ADC value 0 to position $FE (full on), causing PMD to be written as 72 instead of 8.

### Accuracy Results
- FirmwareLogic register writes: **within ±1 TL** of ROM firmware on all tested voices
- B01 Solid Bass spectral centroid: **1.09x** hardware (native rate)
- C06 S/H Synth spectral centroid: **1.12x** hardware (native rate)
- Remaining gap is from LFO phase/LFSR state differences (inherent, not fixable)
## v0.6 — 2026-03-21 — Initial Release

- Full 4-operator FM synthesis via Nuked-OPM (cycle-accurate YM2164)
- FirmwareLogic MIDI→register translation
- React + Tailwind CSS WebView UI
- 192 factory voices across 8 banks
- SysEx import (VCED + VMEM)
- Analog output stage (reconstruction filter + DC blocking)
- Sinc resampling (55930 Hz → host rate)
- VST3 + CLAP + Standalone formats
