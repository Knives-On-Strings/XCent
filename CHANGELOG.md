# XCent Release Notes

What's new, what's better and what's fixed in each version of XCent, newest
first. XCent is an emulation of the Yamaha DX100 by Knives on Strings.

ASIO is a trademark of Steinberg Media Technologies GmbH, registered in Europe
and other countries.

## v0.15.92 -- 2026-11-04

### Documentation

- **Improved:** These release notes are now grouped by part of XCent -- sound engine, patches, interface, installers and so on -- and maintenance releases say what they changed.

## v0.15.91 -- 2026-11-03

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.90 -- 2026-11-03

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.89 -- 2026-11-03

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.88 -- 2026-11-02

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.87 -- 2026-11-01

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.86 -- 2026-11-01

### UI

- **Improved:** When a new version is out, a small red dot on the About button tells you; About shows the new version and links to what changed.

## v0.15.85 -- 2026-11-01

### Maintenance

- Built on JUCE 9.0.4.

## v0.15.84 -- 2026-11-01

### Sound engine

- **Fixed:** Turning master tune while notes are playing no longer risks a glitch.

## v0.15.83 -- 2026-11-01

### Maintenance

- Groundwork for the upcoming ROM mode; nothing changes in normal use.

## v0.15.82 -- 2026-10-31

### Maintenance

- Groundwork for the upcoming ROM mode; nothing changes in normal use.

## v0.15.81 -- 2026-10-31

### Maintenance

- Internal code tidy-up only; nothing changes in how XCent sounds or behaves.

## v0.15.80 -- 2026-10-31

### Patches & ROM

- **Improved:** Opening the patch browser and saving patches no longer repeat work.

### UI

- **Improved:** The interface uses less CPU while idle.
- **Improved:** The scope window draws with far less overhead.

## v0.15.79 -- 2026-10-31

### Sound engine

- **Improved:** Editing a sound while it plays is smoother: no more glitches or late notes while you turn a knob, and the LFO keeps running steadily during edits.
- **Fixed:** LFO Sync now restarts the LFO at every note, as on the original hardware.

## v0.15.78 -- 2026-10-30

### Patches & ROM

- **Fixed:** Switching back from ROM mode no longer leaves notes hanging; the next note plays the current sound.

## v0.15.77 -- 2026-10-30

### Maintenance

- Test cleanup only; the plugin itself is unchanged.

## v0.15.76 -- 2026-10-30

### Sound engine

- **Fixed:** Sounds with a slow release now fade out fully instead of stopping 3 seconds after you let go, and a note held in ROM mode no longer goes silent.

### Patches & ROM

- **Improved:** The patch browser remembers where you were: it reopens on the same bank, scrolled to the sound you are playing.
- **Improved:** Smoother behaviour under heavy MIDI in ROM mode.
- **Fixed:** Saving a bank can no longer leave it damaged if the computer or your host crashes in the middle of the save.

### UI

- **Fixed:** Very fast playing on the on-screen keyboard can no longer leave a note stuck.

### Hosts & formats

- **Improved:** XCent now tells your host how long each sound's tail really is, so bounces include the whole release.

## v0.15.75 -- 2026-10-28

### Patches & ROM

- **Improved:** In the patch browser, the DX100, FB-01 and My Banks groups can each be collapsed and expanded.

### UI

- **New:** The level meter is now stereo, with a column for the left channel and one for the right, so you can see the chorus spread the sound.
- **New:** Settings shows where your own patches are saved, with a button to open that folder.

### Hosts & formats

- **Improved:** The licence agreement in the Windows and Mac installers now reads cleanly instead of wrapping awkwardly.

## v0.15.74 -- 2026-10-27

### Sound engine

- **Fixed:** Fast repeated notes, trills and chords no longer click. Notes now take turns on all eight voices, as on the DX100, so a released note rings out under the next one instead of being cut off.
- **Fixed:** In mono mode, notes played legato change pitch smoothly without restarting the sound, as on the DX100.
- **Fixed:** Big chords that arrive all at once, for example from a sequencer, no longer come out garbled.
- **Fixed:** Editing a sound while a chord is held no longer changes the attack of the held notes.

### Patches & ROM

- **Fixed:** In ROM mode, the first moments after the original firmware starts, or after a bank file loads, no longer buzz or go silent.

## v0.15.73 -- 2026-10-26

### Maintenance

- Tests, diagnostics and documentation only; the plugin itself is unchanged.

## v0.15.72 -- 2026-10-26

### Sound engine

