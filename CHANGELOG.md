# XCent Release Notes

What's new, what's better and what's fixed in each version of XCent, newest
first. XCent is an emulation of the Yamaha DX100 by Knives on Strings.

## v0.15.44 -- 2026-10-10 -- FB-01 sounds import correctly

### Improved

- FB-01 velocity mode now also gives factory patches the FB-01's attack
  response where the FB-01 sound has one: play harder and the attack gets
  faster, softer and it gets slower.
- Whistling now gets its FB-01 velocity response in FB-01 velocity mode.

### Fixed

- Importing a Yamaha FB-01 bank now brings in its real sounds (the first 24
  of its 48). Before, the voices came in scrambled, and many were silent.

## v0.15.43 -- 2026-10-10 -- Factory patches play like a DX100

### Improved

- FB-01 velocity mode is now off by default, so factory patches respond to
  velocity exactly like a DX100. It has been switched off once for everyone;
  turn it back on in Settings, Sound, Velocity Mode.
- The tour has a new step that explains FB-01 velocity mode and lets you turn
  it on, so it runs once more after this update.

### Fixed

- FB-01 velocity mode gave most factory patches the wrong velocity response;
  IvoryEbony, for one, lost its brightness when played softly.
- FB-01 velocity mode now gives each factory patch the response of the same
  sound on the Yamaha FB-01. Patches the FB-01 lacks keep their own.

## v0.15.42 -- 2026-10-10 -- Attacks match the DX100

### Fixed

- Attacks now match the DX100: many voices, such as IvoryEbony and Hard
  Brass, no longer start slow or dull.

## v0.15.41 -- 2026-10-09

- Maintenance release -- no changes you'll notice.

## v0.15.40 -- 2026-10-09

- Maintenance release -- no changes you'll notice.

## v0.15.39 -- 2026-10-09 -- Release notes written for players

### Improved

- These release notes are new: short, plain-language notes about what changed
  for you. They replace the developer changelog in the Windows installer, the
  Windows zip and the Linux package.

## v0.15.38 -- 2026-10-09

- Maintenance release -- no changes you'll notice.

## v0.15.37 -- 2026-10-09 -- A tidier header

### Fixed

- The XCent logo now sits centred in the header, and the OUTPUT knob is no
  longer cut off at the right edge of the window.

## v0.15.36 -- 2026-10-08

- Maintenance release -- no changes you'll notice.

## v0.15.35 -- 2026-10-08

- Maintenance release -- no changes you'll notice.

## v0.15.34 -- 2026-10-08

- Maintenance release -- no changes you'll notice.

## v0.15.33 -- 2026-10-08 -- Plain punctuation in the licence texts

### Improved

- The licence agreement and the third-party notices now use plain punctuation,
  so they display cleanly in any text editor. Not a word of the terms changed.

## v0.15.32 -- 2026-10-08

- Maintenance release -- no changes you'll notice.

## v0.15.31 -- 2026-10-08 -- The whole licence, up front

### Improved

- Both installers now show the complete XCent licence agreement on their
  licence page, word for word the copy they install, instead of a summary.

## v0.15.30 -- 2026-10-08 -- Mac release notes move online

### Improved

- The macOS disk image now links to the release notes on the Knives on Strings
  website, which always cover the newest version, instead of carrying a copy.

## v0.15.29 -- 2026-10-07 -- New contact addresses

### Improved

- Our contact addresses have moved: support@knivesonstrings.com for support,
  refunds and general questions, privacy@knivesonstrings.com for privacy, and
  legal@knivesonstrings.com for licensing.

## v0.15.28 -- 2026-10-07 -- Undo keeps your edits

### Fixed

- Undo and redo no longer rename the patch you are editing to INIT VOICE or
  reset its pitch envelope.
- A project saved after an undo no longer throws your edits away when it
  reopens: your edited patch comes back, still marked as modified.
- A project saved while you were building a sound from Init Voice now reopens
  with your sound, not the stored patch.

## v0.15.26 -- 2026-10-06 -- The modified indicator stays honest

### Fixed

- When a reopened project's patch no longer matches its bank slot, the modified
  indicator now stays on until you load, store or recall a patch, even if you
  close and reopen the window.
- When your DAW restores XCent with its window open, for example a preset or
  snapshot recall, the display now switches to the restored patch.
- Renaming a patch no longer marks your unsaved edits as saved, and the chorus
  switch, master tune and undo now mark the patch as modified in the project.

## v0.15.25 -- 2026-10-05 -- Licence agreement completed

### Improved

- The licence agreement now names the publisher, the governing law and the
  privacy policy, and states the refund policy: a full refund on request within
  30 days of purchase.
- It also points to the right place to opt out of anonymous usage data:
  Settings > Appearance > Privacy.

