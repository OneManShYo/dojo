# ONEMANSHYO App [Beta V5_2_0] - COMPLETE CHANGELOG

**Version:** Beta V5_2_0  
**Release Date:** December 26, 2025  
**Base Version:** V5_1_0  
**Development Iterations:** 51 sub-iterations (V5_1_01 through V5_1_51)

---

## EXECUTIVE SUMMARY

Beta V5_2_0 is a major release introducing comprehensive audio control systems: 3-band EQ, DJ FX (Washout/Dubby/Break), expanded 5-cue system, and per-track audio processing in PERFORM tab. The centralized focus mode architecture simplifies keyboard navigation and prevents mode conflicts. Audio Focus Mode (Shift+A) provides complete keyboard control of audio parameters.

**Key Achievements:**
1. **Audio Focus Mode** - Complete keyboard navigation for CREATE tab audio controls
2. **3-Band EQ System** - Professional DJ-style EQ with kill switches (H/M/L)
3. **DJ FX System** - Washout, Dubby, Break effects replacing generic FX
4. **5 Cue Points** - Expanded from 3, with visual panel and keyboard navigation
5. **Per-Track Audio Processing** - Independent EQ/FX per PERFORM track
6. **Centralized Focus Mode** - Single `activeFocusMode` replaces 8+ booleans
7. **Tab Isolation** - Proper keyboard handler separation between tabs

---

## MAJOR FEATURES

### 1. Audio Focus Mode (V5_1_01)

✅ **Complete Keyboard Navigation for Audio Controls**
| Feature | Description |
|---------|-------------|
| Activation | Shift+A (toggle) |
| Columns | 12 navigable controls |
| Acceleration | Hold keys for faster changes |
| Sub-Panels | V/C/E/F access Volume, Cue, EQ, FX |

✅ **Gate + Panel Pattern**
| Step | Keys | Result |
|------|------|--------|
| 1 | Shift+A | Enter Audio Focus (gate) |
| 2 | V/C/E/F | Open sub-panel |
| 3 | ↑↓←→ | Navigate within panel |
| 4 | Escape | Exit panel → focus → normal |

✅ **Audio Focus Columns (12 total)**
| Col | Control | Action |
|-----|---------|--------|
| 0-4 | Cue 1-5 | Enter=jump |
| 5 | Audio Start | ←→=adjust |
| 6 | Audio End | ←→=adjust |
| 7 | Tempo | ←→=adjust |
| 8 | Pitch | ←→=adjust |
| 9 | Loop In | ←→=adjust |
| 10 | Loop Out | ←→=adjust |
| 11 | Loop Toggle | Enter=toggle |

### 2. 3-Band EQ System (V5_1_02-09)

✅ **Professional DJ-Style EQ**
| Band | Frequency | Type | Q |
|------|-----------|------|---|
| High | 2500 Hz | Highshelf | - |
| Mid | 1000 Hz | Peaking | 0.7 |
| Low | 250 Hz | Lowshelf | - |

✅ **Ableton-Style Value Mapping**
| Knob Position | Value | dB | Description |
|---------------|-------|-----|-------------|
| Full left | 0.0 | -∞ | Kill (mute) |
| 9 o'clock | 0.25 | -12dB | Cut |
| 12 o'clock | 0.5 | -6dB | Attenuate |
| 3 o'clock | 0.75 | 0dB | Unity (default) |
| Full right | 1.0 | +6dB | Boost |

✅ **Visual States**
| State | Appearance |
|-------|------------|
| Normal | Cyan arc on dark background |
| Killed | Magenta fill (full circle) |
| Selected | Cyan ring around knob |

✅ **Keyboard (E Panel)**
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate H/M/L |
| ←/→ | Adjust gain |
| Enter | Kill/restore |
| Escape | Exit panel |

### 3. DJ FX System (V5_1_10-11, V5_1_49)

✅ **DJ Transition FX (V5_1_49 Redesign)**
| Slot | Name | Effect | Implementation |
|------|------|--------|----------------|
| 0 | Washout | Filter sweep | Highpass 20Hz→2kHz |
| 1 | Dubby | Dub echo | DelayNode + feedback |
| 2 | Break | Vinyl stop | playbackRate 1.0→0.1 |