- **New:** MIDI CC11 (Expression) can now control the volume, the level of the carriers, the level of every operator, or nothing.
- **Improved:** Pitch bend is smooth, as on the DX100, instead of moving in semitone steps.
- **Improved:** The highest and lowest keys fold back by an octave, as they do on the DX100.
- **Improved:** Editing a sound no longer changes the level of a note you are holding, as on the DX100; the next note takes the change.

### Patches & ROM

- **New:** The DX100's factory sounds are grouped under their own heading in the browser.
- **Fixed:** The browser's Favorites and Recent buttons switch off again with a second click.

### UI

- **Fixed:** If you have not agreed to share usage data, XCent keeps no identifier and deletes any it stored before.

### Hosts & formats

- **Improved:** Download files now follow one naming pattern on every platform: product, version, system and processor type.

## v0.15.71 -- 2026-10-24

### UI

- **Improved:** Privacy settings now have their own Privacy tab in Settings.

## v0.15.70 -- 2026-10-24

### Sound engine

- **Fixed:** Sounds no longer change tone or level abruptly between C and C# as you
  play up the keyboard. Key level scaling now changes a little every three
  notes, exactly as on the DX100 and the DX21.

## v0.15.69 -- 2026-10-23

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.68 -- 2026-10-23

### Maintenance

- Mac and Windows builds now round their arithmetic identically; the difference was far below anything you could hear.

## v0.15.67 -- 2026-10-23 -- FB-01 velocity mode retired

### Sound engine

- **Improved:** Factory patches in banks 100 to 400 and A to D now always respond as on a
  DX100. The FB-01 velocity mode setting is gone; if you had it on, these
  patches now play as a DX100 plays them.
- **Improved:** Projects saved with FB-01 velocity mode on open with the DX100 response.
  Patches you saved into your own banks while it was on keep the response
  they were saved with.

### Patches & ROM

- **Improved:** To play a factory sound the way the Yamaha FB-01 plays it, choose it from
  the FB-01 section of the patch browser. Those sounds respond to how hard you
  play exactly as on an FB-01.

## v0.15.66 -- 2026-10-23

### Maintenance

- Test tooling only; the plugin itself is unchanged.

## v0.15.65 -- 2026-10-23

### Sound engine

- **Improved:** FB-01 sounds now play with the FB-01's own velocity and keyboard
  response. Sounds you imported from an FB-01 earlier may sound louder,
  especially sounds with several audible layers.

### Patches & ROM

- **New:** The Yamaha FB-01's 240 factory sounds are now in the browser, in five
  banks of their own.
- **Fixed:** A project saved on an FB-01 sound reopens on that sound.
- **Fixed:** A project saved on a library patch no longer reopens as modified.

### UI

- **New:** XCent can tell you when an update is available. It checks weekly by
  default, sends only the product name, and is set in Settings.

## v0.15.64 -- 2026-10-23

### Patches & ROM

- **Improved:** Importing a bank from an FB-01 now keeps all 48 voices, and 32-voice DX21 and
  TX81Z banks keep all 32, instead of being cut short.

### Hosts & formats

- **Improved:** Mac: the installer now also installs the licence texts.
- **Improved:** Each download now comes with a checksum file, so you can check it
  arrived intact.

## v0.15.63 -- 2026-10-23

### Sound engine

- **Fixed:** TX81Z mode no longer applies a pitch envelope. The TX81Z has none, so
  a patch now plays at a steady pitch there, whatever its pitch envelope
  settings.

### Patches & ROM

- **Fixed:** A program change sent right after a bank over MIDI now plays the new
  sound, not the one the bank replaced.
- **Fixed:** A second copy of a RAM bank now gets its own name ("User Bank A 2")
  instead of repeating "User Bank A".

### Hosts & formats

- **New:** The Windows Standalone supports ASIO audio devices again. Choose ASIO
  as the driver type in Settings, then your audio interface.

## v0.15.62 -- 2026-10-14

### Sound engine

- **Fixed:** In DX21 and DX11 modes, the pitch envelope now keeps moving after you
  release a key, as on the hardware, instead of stopping where it was. On
  a DX11, lifting the sustain pedal starts it too.

## v0.15.61 -- 2026-10-14

### Sound engine

- **Improved:** FB-01 velocity mode now responds as an FB-01 does: medium velocity
  sounds as on the DX100, harder playing is louder and brighter, softer
  playing quieter and darker.

### Patches & ROM

- **Improved:** FB-01 bank imports keep each operator's keyboard level scaling.
- **Fixed:** TX81Z banks and single patches now load from, and send to, a real TX81Z
  correctly. Operators 2 and 3 were swapped, and banks were cut to 16
  voices.
- **Fixed:** Factory patches sent to a DX21, DX11 or TX81Z no longer carry a wrong
  pitch envelope setting.
