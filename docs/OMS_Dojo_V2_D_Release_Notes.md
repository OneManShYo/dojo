# ONEMANSHYO Dojo App V2 — Release Notes

**Release Date:** November 4, 2025  
**Version:** V2  
**License:** GPL-3.0  
**Status:** Foundation release — tempo-synced modulation

---

## Executive Summary

Dojo V2 is a browser-native tool for authoring audio-visual shader content. It
compiles WGSL fragment shaders, drives them with three tempo-synced modulation
sources, renders live via WebGPU, and exports to WebM.

V2 is the culmination of the V1 development series (V1.0.0 → V1.0.5). The defining
change across that series was the move from time-based animation to musical time:
early builds animated on elapsed seconds, V2 animates on beats.

**What's included:**
- WebGPU shader compilation and live preview
- Tempo control (60–200 BPM)
- Three independent modulation sources with 10 waveform generators each
- Five subdivisions per mod (Whole through 1/16)
- Resolution, frame rate, and orientation control
- WebM export with self-describing filenames

**What's NOT included:**
- No audio input or audio-reactive modulation
- No preset system
- No shader library or file management
- No transport controls (scrub, loop, pause)
- No MIDI control

---

## Release Contents

**Application**  
`OMS_Dojo_V2.html` — ~35KB, single file, no dependencies

**Documentation**
1. `OMS_Dojo_V2_B_Complete_Changelog.md` — V1 series development history
2. `OMS_Dojo_V2_C_Developer_Documentation.md` — Technical reference
3. `OMS_Dojo_V2_D_Release_Notes.md` — This file
4. `OMS_Dojo_V2_E_UserGuide_All.md` — Complete user guide
5. `OMS_Dojo_V2_Z_ClaudeProject_Instructions.md` — AI development context

---

## What Works in V2

**Shader Authoring**
- Paste WGSL into the editor, compile, see it immediately
- Uniform struct prepended automatically — authors write only `fs_main`
- Compile errors surface with the WGSL compiler's own message

**Tempo Sync**
- Tempo exposed to shaders as a uniform
- Beat position derivable in-shader: `time × (tempo / 60)`
- Mod values computed from beat position, not wall-clock time

**Modulation**
- Mod A, B, C independently configurable
- Subdivision and waveform per mod
- Values normalized 0.0–1.0, ready to drive any shader parameter
- Defaults ship as 1/4 Sine, 1/8 Sawtooth, 1/16 Square — three rates producing
  layered motion rather than uniform pulsing

**Export**
- Real-time canvas capture to WebM (VP8, 8 Mbps)
- 720p / 1080p, 24 / 30 / 60 FPS, landscape or portrait
- Duration 1–600 seconds
- Filenames encode resolution, frame rate, duration, and tempo

---

## Known Limitations

Stated plainly — these are the honest boundaries of this build, not defects.

**No audio.** Tempo is set by hand. The app has no awareness of what's playing, and
modulation is derived from the clock rather than from sound. Pairing output with
music requires matching the BPM manually.

**Nothing persists.** Mod settings, tempo, and shader code are lost on reload.
There is no save, no preset, no library.

**Export is real-time.** A 60-second render takes 60 seconds. No offline path.

**Chaos ignores subdivision.** The Chaos generator derives from beat position
directly and doesn't use the subdivision setting. Changing it has no effect.

**Memory grows during long exports.** Recorded chunks accumulate before the blob is
assembled — a 10-minute 1080p capture holds roughly 600MB.

**One shader at a time.** No library, no switching, no comparison.

**WebGPU required.** Firefox was unsupported at build time.

---

## System Requirements

- Chrome 113+ on macOS (primary target)
- WebGPU enabled
- GPU capable of running the authored shader at the chosen resolution

---

## Upgrade Notes

V2 is published from V1.0.5 with no functional changes. Applied at publication:
GPL-3.0 license header, ONEMANSHYO branding, standardized title and version badge.

Shaders written for V1.0.5 run unchanged in V2. The uniform contract is identical.

---

## Looking Ahead

The V2 series (V2.0.1 → V2.9.0) took on audio directly — waveform display, FFT
analysis, genuinely audio-reactive modulation, In/Out bar regions, and the container
format that became `.oms`.

---

**End of Release Notes**