✅ **Original FX (Replaced)**
| Old | New | Change |
|-----|-----|--------|
| Reverb | Washout | Multi-tap delay → Highpass sweep |
| Delay | Dubby | Same function, DJ name |
| Wash | Break | Highpass → Vinyl stop |

✅ **Audio Chain**
```
Source → EQ → Washout filter → [Dubby dry/wet] → Analyser → Gain → Destination
```
Note: Break uses playbackRate directly (no nodes in chain).

✅ **Keyboard (F Panel)**
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate Washout/Dubby/Break |
| ←/→ | Adjust wet amount |
| Enter | Kill/restore |
| Escape | Exit panel |

### 4. 5 Cue Points (V5_1_16)

✅ **Expanded from 3 to 5 Cues**
| Cue | Bar.Beat | Beats | Use Case |
|-----|----------|-------|----------|
| 1 | 1.1 | 0 | Intro |
| 2 | 17.1 | 64 | First drop |
| 3 | 33.1 | 128 | Second drop |
| 4 | 49.1 | 192 | Third section |
| 5 | 65.1 | 256 | Fourth section |

✅ **Cue Panel (C)**
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate cues 1-5 |
| ←/→ | Adjust position (bar snap) |
| Enter | Jump to cue |
| 1-5 | Direct jump |
| Escape | Exit panel |

✅ **.oms Storage**
```json
{
    "audioCuePoints": [0, 64, 128, 192, 256]
}
```
Legacy 3-cue containers auto-padded to 5.

### 5. Audio Controls Module (V5_1_14-19)

✅ **Unified Panel Overlay**
| Feature | Description |
|---------|-------------|
| Position | Right edge of waveform |
| Width | 115px |
| Panels | Volume, Cue, EQ, FX |
| Access | Shift+A then V/C/E/F |

✅ **Volume Knob with Kill (V5_1_41)**
| Feature | Description |
|---------|-------------|
| Size | 24×24 cyan knob |
| Position | Audio header (left of title) |
| Kill | Magenta when killed |
| Range | 0-100% |

✅ **Audio Header Layout (V5_1_51)**
```
Before: AUDIO CONTROLS | [Title scrolling] | [Time] [Vol] [SWAP] [X]
After:  AUDIO | [Vol] | [Title scrolling] | [Time] | [SWAP] [X]
```

### 6. Per-Track Audio Processing (V5_1_20-40)

✅ **Track Modules Replace Nudge Buttons**
| Before | After |
|--------|-------|
| Nudge buttons (│<, <<, >>, >│) | Cue/FX/EQ modules |
| Grid: `1fr 70px` | Grid: `1fr 115px` |

✅ **Per-Track Audio Routing**
```
AudioBufferSource
    ↓
EQ Filters (High → Mid → Low)
    ↓
FX Chain (Washout → Dubby)
    ↓ [Break uses playbackRate]
Analyser
    ↓
Track GainNode
    ↓
Master GainNode
    ↓
Destination
```

✅ **Per-Track State Arrays**
```javascript
// EQ [4 tracks][3 bands]
trackEQValues = [[0.75,0.75,0.75], ...];
trackEQKilled = [[false,false,false], ...];

// FX [4 tracks][3 slots]
trackFXValues = [[0.5,0.5,0.5], ...];
trackFXKilled = [[false,false,false], ...];

// Cues [4 tracks][5 cues]
trackCuePoints = [[0,64,128,192,256], ...];
```

✅ **PERFORM Keyboard**
| Keys | Action |
|------|--------|
| Shift+1-4 | Select track (gate) |
| C | Cue panel |
| E | EQ panel |
| F | FX panel |
| 1-5 | Jump to cue (no gate needed) |

### 7. Centralized Focus Mode System (V5_1_50)

✅ **Single Source of Truth**
```javascript
let activeFocusMode = 'none';
// Values: 'none', 'name', 'orientation', 'musicTiming', 
//         'transform', 'reactivity', 'audio', 'bumpr'

let activeAudioPanel = 'none';
// Values: 'none', 'volume', 'cue', 'eq', 'fx'
```