- **Fixed:** A library bank used by a RAM bank can no longer be deleted. A RAM bank
  whose bank is missing gets its factory sounds back.

## v0.15.60 -- 2026-10-13

### Patches & ROM

- **Fixed:** Single-voice SysEx files, and patches sent to XCent over MIDI, now load
  with their own pitch bend range, poly/mono, portamento and foot volume
  settings. Before, these were read in the wrong order.
- **Fixed:** XCent's single-voice exports now write these settings the way a DX100
  reads them.
- **Fixed:** Bank files now load poly/mono, portamento mode and chorus correctly.
  The factory patches Mono Bass, Mono Lead and Mono Sax now play mono,
  with fingered portamento, as on the DX100.
- **Fixed:** Exported patches keep the foot switch set to sustain instead of
  switching the sustain pedal off.
- **Fixed:** DX100 single-voice files are no longer mistaken for DX21 patches with
  chorus switched on.
- **Fixed:** Projects saved with earlier versions keep their patch settings. A
  project left on an unedited Mono Bass, Mono Lead or Mono Sax picks up
  the corrected mono setting.

## v0.15.59 -- 2026-10-13

### Patches & ROM

- **Fixed:** SysEx files you put in your Library folder are now imported the next
  time XCent starts, instead of deleted. A message says how many were
  imported, and your files are left as they are.
- **Fixed:** TX81Z single-voice exports can now be read back by XCent.
- **Fixed:** DX21 bank exports now keep each patch's chorus and foot volume settings.

## v0.15.58 -- 2026-10-13

### Patches & ROM

- **Fixed:** Opening a SysEx bank file now imports the whole bank as a bank of its
  own, named after the file. Before, only its first patch was kept.
- **Fixed:** Opening a SysEx file now tells you when it can't be read or holds no
  patches XCent understands, and a bank with more than 24 patches says how
  many were left out.
- **Fixed:** Bank actions tell you when they fail: new bank, rename, delete and
  assigning a bank to a RAM bank. A bank received over MIDI that couldn't be
  saved is reported too.
- **Fixed:** My Sounds no longer lists its empty slots.
- **Fixed:** A TX81Z single patch now imports as one patch with its own name.

## v0.15.57 -- 2026-10-13

### Patches & ROM

- **Fixed:** Save As now tells you when it can't save, and no longer marks the patch
  saved if it wasn't.
- **Fixed:** Save As waits until the patch is saved before it closes, and saves to My
  Sounds straight away. Before, Save could do nothing until you picked a
  destination.
- **Fixed:** Opening a single-patch SysEx file now tells you when it couldn't be added
  to My Sounds, for example because My Sounds is full.

## v0.15.56 -- 2026-10-12

### Patches & ROM

- **Fixed:** Store in the patch browser no longer says a patch was saved when it could
  not be. It is greyed out for factory banks and for the All patches view.
- **Fixed:** When a patch can't be stored, renamed, deleted or copied, XCent now tells
  you why, for example that the bank is a protected factory bank.

### UI

- **New:** Keyboard shortcuts: Ctrl+O (Cmd+O on the Mac) opens the patch browser,
  Ctrl+K (Cmd+K) shows or hides the on-screen keyboard, the Left and Right
  arrow keys step through patches, and F1, F2 and F3 press PLAY, EDIT and
  FUNC.
- **New:** The shortcuts stay out of the way while you type a name or a window or
  menu is open, and the arrows while a knob or parameter is selected.

## v0.15.55 -- 2026-10-12

### Patches & ROM

- **Fixed:** Renaming, deleting and copying patches in your own library banks now works,
  and the change is kept. Before, it looked done but the bank was unchanged.
- **Fixed:** Saving the current bank as a SysEx file now saves the library bank your
  patch came from. Before, it saved factory bank 100 instead.

### UI

- **Fixed:** Cmd+S saves on the Mac, and Cmd+Shift+S opens Save As.
- **Fixed:** Redo with Ctrl+Shift+Z (Cmd+Shift+Z on the Mac) now works, and shortcuts
  work with Caps Lock on.
- **Fixed:** The manual's list of keyboard shortcuts now shows only the ones that work.

## v0.15.54 -- 2026-10-12

### Patches & ROM

- **Fixed:** Ctrl+S now really saves a patch back to your own library bank. Before, it
  looked saved but the change was gone next time.
- **Fixed:** Saving, renaming or deleting a patch in RAM banks A to D now lasts. Before,
  changes over the factory sounds there were lost when XCent closed.
- **Fixed:** Your first change to a RAM bank now copies it into a new bank of your own,
  User Bank A (B, C, D), which keeps your changes. The factory sounds stay
  untouched in the library.