## v0.15.24 -- 2026-10-05 -- New audio options for the Windows Standalone

### New

- The Windows Standalone now plays through Windows Audio, with shared,
  exclusive and low-latency modes, or through DirectSound.
- ASIO is no longer included in the Standalone; if you used it, choose your
  audio interface under Windows Audio. The VST3 and CLAP plugins are unaffected.

## v0.15.21 -- 2026-09-16 -- Intel Macs and an honest manual

### New

- On macOS, every format now runs natively on Intel Macs as well as on Apple
  Silicon.
- macOS 15 Sequoia or later is now required, and the installer checks your Mac
  before it installs anything.

### Improved

- The macOS installer always puts each format in its standard folder, even when
  another copy of XCent exists elsewhere on the Mac.
- The manual now describes correctly where XCent keeps your data, how the
  banks are laid out and how to install.
- The manual in every download is printed fresh for that exact release.

### Fixed

- macOS: the plugins and the Standalone now load on every Mac, not only on the
  one they were built on.
- The manual's eight FM algorithm diagrams now appear, instead of broken images.
- The Linux installer now sends you to the right place for support.

## v0.15.20 -- 2026-09-16 -- The same look on every computer

### Improved

- The screens XCent shows when its interface cannot start now use the same
  typeface as the interface.
- The Windows installer and zip now include the licence agreement, the
  third-party notices and the full third-party licence texts.

### Fixed

- On computers without its fonts installed, the interface no longer falls back
  to Courier New and borrowed symbols: every font now ships with XCent.
- The About box now shows the version you actually installed; since 0.15.12 it
  always said 0.15.11.

## v0.15.19 -- 2026-09-08

- Maintenance release -- no changes you'll notice.

## v0.15.18 -- 2026-09-07 -- Clean exits

### Fixed

- Closing the Standalone, or quitting a DAW with XCent loaded, no longer ends in
  a crash report. Nothing was ever lost, but Windows logged an error.
- A project saved on a factory patch no longer shows that patch as modified
  when it reopens.

## v0.15.17 -- 2026-09-07 -- Move your XCent folder

### New

- For portable or multi-profile setups, the folder where XCent keeps your
  patches, preferences and log can now be moved; the manual explains how.

## v0.15.16 -- 2026-09-06 -- CLAP hosts show the right values

### Fixed

- CLAP hosts now show the right parameter values after loading a project. The
  values were always restored, but the host could keep showing the old ones.

## v0.15.15 -- 2026-09-06 -- No false alarm on slow-opening windows

### Fixed

- In DAWs that create the plugin window well before showing it, a working
  interface is no longer replaced by an error panel after about 8 seconds.

## v0.15.14 -- 2026-09-06 -- No more white window

### Fixed

- Opening XCent in a VST3 host could show a blank white window instead of the
  interface, with no way out. XCent now recovers on its own as the window opens.

## v0.15.13 -- 2026-09-06

- Maintenance release -- no changes you'll notice.

## v0.15.12 -- 2026-09-05 -- Loads in Cubase and Nuendo

### Fixed

- Windows: XCent now loads in Cubase, Nuendo and other hosts that use
  Steinberg's plugin loader, which refused it before.
- If XCent's sound engine is ever missing from its folder, the plugin still
  loads, stays silent and explains what is wrong, instead of failing.

## v0.15.11 -- 2026-05-17 -- Report an issue from inside XCent

### New

- A Report an Issue button in the About box opens a form for sending us a bug
  report.
- XCent sends anonymous usage data, tied to a random install ID and no personal
  data. The first-run tour asks, and Settings > Appearance > Privacy switches it
  off at any time.
- The macOS disk image includes an uninstall script that removes all four
  formats and keeps your patches unless you ask it to remove them too.

### Improved

- All your preferences are now stored together in XCent's folder in Documents.

## v0.15.1-rc2 -- 2026-05-14 -- Portamento like the hardware

### Improved

- Portamento now glides at the DX100's speed, where it was about half as fast,
  and as on the hardware it only engages in mono mode when you play legato.

## v0.15.0-rc2 -- 2026-05-12 -- Expression control and a Mac installer

### New

- XCent responds to MIDI CC11 (Expression), scaling the volume on top of CC7 so
  you can ride expression separately.
- A macOS installer, on a disk image, installs the VST3, CLAP, Audio Unit and
  Standalone.

### Fixed

- macOS: the Standalone no longer forgets settings made during a session when
  you close its window, and quitting no longer crashes.
- Issue reports sent from the Standalone are no longer rejected.
- When a project's patch has changed in its bank since you saved, XCent now
  shows it as modified, so you know the sound is not what the slot holds.

## v0.14.0-rc1 -- 2026-05-09 -- A guided tour and tooltips

