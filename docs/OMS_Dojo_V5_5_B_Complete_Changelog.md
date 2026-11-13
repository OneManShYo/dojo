# ONEMANSHYO App [Beta V5_5_0] - COMPLETE CHANGELOG

**Version:** Beta V5_5_0  
**Release Date:** January 3, 2026  
**Base Version:** V5_4_0  
**Development Iterations:** 29 iterations + ~45 sub-iterations (V5_4_01 through V5_4_29)

---

## V5_5_0 EXECUTIVE SUMMARY

Beta V5_5_0 is a major release delivering **creative expression tools and workflow efficiency**. The Transform Animation system enables beat-synced rotation, scale, and tile modes with exponential speed control. Per-Mod ADSR+T envelopes provide independent audio reactivity tuning for each frequency band. The PIP Camera system adds professional multi-overlay video for streaming/recording. ORGANIZE tab receives Apple Music-style UX with tag autocomplete. PERFORM tab gets keyboard-driven rapid loading workflow.

**Key Achievements:**
1. **Transform Animation System** - Beat-synced rotation/scale/tile with 0.03x-256x speed range
2. **Per-Mod ADSR+T Envelopes** - Independent Q/A/D/S/R/T for ModA/B/C (18 parameters)
3. **PIP Camera System** - 3 PIP types + full-screen overlay with shapes/colors/persistence
4. **ORGANIZE Tab UX Overhaul** - Apple Music-style auto-save, tag autocomplete
5. **PERFORM Keyboard Workflow** - Bare number keys, SHIFT+Right quick-load
6. **FX Overlay System** - Toggle panel with VFX placeholders
7. **Audio Timing Bug Fix** - Critical fix for correct tempo on all load methods

---

## V5_5_0 MAJOR FEATURES

### 1. Transform Animation System (V5_4_01-07)

✅ **Beat-Synced Modes**
| Transform | Modes | Range |
|-----------|-------|-------|
| Rotation | Stop, CW, CCW, PingPong | 0→360° |
| Scale | Stop, Grow, Shrink, PingPong | 0.1→4.0 |
| Tile | Stop, Increase, Decrease, PingPong | 1→8 integers |

✅ **Mode Timing**
| Mode | Waveform | Behavior |
|------|----------|----------|
| CW / Grow / Increase | Sawtooth | Linear ramp up, reset |
| CCW / Shrink / Decrease | Inverse Sawtooth | Linear ramp down, reset |
| PingPong | Triangle | Bounce between min/max |

✅ **Speed Control**
| Input | Range | Formula |
|-------|-------|---------|
| Trackpad scroll | 0.03x - 256x | `factor = Math.pow(1.01, -deltaY)` |
| Mode keys | 0/1/2/3 | Stop/CW/CCW/PingPong |

✅ **Loop Length Dropdown (Reordered)**
```
No Loop
───────
1/64, 1/32, 1/16, 1/8, 1/4, 1/2, 1 Bar, 2 Bars, 4 Bars, 8 Bars, 16 Bars, 32 Bars
```
- Removed: cycler, chaosmode, madmax, psycho (not timing modes)

✅ **Container Persistence**
```javascript
controlStates: {
    transformRotationMode: 'clockwise',
    transformRotationSpeed: 1.0,
    transformScaleMode: 'pingpong',
    transformScaleSpeed: 2.0,
    transformTileMode: 'stop',
    transformTileSpeed: 1.0
}
```

### 2. Per-Mod ADSR+T Envelopes (V5_4_08-09)

✅ **Independent Parameters (18 total)**
| Mod | Q | Attack | Decay | Sustain | Release | Threshold |
|-----|---|--------|-------|---------|---------|-----------|
| A (Bass) | 0.70 | 5ms | 20ms | 70% | 50ms | 10% |
| B (Mids) | 0.70 | 10ms | 30ms | 80% | 100ms | 5% |
| C (Treble) | 0.70 | 2ms | 15ms | 60% | 80ms | 5% |

✅ **N-Panel Redesign**
| Before | After |
|--------|-------|
| Global ADSR for all mods | Per-mod ADSR+T |
| Threshold 0-20% | Threshold 0-100% |
| Horizontal layout | Vertical column with readouts |
| Fixed panel | Syncs to selected Reactivity row |