- **Fixed:** A sound sent to XCent over MIDI is now kept even when XCent's window is
  closed, so your project saves it. A bank sent over MIDI is kept in RAM
  bank A for next time, window open or not.

## v0.15.53 -- 2026-10-11

### Maintenance

- Internal code tidy-up only; nothing changes in what XCent does or sounds like.

## v0.15.52 -- 2026-10-11

### Maintenance

- Internal code tidy-up only; nothing changes in what XCent does or sounds like.

## v0.15.51 -- 2026-10-11

### Maintenance

- Internal code tidy-up only; nothing changes in what XCent does or sounds like.

## v0.15.50 -- 2026-10-11

### Maintenance

- Internal code tidy-up only; nothing changes in what XCent does or sounds like.

## v0.15.49 -- 2026-10-11

### Patches & ROM

- **Fixed:** Sending XCent a voice or a bank over MIDI while it plays no longer risks
  a glitch or dropout in the audio.

### UI

- **Fixed:** Closing XCent's window while a file dialog is open is now safer.

### Hosts & formats

- **Fixed:** On Windows, leftover interface data folders from older versions of XCent
  are now cleaned up once nothing is using them.

## v0.15.48 -- 2026-10-10

### Hosts & formats

- **Fixed:** On older Macs (macOS 10.15 to 11), some of XCent's knobs and switches could
  move on screen without changing the sound. They now work on every macOS
  version XCent supports.

## v0.15.47 -- 2026-10-10

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.46 -- 2026-10-10 -- The whole window in Ableton Live

### UI

- **Fixed:** XCent now opens straight at your saved interface size, instead of opening
  small and then jumping to it.

### Hosts & formats

- **Fixed:** In Ableton Live on Windows, with Auto-Scale Plug-In Window switched on and
  the display set above 100 %, XCent's window showed only the top-left part
  of the interface. It now shows all of it, at any interface size.

## v0.15.44 -- 2026-10-10 -- FB-01 sounds import correctly

### Sound engine

- **Improved:** FB-01 velocity mode now also gives factory patches the FB-01's attack
  response where the FB-01 sound has one: play harder and the attack gets
  faster, softer and it gets slower.
- **Improved:** Whistling now gets its FB-01 velocity response in FB-01 velocity mode.

### Patches & ROM

- **Fixed:** Importing a Yamaha FB-01 bank now brings in its real sounds (the first 24
  of its 48). Before, the voices came in scrambled, and many were silent.

## v0.15.43 -- 2026-10-10 -- Factory patches play like a DX100

### Sound engine

- **Improved:** FB-01 velocity mode is now off by default, so factory patches respond to
  velocity exactly like a DX100. It has been switched off once for everyone;
  turn it back on in Settings, Sound, Velocity Mode.
- **Fixed:** FB-01 velocity mode gave most factory patches the wrong velocity response;
  IvoryEbony, for one, lost its brightness when played softly.
- **Fixed:** FB-01 velocity mode now gives each factory patch the response of the same
  sound on the Yamaha FB-01. Patches the FB-01 lacks keep their own.

### UI

- **Improved:** The tour has a new step that explains FB-01 velocity mode and lets you turn
  it on, so it runs once more after this update.

## v0.15.42 -- 2026-10-10 -- Attacks match the DX100

### Sound engine

- **Fixed:** Attacks now match the DX100: many voices, such as IvoryEbony and Hard
  Brass, no longer start slow or dull.

## v0.15.41 -- 2026-10-09

### Maintenance

- Release tooling only; the plugin itself is unchanged.

## v0.15.40 -- 2026-10-09

### Maintenance

- Test tooling only; the plugin itself is unchanged.

## v0.15.39 -- 2026-10-09 -- Release notes written for players

### Documentation

- **Improved:** These release notes are new: short, plain-language notes about what changed
  for you. They replace the developer changelog in the Windows installer, the
  Windows zip and the Linux package.

## v0.15.38 -- 2026-10-09

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.37 -- 2026-10-09 -- A tidier header

### UI

- **Fixed:** The XCent logo now sits centred in the header, and the OUTPUT knob is no
  longer cut off at the right edge of the window.

## v0.15.36 -- 2026-10-08

### Maintenance

- Plugin files are 8 to 10 percent smaller; the sound is unchanged.

## v0.15.35 -- 2026-10-08

### Maintenance

- Build and test tooling only; the plugin itself is unchanged.

## v0.15.34 -- 2026-10-08

### Maintenance

- Documentation punctuation only; the plugin itself is unchanged.

## v0.15.33 -- 2026-10-08 -- Plain punctuation in the licence texts

### Hosts & formats

