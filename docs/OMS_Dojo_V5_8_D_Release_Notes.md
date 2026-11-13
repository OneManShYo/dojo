# ONEMANSHYO App [Beta V5_8_0] - RELEASE NOTES

**Version:** Beta V5_8_0  
**Date:** January 22, 2026  
**License:** GPL-3.0  
**Line Count:** ~43,800 (V5_7_0 ~39,667)  
**Document Purpose:** User-facing release summary for V5_8_0

---

## WHAT'S NEW

### 🎛️ Transform Automation (Major!)

**Visual programming for transform parameters!**

Draw automation curves directly on the waveform. Control rotation, scale, tile, position, and pivot over time with a professional graph editor.

**Getting Started:**
1. Press **G** to open the Automation Overlay
2. Press **1-5** to select a control
3. **Click** canvas to add nodes
4. **Drag** nodes to position them
5. **Right-click** segments to change curve type

| Key | Control | Color |
|-----|---------|-------|
| 1 | Rotation | Red |
| 2 | Scale | Teal |
| 3 | Tile | Yellow |
| 4 | Position | Mint |
| 5 | Pivot | Plum |

**Curve Types:**
| Type | Effect |
|------|--------|
| Linear | Straight interpolation |
| Sine | S-curve ease in/out |
| EaseIn | Slow start, fast end |
| EaseOut | Fast start, slow end |
| Hold | Step function (instant jump) |

**Mouse Controls:**
- Click canvas → Add node
- Click node → Delete node
- Drag node → Move freely
- Shift+drag → Snap to beat/0.1
- Right-click segment → Curve menu

---

### 🎬 BumpR Overhaul (Major!)

**Nested panels, video transport, and sequential firing!**

BumpR StickRs gets a complete UX upgrade with professional workflow features.

**B+N Panel System:**
- **B** opens slot panel (right slideout)
- **N** opens settings panel (left of B)
- Both panels persist independently
- Works in CREATE and PERFORM tabs

**JKL Video Transport:**
| Key | Single | Double |
|-----|--------|--------|
| J | Back 1s | Back 5s |
| K | Play/Pause | Swap z-index |
| L | Forward 1s | Forward 5s |

**A/D Sequential Firing:**
| Key | Action |
|-----|--------|
| A | Previous slot + fire (wraps 10→1) |
| D | Next slot + fire (wraps 1→10) |

**Drag-Drop Reordering:**
- Drag slots to rearrange order
- All settings (timing, scale, animate, duration) swap together

**Z-Index Control:**
- K×2 toggles BumpR above/below PIPs
- Render respects current z-index state

---

### 🎧 Audio Input Routing (Major!)

**Drive modulators from external audio!**

Route audio from Ableton (via Loopback) directly to shader modulators.

**Setup:**
1. Install Loopback (or similar virtual audio device)
2. Route DAW output to virtual device
3. ADMIN → Audio Card → Scan Devices
4. Select your device in INPUT ROUTING dropdown

**Signal Flow:**
```
Ableton → Loopback → OMS Input → Modulators
```

**CREATE Header Controls:**
- Green 24×24 input knob
- **I** key in Audio Focus
- Tap to mute, drag to adjust
- 4×24 vertical level meter

**Status Indicator:**
- "✓ Input active → Modulators" when connected

---

### 🟢 Chroma Key (Major!)

**WebGPU green screen removal for PIP3!**

Remove green screen backgrounds from your camera feed in real-time.

**Setup:**
1. ADMIN → PIP3 Settings
2. Enable "Chroma Key"
3. Adjust parameters to match your green screen

| Parameter | Range | Default |
|-----------|-------|---------|
| Tolerance | 0-1 | 0.3 |
| Softness | 0-1 | 0.1 |
| Spill | 0-1 | 0.2 |

**Technical Notes:**
- Uses bgra8unorm format (matches main pipeline)
- Processed every frame
- Captured correctly in render output

---

### 🎚️ PERFORM Tab Overhaul (Major!)

**Unified rotary control and keyboard mixing!**

Complete redesign of track/MAIN control with consistent interactions.

**Track Selection:**
| Key | Selection |
|-----|-----------|
| 1-4 | Select Track 1-4 |
| 5 | Select MAIN |
| 0 | Deselect |

**Arrow Key Control:**
| Keys | Audio (default) | Visual (Cmd+) |
|------|-----------------|---------------|
| ←/→ | Volume ±5% | Opacity ±5% |
| ↑/↓ | Volume ±1% | Opacity ±1% |

**Unified Rotary Behavior:**
| Action | Result |
|--------|--------|
| Hover | Help in message bar |
| Drag | Coarse adjustment |
| Shift+drag | Fine adjustment |
| Click | Keyboard mode (cyan ring) |
| Double-click | Toggle kill (magenta fill) |
| Enter×2 | Toggle kill |

**Stop All Safety:**
- Click once → "Press again to stop all"
- Click twice within 500ms → Execute
- ESC×1 primes, ESC×2 executes

