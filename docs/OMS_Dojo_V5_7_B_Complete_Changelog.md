# ONEMANSHYO App [Beta V5_7_0] - COMPLETE CHANGELOG

**Version:** Beta V5_7_0  
**Date:** January 16, 2026  
**License:** GPL-3.0  
**Line Count:** 336 (V5_6_0) → 340 (V5_7_0)  
**Document Purpose:** Complete development history for V5_7_0 release

---

## V5_7_0 EXECUTIVE SUMMARY

Beta V5_7_0 delivers **MP4 native rendering, video editor-style transport controls, complete PIP system overhaul, and BumpR persistence fixes**. The Format dropdown enables direct MP4 export for universal iOS/social media playback. JKL transport brings familiar video editor navigation to Audio Focus mode. The PIP system now uses consistent P+Number chord hotkeys with mutual exclusivity. BumpR StickRs properly persist across page reloads with container association.

**Key Achievements:**
1. **MP4 Native Render** - Format dropdown with H.264+AAC codec for universal playback
2. **JKL Transport** - Video editor-style navigation (J=back, K=play/pause, L=forward)
3. **PIP System Overhaul** - P+1/2/3/4 chord hotkeys, mutual exclusivity, dropdown settings
4. **BumpR Persistence** - Folder handle restoration, container association, thumbnail fixes
5. **Render Improvements** - Cancel button, Loop ON/OFF respect, BumpR capture, portrait fix
6. **Audio Focus UX** - Bare A key entry, two-phase messages, waveform click entry
7. **loopLength Uniform** - Shader access to loop length for perfect infinite loops

---

## CORTEX CHANGES

### loopLength Uniform Added

New uniform at index 25 (replaces `padding1`) provides shader access to loop length.

| Index | Old Name | New Name | Type |
|-------|----------|----------|------|
| 25 | `padding1` | `loopLength` | f32 |

**Buffer size unchanged:** 112 bytes (28 floats × 4)

**Shader Usage:**
```wgsl
// Perfect infinite loop synced to UI dropdown
let cycle = fract(uniforms.beat / uniforms.loopLength);
```

**createUniformData() Updated (4 locations):**
| Location | Source |
|----------|--------|
| CREATE render() | shaderLoopLength field |
| Thumbnail preview | Static 16.0 |
| LIVE inset render | containerCS loopBeats |
| LIVE pop-out render | containerCS loopBeats |

---

## V5_7_0 MAJOR FEATURES

### 1. MP4 Native Render (V5_6_14)

✅ **Format Dropdown**
| Option | Codec | Use Case |
|--------|-------|----------|
| WebM | VP8 + Opus | Alpha support (not used in OMS) |
| MP4 | H.264 + AAC | Universal playback (iOS, social) |

✅ **Implementation**
- Resolution | FPS | Format row layout
- Persists to .omsp project state
- Fallback chain: H.264+AAC → H.264 only → WebM

### 2. JKL Transport (V5_6_16)

✅ **Video Editor Controls (Audio Focus)**
| Key | Single Tap | Double Tap (300ms) |
|-----|------------|---------------------|
| J | Back 1 beat | Back 1 bar (4 beats) |
| K | Play/Pause | — |
| L | Forward 1 beat | Forward 1 bar (4 beats) |

✅ **Additional Shortcuts**
| Key | Action |
|-----|--------|
| Cmd+L | Toggle loop ON/OFF |
| Waveform click | Enter Audio Focus mode |

### 3. PIP System Overhaul (V5_6_16b-16h)

✅ **Chord Hotkeys**
| Chord | PIP | Availability |
|-------|-----|--------------|
| P+1 | PIP1 slideout | PERFORM only |
| P+2 | PIP2 corner | ALL tabs |
| P+3 | PIP3 center | CREATE (render) |
| P+4 | Camera overlay | ALL tabs |

✅ **Mutual Exclusivity**
- Only one PIP visible at a time
- Turning on any PIP hides the others
- P2 persists across tab switches

