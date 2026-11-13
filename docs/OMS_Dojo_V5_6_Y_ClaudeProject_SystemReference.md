# ONEMANSHYO App [Beta V5_6_0] - SYSTEM REFERENCE

**Version:** Beta V5_6_0  
**Date:** January 11, 2026  
**License:** GPL-3.0  
**Line Count:** 5,290 (V5_5_0) → 5,542 (V5_6_0)  
**Document Purpose:** Complete technical implementation reference for Claude AI development

---

## TABLE OF CONTENTS

1. [Leaf Nodes](#leaf-nodes)
2. [Organizational Architecture](#organizational-architecture)
3. [File Format Specifications](#file-format-specifications)
4. [Core Systems Architecture](#core-systems-architecture)
5. [DOM Elements & Controls](#dom-elements--controls)
6. [Data Structures](#data-structures)
7. [Data Flow](#data-flow)
8. [Function Reference](#function-reference)
9. [Naming Conventions](#naming-conventions)

---

## LEAF NODES

**Purpose:** Dependency hierarchy defining what can and cannot be modified.

---

### Tier 1: Cortex

Foundational elements. Everything traces back to these. Change = cascade failure.

**File System Handles (V4.3.0_A):**
- `applicationFolderHandle` - Root app folder (user selects via picker)
- `libraryFolderHandle` - /Library/ subfolder handle
- `bumprStickrsFolderHandle` - /BumpRStickRs/ subfolder handle (V5_1_0)
- `setsFolderHandle` - /Sets/ subfolder handle (V5_3_0)

**Master Clock / Seconds:**
- `audioContext` - Web Audio API instance
- `audioContext.currentTime` - browser clock, returns seconds
- `trackMasterClockStart` - session start timestamp, stored in seconds

**Master Clock / Musical Time:**
- `gridState.masterTempo` - BPM value (conversion rate)
- `trackAudioMasterClock()` - THE FUNCTION that converts seconds → beats
- `masterBeatNow()` - calls trackAudioMasterClock(), returns current beat
- `masterBarNow()` - calls masterBeatNow() / 4, returns current bar

**V5_4_0 Terminology Clarification:**
```
"Master" = Timing/Grid systems (Master Clock, Master Tempo, Master Grid)
"MAIN"   = Output bus (MAIN Column, MAIN meters, MAIN volume, trackMain*)
```

**Conversion Chain:**
```
audioContext.currentTime        → seconds (browser provides)
trackMasterClockStart           → seconds (stored on first launch)
gridState.masterTempo           → BPM (user sets)
trackAudioMasterClock()         → DOES THE MATH, outputs beats
masterBeatNow()                 → returns beats
masterBarNow()                  → returns bars
```

**The Math (inside trackAudioMasterClock):**
```javascript
return (audioContext.currentTime - trackMasterClockStart) * (gridState.masterTempo / 60);
```

**Pixel Formula (Derived from Musical Time):**
- `BASE_PIXELS_PER_SECOND = 6.25`
- `getPixelsPerBeat()` = BASE_PIXELS_PER_SECOND × (60 / masterTempo) / zoom
- `getPixelsPerBar()` = getPixelsPerBeat() × 4
- Golden Rule: Same formula everywhere

**Coordinate System:**
- `playheadX = container.clientWidth / 2`
- Container CSS: `position: relative; overflow: hidden` (NO flex)
- Canvas CSS: `position: absolute; top: 0; left: 0`
- Playhead: Canvas-drawn at playheadX

**Transform Formula:**
- `translateX = playheadX - bar1X - scrollPx`

**Bar1X Calculation:**
- `bar1X = (audioStartTime × originalBPM / 60) × pixelsPerBeat`

**Visual Foundation:**
- **The Shader Contract** (CRITICAL - all shaders must follow):
  - Uniform struct (V5_0_0): 112 bytes, 28 floats - see full definition below
  - Fragment signature: `@fragment fn fs_main(@builtin(position) pos: vec4<f32>) -> @location(0) vec4<f32>`
  - NO struct/binding in user code (app injects automatically)
  - Use: `uniforms.time`, `uniforms.modA`, etc.
  - Must return: `vec4<f32>(r, g, b, a)` with alpha = 1.0
- **WebGPU device/context** (obtained once, never recreated)
- **Format**: 'bgra8unorm' (immutable)
- **calculateMod()** function - centralized modulator calculator

**V5_0_0 CORTEX Constants (Shader Contract Consolidation):**

V4_9_15 consolidated the shader contract into 3 constants. Before this, the uniform contract was scattered across 6+ locations. Now: single source of truth.

```javascript
// CORTEX CONSTANT 1: WGSL struct definition (injected into every shader)
// V5_6_0: shaderModAFFT renamed to shaderModAAudio (and B/C) - reflects actual pipeline
const SHADER_UNIFORM_STRUCT = `
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    beat: f32,
    resolutionX: f32,
    resolutionY: f32,
    shaderModAWave: f32,
    shaderModAGain: f32,
    shaderModAAudio: f32,
    shaderModBWave: f32,
    shaderModBGain: f32,
    shaderModBAudio: f32,
    shaderModCWave: f32,
    shaderModCGain: f32,
    shaderModCAudio: f32,
    transformRotation: f32,
    transformScaleX: f32,
    transformScaleY: f32,
    transformTranslateX: f32,
    transformTranslateY: f32,
    transformPivotX: f32,
    transformPivotY: f32,
    transformTile: f32,
    padding1: f32,
    padding2: f32,
    padding3: f32,
}
@group(0) @binding(0) var<uniform> uniforms: Uniforms;
`;

// CORTEX CONSTANT 2: Buffer size (must match struct)
const UNIFORM_BUFFER_SIZE = 112;  // 28 floats × 4 bytes

// CORTEX CONSTANT 3: Factory function (creates Float32Array for GPU)
function createUniformData(p) {
    return new Float32Array([
        p.time, p.tempo, p.modA, p.modB, p.modC, p.beat,
        p.resolutionX, p.resolutionY,
        p.shaderModAWave, p.shaderModAGain, p.shaderModAAudio,
        p.shaderModBWave, p.shaderModBGain, p.shaderModBAudio,
        p.shaderModCWave, p.shaderModCGain, p.shaderModCAudio,
        p.transformRotation,
        p.transformScaleX, p.transformScaleY,
        p.transformTranslateX, p.transformTranslateY,
        p.transformPivotX, p.transformPivotY,
        p.transformTile,
        0, 0, 0  // padding
    ]);
}
```

**V5_6_0 CORTEX Rename:** `shaderModAFFT` → `shaderModAAudio`
- The "FFT" name was misleading - actual pipeline uses bandpass → peak detection → ADSR
- Renamed to "Audio" to accurately reflect what it measures
- Same rename for B and C modulators
- **Breaking change:** Existing shaders using `uniforms.shaderModAFFT` must update

**Adding a New Uniform (4 steps):**
1. Add field to SHADER_UNIFORM_STRUCT (maintain GPU alignment)
2. Update UNIFORM_BUFFER_SIZE if needed
3. Add parameter to createUniformData() factory
4. Pass value in render() call

**Why This Matters:**
- Before V4_9_15: 6+ scattered locations, ~20,000 line hunts
- After V4_9_15: 3 constants, 4-step process
- Thumbnail buffer bug fixed (was 96, needed 112)

**V5_0_0 Uniform Struct (28 floats = 112 bytes), V5_6_0 Rename:**

| Index | Uniform | Type | Range | Purpose |
|-------|---------|------|-------|---------|
| 0 | time | f32 | 0+ | Seconds since start |
| 1 | tempo | f32 | 60-200 | BPM for timing |
| 2 | modA | f32 | 0-1 | Combined bass (legacy) |
| 3 | modB | f32 | 0-1 | Combined mids (legacy) |
| 4 | modC | f32 | 0-1 | Combined treble (legacy) |
| 5 | beat | f32 | 0+ | Tempo-synced beat position |
| 6 | resolutionX | f32 | pixels | Canvas width |
| 7 | resolutionY | f32 | pixels | Canvas height |
| 8 | shaderModAWave | f32 | 0-1 | Mod A waveform output |
| 9 | shaderModAGain | f32 | 0-1 | Mod A gain value |
| 10 | shaderModAAudio | f32 | 0-1 | Mod A audio level (V5_6_0 rename from FFT) |
| 11 | shaderModBWave | f32 | 0-1 | Mod B waveform output |
| 12 | shaderModBGain | f32 | 0-1 | Mod B gain value |
| 13 | shaderModBAudio | f32 | 0-1 | Mod B audio level (V5_6_0 rename from FFT) |
| 14 | shaderModCWave | f32 | 0-1 | Mod C waveform output |
| 15 | shaderModCGain | f32 | 0-1 | Mod C gain value |
| 16 | shaderModCAudio | f32 | 0-1 | Mod C audio level (V5_6_0 rename from FFT) |
| 17 | transformRotation | f32 | radians | Rotation angle (V5_0_0) |
| 18 | transformScaleX | f32 | 0-4 | Scale X (V5_0_0) |
| 19 | transformScaleY | f32 | 0-4 | Scale Y (V5_0_0) |
| 20 | transformTranslateX | f32 | -1 to 1 | Position X (V5_0_0) |
| 21 | transformTranslateY | f32 | -1 to 1 | Position Y (V5_0_0) |
| 22 | transformPivotX | f32 | -1 to 1 | Pivot X (V5_0_0) |
| 23 | transformPivotY | f32 | -1 to 1 | Pivot Y (V5_0_0) |
| 24 | transformTile | f32 | 1-8 | Tile count (V5_0_0) |
| 25-27 | padding | f32 | - | GPU alignment |


**Modulator Architecture (V4_8_0):**
- **Decoupled system** - 9 independent values instead of 3 combined
- **Legacy support** - modA/B/C still work (Wave × Gain × FFT)
- **Independent access** - shaders can use Wave, Gain, FFT separately
- **Works without audio** - Wave and Gain function, FFT = 0

**Engines:**
- WebGPU device/context
- Uniform struct (112 bytes, 28 floats)
- `calculateMod()` function
- .oms container schema

**Message Bar System (V4_7_0, V4_8_0, V5_1_0):**
- `messageBar` - DOM element ID (single message display)
- `showMessage(text, type, onClick)` - THE FUNCTION for all user feedback
- Types: `'info'` (cyan), `'state'` (orange), `'error'` (magenta)
- V4_8_0: Optional `onClick` callback makes message clickable (cursor: pointer)
- V5_1_0: CORTEX hover patterns for Container Output section
- CSS class: `.message-bar` + type modifier
- Single entry point - ALL transient user communication flows through this

**Message Bar Contract:**
```javascript
// CORTEX - the ONLY way to communicate transient info to user
showMessage(text, type, onClick)  // onClick optional (V4_8_0)

// Types:
'info'   // Cyan #00d4ff - help text, shortcuts
'state'  // Orange #ffa500 - current values, confirmations  
'error'  // Magenta #ff006e - failures, problems

// Examples:
showMessage('Audio Volume (↑/↓)', 'info')    // hover help
showMessage('Track 1 Shader: 80%', 'state')  // action result
showMessage('Container not found', 'error')  // failure

// V4_8_0: Clickable message
showMessage('Set application folder', 'state', () => setupApplicationFolder())

// V5_1_0: Container Output hover messages
showMessage('Container = Shader + Audio + Settings', 'info')  // header hover
showMessage('Save as NEW container to Library', 'info')       // Save hover
showMessage('Update loaded container (keeps name)', 'info')   // Update hover
```

**Why Message Bar is CORTEX:**
- Every module depends on it for user communication
- Change message bar = cascade to ALL user feedback
- No module should create its own feedback mechanism
- Enables future: internationalization, logging, history

---

### Tier 2: Nodes

Built on cortex. Other things depend on these. Trace up AND down before touching.

**Library System (V4.3.0_A):**
- `libraryContainersCache[]` - Full container data from folder scan
- `librarySortBy` - 'name', 'bpm', or 'audio' (V4_4_0: added 'audio')
- `librarySortDir` - 'asc' or 'desc'
- `sanitizeFilename()` - Filesystem-safe conversion
- `refreshLibraryList()` - Scans folder, populates cache

**Sets System (V4.4.0):**
- `selectedLibrarySetId` - Current set filter ('all', 'presets')
- `selectedLiveSetId` - Synced with Library tab selection
- `userPresetsCache` - User presets from /Presets/ folder
- `isExternalFileDrop()` - Distinguishes external file drops from internal drags
- `refreshLibraryListRight()` - Renders right panel only

**Track System:**
- `trackBar1Positions[]` - stores bar1X per track
- `trackAudioMasterClock()` - seconds → beats conversion
- Quantization math (next bar boundary)
- `playbackRate` calculation
- Timeline arrays (shaderModATimeline)
- Track state arrays (trackAudioPlaying, trackShaderOpacity)
- GainNode routing
- Compositing order (z-index)

**Cue Points System (V5_2_0):**
- `audioCuePoints[]` - 5 beat positions per container (default: [0, 64, 128, 192, 256])
- Stored relative to audioStartTime
- Visual flags (orange #ffa500) on waveform
- Draggable with bar snap (4 beats)
- Click-to-jump functionality
- V5_2_0: Expanded from 3 to 5 cues

**EQ3 System (V5_2_0):**
- `createEQHigh`, `createEQMid`, `createEQLow` - BiquadFilterNode instances (CREATE tab)
- `trackEQNodes[track][band]` - Per-track EQ filters (PERFORM tab)
- High: 2500Hz highshelf, Mid: 1000Hz peaking Q=0.7, Low: 250Hz lowshelf
- Ableton-style mapping: 0=kill, 0.25=-12dB, 0.5=-6dB, 0.75=0dB (unity), 1.0=+6dB

**DJ FX System (V5_2_0):**
- `createFXWashout`, `createFXDubby`, `createFXBreak` - FX nodes (CREATE tab)
- `trackFXNodes[track][slot]` - Per-track FX chains (PERFORM tab)
- Washout: Highpass sweep 20Hz→2kHz
- Dubby: DelayNode 375ms with feedback
- Break: playbackRate 1.0→0.1

**Mod Tuning System (V5_3_0, V5_5_0 Per-Mod):**
- `modTuningPanelOpen` - Panel visibility state
- `modTuningSelectedMod` - Which mod panel shows ('A', 'B', 'C') - syncs to Reactivity row

**Per-Mod Envelope Parameters (V5_5_0 - 18 total, 6 per mod):**
```javascript
// Mod A (Bass) - optimized for kicks
modAQ: 0.70,           // Q (0.5-10)
modAAttack: 5,         // Attack (0-100ms)
modADecay: 20,         // Decay (0-200ms)
modASustain: 70,       // Sustain (0-100%)
modARelease: 50,       // Release (0-500ms)
modAThreshold: 10,     // Threshold (0-100%)

// Mod B (Mids) - optimized for snares/claps
modBQ: 0.70,
modBAttack: 10,
modBDecay: 30,
modBSustain: 80,
modBRelease: 100,
modBThreshold: 5,

// Mod C (Highs) - optimized for hats/cymbals
modCQ: 0.70,
modCAttack: 2,
modCDecay: 15,
modCSustain: 60,
modCRelease: 80,
modCThreshold: 5,
```

**V5_5_0 Changes from V5_4_0:**
- ADSR was global (shared by all mods) → now per-mod (independent)
- Threshold range expanded: 0-20% → 0-100%
- N-panel syncs to selected Reactivity row (↑/↓ navigation)

**Audio Analysis System (V5_3_0 Rewrite):**
- `createBandpassFilter()` - Bandpass filter per modulator
- `peakDetector()` - Peak detection with ADSR envelope
- Replaces flat FFT bin averaging with true dynamics
- Result: kick hit → 0.72, silence → 0.01 (was constant ~0.02)

**Unified Sets System (V5_3_0):**
- `setsFolderHandle` - /Sets/ subfolder handle
- `setsCache[]` - Combined manual + smart sets
- `setType` - 'manual' or 'smart' per set
- `smartSetQuery` - Search query for smart sets
- Storage: /Sets/[name].oms-set

**Audio Card & Cue Bus System (V5_4_0):**
- `audioCardMasterOutputId` - Selected master device ID
- `audioCardCueOutputId` - Selected cue device ID
- `audioCardCueMix` - Cue/MAIN blend (0-1, default 0.5)
- `trackCues` - Per-track cue state [false, false, false, false]
- `cueGainNode` - Pre-fader tap for cue bus
- `cueBusGainNode` - Cue bus output gain
- `cueMixGainNode` - Cue mix blend control
- `cueDestination` - MediaStreamAudioDestinationNode for cue output

**Signal Chain (V5_4_0):**
```
Track Source
    ↓
Cue Tap (Pre-Fader) ──────────────────→ Cue Bus → Cue Output (Headphones)
    ↓
Track Fader → EQ → FX → Track Gain
    ↓
MAIN Bus → DSP Compressor → MAIN Gain → Master Output (Speakers)
                                ↓
                          Cue Mix Blend → Cue Output
```

**MAIN Output System (V5_4_0):**
- `trackMainShaderOpacity` - MAIN visual opacity (was masterShaderOpacity)
- `trackMainShaderMute` - MAIN visual mute (was masterShaderMute)
- `trackMainVolume` - MAIN audio volume (was masterVolume)
- `trackMainAudioMute` - MAIN audio mute (was masterAudioMute)
- `trackMainGainNode` - MAIN output gain node
- `trackMainAnalyserL` - Left channel analyser
- `trackMainAnalyserR` - Right channel analyser
- `trackMainChannelSplitter` - ChannelSplitterNode for stereo

**DSP Output Processing (V5_4_0):**
- `dspEnabled` - DSP on/off state
- `dspPreset` - Current preset name ('off', 'gentle', 'medium', 'aggressive', 'limiter')
- `dspThreshold` - Compressor threshold (-48 to 0 dB)
- `dspRatio` - Compression ratio (1:1 to 20:1)
- `dspOutput` - Makeup gain (-12 to +12 dB)
- `dspCompressor` - DynamicsCompressorNode
- `dspMakeupGain` - GainNode for output level

**VFX Post-Processing (V5_4_0):**
- `vfxEffect` - Current effect name ('off', 'rgbsplit', 'glitch', etc.)
- `vfxAmount` - Effect intensity (0-1)
- `vfxPipeline` - WebGPU render pipeline for VFX
- `vfxFrameBuffer` - Frame buffer for VFX processing
- `vfxUniformBuffer` - VFX-specific uniforms

**Video Sources & PIP Camera System (V5_5_0):**
- `videoSourcesSettings` - Camera device ID, mirror mode
- `cameraStream` - MediaStream instance (shared across all PIPs)
- `cameraOverlayVisible` - Full-screen overlay state (O key)
- `pip1Visible` - PIP 1 slideout state (PERFORM tab, P key)
- `pip2Visible` - PIP 2 corner state (CREATE/ORG/ADMIN, P key)
- `pip3Visible` - PIP 3 center state (CREATE/ORG/ADMIN, Shift+P)
- `pip1Settings` - {visible, shape, color, label}
- `pip2Settings` - {shape, color}
- `pip3Settings` - {shape, color}
- `pipColorPickerOpen` - Color picker popup state
- `activePIPForColorPicker` - Which PIP is being configured

**PIP Shapes:** 'landscape' (16:9), 'portrait' (9:16), 'square' (1:1), 'circle' (1:1 rounded)
**PIP Colors:** 'none', 'cyan', 'magenta', 'orange', 'yellow', 'green', 'purple', 'pink', 'white'

**Smart Stream Management:**
- Camera starts when first PIP opens
- Camera continues while ANY PIP is visible
- Camera stops only when ALL PIPs close

**Transform Animation System (V5_5_0):**
- `transformRotationMode` - 'stop', 'cw', 'ccw', 'pingpong'
- `transformRotationSpeed` - 0.03 to 256 (exponential scale)
- `transformScaleMode` - 'stop', 'cw', 'ccw', 'pingpong'
- `transformScaleSpeed` - 0.03 to 256
- `transformTileMode` - 'stop', 'cw', 'ccw', 'pingpong'
- `transformTileSpeed` - 0.03 to 256

**Mode Behaviors:**
| Mode | Key | Rotation | Scale | Tile |
|------|-----|----------|-------|------|
| Stop | 0 | Manual only | Manual only | Manual only |
| CW | 1 | Clockwise | Grow | Increase |
| CCW | 2 | Counter-clockwise | Shrink | Decrease |
| PingPong | 3 | Oscillate | Oscillate | Oscillate |

**Speed Control:** Trackpad scroll adjusts speed exponentially (0.03× to 256×)

**FX Overlay System (V5_5_0, V5_6_0 Per-Track):**
- `fxOverlayVisible` - Panel visibility state
- `trackVFXValues[track][slot]` - Per-track VFX values [4 tracks][3 slots] (V5_6_0: functional)
- `trackRGBValues[track][channel]` - Per-track RGB gain [4 tracks][3 channels] (V5_6_0)
- `trackFXValues[track][slot]` - Per-track audio FX values [4 tracks][3 slots]
- `trackVFXKilled[track][slot]` - Kill state per VFX slot
- `trackRGBKilled[track][channel]` - Kill state per RGB channel
- `trackFXKilled[track][slot]` - Kill state per FX slot

**Unified Mic System (V5_6_0):**
- `micEnabled` - Mic input active state
- `micGainNode` - Mic input gain node
- `micAnalyser` - Mic analyser node
- `micStream` - MediaStream from getUserMedia
- `micDeviceId` - Selected input device ID
- `micMuted` - Mic mute state
- `micVolume` - Mic input level (0-100)

**Mic System State Logic:**
```javascript
// Audio source priority
if (audioBuffer) {
    // File audio takes precedence
    source = audioBuffer;
} else if (micEnabled && micStream) {
    // Mic input when no file
    source = micStream;
} else {
    // Wave × Gain only (no audio analysis)
    source = null;
}

// Freq knobs enabled when:
freqKnobsEnabled = (audioBuffer !== null) || (micEnabled && micStream);
```

**Effect Pool System (V5_6_0):**
- `vfxPool` - Array of 10 visual effects
- `fxPool` - Array of 10 audio effects
- `vfxAssignments` - {A: effectName, B: effectName, C: effectName}
- `fxAssignments` - {R: effectName, D: effectName, W: effectName}

**VFX Pool (10 effects):**
```javascript
['RGB Split', 'Glitch', 'Pixelate', 'Scanlines', 'Feedback',
 'Mirror', 'Kaleidoscope', 'VHS', 'Shake', 'Off']
```

**FX Pool (10 effects):**
```javascript
['Washout', 'Dubby', 'Break', 'Reverb', 'Flanger',
 'Phaser', 'Bitcrush', 'Filter', 'Delay', 'Off']
```

**Per-Track RGB Control (V5_6_0):**
- `trackRGBValues[track][0]` - Red gain (0-1, 0.75=unity)
- `trackRGBValues[track][1]` - Green gain (0-1, 0.75=unity)
- `trackRGBValues[track][2]` - Blue gain (0-1, 0.75=unity)
- Unity at 0.75 (not 0.5) for boost headroom
- Click-to-kill toggles channel on/off

---

### Tier 3: Leafs

End of chain. Nothing depends on these. Safe to modify.

**Library UI:**
- Sort arrow indicators (▼/▲)
- Library card styling
- Column header text/colors
- Tab colors (cyan/magenta/yellow by function) - V4.4.0
- Library two-panel layout (Sets left, Containers right) - V4.4.0
- Shader thumbnails (32x18 on cards) - V4.4.0
- Audio column Y/N indicator - V4.4.0
- Loop bracket colors (cyan vs magenta) - V4.4.0

**Visual:**
- Visibility states (masterGridVisible, trackGridVisible, waveformVisible)
- Colors (waveform, grid lines, playhead)
- Knob/button styling
- Cue point flag colors (orange #ffa500) - V5_1_0

**Controls:**
- Keyboard shortcut mappings
- Header text
- Tab labels
- Font choices

**BumpR StickRs (V5_1_0):**
- Slot thumbnails and styling
- Animation CSS keyframes (15 types)
- Dropdown styling (Scale/Timing/Animate/Duration)
- Slideout panel layout

**Scene Labels & Colors (V5_4_0):**
- `sceneLabels[]` - 8 label strings (up to 128 chars each)
- `sceneColors[]` - 8 color presets (null = default grey)
- Ticker animation CSS/JS
- Color picker popup

**Meter Toggle (V5_4_0):**
- `meterOverlayVisible` - Toggle state
- Split-cube button styling
- Per-track V/A module layout

**FX Overlay (V5_5_0):**
- `fxOverlayVisible` - Panel visibility state
- Orange FX button styling
- 120px slideout panel layout
- Per-track VFX/FX modules (4 tracks × 6 knobs)
- 20×20 knob standardization
- Purple arcs (VFX), Orange arcs (FX)

**PIP Camera Styling (V5_5_0):**
- PIP container positioning (corner, center, slideout)
- Shape CSS (landscape/portrait/square/circle)
- Border colors (9 options)
- Genie zoom animation CSS
- Color picker popup styling

---

## ORGANIZATIONAL ARCHITECTURE

**Purpose:** Understanding how the codebase is organized by domain, function, and file type.

---

### 1. FUNCTIONS (Organized by Operational Domain)

**GENERIC (No Domain Prefix):**
- `masterBeatNow()` - current beat in master timeline
- `masterBarNow()` - current bar in master timeline  
- `masterSecondsPerBar()` - seconds per bar at current tempo
- `secondsToTimeString()` - convert seconds to MM:SS.mmm
- `timeStringToSeconds()` - convert MM:SS.mmm to seconds
- `beatsToBarBeat()` - convert beat number to bar.beat format
- `barBeatToBeats()` - convert bar.beat to beat number
- `getCurrentControlState()` - get current control state from UI
- `applyControlState()` - apply control state to UI fields
- `getCurrentProjectState()` - get current LIVE tab state
- `applyProjectState()` - restore LIVE tab from project state
- `calculateMod()` - calculate waveform value (handles ALL waveform types including meta-modes: sine, square, triangle, sawtooth, cycler, chaosmode, madmax, psycho)

**AUDIO DOMAIN (`audio*` prefix):**
- `analyzeAudio()` - FFT analysis (returns {bass, mid, treble})
- `analyzeAudioTimeline()` - GENERIC: capture FFT arrays over time
  - NOTE: Function is generic (audio domain)
  - Storage fields are domain-specific (shaderModATimeline, etc.)

**SHADER DOMAIN (`shader*` or similar):**
- `compileShader()` - compile WGSL shader code
- `render()` - render single frame
- `startRender()` - start video recording
- `initWebGPU()` - initialize WebGPU
- `toggleShaderTempoLink()` - V5_1_0: Link/unlink shader tempo from audio
- `updateShaderTempoLinkUI()` - V5_1_0: Update link button visual

**TRACK AUDIO DOMAIN (LIVE Tab):**
- `playCellAudio()` - start playing track audio
- `initAudioRouting()` - initialize Web Audio API routing

**WAVEFORM DOMAIN:**
- `drawWaveformFromAudioBuffer()` - draw blue amplitude bars + cue flags (V5_1_0)
- `drawBeatGridOnWaveform()` - draw cyan/magenta beat grid
- `positionTrackAudioWaveformAtPlayhead()` - position Bar 1 at playhead
- `startAudioWaveformScroll()` - start scroll animation
- `drawProgress()` - draw playback progress + cue flags (V5_1_0)

**LIVE TAB DOMAIN:**
- `initLiveTab()` - initialize entire LIVE tab
- `initLiveMixer()` - initialize mixer controls
- `initLiveShader()` - initialize shader system
- `launchScene()` - launch all tracks in scene
- `drawMasterGrid()` - draw white overlay bars
- `toggleMasterGrid()` - toggle Master Grid visibility

**UNIFIED DROP DOMAIN (V4_4_0):**
- `handleUnifiedDrop()` - Routes all external file drops by type + active tab
- `isExternalFileDrop()` - Distinguishes external files from internal card drags

**BUMPR STICKRS DOMAIN (V5_1_0):**
- `handleBumprSlotDrop()` - Handle WebM drop onto slot
- `generateBumprThumbnail()` - Create 160×90 thumbnail
- `updateBumprSlotUI()` - Update slot visual state
- `removeBumprSlot()` - Clear slot
- `triggerBumprStickr()` - Fire sticker with timing logic
- `playBumprVideo()` - Start video overlay playback
- `stopBumprVideo()` - Stop current playback
- `getBumprDelay()` - Calculate timing delay
- `enterBumprMode()` - Enter Shift+B keyboard mode
- `exitBumprMode()` - Exit keyboard mode

**FOCUS MODE DOMAIN (V5_2_0 Centralized System):**

V5_2_0 introduced a centralized focus mode system using Object.defineProperty for backward compatibility.

**CORTEX Variables:**
- `activeFocusMode` - Current mode: null, 'reactivity', 'transform', 'musicTiming', 'audio', 'bumpr'
- `activeAudioPanel` - Current audio panel: null, 'volume', 'cue', 'eq', 'fx'
- `exitAllFocusModes()` - Central cleanup function, clears all modes and panels

**Mode Variables (backward compatible via defineProperty):**
- `createReactivityFocus` - Reactivity mode (V5_2_0 rename from shaderReactivityFocus)
- `createTransformFocus` - Transform mode (V5_2_0 rename from shaderTransformFocus)
- `createMusicTimingFocus` - Music Timing mode
- `createAudioFocus` - Audio Focus mode (NEW V5_2_0)

**Audio Focus Sub-Panels (V5_2_0):**
- V panel: Volume control
- C panel: Cue points (1-5)
- E panel: EQ3 (H/M/L)
- F panel: FX (Washout/Dubby/Break)

**Gate + Panel Pattern (V5_2_0):**
```javascript
// Enter Audio Focus (gate)
Shift+A → activeFocusMode = 'audio'

// Open sub-panel
V/C/E/F → activeAudioPanel = 'volume'/'cue'/'eq'/'fx'

// Navigate within panel
↑/↓ → row selection, ←/→ → value adjustment

// Exit (nested)
Escape → if activeAudioPanel, close panel; else exit Audio Focus
```

**Tab Isolation (V5_2_0):**
- `isCreateTabActive()` - Helper to check CREATE tab
- `isPerformTabActive()` - Helper to check PERFORM tab
- Keyboard handlers check active tab before processing

**AUDIO CARD DOMAIN (V5_4_0):**
- `initAudioCard()` - Initialize audio card system
- `enumerateAudioDevices()` - Get available output devices
- `setMasterOutput(deviceId)` - Set master output device
- `setCueOutput(deviceId)` - Set cue output device
- `updateCueMix(value)` - Update cue/MAIN blend
- `toggleTrackCue(trackIndex)` - Toggle track cue state
- `testAudioOutput(deviceId, type)` - Play test tone
- `createCueBus()` - Create cue bus routing

**DSP DOMAIN (V5_4_0):**
- `initDSP()` - Initialize DSP chain
- `setDSPPreset(preset)` - Apply preset settings
- `updateDSPThreshold(value)` - Update compressor threshold
- `updateDSPRatio(value)` - Update compression ratio
- `updateDSPOutput(value)` - Update makeup gain
- `getDSPGainReduction()` - Get current GR value
- `drawGRMeter(canvas, dB)` - Draw gain reduction meter

**VFX DOMAIN (V5_4_0):**
- `initVFX()` - Initialize VFX pipeline
- `setVFXEffect(effect)` - Set active effect
- `updateVFXAmount(value)` - Update effect intensity
- `createVFXPipeline(effect)` - Create WebGPU pipeline for effect
- `renderVFXPass()` - Execute VFX render pass
- `getVFXShaderCode(effect)` - Get shader code for effect
- `resetVFXAmount()` - Reset amount to 0 (auto on "Off" select)

**METER DOMAIN (V5_4_0):**
- `toggleMeterOverlay()` - Toggle meter visibility
- `initStereoMetering()` - Initialize stereo analysers
- `updateMainStereoMeters()` - Update MAIN L/R meters
- `updateTrackMeters()` - Update per-track meters
- `drawStereoMeter(canvas, L, R)` - Draw stereo meter bars
- `calculatePeakdB(analyser)` - Get peak level in dBFS

**SCENE LABELS DOMAIN (V5_4_0):**
- `setSceneLabel(sceneIndex, label)` - Set scene label
- `setSceneColor(sceneIndex, color)` - Set scene color
- `startSceneLabelEdit(sceneIndex)` - Begin inline edit
- `saveSceneLabelEdit()` - Save inline edit
- `showColorPicker(sceneIndex, event)` - Show color picker popup
- `hideColorPicker()` - Hide color picker
- `initTickerAnimation(element)` - Initialize ticker scroll

**SAFETY DOMAIN (V5_4_0):**
- `handleEscapeKey(event)` - ESC double-tap handler
- `stopAllWithConfirmation()` - Execute stop all
- `flashStopButton()` - Visual feedback on stop

**VIDEO SOURCES & PIP CAMERA DOMAIN (V5_5_0):**
- `initVideoSources()` - Initialize camera system
- `enumerateCameras()` - Get available video input devices
- `startCameraStream(deviceId)` - Start MediaStream from camera
- `stopCameraStream()` - Stop stream (only when ALL PIPs closed)
- `toggleCameraOverlay()` - Toggle full-screen overlay (O key)
- `togglePIP1()` - Toggle PIP 1 slideout (PERFORM, P key)
- `togglePIP2()` - Toggle PIP 2 corner (CREATE/ORG/ADMIN, P key)
- `togglePIP3()` - Toggle PIP 3 center (CREATE/ORG/ADMIN, Shift+P)
- `showPIPColorPicker(pipIndex, event)` - Show color swatch popup
- `hidePIPColorPicker()` - Hide color picker
- `setPIPColor(pipIndex, color)` - Set border color
- `setPIPShape(pipIndex, shape)` - Set shape (landscape/portrait/square/circle)
- `updatePIPStyle(pipIndex)` - Apply shape/color to element
- `testCamera()` - Preview camera in test mode
- `copyDiagnosticReport()` - Copy device info to clipboard

**TRANSFORM ANIMATION DOMAIN (V5_5_0):**
- `setTransformMode(control, mode)` - Set animation mode (0=stop, 1=cw, 2=ccw, 3=pingpong)
- `setTransformSpeed(control, speed)` - Set animation speed (0.03-256)
- `applyTransformAnimations()` - Called each frame, updates values
- `getTransformDelta(mode, speed, loopLength, beat)` - Calculate per-frame delta
- `handleTransformScroll(event)` - Adjust speed via trackpad scroll

**FX OVERLAY DOMAIN (V5_5_0):**
- `toggleFXOverlay()` - Toggle 120px panel visibility
- `initFXOverlayKnobs()` - Initialize 20×20 knobs (VFX + FX per track)
- `updateTrackVFXKnob(track, slot, value)` - Update VFX knob (placeholder)
- `drawFXOverlayKnob(canvas, value, color)` - Draw 20×20 knob (purple/orange)

**ORGANIZE AUTO-SAVE DOMAIN (V5_5_0):**
- `autoSaveDrawerField(fieldId)` - Save field on blur/Enter
- `navigateContainerWithAutoSave(direction)` - Navigate ↑/↓ with auto-save
- `initDrawerAutoSave()` - Initialize Apple Music-style auto-save
- `startQuickRename(containerId)` - Begin inline rename (Enter key)

**TAG AUTOCOMPLETE DOMAIN (V5_5_0):**
- `initTagAutocomplete()` - Initialize autocomplete system
- `showTagSuggestions(query)` - Show matching tags with counts
- `hideTagSuggestions()` - Hide suggestions dropdown
- `selectTagSuggestion(tag)` - Apply selected tag
- `navigateTagSuggestions(direction)` - Navigate ↑/↓ in suggestions

**#ForReview - LIVE TAB DOMAIN NAMING:**
- `startMasterClock()` - #ForReview should be `liveStartMasterClock()`
- `getNextBarBoundary()` - #ForReview should be `liveGetNextBarBoundary()`
- `isAnyTrackPlaying()` - #ForReview should be `liveIsAnyTrackPlaying()`
- Many LIVE tab functions lack `live*` prefix
- Waveform functions lack `waveform*` or `live*` prefix
- Mixer functions lack `mixer*` or `live*` prefix

---

### 2. UI FIELDS/VARIABLES (Organized by Usage Domain)

**SHADER DOMAIN (V5_1_0 Renames):**
- `shaderTempo` - shader tempo (was `tempo`) - visual animation speed, NOT audio BPM
- `shaderRenderDuration` - MM:SS.mmm format (was `renderDuration`) - when no audio
- `shaderLoopLength` - calculated loop length display (was `loopLength`)
- `shaderLoopLengthDropdown` - OmsDropdown instance (was `loopLengthDropdown`)
- `shaderTempoLinked` - V5_1_0: Whether shader tempo follows audio tempo (default: true)
- `modAWave`, `modBWave`, `modCWave` - modulator waveforms
- `modASubdiv`, `modBSubdiv`, `modCSubdiv` - modulator subdivisions
- `shaderCode` - WGSL fragment shader code
- `messageBar` - status message display

**AUDIO DOMAIN:**
- `audioFile` - file input element
- `audioFileName` - display text for filename
- `audioTempoField` - Audio Tempo (musical timing, NOT shader tempo)
- `audioStartField` - manual beatgrid start (seconds)
- `audioEndField` - manual beatgrid end (seconds)
- `inBeat` - loop In point (bar.beat format)
- `outBeat` - loop Out point (bar.beat format)
- `audioCuePoints` - V5_2_0: Array of 5 beat positions (default: [0, 64, 128, 192, 256])

**AUDIO REACTIVITY DOMAIN:**
- `bassGain` - 0.0-2.0 (multiplies modA)
- `midGain` - 0.0-2.0 (multiplies modB)
- `trebleGain` - 0.0-2.0 (multiplies modC)
- `bassGainGroup`, `midGainGroup`, `trebleGainGroup` - visual grouping

**RENDER SETTINGS DOMAIN:**
- `filename` - output filename (no extension)
- `resolution` - 720p, 1080p, 1440p, 2160p
- `fps` - 24, 30, 60
- `orientation` - landscape, portrait, square

**TRACK STATE ARRAYS (LIVE Tab):**
- `trackShaderOpacity[]` - per-track opacity (0-100, indices 0-3)
- `trackShaderMute[]` - per-track mute state (boolean, indices 0-3)
- `trackShaderSolo[]` - per-track solo state (boolean, indices 0-3)
- `trackAudioVolume[]` - per-track volume (0-100, indices 0-3)
- `trackAudioMute[]` - per-track audio mute (boolean, indices 0-3)
- `trackAudioSolo[]` - per-track audio solo (boolean, indices 0-3)

---

### 3. CONTAINERS (Data Structures)

**containerData** - .oms file structure
```javascript
{
  name: "Bass Drop Tunnel",
  version: "5.2.0",
  shaderCode: "/* WGSL */",
  tempo: 128,
  modAWave: "sine",
  audioCuePoints: [0, 64, 128, 192, 256],  // V5_2_0: 5 cues
  // ... full structure documented in File Formats section
}
```

**projectState** - .omsp file structure  
```javascript
{
  projectName: "Friday Set",
  version: "5.1.0",
  gridCells: [...],
  masterTempo: 128,
  bumprStickrsSlots: [...],      // V5_1_0
  bumprTimingModes: [...],       // V5_1_0
  bumprScaleModes: [...],        // V5_1_0
  bumprAnimateModes: [...],      // V5_1_0
  bumprDurationModes: [...],     // V5_1_0
  // ... full structure documented in File Formats section
}
```

**gridState** - LIVE tab runtime state
```javascript
{
  cells: [[...], [...], [...], [...]],  // 4 tracks × 8 scenes
  masterTempo: 120,
  zoomValue: 50,
  selectedTrack: 0,
  selectedScene: 0
}
```

---

### 4. FILE TYPES (Physical Storage)

**.oms Files:**
- Purpose: Single shader container
- Contains: Shader code, timing, modulators, audio reference, render settings, cue points (V5_1_0)
- Location: `/LIBRARY/[name].oms`

**.omsp Files:**
- Purpose: LIVE tab project state + BumpR StickRs state (V5_1_0)
- Contains: Grid layout, track states, master settings, BumpR configuration
- Location: `/LIBRARY/[projectName].omsp`
- Auto-save: `default.omsp`

**.oms-set Files:**
- Purpose: Container collections
- Contains: Array of .oms containerData objects
- Location: `/LIBRARY/[setName].oms-set`
- Use: Distribution, backup, sharing

---

### CRITICAL SEPARATION

**shaderTempo vs audioTempoField (V5_1_0 Naming):**
```javascript
shaderTempo     → Shader Tempo (visual animation speed)
audioTempoField → Audio Tempo (musical timing/beat positioning)

// These are NEVER the same value!
// Example:
shaderTempo = 128           // Shader animates at 128 BPM
audioTempoField = 124       // Audio track is 124 BPM
// Both independent, both valid

// V5_1_0: shaderTempoLinked
// When true: shader tempo automatically follows audio tempo
// When false: shader tempo is independent
```

**Why This Matters:**
- Shaders can animate at different speeds than music
- Pure visual mode (no audio) uses only `shaderTempo`
- Audio-reactive mode uses BOTH (independently or linked)

---

## FILE FORMAT SPECIFICATIONS

### .oms Files (Containers)

**Purpose:** Complete shader package with all settings and optional audio reference

**File Structure (V5_1_0):**
```javascript
{
  // Metadata
  name: "Bass Drop Tunnel",
  version: "5.1.0",
  created: "2025-12-24T10:30:00Z",
  
  // Shader Code
  shaderCode: "/* WGSL fragment shader code */",
  
  // Visual Timing (Independent from Audio)
  tempo: 128,                    // Shader Tempo (visual animation speed)
  
  // Modulators (A/B/C)
  modAWave: "sine",             // sine, square, triangle
  modASubdiv: 4,                // 1-16 (subdivision of tempo)
  modBWave: "square",
  modBSubdiv: 8,
  modCWave: "triangle",
  modCSubdiv: 16,
  
  // Audio Reference (Optional)
  audioReference: {
    filename: "bass_track.mp3",
    audioTempo: 124,              // Original track tempo
    audioStartTime: 17.1,       // Bar 1 position in file (seconds)
    audioEndTime: 300.0,        // Container end position (seconds)
    
    // Loop Points (bar.beat format)
    inBeat: "1.1",              // Loop In
    outBeat: "65.1",            // Loop Out
    
    // Audio Reactivity Gains
    bassGain: 1.0,              // 0.0-2.0 (modA multiplier)
    midGain: 1.0,               // 0.0-2.0 (modB multiplier)
    trebleGain: 1.0             // 0.0-2.0 (modC multiplier)
  },
  
  // Cue Points (V5_2_0: expanded to 5)
  audioCuePoints: [0, 64, 128, 192, 256],   // 5 beat positions relative to audioStartTime
  
  // Control States (V5_1_0)
  controlStates: {
    shaderTempoLinked: true,      // V5_1_0: Link shader tempo to audio tempo
    // ... other control states
  },
  
  // Mod Tuning (V5_3_0, V5_5_0: Per-Mod ADSR+T)
  modTuning: {
    // V5_5_0: Per-mod envelope parameters (18 total)
    modAQ: 0.70,           // Q (0.5-10)
    modAAttack: 5,         // Attack (0-100ms)
    modADecay: 20,         // Decay (0-200ms)
    modASustain: 70,       // Sustain (0-100%)
    modARelease: 50,       // Release (0-500ms)
    modAThreshold: 10,     // Threshold (0-100%, was 0-20%)
    
    modBQ: 0.70,
    modBAttack: 10,
    modBDecay: 30,
    modBSustain: 80,
    modBRelease: 100,
    modBThreshold: 5,
    
    modCQ: 0.70,
    modCAttack: 2,
    modCDecay: 15,
    modCSustain: 60,
    modCRelease: 80,
    modCThreshold: 5
  },
  
  // Transform Animation (V5_5_0)
  transformAnimation: {
    rotationMode: 'stop',      // 'stop', 'cw', 'ccw', 'pingpong'
    rotationSpeed: 1.0,        // 0.03-256 (exponential)
    scaleMode: 'stop',
    scaleSpeed: 1.0,
    tileMode: 'stop',
    tileSpeed: 1.0
  },
  
  // Container Tags (V5_3_0)
  tags: ["bass", "128", "drop"],  // Array of tag strings
  
  // Render Settings
  renderSettings: {
    resolution: "1080p",        // 720p, 1080p, 1440p, 2160p
    fps: 60,                    // 24, 30, 60
    orientation: "landscape",   // landscape, portrait, square
    filename: "bass_drop"       // Output filename prefix
  },
  
  // Thumbnail (base64 encoded PNG)
  thumbnail: "data:image/png;base64,..."
}
```

**Storage Location:** `/LIBRARY/[name].oms` (File System Access API)

**Audio Handling:**
- .oms files **reference** audio but don't store it
- Audio file must exist separately in user's filesystem
- User re-selects audio file when needed
- Audio is optional (pure shader mode works without it)

**Cue Points (V5_2_0):**
- Array of 5 beat positions (expanded from 3 in V5_1_0)
- Stored relative to audioStartTime (beat 0 = Bar 1.1)
- Default values: `[0, 64, 128, 192, 256]` (Bar 1.1, 17.1, 33.1, 49.1, 65.1)
- Draggable with bar snap (4 beats)
- Legacy 3-cue containers auto-padded to 5

**WAVEFORM ARCHITECTURE - What's Stored vs. Calculated:**

**What's STORED in .oms (controlStates):**
```javascript
"controlStates": {
  // Modulator A Settings (STORED)
  "shaderModAWaveform": "cycler",     // Waveform type
  "shaderModASubdiv": 1,              // Subdivision
  "shaderModAGain": 1.0,              // Gain multiplier
  "shaderModAFreq1": 30,              // FFT range start (Hz) - analysis only
  "shaderModAFreq2": 50,              // FFT range end (Hz) - analysis only
  
  // Modulator B Settings (STORED)
  "shaderModBWaveform": "sawtooth",
  "shaderModBSubdiv": 4,
  "shaderModBGain": 1.0,
  "shaderModBFreq1": 50,
  "shaderModBFreq2": 75,
  
  // Modulator C Settings (STORED)
  "shaderModCWaveform": "triangle",
  "shaderModCSubdiv": 4,
  "shaderModCGain": 1.0,
  "shaderModCFreq1": 100,
  "shaderModCFreq2": 500,
  
  // V5_1_0: Shader Tempo Link
  "shaderTempoLinked": true
}

// Timeline Arrays - Pre-analyzed FFT data (STORED)
"shaderModATimeline": [0.062, 0.052, 0.036, ...],  // Bass levels
"shaderModBTimeline": [0.125, 0.089, 0.043, ...],  // Mid levels
"shaderModCTimeline": [0.234, 0.156, 0.078, ...]   // Treble levels
```

**What's NOT STORED (Calculated Live):**
- Waveform OUTPUT values - calculated each frame using `calculateMod(beat, subdivision, waveformType)`
- These are deterministic: same settings + time = same output

**5 Columns per Modulator (UI Controls):**
1. **Subdivision** (1/4, Whole) - Waveform timing - ✅ LIVE needs
2. **Waveform** (Cycler, Sawtooth) - Waveform type - ✅ LIVE needs
3. **Gain** (1.0, 2.0) - Amplitude boost - ✅ LIVE needs
4. **Freq1** (30, 50, 100 Hz) - FFT range start - ❌ Analysis only
5. **Freq2** (50, 75, 500 Hz) - FFT range end - ❌ Analysis only

**Why Freq1/Freq2 Not Needed in LIVE Tab:**
- Used DURING timeline analysis (Controls Tab → Add to Library)
- Define which audio frequencies (Hz) get analyzed into timeline arrays
- Timeline arrays contain the RESULTS of that analysis
- LIVE tab just reads pre-analyzed timeline data

**Two-Layer Modulator System:**
```javascript
// Layer 1: Waveform (tempo-synced oscillator)
const waveform = calculateMod(beat, subdivision, waveformType);  // From settings

// Layer 2: Audio (pre-analyzed FFT)
const audioLevel = timeline[index];  // From timeline array

// Layer 3: Gain
const gain = controlStates.shaderModAGain;  // From settings

// Final Modulator
const modA = waveform × audioLevel × gain;
```

**The Prep → .oms → LIVE Contract:**
- Controls Tab (Prep): Author/test with both layers
- .oms file: Store settings + timeline arrays
- LIVE Tab (Performance): Reconstruct identically using same math + stored data
- Test: "Does it look the same?" - must be pixel-perfect match

---

### .omsp Files (Project States)

**Purpose:** Save/restore LIVE tab performance grid layout + BumpR StickRs

**File Structure (V5_1_0):**
```javascript
{
  // Metadata
  projectName: "Friday Set",
  version: "5.1.0",
  saved: "2025-12-24T22:45:00Z",
  
  // Grid State (4 tracks × 8 scenes = 32 cells)
  gridCells: [
    {
      track: 1,
      scene: 1,
      containerName: "Bass Drop Tunnel",
      containerPath: "/LIBRARY/Bass Drop Tunnel.oms"
    },
    {
      track: 1,
      scene: 2,
      containerName: "Melodic Waves",
      containerPath: "/LIBRARY/Melodic Waves.oms"
    },
    // ... up to 32 cells
  ],
  
  // Master Settings
  masterTempo: 128,             // Global BPM (60-200)
  zoomLevel: 50,                // Pixels per second
  selectedSetId: "all",         // V4.4.0: Current set selection
  
  // Project Defaults (V4_6_0)
  defaultLoopIn: "1.1",         // Default Loop In for fresh audio (bar.beat)
  defaultLoopOut: "17.1",       // Default Loop Out for fresh audio (bar.beat)
  defaultOrientation: "landscape", // Default orientation (V4_6_0)
  
  // Track States
  tracks: [
    {
      // Shader Controls
      shaderOpacity: 100,       // 0-100%
      shaderMute: false,
      shaderSolo: false,
      shaderManual: false,      // AUTO vs MANUAL mode
      
      // Audio Controls
      audioVolume: 75,          // 0-100%
      audioMute: false,
      audioSolo: false
    },
    // ... tracks 2-4
  ],
  
  // MAIN Controls (V5_4_0: renamed from master*)
  trackMainShaderMute: false,
  trackMainShaderSolo: false,
  trackMainVolume: 100,
  trackMainAudioMute: false,
  
  // Audio Card Settings (V5_4_0)
  audioCardSettings: {
    masterOutputId: null,       // Device ID or null for default
    cueOutputId: null,          // Device ID or null for none
    cueMix: 0.5                 // 0 = MAIN only, 1 = Cue only
  },
  
  // Per-Track Cue State (V5_4_0)
  trackCues: [false, false, false, false],
  
  // Scene Customization (V5_4_0)
  sceneLabels: ['1', '2', '3', '4', '5', '6', '7', '8'],
  sceneColors: [null, null, null, null, null, null, null, null],
  // Colors: null (default), 'cyan', 'magenta', 'orange', 'yellow', 'green', 'purple', 'red'
  
  // Audio Preferences (V5_4_0)
  audioPrefs: {
    masterTempo: 120            // Default tempo for new sessions
  },
  
  // DSP Settings (V5_4_0)
  dspSettings: {
    enabled: false,
    preset: 'off',              // 'off', 'gentle', 'medium', 'aggressive', 'limiter'
    threshold: -18,             // -48 to 0 dB
    ratio: 4,                   // 1 to 20
    output: 4                   // -12 to +12 dB
  },
  
  // VFX Settings (V5_4_0)
  vfxSettings: {
    effect: 'off',              // 'off', 'rgbsplit', 'glitch', 'pixelate', etc.
    amount: 0                   // 0-100
  },
  
  // Video Sources & PIP Settings (V5_5_0)
  videoSourcesSettings: {
    deviceId: null,             // Camera device ID or null for default
    mirror: false               // Mirror mode toggle
  },
  pip1Settings: {
    visible: false,             // PIP 1 slideout (PERFORM)
    shape: 'landscape',         // 'landscape', 'portrait', 'square', 'circle'
    color: 'cyan',              // 'none', 'cyan', 'magenta', 'orange', etc.
    label: ''                   // Optional label for streaming
  },
  pip2Settings: {
    shape: 'landscape',         // PIP 2 corner (CREATE/ORG/ADMIN)
    color: 'cyan'
  },
  pip3Settings: {
    shape: 'landscape',         // PIP 3 center (CREATE/ORG/ADMIN)
    color: 'cyan'
  },
  
  // Quantize Setting (V5_3_0)
  quantizeMode: "bar",              // 'off', 'beat', 'bar' (default)
  
  // BumpR StickRs (V5_1_0)
  bumprStickrsSlots: [
    { filename: "logo.webm", thumbnail: "data:image/png;base64,..." },
    null, null, null, null, null, null, null, null, null
  ],
  bumprTimingModes: ['instant', 'instant', 'instant', 'instant', 'instant', 
                     'instant', 'instant', 'instant', 'instant', 'instant'],
  bumprScaleModes: [100, 100, 100, 100, 100, 100, 100, 100, 100, 100],
  bumprAnimateModes: ['none', 'none', 'none', 'none', 'none',
                      'none', 'none', 'none', 'none', 'none'],
  bumprDurationModes: ['original', 'original', 'original', 'original', 'original',
                       'original', 'original', 'original', 'original', 'original']
}
```

**Storage Location:** `/LIBRARY/[projectName].omsp`

**Auto-Save:**
- Default project: `default.omsp` (auto-saves every action)
- Manual saves: Custom names via Settings tab

---

### .oms-set Files (Preset Collections)

**Purpose:** Bundle multiple presets for distribution/backup

**File Structure (V5_3_0):**
```javascript
{
  // Metadata
  setName: "Bass Music Collection",
  author: "ONEMANSHYO",
  version: "5.3.0",
  created: "2025-12-27T10:00:00Z",
  description: "Heavy bass presets for 128-140 BPM",
  
  // V5_3_0: Set type field
  setType: "manual",                   // 'manual' or 'smart'
  smartQuery: null,                    // Query string if smart set
  
  // Preset Array
  presets: [
    {
      name: "Bass Drop Tunnel",
      category: "Bass",
      shaderCode: "/* ... */",
      tempo: 128,
      audioCuePoints: [0, 64, 128, 192, 256],  // V5_2_0
      tags: ["bass", "drop"],                   // V5_3_0
      // ... full .oms structure
    },
    {
      name: "Sub Wobble Grid",
      category: "Bass",
      shaderCode: "/* ... */",
      tempo: 140,
      audioCuePoints: [0, 64, 128, 192, 256],  // V5_2_0
      tags: ["bass", "wobble"],                 // V5_3_0
      // ... full .oms structure
    },
    // ... multiple presets
  ],
  
  // Set-level settings (optional)
  defaultTempo: 128,
  defaultResolution: "1080p",
  tags: ["bass", "dubstep", "heavy"]
}
```

**Storage Location:** `/Sets/[setName].oms-set` (V5_3_0: moved from /Library/)

**Usage:**
- Import/export preset collections
- Share curated sets with community
- Backup entire LIBRARY

---

## CORE SYSTEMS ARCHITECTURE

### 1. Audio System (Web Audio API)

**Purpose:** Audio playback, analysis, and mixing

**Key Components:**
```javascript
// Audio Context (single global instance)
audioContext = new AudioContext();

// Master Clock
master = {
  bpm: 120,                           // Current master tempo
  startContextTime: null,             // First launch time (never reset)
  playing: false
};

// Track Audio Sources (per track)
trackAudioSources = [null, null, null, null];      // AudioBufferSourceNode[]
trackGainNodes = [null, null, null, null];         // GainNode[]
trackAnalysers = [null, null, null, null];         // AnalyserNode[]

// Audio Buffers (loaded containers)
audioBuffers = new Map();             // containerName → AudioBuffer

// Playback State
trackAudioStartTimes = [null, null, null, null];   // When each track started
trackAudioPlaying = [false, false, false, false];
```

**Audio Graph:**
```
AudioBufferSourceNode (per track)
    ↓ (playbackRate for time-stretching)
GainNode (per track volume)
    ↓
AnalyserNode (per track FFT)
    ↓
Master GainNode
    ↓
audioContext.destination (speakers)
```

**Time-Stretching:**
```javascript
// All tracks play at master tempo
const playbackRate = containerBPM / master.bpm;
source.playbackRate.value = playbackRate;

// Example: 124 BPM track → 120 BPM master
// playbackRate = 124 / 120 = 1.033 (plays 3.3% faster)
```

**Quantized Launching:**
```javascript
// Calculate next bar boundary
const currentBeat = (audioContext.currentTime - master.startContextTime) 
                    * (master.bpm / 60);
const nextBar = Math.ceil(currentBeat / 4) * 4;
const launchTime = master.startContextTime + (nextBar * 60 / master.bpm);

// Start audio at calculated time
source.start(launchTime, containerData.audioStartTime);
```

**Quantize Modes (V5_3_0):**
```javascript
// Quantize dropdown replaces RMS input
const quantizeModes = ['off', 'beat', 'bar'];  // Default: 'bar'

function getNextBoundary(quantizeMode) {
    const currentBeat = masterBeatNow();
    if (quantizeMode === 'off') return currentBeat;      // Immediate
    if (quantizeMode === 'beat') return Math.ceil(currentBeat);  // Next beat
    if (quantizeMode === 'bar') return Math.ceil(currentBeat / 4) * 4;  // Next bar
}

// V5_3_0: Forward-only - always snaps ahead, never backward
```

**Real Audio Reactivity (V5_3_0):**

Previous versions used flat FFT bin averaging:
```javascript
// OLD (V5_2_0 and earlier): Constant ~0.02 regardless of content
const sum = bins.reduce((a, b) => a + b, 0);
const avg = sum / bins.length;  // Always ~0.02
```

V5_3_0 uses bandpass-filtered peak detection:
```javascript
// NEW (V5_3_0): Real dynamics
// Kick hit → 0.72, Silence → 0.01, Hi-hat → 0.15

function analyzeModulator(analyser, centerFreq, Q, adsr) {
    // 1. Bandpass filter isolates frequency range
    const filtered = applyBandpass(analyser, centerFreq, Q);
    
    // 2. Peak detection captures transients
    const peak = detectPeak(filtered);
    
    // 3. ADSR envelope shapes response
    const shaped = applyADSR(peak, adsr);
    
    return shaped;  // 0.0-1.0 with real dynamics
}
```

**Bandpass Filter Chain (per modulator):**
```
Audio Source → BiquadFilterNode (bandpass) → Peak Detector → ADSR Envelope → modValue
```

**Filter Parameters:**

| Modulator | Center Freq | Q Range | Default Q |
|-----------|-------------|---------|-----------|
| Mod A | (Freq1+Freq2)/2 | 0.5-10 | 1.0 |
| Mod B | (Freq1+Freq2)/2 | 0.5-10 | 1.0 |
| Mod C | (Freq1+Freq2)/2 | 0.5-10 | 1.0 |

**ADSR Envelope Parameters:**

| Parameter | Range | Default | Effect |
|-----------|-------|---------|--------|
| Attack | 0-100ms | 5ms | Rise time to peak |
| Decay | 0-200ms | 20ms | Fall time to sustain |
| Sustain | 0-100% | 70% | Held level |
| Release | 0-500ms | 50ms | Return to zero |
| Threshold | 0-20% | 1% | Noise gate |

---

### 2. Shader System (WebGPU)

**Purpose:** Real-time shader rendering and compositing

**Key Components:**
```javascript
// WebGPU Device/Context
device = await navigator.gpu.requestAdapter()
                          .then(a => a.requestDevice());

// Per-Track Rendering Pipelines
trackShaderPipelines = [null, null, null, null];   // GPURenderPipeline[]
trackCanvases = [canvas1, canvas2, canvas3, canvas4]; // HTMLCanvasElement[]

// Shader Uniforms (sent to GPU each frame) - V5_0_0: 112 bytes
uniforms = {
  time: 0.0,              // Seconds since shader start
  tempo: 128,             // Shader Tempo (visual timing)
  modA: 0.0,              // Combined bass (0-1) - legacy
  modB: 0.0,              // Combined mids (0-1) - legacy
  modC: 0.0,              // Combined treble (0-1) - legacy
  beat: 0.0,              // Tempo-synced beat position
  resolutionX: 1920,      // Canvas width
  resolutionY: 1080,      // Canvas height
  shaderModAWave: 0.0,    // Mod A waveform output
  shaderModAGain: 0.1,    // Mod A gain value
  shaderModAAudio: 0.0,   // Mod A audio level (V5_6_0 rename from FFT)
  shaderModBWave: 0.0,    // Mod B waveform
  shaderModBGain: 0.1,    // Mod B gain
  shaderModBAudio: 0.0,   // Mod B audio level
  shaderModCWave: 0.0,    // Mod C waveform
  shaderModCGain: 0.1,    // Mod C gain
  shaderModCAudio: 0.0,   // Mod C audio level
  transformRotation: 0.0, // Rotation (radians)
  transformScaleX: 1.0,   // Scale X
  transformScaleY: 1.0,   // Scale Y
  transformTranslateX: 0.0, // Position X
  transformTranslateY: 0.0, // Position Y
  transformPivotX: 0.0,   // Pivot X
  transformPivotY: 0.0,   // Pivot Y
  transformTile: 1.0,     // Tile count
  padding1: 0.0,          // GPU alignment
  padding2: 0.0,
  padding3: 0.0           // 28 floats = 112 bytes
};
```

**Shader Compilation:**
```javascript
// App injects struct and binding - user code just uses uniforms.*
// User writes ONLY the fragment function:

@fragment
fn fs_main(@builtin(position) pos: vec4<f32>) -> @location(0) vec4<f32> {
  // Access any uniform directly:
  let t = uniforms.time;
  let bass = uniforms.modA;           // Legacy combined
  let bassWave = uniforms.shaderModAWave;  // Just waveform
  let bassAudio = uniforms.shaderModAAudio;  // Just audio level (V5_6_0)
  
  return vec4<f32>(r, g, b, 1.0);
}
```

**Modulator Calculation:**

**CRITICAL:** Uses centralized `calculateMod(beat, subdivision, waveform)` function

**Available Waveform Types:**
```javascript
// V4_8_0: Added none and static
- 'none'     // Returns 0 - modulator disabled
- 'static'   // Returns 1.0 - no oscillation, gain controls directly

// Basic Waveforms:
- 'sine'     // Sine wave
- 'square'   // Square wave  
- 'triangle' // Triangle wave
- 'sawtooth' // Sawtooth wave

// Meta-Modes (Complex waveforms):
- 'cycler'    // Bounces up and down
- 'chaosmode' // Chaotic/random behavior
- 'madmax'    // Aggressive variations
- 'psycho'    // Extreme modulation

// All normalize to 0.0-1.0 range
wave = (wave + 1.0) * 0.5;
```

**V4_8_0 Decoupled Modulator System:**

The modulator system was fundamentally redesigned in V4_8_0 to provide both legacy compatibility and independent component access.

**Three Independent Components (per modulator):**
```javascript
// Each modulator (A/B/C) has three independent values:
Wave  // Waveform oscillator output (0-1) - works without audio
Gain  // User knob value (0-1, default 0.1)
FFT   // Audio frequency level (0-1) - 0 when no audio loaded
```

**Legacy Combined Value:**
```javascript
// For backward compatibility, modA/B/C combine all three:
modA = Wave × Gain × Audio  // When audio loaded (V5_6_0 terminology)
modA = Wave × Gain          // When no audio (Audio treated as 1.0)
```

**Independent Shader Access (V5_6_0 Rename):**
```javascript
// Shaders can access components separately:
uniforms.shaderModAWave   // Just the waveform
uniforms.shaderModAGain   // Just the gain knob value
uniforms.shaderModAAudio  // Just the audio level (V5_6_0 rename from FFT)

// Enables custom combinations:
let customMod = uniforms.shaderModAWave * uniforms.shaderModBAudio;
```

**Works Without Audio:**
```javascript
// When no audio loaded:
- Wave functions normally (tempo-synced oscillation)
- Gain functions normally (user-controlled)
- Audio = 0 (no audio to analyze)
- Freq knobs disabled (meaningless without audio/mic)

// When audio loaded OR mic active (V5_6_0):
- All three components active
- Freq knobs enable, define analysis range
```

**Four-Track Compositing:**
```javascript
// Z-index layering (CSS)
Track 1 canvas: z-index 4 (top)
Track 2 canvas: z-index 3
Track 3 canvas: z-index 2
Track 4 canvas: z-index 1 (bottom)

// Opacity blending
canvas.style.opacity = trackShaderOpacity[track] / 100;

// Composite output → pop-out window
```

---

### 3. Master Clock System

**Purpose:** Single timeline for all tracks - session-based architecture

**THE MASTER CLOCK TIMELINE**

**What the master clock actually is:**
- `audioContext.currentTime` - Web Audio API's timeline (never stops, never resets - browser limitation)
- `trackMasterClockStart` - records when session started (when first track launched)
- `trackAudioMasterClock()` - converts between computer time and musical beats

**How sessions work:**
- First track launches → `trackMasterClockStart` = current `audioContext.currentTime`
- This becomes beat 0 (Bar 1.1.1) for the session
- All subsequent tracks reference this session start time
- Master clock counts beats from session start: 0, 4, 8, 12, 16, 20, 24...

**Session restart conditions:**
- Escape key (emergency stop) → `trackMasterClockStart` = null
- All tracks stopped → `trackMasterClockStart` = null

**What happens on restart:**
- `trackMasterClockStart` resets to null
- Next track that launches starts NEW session
- That launch becomes new beat 0 (Bar 1.1.1)
- Clock dot animation stops/restarts
- DAW clock readout stops/restarts

**CRITICAL NOTE:** `audioContext.currentTime` never resets (keeps counting forever), but `trackMasterClockStart` resets, which effectively restarts the session timeline.

**The Two-Timeline System:**

```javascript
// Timeline 1: Browser Timeline (Absolute)
audioContext.currentTime  // Never resets, runs forever

// Timeline 2: Session Timeline (Relative)  
trackMasterClockStart     // Resets to null, marks session start

// The Math:
sessionTime = audioContext.currentTime - trackMasterClockStart;

// Convert to musical beats:
function trackAudioMasterClock() {
  if (!trackMasterClockStart) return 0;
  const sessionTime = audioContext.currentTime - trackMasterClockStart;
  return (sessionTime * master.bpm) / 60.0;  // Beats since session start
}
```

**Analogy:**
```
audioContext.currentTime = wall clock (never stops)
trackMasterClockStart = "start of class" marker (resets each class)
Session time = "minutes into class" (resets when class ends)
```

**Critical Implementation Notes:**
```javascript
// CORRECT - always check for null
if (!trackMasterClockStart) {
  return 0;  // No session running
}
const beat = trackAudioMasterClock();

// WRONG - will crash if no session
const beat = trackAudioMasterClock();  // Error if trackMasterClockStart is null!

// CORRECT - reset session timeline
trackMasterClockStart = null;

// WRONG - impossible, browser limitation
audioContext.currentTime = 0;  // Does nothing!

// CORRECT - session start only on first launch
if (!trackMasterClockStart) {
  trackMasterClockStart = audioContext.currentTime;  // Set once
}

// WRONG - resets timeline on every launch
trackMasterClockStart = audioContext.currentTime;  // Breaks multi-track sync!
```

---

### 4. Track Display System (LIVE Tab)

**Purpose:** Manages complete track visualization (waveform + beat grid + positioning + scroll)

---

**Layer Architecture:**

| Layer | Behavior | Drawing |
|-------|----------|---------|
| Master Grid | FIXED | Bar 1 at playheadX (center), extends left/right |
| Track Grid | SCROLLS | Drawn on canvas at bar positions, canvas translateX moves |
| Waveform | SCROLLS | Blue amplitude bars, same canvas as Track Grid |
| Cue Flags | SCROLLS | Orange flags at cue point positions (V5_1_0) |
| Playhead | FIXED | Magenta line drawn on Master Grid canvas at playheadX |

**Key Insight:** Master Grid is stationary ruler. Track Grid + Waveform + Cue Flags scroll under it.

---

**CSS Positioning Model (Critical):**

**Container CSS:**
```css
.waveform-canvas-container {
    position: relative;
    overflow: hidden;
    height: 100%;
    /* NO display: flex - causes container collapse */
}
```

**Canvas CSS:**
```css
.waveform-canvas {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
}
```

**Why Absolute Positioning:**
- `position: relative` + translateX = moves from document flow position (unpredictable)
- `position: absolute` + translateX = moves from container origin (0,0) (predictable)

**Why NOT Flex:**
- Flex container sizes based on children's content
- Absolute-positioned child removed from flow
- Container has no content → collapses to 0 height → nothing visible

---

**Bar1X Positioning Model:**

**The Two-Timeline Problem:**

There are two different time references in the system:

1. **Audio File Time (seconds):** The original audio file's timeline
   - `audioStartTime` = where musical Bar 1 appears in the file (e.g., 17.1 seconds)
   - Measured in seconds at original BPM
   - Changes when audio is time-stretched

2. **Musical Time (beats/bars):** The master clock's timeline
   - Bar 1, Bar 2, Bar 3... at master tempo
   - Measured in beats/bars at master BPM
   - Never changes - always counts from session start

**The Critical Mistake (V3.9.0 bug):**
```javascript
// WRONG - uses audio file seconds directly
const bar1PositionPx = audioStartTime * pixelsPerSecond;  // 17.1s × 50px/s = 855px

// Result: Bar 1 positioned 855px from canvas origin
// But Master Grid draws Bar 1 at playhead (center)
// = constant 855px offset between grids
```

**Why This Fails:**
- `audioStartTime` is in the audio file's timeline (original BPM)
- After time-stretching (124 BPM → 120 BPM), those seconds don't map to same musical position
- Master Grid uses musical bars at master tempo
- Coordinate system mismatch = offset

**The Correct Approach:**

Convert audio file seconds → musical beats → pixels:

```javascript
// 1. Calculate playhead position (center of container)
const playheadX = container.clientWidth / 2;

// 2. Calculate where Bar 1 sits on canvas
// CRITICAL: Convert audioStartTime (seconds) → beats → pixels
const offsetBeats = (audioStartTime * originalBPM) / 60;  // Seconds → beats at original BPM
const bar1X = offsetBeats * pixelsPerBeat;                // Beats → pixels at master tempo

// Example: audioStartTime = 17.1s, originalBPM = 124, pixelsPerBeat = 100
// offsetBeats = (17.1 * 124) / 60 = 35.34 beats
// bar1X = 35.34 * 100 = 3534px

// 3. Store for scroll animation
trackBar1Positions[track - 1] = bar1X;

// 4. Initial position: put Bar 1 at playhead
canvas.style.transform = `translateX(${playheadX - bar1X}px)`;

// 5. Scroll: subtract elapsed pixels
const scrollPx = elapsedBeats * pixelsPerBeat;
canvas.style.transform = `translateX(${playheadX - bar1X - scrollPx}px)`;
```

**The Formula:** `translateX = playheadX - bar1X - scrollPx`

All three terms required. Miss one = broken alignment.

**Why This Works:**
- `bar1X` is calculated in **musical beats** (using original BPM)
- Then converted to **pixels** (using master tempo spacing)
- Master Grid also uses musical beats → pixels (master tempo)
- **Same coordinate system = perfect alignment**

---

**Canvas-Drawn Playhead:**

**Before (CSS pseudo-element) - BROKEN:**
```css
.waveforms-section::before {
    content: '';
    position: absolute;
    left: calc(50% - 1px);  /* Diverges from JS calculation! */
}
```

**After (Canvas-drawn) - CORRECT:**
```javascript
function drawMasterGrid() {
    const playheadX = canvas.width / 2;  // Same calculation as scroll positioning
    
    // ALWAYS draw playhead (even when grid hidden)
    ctx.strokeStyle = '#ff00ff';  // Magenta
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(playheadX, 0);
    ctx.lineTo(playheadX, canvas.height);
    ctx.stroke();
    
    // Draw grid lines only if visible
    if (masterGridVisible) {
        // Draw bar lines...
    }
}
```

**Why Canvas-Drawn:**
- Uses exact same playheadX calculation as scroll positioning
- No CSS calc() divergence
- Same coordinate system as grid lines
- Always visible (even when grid hidden)

---

**#ForReview - CURRENT NAMING ISSUES:**
All components named "waveform" when they actually manage complete track display system.

**Component 1: Waveform (Amplitude Bars)**
- Physical: Blue/cyan bars showing audio levels
- Affected by: RMS calibration (height), Zoom (spacing)
- Function: `drawWaveformFromAudioBuffer()` #ForReview misleading name - also draws grid + cue flags
- Independence: Changes with RMS, grid does NOT (proves separation)

**Component 2: Beat Grid (Timing Markers)**
- Physical: Magenta phrase markers + white bar markers
- Affected by: Zoom (spacing), NOT affected by RMS
- Function: `drawBeatGridOnWaveform()` - accurate name
- Independence: Separate from waveform, drawn on same canvas

**Component 3: Cue Flags (V5_1_0)**
- Physical: Orange (#ffa500) vertical line + triangle + number
- Position: Based on audioCuePoints[] beat positions
- Draggable in CREATE tab
- Click-to-jump in both tabs

**Component 3b: Bar 1 Marker (V5_3_0)**
- Physical: White "1" text marker at audioStartTime position
- Draggable in CREATE tab to adjust beatgrid
- Updates audioStartTime on drag release
- Visual indicator of where musical Bar 1 falls in audio file

**Bar 1 Dragging (V5_3_0):**
```javascript
// State
let bar1Dragging = false;
let bar1DragStartX = 0;
let bar1DragStartTime = 0;

// Mouse handlers (CREATE tab only)
handleBar1MouseDown(event)
  // Check if click near Bar 1 marker
  // Sets: bar1Dragging = true, stores start position

handleBar1MouseMove(event)
  // Update marker position during drag
  // Visual feedback only (no audio change yet)
  // Marker follows mouse horizontally

handleBar1MouseUp(event)
  // Calculate new audioStartTime from final position
  // Update audioStartField
  // Trigger waveform redraw
```

**Waveform Coloring (V5_3_0):**

| Time Region | Color | Meaning |
|-------------|-------|---------|
| Before Bar 1 | Grey (#666) | Intro/count-in (pre-musical content) |
| After Bar 1 | Blue (#00d4ff) | Main content (musical material) |

```javascript
function drawWaveformFromAudioBuffer() {
    for (let i = 0; i < samples; i++) {
        const sampleTime = i / sampleRate;
        const isBeforeBar1 = sampleTime < audioStartTime;
        
        ctx.fillStyle = isBeforeBar1 ? '#666666' : '#00d4ff';
        // Draw amplitude bar...
    }
}
```

**Component 4: Canvas Positioning**
- Physical: translateX() CSS transform
- Purpose: Position Bar 1 at playhead, scroll during playback
- Function: `positionTrackAudioWaveformAtPlayhead()` #ForReview misleading - manages entire display

**Component 5: Scroll Animation**
- Physical: requestAnimationFrame loop
- Purpose: Move canvas left during playback
- Functions: `startAudioWaveformScroll()`, `stopAudioWaveformScroll()` #ForReview misleading - affects entire display

**Master Orchestrator:**
- Function: `updateTrackWaveform()` #ForReview should be `updateTrackDisplay()` or `redrawTrack()`
- What it does: Stops scroll → Redraws everything → Repositions → Restarts scroll
- Problem: Name suggests waveform-only, actually controls ENTIRE track display

**Beat Grid Visual Reference:**
```
M = Magenta (phrase marker)
W = White (bar marker)
| = Beat marker
C = Cue flag (orange, V5_1_0)

Phrase 1 (bars 1-4):
C | | | | W | | | | W | | | | W | | | | M
^         ^         ^         ^         ^
1.1.1     2.1.1     3.1.1     4.1.1     5.1.1 (Phrase 2 starts)

Phrase 2 (bars 5-8):
M | | | | W | | | | W | | | | W | | | | M
^         ^         ^         ^         ^
5.1.1     6.1.1     7.1.1     8.1.1     9.1.1 (Phrase 3 starts)
```

---

### 5. Pixel Calculation System

**Purpose:** Unified formula for all grid/waveform positioning

**Core Constants:**
```javascript
const BASE_PIXELS_PER_SECOND = 6.25;  // CORTEX - do not change
const BEATS_PER_BAR = 4;
```

**Core Functions:**
```javascript
function getPixelsPerBeat() {
    const masterBPM = gridState.masterTempo || 120;
    const secondsPerBeat = 60 / masterBPM;
    return BASE_PIXELS_PER_SECOND * secondsPerBeat / (gridState.waveformZoom || 1.0);
}

function getPixelsPerBar() {
    return getPixelsPerBeat() * BEATS_PER_BAR;
}
```

**Golden Rule:** Same formula used EVERYWHERE = perfect alignment.

**Used In:**
- `drawMasterGrid()` - Master Grid bar positions
- `drawBeatGridOnWaveform()` - Track Grid bar positions
- `drawWaveformFromAudioBuffer()` - Canvas width calculation
- `startClockDotAnimation()` - Clock dot movement
- `startAudioWaveformScroll()` - Waveform scroll rate
- `drawProgress()` - Cue flag positions (V5_1_0)

---

### 6. LIBRARY System (File System Access API)

**Purpose:** Persistent storage for containers and projects

**V4.3.0_A Architecture Change:** Folder is now single source of truth. localStorage no longer used for container storage.

---

**Directory Structure (V5_3_0):**
```
[User's Selected Folder]/              ← applicationFolderHandle
  ├── onemanshyo.omsp                  ← App state (auto-save, includes BumpR settings)
  │
  ├── Library/                         ← libraryFolderHandle
  │   ├── Container1.oms               ← Container files
  │   ├── Container2.oms
  │   ├── audio1.mp3                   ← Referenced audio
  │   ├── audio2.wav
  │   └── MyProject.omsp               ← Named project saves
  │
  ├── Presets/                         ← User shader presets (V4.4.0)
  │   ├── Preset1.oms                 ← .oms containers
  │   ├── Preset2.oms
  │   └── MyShader.wgsl               ← V4_8_0: Raw WGSL files also supported
  │
  ├── Sets/                            ← V5_3_0: Manual and smart sets
  │   ├── Bass Collection.oms-set     ← Manual set
  │   └── 128 BPM.oms-set             ← Smart set (cyan dot indicator)
  │
  ├── BumpRStickRs/                    ← V5_1_0: Video overlays
  │   ├── logo.webm                   ← WebM VP9 with alpha
  │   ├── effect1.webm
  │   └── effect2.webm
  │
  └── Exports/
      └── Container1.oms.zip           ← ZIP exports
```

---

**Source of Truth:**

| Aspect | Before V4.3.0 | After V4.3.0_A |
|--------|---------------|----------------|
| Container list | localStorage | Folder scan → cache |
| Container ID | Internal UUID | Filename (without .oms) |
| App state location | /Library/default.omsp | /onemanshyo.omsp |
| Storage limit | 5MB (localStorage) | Unlimited (filesystem) |

---

**Four File Handles (CORTEX) - V5_3_0:**
```javascript
let applicationFolderHandle = null;   // Root folder (user selects)
let libraryFolderHandle = null;       // /Library/ subfolder
let bumprStickrsFolderHandle = null;  // /BumpRStickRs/ subfolder (V5_1_0)
let setsFolderHandle = null;          // /Sets/ subfolder (V5_3_0)

// On folder selection:
applicationFolderHandle = await window.showDirectoryPicker();
libraryFolderHandle = await applicationFolderHandle.getDirectoryHandle('Library', { create: true });
bumprStickrsFolderHandle = await applicationFolderHandle.getDirectoryHandle('BumpRStickRs', { create: true });
setsFolderHandle = await applicationFolderHandle.getDirectoryHandle('Sets', { create: true });
```

---

**Container ID = Filename:**
```javascript
// File on disk: "My Cool Shader.oms"
// Container ID: "My Cool Shader"

// Load by ID:
const fileHandle = await libraryFolderHandle.getFileHandle(containerId + '.oms');

// Rename = rename file:
await libraryFolderHandle.removeEntry(oldId + '.oms');
await libraryFolderHandle.getFileHandle(newId + '.oms', { create: true });
```

---

**Cache System:**
```javascript
let libraryContainersCache = [];

async function refreshLibraryList() {
    const containers = [];
    
    for await (const entry of libraryFolderHandle.values()) {
        if (entry.name.endsWith('.oms')) {
            const file = await entry.getFile();
            const data = JSON.parse(await file.text());
            
            containers.push({
                ...data,                              // Full container data
                id: entry.name.replace('.oms', ''),   // ID from filename
                filename: entry.name
            });
        }
    }
    
    // Sort by current settings
    containers.sort((a, b) => {
        let cmp = librarySortBy === 'name' 
            ? a.name.localeCompare(b.name)
            : (a.audioTempo || 0) - (b.audioTempo || 0);
        return librarySortDir === 'asc' ? cmp : -cmp;
    });
    
    libraryContainersCache = containers;
}

// LIVE tab uses cache (returns FULL data)
function getLibraryContainers() {
    return libraryContainersCache;
}

// DEPRECATED - no longer writes to localStorage
function saveLibraryContainers() {
    // No-op in V4.3.0_A
}
```

---

**Permissions:**
- Requested once per browser session
- Persists across page reloads (same session)
- User must re-grant on new browser session

---

### 7. BumpR StickRs System (V5_1_0)

**Purpose:** Video overlays triggered during live performance

**Key Components:**
```javascript
// State variables
let bumprStickrsFolderHandle = null;      // Folder handle
let bumprStickrsSlots = [null × 10];      // {filename, thumbnail}
let bumprTimingModes = ['instant' × 10];  // instant, beat, bar
let bumprScaleModes = [100 × 10];         // 25, 50, 75, 100
let bumprAnimateModes = ['none' × 10];    // 16 animation types
let bumprDurationModes = ['original' × 10]; // original, loop, musical, timed
let bumprModeActive = false;               // Keyboard mode
let bumprSelectedSlot = 0;                 // Currently selected
let bumprPlaying = false;                  // Video playing
let bumprQueuedTimeout = null;             // Pending quantized trigger
let bumprActiveVideoEl = null;             // Current video element
```

**Specifications:**
| Aspect | Value |
|--------|-------|
| Content | WebM VP9 with alpha channel only |
| Slots | 10 total |
| Location | ADMIN → Settings → BumpR StickRs |
| Folder | `/BumpRStickRs/` in application folder |
| Files | Copied (not moved) on assignment |
| Thumbnails | Data URLs (160×90), black background fill |

**Animation Types (16 total):**
```javascript
const BUMPR_ANIMATIONS = [
  'none', 'bounce', 'breathe', 'crazy', 'drift', 'drop', 'float', 'glitch',
  'pop', 'pulse', 'shake', 'spin', 'spiral', 'strobe', 'wobble', 'zoom'
];
```

**Duration Types:**
```javascript
const BUMPR_DURATIONS = [
  'original',  // Play once, full length
  'loop',      // Loop continuously
  '1/4', '1/2', '1', '2', '4', '8',  // Musical (bars)
  '1s', '2s', '3s', '5s', '10s'      // Timed (seconds)
];
```

**Timing Sources:**
| Tab | Audio State | Timing Source |
|-----|-------------|---------------|
| PERFORM | Playing | `trackAudioMasterClock.getCurrentBeat()` |
| PERFORM | Stopped | Message: "Start playback for quantized timing" |
| CREATE | Playing | `(audioElement.currentTime - audioStartTime) * beatsPerSecond` |
| CREATE | Stopped | Message: "Start audio for quantized timing" |

**CDJ Cue Behavior:**
- Keys always fire (no toggle)
- Pressing any slot stops current playback and immediately plays new slot
- Matches professional DJ cue button behavior

---

### 8. Cue Points System (V5_2_0)

**Purpose:** CDJ-style hot cue system for quick navigation

**Key Components:**
```javascript
let audioCuePoints = [0, 64, 128, 192, 256];  // V5_2_0: 5 cues (was 3)
let draggingCueIndex = -1;          // Currently dragging (-1 = none)
let dragStartX = 0;                 // Mouse start position
let cueDragged = false;             // Whether drag occurred (vs click)
const CUE_HIT_TOLERANCE = 20;       // Pixels for click detection
const NUM_CUE_POINTS = 5;           // V5_2_0: Constant for cue count
```

**Default Positions:**
| Cue | Bar.Beat | Beats | Typical Use |
|-----|----------|-------|-------------|
| 1 | 1.1 | 0 | Intro |
| 2 | 17.1 | 64 | First drop |
| 3 | 33.1 | 128 | Second drop |
| 4 | 49.1 | 192 | Third section |
| 5 | 65.1 | 256 | Fourth section |

**CREATE Tab Features:**
- Visual flags: Orange (#ffa500) line + triangle + number
- Draggable: Click and drag to reposition
- Grid snap: Snaps to bar (4 beats)
- Click to jump: Click flag (no drag) jumps playhead

**PERFORM Tab Features:**
- Display: Orange flags on track waveforms
- Keyboard jump: 1/2/3/4/5 keys jump to cue on selected track

**Storage:**
```json
{
  "audioCuePoints": [0, 64, 128, 192, 256]
}
```

**Migration (V5_2_0):**
```javascript
// Pad legacy 3-cue containers to 5
if (container.audioCuePoints && container.audioCuePoints.length === 3) {
    container.audioCuePoints.push(192, 256);
}
```

---

## DOM ELEMENTS & CONTROLS

### Tab System

**Tab Buttons (V4_5_0: 4-Tab Workflow):**
```javascript
controlsTab     // [CREATE] - shader creation (CYAN - creation)
libraryTab      // [ORGANIZE] - .oms management (YELLOW - organization)
liveTab         // [PERFORM] - performance mixer (MAGENTA - performance)
adminTab        // [ADMIN] - settings/utilities/about (GREY - administration)
```

**Tab Content:**
```javascript
controlsTabContent   // CREATE tab content (includes code view toggle)
libraryTabContent    // ORGANIZE tab content
liveTabContent       // PERFORM tab content
adminTabContent      // ADMIN tab content (nav items: Settings, Utilities, About)
```

**Keyboard Navigation:**
- **NOT IMPLEMENTED** (documented in Release Notes but not in code)
- Planned: Ctrl+1 → CREATE, Ctrl+2 → ORGANIZE, Ctrl+3 → PERFORM, Ctrl+4 → ADMIN

**Tab Color Coding (V4_5_0):**
- **Cyan:** Creation (CREATE)
- **Yellow:** Organization (ORGANIZE)
- **Magenta:** Performance (PERFORM)
- **Grey:** Administration (ADMIN)

**Default Tab:** CREATE (Controls view)

**Code View Toggle (V4_5_0):**
- [Code] button in CREATE tab header
- Toggles between Controls view and Code view
- Single `shaderCode` textarea (source of truth)
- No separate W tab

---

### Controls Tab Elements

**Shader Controls (Visual Timing) - V5_1_0 ID Renames:**
```javascript
// V5_1_0: Shader Name Dropdown
shaderNameDropdown    // Select - pick shader from Presets/Library
                      // Values: _current, _setfolder, sysPreset:*, userPreset:*, library:*

shaderTempo           // V5_1_0: Shader Tempo (was `tempo`) (60-200 BPM)
                      // NOT audio BPM - visual animation speed only
                      // V4_8_0: DISABLED when audio loaded (grayed out)
                      // Audio tempo becomes THE tempo when audio present

shaderTempoLinked     // V5_1_0: Link button (chains shader tempo to audio tempo)
                      // Default: true (linked)
                      // Toggle: Click to unlink for independent timing
                      
shaderRenderDuration  // V5_1_0: MM:SS.mmm format (was `renderDuration`)
                      // Determines render length for pure shader mode

shaderLoopLength      // V5_1_0: Loop length display (was `loopLength`)
shaderLoopLengthDropdown // V5_1_0: OmsDropdown instance

// V4_6_0: Orientation (moved from Render Settings)
orientation           // Select: landscape, portrait, square
                      // Fundamental to shader - affects canvas aspect ratio

// Modulators (A, B, C)
modAWave              // Wave type: sine, square, triangle
modASubdiv            // Subdivision: 1-16
modBWave              
modBSubdiv            
modCWave              
modCSubdiv            

shaderCode            // Textarea - WGSL fragment shader code
messageBar            // Status display - compile messages
```

**Audio Controls (Musical Timing):**
```javascript
// V4_6_0: Header Controls
audioSwapBtn          // [Swap] - Opens audio picker dropdown
audioRemoveBtn        // [X] - Removes current audio
audioSwapDropdown     // Select - pick audio from Library (absolute positioned)

audioFile             // File input <input type="file">
audioFileName         // Display - selected filename

audioTempoField       // Audio Tempo (separated from Shader Tempo)
                      // Used for beatgrid alignment and time-stretching
                      
audioStartField       // Manual beatgrid start (seconds)
audioEndField         // Manual beatgrid end (seconds)

// V4_6_0: Loop fields save to project defaults
inBeat                // Loop In point (bar.beat format: "1.1")
outBeat               // Loop Out point (bar.beat format: "65.1")

// V5_1_0: Cue Points
audioCuePoints        // V5_2_0: Array of 5 beat positions [0, 64, 128, 192, 256]

inLabel               // Clickable label - jump to In point
outLabel              // Clickable label - jump to Out point
```

**Audio Reactivity (V4_8_0 Knob Controls):**
```javascript
// 9 rotary knobs (30×30 canvas each) - Gain/Freq1/Freq2 per modulator

// Modulator A (Bass)
modAGainKnob          // 0.0-1.0, default 0.1, linear
modAFreq1Knob         // 20-24000 Hz, log scale, disabled when no audio
modAFreq2Knob         // 20-24000 Hz, log scale, disabled when no audio

// Modulator B (Mids)
modBGainKnob          // 0.0-1.0
modBFreq1Knob         // 20-24000 Hz
modBFreq2Knob         // 20-24000 Hz

// Modulator C (Treble)
modCGainKnob          // 0.0-1.0
modCFreq1Knob         // 20-24000 Hz
modCFreq2Knob         // 20-24000 Hz

// Freq knob behavior:
// - Disabled (dimmed) when no audio loaded
// - Push behavior: Freq1 can't exceed Freq2
// - Log scale for natural frequency perception
```

**Focus Mode State (V5_1_0 Renames):**
```javascript
// Reactivity Focus Mode (was Reactivity Mode)
shaderReactivityFocus     // boolean - keyboard mode active (was modReactivityMode)
shaderReactivitySelectedRow  // 0-2 (A/B/C) (was modSelectedRow)
shaderReactivitySelectedCol  // 0-4 (Subdiv/Waveform/Gain/Freq1/Freq2) (was modSelectedCol)

// Transform Focus Mode
shaderTransformFocus      // boolean (was transformKeyboardMode)
shaderTransformSelectedCol  // 0-4 (was transformSelectedCol)

// Music Timing Focus Mode (NEW V5_1_0)
shaderMusicTimingFocus    // boolean
shaderMusicTimingSelectedCol  // column selection

// Progressive acceleration
modKeyRepeatCount         // Counter for held key
modLastKeyTime            // Timestamp for acceleration
freqKnobsDisabled         // boolean - true when no audio loaded
```

**Transform Animation Modes (V5_1_0):**
```javascript
let transformModes = {
    rotation: 'stop',     // stop, clockwise, counterclockwise, pingpong
    scale: 'stop',        // stop, clockwise, counterclockwise, pingpong
    tile: 'stop'          // stop, clockwise, counterclockwise, pingpong
};

// Mode descriptions:
// stop - No animation, manual control only
// clockwise - Continuous increase
// counterclockwise - Continuous decrease  
// pingpong - Beat-synced oscillation (uses shaderLoopLength)
```

**Transport Controls:**
```javascript
playBtn               // Play/Pause toggle
loopBtn               // Loop enable/disable toggle
uploadBtn             // Upload audio file
removeBtn             // Remove loaded audio
stopBtn               // (Deprecated - may be hidden)
```

**Container Output (V5_1_0 Rename from "OUTPUT"):**
```javascript
// Section header: "CONTAINER OUTPUT"
// Hover message: "Container = Shader + Audio + Settings"

filename              // Output filename (no extension)
                      // Placeholder: "Container Name"
                      // V4_6_0: Auto-populated from cleanAudioName() + shader name
                      
resolution            // Select: 720p, 1080p, 1440p, 2160p
fps                   // Select: 24, 30, 60
// orientation moved to SHADER CONTROLS (V4_6_0)

// V5_1_0: Separate Save/Update buttons
saveBtn               // Save as NEW container to Library
updateBtn             // Update currently loaded container (hidden until load)

renderBtn             // [R] Start render button
```

**Display Elements:**
```javascript
canvas                // WebGPU render target (main preview)
waveformCanvas        // Audio waveform display
progressOverlay       // Render progress modal
progressText          // Progress percentage display
gridFreeBtn           // Toggle beatgrid overlay (future)
```

**Critical Separation (V5_1_0 Terms):**
```
shaderTempo    → Shader Tempo (visual animation speed)
audioTempoField → Audio Tempo (musical timing/beat positioning)

These are NEVER the same value!
- Shader can animate at 128 BPM
- Audio track can be 124 BPM
- Both independent, both valid

V4_8_0: When audio is loaded, shader tempo is DISABLED
- Audio tempo becomes THE tempo (single source of truth)
- Shader tempo field grayed out, tooltip explains why
- Re-enabled when audio removed

V5_1_0: shaderTempoLinked
- When true: shader tempo auto-follows audio tempo
- When false: independent control
```

---

### Library Tab Elements (V4_5_0)

**Toolbar:**
```javascript
// Search
librarySearchInput        // Search/filter input field

// Action Buttons
loadBtn                   // [L] Load container (Cmd+L)
updateBtn                 // [U] Update container (Cmd+U)
exportBtn                 // [E] Export container (Cmd+E)
addBtn                    // [+] Add current to library

// View Controls
addSetBtn                 // [+Set] Create new set (future)
listViewBtn               // [≡] List view mode
gridViewBtn               // [▦] Grid view mode (future)
infoBtn                   // [ℹ] Toggle info drawer (Cmd+I)
deleteBtn                 // [X] Delete with confirmation
```

**Sets Panel (V5_3_0: Unified System):**
```javascript
// Panel headers (clickable for focus)
setsPanelHeader               // "Sets" - click to focus sets navigation
containersPanelHeader         // "Containers" - click to focus containers navigation

// Set management
addSetBtn                     // [+ Set] Create new manual set
saveSmartSetBtn               // [Save] Save current search as smart set (appears during search)

// Set types:
// - Manual set: no indicator, explicit container membership
// - Smart set: cyan dot indicator, auto-updates from query

// Set filter options
'presets'                 // Preset containers only
'all'                     // All Library containers
'allaudio'                // V4_6_0: Containers with audio (hasAudio || audioFilename)
```

**Info Drawer:**
```javascript
infoDrawer                // Slide-out panel (280px)
infoNameField             // Container name (V4_6_0: click-to-edit)
infoTempoField            // V4_6_0: Tempo value (was infoBpmField)
infoAudioDropdown         // Audio file dropdown
infoSwapBtn               // Swap audio button
infoRemoveBtn             // Remove audio button
infoDurationField         // Duration display
infoLoopInField           // Loop In (bar.beat)
infoLoopOutField          // Loop Out (bar.beat)
infoCreatedField          // Created date

// V5_3_0: Tags section
infoTagsContainer         // Container for tag chips
infoTagInput              // Text input for new tags
// Chip behavior: Enter/comma creates, × removes, Backspace removes last

infoOkBtn                 // Apply changes
infoCancelBtn             // Discard changes
infoCloseBtn              // Close drawer (X)
```

**Container Cards (V4_6_0):**
```javascript
card.classList.add('loaded')     // Yellow border/tint
card.classList.add('selected')   // Cyan border
// V4_6_0: Click-to-edit container names in cards
// V4_6_0: Tempo column header (was BPM)
```

**ADMIN Tab (V5_1_0 - Alphabetized Navigation):**
```javascript
adminNavItems             // Application, BumpR StickRs, Create, Organize, Perform
adminContent              // Right panel dynamic content
```

---

### LIVE Tab Elements

**Performance Grid:**
```javascript
// 4 tracks × 8 scenes = 32 cells
performanceGrid              // Container div
gridCells[track][scene]      // Cell divs (indexed 0-3, 0-7)

// Cell properties
cell.dataset.track           // 0-3
cell.dataset.scene           // 0-7
cell.dataset.containerName   // Loaded container name
cell.classList.add('loaded') // Has container
cell.classList.add('selected') // Currently selected
cell.classList.add('playing')  // Currently playing
```

**Track Control Modules (4 tracks):**
```javascript
// Per Track (1-4):
trackShaderOpacityKnob[track]    // 30px encoder (0-100%)
trackShaderMuteBtn[track]        // Purple button
trackShaderSoloBtn[track]        // Cyan button

trackAudioVolumeKnob[track]      // 30px encoder (0-100%, V5_3_0: dB scale)
trackAudioMuteBtn[track]         // Orange button
trackAudioSoloBtn[track]         // Green button

// V5_3_0: Peak Meters
trackPeakMeterCanvas[track]      // 4px wide vertical meter
// Colors: Green (-48 to -18dB), Yellow (-18 to -6dB), Red (-6 to 0dB)
```

**MAIN Column (V5_4_0 - renamed from Master Module):**
```javascript
// Scene Launch Buttons (8 total)
sceneLaunchBtn[scene]           // Launch all tracks in scene
// V5_4_0: Customizable labels and colors
sceneLabels[scene]              // Text labels (up to 128 chars)
sceneColors[scene]              // Color preset or null

// MAIN Visual Section
trackMainShaderOpacityKnob      // Opacity knob
trackMainShaderMuteBtn          // [M] button (purple)
mainShaderMeterCanvas           // Purple meter bar
mainShaderOpacityReadout        // % display

// MAIN Audio Section
trackMainAudioVolumeKnob        // Volume knob
trackMainAudioMuteBtn           // [M] button (orange)
mainAudioMeterCanvasL           // Left meter bar
mainAudioMeterCanvasR           // Right meter bar
mainAudioPeakReadout            // dB display

// Bottom Controls
masterClockDisplay              // Bar.Beat.Tick
dspBtn                          // [DSP] button (opens slideout)
vfxBtn                          // [VFX] button (opens slideout)
quantizeDropdown                // Options: 'off', 'beat', 'bar' (default)
zoomKnob                        // Zoom control
tempoKnob                       // Master Tempo (60-200 BPM)
masterGridBtn                   // [M] toggle
trackGridBtn                    // [T] toggle
waveformBtn                     // [W] toggle
stopAllBtn                      // ■ button (16×16)

// V5_4_0: Meter Toggle
meterToggleBtn                  // Split-cube button (purple/cyan gradient)
meterOverlay                    // Per-track meter overlay container
```

**DSP Slideout (V5_4_0):**
```javascript
dspSlideout                     // Slideout panel
dspPresetDropdown               // Preset selector
dspThresholdKnob                // -48 to 0 dB
dspRatioKnob                    // 1:1 to 20:1
dspOutputKnob                   // -12 to +12 dB
dspGRMeterCanvas                // Gain reduction meter
```

**VFX Slideout (V5_4_0):**
```javascript
vfxSlideout                     // Slideout panel
vfxEffectDropdown               // Effect selector (OmsDropdown)
vfxAmountKnob                   // 0-100%
```

**Waveform Panel:**
```javascript
// Per Track (1-4):
trackWaveformCanvas[track]      // Canvas element
trackWaveformOverlay[track]     // Container name + thumbnail

// V5_2_0: Track Audio Modules (replaced nudge buttons)
trackAudioModule[track]         // Container for cue/EQ/FX controls
trackCueButtons[track][1-5]     // 5 cue buttons per track
trackEQKnobs[track][H/M/L]      // 3 EQ knobs per track
trackFXKnobs[track][W/D/B]      // 3 FX knobs per track

// Fixed Overlay
playheadLine                    // Magenta vertical line (z-index 10)
masterGrid                      // White bars overlay (z-index 5) - NOT IMPLEMENTED
```

**BumpR Video Overlays (V5_1_0):**
```javascript
// Video elements for overlay playback
bumprOverlayVideo               // <video> in preview-container (CREATE)
bumprOverlayVideoLive           // <video> in liveShaderOutputContainer (PERFORM)

// Slideout panel
bumprSlideout                   // 80px panel inside preview-container
bumprSlotRow[0-9]               // Slot rows with thumbnails
```

**FX Overlay (V5_5_0, V5_6_0 Per-Track):**
```javascript
// Orange FX button in MAIN column
fxOverlayBtn                    // [FX] button (orange, opens slideout)
fxOverlayPanel                  // 120px slideout panel

// Per-track modules (4 tracks × 3 rows) - V5_6_0
trackVFXKnobs[track][slot]      // VFX knobs: A, B, C (20×20, purple arcs)
trackRGBKnobs[track][channel]   // RGB knobs: R, G, B (20×20, cyan arcs) - V5_6_0
trackFXKnobs[track][slot]       // FX knobs: R, D, W (20×20, orange arcs)
// VFX: Assigned from vfxPool (10 effects)
// RGB: R=Red, G=Green, B=Blue gain (0.75=unity)
// FX: Assigned from fxPool (10 effects)
```

**PIP Camera (V5_5_0, V5_6_0 Position Dropdown):**
```javascript
// Camera overlay (all tabs)
cameraOverlay                   // Full-screen video overlay
cameraOverlayVideo              // <video> element

// PIP containers
pip1Container                   // PIP 1 slideout (PERFORM tab only)
pip1Video                       // <video> element
pip1Border                      // Colored border element

pip2Container                   // PIP 2 corner (CREATE/ORG/ADMIN)
pip2Video                       // <video> element
pip2Border                      // Colored border element

pip3Container                   // PIP 3 center (CREATE/ORG/ADMIN)
pip3Video                       // <video> element
pip3Border                      // Colored border element

// Color picker popup
pipColorPicker                  // 9-swatch popup
pipColorSwatches[color]         // Individual color buttons
```

**ADMIN Tab - Video Sources (V5_5_0, V5_6_0):**
```javascript
// Video Sources section
videoSourceDeviceDropdown       // Camera device selector (OmsDropdown)
videoSourceMirrorToggle         // Mirror mode checkbox
testCameraBtn                   // Test camera button
copyDiagnosticBtn               // Copy diagnostic report button

// PIP Configuration (V5_6_0: 3 dropdowns per PIP)
pip1ShapeDropdown               // PIP 1 shape selector
pip1ColorDropdown               // PIP 1 color selector
pip1PositionDropdown            // PIP 1 position selector (V5_6_0)
pip2ShapeDropdown               // PIP 2 shape selector
pip2ColorDropdown               // PIP 2 color selector
pip2PositionDropdown            // PIP 2 position selector (V5_6_0)
pip3ShapeDropdown               // PIP 3 shape selector
pip3ColorDropdown               // PIP 3 color selector
pip3PositionDropdown            // PIP 3 position selector (V5_6_0)
```

**ADMIN Tab - Effect Pool Configuration (V5_6_0):**
```javascript
// VFX Pool Assignments
vfxSlotADropdown                // VFX slot A effect selector
vfxSlotBDropdown                // VFX slot B effect selector
vfxSlotCDropdown                // VFX slot C effect selector

// Audio FX Pool Assignments
fxSlotRDropdown                 // FX slot R effect selector
fxSlotDDropdown                 // FX slot D effect selector
fxSlotWDropdown                 // FX slot W effect selector
```

**ADMIN Tab - Unified Mic System (V5_6_0):**
```javascript
// Mic Input section (under Audio Card)
micDeviceDropdown               // Mic input device selector (OmsDropdown)
testMicBtn                      // Test mic input button
micLevelMeterCanvas             // Real-time mic level display
```

---

### Keyboard Shortcuts (CREATE Tab) (V5_2_0, V5_6_0 Updates)

**Global (V5_6_0):**
| Shortcut | Action |
|----------|--------|
| R | Start render (V5_6_0: bare key) |
| T | Quick timeline test (V5_6_0: bare key) |

**Audio Navigation:**
| Shortcut | Action |
|----------|--------|
| Spacebar | Play/pause audio |
| Trackpad L/R | Smooth scrub |
| Shift+< | Jump to previous grid line (4 bars) |
| Shift+> | Jump to next grid line (4 bars) |
| G | Toggle beatgrid |

**Audio Focus Mode (V5_2_0):**
| Shortcut | Action |
|----------|--------|
| Shift+A | Enter/exit Audio Focus mode |
| V | Open Volume panel |
| C | Open Cue panel (5 cues) |
| E | Open EQ panel (H/M/L) |
| F | Open FX panel (W/D/B) |
| ↑/↓ | Navigate rows in panel |
| ←/→ | Adjust values |
| Enter | Activate (jump/kill) |
| Escape | Exit panel or mode |

**Reactivity Focus Mode:**
| Shortcut | Action |
|----------|--------|
| Shift+R | Enter/exit reactivity focus mode |
| ←/→ | Navigate columns (Subdiv/Waveform/Gain/Freq1/Freq2) |
| ↑/↓ | Navigate rows (A/B/C) |
| Shift+← | Decrease value |
| Shift+→ | Increase value |
| , (comma) | Mute selected mod (V5_6_0) |
| . (period) | Solo selected mod (V5_6_0) |
| N | Toggle Mod Tuning panel (V5_3_0) |
| Escape | Exit mode |

**Transform Focus Mode (V5_6_0 Redesign):**
| Shortcut | Action |
|----------|--------|
| Shift+T | Enter/exit transform focus mode |
| 1-5 | Select control (1=Rotation, 2=Scale, 3=Tile, 4=XY, 5=Pivot) |
| 0 | Stop animation mode |
| 1 | CW/Grow/Increase mode (when control selected) |
| 2 | CCW/Shrink/Decrease mode |
| 3 | PingPong mode |
| Enter | Freeze/Resume animation |
| Enter×2 | Reset to default (double-tap) |
| Trackpad scroll | Adjust animation speed (0.03×-256×) |
| Escape | Exit mode |

**Music Timing Focus Mode (V5_1_0):**
| Shortcut | Action |
|----------|--------|
| Shift+M | Enter/exit music timing focus mode |
| ←/→ | Navigate columns |
| Shift+← | Decrease value |
| Shift+→ | Increase value |
| Escape | Exit mode |

**BumpR StickRs Mode (V5_1_0):**
| Shortcut | Action |
|----------|--------|
| Shift+B | Enter/exit BumpR mode |
| 1-9, 0 | Trigger slots 1-10 |
| ↑/↓ | Navigate slots |
| Enter | Trigger selected slot |
| Escape | Exit mode |

**PIP Camera (V5_5_0):**
| Shortcut | Action |
|----------|--------|
| O | Toggle camera overlay |
| P | Toggle PIP 2 (corner) |
| Shift+P | Toggle PIP 3 (center) |

**Note:** Hold Shift+Arrow for progressive acceleration (faster value changes).

---

### Keyboard Shortcuts (PERFORM Tab) - V5_4_0, V5_5_0, V5_6_0 Updates

**Note:** In V4_5_0, this tab is labeled "PERFORM" but internal references remain "live" for code continuity.

**Pattern:** All keyboard actions trigger `showMessage()` with state feedback.

**Safety (V5_4_0):**
| Key | Action |
|-----|--------|
| Esc | First press: Warning message "Press ESC again to stop all" |
| Esc Esc | Double-tap within 500ms: Stop All |

**Track Selection (V5_5_0, V5_6_0 - Bare Keys):**
| Key | Action |
|-----|--------|
| 1 | Select track 1 (no modifier!) |
| 2 | Select track 2 |
| 3 | Select track 3 |
| 4 | Select track 4 |
| 0 | Deselect track |
| Shift+5 | Select MAIN |

**V5_5_0 Change:** Bare 1-4 keys now select tracks (was Shift+1-4). Cue jumping moved to C panel workflow.

**Quick-Load Workflow (V5_5_0):**
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate library (when track selected) |
| SHIFT+Right | Load to next empty cell (scan order: T1S1→T2S1→T3S1→T4S1, then S2...) |
| SHIFT+Left | Return focus to library |
| Enter | Load selected container to cursor |

**Overlay Toggles (V5_6_0):**
| Key | Action |
|-----|--------|
| E | Toggle EQ overlay for selected track |
| F | Toggle FX overlay for selected track |
| V | Toggle VFX overlay for selected track |
| A | Toggle All overlays off |

**Panel Access (V5_2_0):**
| Key | Action |
|-----|--------|
| C | Open Cue panel for selected track |

**PIP Camera (V5_5_0):**
| Key | Action |
|-----|--------|
| O | Toggle camera overlay (all tabs) |
| P | Toggle PIP 1 slideout |

**Categories:**
- Safety: `Esc` (double-tap) (V5_4_0)
- Track Selection: `1-4`, `0` (V5_5_0: bare keys)
- Quick-Load: `SHIFT+Right/Left` (V5_5_0)
- Overlay Toggles: `E/F/V/A` (V5_6_0)
- Panel Access: `C` (V5_2_0)
- PIP: `O`, `P` (V5_5_0)
- Grid Navigation: `W/A/S/D`
- Playback: `Enter`, `Space`, `Delete`
- Mixer: Arrow keys, `,` `.` `<` `>`
- Visibility: `Shift+M/T/W`
- Zoom: `[ ]`
- BumpR: `Shift+B`, then `1-0`, `↑/↓`, `Enter`

**Complete list:** See User Guide section 3.4.12

---

### Keyboard Shortcuts (Library Tab) - V5_5_0 Updates

**Pattern:** Cmd+key for actions, arrow keys for navigation.

**Apple Music-Style Auto-Save (V5_5_0):**
Changes save automatically when navigating away - no OK/Cancel buttons.

**Categories:**
- Load/Update: `Cmd+L`, `Cmd+U`
- Swap: `Cmd+S`, `Cmd+A`
- Navigation: `↑/↓`, `Enter`
- Info: `Cmd+I`

**Complete list:** See User Guide section 3.3.11
| Key | Action |
|-----|--------|
| ↑/↓ | Navigate list (auto-saves drawer) |
| Enter | Quick rename selected (V5_5_0) |
| Cmd+I | Toggle info drawer |
| Delete | Delete (with confirmation) |
| Escape | Save and close drawer (V5_5_0) |

**PIP Camera (V5_5_0):**
| Key | Action |
|-----|--------|
| O | Toggle camera overlay |
| P | Toggle PIP 2 (corner) |
| Shift+P | Toggle PIP 3 (center) |

---

### Keyboard Shortcuts (ADMIN Tab) - V5_4_0, V5_5_0

**Section Navigation (V5_4_0):**
| Key | Action |
|-----|--------|
| ↑ | Previous section (wraps from first to last) |
| ↓ | Next section (wraps from last to first) |

**BumpR Mode (V5_1_0):**
| Key | Action |
|-----|--------|
| Shift+B | Enter/exit BumpR keyboard mode |
| 1-9, 0 | Trigger slots 1-10 (when in BumpR mode) |
| ↑/↓ | Navigate slots (when in BumpR mode) |
| Enter | Trigger selected slot |
| Escape | Exit BumpR mode |

**PIP Camera (V5_5_0):**
| Key | Action |
|-----|--------|
| O | Toggle camera overlay |
| P | Toggle PIP 2 (corner) |
| Shift+P | Toggle PIP 3 (center) |

---

## DATA STRUCTURES

### Container Data Object

**Structure (V5_1_0):**
```javascript
containerData = {
  // Metadata
  name: "Bass Drop Tunnel",
  version: "5.1.0",
  created: "2025-12-24T10:30:00Z",
  
  // Shader
  shaderCode: "/* WGSL code */",
  tempo: 128,
  
  // Modulators
  modAWave: "sine",
  modASubdiv: 4,
  modBWave: "square",
  modBSubdiv: 8,
  modCWave: "triangle",
  modCSubdiv: 16,
  
  // Audio Reference (optional)
  audioReference: {
    filename: "bass_track.mp3",
    audioTempo: 124,
    audioStartTime: 17.1,
    audioEndTime: 300.0,
    inBeat: "1.1",
    outBeat: "65.1",
    bassGain: 1.0,
    midGain: 1.0,
    trebleGain: 1.0
  },
  
  // Audio Loop State (V4.4.0)
  audioLoopEnabled: true,        // Loop on/off state
  
  // Cue Points (V5_2_0: expanded to 5)
  audioCuePoints: [0, 64, 128, 192, 256],  // 5 beat positions
  
  // Control States (V5_1_0)
  controlStates: {
    shaderTempoLinked: true,
    // ... other states
  },
  
  // Render Settings
  renderSettings: {
    resolution: "1080p",
    fps: 60,
    orientation: "landscape",
    filename: "bass_drop"
  },
  
  // Thumbnail
  thumbnail: "data:image/png;base64,..."
};
```

---

### Library Cache (V4.3.0_A)

**Purpose:** Full container data cached from folder scan

**Structure:**
```javascript
let libraryContainersCache = [
    {
        // From filename
        id: "Bass Drop Tunnel",
        filename: "Bass Drop Tunnel.oms",
        
        // Full container data (from .oms file)
        name: "Bass Drop Tunnel",
        shaderCode: "/* WGSL code */",
        controlStates: {...},
        shaderModATimeline: [...],
        shaderModBTimeline: [...],
        shaderModCTimeline: [...],
        audioTempo: 128,
        audioFilename: "bass_track.mp3",
        audioStartTime: 17.1,
        audioEndTime: 300.0,
        audioCuePoints: [0, 64, 128, 192, 256],  // V5_2_0: 5 cues
        tags: ["bass", "128", "drop"],            // V5_3_0: Container tags
        modTuning: {...},                         // V5_3_0: Mod tuning settings
        // ... everything from .oms file
    },
    // ... more containers
];
```

**Sort State:**
```javascript
let librarySortBy = 'name';   // 'name', 'bpm', or 'audio' (V4_4_0: added 'audio')
let librarySortDir = 'asc';   // 'asc' or 'desc'
```

**Sets System (V4.4.0):**
```javascript
let selectedLibrarySetId = 'presets';    // 'presets' or 'all'
let selectedLiveSetId = 'presets';       // Synced with Library
let userPresetsCache = [];               // From /Presets/ folder
```

**Access Pattern:**
```javascript
// Get all containers (LIVE tab uses this)
const containers = getLibraryContainers();  // Returns libraryContainersCache

// Find specific container
const container = containers.find(c => c.id === containerId);

// Access full data
const shaderCode = container.shaderCode;
const timelines = container.shaderModATimeline;
const cuePoints = container.audioCuePoints;  // V5_1_0
const tags = container.tags;                  // V5_3_0
```

---

### Unified Sets Cache (V5_3_0)

**Purpose:** Combined manual and smart sets from /Sets/ folder

**Structure:**
```javascript
let setsCache = [
    {
        id: "Bass Collection",
        filename: "Bass Collection.oms-set",
        setType: "manual",            // 'manual' or 'smart'
        smartQuery: null,             // Query string if smart set
        containerIds: ["Container1", "Container2"],  // Manual: explicit list
        containerCount: 12,
        created: "2025-12-27T10:00:00Z"
    },
    {
        id: "128 BPM",
        filename: "128 BPM.oms-set",
        setType: "smart",
        smartQuery: "128",            // Auto-matches containers
        containerIds: [],             // Smart: calculated on demand
        containerCount: 8,            // Recalculated on refresh
        created: "2025-12-27T11:00:00Z"
    }
];

// Sets folder handle
let setsFolderHandle = null;          // /Sets/ subfolder
```

**Set Type Indicators:**
```javascript
// In ORGANIZE tab Sets panel
// Cyan dot = smart set (auto-updates)
// No dot = manual set (explicit membership)
```

---

### Container Tags (V5_3_0)

**Purpose:** Multi-tag organization system per container

**Structure:**
```javascript
// Tags stored in each .oms file
container.tags = ["bass", "128", "drop"];

// Global tag index (rebuilt on library refresh)
let allTagsIndex = {
    "bass": ["Container1", "Container2", "Container5"],
    "128": ["Container1", "Container3"],
    "drop": ["Container2", "Container4"]
};

// Tag normalization
function normalizeTag(tag) {
    return tag.trim().toLowerCase();
}

// Prevent duplicates (case-insensitive)
function addTag(container, newTag) {
    const normalized = normalizeTag(newTag);
    if (!container.tags.some(t => normalizeTag(t) === normalized)) {
        container.tags.push(newTag);
    }
}
```

---

### Track State Arrays (LIVE Tab)

**Shader State:**
```javascript
// Per-track shader controls (indices 0-3)
trackShaderOpacity = [100, 100, 100, 100];     // 0-100%
trackShaderMute = [false, false, false, false];
trackShaderSolo = [false, false, false, false];
trackShaderManual = [false, false, false, false]; // AUTO vs MANUAL mode
```

**Audio State:**
```javascript
// Per-track audio controls (indices 0-3)
trackAudioVolume = [75, 75, 75, 75];           // 0-100%
trackAudioMute = [false, false, false, false];
trackAudioSolo = [false, false, false, false];
```

**Playback State:**
```javascript
// Per-track playback (indices 0-3)
trackAudioPlaying = [false, false, false, false];
trackAudioStartTimes = [null, null, null, null]; // audioContext.currentTime when started
trackAudioSources = [null, null, null, null];    // AudioBufferSourceNode instances
```

**Loaded Containers:**
```javascript
// Grid state (4x8 = 32 cells)
gridState = {
  cells: [
    [null, null, null, null, null, null, null, null], // Track 1, Scenes 1-8
    [null, null, null, null, null, null, null, null], // Track 2, Scenes 1-8
    [null, null, null, null, null, null, null, null], // Track 3, Scenes 1-8
    [null, null, null, null, null, null, null, null]  // Track 4, Scenes 1-8
  ],
  
  // Each cell contains:
  // {
  //   containerName: "Bass Drop Tunnel",
  //   containerData: {...},
  //   audioBuffer: AudioBuffer (if audio loaded)
  // }
  
  masterTempo: 120,
  zoomValue: 50, // pixels per second
  selectedTrack: 0,
  selectedScene: 0
};
```

---

### Pixel/Scroll State (LIVE Tab)

```javascript
// Bar 1 pixel positions (for scroll calculation)
trackBar1Positions = [0, 0, 0, 0];  // One per track

// Scroll animation frame IDs
trackScrollAnimationFrames = [null, null, null, null];

// Launch beat positions
trackLaunchBeats = [null, null, null, null];
```

---

### Visibility State (LIVE Tab)

```javascript
let masterGridVisible = false;  // M button - default OFF
let trackGridVisible = true;    // T button - default ON
let waveformVisible = true;     // W button - default ON
```

---

### UI State (V4_5_0)

```javascript
// Code view toggle
let codeViewActive = false;     // Controls view vs Code view

// Info drawer
let infoDrawerOpen = false;     // Info drawer open/closed
let pendingInfoChanges = null;  // Pending changes object

// Delete confirmation
let pendingDeleteId = null;     // Container ID pending deletion
let pendingDeleteTimeout = null; // 3-second timeout for confirmation
```

---

### UI State (V4_6_0)

```javascript
// Pending operation flags (resume after folder setup)
let pendingSaveAfterFolderSetup = false;    // Resume save workflow
let pendingAudioSwapAfterFolderSetup = false; // Resume audio swap dropdown

// Project-level loop defaults (saved to .omsp)
let projectLoopDefaults = {
    loopIn: '1.1',      // Default Loop In (bar.beat)
    loopOut: '17.1'     // Default Loop Out (bar.beat)
};
// Hierarchy: Container (.oms) overrides → Project (.omsp) defaults → Hardcoded
```

---

### UI State (V4_8_0 / V5_1_0 / V5_2_0 Focus Mode)

```javascript
// Reactivity Focus Mode (V5_2_0: renamed from shader* to create*)
let createReactivityFocus = false;     // Was shaderReactivityFocus, was modReactivityMode
let createReactivitySelectedRow = 0;   // Was shaderReactivitySelectedRow (0-2 = A/B/C)
let createReactivitySelectedCol = 0;   // Was shaderReactivitySelectedCol (0-4)
let modKeyRepeatCount = 0;             // Progressive acceleration counter
let modLastKeyTime = 0;                // Timestamp for acceleration

// Freq knob state
let freqKnobsDisabled = true;          // true when no audio loaded

// Current container tracking
let currentLoadedContainerId = null;   // null = new, "name" = editing existing
```

---

### UI State (V5_0_0 / V5_1_0 / V5_2_0)

```javascript
// Transform Focus Mode (V5_2_0: renamed from shader* to create*)
let createTransformFocus = false;         // Was shaderTransformFocus
let createTransformSelectedCol = 0;       // Was shaderTransformSelectedCol
let transformKeyRepeatCount = 0;          // Progressive acceleration counter
let transformLastKeyTime = 0;             // Timestamp for acceleration
let transformLastEnterTime = 0;           // Double-tap Enter detection

// Music Timing Focus Mode
let createMusicTimingFocus = false;       // Was shaderMusicTimingFocus
let createMusicTimingSelectedCol = 0;

// Transform Animation Modes (V5_5_0: Added speed)
let transformModes = {
    rotation: 'stop',     // stop, cw, ccw, pingpong
    scale: 'stop',
    tile: 'stop'
};
let transformSpeeds = {
    rotation: 1.0,        // 0.03 to 256 (exponential)
    scale: 1.0,
    tile: 1.0
};

// V5_5_0: Speed Control
// - Trackpad scroll adjusts speed exponentially
// - Message bar shows: "Rotation: CW @ 2.5× speed"
// - Speed persists to .oms containers

// Shader Tempo Link
let shaderTempoLinked = true;  // Stored in controlStates

// Transform configuration arrays
const TRANSFORM_COL_IDS = ['transformRotation', 'transformScale', 'transformTile', 'transformXY', 'transformPivot'];
const TRANSFORM_KNOB_IDS = ['transformRotationKnob', 'transformScaleKnob', 'transformTileKnob', 'transformXYPad', 'transformPivotPad'];
const TRANSFORM_COL_NAMES = ['Rotation', 'Scale', 'Tile', 'Translate', 'Pivot'];
const TRANSFORM_CONFIGS = [
    { min: 0, max: 360, step: 5, default: 0, endless: true },      // Rotation (degrees)
    { min: 0, max: 4, step: 0.1, default: 1 },                     // Scale
    { min: 1, max: 8, step: 1, default: 1 },                       // Tile
    { min: -1, max: 1, step: 0.05, default: 0 },                   // XY (both axes)
    { min: -1, max: 1, step: 0.05, default: 0 }                    // Pivot (both axes)
];

// V5_2_0: Centralized Focus Mode System
let activeFocusMode = null;      // CORTEX: Current mode
let activeAudioPanel = null;     // CORTEX: Current audio panel

// V5_2_0: Tab isolation helpers
function isCreateTabActive() { return document.getElementById('controlsTabContent').style.display !== 'none'; }
function isPerformTabActive() { return document.getElementById('liveTabContent').style.display !== 'none'; }
```

---

### Cue Points State (V5_2_0)

```javascript
// Cue point positions (beats relative to audioStartTime)
let audioCuePoints = [0, 64, 128, 192, 256];  // V5_2_0: 5 cues (was 3)
// Default: Bar 1.1, 17.1, 33.1, 49.1, 65.1

// Drag state (CREATE tab)
let draggingCueIndex = -1;          // -1 = not dragging
let dragStartX = 0;                 // Mouse start X
let cueDragged = false;             // Whether actual drag occurred

// Constants
const CUE_HIT_TOLERANCE = 20;       // Pixels for click detection
const NUM_CUE_POINTS = 5;           // V5_2_0: Constant for cue count
```

---

### Audio Focus State (V5_2_0)

```javascript
// Focus mode (centralized system)
let activeFocusMode = null;         // null, 'reactivity', 'transform', 'musicTiming', 'audio', 'bumpr'
let activeAudioPanel = null;        // null, 'volume', 'cue', 'eq', 'fx'

// Audio panel selection
let audioFocusSelectedRow = 0;      // Row within active panel

// CREATE tab audio controls
let createAudioVolume = 75;         // 0-100
let createAudioVolumeKilled = false;

// EQ3 state (CREATE tab)
let createEQHigh = 0.75;            // 0-1 (0.75 = unity/0dB)
let createEQMid = 0.75;
let createEQLow = 0.75;
let createEQHighKilled = false;
let createEQMidKilled = false;
let createEQLowKilled = false;

// DJ FX state (CREATE tab)
let createFXWashout = 0;            // 0-1 (wet amount)
let createFXDubby = 0;
let createFXBreak = 0;
let createFXWashoutKilled = false;
let createFXDubbyKilled = false;
let createFXBreakKilled = false;
```

---

### Per-Track Audio State (V5_2_0)

```javascript
// EQ per track (PERFORM tab) - [4 tracks][3 bands]
let trackEQValues = [
    [0.75, 0.75, 0.75],  // Track 1: H, M, L
    [0.75, 0.75, 0.75],  // Track 2
    [0.75, 0.75, 0.75],  // Track 3
    [0.75, 0.75, 0.75]   // Track 4
];
let trackEQKilled = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
    [false, false, false]
];

// FX per track (PERFORM tab) - [4 tracks][3 slots]
let trackFXValues = [
    [0, 0, 0],  // Track 1: Washout, Dubby, Break
    [0, 0, 0],  // Track 2
    [0, 0, 0],  // Track 3
    [0, 0, 0]   // Track 4
];
let trackFXKilled = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
    [false, false, false]
];

// Audio routing nodes per track
let trackEQNodes = [null, null, null, null];     // Each: {high, mid, low}
let trackFXNodes = [null, null, null, null];     // Each: {washout, dubby, break}
```

---

### Mod Tuning State (V5_3_0, V5_5_0 Per-Mod)

```javascript
// Panel state
let modTuningPanelOpen = false;     // Blender-style slide-out panel
let modTuningSelectedMod = 'A';     // V5_5_0: Which mod panel shows (syncs to Reactivity row)

// V5_5_0: Per-modulator envelope parameters (18 total)
// Mod A (Bass) - optimized for kicks
let modAQ = 0.70;                    // Q (0.5-10)
let modAAttack = 5;                  // Attack (0-100ms)
let modADecay = 20;                  // Decay (0-200ms)
let modASustain = 70;                // Sustain (0-100%)
let modARelease = 50;                // Release (0-500ms)
let modAThreshold = 10;              // Threshold (0-100%, was 0-20%)

// Mod B (Mids) - optimized for snares/claps
let modBQ = 0.70;
let modBAttack = 10;
let modBDecay = 30;
let modBSustain = 80;
let modBRelease = 100;
let modBThreshold = 5;

// Mod C (Highs) - optimized for hats/cymbals
let modCQ = 0.70;
let modCAttack = 2;
let modCDecay = 15;
let modCSustain = 60;
let modCRelease = 80;
let modCThreshold = 5;
```

**V5_5_0 Changes from V5_3_0/V5_4_0:**
- ADSR was global (5 params shared) → now per-mod (18 params)
- Threshold range expanded: 0-20% → 0-100%
- N-panel syncs to selected Reactivity row (↑/↓ navigation)
- Live readouts show current values (ms/%)

**Mod Tuning in .oms files (V5_5_0):**
```javascript
container.modTuning = {
    modAQ: 0.70, modAAttack: 5, modADecay: 20,
    modASustain: 70, modARelease: 50, modAThreshold: 10,
    
    modBQ: 0.70, modBAttack: 10, modBDecay: 30,
    modBSustain: 80, modBRelease: 100, modBThreshold: 5,
    
    modCQ: 0.70, modCAttack: 2, modCDecay: 15,
    modCSustain: 60, modCRelease: 80, modCThreshold: 5
};
```

---

### Volume dB Mapping (V5_3_0)

```javascript
// Ableton-style volume mapping: 0-100% → -∞ to +6dB
function volumeToGain(percent) {
    if (percent === 0) return 0;           // -∞ dB (silence)
    if (percent <= 50) {
        // 0-50% maps to 0.0-1.0 (exponential)
        return Math.pow(percent / 50, 2);
    } else {
        // 50-100% maps to 1.0-2.0 (linear boost)
        return 1.0 + (percent - 50) / 50;
    }
}

// Reference points:
// 0% → 0.0× gain (-∞ dB)
// 50% → 1.0× gain (0 dB, unity)
// 100% → 2.0× gain (+6 dB)
```

---

### Peak Meters State (V5_3_0)

```javascript
// Per-track peak meter values (PERFORM tab)
let trackPeakValues = [
    -48,    // Track 1: current dBFS (-48 to 0)
    -48,    // Track 2
    -48,    // Track 3
    -48     // Track 4
];

// Peak meter canvases
let trackPeakMeterCanvases = [null, null, null, null];

// Meter color zones:
// -48 to -18 dB = Green (safe)
// -18 to -6 dB = Yellow (warm)
// -6 to 0 dB = Red (hot)

// Peak detection interval
const PEAK_METER_UPDATE_MS = 50;  // 20 fps for meters
```

---

### BumpR StickRs State (V5_1_0)

```javascript
// File system
let bumprStickrsFolderHandle = null;

// Slot data (10 slots)
let bumprStickrsSlots = [null, null, null, null, null, null, null, null, null, null];
// Each slot: { filename: string, thumbnail: dataURL } or null

// Per-slot settings
let bumprTimingModes = ['instant', 'instant', 'instant', 'instant', 'instant',
                        'instant', 'instant', 'instant', 'instant', 'instant'];
let bumprScaleModes = [100, 100, 100, 100, 100, 100, 100, 100, 100, 100];
let bumprAnimateModes = ['none', 'none', 'none', 'none', 'none',
                         'none', 'none', 'none', 'none', 'none'];
let bumprDurationModes = ['original', 'original', 'original', 'original', 'original',
                          'original', 'original', 'original', 'original', 'original'];

// Keyboard mode
let bumprModeActive = false;
let bumprSelectedSlot = 0;

// Playback state
let bumprPlaying = false;
let bumprQueuedTimeout = null;
let bumprActiveVideoEl = null;
```

---

### OmsDropdown Instances (V5_0_0 / V5_1_0)

```javascript
// Global dropdown instances (11 total in CREATE tab)
window.orientationDropdown           // Orientation (Landscape/Portrait/Square)
window.shaderLoopLengthDropdown      // V5_1_0: Loop Length (was loopLengthDropdown)
window.modASubdivDropdown            // Mod A Subdivision
window.modAWaveformDropdown          // Mod A Waveform
window.modBSubdivDropdown            // Mod B Subdivision
window.modBWaveformDropdown          // Mod B Waveform
window.modCSubdivDropdown            // Mod C Subdivision
window.modCWaveformDropdown          // Mod C Waveform
window.resolutionDropdown            // Resolution (720p-2160p)
window.fpsDropdown                   // FPS (24/30/60)
// Plus shaderNameDropdown (existing)
```

---

### OmsDropdown Component (V4_8_0)

**Purpose:** Reusable custom dropdown with thumbnail support

**Constructor:**
```javascript
const dropdown = new OmsDropdown('containerId', {
    showThumbnails: true,           // Enable shader previews
    thumbnailSize: { w: 32, h: 18 }, // Canvas dimensions
    onChange: (value, item) => { },  // Selection callback
    onOpen: async () => { }          // Lazy refresh callback
});
```

**Methods:**
```javascript
dropdown.setOptions(items)    // Set dropdown options
dropdown.setValue(value)      // Set selected value
dropdown.getValue()           // Get current value
dropdown.open()               // Programmatically open
dropdown.close()              // Programmatically close
```

**Option Structure:**
```javascript
{
    value: 'library:MyShader',    // Unique identifier
    label: 'MyShader',            // Display text
    shaderCode: '/* WGSL */',     // For thumbnail rendering (optional)
    separator: false              // Add visual separator before item
}
```

**Features:**
- Keyboard navigation (↑/↓, Enter, Escape)
- Click outside to close
- Lazy thumbnail rendering on open
- Separator support for grouping options

---

### Waveform Pixel Grid

**Purpose:** Map musical bars to pixel positions

**Structure:**
```javascript
trackAudioPixelGrid = {
  // Container info
  containerName: "Bass Drop Tunnel",
  audioTempo: 124,
  audioStartTime: 17.1,
  audioEndTime: 300.0,
  
  // Calculated values
  duration: 282.9,              // audioEndTime - audioStartTime
  originalSecondsPerBar: 1.935, // (60 / 124) * 4
  numberOfBars: 146.2,          // duration / originalSecondsPerBar
  
  // Master tempo adjusted
  masterSecondsPerBar: 2.0,     // (60 / 120) * 4
  stretchedDuration: 292.4,     // numberOfBars * masterSecondsPerBar
  
  // Pixel calculations
  pixelsPerSecond: 50,          // From zoom value
  pixelsPerBar: 100,            // masterSecondsPerBar * pixelsPerSecond
  totalPixels: 14620,           // stretchedDuration * pixelsPerSecond
  
  // Bar positions (pixel coordinates)
  barPositions: [
    0,      // Bar 1 at 0px (relative to canvas)
    100,    // Bar 2 at 100px
    200,    // Bar 3 at 200px
    // ... one entry per bar
  ],
  
  // Canvas positioning
  canvasWidth: 14620,           // Total pixels needed
  initialTranslateX: -195       // Position so Bar 1 appears at playhead
};
```

---

## DATA FLOW

### Flow 1: Shader-Only Rendering (No Audio)

```
USER INPUT
  ↓
[Shader Controls]
  - shaderTempo (Shader Tempo) - V5_1_0 rename
  - shaderRenderDuration (MM:SS.mmm) - V5_1_0 rename
  - modAWave/Subdiv
  - modBWave/Subdiv
  - modCWave/Subdiv
  - shaderCode (WGSL)
  ↓
[Timing Engine]
  - duration = parse shaderRenderDuration
  - beatCount = (duration * shaderTempo) / 60
  ↓
[Modulator Calculation]
  - For each frame:
    - Calculate time (frame / fps)
    - modA = calculate wave (modAWave, modASubdiv, time, shaderTempo)
    - modB = calculate wave (modBWave, modBSubdiv, time, shaderTempo)
    - modC = calculate wave (modCWave, modCSubdiv, time, shaderTempo)
  ↓
[Shader Rendering]
  - WebGPU compile shader
  - Create render pipeline
  - For each frame:
    - Update uniforms (time, tempo, modA, modB, modC, resolution)
    - Render to canvas
    - Capture frame
  ↓
[Output]
  - MediaRecorder
  - WebM file (VP8 codec)
  - Filename: OMS_[filename]_[resolution]p_[fps]fps_[orientation].webm
```

---

### Flow 2: Audio-Reactive Rendering

```
USER INPUT
  ↓
[Shader Controls]
  - (same as Flow 1)
  ↓
[Audio Controls]
  - audioFile → load AudioBuffer
  - audioTempoField (Audio Tempo)
  - audioStartField/audioEndField (beatgrid)
  - inBeat/outBeat (loop points)
  - audioCuePoints (V5_1_0)
  ↓
[Audio Reactivity]
  - bassGain (modA multiplier)
  - midGain (modB multiplier)
  - trebleGain (modC multiplier)
  ↓
[Timing Engine]
  - duration = (outBeat - inBeat) * 60 / audioTempo
  - beatCount = calculate from loop points
  ↓
[Audio Analysis]
  - FFT → frequency bins
  - Bass level (30-50 Hz)
  - Mid level (50-75 Hz)
  - Treble level (75+ Hz)
  ↓
[Modulator Calculation]
  - For each frame:
    - Calculate base waves (same as Flow 1)
    - modA *= (bassLevel * bassGain)
    - modB *= (midLevel * midGain)
    - modC *= (trebleLevel * trebleGain)
  ↓
[Shader Rendering]
  - (same as Flow 1, but with audio-reactive modulators)
  ↓
[Output]
  - Combine canvas stream + audio stream
  - MediaRecorder
  - WebM file with audio
  - SILENT video (audio pre-baked into reactivity)
```

---

### Flow 3: LIVE Tab Performance

```
SETUP
  ↓
[Load Containers]
  - Drag .oms files to grid cells
  - Each cell: containerData + optional audioBuffer
  ↓
[Master Clock]
  - First launch: master.startContextTime = audioContext.currentTime
  - Never reset
  ↓
PERFORMANCE
  ↓
[Launch Container]
  - Press Enter or double-click cell
  ↓
[Quantization]
  - Calculate current beat: (ctx.currentTime - master.startContextTime) * (bpm/60)
  - Calculate next bar: Math.ceil(beat / 4) * 4
  - Schedule launch: master.startContextTime + (nextBar * 60 / bpm)
  ↓
[Audio Playback]
  - Create AudioBufferSourceNode
  - Set playbackRate: audioTempo / master.bpm
  - source.start(launchTime, audioStartTime)
  - Connect to gain → analyser → destination
  ↓
[Shader Rendering]
  - Compile shader (if not cached)
  - Create render pipeline
  - Start render loop (60fps)
  ↓
[Waveform Display]
  - Position: Bar 1 at playhead (center)
  - Draw: blue amplitude + cyan beat grid + orange cue flags (V5_1_0)
  - Scroll: left at master tempo rate
  ↓
[Animation Loop]
  - For each frame (requestAnimationFrame):
    - Update modulator values
    - Calculate FFT (if audio playing)
    - Update shader uniforms
    - Render all 4 shaders
    - Composite with opacity blending
    - Update waveform scroll positions
  ↓
[Mixer Controls]
  - Volume knobs → GainNode.gain.value
  - Opacity knobs → canvas.style.opacity
  - Mute/Solo buttons → gain/opacity calculations
  - V5_2_0: EQ knobs → BiquadFilterNode.gain
  - V5_2_0: FX knobs → effect wet/dry mix
  ↓
[Per-Track Audio Chain (V5_2_0)]
  Source → EQ (High→Mid→Low) → Washout → [Dubby] → Analyser → Gain → Master
  ↓
[BumpR StickRs (V5_1_0)]
  - Shift+B mode → slot selection
  - 1-0 keys → trigger video overlay
  - Timing: instant/beat/bar quantized
  ↓
[Output]
  - Composited shaders → pop-out window
  - Audio → speakers (via audioContext.destination)
```

---

## FUNCTION REFERENCE

**Total Functions:** ~260 (V5_2_0 adds ~35 new functions for Audio Focus, EQ3, DJ FX)

---

### GENERIC FUNCTIONS (No Domain Prefix)

**Time/Format Utilities:**
```javascript
secondsToTimeString(seconds)
  // Convert seconds to MM:SS.mmm format
  // Returns: string (e.g., "02:15.500")

timeStringToSeconds(timeStr)
  // Convert MM:SS.mmm to seconds
  // Returns: float

beatsToBarBeat(beats)
  // Convert beat number to bar.beat format
  // Returns: string (e.g., "65.3")

barBeatToBeats(barBeatString)
  // Convert bar.beat to beat number
  // Returns: float

parseBPMFromFilename(filename)
  // Extract BPM from filename patterns
  // Returns: integer or null

parseBPMFromID3(file)
  // V4_8_0: Extract BPM from ID3v2 TBPM tag
  // Supports ID3v2.2, v2.3, v2.4 formats
  // Falls back to filename parsing if no tag
  // Returns: Promise<integer>
  // Priority: ID3 tag → filename → default 120

extractShaderName(shaderCode, filename)
  // Extract shader name from code or filename
  // Returns: string

parseShaderNameFromCode(shaderCode)
  // Parse shader name from WGSL code
  // Returns: string
```

**State Management:**
```javascript
getCurrentControlState()
  // Get current control state from UI
  // V5_1_0: Includes shaderTempoLinked
  // Returns: controlStateObject

applyControlState(state)
  // Apply control state to UI fields
  // Args: controlStateObject

getCurrentProjectState()
  // Get current LIVE tab state
  // V5_1_0: Includes BumpR StickRs state
  // Returns: projectStateObject

applyProjectState(projectData)
  // Restore LIVE tab from project state
  // Args: projectStateObject
```

**UI Utilities:**
```javascript
showMessage(text, type, onClick)
  // V4_7_0: Central message bar API (CORTEX)
  // V4_8_0: Added onClick callback for clickable messages
  // Display message in message bar
  // Args: text (string), type ('info', 'state', 'error'), onClick (optional callback)
  // Colors: info=cyan, state=orange, error=magenta

clampLoopToAudioRange()
  // Ensure loop points within audio bounds

calculateMod(beat, subdivision, waveform)
  // Calculate modulator value
  // V4_8_0: Returns 0 for 'none', 1.0 for 'static'
  // Returns: 0.0-1.0
```

**V5_1_0 Shader Tempo Link:**
```javascript
toggleShaderTempoLink()
  // Toggle link between shader tempo and audio tempo
  // Updates shaderTempoLinked state
  // Syncs tempo values when linking

updateShaderTempoLinkUI()
  // Update link button visual (chain icon)
  // Shows linked/unlinked state
```

**V4_8_0 Knob Functions:**
```javascript
initGainKnobs()
  // Initialize 3 gain knob canvases (30×30)
  // Range: 0.0-1.0, default 0.1

initFreqKnobs()
  // Initialize 6 freq knob canvases (30×30)
  // Range: 20-24000 Hz (log scale)
  // Push behavior: Freq1 can't exceed Freq2

setFreqKnobsDisabled(disabled)
  // Enable/disable freq knobs based on audio state
  // Args: disabled (boolean)
  // When disabled: dimmed, no interaction, tooltip shows reason
```

**V5_2_0 Focus Mode Functions (Centralized System):**
```javascript
// CORTEX: Central focus mode management
exitAllFocusModes()
  // Exit any active focus mode and audio panel
  // Sets: activeFocusMode = null, activeAudioPanel = null
  // Clears: all selection states, hides panels

// Audio Focus (NEW V5_2_0)
enterAudioFocus()
  // Enter Audio Focus mode (Shift+A)
  // Sets: activeFocusMode = 'audio'
  // Shows: Audio Focus message bar

exitAudioFocus()
  // Exit Audio Focus mode
  // Clears: activeAudioPanel if set

openAudioPanel(panel)
  // Open sub-panel: 'volume', 'cue', 'eq', 'fx'
  // Sets: activeAudioPanel = panel

closeAudioPanel()
  // Close current audio panel
  // Sets: activeAudioPanel = null

// Renamed functions (V5_2_0: shader* → create*)
enterReactivityFocus()  // Was enterReactivityMode()
  // Sets: activeFocusMode = 'reactivity', createReactivityFocus = true

exitReactivityFocus()
  // Clears: createReactivityFocus = false

enterTransformFocus()
  // Sets: activeFocusMode = 'transform', createTransformFocus = true

exitTransformFocus()
  // Clears: createTransformFocus = false

enterMusicTimingFocus()
  // Sets: activeFocusMode = 'musicTiming', createMusicTimingFocus = true

exitMusicTimingFocus()
  // Clears: createMusicTimingFocus = false
```

---

### EQ3 DOMAIN (V5_2_0)

**CREATE Tab:**
```javascript
initCreateEQ()
  // Initialize 3-band EQ for CREATE tab
  // Creates: BiquadFilterNode chain (High→Mid→Low)
  // Connects to audio element output

updateCreateEQ(band, value)
  // Update EQ band value
  // Args: band ('high', 'mid', 'low'), value (0-1)
  // Maps to dB: 0=kill, 0.25=-12dB, 0.5=-6dB, 0.75=0dB, 1.0=+6dB

killCreateEQ(band)
  // Toggle kill state for band
  // Args: band ('high', 'mid', 'low')

drawEQKnob(canvas, value, killed)
  // Draw EQ knob visual
  // Args: canvas, value (0-1), killed (boolean)
  // Colors: cyan arc (normal), magenta fill (killed)
```

**PERFORM Tab (Per-Track):**
```javascript
initTrackEQ(trackIndex)
  // Initialize EQ for specific track
  // Creates: BiquadFilterNode chain per track

updateTrackEQ(trackIndex, band, value)
  // Update track's EQ band
  // Args: trackIndex (0-3), band (0-2), value (0-1)

killTrackEQ(trackIndex, band)
  // Toggle kill for track's EQ band
```

---

### DJ FX DOMAIN (V5_2_0)

**CREATE Tab:**
```javascript
initCreateFX()
  // Initialize DJ FX for CREATE tab
  // Creates: Washout (highpass), Dubby (delay), Break (playbackRate)

updateCreateFX(slot, value)
  // Update FX wet amount
  // Args: slot ('washout', 'dubby', 'break'), value (0-1)

killCreateFX(slot)
  // Toggle kill state for FX slot

drawFXKnob(canvas, value, killed)
  // Draw FX knob visual
  // Same style as EQ knobs
```

**PERFORM Tab (Per-Track):**
```javascript
initTrackFX(trackIndex)
  // Initialize FX for specific track

updateTrackFX(trackIndex, slot, value)
  // Update track's FX slot
  // Args: trackIndex (0-3), slot (0-2), value (0-1)

killTrackFX(trackIndex, slot)
  // Toggle kill for track's FX slot
```

**FX Implementation:**
```javascript
// Washout (Highpass Sweep)
// BiquadFilterNode type='highpass'
// frequency: 20Hz (value=0) → 2000Hz (value=1)
// Q: 0.7

// Dubby (Dub Delay)
// DelayNode + GainNode feedback loop
// delay: 375ms (3/16 at 120 BPM)
// feedback: 0 (value=0) → 0.8 (value=1)

// Break (Vinyl Stop)
// playbackRate manipulation
// rate: 1.0 (value=0) → 0.1 (value=1)
```

---

### AUDIO CONTROLS MODULE (V5_2_0)

**Volume Knob:**
```javascript
initCreateVolumeKnob()
  // Initialize 24×24 volume knob in header
  // Position: Left of container name

updateCreateVolume(value)
  // Update volume (0-100)
  // V5_3_0: Uses dB mapping (50% = unity)

killCreateVolume()
  // Toggle volume kill (mute)

drawVolumeKnob(canvas, value, killed)
  // Draw volume knob visual
  // 24×24 cyan knob, magenta when killed
```

---

### MOD TUNING DOMAIN (V5_3_0)

**Panel Management:**
```javascript
initModTuningPanel()
  // Initialize Blender-style slide-out panel
  // Creates: Q knobs (3), ADSR knobs (5)
  // Position: Right side of Reactivity section

toggleModTuningPanel()
  // Toggle panel open/close (N key in Reactivity Focus)
  // Sets: modTuningPanelOpen

showModTuningPanel()
  // Slide panel in from right
  // Animation: 200ms ease-out

hideModTuningPanel()
  // Slide panel out to right
  // Animation: 200ms ease-in
```

**Q Control:**
```javascript
updateModQ(mod, value)
  // Update per-modulator Q value
  // Args: mod ('A', 'B', 'C'), value (0.5-10)
  // Updates: bandpass filter Q parameter in real-time

drawQKnob(canvas, value)
  // Draw Q knob visual
  // Range: 0.5-10, default 1.0
  // Style: cyan arc on dark background
```

**ADSR Control:**
```javascript
updateADSR(param, value)
  // Update envelope parameter
  // Args: param ('attack', 'decay', 'sustain', 'release', 'threshold')
  // Updates: envelope generator settings

drawADSRKnob(canvas, value, config)
  // Draw ADSR parameter knob
  // Args: canvas, value, {min, max, default} config
  // Style: matches Q knobs
```

**State Management:**
```javascript
getModTuningState()
  // Get current mod tuning settings
  // Returns: {modAQ, modBQ, modCQ, attack, decay, sustain, release, threshold}

applyModTuningState(state)
  // Apply mod tuning settings to UI and audio
  // Args: modTuning object from .oms file
```

---

### AUDIO REACTIVITY DOMAIN (V5_3_0 Rewrite)

**Bandpass Filtering:**
```javascript
createBandpassFilter(audioContext, centerFreq, Q)
  // Create BiquadFilterNode with bandpass type
  // Args: audioContext, centerFreq (Hz), Q (0.5-10)
  // Returns: BiquadFilterNode
  // Note: centerFreq = (Freq1 + Freq2) / 2

updateBandpassQ(filter, Q)
  // Update filter Q value in real-time
  // Args: BiquadFilterNode, Q value
```

**Peak Detection:**
```javascript
analyzeBandpassPeak(analyser)
  // Get peak value from bandpass-filtered signal
  // Args: AnalyserNode connected after bandpass
  // Returns: 0.0-1.0 (actual peak, not average)
  // Note: Uses getByteTimeDomainData for peak detection

detectPeakFromTimeDomain(dataArray)
  // Find peak amplitude in time-domain samples
  // Args: Uint8Array from analyser
  // Returns: 0.0-1.0 normalized peak
```

**ADSR Envelope:**
```javascript
applyADSREnvelope(input, attack, decay, sustain, release, threshold)
  // Shape raw peak with envelope generator
  // Args: input (0-1), ADSR params (ms/%)
  // Returns: 0.0-1.0 (enveloped value)
  // Implements: attack ramp, decay ramp, sustain hold, release tail

ADSREnvelopeGenerator class
  // Stateful envelope generator per modulator
  // Methods: trigger(level), release(), getValue()
  // Tracks: current phase, elapsed time, target level
```

**Main Analysis Function:**
```javascript
analyzeAudioReactive()
  // V5_3_0: Replaces analyzeAudio() for live reactivity
  // Uses bandpass + peak detection + ADSR per modulator
  // Returns: {modA, modB, modC} with real dynamics
  // Note: Old analyzeAudio() still used for timeline pre-analysis

// Comparison:
// Old: kick hit → 0.02, silence → 0.02 (constant)
// New: kick hit → 0.72, silence → 0.01 (real dynamics)
```

---

### SETS DOMAIN (V5_3_0)

**Folder Management:**
```javascript
initSetsFolder()
  // Initialize /Sets/ subfolder handle
  // Creates folder if not exists
  // Sets: setsFolderHandle

refreshSetsList()
  // Scan /Sets/ folder, populate setsCache
  // Includes both manual and smart sets
  // Recalculates smart set container counts
```

**Manual Sets:**
```javascript
createManualSet(name)
  // Create new empty manual set
  // Args: name (string)
  // Creates: /Sets/[name].oms-set with setType: 'manual'

addContainerToSet(setId, containerId)
  // Add container to manual set
  // Args: setId, containerId
  // Updates: set's containerIds array

removeContainerFromSet(setId, containerId)
  // Remove container from manual set
  // Args: setId, containerId
  // Updates: set's containerIds array

deleteSet(setId)
  // Delete set file from /Sets/ folder
  // Args: setId
  // Confirmation required
```

**Smart Sets:**
```javascript
createSmartSet(name, query)
  // Create smart set from search query
  // Args: name, query string
  // Creates: /Sets/[name].oms-set with setType: 'smart', smartQuery: query

refreshSmartSets()
  // Re-evaluate all smart set queries
  // Updates container counts for each smart set
  // Called on library refresh

getSmartSetContainers(setId)
  // Get containers matching smart set query
  // Args: setId
  // Returns: array of matching containers
  // Evaluates query against all library containers
```

**Set UI:**
```javascript
renderSetsList()
  // Render sets panel in ORGANIZE tab
  // Shows: manual sets (no dot), smart sets (cyan dot)

renderSetCard(set)
  // Render individual set card
  // Shows: name, container count, type indicator

focusSetsPanel()
  // Focus keyboard navigation on sets panel
  // Enables ↑/↓ navigation

focusContainersPanel()
  // Focus keyboard navigation on containers panel
```

---

### TAGS DOMAIN (V5_3_0)

**Tag Management:**
```javascript
addTagToContainer(containerId, tag)
  // Add tag to container's tags array
  // Args: containerId, tag (string)
  // Normalizes: lowercase, trimmed
  // Prevents: duplicates (case-insensitive)
  // Saves: updates .oms file

removeTagFromContainer(containerId, tag)
  // Remove tag from container
  // Args: containerId, tag
  // Saves: updates .oms file

getContainerTags(containerId)
  // Get tags for specific container
  // Returns: array of tag strings
```

**Global Tag Operations:**
```javascript
renameTagGlobal(oldTag, newTag)
  // Rename tag across all containers
  // Args: oldTag, newTag
  // Updates: all .oms files containing oldTag
  // Prevents: creating duplicate (if newTag exists)

deleteTagGlobal(tag)
  // Remove tag from all containers
  // Args: tag
  // Updates: all .oms files containing tag
  // Confirmation required

getAllTags()
  // Get all unique tags across library
  // Returns: array of {tag, count} objects
  // Sorted: by count descending
```

**Tag Index:**
```javascript
rebuildTagIndex()
  // Rebuild global tag index from all containers
  // Called: on library refresh
  // Sets: allTagsIndex map

getContainersByTag(tag)
  // Get all containers with specific tag
  // Args: tag (string)
  // Returns: array of container objects
```

**Tag UI:**
```javascript
renderTagChips(container, tags)
  // Render tag chips in Info Drawer
  // Args: container element, tags array
  // Creates: chip elements with × buttons

initTagInput(container)
  // Initialize tag input field
  // Handles: Enter, comma for chip creation
  // Handles: Backspace for chip deletion

renderTagsAdminPanel()
  // Render tags management in ADMIN tab
  // Shows: all tags with counts
  // Enables: rename, delete operations
```

---

### PEAK METERING DOMAIN (V5_3_0)

**Initialization:**
```javascript
initTrackPeakMeters()
  // Initialize peak meter canvases for all tracks
  // Creates: 4 canvas elements (one per track)
  // Starts: update interval (50ms)

initTrackPeakMeter(trackIndex)
  // Initialize single track peak meter
  // Args: trackIndex (0-3)
  // Creates: canvas, connects to track analyser
```

**Measurement:**
```javascript
calculatePeakdB(analyserNode)
  // Get peak level in dBFS from analyser
  // Args: AnalyserNode
  // Returns: -48 to 0 dB
  // Uses: getByteTimeDomainData, peak detection

updateAllPeakMeters()
  // Update all track peak meters
  // Called: every 50ms by interval
  // Skips: tracks not playing
```

**Rendering:**
```javascript
drawPeakMeter(canvas, dBValue)
  // Draw meter with color zones
  // Args: canvas, dBValue (-48 to 0)
  // Green: -48 to -18 dB (safe)
  // Yellow: -18 to -6 dB (warm)
  // Red: -6 to 0 dB (hot)

updateTrackPeakMeter(trackIndex, dBValue)
  // Update single track meter display
  // Args: trackIndex (0-3), dBValue
  // Stores: value in trackPeakValues array
```

---

### BAR 1 DRAGGING DOMAIN (V5_3_0)

**State:**
```javascript
let bar1Dragging = false;           // Currently dragging
let bar1DragStartX = 0;             // Mouse X at drag start
let bar1DragStartTime = 0;          // audioStartTime at drag start
const BAR1_HIT_TOLERANCE = 15;      // Pixels for click detection
```

**Event Handlers:**
```javascript
handleBar1MouseDown(event)
  // Check if click near Bar 1 marker
  // Args: MouseEvent
  // Sets: bar1Dragging = true if hit
  // Stores: start position and time

handleBar1MouseMove(event)
  // Update marker position during drag
  // Args: MouseEvent
  // Calculates: new position from delta
  // Visual feedback: marker follows mouse

handleBar1MouseUp(event)
  // End drag, update audioStartTime
  // Args: MouseEvent
  // Calculates: final audioStartTime from position
  // Updates: audioStartField
  // Triggers: waveform redraw with grey/blue coloring

isNearBar1Marker(mouseX, canvas)
  // Check if mouse position is near Bar 1 marker
  // Args: mouseX (pixels), canvas element
  // Returns: boolean
  // Uses: BAR1_HIT_TOLERANCE constant
```

**Visual Updates:**
```javascript
drawBar1Marker(ctx, x, height)
  // Draw white "1" marker at position
  // Args: 2D context, x position, canvas height
  // Style: white text, vertical line

updateWaveformColors()
  // Redraw waveform with grey/blue coloring
  // Grey: samples before audioStartTime
  // Blue: samples after audioStartTime
```

---

### TAB ISOLATION DOMAIN (V5_2_0)

```javascript
isCreateTabActive()
  // Check if CREATE tab is visible
  // Returns: boolean

isPerformTabActive()
  // Check if PERFORM tab is visible
  // Returns: boolean

// Keyboard handler pattern:
document.addEventListener('keydown', (e) => {
    // Tab isolation check
    if (!isCreateTabActive()) return;
    
    // Then handle keys...
});
```

---

### BUMPR STICKRS DOMAIN (V5_1_0)

**File Handling:**
```javascript
handleBumprSlotDrop(slotIndex, event)
  // Handle WebM file drop onto slot
  // Copies file to /BumpRStickRs/ folder
  // Generates thumbnail

generateBumprThumbnail(file)
  // Create 160×90 thumbnail from video
  // Uses dark background fill for alpha
  // Returns: Promise<dataURL>

updateBumprSlotUI(slotIndex)
  // Update slot visual state
  // Shows thumbnail, filename, settings

removeBumprSlot(slotIndex)
  // Clear slot and delete file
```

**Playback:**
```javascript
triggerBumprStickr(slotIndex)
  // Fire sticker with timing logic
  // Respects timing mode (instant/beat/bar)
  // Stops current playback first

playBumprVideo(slotIndex)
  // Start video overlay playback
  // Applies scale, animation, duration settings

stopBumprVideo()
  // Stop current video playback
  // Clears queued timeouts

getBumprDelay(slotIndex)
  // Calculate delay for quantized timing
  // Returns: milliseconds to next beat/bar
```

**Mode Management:**
```javascript
enterBumprMode()
  // Enter Shift+B keyboard mode
  // Shows slideout panel
  // Enables 1-0 key triggers

exitBumprMode()
  // Exit keyboard mode
  // Hides slideout panel

updateBumprSlideout()
  // Refresh slideout panel UI
  // Updates selection highlight

showBumprSlotInfo(slotIndex)
  // Display slot info in message bar
```

**Dropdown Controls:**
```javascript
toggleBumprDropdown(dropdownEl)
  // Toggle dropdown open/close

setBumprTiming(slotIndex, mode, optionEl)
  // Set timing mode (instant/beat/bar)

setBumprScale(slotIndex, scale, optionEl)
  // Set scale (25/50/75/100%)

setBumprAnimate(slotIndex, animate, optionEl)
  // Set animation (16 types)

setBumprDuration(slotIndex, duration, optionEl)
  // Set duration (original/loop/musical/timed)
```

---

### CUE POINTS DOMAIN (V5_2_0)

**V5_2_0: Expanded from 3 to 5 cue points**

**Drawing:**
```javascript
// In drawProgress() - draws 5 cue flags
// In drawWaveformFromAudioBuffer() - draws 5 cue flags for PERFORM
```

**Interaction (CREATE Tab):**
```javascript
// Mouse handlers in waveform canvas:
// - mousedown: Check cue hit (1-5), start drag
// - mousemove: Update cue position (bar snap)
// - mouseup: End drag, or jump if no drag occurred
```

**Storage:**
```javascript
// In buildContainerObject():
audioCuePoints: audioCuePoints.slice()  // 5 elements

// Migration (V5_2_0):
if (container.audioCuePoints.length === 3) {
    container.audioCuePoints.push(192, 256);  // Pad to 5
}

// In loadContainer():
audioCuePoints = container.audioCuePoints || [0, 64, 128, 192, 256]

// In updateContainer():
audioCuePoints: audioCuePoints.slice()  // 5 elements
```

---

### UNIFIED MIC SYSTEM DOMAIN (V5_6_0)

**Mic Input Management:**
```javascript
initMicSystem()
  // Initialize mic input system
  // Creates: analyser, gain node
  // Enumerates available input devices

startMicInput(deviceId)
  // Start mic stream from getUserMedia
  // Args: deviceId (string or null for default)
  // Connects: stream → gain → analyser

stopMicInput()
  // Stop mic stream
  // Disconnects: all mic nodes
  // Releases: MediaStream

toggleMic()
  // Toggle mic on/off (tap behavior)
  // When enabling: starts stream if not active
  // When disabling: mutes but keeps stream

updateMicVolume(value)
  // Update mic input gain
  // Args: value (0-100)
  // Uses: dB mapping same as track volume

setMicDevice(deviceId)
  // Change mic input device
  // Args: deviceId (string)
  // Restarts stream with new device
```

**Mic UI:**
```javascript
drawMicKnob(canvas, value, muted)
  // Draw 18×18 mic knob in CREATE header
  // Args: canvas, value (0-100), muted (boolean)
  // Style: cyan arc, red when muted

drawMicMeter(canvas, level)
  // Draw mic level meter
  // Args: canvas, level (0-1)
  // Location: ADMIN → Audio Card section

updateMicLevelDisplay()
  // Update real-time mic level
  // Called: requestAnimationFrame when mic active
```

---

### PER-TRACK VFX DOMAIN (V5_6_0)

**VFX Pipeline:**
```javascript
initTrackVFXPipeline(trackIndex)
  // Initialize VFX pipeline for track
  // Creates: WebGPU compute/render pipeline
  // Args: trackIndex (0-3)

applyTrackVFX(trackIndex, slot, value)
  // Apply VFX effect to track
  // Args: trackIndex, slot (0-2 for A/B/C), value (0-1)
  // Routes to assigned effect from vfxPool

getVFXShaderForEffect(effectName)
  // Get WGSL shader code for effect
  // Args: effectName (from vfxPool)
  // Returns: shader string

killTrackVFX(trackIndex, slot)
  // Toggle kill state for VFX slot
  // Args: trackIndex, slot
  // Bypasses effect when killed
```

**RGB Control:**
```javascript
applyTrackRGB(trackIndex, channel, value)
  // Apply RGB gain to track output
  // Args: trackIndex, channel (0-2 for R/G/B), value (0-1)
  // 0.75 = unity, 0 = kill, 1.0 = boost

killTrackRGB(trackIndex, channel)
  // Toggle kill state for RGB channel
  // Args: trackIndex, channel
  // Sets channel to 0 when killed

drawRGBKnob(canvas, value, killed)
  // Draw RGB knob visual
  // Args: canvas, value, killed (boolean)
  // Style: cyan arc, 0.75 marker for unity
```

---

### EFFECT POOL DOMAIN (V5_6_0)

**Pool Management:**
```javascript
initEffectPools()
  // Initialize VFX and FX pools
  // Sets: default assignments for A/B/C and R/D/W

getVFXPool()
  // Get available VFX effects
  // Returns: array of 10 effect names

getFXPool()
  // Get available audio FX effects
  // Returns: array of 10 effect names

setVFXAssignment(slot, effectName)
  // Assign effect to VFX slot
  // Args: slot ('A', 'B', 'C'), effectName
  // Persists: to project state

setFXAssignment(slot, effectName)
  // Assign effect to FX slot
  // Args: slot ('R', 'D', 'W'), effectName
  // Persists: to project state
```

**Pool Constants:**
```javascript
const VFX_POOL = [
  'RGB Split', 'Glitch', 'Pixelate', 'Scanlines', 'Feedback',
  'Mirror', 'Kaleidoscope', 'VHS', 'Shake', 'Off'
];

const FX_POOL = [
  'Washout', 'Dubby', 'Break', 'Reverb', 'Flanger',
  'Phaser', 'Bitcrush', 'Filter', 'Delay', 'Off'
];
```

---

### AUDIO DOMAIN (Controls Tab)

**Audio Loading/Playback:**
```javascript
loadAudioFile(event)
  // Load audio file from file input
  // Creates: AudioBuffer
  // Updates: UI fields
  // V5_1_0: Resets cue points to defaults

removeAudio()
  // Remove loaded audio
  // Clears: AudioBuffer, resets UI
  // V5_1_0: Resets cue points to defaults

toggleAudioPlayback()
  // Play/pause audio preview
  // Uses: AudioBufferSourceNode

stopAudio()
  // Stop audio playback
  // Cleanup: disconnect nodes

applyPitchToAudio()
  // Apply pitch shift to audio
  // Uses: playbackRate

toggleAudioLoop()
  // Enable/disable loop mode
  // Updates: loopBtn state

toggleAudioGrid()
  // Toggle beatgrid overlay on waveform
```

**Audio Analysis:**
```javascript
analyzeAudio()
  // Perform FFT analysis on audio
  // Returns: {bass: 0.0-1.0, mid: 0.0-1.0, treble: 0.0-1.0}

analyzeAudioTimeline(container)
  // GENERIC: Capture FFT arrays for entire audio track
  // Returns: {
  //   shaderModATimeline: [],
  //   shaderModBTimeline: [],
  //   shaderModCTimeline: []
  // }
  // NOTE: Function is generic, storage fields are shader-specific
```

**Waveform Display (Controls Tab):**
```javascript
generateStaticWaveform()
  // Generate waveform canvas from AudioBuffer
  
drawAudioWaveform(ctx, canvas, audioBuffer)
  // Draw blue amplitude bars

drawBarGrid(ctx, canvas)
  // Draw beat grid on waveform

drawProgress()
  // Draw playback progress indicator
  // V5_1_0: Also draws cue flags

setupWaveformControls()
  // Initialize waveform UI controls
```

**Loop/Timing Controls:**
```javascript
showLoopDuration()
  // Calculate and display loop length

setInToStart()
  // Set loop in to audio start

setOutToEnd()
  // Set loop out to audio end

setAudioStartToZero()
  // Reset audio start time to 0

setAudioEndToMax()
  // Set audio end time to max

setAudioStart(event)  // V4.4.0
  // Click: reset to 0
  // Shift+click: pin to playhead

setAudioEnd(event)  // V4.4.0
  // Click: reset to max
  // Shift+click: pin to playhead

setupInOutArrowKeys()
  // Enable arrow key navigation for loop points
```

---

### SHADER DOMAIN (Controls Tab)

**Shader Compilation/Rendering:**
```javascript
initWebGPU()
  // Initialize WebGPU device and context
  // Returns: Promise<void>

compileShader(resetAudio)
  // Compile WGSL shader code
  // Creates: GPUShaderModule, render pipeline
  // Args: resetAudio (boolean)

render()
  // Render single frame (preview or recording)
  // Updates: uniforms, draws to canvas

startRender()
  // Start video recording
  // Creates: MediaRecorder, begins capture

finishRender(message, isError)
  // Complete video recording
  // Args: message (string), isError (boolean)

updateCanvasSize(orientationOverride)
  // Resize canvas based on resolution/orientation
  // Args: orientationOverride (optional)
```

**Shader Utilities:**
```javascript
copyShaderCode()
  // Copy shader code to clipboard

exportShaderTxt()
  // Export shader as .txt file

getShaderNameFromCode()
  // Extract shader name from current code
  // Returns: string
```

---

### PRESET/LIBRARY DOMAIN

**Container Building (V4_6_0, V5_1_0):**
```javascript
buildContainerObject(name, audioOverride)
  // UNIFIED container builder - single source of truth for all save paths
  // Args: name (string), audioOverride (optional object)
  // V5_1_0: Includes audioCuePoints, shaderTempoLinked
  // Returns: Complete containerData object
  // Used by: confirmAddToLibrary(), updateContainer(), all save workflows
```

**V5_1_0 Save/Update Separation:**
```javascript
saveContainer()
  // Save as NEW container to Library
  // Always creates new file
  // Switches to ORGANIZE tab

updateContainer()
  // Update currently loaded container
  // Keeps existing name
  // V5_1_0: Stays on CREATE tab
  // V5_1_0: Includes audioCuePoints
```

**Library/Container Management:**
```javascript
addToLibrary()
  // Begin add container workflow
  // Opens: modal dialog

confirmAddToLibrary()
  // Save current state as .oms container
  // Creates: containerData object
  // V5_1_0: Sets currentLoadedContainerId

cancelAddToLibrary()
  // Cancel add to library workflow

saveContainerToOMS(container)
  // Save .oms file to Library/ folder
  // Args: containerData object
  // Returns: Promise<void>

loadContainer(containerId)
  // Load .oms container from Library/
  // Args: containerId (string) = filename without .oms
  // V5_1_0: Restores audioCuePoints
  // Returns: Promise<void>

updateContainer(containerId)
  // Update existing .oms container
  // Args: containerId (string)
  // V5_1_0: Includes audioCuePoints

removeContainer(containerId)
  // Delete .oms file from Library/
  // Args: containerId (string)
  // V5_1_0: Delete key checks for input focus first

exportContainer(containerId)
  // Export .oms.zip to Exports/ folder
  // Args: containerId (string)

handleOmsFileImport(event)
  // Import .oms.zip file (V4.3.0_A: ZIP only)
  // Args: event (file input event)
  // Extracts: container + audio to Library/

refreshLibraryList()
  // V4.3.0_A: Scans folder, populates libraryContainersCache
  // Updates: library UI display
```

---

### V5_0_0 TRANSFORM DOMAIN

**Transform Initialization:**
```javascript
initTransformControls()
  // Initialize all 5 transform controls
  // Creates: Rotation/Scale/Tile encoders, XY/Pivot pads
  // Sets up: wheel handlers, keyboard handlers

initTransformKnob(id, config)
  // Initialize single transform encoder
  // Args: id (DOM id), config (min/max/step/default/endless)
  // Returns: Canvas reference

initTransformPad(id, config)
  // Initialize XY/Pivot pad
  // Args: id (DOM id), config (min/max/step/default)
  // Returns: Canvas reference

drawTransformKnob(canvas, value, config)
  // Render encoder visual (360° circle for endless, 300° arc for bounded)
  // Args: canvas, value, config object

drawTransformPad(canvas, x, y)
  // Render XY pad visual (crosshair with dot)
  // Args: canvas, x value, y value
```

**Transform Focus Mode (V5_1_0 Renames):**
```javascript
enterTransformFocus()     // Was enterTransformMode()
  // Enter keyboard mode (Shift+T)
  // Shows cyan selection ring
  // Message bar shows "Transform Focus"

exitTransformFocus()      // Was exitTransformMode()
  // Exit keyboard mode (Escape)
  // Clears selection ring

updateTransformSelection()
  // Update visual selection feedback
  // Draws cyan ring on selected control
  // Updates message bar with control name

adjustTransformValue(delta)
  // Adjust selected control value
  // Args: delta (positive or negative)
  // Handles: Encoders (rotation, scale, tile), Pads (x/y)
  // Respects endless flag for rotation wrapping

resetTransformToDefault()
  // Reset selected control to default value
  // Double-tap Shift+Enter required (300ms window)
```

**Transform Animation (V5_1_0):**
```javascript
setTransformAnimationMode(control, mode)
  // Set animation mode for control
  // Args: control ('rotation', 'scale', 'tile'), mode ('stop', 'clockwise', 'counterclockwise', 'pingpong')
  // Keyboard: 0=stop, 1=cw, 2=ccw, 3=pingpong

applyTransformAnimation()
  // Called each frame
  // Updates transform values based on modes
  // PingPong uses shaderLoopLength for timing
```

---

### V5_0_0 UNIFIED WHEEL HANDLER

**Single document-level wheel handler for all adjustable controls:**
```javascript
document.addEventListener('wheel', (e) => {
    // Only active with Shift key held
    if (!e.shiftKey) return;
    
    // Get control under cursor or keyboard-selected control
    // Route to appropriate adjustment function
});
```

**Supported Controls (14 total):**
- 5 Transform controls: Rotation, Scale, Tile, XY, Pivot
- 9 Reactivity knobs: Gain×3, Freq1×3, Freq2×3

**Two Operation Modes:**
1. **Hover mode:** Shift + hover over control + scroll
2. **Keyboard mode:** Enter mode, select control, Shift + scroll anywhere

**Wheel Sensitivity Reference:**
| Control Type | deltaY | Result |
|--------------|--------|--------|
| Rotation | ÷120 | 1° per notch |
| Scale | ÷1200 | 0.1 per notch |
| Tile | ÷120 | 1 per notch |
| XY/Pivot | ÷2400 | 0.05 per notch |
| Gain knobs | ÷1200 | 0.01 per notch |
| Freq knobs | ×2 | Log scale |

---

### V5_0_0 OMS DESIGN SYSTEM

**Purpose:** Universal CSS components for consistent UI.

**Design Tokens:**
```css
--oms-surface-bg: rgba(255, 255, 255, 0.05);
--oms-surface-hover: rgba(255, 255, 255, 0.08);
--oms-control-height: 34px;
--oms-border-radius: 0;
--oms-focus-color: #00d4ff;
```

**Universal Components:**
```css
.oms-header-bar       /* Section headers - sharp corners, 34px height */
.oms-field            /* Input fields - borderless, 34px height */
.oms-control-box      /* Grouped controls container */
.oms-dropdown         /* Custom dropdown styling (via OmsDropdown) */
.mod-knob-box         /* Modulator knob containers */
```

**CSS Component Examples:**
```css
.oms-header-bar {
    background: rgba(255, 255, 255, 0.05);
    padding: 6px 10px;
    font-size: 11px;
    font-weight: 600;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    border-radius: 0;
}

.oms-field {
    background: rgba(255, 255, 255, 0.05);
    border: none;
    padding: 8px 10px;
    font-size: 13px;
    color: #fff;
    height: 34px;
    border-radius: 0;
}

.oms-field:focus {
    outline: 1px solid #00d4ff;
}
```

---

### V5_1_0 BUMPR STICKRS CSS

**Animation Keyframes (15 types):**
```css
@keyframes bumpr-bounce { /* Vertical bounce */ }
@keyframes bumpr-breathe { /* Scale in/out slowly */ }
@keyframes bumpr-crazy { /* Chaotic movement */ }
@keyframes bumpr-drift { /* Slow horizontal drift */ }
@keyframes bumpr-drop { /* Drop from above */ }
@keyframes bumpr-float { /* Gentle float up */ }
@keyframes bumpr-glitch { /* Glitch effect */ }
@keyframes bumpr-pop { /* Pop in with scale */ }
@keyframes bumpr-pulse { /* Opacity pulse */ }
@keyframes bumpr-shake { /* Horizontal shake */ }
@keyframes bumpr-spin { /* 360° rotation */ }
@keyframes bumpr-spiral { /* Spiral in */ }
@keyframes bumpr-strobe { /* Strobe flash */ }
@keyframes bumpr-wobble { /* Rotation wobble */ }
@keyframes bumpr-zoom { /* Zoom in/out */ }
```

**Slideout Panel:**
```css
.bumpr-slideout {
    position: absolute;
    right: 0;
    top: 0;
    width: 80px;
    height: 100%;
    background: rgba(0, 0, 0, 0.9);
    transform: translateX(100%);
    transition: transform 0.2s;
}

.bumpr-slideout.active {
    transform: translateX(0);
}
```

**Dropdown Widths:**
```css
.bumpr-dropdown.scale { width: 50px; }
.bumpr-dropdown.timing { width: 55px; }
.bumpr-dropdown.animate { width: 65px; }
.bumpr-dropdown.duration { width: 60px; }
```

---

## NAMING CONVENTIONS

### ID Naming Rules

**Buttons:**
```
[action]Btn
Examples: playBtn, loopBtn, renderBtn, uploadBtn, saveBtn, updateBtn
```

**Input Fields (V5_1_0 Updates):**
```
Audio timing: [name]Field
Examples: audioTempoField, audioStartField, audioEndField

Shader timing: shader[Name]
Examples: shaderTempo, shaderRenderDuration, shaderLoopLength

Other inputs: [name]
Examples: filename, resolution
```

**Display Fields:**
```
[name]
Examples: audioFileName, messageBar
```

**Modulators:**
```
mod[A/B/C][property]
Examples: modAWave, modBSubdiv, modCWave
```

**Groups:**
```
[name]Group
Examples: bassGainGroup, midGainGroup, trebleGainGroup
```

**Labels:**
```
[fieldName]Label
Examples: inLabel, outLabel
```

### Variable Naming Rules (V5_2_0 Updates)

**Focus Mode State (V5_2_0 Centralized + Renames):**
```javascript
// V5_2_0: Centralized focus mode variables (CORTEX)
activeFocusMode           // Current active mode (null or string)
activeAudioPanel          // Current audio panel (null or string)
exitAllFocusModes()       // Central cleanup function

// V5_2_0: CREATE tab renames (shader* → create*)
createReactivityFocus     // Was shaderReactivityFocus
createTransformFocus      // Was shaderTransformFocus
createMusicTimingFocus    // Was shaderMusicTimingFocus
createAudioFocus          // NEW V5_2_0

// Selected positions
createReactivitySelectedRow  // Was shaderReactivitySelectedRow
createReactivitySelectedCol  // Was shaderReactivitySelectedCol
createTransformSelectedCol   // Was shaderTransformSelectedCol

// Backward compatibility via Object.defineProperty
// Old names still work but map to new names
```

**V5_4_0 Master→MAIN Rename Pattern:**

**Rule:** "Master" = timing/grid systems. "MAIN" = output bus.

| Old Name (V5_3_0) | New Name (V5_4_0) |
|-------------------|-------------------|
| `masterShaderOpacity` | `trackMainShaderOpacity` |
| `masterShaderMute` | `trackMainShaderMute` |
| `masterShaderSolo` | `trackMainShaderSolo` |
| `masterVolume` | `trackMainVolume` |
| `masterAudioMute` | `trackMainAudioMute` |
| `masterGainNode` | `trackMainGainNode` |
| `masterAnalyser` | `trackMainAnalyser` |
| `masterShaderOpacityKnob` | `trackMainShaderOpacityKnob` |
| `masterShaderMuteBtn` | `trackMainShaderMuteBtn` |
| `masterAudioVolumeKnob` | `trackMainAudioVolumeKnob` |
| `masterAudioMuteBtn` | `trackMainAudioMuteBtn` |

**Unchanged (still use "master"):**
- `masterTempo` - timing setting
- `masterBeatNow()` - timing function
- `masterBarNow()` - timing function
- `masterGridVisible` - grid visibility
- `trackMasterClockStart` - clock reference
- `gridState.masterTempo` - tempo state

**New V5_4_0 Naming Patterns:**
```javascript
// Audio Card
audioCard[Property]           // audioCardMasterOutputId, audioCardCueMix

// DSP
dsp[Property]                 // dspEnabled, dspPreset, dspThreshold

// VFX
vfx[Property]                 // vfxEffect, vfxAmount, vfxPipeline

// Scene Labels
scene[Property]               // sceneLabels, sceneColors
sceneLabelEdit[Property]      // sceneLabelEditingIndex
```

**Arrays:**
```javascript
// Per-track arrays (always length 4, index 0-3)
trackShaderOpacity = [100, 100, 100, 100];
trackAudioVolume = [75, 75, 75, 75];
trackAudioPlaying = [false, false, false, false];

// Cue points (V5_2_0: length 5, was 3)
audioCuePoints = [0, 64, 128, 192, 256];

// BumpR slots (always length 10)
bumprStickrsSlots = [null, null, null, null, null, null, null, null, null, null];

// Per-track EQ/FX (V5_2_0) - [4 tracks][3 values]
trackEQValues = [[0.75,0.75,0.75], [0.75,0.75,0.75], [0.75,0.75,0.75], [0.75,0.75,0.75]];
trackFXValues = [[0,0,0], [0,0,0], [0,0,0], [0,0,0]];
```

**Maps/Objects:**
```javascript
// Use Map for dynamic collections
audioBuffers = new Map(); // containerName → AudioBuffer

// Use objects for structured data
containerData = {
  name: "...",
  version: "...",
  audioCuePoints: [...],  // V5_1_0
  // ...
};

// Transform animation modes (V5_1_0)
transformModes = {
  rotation: 'stop',
  scale: 'stop',
  tile: 'stop'
};
```

**Functions:**
```javascript
// camelCase, verb-first
loadAudioFile()
playCellAudio()
calculateModulator()
updateShaderUniforms()
triggerBumprStickr()      // V5_1_0
enterReactivityFocus()    // V5_1_0 rename
```

**Constants:**
```javascript
// UPPER_SNAKE_CASE
const MAX_TRACKS = 4;
const MAX_SCENES = 8;
const MIN_BPM = 60;
const MAX_BPM = 200;
const CUE_HIT_TOLERANCE = 20;  // V5_1_0
```

---

## VERSION COMPATIBILITY

**File Format Versioning:**
```javascript
// All files include version field
{
  version: "5.4.0",
  // ...
}

// Forward compatibility: newer versions can read older files
// Backward compatibility: not guaranteed (add migration if needed)
```

**Migration Strategy (V5_4_0, V5_5_0):**
```javascript
function migrateProjectState(projectData) {
  const version = projectData.version;
  
  // V5_4_0: Add new fields if missing
  if (!projectData.audioCardSettings) {
    projectData.audioCardSettings = {
      masterOutputId: null,
      cueOutputId: null,
      cueMix: 0.5
    };
  }
  
  if (!projectData.trackCues) {
    projectData.trackCues = [false, false, false, false];
  }
  
  if (!projectData.sceneLabels) {
    projectData.sceneLabels = ['1', '2', '3', '4', '5', '6', '7', '8'];
  }
  
  if (!projectData.sceneColors) {
    projectData.sceneColors = [null, null, null, null, null, null, null, null];
  }
  
  if (!projectData.dspSettings) {
    projectData.dspSettings = {
      enabled: false,
      preset: 'off',
      threshold: -18,
      ratio: 4,
      output: 4
    };
  }
  
  if (!projectData.vfxSettings) {
    projectData.vfxSettings = {
      effect: 'off',
      amount: 0
    };
  }
  
  // V5_5_0: Add Video Sources & PIP settings
  if (!projectData.videoSourcesSettings) {
    projectData.videoSourcesSettings = {
      deviceId: null,
      mirror: false
    };
  }
  
  if (!projectData.pip1Settings) {
    projectData.pip1Settings = { visible: false, shape: 'landscape', color: 'cyan', label: '' };
  }
  
  if (!projectData.pip2Settings) {
    projectData.pip2Settings = { shape: 'landscape', color: 'cyan' };
  }
  
  if (!projectData.pip3Settings) {
    projectData.pip3Settings = { shape: 'landscape', color: 'cyan' };
  }
  
  // V5_4_0: Rename master* to trackMain* in tracks array
  if (projectData.masterShaderOpacity !== undefined) {
    projectData.trackMainShaderOpacity = projectData.masterShaderOpacity;
    delete projectData.masterShaderOpacity;
  }
  // ... repeat for other master* properties
  
  return projectData;
}

function migrateContainer(containerData) {
  const version = containerData.version;
  
  // V5_2_0: Expand cue points from 3 to 5
  if (!containerData.audioCuePoints) {
    containerData.audioCuePoints = [0, 64, 128, 192, 256];
  } else if (containerData.audioCuePoints.length === 3) {
    // Pad existing 3-cue containers to 5
    containerData.audioCuePoints.push(192, 256);
  }
  
  // V5_1_0: Add shaderTempoLinked if missing
  if (containerData.controlStates && 
      containerData.controlStates.shaderTempoLinked === undefined) {
    containerData.controlStates.shaderTempoLinked = true;
  }
  
  // V5_5_0: Migrate global ADSR to per-mod ADSR+T
  if (containerData.modTuning && containerData.modTuning.attack !== undefined) {
    // Old format had global ADSR - migrate to per-mod
    const oldAttack = containerData.modTuning.attack || 5;
    const oldDecay = containerData.modTuning.decay || 20;
    const oldSustain = containerData.modTuning.sustain || 70;
    const oldRelease = containerData.modTuning.release || 50;
    const oldThreshold = containerData.modTuning.threshold || 1;
    
    // Apply global values to all mods
    containerData.modTuning = {
      modAQ: containerData.modTuning.modAQ || 0.70,
      modAAttack: oldAttack, modADecay: oldDecay,
      modASustain: oldSustain, modARelease: oldRelease,
      modAThreshold: oldThreshold,
      
      modBQ: containerData.modTuning.modBQ || 0.70,
      modBAttack: oldAttack, modBDecay: oldDecay,
      modBSustain: oldSustain, modBRelease: oldRelease,
      modBThreshold: oldThreshold,
      
      modCQ: containerData.modTuning.modCQ || 0.70,
      modCAttack: oldAttack, modCDecay: oldDecay,
      modCSustain: oldSustain, modCRelease: oldRelease,
      modCThreshold: oldThreshold
    };
  }
  
  // V5_5_0: Add transform animation if missing
  if (!containerData.transformAnimation) {
    containerData.transformAnimation = {
      rotationMode: 'stop', rotationSpeed: 1.0,
      scaleMode: 'stop', scaleSpeed: 1.0,
      tileMode: 'stop', tileSpeed: 1.0
    };
  }
  
  return containerData;
}
```

---

## PERFORMANCE NOTES

**Critical Numbers:**
- 60fps shader rendering (all 4 tracks)
- <16ms per frame budget
- 4 WebGPU pipelines (one per track)
- +1 VFX pipeline (V5_4_0: post-processing)
- FFT analysis: 2048 samples per track
- Stereo metering: 2048 samples × 2 channels (V5_4_0)
- DSP: Real-time compression (no latency) (V5_4_0)
- Waveform scroll: CSS transform (GPU accelerated)
- BumpR video: WebM VP9 with alpha (hardware decoded)

**Optimization Techniques:**
- Shader compilation cached per container
- Audio buffers cached in Map
- Waveform canvas pre-rendered (not drawn per frame)
- Beat grid drawn once (only on tempo change)
- requestAnimationFrame for all animations
- BumpR thumbnails cached as data URLs
- VFX pipeline cached per effect (V5_4_0)

**Browser Requirements (V5_4_0, V5_6_0):**
- WebGPU support (Chrome 113+, Edge 113+)
- setSinkId API (Chrome 110+, Edge 110+) - for Audio Card
- getUserMedia API (all modern browsers) - for Unified Mic (V5_6_0)
- Web Audio API (all modern browsers)
- File System Access API (Chrome/Edge only)
- WebM VP9 alpha (Chrome/Edge for BumpR)
- Minimum 4GB RAM (for audio buffers)
- Dedicated GPU recommended (especially for VFX)

---

## END OF SYSTEM REFERENCE

**Version:** Beta V5_6_0  
**Date:** January 11, 2026  
**License:** GPL-3.0