✅ **Compatibility Layer (Object.defineProperty)**
| Old Variable | Maps To |
|--------------|---------|
| `shaderNameFocus` | `activeFocusMode === 'name'` |
| `shaderOrientationFocus` | `activeFocusMode === 'orientation'` |
| `shaderMusicTimingFocus` | `activeFocusMode === 'musicTiming'` |
| `shaderTransformFocus` | `activeFocusMode === 'transform'` |
| `shaderReactivityFocus` | `activeFocusMode === 'reactivity'` |
| `createAudioFocus` | `activeFocusMode === 'audio'` |
| `bumprModeActive` | `activeFocusMode === 'bumpr'` |

✅ **Core Functions**
```javascript
enterFocusMode(mode, audioPanel)   // Auto-exits all others
exitAllFocusModes()                // Central cleanup
isAnyFocusModeActive()             // Simple check
setAudioPanel(panel)               // Audio sub-panel management
```

✅ **Problems Solved**
| Problem | Solution |
|---------|----------|
| Dual selection rings | Clear state BEFORE visuals |
| Dropdowns during focus | isAnyFocusModeActive() check |
| Code duplication (~10 blocks) | Single exitAllFocusModes() |
| Maintenance burden | One place to add modes |

### 8. Tab Isolation (V5_1_44)

✅ **Universal Tab Helpers (CORTEX)**
```javascript
isCreateTabActive()
isPerformTabActive()
isOrganizeTabActive()
isAdminTabActive()
```

✅ **Keyboard Handler Pattern**
```javascript
// All handlers now properly gated
if (isCreateTabActive() && createAudioPanel === 'eq') { ... }
if (isPerformTabActive() && performActivePanel !== 'none') { ... }
```

✅ **Variable Renames (97 occurrences)**
| Old | New |
|-----|-----|
| `shaderAudioFocus` | `createAudioFocus` |
| `shaderActivePanel` | `createAudioPanel` |
| `shaderEQSelectedBand` | `createEQSelectedBand` |
| `shaderFXSelectedSlot` | `createFXSelectedSlot` |
| `shaderCueSelectedRow` | `createCueSelectedRow` |

---

## ITERATION HISTORY

### V5_1_01: Audio Focus Mode
- Shift+A keyboard navigation
- 12 columns for audio controls
- Progressive acceleration

### V5_1_02-09: EQ3 System
| Build | Focus |
|-------|-------|
| V5_1_02 | Visual panel layout |
| V5_1_03 | Audio wiring (BiquadFilterNode) |
| V5_1_04 | Kill switches |
| V5_1_05 | Layout order fix (H/M/L) |
| V5_1_06-09 | Knob interactions, visual polish |

### V5_1_10-11: FX System (Original)
| Build | Focus |
|-------|-------|
| V5_1_10 | Visual UI panel |
| V5_1_11 | Standardized cyan color |

### V5_1_13: ADMIN Tab Restructure
- New navigation: AUDIO > EQ/FX, VISUALS > BumpR StickRs, Settings...

### V5_1_14-19: Audio Controls Module
| Build | Focus |
|-------|-------|
| V5_1_14 | Initial panel overlay |
| V5_1_15 | Cue panel |
| V5_1_16 | 5 cue points (expanded from 3) |
| V5_1_17-18 | EQ/FX panels |
| V5_1_19 | Cue 4/5 hotkeys |

### V5_1_20-40: PERFORM Tab Track Modules
| Build | Focus |
|-------|-------|
| V5_1_20-25 | Per-track EQ wiring |
| V5_1_26-28 | Per-track FX wiring |
| V5_1_29-33 | Cue jump implementation |
| V5_1_34-35 | Master clock fixes |
| V5_1_36-37 | Unified panel architecture |
| V5_1_38-40 | Track module layout |

### V5_1_41-43: CREATE Tab Enhancements
| Build | Focus |
|-------|-------|
| V5_1_41 | Volume knob with kill |
| V5_1_42 | Kill visual states |
| V5_1_43 | Keyboard navigation |

### V5_1_44: Tab Isolation
- Universal tab helpers
- Variable renames (shader* → create*)
- Keyboard handler gating

