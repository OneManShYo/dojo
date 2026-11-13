# ONEMANSHYO App [Beta V5_0_0] - SYSTEM REFERENCE

**Date:** December 21, 2025  
**Version:** V5_0_0  
**Purpose:** Complete technical implementation reference

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

**Master Clock / Seconds:**
- `audioContext` - Web Audio API instance
- `audioContext.currentTime` - browser clock, returns seconds
- `trackMasterClockStart` - session start timestamp, stored in seconds

**Master Clock / Musical Time:**
- `gridState.masterTempo` - BPM value (conversion rate)
- `trackAudioMasterClock()` - THE FUNCTION that converts seconds → beats
- `masterBeatNow()` - calls trackAudioMasterClock(), returns current beat
- `masterBarNow()` - calls masterBeatNow() / 4, returns current bar

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
    shaderModAFFT: f32,
    shaderModBWave: f32,
    shaderModBGain: f32,
    shaderModBFFT: f32,
    shaderModCWave: f32,
    shaderModCGain: f32,
    shaderModCFFT: f32,
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
        p.shaderModAWave, p.shaderModAGain, p.shaderModAFFT,
        p.shaderModBWave, p.shaderModBGain, p.shaderModBFFT,
        p.shaderModCWave, p.shaderModCGain, p.shaderModCFFT,
        p.transformRotation,
        p.transformScaleX, p.transformScaleY,
        p.transformTranslateX, p.transformTranslateY,
        p.transformPivotX, p.transformPivotY,
        p.transformTile,
        0, 0, 0  // padding
    ]);
}
```

**Adding a New Uniform (4 steps):**
1. Add field to SHADER_UNIFORM_STRUCT (maintain GPU alignment)
2. Update UNIFORM_BUFFER_SIZE if needed
3. Add parameter to createUniformData() factory
4. Pass value in render() call

**Why This Matters:**
- Before V4_9_15: 6+ scattered locations, ~20,000 line hunts
- After V4_9_15: 3 constants, 4-step process
- Thumbnail buffer bug fixed (was 96, needed 112)

**V5_0_0 Uniform Struct (28 floats = 112 bytes):**

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
| 10 | shaderModAFFT | f32 | 0-1 | Mod A FFT level |
| 11 | shaderModBWave | f32 | 0-1 | Mod B waveform output |
| 12 | shaderModBGain | f32 | 0-1 | Mod B gain value |
| 13 | shaderModBFFT | f32 | 0-1 | Mod B FFT level |
| 14 | shaderModCWave | f32 | 0-1 | Mod C waveform output |
| 15 | shaderModCGain | f32 | 0-1 | Mod C gain value |
| 16 | shaderModCFFT | f32 | 0-1 | Mod C FFT level |
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

**Message Bar System (V4_7_0, V4_8_0):**
- `messageBar` - DOM element ID (single message display)
- `showMessage(text, type, onClick)` - THE FUNCTION for all user feedback
- Types: `'info'` (cyan), `'state'` (orange), `'error'` (magenta)
- V4_8_0: Optional `onClick` callback makes message clickable (cursor: pointer)
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

**Controls:**
- Keyboard shortcut mappings
- Header text
- Tab labels
- Font choices

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
- `calculateMod()` - calculate waveform value (lines 3139-3209, handles ALL waveform types including meta-modes: sine, square, triangle, sawtooth, cycler, chaosmode, madmax, psycho)

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

**TRACK AUDIO DOMAIN (LIVE Tab):**
- `playCellAudio()` - start playing track audio
- `initAudioRouting()` - initialize Web Audio API routing

**WAVEFORM DOMAIN:**
- `drawWaveformFromAudioBuffer()` - draw blue amplitude bars
- `drawBeatGridOnWaveform()` - draw cyan/magenta beat grid
- `positionTrackAudioWaveformAtPlayhead()` - position Bar 1 at playhead
- `startAudioWaveformScroll()` - start scroll animation

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

**#ForReview - LIVE TAB DOMAIN NAMING:**
- `startMasterClock()` - #ForReview should be `liveStartMasterClock()`
- `getNextBarBoundary()` - #ForReview should be `liveGetNextBarBoundary()`
- `isAnyTrackPlaying()` - #ForReview should be `liveIsAnyTrackPlaying()`
- Many LIVE tab functions lack `live*` prefix
- Waveform functions lack `waveform*` or `live*` prefix
- Mixer functions lack `mixer*` or `live*` prefix

---

### 2. UI FIELDS/VARIABLES (Organized by Usage Domain)

**SHADER DOMAIN:**
- `tempo` - shader tempo (visual animation speed, NOT audio BPM)
- `renderDuration` - MM:SS.mmm format (when no audio)
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
- `loopLength` - calculated duration display

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
  version: "3.8.0",
  shaderCode: "/* WGSL */",
  tempo: 128,
  modAWave: "sine",
  // ... full structure documented in File Formats section
}
```

**projectState** - .omsp file structure  
```javascript
{
  projectName: "Friday Set",
  version: "3.8.0",
  gridCells: [...],
  masterTempo: 128,
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
- Contains: Shader code, timing, modulators, audio reference, render settings
- Location: `/LIBRARY/[name].oms`

**.omsp Files:**
- Purpose: LIVE tab project state
- Contains: Grid layout, track states, master settings
- Location: `/LIBRARY/[projectName].omsp`
- Auto-save: `default.omsp`

**.oms-set Files:**
- Purpose: Container collections
- Contains: Array of .oms containerData objects
- Location: `/LIBRARY/[setName].oms-set`
- Use: Distribution, backup, sharing

---

### CRITICAL SEPARATION

**tempo vs audioTempoField:**
```javascript
tempo          → Shader Tempo (visual animation speed)
audioTempoField → Audio Tempo (musical timing/beat positioning)