✅ **Settings Panel Redesign**
- Dropdown-based (not toggle buttons)
- Orientation: Landscape (16:9) / Portrait (9:16) / Square (1:1)
- Size: 25% / 50% / 75% / 100% of canvas
- Shape: Rectangle / Circle (requires Square)

✅ **Shape-Orientation Logic**
- Shape dropdown disabled when orientation ≠ square
- Circle selection auto-snaps to square orientation
- Non-square selection resets shape to rectangle

### 4. BumpR Persistence (V5_6_11 series)

✅ **Fixes Applied**
| Issue | Fix |
|-------|-----|
| Slots empty after reload | Restore `bumprStickrsFolderHandle` from IndexedDB |
| Thumbnail generation fail | Generate BEFORE file write |
| Quantize crash | Move OmsDropdown init to DOMContentLoaded |

✅ **Container Association**
- `bumprSlotIndex` field added to .oms schema
- Tracks which BumpR slot was triggered during authoring
- Restores on container load

### 5. Render Improvements (V5_6_10, 12, 15, 16a)

✅ **Cancel Render**
- R button shows X during render
- Click X to cancel immediately
- Cleans up MediaRecorder, audio, mic connections

✅ **Loop ON/OFF Respect**
| Loop State | Render Duration |
|------------|-----------------|
| Loop ON | Loop In → Loop Out |
| Loop OFF | Audio Start → Audio End |

✅ **BumpR Capture**
- BumpR overlay now composited into render output
- Separate width/height ratios for correct aspect

✅ **Portrait Fix**
- Corrected dimension calculation for portrait orientation

✅ **MP4 Black Frame Fix**
- Render initial frame before starting MediaRecorder
- 50ms delay ensures H.264 keyframe exists

### 6. Audio Focus UX (V5_6_07-08)

✅ **Bare A Key Entry**
| Focus Mode | Entry Key |
|------------|-----------|
| Reactivity | R |
| Transform | T |
| Audio | A |
| Music Timing | M |

✅ **Two-Phase Message Pattern**
| Phase | Type | Content |
|-------|------|---------|
| Entry | info (cyan) | Shortcuts help |
| Value change | state (orange) | Value only |

✅ **Panel Navigation**
| Key | Panel |
|-----|-------|
| V | Volume (audio knob) |
| M | Mic knob |
| C | Cue points |
| E | EQ |
| F | FX |

---

## ITERATION DETAILS

### Audio Filter Sync (V5_6_01a)

- Freq1/Freq2 now sync to bandpass filters in real-time
- Range extended: 1Hz - 20kHz (was 20Hz - 24kHz)
- Q values from globals (was hardcoded 0.7)

### UI Fixes (V5_6_02a-04)

- Swap dropdown: solid background, z-index 100
- Time separator: removed (was causing bleed-through)
- N-panel: Blender-style close (only N key closes it)
- Swap dropdown flag prevents render loop override

### New Container Button (V5_6_05a)

- "+" button in SHADER CONTROLS header
- Resets all controls to defaults
- Clears shader code and canvas

### Orientation Labels (V5_6_09a)

- Landscape (16:9) / Portrait (9:16) / Square (1:1)
- Clarifies aspect ratio for shader development

### Loop Out Fix (V5_6_16)

- Arrow keys maintain .1 positions throughout
- Changed comparison from `<=` to `<`

---

## NEW STATE VARIABLES

```javascript
let createAudioSwapDropdownOpen = false;  // Prevent render loop override
let pipPKeyHeld = false;                   // P key chord detection
let renderCancelled = false;               // Track cancelled renders
let activeMediaRecorder = null;            // Module-scoped for cancel
let containerBumprSlotIndex = null;        // BumpR container association
let lastJTapTime = 0;                       // J double-tap detection
let lastLTapTime = 0;                       // L double-tap detection
const JKL_DOUBLE_TAP_MS = 300;             // Double-tap threshold
let pip3Orientation = 'square';            // Renamed from pip3Aspect
let pip3SizePercent = '50';                // Renamed from pip3Size
```

