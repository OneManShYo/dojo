# ONEMANSHYO App [Beta V5_7_0] - RELEASE NOTES

**Version:** Beta V5_7_0  
**Date:** January 16, 2026  
**License:** GPL-3.0  
**Line Count:** 342 (V5_6_0) → 292 (V5_7_0)  
**Document Purpose:** User-facing release summary for V5_7_0

---

## WHAT'S NEW

### 🎬 MP4 Native Render (Major!)

**Export directly to MP4 for universal playback!**

No more WebM-only output. Choose your format from a new dropdown in Container Output.

| Format | Codec | Best For |
|--------|-------|----------|
| WebM | VP8 + Opus | Web embedding |
| MP4 | H.264 + AAC | iOS, social media, universal |

**Setup:**
1. Go to Container Output section
2. Select Resolution, FPS, and Format
3. Press R to render
4. Output plays everywhere!

---

### ⏯️ JKL Transport Controls (Major!)

**Video editor-style navigation in Audio Focus!**

Navigate audio with the same shortcuts used in Premiere, DaVinci, and other editors.

| Key | Single Tap | Double Tap |
|-----|------------|------------|
| J | Back 1 beat | Back 1 bar |
| K | Play/Pause | — |
| L | Forward 1 beat | Forward 1 bar |

**Additional Shortcuts:**
| Key | Action |
|-----|--------|
| Cmd+L | Toggle loop ON/OFF |
| Click waveform | Enter Audio Focus |

---

### 📹 PIP System Overhaul (Major!)

**Unified hotkeys and mutual exclusivity!**

All PIPs now use consistent P+Number chord pattern. Only one PIP visible at a time.

| Chord | PIP | Description |
|-------|-----|-------------|
| P+1 | PIP1 | Slideout (PERFORM only) |
| P+2 | PIP2 | Corner (all tabs) |
| P+3 | PIP3 | Center (CREATE render) |
| P+4 | Camera | Full overlay (all tabs) |

**Settings Panel Redesign:**
- Clean dropdown style (not chunky toggles)
- Orientation: Landscape (16:9) / Portrait (9:16) / Square (1:1)
- Size: 25% / 50% / 75% / 100% of canvas
- Shape: Rectangle / Circle

**Smart Logic:**
- Shape dropdown disabled when not square
- Selecting circle auto-snaps to square
- P2 persists across tab switches

---

### 🎯 BumpR Persistence (Major!)

**Slots actually stay after page reload!**

BumpR StickRs now properly persist across browser sessions.

**Fixes:**
- Folder handle restored from IndexedDB
- Thumbnails generate before file write
- Quantize dropdown init moved to startup

**Container Association:**
- BumpR slot saved with container
- Trigger a slot while authoring → saved
- Load container → slot pre-selected

---

### ⏹️ Render Improvements

**Cancel, Loop respect, BumpR capture, and more!**

| Feature | Description |
|---------|-------------|
| Cancel | R button shows X during render, click to cancel |
| Loop ON | Renders Loop In → Loop Out |
| Loop OFF | Renders Audio Start → Audio End |
| BumpR | Now captured in render output |
| Portrait | Dimensions now correct |
| MP4 Start | No black frames at beginning |

---

### 🎹 Audio Focus UX

**Bare A key and two-phase messages!**

| Focus Mode | Entry Key |
|------------|-----------|
| Reactivity | R |
| Transform | T |
| Audio | A (was Shift+A) |
| Music Timing | M (was Shift+M) |

**Panel Navigation:**
| Key | Panel |
|-----|-------|
| V | Volume knob |
| M | Mic knob |
| C | Cue points |
| E | EQ |
| F | FX |

**Two-Phase Messages:**
- Entry: Cyan help text with shortcuts
- Value change: Orange value only (no shortcuts)

---

### 🔄 loopLength Shader Uniform

**Perfect infinite loops synced to UI!**

New `uniforms.loopLength` provides shader access to the Loop Length dropdown value.

```wgsl
// Cycle exactly once per loop
let cycle = fract(uniforms.beat / uniforms.loopLength);
```