### New

- A first-run tour walks you through modes, operators, editing, Modern view,
  Func mode and the patch browser. Replay it from Settings > Appearance.
- Hover over an Edit or Func button to see the parameter's full name and range.
- A setting opens Edit mode straight into Modern view, the new name for EZ mode.

### Improved

- The Appearance settings are laid out in two columns, and the patch browser
  has clearer type and contrast.

### Fixed

- Tooltips, right-click menus and the tag picker no longer drift off-screen at
  window scales other than 100%.
- The modified indicator now changes as soon as you edit a parameter.

## v0.12.7 -- 2026-05-07 -- Breath control fixed

### Fixed

- Breath-controller patches now respond to CC2 the right way round, quieter at
  low breath as on the hardware, even for notes played before any breath data.
- Cubase 15 and Nuendo 15 no longer show a blank white window instead of the
  interface.

## v0.12.6 -- 2026-05-05 -- Better answers when the window stays blank

### Fixed

- If a Windows "optimisation" tool has removed parts of WebView2, XCent now
  offers to reinstall it instead of showing a white screen.
- When a sandboxed host stops XCent's interface from saving its cache, XCent
  now says so and explains the workaround, instead of showing a white screen.

## v0.12.5 -- 2026-05-05 -- Recovery from a blank window

### New

- If the interface fails to load, XCent shows a panel explaining what went
  wrong, with buttons to open the log folder and clear the interface cache.

### Improved

- The Windows installer clears the interface cache when it installs and
  uninstalls, so each version starts fresh. Your patches and settings are kept.

## v0.12.4 -- 2026-05-03 -- Idle means idle

### Fixed

- With a keyboard or clock sending MIDI, the CPU meter showed 30-50% while
  nothing played. XCent now rests at 0% when nothing is playing.

## v0.12.3 -- 2026-05-03 -- Undo that remembers

### Improved

- The mouse wheel now changes parameters 50% faster.

### Fixed

- Undo keeps its full 50-step history, and redo works across several steps.
- The display no longer cuts off the names of patch-wide parameters such as
  pitch-bend range.

## v0.12.2 -- 2026-05-03 -- Wheels and velocity

### New

- The on-screen pitch-bend and mod wheels respond to the mouse wheel, and pitch
  bend springs back to the centre on its own.

### Improved

- Velocity sensitivity now follows the DX100's own response. Soft notes on
  velocity-sensitive patches were up to 10 dB too quiet.

### Fixed

- A voice received over MIDI SysEx shows its own name and comes back correctly
  when the project reopens.
- Loading a patch or using undo no longer confuses the modified indicator.
- The meters stop using processor time once they settle at zero.

## v0.12.0 -- 2026-04-29 -- Pitch envelope done properly

### Improved

- The pitch envelope now behaves like the hardware's: DX21, TX81Z and DX11
  voices move through its stages at the hardware's rates, and DX100 and FB-01
  voices jump straight to the first level.
- Pitch envelope levels now span the hardware's full four octaves either way.

## v0.11.10 -- 2026-04-27 -- Closer to the hardware

### Improved

- XCent's output level now matches a real DX100; it was about 3 dB quieter.
- LFO vibrato and tremolo depth now scale like the hardware's, audibly
  improving patches such as Good Vibes, Helicopter and Windbells.

## v0.11.9 -- 2026-04-25 -- Mode letters on the display

### New

- In Play mode the display shows a P before the patch name, as the DX100 does.
- A Modified Indicator setting chooses whether an edited patch is shown with a
  lowercase mode letter or an asterisk.

### Fixed

- Patch-wide parameters in Edit mode, and the Func mode screen, now show the
  right mode letter.

## v0.11.8 -- 2026-04-21 -- Scroll to edit

### New

- Scroll over an EZ mode knob or slider to change it: hold Ctrl or Cmd for big
  steps and Shift for fine ones, or switch it off in Settings > Appearance.
- Smarter automatic tagging recognises more leads, brass, organs, percussion
  and strings.

### Fixed

- The factory banks are now read-only everywhere: Store is disabled on them,
  with a padlock and a tooltip saying why.
- A 32-voice bank dump received over MIDI is now saved to bank A, and all of
  its names show in the browser.

## v0.11.7 -- 2026-04-19 -- A Windows installer

### New

- A Windows installer puts the VST3, CLAP and Standalone in place, for all
  users or just you, and installs Microsoft's WebView2 runtime if it is missing.
- It adds Start Menu shortcuts and an uninstaller that leaves your patches and
  settings alone.

## v0.11.6 -- 2026-04-21 -- A clear message instead of a blank window

### Fixed

- On Windows without a recent WebView2 runtime, XCent now explains what is
  missing, with a button to install it, instead of showing a blank window.

