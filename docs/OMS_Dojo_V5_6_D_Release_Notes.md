# ONEMANSHYO App [Beta V5_6_0] - RELEASE NOTES

**Version:** Beta V5_6_0  
**Date:** January 11, 2026  
**License:** GPL-3.0  
**Line Count:** 492 (V5_5_0) → 342 (V5_6_0)  
**Document Purpose:** User-facing release summary for V5_6_0

---

## WHAT'S NEW

### 🎨 Per-Track Visual Effects (Major!)

**Independent VFX processing for each track!**

Every track now has its own visual effects pipeline. Apply different effects to each layer before they composite together.

| Track | VFX Slot 1 | VFX Slot 2 | VFX Slot 3 |
|-------|------------|------------|------------|
| 1-4 | Assignable | Assignable | Assignable |

**Processing Order:**
```
Track Shader → Track VFX → RGB Color → Composite → MAIN VFX → Output
```

**Available Effects (7 implemented):**
| Effect | Abbrev | Description |
|--------|--------|-------------|
| Off | OFF | No effect |
| RGB Split | RGB | Chromatic aberration |
| Glitch | GLI | Digital glitch |
| Pixelate | PIX | Pixelation |
| Scanlines | SCN | CRT scanlines |
| Mirror | MIR | Mirror reflection |
| Kaleidoscope | KAL | Kaleidoscope |

**Controls:** 3 purple knobs per track in FX overlay panel

---

### 🌈 Per-Track RGB Control (Major!)

**Real-time color channel manipulation!**

Adjust red, green, and blue channels independently for each track. Create color shifts, isolate channels, or completely remove colors.

| Knob Value | Result |
|------------|--------|
| 0 | Channel removed (black) |
| 0.75 | Unity (no change) |
| 1.0 | Boosted (+33%) |
| Click | Kill toggle (instant black) |

**UI:** 3 purple R/G/B knobs (20×20) per track, between VFX and EQ columns

---

### 🎚️ Effect Pool Systems (Major!)

**Centralized effect management with per-track assignment!**

Configure which effects appear in each track's slots via ADMIN tab.

**Visual VFX Pool (10 effects):**
```
OFF, RGB Split, Glitch, Pixelate, Scanlines, Mirror, Kaleidoscope, Blur*, Sharpen*, Invert*
(* coming soon)
```

**Audio FX Pool (10 effects):**
```
Washout, Dubby, Break, Reverb*, Phaser*, Flanger*, BitCrush*, LPF*, Gate*, Ring Mod*
(* coming soon)
```

**ADMIN Configuration:**
- Navigate to ADMIN → Audio FX or Visual VFX
- 4 track rows × 3 slot dropdowns
- Changes saved to project state

---

### 🎤 Unified Mic System (Major!)

**Voice-reactive shaders without loading an audio file!**

Your microphone can now drive shader reactivity directly. Perfect for voice-controlled visuals or when you don't have a track loaded.

**Features:**
- Mic feeds same reactivity chain as file audio
- Works alone OR combined with audio file
- Freq knobs enabled when mic is unmuted
- Volume knob in CREATE header (orange, tap to mute)
- Level meter shows input activity

**Setup:**
1. Go to ADMIN → Video Sources
2. Select mic device from dropdown
3. Unmute mic knob in CREATE header
4. Shader responds to your voice!

---

### ⌨️ Transform Keyboard Redesign (Major!)

**Direct control selection for live performance!**

Transform mode completely redesigned for faster access during performance.

| Key | Action |
|-----|--------|
| T | Enter/Exit Transform mode (was Shift+T) |
| 1 | Select Rotation |
| 2 | Select Scale |
| 3 | Select Tile |
| 4 | Select Translate |
| 5 | Select Pivot |
| ↑↓ | Cycle mode (Stop→CW→CCW→PingPong) |
| ←→ | Adjust speed or position |
| Enter | Freeze/Resume animation |
| 2×Enter | Reset to default |

**Freeze/Resume:**
- Press Enter to pause any animation instantly
- Press Enter again to resume from where you left off
- Double-tap Enter (within 300ms) to reset

---

### 📹 PIP Camera Enhancements

**More control over picture-in-picture!**

**Three-Dropdown Settings:**
| Control | Options |
|---------|---------|
| Aspect | 16:9, 9:16, 1:1 |
| Size | Small, Medium, Large |
| Shape | Sharp, Rounded, Circle |

**PIP3 Settings Panel:**
- Click on PIP3 to open settings popup
- Aspect, Size, Shape toggles
- 9 color swatches
- Click outside to close

**Render Composite:**
- PIP3 now appears in rendered output
- WYSIWYG sizing (matches preview exactly)
- Great for recording tutorials or commentary

---

### 🎛️ Audio Control Improvements