---

## FILE FORMAT CHANGES

### Container (.oms) - 1 New Field
```javascript
{
    bumprSlotIndex: 2,  // 0-9 or null (NEW V5_7_0)
}
```

### Project State (.omsp) - 1 New Field
```javascript
{
    renderFormat: 'webm',  // 'webm' or 'mp4' (NEW V5_7_0)
}
```

### Uniform Buffer - 1 Field Repurposed
```javascript
// Index 25: padding1 → loopLength
```

---

## NEW FUNCTIONS

| Function | Purpose |
|----------|---------|
| `newContainer()` | Reset to empty container |
| `handleRenderClick()` | Route start/cancel render |
| `cancelRender()` | Stop active render, cleanup |
| `jumpAudioByBeats(beats)` | Jump audio position |
| `hidePip1()` | Hide P1 without toggle |
| `hidePip2()` | Hide P2 without toggle |
| `hidePip3()` | Hide P3 without toggle |
| `setPip3Orientation(value)` | Set orientation with shape reset |
| `setPip3Size(value)` | Set size percentage |
| `setPip3Shape(value)` | Set shape with orientation snap |
| `compositeOverlaysToCanvas()` | Composite PIP3+BumpR to render |

---

## BUG FIXES

| Build | Issue | Fix |
|-------|-------|-----|
| V5_6_01a | Freq filter not syncing | Add updateBandpassFilters() calls |
| V5_6_02a | Swap dropdown transparent | Solid #1a1a1a background |
| V5_6_04 | Time display bleed-through | createAudioSwapDropdownOpen flag |
| V5_6_11 | BumpR slots empty after reload | Restore folder handle from IndexedDB |
| V5_6_11a | Thumbnail generation fail | Generate before file write |
| V5_6_11f | Quantize dropdown crash | Move init to DOMContentLoaded |
| V5_6_12 | Portrait dimensions wrong | Correct width/height swap |
| V5_6_15 | BumpR not in render | Composite with separate ratios |
| V5_6_16 | Loop Out arrows break .1 | Change <= to < comparison |
| V5_6_16a | MP4 black frames at start | Pre-render frame + 50ms delay |
| V5_6_16h | P2 hiding on tab switch | Remove hide code from tab handler |

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Total Iterations | 16 major (01a-16h) |
| Date Range | January 12-15, 2026 |
| Lines Added | ~560 |
| New Functions | 11 |
| New State Variables | 10 |
| Bug Fixes | 11 |
| Schema Additions | 1 (.oms) + 1 (.omsp) |
| CORTEX Changes | 1 (loopLength uniform) |

---

## COMPATIBILITY

✅ Backward compatible (no breaking changes)
✅ Existing .oms files load correctly
✅ Existing .omsp projects load correctly
✅ New fields have sensible defaults
✅ loopLength defaults to 16.0 (4 bars)

---

## KNOWN ISSUES

1. **Orientation dropdown sync** - Visual dropdown may not match loaded orientation value (inherited from V5_1_0)
2. **Unimplemented pool effects** - Blur, Sharpen, Invert (VFX) and Reverb, Phaser, etc. (FX) show in dropdowns but have no effect (inherited from V5_6_0)

---

## PRODUCTION READINESS: ~98%

**100% Complete (V5_7_0):**
- MP4 Native Render
- JKL Transport Controls
- PIP System Overhaul
- BumpR Persistence
- Render Improvements
- Audio Focus UX
- loopLength Uniform

**Inherited from V5_6_0 (100%):**
- Per-Track VFX System
- Unified Mic System
- Transform Keyboard
- Audio FX/VFX Pools

**Remaining:**
- Pool effects completion (7 FX, 3 VFX)
- MIDI integration
- Orientation dropdown sync

---

**END OF COMPLETE CHANGELOG**