- **Improved:** The licence agreement and the third-party notices now use plain punctuation,
  so they display cleanly in any text editor. Not a word of the terms changed.

## v0.15.32 -- 2026-10-08

### Maintenance

- The macOS installer no longer lists a background image it never had; it looks and behaves as before.

## v0.15.31 -- 2026-10-08 -- The whole licence, up front

### Hosts & formats

- **Improved:** Both installers now show the complete XCent licence agreement on their
  licence page, word for word the copy they install, instead of a summary.

## v0.15.30 -- 2026-10-08 -- Mac release notes move online

### Hosts & formats

- **Improved:** The macOS disk image now links to the release notes on the Knives on Strings
  website, which always cover the newest version, instead of carrying a copy.

## v0.15.29 -- 2026-10-07 -- New contact addresses

### Documentation

- **Improved:** Our contact addresses have moved: support@knivesonstrings.com for support,
  refunds and general questions, privacy@knivesonstrings.com for privacy, and
  legal@knivesonstrings.com for licensing.

## v0.15.28 -- 2026-10-07 -- Undo keeps your edits

### Patches & ROM

- **Fixed:** A project saved after an undo no longer throws your edits away when it
  reopens: your edited patch comes back, still marked as modified.
- **Fixed:** A project saved while you were building a sound from Init Voice now reopens
  with your sound, not the stored patch.

### UI

- **Fixed:** Undo and redo no longer rename the patch you are editing to INIT VOICE or
  reset its pitch envelope.

## v0.15.26 -- 2026-10-06 -- The modified indicator stays honest

### Patches & ROM

- **Fixed:** When a reopened project's patch no longer matches its bank slot, the modified
  indicator now stays on until you load, store or recall a patch, even if you
  close and reopen the window.
- **Fixed:** When your DAW restores XCent with its window open, for example a preset or
  snapshot recall, the display now switches to the restored patch.
- **Fixed:** Renaming a patch no longer marks your unsaved edits as saved, and the chorus
  switch, master tune and undo now mark the patch as modified in the project.

## v0.15.25 -- 2026-10-05 -- Licence agreement completed

### Hosts & formats

- **Improved:** The licence agreement now names the publisher, the governing law and the
  privacy policy, and states the refund policy: a full refund on request within
  30 days of purchase.
- **Improved:** It also points to the right place to opt out of anonymous usage data:
  Settings > Appearance > Privacy.

## v0.15.24 -- 2026-10-05 -- New audio options for the Windows Standalone

### Hosts & formats

- **New:** The Windows Standalone now plays through Windows Audio, with shared,
  exclusive and low-latency modes, or through DirectSound.
- **New:** ASIO is no longer included in the Standalone; if you used it, choose your
  audio interface under Windows Audio. The VST3 and CLAP plugins are unaffected.

## v0.15.21 -- 2026-09-16 -- Intel Macs and an honest manual

### UI

- **Improved:** The manual now describes correctly where XCent keeps your data, how the
  banks are laid out and how to install.
- **Improved:** The manual in every download is printed fresh for that exact release.
- **Fixed:** The manual's eight FM algorithm diagrams now appear, instead of broken images.

### Hosts & formats

- **New:** On macOS, every format now runs natively on Intel Macs as well as on Apple
  Silicon.
- **New:** macOS 15 Sequoia or later is now required, and the installer checks your Mac
  before it installs anything.
- **Improved:** The macOS installer always puts each format in its standard folder, even when
  another copy of XCent exists elsewhere on the Mac.
- **Fixed:** macOS: the plugins and the Standalone now load on every Mac, not only on the
  one they were built on.
- **Fixed:** The Linux installer now sends you to the right place for support.

## v0.15.20 -- 2026-09-16 -- The same look on every computer

### UI

- **Improved:** The screens XCent shows when its interface cannot start now use the same
  typeface as the interface.
- **Fixed:** On computers without its fonts installed, the interface no longer falls back
  to Courier New and borrowed symbols: every font now ships with XCent.
- **Fixed:** The About box now shows the version you actually installed; since 0.15.12 it
  always said 0.15.11.

### Hosts & formats

- **Improved:** The Windows installer and zip now include the licence agreement, the
  third-party notices and the full third-party licence texts.

## v0.15.19 -- 2026-09-08

### Maintenance

- Fixed an out-of-range read in the sound engine that only test code could trigger; the sound is unchanged.

## v0.15.18 -- 2026-09-07 -- Clean exits

### Patches & ROM

- **Fixed:** A project saved on a factory patch no longer shows that patch as modified
  when it reopens.

### Hosts & formats

- **Fixed:** Closing the Standalone, or quitting a DAW with XCent loaded, no longer ends in
  a crash report. Nothing was ever lost, but Windows logged an error.

