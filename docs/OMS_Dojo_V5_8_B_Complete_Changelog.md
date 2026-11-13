# ONEMANSHYO App [Beta V5_8_0] - COMPLETE CHANGELOG

**Version:** Beta V5_8_0  
**Date:** January 22, 2026  
**License:** GPL-3.0  
**Line Count:** 5,594 (V5_7_0) → ~43,800 (V5_8_0)  
**Document Purpose:** Complete development history for V5_8_0 release

---

## V5_8_0 EXECUTIVE SUMMARY

Beta V5_8_0 delivers **Transform Automation System, BumpR StickRs overhaul, Audio Input Routing, Chroma Key for PIP3, and PERFORM tab unification**. This is a major feature release spanning 20 iterations (V5_7_02 through V5_7_21) with ~38,000 lines of new code.

**Key Achievements:**
1. **Transform Automation** - Visual graph editor for automating rotation, scale, tile, position, pivot with curve types and save/load
2. **BumpR StickRs Overhaul** - B+N nested panels, JKL transport, layer control, drag-drop reordering, A/D sequential firing
3. **Audio Input Routing** - External audio input (Loopback/DAW) drives shader modulators for live performance
4. **Chroma Key System** - WebGPU green screen removal for PIP3 camera with render export support
5. **PERFORM Tab Unification** - Unified rotary control system, waveform grid alignment, header customization
6. **Cue Output Fixes** - Auto-disable when same device as main, proper status feedback
7. **Design System Constants** - Centralized timing values (hold-to-edit, double-tap, confirm timeout)

---

## CORTEX CHANGES

### OMS Design System Timing Constants (V5_7_19)

New CORTEX-tier constants for consistent UI interaction timing:

```javascript
const OMS_HOLD_TO_EDIT_MS = 500;    // Hold duration to trigger inline edit
const OMS_DOUBLE_TAP_MS = 300;      // Window for double-tap detection
const OMS_CONFIRM_TIMEOUT_MS = 500; // Timeout for confirmation actions
```

**Used by:** Scene label hold, volume double-tap, JKL double-tap, stop button, ESC×2

### Automation Data in .oms Schema (V5_7_14)

New automationData object added to container schema:

```javascript
automationData: {
    rotation: [{beat, value, curveOut}, ...],
    scale: [{beat, value, curveOut}, ...],
    tile: [{beat, value, curveOut}, ...],
    translateX: [{beat, value, curveOut}, ...],
    translateY: [{beat, value, curveOut}, ...],
    pivotX: [{beat, value, curveOut}, ...],
    pivotY: [{beat, value, curveOut}, ...]
}
```

### Audio Card Input Settings (V5_7_10)

Extended audioCardSettings in .omsp:

```javascript
audioCardSettings: {
    masterOutputId: null,
    cueOutputId: 'none',      // Changed default from 'default'
    cueMix: 0.5,
    inputDeviceId: 'none'     // NEW V5_8_0
}
```

---

## V5_8_0 MAJOR FEATURES

### 1. Transform Automation System (V5_7_12-15)

✅ **Automation Overlay (G Key)**
- Semi-transparent canvas over waveform
- Horizontal center line at 0.5 (neutral)
- Click pass-through when not editing

