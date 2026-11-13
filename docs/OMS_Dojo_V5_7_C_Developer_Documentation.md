# ONEMANSHYO App [Beta V5_7_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_7_0  
**Date:** January 16, 2026  
**License:** GPL-3.0  
**Line Count:** 5,594  
**Document Purpose:** Technical implementation reference for developers

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [CORTEX Systems](#cortex-systems)
3. [Audio Analysis System](#audio-analysis-system)
4. [PIP Camera System](#pip-camera-system)
5. [Audio Focus Mode](#audio-focus-mode)
6. [JKL Transport System](#jkl-transport-system)
7. [Render Pipeline](#render-pipeline)
8. [BumpR StickRs System](#bumpr-stickrs-system)
9. [Signal Flow Diagrams](#signal-flow-diagrams)
10. [Function Reference](#function-reference)
11. [State Variables Reference](#state-variables-reference)
12. [Schema Specifications](#schema-specifications)
13. [Keyboard System](#keyboard-system)
14. [CSS Architecture](#css-architecture)

---

## 1. ARCHITECTURE OVERVIEW

### Single-File Philosophy

OMS App is a single HTML file (~5,594 lines) containing all HTML, CSS, and JavaScript. This enables:
- Zero installation (open in browser)
- Offline capability
- Portability (copy file anywhere)
- No build process required

### Tab Architecture

| Tab | Purpose | Color |
|-----|---------|-------|
| CREATE | Shader authoring, audio reactivity | Cyan |
| ORGANIZE | Container library management | Yellow |
| PERFORM | Live mixing, 4-track grid | Magenta |
| ADMIN | Settings, utilities, about | Grey |

### Leaf Node Hierarchy

**Tier 1: CORTEX** - Never modify (change = cascade failure)
- Master Clock (`audioContext.currentTime`, `trackMasterClockStart`)
- WebGPU device/context
- Uniform buffer structure (112 bytes)
- .oms container schema
- Message bar system

**Tier 2: NODES** - Trace dependencies before modifying
- Track state arrays
- Library cache system
- Audio routing nodes
- BumpR slot arrays

**Tier 3: LEAFS** - Safe to modify
- UI styling
- Keyboard shortcuts
- Colors, fonts
- Help text

---

## 2. CORTEX SYSTEMS

### loopLength Uniform (V5_7_0)

New uniform at index 25 provides shader access to loop length for perfect infinite loops.

```javascript
// Index 25: padding1 → loopLength
// Buffer size unchanged: 112 bytes (28 floats × 4)

// Shader usage:
// let cycle = fract(uniforms.beat / uniforms.loopLength);
```

**createUniformData() sources:**

| Location | loopLength Source |
|----------|-------------------|
| CREATE render() | shaderLoopLength field |
| Thumbnail preview | Static 16.0 |
| LIVE inset render | containerCS loopBeats |
| LIVE pop-out render | containerCS loopBeats |

### Master Clock System

```javascript
// Browser clock - never resets
audioContext.currentTime  // Seconds since AudioContext creation

// Session start - set once per session
trackMasterClockStart     // audioContext.currentTime when first track launches

// Beat calculation
function trackAudioMasterClock() {
    if (!trackMasterClockStart) return 0;
    return (audioContext.currentTime - trackMasterClockStart) * (gridState.masterTempo / 60);
}

function masterBeatNow() {
    return trackAudioMasterClock();
}

function masterBarNow() {
    return masterBeatNow() / 4;
}
```

### Uniform Buffer Structure (112 bytes)

```javascript
const SHADER_UNIFORM_STRUCT = `
struct Uniforms {
    time: f32,              // 0: Seconds since start
    tempo: f32,             // 1: BPM
    modA: f32,              // 2: Combined bass (legacy)
    modB: f32,              // 3: Combined mids (legacy)
    modC: f32,              // 4: Combined treble (legacy)
    beat: f32,              // 5: Beat position
    resolutionX: f32,       // 6: Canvas width
    resolutionY: f32,       // 7: Canvas height
    shaderModAWave: f32,    // 8: Mod A waveform
    shaderModAGain: f32,    // 9: Mod A gain
    shaderModAAudio: f32,   // 10: Mod A audio level
    shaderModBWave: f32,    // 11: Mod B waveform
    shaderModBGain: f32,    // 12: Mod B gain
    shaderModBAudio: f32,   // 13: Mod B audio level
    shaderModCWave: f32,    // 14: Mod C waveform
    shaderModCGain: f32,    // 15: Mod C gain
    shaderModCAudio: f32,   // 16: Mod C audio level
    transformRotation: f32, // 17: Rotation (radians)
    transformScaleX: f32,   // 18: Scale X
    transformScaleY: f32,   // 19: Scale Y
    transformTranslateX: f32, // 20: Position X
    transformTranslateY: f32, // 21: Position Y
    transformPivotX: f32,   // 22: Pivot X
    transformPivotY: f32,   // 23: Pivot Y
    transformTile: f32,     // 24: Tile count
    loopLength: f32,        // 25: Loop length in beats (V5_7_0)
    padding1: f32,          // 26: GPU alignment
    padding2: f32           // 27: GPU alignment
}
@group(0) @binding(0) var<uniform> uniforms: Uniforms;
`;

const UNIFORM_BUFFER_SIZE = 112;  // 28 floats × 4 bytes
```

### Message Bar Two-Phase Pattern (V5_6_0)

```javascript
// CORTEX - single entry point for user feedback
showMessage(text, type, onClick)

// Types:
// 'info'  - Cyan #00d4ff - help text, shortcuts (on panel ENTRY)
// 'state' - Orange #ffa500 - values, confirmations (on value CHANGE)
// 'error' - Magenta #ff006e - failures

// Two-phase pattern for panels:
// Phase 1 (entry): showMessage('Volume (←→ adjust, Enter=kill)', 'info')
// Phase 2 (change): showMessage('Audio: +1.5 dB', 'state')
```

---

## 3. AUDIO ANALYSIS SYSTEM

### Frequency Filter Sync (V5_6_01a)

Freq1/Freq2 knobs now sync to bandpass filters in real-time.

**Signal Chain:**
```
Audio/Mic Source → modHighpass (BiquadFilter) → modLowpass (BiquadFilter) → modAnalyser → analyzeAudio() → ADSR envelope → shaderModAudio uniform
```

**Fixes Applied:**

| Fix | Description |
|-----|-------------|
| Freq Knob Drag → Filter Sync | Added `updateBandpassFilters()` call to `updateKnob1()`, `updateKnob2()` |
| Container Load → Filter Sync | Added `updateBandpassFilters()` call to `applyControlState()` |
| Q Values from Globals | Changed hardcoded `Q=0.7` to `modAQ`/`modBQ`/`modCQ` |
| Freq Range Constants | Changed MIN_FREQ=1, MAX_FREQ=20000 (was 20Hz-24kHz) |

**updateBandpassFilters() Function:**

```javascript
function updateBandpassFilters() {
    // Get current freq values from inputs
    const aFreq1 = parseFloat(modAFreq1Input.value) || 30;
    const aFreq2 = parseFloat(modAFreq2Input.value) || 250;
    // ... B and C
    
    // Update filter frequencies
    if (modAHighpass) modAHighpass.frequency.value = aFreq1;
    if (modALowpass) modALowpass.frequency.value = aFreq2;
    // ... B and C
    
    // Update Q values from globals
    if (modAHighpass) modAHighpass.Q.value = modAQ;
    if (modALowpass) modALowpass.Q.value = modAQ;
    // ... B and C
}
```

---

## 4. PIP CAMERA SYSTEM

### Chord Hotkey System (V5_6_06, V5_6_16b-16h)

```javascript
let pipPKeyHeld = false;

document.addEventListener('keydown', (e) => {
    if (e.key === 'p' || e.key === 'P') {
        pipPKeyHeld = true;
    }
    
    if (pipPKeyHeld && e.key >= '1' && e.key <= '4') {
        e.preventDefault();
        const num = parseInt(e.key);
        handlePIPChord(num);
    }
});

document.addEventListener('keyup', (e) => {
    if (e.key === 'p' || e.key === 'P') {
        pipPKeyHeld = false;
    }
});
```

### PIP Chord Mapping

| Chord | PIP | Tab Availability |
|-------|-----|------------------|
| P+1 | PIP1 slideout | PERFORM only |
| P+2 | PIP2 corner | ALL tabs |
| P+3 | PIP3 center | CREATE (render capture) |
| P+4 | Camera overlay | ALL tabs |

### Mutual Exclusivity (V5_6_16b)

Only one PIP visible at a time. Opening any PIP hides the others.

```javascript
function showPIP1() {
    hidePIP2();
    hidePIP3();
    pip1Visible = true;
    // ... show PIP1 ...
}

// P2 persists across tab switches (V5_6_16h)
function handleTabSwitch(tab) {
    // Don't hide P2 on tab switch
    if (pip1Visible) hidePIP1();
    if (pip3Visible) hidePIP3();
}
```

### PIP3 Settings Panel (V5_6_16c-16h)

Dropdown-based settings with shape-orientation logic.

```javascript
let pip3Orientation = 'square';   // landscape, portrait, square
let pip3SizePercent = '50';       // 25, 50, 75, 100
let pip3Shape = 'rectangle';      // rectangle, circle

function setPip3Orientation(value) {
    pip3Orientation = value;
    // Circle requires square
    if (value !== 'square' && pip3Shape === 'circle') {
        pip3Shape = 'rectangle';
    }
    updatePip3Display();
}

function setPip3Shape(value) {
    pip3Shape = value;
    // Circle auto-snaps to square
    if (value === 'circle') {
        pip3Orientation = 'square';
    }
    updatePip3Display();
}

function getPipDimensions() {
    const canvas = document.getElementById('canvas');
    const base = Math.min(canvas.width, canvas.height);
    const size = base * (parseInt(pip3SizePercent) / 100);
    
    switch (pip3Orientation) {
        case 'landscape': return { w: size * 16/9, h: size };
        case 'portrait': return { w: size * 9/16, h: size };
        case 'square': return { w: size, h: size };
    }
}
```

---

## 5. AUDIO FOCUS MODE

### Bare A Key Entry (V5_6_07a-08d)

```javascript
function handleAudioFocusKey() {
    if (activeFocusMode === 'audio') {
        exitAudioFocus();
    } else {
        enterAudioFocus();
    }
}

function enterAudioFocus() {
    exitAllFocusModes();
    activeFocusMode = 'audio';
    showMessage('Audio Focus: V/M/C/E/F panels, JKL transport, Esc exit', 'info');
}
```

### Panel Navigation

| Key | Panel |
|-----|-------|
| V | Volume (audio knob) |
| M | Mic knob |
| C | Cue points |
| E | EQ (H/M/L) |
| F | FX (W/D/B) |

### Two-Phase Message Pattern

```javascript
function openAudioPanel(panel) {
    activeAudioPanel = panel;
    audioFocusSelectedRow = 0;
    
    // Phase 1: Entry message (cyan, with shortcuts)
    const messages = {
        volume: 'Volume (←→ adjust, Enter=kill, V/M switch, Esc exit)',
        mic: 'Mic (←→ adjust, Enter=mute, V/M switch, Esc exit)',
        cue: 'Cue: 1-5 select, Enter jump, ←→ nudge',
        eq: 'EQ: ↑↓ H/M/L, ←→ adjust, Enter=kill',
        fx: 'FX: ↑↓ W/D/B, ←→ adjust, Enter=kill'
    };
    showMessage(messages[panel], 'info');
}

function adjustPanelValue(delta) {
    // ... adjust value ...
    
    // Phase 2: Value message (orange, value only)
    showMessage(`Audio: ${formatDB(newValue)}`, 'state');
}
```

### Waveform Click Entry (V5_6_16)

```javascript
waveformCanvas.addEventListener('click', () => {
    if (activeFocusMode !== 'audio') {
        enterAudioFocus();
    }
});
```

---

## 6. JKL TRANSPORT SYSTEM

### Video Editor Controls (V5_6_16)

```javascript
const JKL_DOUBLE_TAP_MS = 300;
let lastJTapTime = 0;
let lastLTapTime = 0;

function handleJKLTransport(key) {
    if (activeFocusMode !== 'audio') return;
    
    const now = Date.now();
    
    switch (key) {
        case 'j':
            if (now - lastJTapTime < JKL_DOUBLE_TAP_MS) {
                jumpAudioByBeats(-4);  // JJ = back 1 bar
            } else {
                jumpAudioByBeats(-1);  // J = back 1 beat
            }
            lastJTapTime = now;
            break;
            
        case 'k':
            toggleAudioPlayback();
            break;
            
        case 'l':
            if (now - lastLTapTime < JKL_DOUBLE_TAP_MS) {
                jumpAudioByBeats(4);   // LL = forward 1 bar
            } else {
                jumpAudioByBeats(1);   // L = forward 1 beat
            }
            lastLTapTime = now;
            break;
    }
}

function jumpAudioByBeats(beats) {
    if (!audioElement) return;
    const bpm = parseFloat(audioTempoField.value) || 120;
    const secondsPerBeat = 60 / bpm;
    audioElement.currentTime += beats * secondsPerBeat;
    showMessage(`Jump: ${beats > 0 ? '+' : ''}${beats} beat${Math.abs(beats) > 1 ? 's' : ''}`, 'state');
}
```

### Loop Toggle (Cmd+L)

```javascript
function handleCmdL(e) {
    if (e.metaKey && e.key === 'l') {
        e.preventDefault();
        toggleAudioLoop();
        showMessage(`Loop: ${audioLoopEnabled ? 'ON' : 'OFF'}`, 'state');
    }
}
```

---

## 7. RENDER PIPELINE

### Format Selection (V5_6_14)

```javascript
let renderFormat = 'webm';  // 'webm' or 'mp4'

function startRender() {
    let mimeType;
    
    if (renderFormat === 'mp4') {
        // H.264 + AAC
        mimeType = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';
        
        // Fallback chain
        if (!MediaRecorder.isTypeSupported(mimeType)) {
            mimeType = 'video/mp4; codecs="avc1.42E01E"';  // H.264 only
        }
        if (!MediaRecorder.isTypeSupported(mimeType)) {
            mimeType = 'video/webm; codecs="vp8, opus"';  // Fall back to WebM
        }
    } else {
        mimeType = 'video/webm; codecs="vp8, opus"';
    }
    
    // ...
}
```

### MP4 Black Frame Fix (V5_6_16a)

```javascript
async function startRender() {
    // Pre-render frame to ensure keyframe exists
    await renderSingleFrame();
    await new Promise(r => setTimeout(r, 50));
    
    // Now start MediaRecorder
    mediaRecorder = new MediaRecorder(stream, { mimeType });
    // ...
}
```

### Loop Duration Respect (V5_6_10)

```javascript
function getRenderDuration() {
    if (audioLoopEnabled) {
        // Loop ON: In → Out
        const inBeats = barBeatToBeats(inBeat.value);
        const outBeats = barBeatToBeats(outBeat.value);
        return (outBeats - inBeats) * 60 / tempo;
    } else {
        // Loop OFF: Start → End
        return parseFloat(audioEndField.value) - parseFloat(audioStartField.value);
    }
}
```

### Cancel Render (V5_6_10)

```javascript
let renderCancelled = false;
let activeMediaRecorder = null;

function cancelRender() {
    renderCancelled = true;
    
    if (activeMediaRecorder && activeMediaRecorder.state !== 'inactive') {
        activeMediaRecorder.stop();
    }
    
    // Cleanup
    if (audioElement) audioElement.pause();
    if (micStream) {
        micStream.getTracks().forEach(t => t.stop());
    }
    
    showMessage('Render cancelled', 'state');
}

function handleRenderClick() {
    if (activeMediaRecorder && activeMediaRecorder.state === 'recording') {
        cancelRender();
    } else {
        startRender();
    }
}
```

### BumpR Render Capture (V5_6_15)

```javascript
function compositeOverlaysToCanvas(ctx, canvas) {
    // BumpR video with separate aspect ratios
    if (bumprPlaying && bumprActiveVideoEl) {
        const video = bumprActiveVideoEl;
        const scale = bumprScaleModes[bumprSelectedSlot] / 100;
        
        // Separate width/height ratios for correct aspect
        const wRatio = canvas.width / video.videoWidth;
        const hRatio = canvas.height / video.videoHeight;
        const ratio = Math.min(wRatio, hRatio) * scale;
        
        const w = video.videoWidth * ratio;
        const h = video.videoHeight * ratio;
        const x = (canvas.width - w) / 2;
        const y = (canvas.height - h) / 2;
        
        ctx.drawImage(video, x, y, w, h);
    }
}
```

### Portrait Dimension Fix (V5_6_12)

```javascript
function getCanvasDimensions(orientation, resolution) {
    const res = RESOLUTIONS[resolution];  // e.g., { w: 1920, h: 1080 }
    
    switch (orientation) {
        case 'landscape':
            return { width: res.w, height: res.h };
        case 'portrait':
            return { width: res.h, height: res.w };  // Swap for portrait
        case 'square':
            return { width: res.h, height: res.h };
    }
}
```

---

## 8. BUMPR STICKRS SYSTEM

### Persistence Fixes (V5_6_11 series)

**Problem:** BumpR slots empty after page reload

**Fix:** Restore `bumprStickrsFolderHandle` from IndexedDB before loading slots.

```javascript
async function restoreBumprFolder() {
    const db = await openIndexedDB();
    const handle = await db.get('bumprStickrsFolderHandle');
    if (handle) {
        bumprStickrsFolderHandle = handle;
        await loadBumprSlots();
    }
}
```

**Thumbnail Generation Fix (V5_6_11a):**

Generate thumbnail BEFORE file write to avoid race condition.

```javascript
async function addBumprSlot(file, slotIndex) {
    // 1. Generate thumbnail first
    const thumbnail = await generateBumprThumbnail(file);
    
    // 2. Then write file
    const newHandle = await bumprStickrsFolderHandle.getFileHandle(file.name, { create: true });
    const writable = await newHandle.createWritable();
    await writable.write(file);
    await writable.close();
    
    // 3. Store slot data
    bumprStickrsSlots[slotIndex] = { filename: file.name, thumbnail };
}
```

**Quantize Dropdown Crash Fix (V5_6_11f):**

Move OmsDropdown init to DOMContentLoaded.

```javascript
document.addEventListener('DOMContentLoaded', () => {
    // Initialize quantize dropdown after DOM ready
    initQuantizeDropdown();
});
```

### Container Association (V5_6_11c)

```javascript
let containerBumprSlotIndex = null;  // 0-9 or null

// Save to container
function buildContainerObject() {
    return {
        // ... other fields ...
        bumprSlotIndex: containerBumprSlotIndex
    };
}

// Load from container
function loadContainer(container) {
    // ... other loading ...
    containerBumprSlotIndex = container.bumprSlotIndex ?? null;
    if (containerBumprSlotIndex !== null) {
        bumprSelectedSlot = containerBumprSlotIndex;
        updateBumprSlideout();
    }
}
```

---

## 9. SIGNAL FLOW DIAGRAMS

### Audio Analysis Chain

```
┌─────────────────────────────────────────────────────────────┐
│                    AUDIO SOURCES                             │
├─────────────────────────────────────────────────────────────┤
│  File Audio          Mic Input                               │
│  (AudioBuffer)       (getUserMedia)                          │
│       │                   │                                  │
│       └───────────────────┘                                  │
│                 │                                            │
│           GainNode                                           │
│                 │                                            │
│    ┌────────────┼────────────┐                               │
│    │            │            │                               │
│ modAHighpass modBHighpass modCHighpass                       │
│    │            │            │                               │
│ modALowpass  modBLowpass  modCLowpass                        │
│    │            │            │                               │
│ modAAnalyser modBAnalyser modCAnalyser                       │
│    │            │            │                               │
│ peakDetect   peakDetect   peakDetect                         │
│    │            │            │                               │
│ ADSREnvelope ADSREnvelope ADSREnvelope                       │
│    │            │            │                               │
│    └────────────┼────────────┘                               │
│                 │                                            │
│         Shader Uniforms                                      │
│    (shaderModAAudio, B, C)                                   │
└─────────────────────────────────────────────────────────────┘
```

### WebGPU Render Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    SHADER RENDER LOOP                        │
├─────────────────────────────────────────────────────────────┤
│  1. Calculate uniforms (time, beat, modulators, loopLength)  │
│       │                                                      │
│  2. Write uniform buffer (112 bytes)                         │
│       │                                                      │
│  3. Execute render pass (fragment shader)                    │
│       │                                                      │
│  4. Apply transforms (rotation, scale, tile, position)       │
│       │                                                      │
│  5. Composite layers (tracks 1-4 by opacity)                 │
│       │                                                      │
│  6. Apply VFX post-processing                                │
│       │                                                      │
│  7. Composite overlays (BumpR, PIP)                          │
│       │                                                      │
│  8. Output to canvas / MediaRecorder                         │
└─────────────────────────────────────────────────────────────┘
```

---

## 10. FUNCTION REFERENCE

### Audio Analysis

| Function | Purpose |
|----------|---------|
| `updateBandpassFilters()` | Sync freq knobs to filter nodes |
| `analyzeAudio()` | FFT analysis → modulator values |

### Audio Focus

| Function | Purpose |
|----------|---------|
| `enterAudioFocus()` | A key enter |
| `exitAudioFocus()` | A key / Esc exit |
| `openAudioPanel(panel)` | V/M/C/E/F panel open |
| `handleJKLTransport(key)` | JKL key handler |
| `jumpAudioByBeats(beats)` | Move playhead |

### PIP System

| Function | Purpose |
|----------|---------|
| `handlePIPChord(num)` | P+1/2/3/4 handler |
| `hidePip1()` / `hidePip2()` / `hidePip3()` | Hide specific PIP |
| `setPip3Orientation(value)` | Set orientation with shape reset |
| `setPip3Size(value)` | Set size percentage |
| `setPip3Shape(value)` | Set shape with orientation snap |
| `getPipDimensions()` | Calculate dimensions from settings |

### Render

| Function | Purpose |
|----------|---------|
| `handleRenderClick()` | Route start/cancel render |
| `startRender()` | Begin render with format selection |
| `cancelRender()` | Stop active render, cleanup |
| `getRenderDuration()` | Calculate duration from loop state |
| `compositeOverlaysToCanvas()` | Composite PIP/BumpR to render |

### Container

| Function | Purpose |
|----------|---------|
| `newContainer()` | Reset to empty container |
| `buildContainerObject()` | Build container data |
| `loadContainer()` | Load container from library |

---

## 11. STATE VARIABLES REFERENCE

### Audio Focus

```javascript
let activeFocusMode = null;      // 'audio', 'reactivity', 'transform', 'musicTiming'
let activeAudioPanel = null;     // 'volume', 'mic', 'cue', 'eq', 'fx'
let audioFocusSelectedRow = 0;   // Row within panel
```

### JKL Transport

```javascript
let lastJTapTime = 0;            // J double-tap detection
let lastLTapTime = 0;            // L double-tap detection
const JKL_DOUBLE_TAP_MS = 300;   // Double-tap threshold
```

### PIP System

```javascript
let pip1Visible = false;
let pip2Visible = false;
let pip3Visible = false;
let pipPKeyHeld = false;         // Chord detection
let pip3Orientation = 'square';  // landscape, portrait, square
let pip3SizePercent = '50';      // 25, 50, 75, 100
let pip3Shape = 'rectangle';     // rectangle, circle
```

### Render

```javascript
let renderFormat = 'webm';       // 'webm' or 'mp4'
let renderCancelled = false;     // Track cancelled renders
let activeMediaRecorder = null;  // Module-scoped for cancel
```

### BumpR

```javascript
let containerBumprSlotIndex = null;  // Container association (0-9 or null)
```

### UI Flags

```javascript
let createAudioSwapDropdownOpen = false;  // Prevent render loop override
```

---

## 12. SCHEMA SPECIFICATIONS

### .oms Container (V5_7_0)

```javascript
{
    // Metadata
    name: "Container Name",
    version: "5.7.0",
    created: "2026-01-16T10:30:00Z",
    
    // Shader
    shaderCode: "/* WGSL */",
    tempo: 128,
    
    // Modulators
    modAWave: "sine", modASubdiv: 4,
    modBWave: "square", modBSubdiv: 8,
    modCWave: "triangle", modCSubdiv: 16,
    
    // Audio Reference
    audioReference: {
        filename: "track.mp3",
        audioTempo: 124,
        audioStartTime: 17.1,
        audioEndTime: 300.0,
        inBeat: "1.1",
        outBeat: "65.1",
        bassGain: 1.0, midGain: 1.0, trebleGain: 1.0
    },
    
    // Cue Points (5 positions)
    audioCuePoints: [0, 64, 128, 192, 256],
    
    // Control States
    controlStates: { /* all UI values */ },
    
    // Mod Tuning (per-mod ADSR)
    modTuning: { /* ... */ },
    
    // Transform Animation
    transformAnimation: { /* ... */ },
    
    // Tags
    tags: ["bass", "128", "drop"],
    
    // Render Settings
    renderSettings: {
        resolution: "1080p",
        fps: 60,
        orientation: "landscape",
        filename: "output"
    },
    
    // BumpR Association (NEW V5_7_0)
    bumprSlotIndex: 2,  // 0-9 or null
    
    // Thumbnail
    thumbnail: "data:image/png;base64,..."
}
```

### .omsp Project State (V5_7_0)

```javascript
{
    // ... existing fields ...
    
    // Render Format (NEW V5_7_0)
    renderFormat: 'webm'  // 'webm' or 'mp4'
}
```

### Uniform Buffer (V5_7_0)

| Index | Name | Type | Notes |
|-------|------|------|-------|
| 0-24 | (unchanged) | f32 | See V5_6_0 |
| 25 | loopLength | f32 | NEW - was padding1 |
| 26-27 | padding | f32 | GPU alignment |

---

## 13. KEYBOARD SYSTEM

### CREATE Tab Focus Modes

| Section | Hotkey | Type |
|---------|--------|------|
| Reactivity | R | Focus mode |
| Transforms | T | Focus mode |
| Audio | A | Focus mode |
| Music Timing | M | Focus mode |
| PIP | P + 1/2/3/4 | Chord pattern |

### Audio Focus Panel Keys

| Key | Action |
|-----|--------|
| V | Volume knob |
| M | Mic knob |
| C | Cue panel |
| E | EQ panel |
| F | FX panel |
| ←/→ | Adjust value |
| Enter | Toggle mute/kill |
| Esc | Exit |

### JKL Transport (Audio Focus)

| Key | Single | Double |
|-----|--------|--------|
| J | Back 1 beat | Back 1 bar |
| K | Play/Pause | — |
| L | Forward 1 beat | Forward 1 bar |
| Cmd+L | Toggle loop | — |

### PIP Hotkeys (All Tabs)

| Chord | Action | Availability |
|-------|--------|--------------|
| P+1 | PIP1 slideout | PERFORM only |
| P+2 | PIP2 corner | ALL tabs |
| P+3 | PIP3 center | CREATE (render) |
| P+4 | Camera overlay | ALL tabs |

---

## 14. CSS ARCHITECTURE

### N-Panel Blender-Style Close (V5_6_03)

```javascript
// N opens panel, N closes panel - nothing else auto-closes it

// Removed auto-close from exitReactivityMode()
// Added close on tab switch only
```

### Swap Dropdown Fixes (V5_6_02-04)

```css
/* Solid background, high z-index */
#createAudioSwapSelect {
    z-index: 100;
    background: #1a1a1a;
}
```

```javascript
// Flag prevents render loop from showing time element
let createAudioSwapDropdownOpen = false;

// In render loop:
if (!createAudioSwapDropdownOpen) {
    timeEl.style.display = 'inline';
}
```

### Orientation Labels (V5_6_09a)

| Value | Label |
|-------|-------|
| landscape | Landscape (16:9) |
| portrait | Portrait (9:16) |
| square | Square (1:1) |

---

**END OF DEVELOPER DOCUMENTATION**

**Version:** Beta V5_7_0  
**Date:** January 16, 2026  
**License:** GPL-3.0