### V5_1_45-48: Audio Header Info
| Build | Focus |
|-------|-------|
| V5_1_45 | Scrolling container name |
| V5_1_46 | Time display |
| V5_1_47 | Marquee animation |
| V5_1_48 | Dropdown loading optimization |

### V5_1_49: DJ FX Redesign
- Reverb/Delay/Wash → Washout/Dubby/Break
- Multi-knob drag bug fix
- Dropdown focus mode blocking

### V5_1_50: Centralized Focus Mode
- Single activeFocusMode variable
- Compatibility layer (Object.defineProperty)
- exitAllFocusModes() central cleanup

### V5_1_51: Audio Header Layout
- Header shortened: "AUDIO CONTROLS" → "AUDIO"
- Volume knob moved left
- Compatibility layer bug fix

---

## NEW GLOBAL STATE

### Centralized Focus Mode (V5_1_50)
```javascript
let activeFocusMode = 'none';
let activeAudioPanel = 'none';
```

### CREATE Tab Audio Controls
```javascript
// Volume
let createAudioVolume = 0.5;
let createAudioVolumeKilled = false;
let createAudioGainNode = null;

// EQ
let eqLowValue = 0.75, eqMidValue = 0.75, eqHighValue = 0.75;
let eqLowKilled = false, eqMidKilled = false, eqHighKilled = false;
let eqLowFilter = null, eqMidFilter = null, eqHighFilter = null;
let createEQSelectedBand = 0;
let eqKeyRepeatCount = 0, eqLastKeyTime = 0;

// FX (V5_1_49 renamed)
let fxWashoutValue = 0.5, fxDubbyValue = 0.5, fxBreakValue = 0.5;
let fxWashoutKilled = false, fxDubbyKilled = false, fxBreakKilled = false;
let createFXSelectedSlot = 0;

// Drag tracking (V5_1_49 bug fix)
let fxDraggingSlot = -1;
let eqDraggingBand = -1;

// Cue
let createCueSelectedRow = 0;

// Audio Focus
let shaderAudioSelectedCol = 0;
let audioKeyRepeatCount = 0;
let audioLastKeyTime = 0;
```

### PERFORM Tab
```javascript
let performActivePanel = 'none';
let performEQSelectedBand = 0;
let performFXSelectedSlot = 0;
let performCueSelectedRow = 0;

// Per-Track EQ [4 tracks][3 bands]
let trackEQValues = [[0.75,0.75,0.75], [0.75,0.75,0.75], [0.75,0.75,0.75], [0.75,0.75,0.75]];
let trackEQKilled = [[false,false,false], [false,false,false], [false,false,false], [false,false,false]];

// Per-Track FX [4 tracks][3 slots]
let trackFXValues = [[0.5,0.5,0.5], [0.5,0.5,0.5], [0.5,0.5,0.5], [0.5,0.5,0.5]];
let trackFXKilled = [[false,false,false], [false,false,false], [false,false,false], [false,false,false]];

// Per-Track Cues [4 tracks][5 cues]
let trackCuePoints = [[0,64,128,192,256], [0,64,128,192,256], [0,64,128,192,256], [0,64,128,192,256]];

// Audio Nodes
let trackEQFilters = [];
let trackFXDelayNodes = [];
let trackFXDelayFeedback = [];
let trackFXDelayWet = [];
let trackFXWashFilter = [];
let trackFXWashWet = [];
```

### Cue Points (Expanded)
```javascript
let audioCuePoints = [0, 64, 128, 192, 256];  // Was [0, 64, 128]
```

---

## NEW FUNCTIONS (~50 total)

### Focus Mode Core
```javascript
enterFocusMode(mode, audioPanel)
exitAllFocusModes()
isAnyFocusModeActive()
isFocusModeActive(mode)
setAudioPanel(panel)
```

### Audio Focus Mode
```javascript
updateAudioSelection()
exitAudioMode()
jumpToCue(cueIndex)
adjustAudioValue(delta)
activateAudioControl()
```