✅ **Row Sync Behavior**
| Reactivity Row | N-Panel Shows |
|----------------|---------------|
| Mod A selected | ModA envelope controls |
| Mod B selected | ModB envelope controls |
| Mod C selected | ModC envelope controls |

✅ **Container Persistence**
```javascript
controlStates: {
    modAQ: 0.70, modAAttack: 5, modADecay: 20, modASustain: 70, modARelease: 50, modAThreshold: 10,
    modBQ: 0.70, modBAttack: 10, modBDecay: 30, modBSustain: 80, modBRelease: 100, modBThreshold: 5,
    modCQ: 0.70, modCAttack: 2, modCDecay: 15, modCSustain: 60, modCRelease: 80, modCThreshold: 5
}
```

### 3. PIP Camera System (V5_4_25-29)

✅ **Architecture**
| Layer | Prefix | Key | Tabs | Purpose |
|-------|--------|-----|------|---------|
| Services | `cam*` | - | - | Shared camera stream/settings |
| Full Screen | `camOverlay*` | O | ALL | Full-screen camera overlay |
| PIP 1 | `pip1*` | P | PERFORM | Slideout above shader |
| PIP 2 | `pip2*` | P | CREATE/ORG/ADMIN | Top-right corner |
| PIP 3 | `pip3*` | Shift+P | CREATE/ORG/ADMIN | Canvas center |

✅ **Shape Options**
| Shape | Aspect | Border Radius |
|-------|--------|---------------|
| Landscape | 16:9 | 4px |
| Portrait | 9:16 | 4px |
| Square | 1:1 | 4px |
| Circle | 1:1 | 50% |

✅ **Color Options (9)**
| Name | Value |
|------|-------|
| None | transparent |
| Cyan | #00d4ff |
| Magenta | #ff00ff |
| Orange | #ffa500 |
| Yellow | #ffff00 |
| Green | #00ff00 |
| Purple | #8800ff |
| Pink | #ff69b4 |
| White | #ffffff |

✅ **Border-Click Color Picker**
| Feature | Behavior |
|---------|----------|
| Trigger | Click PIP border (video has pointer-events: none) |
| UI | 9 swatches (16×16px) in floating div |
| Dismiss | Click outside picker |

✅ **Genie Zoom Animations**
| PIP | transform-origin |
|-----|------------------|
| pip1 | center |
| pip2 | top right |
| pip3 | center |

✅ **ADMIN → Video Sources**
```
CAMERA SOURCE
├── Device dropdown
└── Mirror checkbox

PIP 1 (PERFORM Slideout)
├── Frame Color (swatch picker)
└── Shape (dropdown)

PIP 2 (Corner)
├── Frame Color (swatch picker)
└── Shape (dropdown)

PIP 3 (Center)
├── Frame Color (swatch picker)
└── Shape (dropdown)

DIAGNOSTICS
├── Test Camera / Copy Report
└── Log output
```

✅ **Smart Stream Management**
```javascript
function isCamInUse() {
    return pip1Visible || pip2Visible || pip3Visible || camOverlayVisible;
}
// stopCamStream() only stops if no outputs using it
```

✅ **Project State Persistence (.omsp)**
```javascript
{
    videoSourcesSettings: { cameraSourceId: 'default', cameraMirror: false },
    pip1Settings: { visible: false, color: '#00d4ff', label: 'PIP 1', shape: 'landscape' },
    pip2Settings: { color: '#00d4ff', shape: 'landscape' },
    pip3Settings: { color: '#00d4ff', shape: 'square' }
}
```

### 4. ORGANIZE Tab UX Overhaul (V5_4_13-16)

✅ **Apple Music-Style Info Drawer**
| Before | After |
|--------|-------|
| OK/Cancel buttons | Auto-save on blur |
| Modal workflow | Always-editable fields |
| Separate delete button | Removed (use Delete key) |

✅ **Editable Fields**
| Field | Behavior |
|-------|----------|
| NAME | Text input, renames file on blur |
| TEMPO | Number input (60-200), saves on blur |
| LOOP IN/OUT | Bar.beat format, saves on blur |
| TAGS | Chips with autocomplete |