## v0.15.17 -- 2026-09-07 -- Move your XCent folder

### Patches & ROM

- **New:** For portable or multi-profile setups, the folder where XCent keeps your
  patches, preferences and log can now be moved; the manual explains how.

## v0.15.16 -- 2026-09-06 -- CLAP hosts show the right values

### Hosts & formats

- **Fixed:** CLAP hosts now show the right parameter values after loading a project. The
  values were always restored, but the host could keep showing the old ones.

## v0.15.15 -- 2026-09-06 -- No false alarm on slow-opening windows

### Hosts & formats

- **Fixed:** In DAWs that create the plugin window well before showing it, a working
  interface is no longer replaced by an error panel after about 8 seconds.

## v0.15.14 -- 2026-09-06 -- No more white window

### Hosts & formats

- **Fixed:** Opening XCent in a VST3 host could show a blank white window instead of the
  interface, with no way out. XCent now recovers on its own as the window opens.

## v0.15.13 -- 2026-09-06

### Maintenance

- Built on JUCE 9.0.1; sound and saved projects are unchanged.

## v0.15.12 -- 2026-09-05 -- Loads in Cubase and Nuendo

### Hosts & formats

- **Fixed:** Windows: XCent now loads in Cubase, Nuendo and other hosts that use
  Steinberg's plugin loader, which refused it before.
- **Fixed:** If XCent's sound engine is ever missing from its folder, the plugin still
  loads, stays silent and explains what is wrong, instead of failing.

## v0.15.11 -- 2026-05-17 -- Report an issue from inside XCent

### UI

- **New:** A Report an Issue button in the About box opens a form for sending us a bug
  report.
- **New:** XCent sends anonymous usage data, tied to a random install ID and no personal
  data. The first-run tour asks, and Settings > Appearance > Privacy switches it
  off at any time.
- **Improved:** All your preferences are now stored together in XCent's folder in Documents.

### Hosts & formats

- **New:** The macOS disk image includes an uninstall script that removes all four
  formats and keeps your patches unless you ask it to remove them too.

## v0.15.1-rc2 -- 2026-05-14 -- Portamento like the hardware

### Sound engine

- **Improved:** Portamento now glides at the DX100's speed, where it was about half as fast,
  and as on the hardware it only engages in mono mode when you play legato.

## v0.15.0-rc2 -- 2026-05-12 -- Expression control and a Mac installer

### Sound engine

- **New:** XCent responds to MIDI CC11 (Expression), scaling the volume on top of CC7 so
  you can ride expression separately.

### Patches & ROM

- **Fixed:** When a project's patch has changed in its bank since you saved, XCent now
  shows it as modified, so you know the sound is not what the slot holds.

### UI

- **Fixed:** Issue reports sent from the Standalone are no longer rejected.

### Hosts & formats

- **New:** A macOS installer, on a disk image, installs the VST3, CLAP, Audio Unit and
  Standalone.
- **Fixed:** macOS: the Standalone no longer forgets settings made during a session when
  you close its window, and quitting no longer crashes.

## v0.14.0-rc1 -- 2026-05-09 -- A guided tour and tooltips

### UI

- **New:** A first-run tour walks you through modes, operators, editing, Modern view,
  Func mode and the patch browser. Replay it from Settings > Appearance.
- **New:** Hover over an Edit or Func button to see the parameter's full name and range.
- **New:** A setting opens Edit mode straight into Modern view, the new name for EZ mode.
- **Improved:** The Appearance settings are laid out in two columns, and the patch browser
  has clearer type and contrast.
- **Fixed:** Tooltips, right-click menus and the tag picker no longer drift off-screen at
  window scales other than 100%.
- **Fixed:** The modified indicator now changes as soon as you edit a parameter.

## v0.12.7 -- 2026-05-07 -- Breath control fixed

### Sound engine

- **Fixed:** Breath-controller patches now respond to CC2 the right way round, quieter at
  low breath as on the hardware, even for notes played before any breath data.

### Hosts & formats

- **Fixed:** Cubase 15 and Nuendo 15 no longer show a blank white window instead of the
  interface.

## v0.12.6 -- 2026-05-05 -- Better answers when the window stays blank

### Hosts & formats

- **Fixed:** If a Windows "optimisation" tool has removed parts of WebView2, XCent now
  offers to reinstall it instead of showing a white screen.
- **Fixed:** When a sandboxed host stops XCent's interface from saving its cache, XCent
  now says so and explains the workaround, instead of showing a white screen.

## v0.12.5 -- 2026-05-05 -- Recovery from a blank window

### UI