### EQ System (CREATE)
```javascript
initEQ3Knobs(idPrefix)
applyEQToFilters()
updateEQSelection()
exitEQMode()
adjustEQValue(delta)
knobToDb(value)
getEQdBString(band)
drawEQKnob(canvas, value, color, killed, selected)
```

### FX System (CREATE)
```javascript
initFXKnobs(idPrefix)
applyFXToNodes()
updateFXSelection()
adjustFXValue(delta)
toggleFXKill(slotId)
drawFXKnob(canvas, value, on, killed, selected)
applyWashoutFilter()
applyDubbyDelay()
applyBreakEffect()
```

### Volume Knob (CREATE)
```javascript
initCreateAudioVolumeKnob()
drawCreateAudioVolumeKnob()
applyCreateAudioVolume()
toggleCreateAudioVolumeKill()
updateVolumeSelection()
adjustVolumeValue(delta)
```

### Panel Management (CREATE)
```javascript
setShaderPanel(panel)
exitShaderPanel()
```

### Header Info (CREATE)
```javascript
updateCreateWaveformOverlay()
hideCreateWaveformOverlay()
```

### EQ System (PERFORM)
```javascript
redrawTrackEQKnobs(trackIndex)
adjustPerformEQValue(delta)
updatePerformEQSelection()
togglePerformEQKill()
applyTrackEQ(trackIndex)
```

### FX System (PERFORM)
```javascript
redrawTrackFXKnobs(trackIndex)
adjustPerformFXValue(delta)
updatePerformFXSelection()
togglePerformFXKill()
applyTrackFX(trackIndex)
```

### Cue System (PERFORM)
```javascript
highlightTrackCue(trackIndex, cueIndex)
clearTrackCueHighlights(trackIndex)
jumpToSelectedPerformCue()
jumpToTrackCue(trackNum, cueIndex)
updatePerformCueSelection()
```

### Panel Management (PERFORM)
```javascript
setPerformPanel(panel)
exitPerformPanel()
updatePerformPanelHighlights(oldTrack)
clearPerformPanelHighlights(trackIndex)
```

### Tab Helpers (CORTEX)
```javascript
isCreateTabActive()
isPerformTabActive()
isOrganizeTabActive()
isAdminTabActive()
```

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| `buildContainerObject()` | Added 5 cue points |
| `loadContainer()` | Restore cues, auto-pad legacy |
| `drawProgress()` | Draw 5 cue flags |
| `drawWaveformFromAudioBuffer()` | PERFORM 5 cue flags |
| `initAudioRouting()` | EQ → FX → Analyser chain |
| `OmsDropdown.toggle()` | Check isAnyFocusModeActive() |

---

## CSS ADDITIONS

### EQ/FX Knob System
```css
.eq-knob-container { ... }
.fx-knob-container { ... }
.eq-knob-canvas { width: 24px; height: 24px; }
.fx-knob-canvas { width: 24px; height: 24px; }
.mod-selected { box-shadow: 0 0 0 2px #00d4ff; }
```

### Audio Controls Module
```css
.audio-controls-module {
    position: absolute;
    right: 0;
    top: 0;
    width: 115px;
    background: rgba(30, 30, 30, 0.95);
}
.audio-panel { ... }
.cue-panel { ... }
.eq-panel { ... }
.fx-panel { ... }
```

### Track Modules (PERFORM)
```css
.track-module {
    width: 115px;
    display: grid;
    grid-template-rows: auto auto auto;
}
.track-cue-row { ... }
.track-eq-row { ... }
.track-fx-row { ... }
```

### Volume Knob
```css
.audio-volume-knob {
    width: 24px;
    height: 24px;
    cursor: pointer;
}
```

---

## DOM ELEMENTS ADDED

### Audio Controls Module (CREATE)
- `.audio-controls-module` overlay on waveform
- Volume panel with knob
- Cue panel with 5 buttons
- EQ panel with 3 knobs
- FX panel with 3 knobs

### Track Modules (PERFORM)
- Per-track `.track-module` (4 total)
- Cue row with 5 buttons each
- EQ row with 3 knobs each
- FX row with 3 knobs each

### Audio Header (CREATE)
- Volume knob in header
- Scrolling title container
- Time display with separator