**MAIN Header Features:**
- Hold 500ms → Inline label edit
- Right-click → Color picker

---

### ⏱️ OMS Timing Constants

**Consistent interaction timing across the app!**

New CORTEX-tier constants ensure uniform UX:

| Constant | Value | Usage |
|----------|-------|-------|
| OMS_HOLD_TO_EDIT_MS | 500ms | Scene/MAIN label edit |
| OMS_DOUBLE_TAP_MS | 300ms | Kill toggle, JKL double-tap |
| OMS_CONFIRM_TIMEOUT_MS | 500ms | Stop All, ESC×2 |

---

### 🔧 Other Improvements

**Waveform Grid Alignment:**
- Dynamic playheadX calculation (was hardcoded)
- Correct BPM fallback (120, not 128)
- Non-playing tracks now display correctly

**Cue Point Fixes:**
- Cue 5 displays at correct position
- Jump works for all 5 cues
- Visual indicators render properly

**Render Improvements:**
- Chroma key captured correctly
- RAF keeps running (no pause during render)
- Stop-and-save behavior (X saves partial, not discard)

**Audio Card Updates:**
- Cue Output "None" option added
- Input device dropdown in ADMIN

---

## KEYBOARD QUICK REFERENCE

### Key Additions ⚠️

| Function | Key |
|----------|-----|
| Automation Overlay | **G** |
| Automation Controls | **1-5** (in G mode) |
| BumpR Settings Panel | **N** (in B mode) |
| BumpR Previous+Fire | **A** (in B mode) |
| BumpR Next+Fire | **D** (in B mode) |
| Input Panel | **I** (in Audio Focus) |
| Track Selection | **1-4** (PERFORM) |
| MAIN Selection | **5** (PERFORM) |
| Deselect | **0** (PERFORM) |

### CREATE Tab - Automation

| Key | Action |
|-----|--------|
| G | Toggle Automation Overlay |
| 1 | Select Rotation |
| 2 | Select Scale |
| 3 | Select Tile |
| 4 | Select Position |
| 5 | Select Pivot |
| Esc | Close overlay |

### CREATE Tab - BumpR Mode

| Key | Action |
|-----|--------|
| B | Enter/Exit BumpR mode |
| 1-0 | Trigger slots 1-10 |
| ↑/↓ | Navigate slots |
| N | Toggle settings panel |
| A | Previous + fire |
| D | Next + fire |
| J/K/L | Video transport |
| K×2 | Swap z-index |

### PERFORM Tab - Mixing

| Key | Action |
|-----|--------|
| 1-4 | Select track |
| 5 | Select MAIN |
| 0 | Deselect |
| ←/→ | Volume ±5% |
| ↑/↓ | Volume ±1% |
| Cmd+←/→ | Opacity ±5% |
| Cmd+↑/↓ | Opacity ±1% |
| Enter×2 | Toggle kill |
| ESC | Prime stop |
| ESC×2 | Execute stop all |

---

## CORTEX CHANGES

### New Timing Constants

```javascript
const OMS_HOLD_TO_EDIT_MS = 500;
const OMS_DOUBLE_TAP_MS = 300;
const OMS_CONFIRM_TIMEOUT_MS = 500;
```

### .oms Schema Addition

```javascript
automationData: {
    rotation: [{beat, value, curveOut}, ...],
    scale: [...],
    tile: [...],
    translateX: [...],
    translateY: [...],
    pivotX: [...],
    pivotY: [...]
}
```

### .omsp Schema Updates

```javascript
audioCardSettings: {
    // ... existing ...
    inputDeviceId: 'none'  // NEW
}
```

---

## BUG FIXES

- ✅ Waveform grid uses dynamic playheadX (was hardcoded 387.5px)
- ✅ Correct BPM fallback is 120 (was incorrectly 128)
- ✅ Non-playing tracks display beat grid correctly
- ✅ Cue 5 renders at correct position
- ✅ All 5 cue jumps work properly
- ✅ Chroma key captured in render output
- ✅ RAF continues during render (no pause)
- ✅ A key in BumpR mode doesn't trigger Audio Focus
- ✅ Stop All requires double-click confirmation
- ✅ Arrow keys don't conflict with library navigation when track selected
- ✅ Render X button saves partial (not discard)

---

## COMPATIBILITY

✅ Fully backward compatible  
✅ All .oms containers work unchanged  
✅ All .omsp projects work (new fields use defaults)  
✅ automationData defaults to empty arrays  
✅ inputDeviceId defaults to 'none'

---

## INHERITED FEATURES

All features from previous versions remain:

**V5_7_0:** MP4 Render, JKL Transport, PIP Overhaul, BumpR Persistence, loopLength Uniform  
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
| Development Period | January 16-22, 2026 |
| Iterations | 20 (V5_7_02 - V5_7_21) |
| Lines Added | ~4,100 |
| New Functions | 35+ |
| New State Variables | 25+ |
| Bug Fixes | 11 |

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**