**CREATE Tab Header:**
```
AUDIO | [volume knob] [meter] | [mic knob] [meter] | filename...
```

| Interaction | Volume Knob | Mic Knob |
|-------------|-------------|----------|
| Drag | Adjust (1dB steps) | Adjust (1dB steps) |
| Shift+Drag | Fine adjustment | Fine adjustment |
| Tap | Toggle mute | Toggle mute |

**N-Panel Mute/Solo:**
- M/S buttons in Reactivity N-panel header
- Quickly isolate frequency bands for tuning
- Keys: `,` = mute, `.` = solo

**Stepped Frequency Presets:**
```
0, 5, 10, 20, 30, 50, 75, 100, 150, 200, 300, 500, 1000, 2000, 5000, 10000, 20000 Hz
```
Arrow keys snap to nearest preset, mouse drag remains continuous.

---

### 🔧 Other Improvements

**EQ/FX Knob Controls (PERFORM):**
- Drag up/down to adjust
- Scroll wheel to adjust
- Click to toggle kill
- Cyan hover ring

**Panel Toggle Buttons:**
- EQ button: Purple/cyan split cube
- FX button: Purple/cyan split cube
- Keys: E = EQ overlay, F = FX overlay

**Shader Render Duration:**
- Now in bar.beat format (was MM:SS.mmm)
- Default: "17.1" (64 beats)
- Arrow keys: ↑↓ = ±beat, ←→ = ±bar

**ADMIN Navigation:**
- Alphabetized: Application, BumpR StickRs, Create, Organize, Perform, Video Sources

---

## KEYBOARD QUICK REFERENCE

### Key Remapping ⚠️

| Function | Before | After |
|----------|--------|-------|
| Reactivity mode | Shift+R | **R** (bare) |
| Render trigger | R | **Shift+R** |
| Transform mode | Shift+T | **T** (bare) |

### CREATE Tab

| Key | Action |
|-----|--------|
| R | Enter Reactivity Focus |
| T | Enter Transform Focus |
| 1-5 | Select transform control |
| ↑↓ | Cycle mode (in Transform) |
| Enter | Freeze/Resume animation |
| N | Toggle Mod Tuning panel |
| `,` | Mute current mod (N-panel) |
| `.` | Solo current mod (N-panel) |
| Shift+R | Start render |

### PERFORM Tab

| Key | Action |
|-----|--------|
| E | Toggle EQ & Meters overlay |
| F | Toggle FX overlay |
| V | Open VFX panel (requires track) |
| A | Open AFX panel (requires track) |

---

## BREAKING CHANGES

### Shader Uniform Rename ⚠️

**Affects existing containers with custom shaders!**

| Old Name | New Name |
|----------|----------|
| `shaderModAFFT` | `shaderModAAudio` |
| `shaderModBFFT` | `shaderModBAudio` |
| `shaderModCFFT` | `shaderModCAudio` |

**Why:** Names now match implementation (bandpass peak detection, not FFT).

**Migration Script:**
```bash
for f in *.oms; do
  sed -i '' 's/shaderModAFFT/shaderModAAudio/g; s/shaderModBFFT/shaderModBAudio/g; s/shaderModCFFT/shaderModCAudio/g' "$f"
done
```

### VFX Params Buffer

- Size increased: 16 → 32 bytes
- Added RGB multiplier fields
- Existing shaders continue to work

---

## BUG FIXES

- ✅ RGB control now affects track output (was no-op)
- ✅ FX values correctly use 0-1 range (was 0-100)
- ✅ WebM renders include audio when using mic
- ✅ Mic meter starts reliably in ADMIN
- ✅ ModTuning parameters all save to container
- ✅ PIP3 render size matches preview exactly
- ✅ Audio files copy to Library on save

---

## REMOVED

**Frequency Signal LEDs:**
- 6 LED indicators removed from Reactivity section
- Were showing raw FFT, not actual mod values
- N-panel meters provide accurate feedback

**Legacy FFT Fallback:**
- ~60 lines of fallback code removed
- Bandpass system is robust, fallback never triggered

---

## COMPATIBILITY

✅ Containers with old uniform names need shader update (see migration)  
✅ All other .oms containers work unchanged  
✅ All .omsp projects work (new fields use defaults)  
✅ Pool assignments default to standard configuration  
✅ Mic settings default to system mic, muted

---

## INHERITED FEATURES

All features from previous versions remain:

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
| Development Period | January 3-11, 2026 |
| Iterations | 15 |
| Sub-iterations | ~70 |
| HTML Lines | ~39,107 |
| VFX Effects | 7 implemented, 3 coming |
| Audio FX | 3 implemented, 7 coming |

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**

Version: Beta V5_6_0  
Date: January 11, 2026  
License: GPL-3.0

*"Every track. Every effect. Every color. Under your control."*