- **New:** If the interface fails to load, XCent shows a panel explaining what went
  wrong, with buttons to open the log folder and clear the interface cache.

### Hosts & formats

- **Improved:** The Windows installer clears the interface cache when it installs and
  uninstalls, so each version starts fresh. Your patches and settings are kept.

## v0.12.4 -- 2026-05-03 -- Idle means idle

### Sound engine

- **Fixed:** With a keyboard or clock sending MIDI, the CPU meter showed 30-50% while
  nothing played. XCent now rests at 0% when nothing is playing.

## v0.12.3 -- 2026-05-03 -- Undo that remembers

### UI

- **Improved:** The mouse wheel now changes parameters 50% faster.
- **Fixed:** Undo keeps its full 50-step history, and redo works across several steps.
- **Fixed:** The display no longer cuts off the names of patch-wide parameters such as
  pitch-bend range.

## v0.12.2 -- 2026-05-03 -- Wheels and velocity

### Sound engine

- **Improved:** Velocity sensitivity now follows the DX100's own response. Soft notes on
  velocity-sensitive patches were up to 10 dB too quiet.

### Patches & ROM

- **Fixed:** A voice received over MIDI SysEx shows its own name and comes back correctly
  when the project reopens.
- **Fixed:** Loading a patch or using undo no longer confuses the modified indicator.

### UI

- **New:** The on-screen pitch-bend and mod wheels respond to the mouse wheel, and pitch
  bend springs back to the centre on its own.
- **Fixed:** The meters stop using processor time once they settle at zero.

## v0.12.0 -- 2026-04-29 -- Pitch envelope done properly

### Sound engine

- **Improved:** The pitch envelope now behaves like the hardware's: DX21, TX81Z and DX11
  voices move through its stages at the hardware's rates, and DX100 and FB-01
  voices jump straight to the first level.
- **Improved:** Pitch envelope levels now span the hardware's full four octaves either way.

## v0.11.10 -- 2026-04-27 -- Closer to the hardware

### Sound engine

- **Improved:** XCent's output level now matches a real DX100; it was about 3 dB quieter.
- **Improved:** LFO vibrato and tremolo depth now scale like the hardware's, audibly
  improving patches such as Good Vibes, Helicopter and Windbells.

## v0.11.9 -- 2026-04-25 -- Mode letters on the display

### UI

- **New:** In Play mode the display shows a P before the patch name, as the DX100 does.
- **New:** A Modified Indicator setting chooses whether an edited patch is shown with a
  lowercase mode letter or an asterisk.
- **Fixed:** Patch-wide parameters in Edit mode, and the Func mode screen, now show the
  right mode letter.

## v0.11.8 -- 2026-04-21 -- Scroll to edit

### Patches & ROM

- **New:** Smarter automatic tagging recognises more leads, brass, organs, percussion
  and strings.
- **Fixed:** The factory banks are now read-only everywhere: Store is disabled on them,
  with a padlock and a tooltip saying why.
- **Fixed:** A 32-voice bank dump received over MIDI is now saved to bank A, and all of
  its names show in the browser.

### UI

- **New:** Scroll over an EZ mode knob or slider to change it: hold Ctrl or Cmd for big
  steps and Shift for fine ones, or switch it off in Settings > Appearance.

## v0.11.7 -- 2026-04-19 -- A Windows installer

### Hosts & formats

- **New:** A Windows installer puts the VST3, CLAP and Standalone in place, for all
  users or just you, and installs Microsoft's WebView2 runtime if it is missing.
- **New:** It adds Start Menu shortcuts and an uninstaller that leaves your patches and
  settings alone.

## v0.11.6 -- 2026-04-21 -- A clear message instead of a blank window

### Hosts & formats

- **Fixed:** On Windows without a recent WebView2 runtime, XCent now explains what is
  missing, with a button to install it, instead of showing a blank window.

## v0.11.5 -- 2026-04-20 -- Export to SysEx

### Patches & ROM

- **New:** Save a voice or a 32-voice bank as SysEx for the DX100, DX21 or TX81Z.
- **New:** XCent suggests the format that keeps the most of the voice, and lists exactly
  what any other format would drop.

## v0.11.4 -- 2026-04-20 -- Nothing lost in your library

### Patches & ROM

- **Improved:** Your patch library is now stored in XCent's own format, which keeps every
  detail of a voice. SysEx files are still imported as before.

## v0.11.3 -- 2026-04-20 -- More SysEx imports

### Patches & ROM

- **New:** Import TX81Z single voices and banks, with their extra waveform and
  fixed-frequency settings.
- **New:** DX11 and DX21 voices are recognised automatically on import.

## v0.11.2 -- 2026-04-20

### Maintenance

