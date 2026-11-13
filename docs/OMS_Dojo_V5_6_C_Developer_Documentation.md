# ONEMANSHYO App [Beta V5_6_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_6_0  
**Date:** January 11, 2026  
**License:** GPL-3.0  
**Line Count:** 1260 (V5_5_0) → 799 (V5_6_0)  
**Document Purpose:** Technical implementation reference for V5_6_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [CORTEX Changes](#cortex-changes)
3. [Per-Track VFX System](#per-track-vfx-system)
4. [Per-Track RGB Control](#per-track-rgb-control)
5. [Effect Pool Systems](#effect-pool-systems)
6. [Unified Mic System](#unified-mic-system)
7. [Transform Keyboard Redesign](#transform-keyboard-redesign)
8. [PIP Camera Enhancements](#pip-camera-enhancements)
9. [Audio Control Improvements](#audio-control-improvements)
10. [Schema Additions](#schema-additions)
11. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_6_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  PER-TRACK VFX SYSTEM                                               │
│  Independent WebGPU pipeline per track with RGB color control       │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Shader → Track VFX → RGB Multiply → Composite → MAIN VFX    │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  EFFECT POOL SYSTEMS                                                │
│  Centralized FX/VFX definitions with per-track slot assignments     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Pool constants │ ADMIN dropdowns │ Dynamic knob labels       │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  UNIFIED MIC SYSTEM                                                 │
│  Voice-reactive shaders without audio file requirement              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ micSource → micGain → modA/B/C Highpass → Reactivity        │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  TRANSFORM KEYBOARD REDESIGN                                        │
│  Direct selection, freeze/resume for live performance               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Bare T entry │ 1-5 direct select │ Enter freeze/resume      │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Pre-Composite VFX** - Effects apply per-track before CSS layering
2. **Pool Architecture** - Centralized definitions, per-track assignments
3. **Unified Audio Input** - Mic and file audio share same reactivity chain
4. **Direct Selection** - Number keys map directly to transform controls
5. **WYSIWYG Render** - PIP composite matches preview scaling

---

## CORTEX CHANGES

### 1. Shader Uniform Rename ⚠️ BREAKING

**Change:** FFT → Audio uniform names

| Old Name | New Name |
|----------|----------|
| `shaderModAFFT` | `shaderModAAudio` |
| `shaderModBFFT` | `shaderModBAudio` |
| `shaderModCFFT` | `shaderModCAudio` |

**Rationale:** Names now match implementation. System uses bandpass filtering + time-domain peak detection since V5_3_0, not FFT bin averaging.

**Updated Locations (3):**
```javascript
// 1. SHADER_UNIFORM_STRUCT (WGSL definition)
const SHADER_UNIFORM_STRUCT = `
struct Uniforms {
    // ...
    shaderModAAudio: f32,  // Was shaderModAFFT
    shaderModBAudio: f32,  // Was shaderModBFFT
    shaderModCAudio: f32,  // Was shaderModCFFT
    // ...
}`;

// 2. createUniformData() factory
function createUniformData(p) {
    return new Float32Array([
        // ...
        p.shaderModAAudio,  // Was p.shaderModAFFT
        p.shaderModBAudio,
        p.shaderModCAudio,
        // ...
    ]);
}

// 3. render() calls - all uniform references
```

**Migration Script:**
```bash
for f in *.oms; do
  sed -i '' 's/shaderModAFFT/shaderModAAudio/g; s/shaderModBFFT/shaderModBAudio/g; s/shaderModCFFT/shaderModCAudio/g' "$f"
done
```

### 2. VFX Params Struct Expansion

**Buffer Size:** 16 → 32 bytes

```wgsl
// Old (V5_5_0)
struct Params { 
    amount: f32,
    time: f32,
    pad1: f32,
    pad2: f32
}

// New (V5_6_0)
struct Params { 
    amount: f32,
    time: f32,
    rgbR: f32,      // NEW: Red multiplier
    rgbG: f32,      // NEW: Green multiplier
    rgbB: f32,      // NEW: Blue multiplier
    pad1: f32,
    pad2: f32,
    pad3: f32
}
```

**All 10 VFX shaders updated:**
```wgsl
// RGB multiply at end of each effect shader
col = vec4(col.r * params.rgbR, col.g * params.rgbG, col.b * params.rgbB, col.a);
```

---

## PER-TRACK VFX SYSTEM

### Architecture

```
Track 1 Shader → Track 1 VFX Pipeline → ┐
Track 2 Shader → Track 2 VFX Pipeline → ├→ CSS Composite → MAIN VFX → Output
Track 3 Shader → Track 3 VFX Pipeline → ┤
Track 4 Shader → Track 4 VFX Pipeline → ┘
```

### State Arrays

```javascript
// Value and kill state (4 tracks × 3 slots)
let trackVFXValues = [[0,0,0], [0,0,0], [0,0,0], [0,0,0]];
let trackVFXKilled = [[false,false,false], [false,false,false], [false,false,false], [false,false,false]];

// WebGPU resources per track
let trackVFXCanvases = [null, null, null, null];
let trackVFXContexts = [null, null, null, null];
let trackVFXPipelines = [null, null, null, null];
let trackVFXUniformBuffers = [null, null, null, null];
let trackVFXCurrentEffect = [null, null, null, null];  // null = no VFX
let trackVFXInitialized = [false, false, false, false];
```

### Pipeline Functions

```javascript
// Initialize VFX pipeline for track
function initTrackVFXPipeline(trackIndex) {
    if (trackVFXInitialized[trackIndex]) return;
    
    // Create offscreen canvas
    trackVFXCanvases[trackIndex] = document.createElement('canvas');
    trackVFXContexts[trackIndex] = trackVFXCanvases[trackIndex].getContext('webgpu');
    
    trackVFXInitialized[trackIndex] = true;
}

// Build pipeline for specific effect
function buildTrackVFXPipeline(trackIndex, effectName) {
    const shaderCode = getVFXShaderCode(effectName);  // From VISUAL_VFX_POOL
    
    const shaderModule = device.createShaderModule({
        code: shaderCode
    });
    
    trackVFXPipelines[trackIndex] = device.createRenderPipeline({
        layout: 'auto',
        vertex: { module: shaderModule, entryPoint: 'vs_main' },
        fragment: { module: shaderModule, entryPoint: 'fs_main', targets: [{ format }] }
    });
    
    trackVFXCurrentEffect[trackIndex] = effectName;
}

// Render VFX pass for track
function renderTrackVFXPass(trackIndex) {
    if (!trackHasActiveVFX(trackIndex)) return;
    
    const effect = getFirstActiveVFXForTrack(trackIndex);
    if (trackVFXCurrentEffect[trackIndex] !== effect) {
        buildTrackVFXPipeline(trackIndex, effect);
    }
    
    // Get RGB values
    const rgb = trackRGBValues[trackIndex];
    
    // Update uniform buffer (32 bytes)
    const data = new Float32Array([
        trackVFXValues[trackIndex][0],  // amount
        performance.now() / 1000,        // time
        rgb[0] / 0.75,                   // rgbR (0.75 = unity)
        rgb[1] / 0.75,                   // rgbG
        rgb[2] / 0.75,                   // rgbB
        0, 0, 0                          // padding
    ]);
    
    device.queue.writeBuffer(trackVFXUniformBuffers[trackIndex], 0, data);
    
    // Execute render pass
    // ...
}
```

---

## PER-TRACK RGB CONTROL

### State Arrays

```javascript
let trackRGBValues = [
    [0.75, 0.75, 0.75],  // Track 1: R, G, B
    [0.75, 0.75, 0.75],  // Track 2
    [0.75, 0.75, 0.75],  // Track 3
    [0.75, 0.75, 0.75]   // Track 4
];
let trackRGBKilled = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
    [false, false, false]
];
```

### Value Mapping

| Knob Value | Multiplier | Result |
|------------|------------|--------|
| 0 | 0× | Channel removed |
| 0.75 | 1× | Unity (no change) |
| 1.0 | 1.33× | Boost |
| Killed | 0× | Channel black |

### UI Implementation

```html
<!-- RGB knobs in FX overlay (20×20 purple) -->
<div class="track-rgb-panel">
    <div class="track-rgb-label">R</div>
    <canvas id="track1RgbRKnob" width="20" height="20"></canvas>
    <div class="track-rgb-label">G</div>
    <canvas id="track1RgbGKnob" width="20" height="20"></canvas>
    <div class="track-rgb-label">B</div>
    <canvas id="track1RgbBKnob" width="20" height="20"></canvas>
</div>
```

---

## EFFECT POOL SYSTEMS

### Visual VFX Pool

```javascript
const VISUAL_VFX_POOL = [
    { id: 'off', name: 'Off', abbrev: 'OFF', desc: 'No effect', implemented: true },
    { id: 'rgbsplit', name: 'RGB Split', abbrev: 'RGB', desc: 'Chromatic aberration', implemented: true },
    { id: 'glitch', name: 'Glitch', abbrev: 'GLI', desc: 'Digital glitch', implemented: true },
    { id: 'pixelate', name: 'Pixelate', abbrev: 'PIX', desc: 'Pixelation', implemented: true },
    { id: 'scanlines', name: 'Scanlines', abbrev: 'SCN', desc: 'CRT scanlines', implemented: true },
    { id: 'mirror', name: 'Mirror', abbrev: 'MIR', desc: 'Mirror reflection', implemented: true },
    { id: 'kaleido', name: 'Kaleidoscope', abbrev: 'KAL', desc: 'Kaleidoscope', implemented: true },
    { id: 'blur', name: 'Blur', abbrev: 'BLR', desc: 'Gaussian blur', implemented: false },
    { id: 'sharpen', name: 'Sharpen', abbrev: 'SHP', desc: 'Edge sharpen', implemented: false },
    { id: 'invert', name: 'Invert', abbrev: 'INV', desc: 'Color invert', implemented: false }
];
```

### Audio FX Pool

```javascript
const AUDIO_FX_POOL = [
    { id: 'washout', name: 'Washout', abbrev: 'WO', desc: 'HPF Sweep', implemented: true },
    { id: 'dubby', name: 'Dubby', abbrev: 'DUB', desc: 'Dub Echo', implemented: true },
    { id: 'break', name: 'Break', abbrev: 'BRK', desc: 'Vinyl Stop', implemented: true },
    { id: 'reverb', name: 'Reverb', abbrev: 'REV', desc: 'Room Reverb', implemented: false },
    { id: 'phaser', name: 'Phaser', abbrev: 'PHA', desc: 'Phase Shift', implemented: false },
    { id: 'flanger', name: 'Flanger', abbrev: 'FLA', desc: 'Flange', implemented: false },
    { id: 'bitcrush', name: 'BitCrush', abbrev: 'BIT', desc: 'Bit Reducer', implemented: false },
    { id: 'lpf', name: 'LPF', abbrev: 'LPF', desc: 'Low Pass Filter', implemented: false },
    { id: 'gate', name: 'Gate', abbrev: 'GAT', desc: 'Noise Gate', implemented: false },
    { id: 'ringmod', name: 'Ring Mod', abbrev: 'RNG', desc: 'Ring Modulation', implemented: false }
];
```

### Per-Track Assignments

```javascript
let fxModuleAssignments = {
    track1: ['break', 'dubby', 'washout'],
    track2: ['break', 'dubby', 'washout'],
    track3: ['break', 'dubby', 'washout'],
    track4: ['break', 'dubby', 'washout']
};

let vfxModuleAssignments = {
    track1: ['rgbsplit', 'glitch', 'pixelate'],
    track2: ['rgbsplit', 'glitch', 'pixelate'],
    track3: ['rgbsplit', 'glitch', 'pixelate'],
    track4: ['rgbsplit', 'glitch', 'pixelate']
};
```

### Pool Access Functions

```javascript
function getVFXFromPool(effectId) {
    return VISUAL_VFX_POOL.find(e => e.id === effectId);
}

function getVFXForSlot(module, slotIndex) {
    const assignments = vfxModuleAssignments[module];
    if (!assignments || slotIndex >= assignments.length) return null;
    return getVFXFromPool(assignments[slotIndex]);
}

function setVFXModuleSlot(module, slot, effectId) {
    vfxModuleAssignments[module][slot] = effectId;
    // Update UI, rebuild pipeline if needed
}
```

---

## UNIFIED MIC SYSTEM

### Architecture

```
MIC: micSource → micGainNode ─┬→ micAnalyser (meter display)
                              ├→ modAHighpass (bass reactivity)
                              ├→ modBHighpass (mid reactivity)
                              └→ modCHighpass (treble reactivity)
```

### State Variables

```javascript
let micStream = null;           // MediaStream from getUserMedia
let micSource = null;           // MediaStreamSource in main audioContext
let micGainNode = null;         // GainNode controlled by CREATE header knob
let micAnalyser = null;         // AnalyserNode for meters
let micMeterAnimationId = null;
let micMuted = false;
let micVolume = 0.5;            // 0-1 (0.5 = 0dB)
let micDevices = [];            // Available input devices
```

### Key Functions

```javascript
async function startMicStream() {
    if (micStream) return;  // Already running
    
    try {
        const constraints = { audio: { deviceId: micSourceId } };
        micStream = await navigator.mediaDevices.getUserMedia(constraints);
        
        // Connect to main audioContext (not separate context)
        micSource = audioContext.createMediaStreamSource(micStream);
        micGainNode = audioContext.createGain();
        micAnalyser = audioContext.createAnalyser();
        
        micSource.connect(micGainNode);
        micGainNode.connect(micAnalyser);
        
        // Connect to reactivity filters
        connectMicToReactivity();
        
    } catch (e) {
        showMessage('Mic access denied', 'error');
    }
}

function connectMicToReactivity() {
    // Ensure mod filters exist
    ensureModFiltersExist();
    
    // Mic feeds same filters as audio file
    micGainNode.connect(modAHighpass);
    micGainNode.connect(modBHighpass);
    micGainNode.connect(modCHighpass);
    
    // Enable freq knobs
    updateFreqKnobsState();
}

function stopMicStream() {
    if (!micStream) return;
    
    micStream.getTracks().forEach(t => t.stop());
    micStream = null;
    
    if (micSource) micSource.disconnect();
    micSource = null;
    
    updateFreqKnobsState();
}
```

### Freq Knobs State Logic

```javascript
function updateFreqKnobsState() {
    // Freq knobs enabled when audio loaded OR mic unmuted
    const hasAudio = !!audioBuffer;
    const hasMic = micStream && !micMuted;
    
    freqKnobsDisabled = !(hasAudio || hasMic);
    
    // Update visual state
    document.querySelectorAll('.freq-knob').forEach(k => {
        k.classList.toggle('disabled', freqKnobsDisabled);
    });
}
```

---

## TRANSFORM KEYBOARD REDESIGN

### Key Mappings

| Key | Action |
|-----|--------|
| T | Enter/Exit Transform mode (was Shift+T) |
| 1 | Select Rotation |
| 2 | Select Scale |
| 3 | Select Tile |
| 4 | Select Translate |
| 5 | Select Pivot |
| ↑↓ | Cycle mode (Stop→CW→CCW→PingPong) |
| ←→ | Adjust speed (cols 1-3) or position (cols 4-5) |
| Enter | Freeze/Resume animation |
| 2×Enter | Reset to default (300ms window) |
| Esc | Exit mode |

### Freeze/Resume System

```javascript
let transformPreviousModes = {
    rotation: null,  // Saved mode when frozen
    scale: null,
    tile: null
};

function handleTransformEnter() {
    const transform = getSelectedTransform();
    const config = transformModes[transform];
    
    // Check for double-tap reset (300ms window)
    const now = Date.now();
    if (now - transformLastEnterTime < 300) {
        resetTransformToDefault(transform);
        transformLastEnterTime = 0;
        return;
    }
    transformLastEnterTime = now;
    
    if (config.mode !== 'stop') {
        // Freeze: Save mode and stop
        transformPreviousModes[transform] = config.mode;
        config.mode = 'stop';
        showMessage(`${transform}: Frozen`, 'state');
    } else if (transformPreviousModes[transform]) {
        // Resume: Restore saved mode
        config.mode = transformPreviousModes[transform];
        transformPreviousModes[transform] = null;
        showMessage(`${transform}: ${config.mode}`, 'state');
    }
    
    updateTransformUI(transform);
}
```

---

## PIP CAMERA ENHANCEMENTS

### Three-Dropdown Settings

```javascript
// Aspect options
const PIP_ASPECTS = ['16:9', '9:16', '1:1'];

// Size options (px values depend on aspect)
const PIP_SIZES = ['small', 'medium', 'large'];

// Shape options
const PIP_SHAPES = ['rectangle', 'rounded', 'circle'];

function getPIPDimensions(aspect, size) {
    const sizeMap = {
        '16:9': { small: [160, 90], medium: [240, 135], large: [320, 180] },
        '9:16': { small: [90, 160], medium: [135, 240], large: [180, 320] },
        '1:1':  { small: [120, 120], medium: [200, 200], large: [320, 320] }
    };
    return sizeMap[aspect][size];
}
```

### PIP3 Settings Panel

```javascript
function showPip3SettingsPanel(event) {
    const panel = document.getElementById('pip3-settings-panel');
    panel.style.display = 'block';
    
    // Position near click
    panel.style.left = event.clientX + 'px';
    panel.style.top = event.clientY + 'px';
    
    updatePip3SettingsUI();
}

function updatePip3SettingsUI() {
    // Update button states to match current settings
    document.querySelectorAll('.pip3-aspect-btn').forEach(btn => {
        btn.classList.toggle('active', btn.dataset.aspect === pip3Settings.aspect);
    });
    // Same for size, shape, color
}
```

### Render Composite (WYSIWYG)

```javascript
function compositePipToCanvas(renderCanvas) {
    if (!pip3Visible || !pip3Video) return;
    
    const ctx = renderCanvas.getContext('2d');
    const pipEl = document.getElementById('pip3-container');
    const previewEl = document.getElementById('preview-container');
    
    // Get actual rendered sizes
    const pipRect = pipEl.getBoundingClientRect();
    const previewRect = previewEl.getBoundingClientRect();
    
    // Calculate scale ratio (preview → render)
    const scaleX = renderCanvas.width / previewRect.width;
    const scaleY = renderCanvas.height / previewRect.height;
    
    // Calculate position in render coordinates
    const pipX = (pipRect.left - previewRect.left) * scaleX;
    const pipY = (pipRect.top - previewRect.top) * scaleY;
    const pipW = pipRect.width * scaleX;
    const pipH = pipRect.height * scaleY;
    
    // Draw video frame
    ctx.drawImage(pip3Video, pipX, pipY, pipW, pipH);
}
```

---

## AUDIO CONTROL IMPROVEMENTS

### CREATE Header Knobs

```html
<!-- Audio/Mic header controls -->
<div class="create-audio-header">
    <span>AUDIO</span>
    <canvas id="createAudioVolumeKnob" width="24" height="24"></canvas>
    <canvas id="createAudioMeter" width="4" height="24"></canvas>
    <canvas id="createMicVolumeKnob" width="24" height="24"></canvas>
    <canvas id="createMicMeter" width="4" height="24"></canvas>
    <span id="audioFileName">No audio loaded</span>
</div>
```

### Knob Interaction

```javascript
// Tap = toggle mute
// Drag = adjust volume (1dB steps)
// Shift+Drag = fine adjustment

function handleCreateAudioKnobDown(e) {
    createAudioDragging = true;
    createAudioDragLastY = e.clientY;
    createAudioDidDrag = false;
}

function handleCreateAudioKnobMove(e) {
    if (!createAudioDragging) return;
    
    const deltaY = createAudioDragLastY - e.clientY;
    if (Math.abs(deltaY) > 2) createAudioDidDrag = true;
    
    const step = e.shiftKey ? 0.01 : roundToNearestDb(0.05);
    createAudioVolume = Math.max(0, Math.min(1, createAudioVolume + deltaY * step / 50));
    
    updateCreateAudioGain();
    drawCreateAudioVolumeKnob();
    
    createAudioDragLastY = e.clientY;
}

function handleCreateAudioKnobUp(e) {
    if (!createAudioDidDrag) {
        // Tap = toggle mute
        createAudioMuted = !createAudioMuted;
        updateCreateAudioGain();
    }
    createAudioDragging = false;
}
```

### N-Panel Mute/Solo

```javascript
let modMuteState = [false, false, false];  // A, B, C
let modSoloState = [false, false, false];

function toggleModMute(modIndex) {
    modMuteState[modIndex] = !modMuteState[modIndex];
    updateModMuteSoloUI();
}

function toggleModSolo(modIndex) {
    // Radio behavior: only one solo at a time
    const wasOn = modSoloState[modIndex];
    modSoloState = [false, false, false];
    if (!wasOn) modSoloState[modIndex] = true;
    updateModMuteSoloUI();
}

// Keyboard: , = mute, . = solo (in N-panel)
```

### Stepped Frequency Presets

```javascript
const FREQ_PRESETS = [0, 5, 10, 20, 30, 50, 75, 100, 150, 200, 300, 500, 1000, 2000, 5000, 10000, 20000];

function snapToNearestPreset(value) {
    return FREQ_PRESETS.reduce((prev, curr) => 
        Math.abs(curr - value) < Math.abs(prev - value) ? curr : prev
    );
}

// Arrow keys snap to presets
// Mouse drag remains continuous
```

---

## SCHEMA ADDITIONS

### .oms controlStates (V5_6_0)

```javascript
controlStates: {
    // Existing fields...
    
    // Audio/Mic Volumes
    createAudioVolume: 0.5,      // 0-1 (0.5 = 0dB)
    createAudioMuted: false,
    micVolume: 0.5,
    micMuted: false,
    
    // Device IDs
    micSourceId: 'default',
    camSourceId: 'default',
    camMirror: false,
    
    // PIP3 Settings
    pip3Aspect: '1:1',
    pip3Size: 'large',
    pip3Shape: 'rectangle',
    pip3Color: '#00d4ff'
}
```

### .omsp Additions (V5_6_0)

```javascript
{
    // Existing fields...
    
    // Pool Assignments
    fxModuleAssignments: {
        track1: ['break', 'dubby', 'washout'],
        track2: ['break', 'dubby', 'washout'],
        track3: ['break', 'dubby', 'washout'],
        track4: ['break', 'dubby', 'washout']
    },
    vfxModuleAssignments: {
        track1: ['rgbsplit', 'glitch', 'pixelate'],
        track2: ['rgbsplit', 'glitch', 'pixelate'],
        track3: ['rgbsplit', 'glitch', 'pixelate'],
        track4: ['rgbsplit', 'glitch', 'pixelate']
    },
    
    // Mic Settings
    micSettings: {
        sourceId: 'default',
        muted: false,
        volume: 0.5
    }
}
```

---

## TESTING NOTES

### Per-Track VFX Tests

1. **Pipeline Creation:** VFX pipeline initializes per track
2. **Effect Assignment:** ADMIN dropdowns update knob labels
3. **RGB Multiply:** Color channels affect track output
4. **Kill State:** Click-to-kill zeros channel
5. **Independence:** Track 1 VFX doesn't affect Track 2

### Pool System Tests

1. **Pool Access:** getVFXFromPool returns correct effect
2. **Slot Assignment:** setVFXModuleSlot updates state
3. **Persistence:** Assignments saved to .omsp
4. **Non-Implemented:** "(coming soon)" shows in hover

### Mic System Tests

1. **Permission:** getUserMedia prompts once
2. **Reactivity:** Shaders respond to voice without audio file
3. **Freq Knobs:** Enabled when mic unmuted
4. **Coexistence:** Mic + file audio work together
5. **Device Selection:** ADMIN dropdown changes source

### Transform Tests

1. **Direct Select:** 1-5 keys select controls
2. **Mode Cycle:** ↑↓ cycles Stop→CW→CCW→PingPong
3. **Freeze:** Enter pauses animation
4. **Resume:** Enter restores saved mode
5. **Reset:** Double-Enter resets to default

### PIP Tests

1. **Three Dropdowns:** Aspect/Size/Shape all work
2. **Settings Panel:** Click PIP3 opens panel
3. **Render Composite:** PIP3 appears in rendered output
4. **WYSIWYG:** Render size matches preview ratio
5. **Persistence:** Settings saved to .oms/.omsp

### Audio Controls Tests

1. **Header Knobs:** Drag adjusts, tap mutes
2. **Meters:** Level display updates in real-time
3. **N-Panel M/S:** Mute/Solo affect reactivity
4. **Freq Presets:** Arrow keys snap to values

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_6_0
Date: January 11, 2026
License: GPL-3.0