---

## BUG FIXES (18 total)

| Build | Issue | Fix |
|-------|-------|-----|
| V5_1_05 | EQ layout order wrong | H/M/L top to bottom |
| V5_1_19 | Cue 4/5 hotkeys missing | Extended handlers |
| V5_1_27 | Cue jump placeholder | Implemented seeking |
| V5_1_29 | Cue launch from stopped | Start playback on jump |
| V5_1_30 | Tab detection broken | Use liveMixerContainer |
| V5_1_30-31 | Waveform not scrolling | Init master clock |
| V5_1_31-32 | Arrow keys leaking | All modes handle arrows |
| V5_1_33 | Multi-track cue launch | stopImmediatePropagation |
| V5_1_35 | Master clock check | Check null not beat value |
| V5_1_36 | Panel highlight persist | Single state variable |
| V5_1_44 | Tab isolation | Add tab visibility helpers |
| V5_1_49 | Multi-knob drag | Track dragging slot |
| V5_1_49 | Dropdown during focus | isAnyFocusModeActive check |
| V5_1_50 | Dual selection rings | Clear state before visuals |
| V5_1_50 | Exit code duplication | Centralized exitAllFocusModes |
| V5_1_51 | Keyboard shortcuts broken | Add compatibility layer |

---

## BREAKING CHANGES

### Cue Points
- **Before:** 3 cue points [0, 64, 128]
- **After:** 5 cue points [0, 64, 128, 192, 256]
- **Migration:** Legacy auto-padded

### FX Slots (V5_1_49)
| Old | New |
|-----|-----|
| fxReverbValue/On/Killed | fxWashoutValue/On/Killed |
| fxDelayValue/On/Killed | fxDubbyValue/On/Killed |
| fxWashValue/On/Killed | fxBreakValue/On/Killed |

### Focus Mode Variables
- **Before:** 8+ independent booleans
- **After:** Single `activeFocusMode` + `activeAudioPanel`
- **Compatibility:** Old names work via getters/setters

### CREATE Tab Variables
| Old | New |
|-----|-----|
| shaderAudioFocus | createAudioFocus |
| shaderActivePanel | createAudioPanel |
| shaderEQSelectedBand | createEQSelectedBand |
| shaderFXSelectedSlot | createFXSelectedSlot |
| shaderCueSelectedRow | createCueSelectedRow |

### PERFORM Track Layout
- Nudge buttons removed
- Track modules added (115px)

### Keyboard Pattern
- **Before:** Direct panel toggle (Shift+E/F/C)
- **After:** Gate pattern (Shift+A then E/F/C)

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Total Iterations | 51 |
| Lines Added | ~3,500 |
| New Functions | ~50 |
| New State Variables | ~30 |
| Bug Fixes | 18 |
| Cue Points | 5 (was 3) |
| EQ Bands | 3 |
| FX Slots | 3 |
| Per-Track Arrays | 12 (4×3) |

---

## COMPATIBILITY

✅ Backward compatible with V5_1_0, V5_0_0
✅ Existing shaders work unchanged
✅ Existing 3-cue .oms containers load (auto-padded to 5)
✅ Existing .omsp projects load (EQ/FX arrays initialized)
✅ Old focus mode variable names still work
✅ Uniform struct unchanged (112 bytes)

---

## KNOWN ISSUES

1. **Orientation dropdown sync** - Visual dropdown may not match loaded orientation value (inherited from V5_1_0)

---

## PRODUCTION READINESS: ~99%

**100% Complete:**
- Audio Focus Mode
- EQ3 System (CREATE + PERFORM)
- DJ FX System (CREATE + PERFORM)
- 5 Cue Points
- Per-Track Audio Processing
- Centralized Focus Mode
- Tab Isolation
- Volume Knob with Kill

**Inherited from V5_1_0 (100%):**
- BumpR StickRs
- Transform controls
- Reactivity controls
- Design System

**Remaining:**
- MIDI integration
- Orientation dropdown sync

---

**END OF COMPLETE CHANGELOG**

Version: Beta V5_2_0
Date: December 26, 2025
Total Development: 51 sub-iterations
License: GPL-3.0