// These are NEVER the same value!
// Example:
tempo = 128           // Shader animates at 128 BPM
audioTempoField = 124 // Audio track is 124 BPM
// Both independent, both valid
```

**Why This Matters:**
- Shaders can animate at different speeds than music
- Pure visual mode (no audio) uses only `tempo`
- Audio-reactive mode uses BOTH (independently)

---

## FILE FORMAT SPECIFICATIONS

### .oms Files (Containers)

**Purpose:** Complete shader package with all settings and optional audio reference

**File Structure:**
```javascript
{
  // Metadata
  name: "Bass Drop Tunnel",
  version: "3.8.0",
  created: "2025-11-15T10:30:00Z",
  
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
  "shaderModCFreq2": 500
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

**Purpose:** Save/restore LIVE tab performance grid layout

**File Structure:**
```javascript
{
  // Metadata
  projectName: "Friday Set",
  version: "3.8.0",
  saved: "2025-11-15T22:45:00Z",
  
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
  
  // Master Controls
  masterShaderMute: false,
  masterShaderSolo: false,
  masterAudioVolume: 100,
  masterAudioMute: false
}
```

**Storage Location:** `/LIBRARY/[projectName].omsp`

**Auto-Save:**
- Default project: `default.omsp` (auto-saves every action)
- Manual saves: Custom names via Settings tab

---

### .oms-set Files (Preset Collections)

**Purpose:** Bundle multiple presets for distribution/backup

**File Structure:**
```javascript
{
  // Metadata
  setName: "Bass Music Collection",
  author: "ONEMANSHYO",
  version: "3.8.0",
  created: "2025-11-15T10:00:00Z",
  description: "Heavy bass presets for 128-140 BPM",
  
  // Preset Array
  presets: [
    {
      name: "Bass Drop Tunnel",
      category: "Bass",
      shaderCode: "/* ... */",
      tempo: 128,
      // ... full .oms structure
    },
    {
      name: "Sub Wobble Grid",
      category: "Bass",
      shaderCode: "/* ... */",
      tempo: 140,
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

**Storage Location:** `/LIBRARY/[setName].oms-set`

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

// Shader Uniforms (sent to GPU each frame) - V4_8_0: 80 bytes
uniforms = {
  time: 0.0,              // Seconds since shader start
  tempo: 128,             // Shader Tempo (visual timing)
  modA: 0.0,              // Combined bass (0-1) - legacy
  modB: 0.0,              // Combined mids (0-1) - legacy
  modC: 0.0,              // Combined treble (0-1) - legacy
  beat: 0.0,              // Tempo-synced beat position (V4_8_0)
  resolutionX: 1920,      // Canvas width
  resolutionY: 1080,      // Canvas height
  shaderModAWave: 0.0,    // Mod A waveform output (V4_8_0)
  shaderModAGain: 0.1,    // Mod A gain value (V4_8_0)
  shaderModAFFT: 0.0,     // Mod A FFT level (V4_8_0)
  shaderModBWave: 0.0,    // Mod B waveform
  shaderModBGain: 0.1,    // Mod B gain
  shaderModBFFT: 0.0,     // Mod B FFT
  shaderModCWave: 0.0,    // Mod C waveform
  shaderModCGain: 0.1,    // Mod C gain
  shaderModCFFT: 0.0,     // Mod C FFT
  padding1: 0.0,          // GPU alignment
  padding2: 0.0,
  padding3: 0.0           // 20 floats = 80 bytes
};
```

**Shader Compilation (V4_8_0):**
```javascript
// App injects struct and binding - user code just uses uniforms.*
// User writes ONLY the fragment function:

@fragment
fn fs_main(@builtin(position) pos: vec4<f32>) -> @location(0) vec4<f32> {
  // Access any uniform directly:
  let t = uniforms.time;
  let bass = uniforms.modA;           // Legacy combined
  let bassWave = uniforms.shaderModAWave;  // Just waveform
  let bassFFT = uniforms.shaderModAFFT;    // Just FFT
  
  return vec4<f32>(r, g, b, 1.0);
}
```

**Modulator Calculation:**

**CRITICAL:** Uses centralized `calculateMod(beat, subdivision, waveform)` function (lines 3139-3209)

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

// Example implementations:

// SINE WAVE
function calculateSine(time, freq) {
  return Math.sin(time * freq * Math.PI * 2);
}

// SQUARE WAVE
function calculateSquare(time, freq) {
  return Math.sin(time * freq * Math.PI * 2) > 0 ? 1 : -1;
}

// TRIANGLE WAVE
function calculateTriangle(time, freq) {
  return Math.asin(Math.sin(time * freq * Math.PI * 2)) * (2/Math.PI);
}

// SAWTOOTH WAVE
function calculateSawtooth(time, freq) {
  return (time * freq) % 1.0;
}

// CYCLER (Custom - bounces up and down)
function calculateCycler(time, freq) {
  return Math.abs(((time * freq) % 2.0) - 1.0);
}

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
modA = Wave × Gain × FFT  // When audio loaded
modA = Wave × Gain        // When no audio (FFT treated as 1.0)
```

**Independent Shader Access (V4_8_0):**
```javascript
// Shaders can access components separately:
uniforms.shaderModAWave   // Just the waveform
uniforms.shaderModAGain   // Just the gain knob value
uniforms.shaderModAFFT    // Just the FFT level

// Enables custom combinations:
let customMod = uniforms.shaderModAWave * uniforms.shaderModBFFT;
```

**Works Without Audio:**
```javascript
// When no audio loaded:
- Wave functions normally (tempo-synced oscillation)
- Gain functions normally (user-controlled)
- FFT = 0 (no audio to analyze)
- Freq knobs disabled (meaningless without audio)

// When audio loaded:
- All three components active
- Freq knobs enable, define FFT analysis range
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
| Playhead | FIXED | Magenta line drawn on Master Grid canvas at playheadX |

**Key Insight:** Master Grid is stationary ruler. Track Grid + Waveform scroll under it.

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
- Function: `drawWaveformFromAudioBuffer()` #ForReview misleading name - also draws grid
- Independence: Changes with RMS, grid does NOT (proves separation)

**Component 2: Beat Grid (Timing Markers)**
- Physical: Magenta phrase markers + white bar markers
- Affected by: Zoom (spacing), NOT affected by RMS
- Function: `drawBeatGridOnWaveform()` - accurate name
- Independence: Separate from waveform, drawn on same canvas

**Component 3: Canvas Positioning**
- Physical: translateX() CSS transform
- Purpose: Position Bar 1 at playhead, scroll during playback
- Function: `positionTrackAudioWaveformAtPlayhead()` #ForReview misleading - manages entire display

**Component 4: Scroll Animation**
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

Phrase 1 (bars 1-4):
M | | | | W | | | | W | | | | W | | | | M
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

---

### 6. LIBRARY System (File System Access API)

**Purpose:** Persistent storage for containers and projects

**V4.3.0_A Architecture Change:** Folder is now single source of truth. localStorage no longer used for container storage.

---

**Directory Structure (V4.4.0):**
```
[User's Selected Folder]/              ← applicationFolderHandle
  ├── onemanshyo.omsp                  ← App state (auto-save, includes selectedSetId)
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

**Two File Handles (CORTEX):**
```javascript
let applicationFolderHandle = null;   // Root folder (user selects)
let libraryFolderHandle = null;       // /Library/ subfolder

// On folder selection:
applicationFolderHandle = await window.showDirectoryPicker();
libraryFolderHandle = await applicationFolderHandle.getDirectoryHandle('Library', { create: true });
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

**Shader Controls (Visual Timing):**
```javascript
// V4_6_0: Shader Name Dropdown
shaderNameDropdown    // Select - pick shader from Presets/Library
                      // Values: _current, _setfolder, sysPreset:*, userPreset:*, library:*

tempo                 // Shader Tempo (60-200 BPM)
                      // NOT audio BPM - visual animation speed only
                      // V4_8_0: DISABLED when audio loaded (grayed out)
                      // Audio tempo becomes THE tempo when audio present
                      
renderDuration        // MM:SS.mmm format (when no audio loaded)
                      // Determines render length for pure shader mode

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
messageBar          // Status display - compile messages
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
loopLength            // Display - calculated duration

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

**Reactivity Keyboard Mode (V4_8_0):**
```javascript
// State variables
modReactivityMode     // boolean - keyboard mode active
modSelectedRow        // 0-2 (A/B/C)
modSelectedCol        // 0-4 (Subdiv/Waveform/Gain/Freq1/Freq2)
modKeyRepeatCount     // Progressive acceleration counter
modLastKeyTime        // Timestamp for acceleration
freqKnobsDisabled     // boolean - true when no audio loaded
```

**Transport Controls:**
```javascript
playBtn               // Play/Pause toggle
loopBtn               // Loop enable/disable toggle
uploadBtn             // Upload audio file
removeBtn             // Remove loaded audio
stopBtn               // (Deprecated - may be hidden)
```

**Render Settings:**
```javascript
filename              // Output filename (no extension)
                      // V4_6_0: Auto-populated from cleanAudioName() + shader name
resolution            // Select: 720p, 1080p, 1440p, 2160p
fps                   // Select: 24, 30, 60
// orientation moved to SHADER CONTROLS (V4_6_0)
renderBtn             // Start render button
```

**Display Elements:**
```javascript
canvas                // WebGPU render target (main preview)
waveformCanvas        // Audio waveform display
progressOverlay       // Render progress modal
progressText          // Progress percentage display
gridFreeBtn           // Toggle beatgrid overlay (future)
```

**Critical Separation:**
```
tempo          → Shader Tempo (visual animation speed)
audioTempoField → Audio Tempo (musical timing/beat positioning)

These are NEVER the same value!
- Shader can animate at 128 BPM
- Audio track can be 124 BPM
- Both independent, both valid

V4_8_0: When audio is loaded, shader tempo is DISABLED
- Audio tempo becomes THE tempo (single source of truth)
- Shader tempo field grayed out, tooltip explains why
- Re-enabled when audio removed
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

**Sets (V4_6_0):**
```javascript
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

**ADMIN Tab:**
```javascript
adminNavItems             // Settings, Utilities, About
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

trackAudioVolumeKnob[track]      // 30px encoder (0-100%)
trackAudioMuteBtn[track]         // Orange button
trackAudioSoloBtn[track]         // Green button
```

**Master Module:**
```javascript
// Scene Launch Buttons (8 total)
sceneLaunchBtn[scene]           // Launch all tracks in scene

// Master Shader Controls
masterShaderMuteBtn             // Purple
masterShaderSoloBtn             // Cyan

// Master Audio Controls
masterAudioVolumeKnob           // 60px encoder
masterAudioMuteBtn              // Orange

// Global Controls
masterTempoKnob                 // 60px encoder (60-200 BPM)
masterZoomKnob                  // 60px encoder (10-100 px/s)
```

**Waveform Panel:**
```javascript
// Per Track (1-4):
trackWaveformCanvas[track]      // Canvas element
trackWaveformOverlay[track]     // Container name + thumbnail

// Nudge Controls (per track)
trackNudgeLeftLarge[track]      // |< button (1 bar left)
trackNudgeLeftSmall[track]      // << button (1 beat left)
trackNudgeRightSmall[track]     // >> button (1 beat right)
trackNudgeRightLarge[track]     // >| button (1 bar right)

// Fixed Overlay
playheadLine                    // Magenta vertical line (z-index 10)
masterGrid                      // White bars overlay (z-index 5) - NOT IMPLEMENTED
```

---

### Keyboard Shortcuts (CREATE Tab) (V4_8_0)

**Audio Navigation:**
| Shortcut | Action |
|----------|--------|
| Spacebar | Play/pause audio |
| Trackpad L/R | Smooth scrub |
| Shift+< | Jump to previous grid line (4 bars) |
| Shift+> | Jump to next grid line (4 bars) |
| G | Toggle beatgrid |

**Reactivity Mode:**
| Shortcut | Action |
|----------|--------|
| Shift+R | Enter/exit reactivity mode |
| ←/→ | Navigate columns (Subdiv/Waveform/Gain/Freq1/Freq2) |
| ↑/↓ | Navigate rows (A/B/C) |
| Shift+← | Decrease value |
| Shift+→ | Increase value |
| Escape | Exit reactivity mode |

**Note:** Hold Shift+Arrow for progressive acceleration (faster value changes).

---

### Keyboard Shortcuts (LIVE Tab)

**Note:** In V4_5_0, this tab is labeled "PERFORM" but internal references remain "live" for code continuity.

**Pattern:** All keyboard actions trigger `showMessage()` with state feedback.

**Categories:**
- Track Selection: `1-5`
- Grid Navigation: `W/A/S/D`
- Playback: `Enter`, `Space`, `Esc`, `Delete`
- Mixer: Arrow keys, `,` `.` `<` `>`
- Visibility: `Shift+M/T/W`
- Zoom: `[ ]`

**Complete list:** See User Guide section 3.4.8

---

### Keyboard Shortcuts (Library Tab)

**Pattern:** Cmd+key for actions, arrow keys for navigation.

**Categories:**
- Load/Update: `Cmd+L`, `Cmd+U`
- Swap: `Cmd+S`, `Cmd+A`
- Navigation: `↑/↓`, `Enter`
- Info: `Cmd+I`

**Complete list:** See User Guide section 3.3.10
- `↑/↓`: Navigate list
- `Enter`: Load container
- `Delete`: Delete (with confirmation)

---

## DATA STRUCTURES

### Container Data Object

**Structure:**
```javascript
containerData = {
  // Metadata
  name: "Bass Drop Tunnel",
  version: "3.8.0",
  created: "2025-11-15T10:30:00Z",
  
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

### UI State (V4_8_0)

```javascript
// Reactivity keyboard mode
let modReactivityMode = false;     // Keyboard mode active
let modSelectedRow = 0;            // 0-2 (A/B/C)
let modSelectedCol = 0;            // 0-4 (Subdiv/Waveform/Gain/Freq1/Freq2)
let modKeyRepeatCount = 0;         // Progressive acceleration counter
let modLastKeyTime = 0;            // Timestamp for acceleration

// Freq knob state
let freqKnobsDisabled = true;      // true when no audio loaded

// Current container tracking
let currentLoadedContainerId = null;  // null = new, "name" = editing existing
```

---


### UI State (V5_0_0)

```javascript
// Transform keyboard mode
let transformKeyboardMode = false;    // Keyboard mode active (Shift+T)
let transformSelectedCol = 0;         // 0-4 (Rotation/Scale/Tile/XY/Pivot)
let transformKeyRepeatCount = 0;      // Progressive acceleration counter
let transformLastKeyTime = 0;         // Timestamp for acceleration
let transformLastEnterTime = 0;       // Double-tap Enter detection

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
```

---

### OmsDropdown Instances (V5_0_0)

```javascript
// Global dropdown instances (11 total in CREATE tab)
window.orientationDropdown           // Orientation (Landscape/Portrait/Square)
window.loopLengthDropdown            // Loop Length (bars display)
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
  - tempo (Shader Tempo)
  - renderDuration (MM:SS.mmm)
  - modAWave/Subdiv
  - modBWave/Subdiv
  - modCWave/Subdiv
  - shaderCode (WGSL)
  ↓
[Timing Engine]
  - duration = parse renderDuration
  - beatCount = (duration * tempo) / 60
  ↓
[Modulator Calculation]
  - For each frame:
    - Calculate time (frame / fps)
    - modA = calculate wave (modAWave, modASubdiv, time, tempo)
    - modB = calculate wave (modBWave, modBSubdiv, time, tempo)
    - modC = calculate wave (modCWave, modCSubdiv, time, tempo)
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
  - Draw: blue amplitude + cyan beat grid
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
  ↓
[Output]
  - Composited shaders → pop-out window
  - Audio → speakers (via audioContext.destination)
```

---

## FUNCTION REFERENCE

**Total Functions:** ~210 (V5_0_0 adds ~15 new functions)

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
  // Returns: controlStateObject

applyControlState(state)
  // Apply control state to UI fields
  // Args: controlStateObject

getCurrentProjectState()
  // Get current LIVE tab state
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

**V4_8_0 Reactivity Mode Functions:**
```javascript
enterReactivityMode()
  // Enter keyboard control mode for modulators
  // Sets: modReactivityMode = true
  // Shows: cyan selection ring

exitReactivityMode()
  // Exit keyboard control mode
  // Clears: selection ring, resets state

updateModSelection()
  // Update visual selection ring on active control
  // Uses: modSelectedRow, modSelectedCol

adjustModValue(delta)
  // Adjust selected modulator value
  // Args: delta (positive or negative)
  // Handles: dropdowns, knobs with appropriate step sizes
```

**V4_8_0 Audio Functions:**
```javascript
parseBPMFromID3(file)
  // Extract BPM from ID3v2 TBPM tag
  // Supports: ID3v2.2, v2.3, v2.4 formats
  // Falls back to: filename parsing, then 120 default
  // Returns: Promise<number>
```

**V4_8_0 OmsDropdown Class:**
```javascript
new OmsDropdown(containerId, options)
  // Custom dropdown with thumbnail support
  // Options: {
  //   showThumbnails: boolean,
  //   thumbnailSize: { w: number, h: number },
  //   onChange: (value, item) => void,
  //   onOpen: async () => void  // Called on every open for refresh
  // }
  // Methods: setItems(items), setValue(value), getValue()
```

---

### AUDIO DOMAIN (Controls Tab)

**Audio Loading/Playback:**
```javascript
loadAudioFile(event)
  // Load audio file from file input
  // Creates: AudioBuffer
  // Updates: UI fields

removeAudio()
  // Remove loaded audio
  // Clears: AudioBuffer, resets UI

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

**Preset Management:**
```javascript
importPreset()
  // Import preset from system presets

deletePreset()
  // Delete user-created preset

getUserPresets()
  // Get array of user presets from localStorage
  // Returns: Array<presetObject>

saveUserPresets(presets)
  // Save user presets to localStorage
  // Args: Array<presetObject>

generateUserPresetCards()
  // Generate UI cards for user presets

loadUserPreset(index)
  // Load user preset by index
  // Args: integer

generatePresetCards()
  // Generate UI cards for system presets

loadPreset(index)
  // Load system preset by index
  // Args: integer

initThumbnailShader(displayCanvas, shaderCode, index)
  // Initialize shader thumbnail rendering
  // Args: canvas, shaderCode, index

updatePresetCardShapes()
  // Update preset card aspect ratios

togglePresetSection(sectionId)
  // Expand/collapse preset section
  // Args: sectionId (string)
```

**UI Functions (V4_5_0):**
```javascript
toggleCodeView()
  // Toggle between Controls view and Code view in CREATE tab
  // Updates codeViewActive state

adminNavSelect(section)
  // Switch ADMIN tab content (Settings, Utilities, About)
  // Args: section (string)

toggleInfoDrawer()
  // Open/close info drawer in ORGANIZE tab
  // Updates infoDrawerOpen state

applyInfoChanges()
  // Apply pending info changes from info drawer
  // Processes pendingInfoChanges object

cancelInfoChanges()
  // Discard pending info changes
  // Clears pendingInfoChanges object

confirmDelete()
  // Handle delete confirmation (status bar pattern)
  // Second press within 3s confirms deletion
```

**Container Building (V4_6_0):**
```javascript
buildContainerObject(name, audioOverride)
  // UNIFIED container builder - single source of truth for all save paths
  // Args: name (string), audioOverride (optional object with audioFilename, audioTempo, etc.)
  // Returns: Complete containerData object
  // Used by: confirmAddToLibrary(), updateContainer(), all save workflows
  // Sources: UI fields for shader, controlStates, timelines
  // Handles: Smart loop validation, tempo from audioTempoField, thumbnail generation

cleanAudioName(filename)
  // Strip extension and BPM patterns from audio filename
  // Args: filename (string) e.g., "Track_128BPM.mp3"
  // Returns: cleaned string e.g., "Track"
  // Used for: Auto-populating render filename
```

**Shader Name Dropdown (V4_6_0):**
```javascript
populateShaderNameDropdown()
  // Populate shader picker dropdown in CREATE tab
  // Sources: Current shader, System Presets, User Presets, Library containers
  // Option values use prefixes: _current, _setfolder, sysPreset:, userPreset:, library:

handleShaderNameDropdownChange(e)
  // Handle shader selection from dropdown
  // Routes by prefix to appropriate load function
  // Preserves current audio when loading new shader
```

**Audio Swap Controls (V4_6_0):**
```javascript
populateAudioSwapDropdown()
  // Populate audio picker dropdown in CREATE tab header
  // Groups: Unused audio first, then In Use
  // Shows only audio files from Library folder

handleAudioSwapSelect(e)
  // Handle audio selection from swap dropdown
  // Loads selected audio file, keeps current shader
  // Triggers folder setup if not configured

removeCurrentAudio()
  // Remove audio from current container (X button)
  // Clears audio fields, keeps shader
```

**Library/Container Management:**
```javascript
addToLibrary()
  // Begin add container workflow
  // Opens: modal dialog

confirmAddToLibrary()
  // Save current state as .oms container
  // Creates: containerData object

cancelAddToLibrary()
  // Cancel add to library workflow

saveContainerToOMS(container)
  // Save .oms file to Library/ folder
  // Args: containerData object
  // Returns: Promise<void>

loadContainer(containerId)
  // Load .oms container from Library/
  // Args: containerId (string) = filename without .oms
  // Returns: Promise<void>

updateContainer(containerId)
  // Update existing .oms container
  // Args: containerId (string)

removeContainer(containerId)
  // Delete .oms file from Library/
  // Args: containerId (string)

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

editContainerName(containerId)
  // Enable inline editing of container name
  // Args: containerId (string)

renameContainer(containerId, newName)
  // Rename container (file + internal name)
  // Args: containerId (string), newName (string)
  // V4.3.0_A: Renames actual .oms file

getLibraryContainers()
  // V4.3.0_A: Returns libraryContainersCache (full data)
  // Returns: Array of container objects

saveLibraryContainers()
  // V4.3.0_A: DEPRECATED (no-op)
  // localStorage no longer used
```

**Library Sort Functions (V4_4_0):**
```javascript
toggleLibrarySort(column)
  // Toggle sort column/direction
  // Args: 'name', 'bpm', or 'audio' (V4_4_0: added 'audio')
  // Calls: updateSortArrows(), refreshLibraryList()

updateSortArrows()
  // Update arrow indicators (▼/▲) in headers
```

**Drag-Drop Import (V4.3.0_A):**
```javascript
handleLibraryAudioDrop(e)
  // Drag audio file → create container
  // Copies audio to Library/
  // Creates .oms with current shader/controls

sanitizeFilename(name)
  // Remove invalid filesystem characters
  // Keeps: brackets [], letters, numbers, spaces
  // Removes: < > : " / \ | ? *
```

**Drag/Drop (V4.4.0):**
```javascript
handleUnifiedDrop(e)
  // Route all external file drops
  // Routes: by file type AND active tab
  // Audio → create container (L) or load (others)
  // Shader → load + compile (all tabs)
  // ZIP → import (L only)
  
isExternalFileDrop(e)
  // Distinguish external files from internal drags
  // Returns: boolean
  // Checks: dataTransfer.types.includes('Files')
```

**Library UI (V4.4.0):**
```javascript
refreshLibraryListRight()
  // Render right panel only (containers)
  // Used after internal changes

renderLibraryThumbnails(containers)
  // Async batch render shader thumbnails
  // Args: Array of container objects
  // Renders: 32x18 canvas per card via initThumbnailShader()
  // Yields: every 5 to avoid UI blocking

selectLibrarySet(setId)
  // Update both panels + dropdown
  // Args: 'presets' or 'all'
  // Syncs with LIVE tab

loadShaderOnly()
  // Shift+Enter: load shader, keep audio
  // Used for shader-only performance
```

**Presets (V4.4.0):**
```javascript
saveShaderToPresets()
  // W tab Save button
  // Save to /Presets/ folder

copyPresetsToLibrary()
  // System presets copy on folder setup
  // Creates initial /Presets/ folder
```

---

### CONTROL STATE DOMAIN (Controls Tab)

**Control State Management:**
```javascript
saveControlState()
  // Begin save control state workflow

confirmSaveControlState()
  // Save named control state preset

cancelSaveControlState()
  // Cancel save workflow

deleteControlState()
  // Delete control state preset

updateControlStatesDropdown()
  // Refresh control states dropdown UI

updateControlStateInfo()
  // Update info display for current state

markAsCustomState()
  // Mark state as "Custom" after user edit
```

---

### LIVE TAB - INITIALIZATION

**Tab Initialization:**
```javascript
initLiveTab()
  // Initialize entire LIVE tab
  // Calls: all init* functions

initLiveHeaders()
  // Initialize track column headers

initLiveScenes()
  // Initialize scene launch buttons

initLiveMixer()
  // Initialize mixer controls (knobs, buttons)

initLiveWaveforms()
  // Initialize waveform display canvases

initLiveTempo()
  // Initialize tempo encoder

initLiveShader()
  // Initialize shader rendering system

initMasterGrid()
  // Initialize Master Grid overlay

initAudioRouting()
  // Initialize Web Audio API routing

initCellSelection()
  // Initialize grid cell selection

initKnobSelection()
  // Initialize knob/track selection
```

---

### LIVE TAB - MASTER CLOCK

```javascript
startMasterClock(actualStartTime)  // #ForReview naming
  // Initialize master clock on first launch
  // Args: actualStartTime (audioContext.currentTime)
  // Sets: trackMasterClockStart

getNextBarBoundary()  // #ForReview naming
  // Calculate next bar boundary for quantization
  // Returns: {barTime: float, when: float}

isAnyTrackPlaying()  // #ForReview naming
  // Check if any track is currently playing
  // Returns: boolean
```

---

### LIVE TAB - AUDIO ROUTING

```javascript
launchCell(track, scene, cellData, startTime)  // V4.4.0: Renamed from playCellAudio
  // Start playing audio/shader for track
  // Args: track (0-3), scene (0-7), cellData, startTime (optional)
  // Creates: AudioBufferSourceNode (if audio present)
  // Calculates: playbackRate, quantized launch time
  // Handles: Shader-only containers (no audio)

// NOTE: stopCellAudio, setupAudioNodes may exist but not found in grep pattern
```

---

### LIVE TAB - MIXER

**Knob Drawing/Updating:**
```javascript
drawKnob()
  // Draw volume knob (per track)
  // Uses: arc drawing, rotation transforms

updateVolume(newVolume)
  // Update track volume
  // Args: newVolume (0-100)
  // Updates: GainNode, knob visual

drawShaderKnob()
  // Draw shader opacity knob (per track)

updateShaderOpacity(newOpacity)
  // Update shader opacity
  // Args: newOpacity (0-100)
  // Updates: canvas.style.opacity, knob visual

drawKnobTempo()
  // Draw master tempo knob (60px)

drawKnobMaster()
  // Draw master volume knob (60px)

updateVolumeMaster(vol)
  // Update master volume
  // Args: vol (0-100)

drawZoomKnob()
  // Draw zoom knob (60px)

recalculateZoomRange()
  // Recalculate pixelsPerSecond range
```

**Meter Animation:**
```javascript
animateMeters()
  // Animate audio level meters
  // Uses: requestAnimationFrame
```

---

### LIVE TAB - GRID/CELLS

**Cell Management:**
```javascript
assignContainerToCell(containerId, containerName, track, scene)
  // Assign container to grid cell
  // Args: containerId, containerName, track (0-3), scene (0-7)
  // Creates: cell visual, stores containerData

updateCellDisplay(track, scene)
  // Update cell visual state
  // Args: track (0-3), scene (0-7)
  // Updates: classes (loaded, playing, selected)

updateCellShader(track, scene)
  // Update shader for cell
  // Args: track (0-3), scene (0-7)

updateSelectionVisuals()
  // Update cell selection highlights

updateSceneSelectionVisuals()
  // Update scene button highlights
```

**Scene Launching:**
```javascript
launchScene(sceneIndex)
  // Launch all 4 tracks in scene
  // Args: sceneIndex (0-7)
  // Calculates: shared quantized launch time
```

---

### LIVE TAB - WAVEFORMS

**Waveform Drawing:**
```javascript
drawWaveformFromAudioBuffer(track, audioBuffer)
  // Draw blue amplitude bars on waveform canvas
  // Args: track (0-3), audioBuffer
  // Uses: containerData.audioStartTime/audioEndTime

drawEmptyWaveform(ctx, canvas)
  // Draw empty/placeholder waveform
  // Args: ctx (CanvasRenderingContext2D), canvas

drawBeatGridOnWaveform(ctx, containerData, audioBuffer, canvasWidth, canvasHeight)
  // Draw cyan/magenta beat grid bars
  // Args: ctx, containerData, audioBuffer, dimensions
  // Spacing: master tempo, not original BPM

drawInfoOverlay(ctx, containerData, canvasHeight)
  // Draw container name overlay on waveform
  // Args: ctx, containerData, canvasHeight
```

**Waveform Positioning:**
```javascript
positionTrackAudioWaveformAtPlayhead(track, audioBuffer)
  // Position waveform so Bar 1 appears at playhead
  // Args: track (0-3), audioBuffer
  // Sets: canvas.style.transform (translateX)

positionTrackAudioWaveformAtCurrentBar(track)
  // Reposition waveform at current bar (tempo change)
  // Args: track (0-3)
  // Calculates: current bar position, maintains alignment
```

**Waveform Scrolling:**
```javascript
startAudioWaveformScroll(track, audioBuffer)
  // Start waveform scroll animation
  // Args: track (0-3), audioBuffer
  // Uses: requestAnimationFrame loop

stopAudioWaveformScroll(track)
  // Stop waveform scroll animation
  // Args: track (0-3)
  // Calls: cancelAnimationFrame

updateWaveformOverlay(track, container)
  // Update waveform overlay (name + thumbnail)
  // Args: track (0-3), container

hideWaveformOverlay(track)
  // Hide waveform overlay
  // Args: track (0-3)
```

---

### LIVE TAB - CLOCK/GRID VISUALIZATION

**Clock Animations:**
```javascript
startClockDotAnimation()
  // Start clock dot animation (magenta dot)
  // Uses: requestAnimationFrame
  // Shows: current bar position

stopClockDotAnimation()
  // Stop clock dot animation

startDAWClockAnimation()
  // Start DAW clock display (bar.beat.tick)
  // Updates: text display with current position

stopDAWClockAnimation()
  // Stop DAW clock display
```

**Master Grid:**
```javascript
toggleMasterGrid()
  // Toggle Master Grid visibility (G key)
  // Updates: masterGridVisible state
  // Redraws: Master Grid canvas

drawMasterGrid()
  // Draw white overlay bars (Master Grid)
  // Spans: all 4 tracks
  // Spacing: master tempo

updateMasterGrid()
  // Update Master Grid after tempo/zoom change
  // Redraws: grid with new spacing
```

---

### LIVE TAB - SHADER

**Shader Management:**
```javascript
getShaderCodeForLive()
  // Get shader code for LIVE rendering
  // Returns: WGSL code string

updateLiveShader(trackIndex, providedShaderCode)
  // Update shader for track
  // Args: trackIndex (0-3), shaderCode (optional)
  // Compiles: new shader, updates pipeline

renderLiveShader()
  // Render all 4 shaders (60fps loop)
  // Uses: requestAnimationFrame
  // Composites: with opacity blending

calculateTrackShaderOpacity(track)
  // Calculate final opacity (manual vs auto)
  // Args: track (0-3)
  // Returns: 0.0-1.0
  // V4_4_0: Detects shader-only via .playing class (not audio source)

updateTrackShaderOpacity(track)
  // Update shader opacity visual
  // Args: track (0-3)
  // Sets: canvas.style.opacity

getActiveShaderTrack(sceneIndex)
  // Get highest-priority visible shader
  // Args: sceneIndex (0-7)
  // Returns: track index or -1

updateLiveShaderOutput(sceneIndex)
  // Update shader output for scene
  // Args: sceneIndex (0-7)
```

**Pop-Out Window:**
```javascript
openPopOutWindow()
  // Open pop-out window for shader display
  // Creates: new window, transfers rendering

closePopOutWindow()
  // Close pop-out window
  // Restores: rendering to main window
```

---

### LIVE TAB - PROJECT STATE

**Project Management:**
```javascript
saveProjectState()
  // Begin save project workflow

confirmSaveProjectState()
  // Save named .omsp project file

cancelSaveProjectState()
  // Cancel save workflow

loadProjectState()
  // Load .omsp project file

deleteProjectState()
  // Delete .omsp project file

updateProjectStatesDropdown()
  // Refresh project states dropdown

updateProjectStateInfo(projectData)
  // Update info display for project

autoSaveProject()
  // Auto-save to default.omsp
  // Throttled: 1 second debounce

autoLoadDefaultProject()
  // Load default.omsp on tab init
```

---

### FILE SYSTEM DOMAIN

**Folder Access:**
```javascript
setupApplicationFolder()
  // Request LIBRARY folder access
  // Uses: File System Access API

storeApplicationFolderHandle(handle)
  // Store folder handle in IndexedDB
  // Args: FileSystemDirectoryHandle

loadApplicationFolderHandle()
  // Load folder handle from IndexedDB
  // Returns: Promise<FileSystemDirectoryHandle>

closeFolderOverlay()
  // Close folder selection overlay

confirmProjectCreation()
  // Confirm and create project folder

cancelProjectCreation()
  // Cancel folder selection
```

**IndexedDB:**
```javascript
openLibraryDB()
  // Open IndexedDB for library data
  // Returns: Promise<IDBDatabase>

getAudioFromLibrary(filename)
  // Get audio file from IndexedDB
  // Args: filename (string)
  // Returns: Promise<File>

checkLibrarySet(autoOpen)
  // Check if LIBRARY folder is set
  // Args: autoOpen (boolean)
  // Returns: boolean

getLibraryContainers()
  // Get containers list from IndexedDB
  // Returns: Array<containerData>

saveLibraryContainers(containers)
  // Save containers list to IndexedDB
  // Args: Array<containerData>
```

**Project Naming:**
```javascript
getDefaultProjectName()
  // Get default project name from localStorage
  // Returns: string or null

setDefaultProjectName(projectName)
  // Set default project name
  // Args: projectName (string)
```

---

### BROADCAST DOMAIN (Future Multi-Instance)

**Communication:**
```javascript
announcePresence()
  // Broadcast instance presence
  // Uses: BroadcastChannel API

broadcastSnapshot()
  // Broadcast current state snapshot

broadcastLibrary()
  // Broadcast library contents

startHeartbeat()
  // Start heartbeat broadcast
  // Keeps: instance alive in network
```

---

### UI/INTERACTION DOMAIN

**Drag & Drop:**
```javascript
setupDragAndDrop()
  // Initialize drag/drop handlers

preventDefaults(e)
  // Prevent default drag behaviors
  // Args: event

handleAudioDrop(e)
  // Handle audio file drop
  // Args: event

handleShaderDrop(e)
  // Handle shader code drop
  // Args: event

handlePresetDrop(e)
  // Handle preset drop onto grid
  // Args: event
```

**Tab Management:**
```javascript
switchTab(tabName)
  // Switch between tabs
  // Args: tabName (string)
  // Shows: tab content, updates active state
```

**Keyboard:**
```javascript
setupInOutArrowKeys()
  // Setup arrow key handlers for loop points
```

**Grid Toggle:**
```javascript
toggleGridFree()
  // Toggle beatgrid overlay (Controls tab)
```

---

### V4_8_0 REACTIVITY DOMAIN

**Modulator Knob Controls:**
```javascript
initGainKnobs()
  // Initialize 3 gain knobs (30×30 canvas each)
  // Range: 0-1, default 0.1
  // Color: Purple (#9d4edd)

initFreqKnobs()
  // Initialize 6 freq knobs (30×30 canvas each)
  // Range: 20-24000 Hz (log scale)
  // Color: Purple (#9d4edd)
  // Push behavior: Freq1 can't exceed Freq2

setFreqKnobsDisabled(disabled)
  // Enable/disable freq knobs based on audio state
  // Args: boolean
  // Visual: Dimmed when disabled, "load audio" tooltip
```

**Reactivity Keyboard Mode:**
```javascript
enterReactivityMode()
  // Enter keyboard navigation mode (Shift+R)
  // Shows cyan selection ring on active control
  // Message bar shows "Reactivity Mode"

exitReactivityMode()
  // Exit keyboard mode (Escape)
  // Clears selection ring

adjustModValue(delta)
  // Adjust selected control value
  // Args: delta (positive or negative)
  // Handles: Dropdowns (cycle options), Knobs (adjust value)
  // Progressive acceleration when key held

updateModSelection()
  // Update visual selection feedback
  // Draws cyan ring on selected control
  // Updates message bar with control info
```

**OmsDropdown Class:**
```javascript
class OmsDropdown
  constructor(containerId, options)
    // Create custom dropdown component
    // Options: showThumbnails, thumbnailSize, onChange, onOpen
  
  setOptions(items)
    // Populate dropdown options
    // Items: [{value, label, shaderCode, separator}]
  
  setValue(value)
    // Set selected value programmatically
  
  getValue()
    // Get current selected value
    // Returns: string
  
  open() / close()
    // Programmatic control
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

**Transform Keyboard Mode:**
```javascript
enterTransformMode()
  // Enter keyboard mode (Shift+T)
  // Shows cyan selection ring
  // Message bar shows "Transform Mode"

exitTransformMode()
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

**Transform Helpers:**
```javascript
getModDropdown(row, col)
  // Get OmsDropdown instance for reactivity grid position
  // Args: row (0-2 = A/B/C), col (0-1 = Subdiv/Waveform)
  // Returns: OmsDropdown instance or null
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

## NAMING CONVENTIONS

### ID Naming Rules

**Buttons:**
```
[action]Btn
Examples: playBtn, loopBtn, renderBtn, uploadBtn
```

**Input Fields:**
```
Audio timing: [name]Field
Examples: audioTempoField, audioStartField, audioEndField

Other inputs: [name]
Examples: tempo, filename, resolution
```

**Display Fields:**
```
[name]
Examples: audioFileName, loopLength, messageBar
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

### Variable Naming Rules

**Arrays:**
```javascript
// Per-track arrays (always length 4, index 0-3)
trackShaderOpacity = [100, 100, 100, 100];
trackAudioVolume = [75, 75, 75, 75];
trackAudioPlaying = [false, false, false, false];
```

**Maps/Objects:**
```javascript
// Use Map for dynamic collections
audioBuffers = new Map(); // containerName → AudioBuffer

// Use objects for structured data
containerData = {
  name: "...",
  version: "...",
  // ...
};
```

**Functions:**
```javascript
// camelCase, verb-first
loadAudioFile()
playCellAudio()
calculateModulator()
updateShaderUniforms()
```

**Constants:**
```javascript
// UPPER_SNAKE_CASE
const MAX_TRACKS = 4;
const MAX_SCENES = 8;
const MIN_BPM = 60;
const MAX_BPM = 200;
```

---

## VERSION COMPATIBILITY

**File Format Versioning:**
```javascript
// All files include version field
{
  version: "3.8.0",
  // ...
}

// Forward compatibility: newer versions can read older files
// Backward compatibility: not guaranteed (add migration if needed)
```

**Migration Strategy:**
```javascript
function migrateContainer(containerData) {
  const version = containerData.version;
  
  // Add migration logic as needed
  if (compareVersions(version, "3.8.0") < 0) {
    // Migrate from pre-3.8.0
    // e.g., add new fields with defaults
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
- FFT analysis: 2048 samples per track
- Waveform scroll: CSS transform (GPU accelerated)

**Optimization Techniques:**
- Shader compilation cached per container
- Audio buffers cached in Map
- Waveform canvas pre-rendered (not drawn per frame)
- Beat grid drawn once (only on tempo change)
- requestAnimationFrame for all animations

**Browser Requirements:**
- WebGPU support (Chrome 113+, Edge 113+)
- Web Audio API (all modern browsers)
- File System Access API (Chrome/Edge only)
- Minimum 4GB RAM (for audio buffers)
- Dedicated GPU recommended

---

## END OF SYSTEM REFERENCE