## v0.11.5 -- 2026-04-20 -- Export to SysEx

### New

- Save a voice or a 32-voice bank as SysEx for the DX100, DX21 or TX81Z.
- XCent suggests the format that keeps the most of the voice, and lists exactly
  what any other format would drop.

## v0.11.4 -- 2026-04-20 -- Nothing lost in your library

### Improved

- Your patch library is now stored in XCent's own format, which keeps every
  detail of a voice. SysEx files are still imported as before.

## v0.11.3 -- 2026-04-20 -- More SysEx imports

### New

- Import TX81Z single voices and banks, with their extra waveform and
  fixed-frequency settings.
- DX11 and DX21 voices are recognised automatically on import.

## v0.11.2 -- 2026-04-20

- Maintenance release -- no changes you'll notice.

## v0.11.1 -- 2026-04-20 -- More even levels

### Improved

- Factory patches now play at much more even levels across algorithms; a
  four-carrier patch could be about 12 dB louder than a one-carrier one.

### Fixed

- Factory tags now show for the patches in the writable banks.
- A project saved on an untouched Init Voice reopens on its saved patch, and a
  fresh install always starts on the first factory patch.

## v0.11.0 -- 2026-04-20 -- DX21 chorus and FB-01 voices

### New

- DX21 chorus, modelled from hardware measurements, with a CHORUS button to
  switch it on for any patch.
- FB-01 bank import, including each operator's attack velocity sensitivity.
- Factory patches are numbered as on the hardware: 101-124 in bank 100, 201-224
  in bank 200, and so on.

### Improved

- Double-click the volume knob to reset it to 0 dB.

### Fixed

- On some Windows setups OneDrive could stop the interface from starting; its
  cache no longer lives in Documents.

## v0.10.0 -- 2026-04-19 -- The patch browser

### New

- A patch browser with search and tag filters, across all 192 factory voices and
  your own banks.
- Every factory voice comes tagged as bass, pad, keys, lead, brass, bell and
  more, and you can add tags of your own.

## v0.9.14 -- 2026-04-06 -- Bend and mod on screen

### New

- On-screen pitch-bend and mod wheels beside the keyboard.

## v0.9.1 -- 2026-04-02 -- FB-01 velocity

### New

- FB-01 Velocity Mode gives 165 of the 192 factory voices per-operator velocity
  response taken from the Yamaha FB-01. Choose it or DX100 Classic in
  Settings > Sound.

## v0.9.0 -- 2026-04-01 -- DCent becomes XCent

### New

- The plugin is now called XCent, with a new logo. Projects that used DCent
  need their instances replaced.

### Improved

- A new Stock LCD display style, and your display style now survives a restart.
- A fresh install starts on a factory patch instead of Init Voice.

### Fixed

- Fast arpeggios no longer click.

## v0.8.4 -- 2026-03-29 -- Accessibility and idle CPU

### Improved

- Screen-reader labels, keyboard focus on knobs and sliders, and keyboard
  navigation inside dialogs.
- With no notes playing, XCent drops from about 17% CPU to under 1%.

### Fixed

- Master Tune now shows its value on the display, and double-clicking a knob
  resets it to the patch's own value.

## v0.8.0 -- 2026-03-28 -- Light mode and full MIDI

### New

- Light mode, an OLED display style, tabbed settings, a window size setting and
  a CPU meter.
- Recall Edit and Init Voice buttons, and a MIDI receive channel setting.
- A Vintage and Modern engine switch; Modern bypasses the analog output stage
  for a cleaner sound.
- MIDI control of portamento, volume and mono or poly mode, matching the DX100's
  MIDI chart.

### Fixed

- LFO delay now matches the hardware; it was about 20 times too fast.

## v0.7.1 -- 2026-03-26 -- The right harmonics

### Fixed

- Operator frequency ratios were wrong for 53 of the 64 settings, affecting
  most factory voices. They now match the DX100.
- A brighter output filter and corrected levels bring XCent within a fraction
  of a decibel of hardware recordings.

## v0.7.0 -- 2026-03-25 -- An accuracy pass

### New

- Save the current voice or bank as a SysEx file.
- Faster plugin scans: XCent skips its heavy start-up while a DAW is scanning.

### Fixed

- All 192 factory voices had the wrong pitch and amplitude modulation
  sensitivity. Corrected.
- Velocity response, keyboard level scaling and operator levels now follow the
  DX100's own curves.

## v0.6 -- 2026-03-21 -- First release

### New

- Four-operator FM synthesis on a cycle-accurate emulation of the DX100's
  Yamaha YM2164 chip.
- All 192 factory voices, across eight banks, with SysEx import.
- The DX100's analog output stage, modelled.
- VST3, CLAP and Standalone formats.