✅ **Tag Autocomplete**
| Feature | Behavior |
|---------|----------|
| Trigger | Typing in tag input |
| Match | Contains (case-insensitive) |
| Display | "House (5)" with usage count |
| Navigate | ↑/↓ arrows |
| Select | Enter or click |
| Limit | 8 suggestions (sorted by count) |
| Filter | Excludes tags already on container |

✅ **Quick Rename**
| Input | Trigger |
|-------|---------|
| Mouse | Long-press (500ms) on name |
| Keyboard | Enter key on selected item |

✅ **Arrow Navigation**
| Key | Behavior |
|-----|----------|
| ↑/↓ | Navigate containers (saves current, refreshes drawer) |
| Tab | Cycle through fields |
| Escape | Save and close drawer |

✅ **Selection Visual States**
| State | Border | Background |
|-------|--------|------------|
| Default | #333 | 3% white |
| Hover | #0ff | unchanged |
| Selected | #0ff | 10% cyan |
| Loaded | #ffcc00 | 10% yellow |
| Selected+Loaded | #ffcc00 | 15% yellow |

### 5. PERFORM Tab Keyboard Workflow (V5_4_17-18)

✅ **Track Selection**
| Key | Action |
|-----|--------|
| 1-4 | Select track 1-4 (bare keys, no modifier) |
| 0 | Deselect track |
| Shift+1-4 | Also selects track (legacy) |

✅ **Quick Load Workflow**
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate library (when track selected) |
| SHIFT+Right | Load container to next empty cell |
| SHIFT+Left | Return focus to library panel |
| Enter | Load selected container to selected cell |

✅ **Scan Order (SHIFT+Right)**
```
Scene 1: T1S1 → T2S1 → T3S1 → T4S1
Scene 2: T1S2 → T2S2 → T3S2 → T4S2
... continues horizontally through all 8 scenes
```

✅ **Visual Feedback**
| Action | Visual |
|--------|--------|
| Press 1-4 | Yellow highlight on track header |
| Hold ←/→ | Purple ring on opacity knob |
| Hold ↑/↓ | Cyan ring on volume knob |
| Release | Ring disappears |

✅ **Audio Timing Bug Fix (V5_4_17h - CRITICAL)**
| Aspect | Detail |
|--------|--------|
| Problem | Containers loaded via SHIFT+Right played with wrong tempo |
| Root Cause | `assignContainerToCell()` overwrote `trackContainerData` while playing |
| Fix | Skip `trackContainerData` update if track is playing |
| Result | Playback always uses correct tempo regardless of load method |

### 6. FX Overlay System (V5_4_23-24)