| Dropdown | Beats |
|----------|-------|
| 1/4 | 0.25 |
| 1 Bar | 4 |
| 4 Bars | 16 (default) |
| 16 Bars | 64 |

---

### 🔧 Other Improvements

**New Container Button:**
- "+" button in SHADER CONTROLS header
- Resets everything to defaults
- Clears shader code and canvas

**Orientation Labels:**
- Now show aspect ratios: Landscape (16:9), Portrait (9:16), Square (1:1)

**Audio Filter Sync:**
- Freq1/Freq2 knobs now sync to filters in real-time
- Range extended: 1Hz - 20kHz

**N-Panel Behavior:**
- Blender-style: only N key closes it
- Stays open when exiting Reactivity Focus
- Closes on tab switch

**Loop Out Fix:**
- Arrow keys maintain .1 positions throughout

---

## KEYBOARD QUICK REFERENCE

### Key Remapping ⚠️

| Function | Before | After |
|----------|--------|-------|
| Audio Focus | Shift+A | **A** (bare) |
| Music Timing | Shift+M | **M** (bare) |
| PIP toggles | O, P, Shift+P | **P+1, P+2, P+3, P+4** |

### CREATE Tab - Audio Focus

| Key | Action |
|-----|--------|
| A | Enter/Exit Audio Focus |
| V | Volume panel |
| M | Mic panel |
| C | Cue panel |
| E | EQ panel |
| F | FX panel |
| J | Back 1 beat (JJ = bar) |
| K | Play/Pause |
| L | Forward 1 beat (LL = bar) |
| Cmd+L | Toggle loop |
| ←→ | Adjust value |
| Enter | Kill/mute toggle |

### All Tabs - PIP

| Chord | Action |
|-------|--------|
| P+1 | PIP1 slideout (PERFORM) |
| P+2 | PIP2 corner |
| P+3 | PIP3 center (CREATE) |
| P+4 | Camera overlay |

---

## CORTEX CHANGES

### loopLength Uniform Added

| Index | Old | New |
|-------|-----|-----|
| 25 | padding1 | loopLength |

Buffer size unchanged (112 bytes). Defaults to 16.0 (4 bars).

---

## BUG FIXES

- ✅ Freq knobs now sync to bandpass filters in real-time
- ✅ Swap dropdown has solid background (no bleed-through)
- ✅ Time display hidden when swap dropdown open
- ✅ BumpR slots persist after page reload
- ✅ BumpR thumbnails generate correctly
- ✅ Quantize dropdown doesn't crash on load
- ✅ Portrait render dimensions correct
- ✅ BumpR appears in render output at correct size
- ✅ Loop Out arrow keys maintain .1 positions
- ✅ MP4 renders have no black frames at start
- ✅ P2 persists across tab switches

---

## COMPATIBILITY

✅ Fully backward compatible  
✅ All .oms containers work unchanged  
✅ All .omsp projects work (new fields use defaults)  
✅ loopLength defaults to 16.0 beats  
✅ renderFormat defaults to 'webm'  
✅ bumprSlotIndex defaults to null

---

## INHERITED FEATURES

All features from previous versions remain:

**V5_6_0:** Per-Track VFX, RGB Control, Effect Pools, Unified Mic, Transform Keyboard  
**V5_5_0:** Transform Animation, Per-Mod ADSR+T, PIP Camera, ORGANIZE UX, FX Overlay  
**V5_4_0:** Audio Card & Cue Bus, MAIN Column, DSP, VFX Post-Processing, Scene Labels  
**V5_3_0:** Real Audio Reactivity, Sets System, Tags, Peak Metering, Quantize  
**V5_2_0:** Audio Focus Mode, EQ3, DJ FX, 5 Cue Points  
**V5_1_0:** BumpR StickRs, Shader Tempo Link  
**V5_0_0:** Transform Controls, 112-byte Uniform Struct

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Development Period | January 12-15, 2026 |
| Iterations | 16 (01a-16h) |
| HTML Lines | ~39,667 |
| New Functions | 11 |
| Bug Fixes | 11 |

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**