✅ **Transform Domain (T Key)**
| Key | Control | Color |
|-----|---------|-------|
| 1 | Rotation | Red (#ff6b6b) |
| 2 | Scale | Teal (#4ecdc4) |
| 3 | Tile | Yellow (#ffe66d) |
| 4 | Position | Mint (#95e1d3) |
| 5 | Pivot | Plum (#dda0dd) |

✅ **Node Interaction**
| Action | Result |
|--------|--------|
| Click canvas | Create new node |
| Click node | Delete node |
| Drag node | Move freely |
| Shift+drag | Snap to beat/0.1 |
| Click segment | Add node |
| Drag segment | Move both endpoints |

✅ **Curve Types (Right-Click)**
| Curve | Description |
|-------|-------------|
| Linear | Straight line (default) |
| Sine | S-curve ease in/out |
| Ease In | Slow start, fast end |
| Ease Out | Fast start, slow end |
| Hold | Stay at start value |

✅ **Save/Load Persistence**
- Stored in .oms containers
- `getAutomationValue(dataKey, beat)` for playback interpolation

### 2. BumpR StickRs System (V5_7_02-04, 06)

✅ **Bare B Key Trigger**
- Changed from Shift+B to bare B key
- B also triggers selected slot directly

✅ **B+N Nested Panel Pattern**
- N key opens settings panel (left of B panel)
- Scale, Timing, Animate, Duration dropdowns
- Drop zone for WebM files
- Remove button

✅ **JKL Transport (BumpR Mode)**
| Key | Single Tap | Double Tap |
|-----|------------|------------|
| J | Back 1 second | Back 5 seconds |
| K | Play/Pause | Z-index swap |
| L | Forward 1 second | Forward 5 seconds |

✅ **PIP/BumpR Layer Control**
| Layer | Z-Index |
|-------|---------|
| B slideout panel | 260 |
| N settings panel | 259 |
| BumpR video (playing) | 250 |
| PIP2/PIP3 | 200 |
| BumpR video (stopped) | 100 |

✅ **Drag-Drop Reordering**
- Drag filled slots to reorder
- Swaps all 5 arrays (slots, timing, scale, animate, duration)

✅ **A/D Sequential Firing (CREATE)**
- D = next slot + fire (wraps 10→1)
- A = previous slot + fire (wraps 1→10)

### 3. Audio Input Routing (V5_7_10)

✅ **ADMIN → Audio Card**
- INPUT DEVICES section with Scan Devices button
- INPUT ROUTING dropdown (None, System Default, specific device)
- Input Level meter (80×12 horizontal)
- Status helper: "✓ Input active → Modulators"

✅ **CREATE Header Input Knob**
- Green 24×24 rotary knob (I key)
- Drag/Shift+drag for coarse/fine
- Tap to toggle mute
- 4×24 vertical level meter

✅ **Signal Flow**
```
External Audio (Ableton/DAW)
    ↓
Loopback Virtual Device
    ↓
getUserMedia → audioCardInputSource
    ↓
audioCardInputGainNode → audioCardInputAnalyser
    ↓
modA/B/C Highpass Filters → Shader Modulators
```

### 4. Chroma Key System (V5_7_06-08, 18)

✅ **WebGPU Implementation**
- Separate WebGPU device for chroma processing
- Green screen removal for PIP3 camera

✅ **UI Controls (ADMIN → PIP3 Settings)**
- Toggle: Chroma ON/OFF
- Color picker (default #00ff00)
- Tolerance slider (0-1)
- Softness slider (0-1)
- Spill slider (0-1)

✅ **Render Export Fix (V5_7_18)**
- Keep RAF running during render (don't pause)
- Simple bitmap grab from live chromaKeyCanvas
- Removed complex renderChromaSyncForExport()

### 5. Cue Output Fixes (V5_7_05)

✅ **Auto-Disable Logic**
Cue disabled when:
- Cue Output = "None"
- Cue Output = same device as Main
- Both = "default"

✅ **Status Helper Text**
| State | Message | Color |
|-------|---------|-------|
| None selected | "Select different output..." | Grey |
| Same as Main | "⚠ Cue disabled" | Orange |
| Different | "✓ Cue active" | Cyan |

✅ **Mix Knob Disabled State**
- Dims to 30% opacity when cue disabled
- Hover: "Cue requires separate output device"

### 6. PERFORM Tab Overhaul (V5_7_19-21)

✅ **Unified Rotary Control System**
| Interaction | Action |
|-------------|--------|
| Hover | Message bar help |
| Drag | Coarse (0.01/px) |
| Shift+Drag | Fine (0.002/px) |
| Single-click | Keyboard mode (cyan ring) |
| Double-click | Toggle kill (magenta) |
| L/R arrows | ±5% |
| U/D arrows | ±1% |
| Enter×2 | Toggle kill |

✅ **Track/MAIN Selection**
| Key | Selection |
|-----|-----------|
| 1-4 | Track 1-4 (bare keys) |
| 5 | MAIN |
| 0 | Deselect |

✅ **Arrow Key Control**
| Keys | Audio (no mod) | Visual (Cmd+) |
|------|----------------|---------------|
| L/R | volume ±5% | opacity ±5% |
| U/D | volume ±1% | opacity ±1% |
| Enter×2 | kill toggle | kill toggle |

✅ **MAIN Header Features**
- Selection highlight (orange border)
- Hold-to-rename (500ms)
- Right-click color picker
- Persists: mainLabel, mainColor

✅ **Waveform Grid Alignment Fix (V5_7_20)**
- Dynamic playheadX (was hardcoded 387.5px)
- Correct BPM fallback (120, was 128)
- Process non-playing tracks via gridState.cells

✅ **Stop All Button**
- Double-click required (matches ESC×2)
- Primed state visual feedback
- ESC×1 primes button visually

✅ **Master Tempo Click-to-Activate**
- Click knob for arrow key control
- ←/→ = ±0.5 BPM, ↑/↓ = ±0.1 BPM
- Orange outline when active

---

## ITERATION DETAILS

### V5_7_02 - BumpR N Panel System
- Bare B key trigger
- ADMIN column headers (FILE | SCALE | TIMING | ANIMATE | DURATION)
- N panel with settings dropdowns
- PERFORM tab support with getBumprElements()

### V5_7_03 - BumpR Video Enhancements
- JKL transport controls
- PIP/BumpR layer control (z-index hierarchy)
- K double-tap for z-index swap
- BumpR slot persistence race condition fix

### V5_7_04 - BumpR Drag-Drop & Clipping
- Drag-drop slot reordering
- Video overlay clipping fix (canvasWrapper, overflow:hidden)

### V5_7_05 - Cue Output Fix
- Auto-disable cue when same device
- Status helper text
- Mix knob disabled state
- OmsDropdown styling for Audio Card

### V5_7_06 - Render Pipeline & A/D Sequential
- Render z-index respects K toggle state
- A/D sequential firing for BumpR
- A key conflict fix (stopImmediatePropagation)

### V5_7_07 - Chroma Render Debug
- Debug build for chroma key render issue
- Diagnostic logging added

### V5_7_08 - Consolidated Improvements
- Render stop-and-save (X saves partial)
- Chroma format fix (rgba8unorm → bgra8unorm)
- ADMIN Shader Controls section
- Default Render Duration setting
- Number input spinner hide CSS
- Header PERFORM controls visibility

### V5_7_09 - ORGANIZE Info Drawer
- Audio file rename utility (propagates to all containers)
- Keyboard navigation fixes (stopPropagation)
- Loop field removed (belongs in CREATE)

### V5_7_10 - Audio Input Routing
- ADMIN Audio Card input section
- CREATE header Input knob (green)
- Meter mute consistency fix

### V5_7_11 - Orientation Dropdown Fix
- Fixed dropdown not syncing on container load
- Pass false to setValue during init

### V5_7_12 - Automation Graph Foundation
- G key automation overlay
- T key transform domain
- Selector tiles (1-5)
- Default breakpoints at 0.5

### V5_7_13 - Automation Controls
- Node click/drag interaction
- Shift+drag grid snap
- Delete key removes node
- Audio navigation (arrow keys, Fn+Home/End)

### V5_7_14 - Automation System Complete
- Curve types (Linear, Sine, Ease In/Out, Hold)
- Right-click segment menu
- Save/load to .oms containers
- Hover feedback (grow +4px, brighten)

### V5_7_15 - Automation Polish
- Cue flags tied to N panel visibility
- Removed automationOverlayVisible from project state
- Click pass-through (.editing class)

### V5_7_16 - Audio Section UI
- Module wrapper border
- Header gap fix (.no-margin-top)
- Label removal from controls strip
- Fn+End keyboard shortcut
- 8px vertical compression

### V5_7_17 - EQ/FX & Transport
- Double-click kill pattern for EQ/FX
- Panel focus activation fix
- Transport row hover messages

### V5_7_18 - Chroma Render Fix
- Keep RAF running during render
- Simple bitmap grab from live canvas

### V5_7_19 - PERFORM Tab Cleanup
- Song info row tab visibility
- Track waveforms full width
- Stop All button double-click
- ESC×2 visual sync
- Header greeting hold-to-edit
- Master tempo click-to-activate

### V5_7_20 - Waveform Grid Alignment
- Dynamic playheadX calculation
- BPM fallback consistency
- Non-playing track handling

### V5_7_21 - Unified Rotary System
- MAIN column rotary pattern
- Unified track/MAIN arrow keys
- MAIN header features

---

## NEW STATE VARIABLES

### BumpR System
```javascript
bumprSettingsPanelOpen      // N panel visibility
bumprLastKTapTime           // K double-tap detection
bumprDragSourceIndex        // Drag-drop source slot
```

### Automation System
```javascript
automationOverlayVisible    // G toggle
automationDomain            // null or 'transform'
selectedAutomationControl   // 'rotation', 'scale', etc.
automationData              // {rotation:[], scale:[], ...}
automationHoverNodeIndex    // -1 or node index
automationHoverSegmentIndex // -1 or segment index
cueFlagsVisible             // Tied to N panel visibility
```

### Audio Input
```javascript
audioCardSettings.inputDeviceId   // 'none', 'default', or deviceId
audioCardInputDevices[]           // Cached input device list
audioCardInputStream              // MediaStream
audioCardInputSource              // MediaStreamSource
audioCardInputGainNode            // GainNode
audioCardInputAnalyser            // AnalyserNode
inputMuted                        // Mute state
inputVolume                       // Level 0-1
```

### Chroma Key
```javascript
chromaKeyEnabled
chromaKeyColor              // RGB 0-1
chromaKeyTolerance          // 0-1
chromaKeySoftness           // 0-1
chromaKeySpill              // 0-1
chromaKeyDevice             // WebGPU device
chromaKeyPipeline           // Render pipeline
chromaKeyContext            // Canvas context
```

### PERFORM Tab
```javascript
performMainVolumeKilled     // MAIN audio kill state
performMainOpacityKilled    // MAIN visual kill state
performKnobFocus            // Which MAIN knob has keyboard focus
performTrackVolumeKilled[4] // Per-track audio kill states
performTrackOpacityKilled[4]// Per-track visual kill states
performTrackEnterTime       // Double-tap timing
mainLabel                   // MAIN header label text
mainColor                   // MAIN header color
```

---

## NEW FUNCTIONS

### BumpR
| Function | Purpose |
|----------|---------|
| `toggleBumprSettingsPanel()` | Toggle N panel |
| `updateBumprSettingsPanel()` | Update for selected slot |
| `closeBumprSettingsPanel()` | Close N panel |
| `getBumprElements()` | Return DOM elements per tab |
| `swapBumprSlots(a, b)` | Swap all 5 arrays |
| `handleBumprDrag*()` | Drag event handlers |

### Automation
| Function | Purpose |
|----------|---------|
| `toggleAutomationOverlay()` | G key handler |
| `selectAutomationControl()` | 1-5 key handler |
| `drawAutomationLines()` | Render curves + nodes |
| `beatToPixelX()` | CORTEX: beat → X position |
| `getAutomationValue()` | Interpolated playback value |
| `applyCurveFunction()` | Curve type math |
| `handleAutomationMouse*()` | Mouse event handlers |
| `handleAutomationContextMenu()` | Right-click menu |

### Audio Input
| Function | Purpose |
|----------|---------|
| `setAudioCardInput()` | Set input device |
| `startAudioCardInput()` | Start stream |
| `stopAudioCardInput()` | Stop stream |
| `initCreateInputKnob()` | Initialize green knob |
| `drawCreateInputKnob()` | Draw knob |
| `updateInputGainValue()` | Apply volume/mute |

### Cue Output
| Function | Purpose |
|----------|---------|
| `getCueStatusHelperHTML()` | Status text HTML |
| `updateCueStatusHelper()` | Update display |
| `updateMixKnobState()` | Enable/disable |
| `isCueModeEnabled()` | Check cue state |

### ORGANIZE
| Function | Purpose |
|----------|---------|
| `saveInfoDrawerAudioName()` | Rename + propagate |

### PERFORM
| Function | Purpose |
|----------|---------|
| `executeStopAll()` | Global stop (was scoped) |
| `initMainHeader()` | Setup MAIN header |
| `updateMainHeaderDisplay()` | Update label/color |
| `showMainColorPicker()` | Color picker popup |
| `startMainLabelEdit()` | Inline label edit |
| `jumpToEnd()` | Fn+End handler |

---

## BUG FIXES

| Version | Issue | Fix |
|---------|-------|-----|
| V5_7_03 | BumpR slot persistence race | `false` param to setValue during init |
| V5_7_03 | B/N panels behind video | Z-indexes 110→260, 109→259 |
| V5_7_04 | WebM bleeding over canvas | canvasWrapper + overflow:hidden |
| V5_7_05 | Audio doubling from cue | Auto-disable when same device |
| V5_7_06 | A key closing BumpR panel | bumprModeActive check + stopImmediatePropagation |
| V5_7_08 | Chroma purple/green artifacts | rgba8unorm → bgra8unorm |
| V5_7_08 | audioBuffer reference error | Changed to audioLoaded |
| V5_7_09 | Enter closing info drawer | stopPropagation added |
| V5_7_11 | Orientation dropdown not syncing | false to setValue during init |
| V5_7_15 | Cue 1 blocking Audio Start | Cue flags tied to N panel |
| V5_7_17 | EQ/FX single-click kill | Double-click pattern |
| V5_7_18 | PIP3 chroma not in render | Keep RAF running |
| V5_7_19 | audioInfoRow re-showing | Fixed RAF loop |
| V5_7_19 | ESC×2 scope issue | executeStopAll to global |
| V5_7_20 | Track grid misalignment | Dynamic playheadX |
| V5_7_21 | Tempo arrows moving library | stopPropagation |

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Total Iterations | 20 (V5_7_02 through V5_7_21) |
| Date Range | January 16-22, 2026 |
| Lines Added | ~38,200 |
| New Functions | 35+ |
| New State Variables | 25+ |
| Bug Fixes | 16 |
| Schema Additions | 2 (.oms automationData, .omsp input/MAIN settings) |
| CORTEX Changes | 1 (timing constants) |

---

## COMPATIBILITY

✅ Backward compatible (no breaking changes)  
✅ Existing .oms files load correctly  
✅ Existing .omsp projects load correctly  
✅ New fields have sensible defaults  
✅ automationData defaults to empty arrays  
✅ inputDeviceId defaults to 'none'  
✅ mainLabel defaults to 'MAIN'  
✅ mainColor defaults to null  

---

## KNOWN ISSUES

1. **Unimplemented pool effects** - Blur, Sharpen, Invert (VFX) and Reverb, Phaser, etc. (FX) show in dropdowns but have no effect (inherited)
2. **Chroma key tuning** - May require adjustment for different lighting conditions
3. **Automation playback** - Not yet integrated into render pipeline (preview only)

---

## PRODUCTION READINESS: ~98%

**100% Complete (V5_8_0):**
- Transform Automation System
- BumpR StickRs Overhaul
- Audio Input Routing
- Chroma Key System
- Cue Output Fixes
- PERFORM Tab Unification
- Design System Constants

**Inherited Complete (from V5_7_0):**
- MP4 Native Render
- JKL Transport Controls
- PIP System Overhaul
- loopLength Uniform

**Remaining:**
- Pool effects completion (7 FX, 3 VFX)
- MIDI integration
- Automation render integration

---

## PRIOR VERSION SUMMARY

### V5_7_0 (January 16, 2026)
- MP4 native rendering with Format dropdown
- JKL transport for Audio Focus
- PIP system overhaul with P+Number chords
- BumpR persistence fixes
- Render improvements (cancel, loop respect, BumpR capture)
- loopLength uniform for shader loops

### V5_6_0 (January 12, 2026)
- Per-track VFX system
- Unified Mic System
- Transform keyboard improvements
- Audio FX/VFX pools
- shaderModAFFT → shaderModAAudio rename

---

**END OF COMPLETE CHANGELOG**