✅ **Toggle Button**
| Aspect | Value |
|--------|-------|
| Size | 16×16px |
| Color | Orange (#ffa500) |
| Position | Next to meter toggle |
| Function | `toggleFX()` |

✅ **Overlay Structure**
```html
<div class="fx-overlay">
    <div class="fx-column" data-track="1">
        <div class="meter-module-wrap"><!-- VFX A/B/C --></div>
        <div class="meter-module-wrap"><!-- FX R/D/W --></div>
    </div>
    <!-- Tracks 2-4 + Scenes spacer -->
</div>
```

✅ **Knob Standardization**
| Before | After |
|--------|-------|
| EQ 30×30 | EQ 20×20 |
| FX 30×30 | FX 20×20 |
| VFX missing | VFX 20×20 (purple) |

✅ **VFX Placeholders**
| Feature | Status |
|---------|--------|
| Knobs | Interactive (drag works) |
| Arc color | Purple (#a855f7) |
| Backend | Not connected (placeholder) |

### 7. Header & Layout Fixes (V5_4_10-12, 19)

✅ **PERFORM Header Consolidation**
| Before | After |
|--------|-------|
| Tempo in MAIN bottom | Tempo in header |
| Quantize in MAIN bottom | Quantize in header |
| Header sparse | Header functional |

✅ **MAIN Cue Panel Relocation**
| Before | After |
|--------|-------|
| Cue panel in waveform overlay | Cue panel in MAIN bottom module |
| Floating over waveforms | Fixed position |
| `.track-audio-module` wrapper | Same pattern |

✅ **Layout Gap Fixes**
| Fix | Before | After |
|-----|--------|-------|
| Scenes column | auto | 1fr |
| MAIN column | 110px | 114px |
| Waveform panel | 115px | 101px |

---

## V5_5_0 ITERATION DETAILS

### Transform Animation System (V5_4_01-07)

**V5_4_01: Rotation Modes Persistence**
- Added `transformRotationMode` and `transformRotationSpeed` to controlStates
- Save/load in `buildContainerObject()` and `loadContainer()`
- Synced to PERFORM tab via container data

**V5_4_02/02a: Scale Modes**
- Added `transformScaleMode` and `transformScaleSpeed`
- V5_4_02a: Fixed frame-based → beat-synced timing

**V5_4_03: Rotation Beat-Sync Conversion**
- Converted rotation from frame-based to beat-synced
- Uses `shaderLoopLength` for timing reference

**V5_4_04/04a: Loop Length Dropdown**
- V5_4_04: Reordered options (smallest to largest)
- V5_4_04a: Removed chaos modes (cycler, chaosmode, madmax, psycho)

**V5_4_05: Tile Modes**
- Added `transformTileMode` and `transformTileSpeed`
- Integer steps (1→8), beat-synced

**V5_4_06a: Focus Mode Fixes**
- Fixed dropdown blocking Transform Focus entry
- Added blur on focus mode entry

**V5_4_07: Extended Speed Range**
- Added 1/64, 1/32 to loop length
- Trackpad exponential: 0.03x-256x range

### Per-Mod Envelopes (V5_4_08-09)

**V5_4_08: Mod Tuning Panel Redesign**
- Threshold expanded to 0-100%
- Vertical layout with live readouts
- Knobs with ms/% units

**V5_4_09: Per-Mod ADSR+T**
- Split global ADSR into per-mod (18 parameters)
- N-panel syncs to selected Reactivity row
- Backward compatibility with global fallback

### Layout Fixes (V5_4_10-12)

**V5_4_10:** Scenes column gap fix (auto → 1fr)
**V5_4_11:** MAIN column width fix (110px → 114px)
**V5_4_12:** Waveform panel gap fix (115px → 101px)

### ORGANIZE Tab UX (V5_4_13-16)

**V5_4_13 (13a-13f): Foundation**
- 13a: Name editing via long-press
- 13b: Enter key for rename
- 13c: Arrow key navigation fixes
- 13d: Selection visual states
- 13e: Navigation edge cases
- 13f: Built-in set protection

**V5_4_14: Info Drawer Auto-Refresh**
- Arrow navigation triggers drawer refresh
- Current container always displayed

**V5_4_15: Apple Music-Style**
- Removed OK/Cancel/Delete buttons
- Auto-save on blur
- Always-editable fields

**V5_4_16: Tag Autocomplete**
- Contains matching
- Usage counts
- ↑/↓ navigation
- 8 suggestion limit

### PERFORM Keyboard Workflow (V5_4_17-18)

**V5_4_17 (17a-17h): Comprehensive Overhaul**
- 17a: Bare number keys (1-4)
- 17b: Key 0 deselect
- 17c: Arrow library navigation
- 17d: SHIFT+Right quick-load
- 17e: SHIFT+Left exit
- 17f: Ghost cell fix
- 17g: Debug functions
- **17h: Critical audio timing bug fix**

**V5_4_18: Track Selection Visual Feedback**
- Yellow header highlight
- Domain-colored knob rings (purple/cyan)
- Rings appear only during adjustment

### Header Consolidation (V5_4_19)

**V5_4_19 (19a-19L): PERFORM Header**
- Tempo knob moved to header
- Quantize dropdown moved to header
- Cue panel relocated to MAIN bottom module
- `.track-audio-module` wrapper pattern

### FX Overlay (V5_4_23-24)

**V5_4_23/23a: FX Slideout**
- 120px slideout panel
- V|A dual-module structure
- Orange toggle button

**V5_4_24/24a: Knob Size**
- Standardized to 20×20
- VFX placeholders added (purple)

### PIP Camera System (V5_4_25-29)

**V5_4_25 (25a-25f): Initial Implementation**
- 25a: Basic slideout PIP
- 25b: Multi-PIP rename (multiPIP* → pip*)
- 25c: ADMIN Video Sources section
- 25d: Flat ADMIN structure
- 25e: .omsp persistence
- 25f: Camera stream management

**V5_4_26: Services Rename**
- pipCamera* → cam*
- O key for full-screen overlay
- Smart stream management

**V5_4_27: pip2 Corner PIP**
- JS-positioned (not CSS)
- ADMIN label management
- P key context-aware

**V5_4_28: pip3 Center Square**
- Shift+P trigger
- Default square shape
- Center positioning

**V5_4_29 (29a-29f): Enhancements**
- 29a: Border-click color picker
- 29b: Genie zoom animations
- 29c: Unified settings architecture
- 29d: Shape controls (all PIPs)
- 29e: Color persistence
- 29f: Final polish

---

## NEW STATE VARIABLES (~40 total)

### Transform Animation (~6 variables)
```javascript
transformModes.rotation = { mode: 'stop', speed: 1.0 };
transformModes.scale = { mode: 'stop', speed: 1.0 };
transformModes.tile = { mode: 'stop', speed: 1.0 };
```

### Per-Mod Envelopes (~18 variables)
```javascript
let modAQ = 0.70, modAAttack = 5, modADecay = 20, modASustain = 70, modARelease = 50, modAThreshold = 10;
let modBQ = 0.70, modBAttack = 10, modBDecay = 30, modBSustain = 80, modBRelease = 100, modBThreshold = 5;
let modCQ = 0.70, modCAttack = 2, modCDecay = 15, modCSustain = 60, modCRelease = 80, modCThreshold = 5;
```

### Tag Autocomplete (~2 variables)
```javascript
let tagAutocompleteIndex = -1;
let tagSuggestions = [];
```

### FX Overlay (~2 variables)
```javascript
let fxVisible = false;
let trackVFXValues = [[0,0,0], [0,0,0], [0,0,0], [0,0,0]];
```

### Camera System (~15 variables)
```javascript
let camSettings = { sourceId: 'default', mirror: false };
let camDevices = [];
let camStream = null;
let camOverlayVisible = false;

let pip1Visible = false;
let pip1Color = '#00d4ff';
let pip1Shape = 'landscape';
let pip1DefaultLabel = 'PIP 1';

let pip2Visible = false;
let pip2Color = '#00d4ff';
let pip2Shape = 'landscape';

let pip3Visible = false;
let pip3Color = '#00d4ff';
let pip3Shape = 'square';
```

---

## NEW FUNCTIONS (~35 total)

### Transform Animation
```javascript
applyTransformAnimation(transform, beat)
getTransformModeValue(mode, speed, beat, min, max)
updateTransformModeUI(transform)
saveTransformModes()
loadTransformModes(controlStates)
```

### Per-Mod Envelopes
```javascript
updateModTuningPanel()
syncModTuningToRow(row)
getPerModEnvelopeState()
applyPerModEnvelopeState(state)
drawModTuningKnob(canvas, value, min, max)
```

### Tag Autocomplete
```javascript
showTagSuggestions(query)
hideTagSuggestions()
selectTagSuggestion(index)
navigateTagSuggestions(direction)
getTagUsageCounts()
```

### Camera System
```javascript
initCameraSystem()
startCamStream()
stopCamStream()
isCamInUse()
toggleCamOverlay()
togglePip1()
togglePip2()
togglePip3()
updatePipShape(pip, shape)
updatePipColor(pip, color)
showPipColorPicker(pip, event)
hidePipColorPicker()
saveCameraSettings()
loadCameraSettings(settings)
enumerateCamDevices()
testCamera()
copyDiagnosticReport()
```

### PERFORM Workflow
```javascript
selectTrackByNumber(trackNum)
deselectTrack()
navigateLibraryFromPerform(direction)
quickLoadToNextCell()
returnToLibraryFocus()
findNextEmptyCell()
showTrackSelectionFeedback(trackNum)
hideTrackSelectionFeedback()
```

### FX Overlay
```javascript
toggleFX()
initFXOverlay()
drawVFXKnob(canvas, value)
```

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| `buildContainerObject()` | Added transform modes, per-mod envelopes |
| `loadContainer()` | Restore transform modes, per-mod envelopes |
| `applyControlState()` | Handle per-mod envelope fields |
| `getCurrentControlState()` | Include per-mod envelope fields |
| `initLiveTab()` | Track selection, quick-load support |
| `assignContainerToCell()` | Skip update if track playing (bug fix) |
| `launchCell()` | Set trackContainerData from actual container |
| `handleKeyDown()` | Bare number keys, SHIFT+arrows |
| `refreshLibraryList()` | Arrow nav from PERFORM |
| `openInfoDrawer()` | Auto-save workflow |
| `closeInfoDrawer()` | Remove OK/Cancel logic |
| `addTagChip()` | Autocomplete integration |

---

## CSS ADDITIONS

### Transform Animation
```css
.transform-mode-indicator {
    font-size: 10px;
    color: #888;
}
.transform-mode-indicator.active {
    color: #00d4ff;
}
```

### Tag Autocomplete
```css
.tag-suggestions {
    position: absolute;
    background: #1a1a1a;
    border: 1px solid #333;
    max-height: 200px;
    overflow-y: auto;
}
.tag-suggestion {
    padding: 6px 10px;
    cursor: pointer;
}
.tag-suggestion:hover,
.tag-suggestion.selected {
    background: rgba(0, 212, 255, 0.2);
}
.tag-count {
    color: #666;
    margin-left: 4px;
}
```

### PIP Camera
```css
.pip-container {
    position: absolute;
    border: 2px solid var(--pip-color, #00d4ff);
    overflow: hidden;
    transition: transform 0.3s ease-out;
}
.pip-container.landscape { aspect-ratio: 16/9; }
.pip-container.portrait { aspect-ratio: 9/16; }
.pip-container.square { aspect-ratio: 1/1; }
.pip-container.circle { aspect-ratio: 1/1; border-radius: 50%; }

.pip-color-picker {
    position: absolute;
    display: flex;
    gap: 4px;
    padding: 4px;
    background: #1a1a1a;
    border: 1px solid #333;
}
.pip-color-swatch {
    width: 16px;
    height: 16px;
    cursor: pointer;
}

.cam-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 9999;
}
```

### FX Overlay
```css
.fx-overlay {
    position: absolute;
    height: 120px;
    background: #111;
}
.fx-column {
    display: flex;
    flex-direction: column;
}
.vfx-knob {
    width: 20px;
    height: 20px;
}
```

### Track Selection Feedback
```css
.track-header.selected {
    background: rgba(255, 255, 0, 0.2);
}
.knob-ring {
    position: absolute;
    border: 2px solid var(--ring-color);
    border-radius: 50%;
    pointer-events: none;
}
```

---

## DOM ELEMENTS ADDED

### Camera System
- `#cam-overlay` - Full-screen camera overlay
- `#pip1-container`, `#pip2-container`, `#pip3-container` - PIP containers
- `.pip-video` - Video elements (pointer-events: none)
- `.pip-color-picker` - Floating color picker

### ADMIN Video Sources
- Camera source dropdown
- Mirror checkbox
- PIP 1/2/3 settings rows
- Test Camera / Copy Report buttons

### Tag Autocomplete
- `.tag-suggestions` - Suggestion dropdown
- `.tag-suggestion` - Individual suggestion items

### FX Overlay
- `.fx-overlay` - Overlay container
- `.fx-column` - Per-track columns
- `.vfx-knob` - VFX knob canvases

---

## BUG FIXES (8 total)

| Build | Issue | Fix |
|-------|-------|-----|
| V5_4_02a | Scale animation frame-based | Converted to beat-synced |
| V5_4_06a | Dropdown blocking focus mode | Added blur on entry |
| V5_4_13c | Arrow keys not navigating | Fixed handler priority |
| V5_4_13f | Built-in sets renameable | Added protection check |
| V5_4_17f | Ghost cells persisting | Clear on navigation |
| **V5_4_17h** | **Wrong tempo on quick-load** | **Skip trackContainerData if playing** |
| V5_4_24a | Knobs wrong size | Corrected 30→20px |
| V5_4_29b | PIP animation origin | Set per-pip transform-origin |

---

## BREAKING CHANGES

### Per-Mod Envelopes
| Before | After |
|--------|-------|
| `modAQ`, `modBQ`, `modCQ` (global) | Per-mod: `modAQ`, `modBQ`, `modCQ` |
| `modAttack`, `modDecay`, `modSustain`, `modRelease`, `modThreshold` (global) | Per-mod: `mod[A|B|C][Attack|Decay|Sustain|Release|Threshold]` |

**Migration:** Containers without per-mod fields use global defaults.

### Transform Modes Schema
| New Field | Type | Default |
|-----------|------|---------|
| `transformRotationMode` | string | 'stop' |
| `transformRotationSpeed` | number | 1.0 |
| `transformScaleMode` | string | 'stop' |
| `transformScaleSpeed` | number | 1.0 |
| `transformTileMode` | string | 'stop' |
| `transformTileSpeed` | number | 1.0 |

### Project State (.omsp)
| New Field | Default |
|-----------|---------|
| `videoSourcesSettings` | `{ cameraSourceId: 'default', cameraMirror: false }` |
| `pip1Settings` | `{ visible: false, color: '#00d4ff', label: 'PIP 1', shape: 'landscape' }` |
| `pip2Settings` | `{ color: '#00d4ff', shape: 'landscape' }` |
| `pip3Settings` | `{ color: '#00d4ff', shape: 'square' }` |

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Total Iterations | 29 + ~45 sub |
| Lines Added | ~3,500 |
| New Functions | ~35 |
| New State Variables | ~40 |
| Bug Fixes | 8 |
| Schema Additions | 6 (transform modes) + 18 (per-mod envelopes) |

---

## COMPATIBILITY

✅ Backward compatible with V5_4_0, V5_3_0, V5_2_0, V5_1_0
✅ Existing shaders work unchanged
✅ Existing .oms containers load (new fields use defaults)
✅ Per-mod envelopes fall back to global defaults
✅ Transform modes default to 'stop'
✅ Uniform struct unchanged (112 bytes)
⚠️ Audio Card requires Chrome/Edge 110+ for setSinkId
⚠️ PIP Camera requires camera permissions

---

## KNOWN ISSUES

1. **Orientation dropdown sync** - Visual dropdown may not match loaded orientation value (inherited from V5_1_0)
2. **VFX knobs not connected** - Placeholder UI only, no backend processing

---

## PRODUCTION READINESS: ~99%

**100% Complete (V5_5_0):**
- Transform Animation System
- Per-Mod ADSR+T Envelopes
- PIP Camera System (3 PIPs + overlay)
- ORGANIZE Tab UX (Apple Music-style)
- PERFORM Keyboard Workflow
- FX Overlay (UI ready)
- Tag Autocomplete

**Inherited from V5_4_0 (100%):**
- Audio Card & Cue Bus
- MAIN Column Architecture
- DSP Processing
- VFX Pipeline
- Meter System
- Scene Labels & Colors
- Grid Cell Visual Overhaul

**Remaining:**
- MIDI integration
- Orientation dropdown sync
- VFX knob backend connection

---

## PREVIOUS VERSION HISTORY

### V5_4_0 (December 31, 2025)
- Audio Card & Cue Bus System
- MAIN Column Architecture (Master → MAIN rename)
- DSP Output Processing
- VFX Post-Processing Pipeline
- Meter System Overhaul
- Grid Cell Visual Overhaul
- Scene Labels & Colors
- ADMIN Tab Restructure
- ESC Double-Tap Safety

### V5_3_0 (December 27, 2025)
- Real Audio Reactivity (bandpass peak detection)
- Mod Tuning Panel (global ADSR)
- Unified Sets System
- Container Tags
- Proper Peak Metering
- Quantize Control
- Draggable Bar 1

### V5_2_0 (December 24, 2025)
- Audio Focus Mode
- EQ3 System
- DJ FX System
- 5 Cue Points
- Per-Track Audio Processing
- Centralized Focus Mode

### V5_1_0 (December 21, 2025)
- BumpR StickRs System
- Shader Tempo Link
- Transform Focus Mode
- Music Timing Focus Mode

### V5_0_0 (December 18, 2025)
- Transform Controls (Rotation/Scale/Tile/XY/Pivot)
- OmsDropdown Component
- Decoupled Modulator System
- OMS Design System

---

**END OF COMPLETE CHANGELOG**

Version: Beta V5_5_0
Date: January 3, 2026
Total Development: 29 iterations + ~45 sub-iterations
License: GPL-3.0