- Added a diagnostic log file for development builds; nothing changes in normal use.

## v0.11.1 -- 2026-04-20 -- More even levels

### Sound engine

- **Improved:** Factory patches now play at much more even levels across algorithms; a
  four-carrier patch could be about 12 dB louder than a one-carrier one.

### Patches & ROM

- **Fixed:** Factory tags now show for the patches in the writable banks.
- **Fixed:** A project saved on an untouched Init Voice reopens on its saved patch, and a
  fresh install always starts on the first factory patch.

## v0.11.0 -- 2026-04-20 -- DX21 chorus and FB-01 voices

### Sound engine

- **New:** DX21 chorus, modelled from hardware measurements, with a CHORUS button to
  switch it on for any patch.

### Patches & ROM

- **New:** FB-01 bank import, including each operator's attack velocity sensitivity.
- **New:** Factory patches are numbered as on the hardware: 101-124 in bank 100, 201-224
  in bank 200, and so on.

### UI

- **Improved:** Double-click the volume knob to reset it to 0 dB.

### Hosts & formats

- **Fixed:** On some Windows setups OneDrive could stop the interface from starting; its
  cache no longer lives in Documents.

## v0.10.0 -- 2026-04-19 -- The patch browser

### Patches & ROM

- **New:** A patch browser with search and tag filters, across all 192 factory voices and
  your own banks.
- **New:** Every factory voice comes tagged as bass, pad, keys, lead, brass, bell and
  more, and you can add tags of your own.

## v0.9.14 -- 2026-04-06 -- Bend and mod on screen

### UI

- **New:** On-screen pitch-bend and mod wheels beside the keyboard.

## v0.9.1 -- 2026-04-02 -- FB-01 velocity

### Sound engine

- **New:** FB-01 Velocity Mode gives 165 of the 192 factory voices per-operator velocity
  response taken from the Yamaha FB-01. Choose it or DX100 Classic in
  Settings > Sound.

## v0.9.0 -- 2026-04-01 -- DCent becomes XCent

### Sound engine

- **Fixed:** Fast arpeggios no longer click.

### Patches & ROM

- **Improved:** A fresh install starts on a factory patch instead of Init Voice.

### UI

- **New:** The plugin is now called XCent, with a new logo. Projects that used DCent
  need their instances replaced.

### Skins

- **Improved:** A new Stock LCD display style, and your display style now survives a restart.

## v0.8.4 -- 2026-03-29 -- Accessibility and idle CPU

### Sound engine

- **Improved:** With no notes playing, XCent drops from about 17% CPU to under 1%.

### UI

- **Improved:** Screen-reader labels, keyboard focus on knobs and sliders, and keyboard
  navigation inside dialogs.
- **Fixed:** Master Tune now shows its value on the display, and double-clicking a knob
  resets it to the patch's own value.

## v0.8.0 -- 2026-03-28 -- Light mode and full MIDI

### Sound engine

- **New:** A Vintage and Modern engine switch; Modern bypasses the analog output stage
  for a cleaner sound.
- **New:** MIDI control of portamento, volume and mono or poly mode, matching the DX100's
  MIDI chart.
- **Fixed:** LFO delay now matches the hardware; it was about 20 times too fast.

### UI

- **New:** Recall Edit and Init Voice buttons, and a MIDI receive channel setting.

### Skins

- **New:** Light mode, an OLED display style, tabbed settings, a window size setting and
  a CPU meter.

## v0.7.1 -- 2026-03-26 -- The right harmonics

### Sound engine

- **Fixed:** Operator frequency ratios were wrong for 53 of the 64 settings, affecting
  most factory voices. They now match the DX100.
- **Fixed:** A brighter output filter and corrected levels bring XCent within a fraction
  of a decibel of hardware recordings.

## v0.7.0 -- 2026-03-25 -- An accuracy pass

### Sound engine

- **Fixed:** Velocity response, keyboard level scaling and operator levels now follow the
  DX100's own curves.

### Patches & ROM

- **New:** Save the current voice or bank as a SysEx file.
- **Fixed:** All 192 factory voices had the wrong pitch and amplitude modulation
  sensitivity. Corrected.

### Hosts & formats

- **New:** Faster plugin scans: XCent skips its heavy start-up while a DAW is scanning.

## v0.6 -- 2026-03-21 -- First release

### Sound engine

- **New:** Four-operator FM synthesis on a cycle-accurate emulation of the DX100's
  Yamaha YM2164 chip.
- **New:** The DX100's analog output stage, modelled.

### Patches & ROM

- **New:** All 192 factory voices, across eight banks, with SysEx import.

### Hosts & formats

- **New:** VST3, CLAP and Standalone formats.
