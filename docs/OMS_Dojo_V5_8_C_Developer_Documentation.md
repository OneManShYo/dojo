# ONEMANSHYO App [Beta V5_8_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_8_0  
**Date:** January 22, 2026  
**License:** GPL-3.0  
**Line Count:** ~43,800  
**Document Purpose:** Technical implementation reference for developers

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [CORTEX Systems](#cortex-systems)
3. [Transform Automation System](#transform-automation-system)
4. [BumpR StickRs System](#bumpr-stickrs-system)
5. [Audio Input Routing](#audio-input-routing)
6. [Chroma Key System](#chroma-key-system)
7. [PIP Camera System](#pip-camera-system)
8. [Audio Focus Mode](#audio-focus-mode)
9. [JKL Transport System](#jkl-transport-system)
10. [PERFORM Tab Systems](#perform-tab-systems)
11. [Render Pipeline](#render-pipeline)
12. [Audio Analysis System](#audio-analysis-system)
13. [Signal Flow Diagrams](#signal-flow-diagrams)
14. [Function Reference](#function-reference)
15. [State Variables Reference](#state-variables-reference)
16. [Schema Specifications](#schema-specifications)
17. [Keyboard System](#keyboard-system)
18. [CSS Architecture](#css-architecture)

---

## 1. ARCHITECTURE OVERVIEW

### Single-File Philosophy

OMS App is a single HTML file (~43,800 lines) containing all HTML, CSS, and JavaScript. This enables:
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
- OMS Design System timing constants (V5_8_0)

**Tier 2: NODES** - Trace dependencies before modifying
- Track state arrays
- Library cache system
- Audio routing nodes
- BumpR slot arrays
- Automation data structures (V5_8_0)

**Tier 3: LEAFS** - Safe to modify
- UI styling
- Keyboard shortcuts
- Colors, fonts
- Help text

---

## 2. CORTEX SYSTEMS

### OMS Design System Timing Constants (V5_8_0)

New CORTEX-tier constants for consistent UI interaction timing:

```javascript
// CORTEX - centralized timing values for UI interactions
const OMS_HOLD_TO_EDIT_MS = 500;    // Hold duration to trigger inline edit
const OMS_DOUBLE_TAP_MS = 300;      // Window for double-tap detection
const OMS_CONFIRM_TIMEOUT_MS = 500; // Timeout for confirmation actions

// Usage examples:
// - Scene label hold-to-edit (500ms)
// - Volume knob double-tap kill (300ms)
// - JKL double-tap detection (300ms)
// - Stop All confirmation (500ms)
// - ESC×2 stop confirmation (500ms)
```

### loopLength Uniform (V5_7_0)

Uniform at index 25 provides shader access to loop length for perfect infinite loops.

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

### Message Bar Two-Phase Pattern

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

## 3. TRANSFORM AUTOMATION SYSTEM

### Overview (V5_7_12-15)

The Transform Automation System enables visual programming of transform parameters over time via a graph editor overlay.

### State Variables

```javascript
let automationOverlayVisible = false;     // G key toggle
let automationDomain = null;              // null or 'transform'
let selectedAutomationControl = null;     // 'rotation', 'scale', 'tile', 'translateX', 'translateY', 'pivotX', 'pivotY'
let automationHoverNodeIndex = -1;        // Node hover state
let automationHoverSegmentIndex = -1;     // Segment hover state
let draggingNodeIndex = -1;               // Currently dragging
let cueFlagsVisible = false;              // Tied to N panel (V5_7_15)

// Automation data structure (per control)
let automationData = {
    rotation: [],    // Array of {beat, value, curveOut}
    scale: [],
    tile: [],
    translateX: [],
    translateY: [],
    pivotX: [],
    pivotY: []
};
```

### Node Structure

```javascript
{
    beat: 4.0,         // X position (beat number)
    value: 0.5,        // Y position (0-1 normalized)
    curveOut: 'linear' // Curve type to next node
}
```

### Curve Types

| Type | Description | Implementation |
|------|-------------|----------------|
| linear | Straight line | `return t;` |
| sine | S-curve ease | `return (1 - Math.cos(t * Math.PI)) / 2;` |
| easeIn | Slow start | `return t * t;` |
| easeOut | Slow end | `return 1 - (1 - t) * (1 - t);` |
| hold | Step function | `return 0;` |

### Control Colors

| Control | Key | Color |
|---------|-----|-------|
| Rotation | 1 | Red (#ff6b6b) |
| Scale | 2 | Teal (#4ecdc4) |
| Tile | 3 | Yellow (#ffe66d) |
| Position | 4 | Mint (#95e1d3) |
| Pivot | 5 | Plum (#dda0dd) |

### Key Functions

```javascript
// Toggle automation overlay (G key)
function toggleAutomationOverlay() {
    automationOverlayVisible = !automationOverlayVisible;
    if (automationOverlayVisible) {
        automationDomain = 'transform';
    } else {
        automationDomain = null;
        selectedAutomationControl = null;
    }
    updateAutomationOverlay();
}

// Select control (1-5 keys in transform domain)
function selectAutomationControl(control) {
    selectedAutomationControl = control;
    drawAutomationLines();
}

// Convert beat to pixel X (CORTEX-level function)
function beatToPixelX(beat, loopStart, loopEnd, canvasWidth) {
    const loopBeats = loopEnd - loopStart;
    const beatOffset = beat - loopStart;
    return (beatOffset / loopBeats) * canvasWidth;
}

// Get interpolated value at beat position
function getAutomationValue(dataKey, beat) {
    const nodes = automationData[dataKey];
    if (!nodes || nodes.length === 0) return 0.5; // Default neutral
    
    // Find surrounding nodes
    let prevNode = null;
    let nextNode = null;
    for (const node of nodes) {
        if (node.beat <= beat) prevNode = node;
        if (node.beat > beat && !nextNode) nextNode = node;
    }
    
    // Edge cases
    if (!prevNode) return nodes[0].value;
    if (!nextNode) return prevNode.value;
    
    // Interpolate with curve
    const t = (beat - prevNode.beat) / (nextNode.beat - prevNode.beat);
    const curved = applyCurveFunction(t, prevNode.curveOut);
    return prevNode.value + (nextNode.value - prevNode.value) * curved;
}

// Apply curve function
function applyCurveFunction(t, curveType) {
    switch (curveType) {
        case 'sine': return (1 - Math.cos(t * Math.PI)) / 2;
        case 'easeIn': return t * t;
        case 'easeOut': return 1 - (1 - t) * (1 - t);
        case 'hold': return 0;
        default: return t; // linear
    }
}
```

### Mouse Interaction

| Action | Result |
|--------|--------|
| Click canvas | Create new node |
| Click node | Delete node |
| Drag node | Move freely |
| Shift+drag | Snap to beat/0.1 |
| Click segment | Add node on segment |
| Drag segment | Move both endpoints |
| Right-click segment | Curve type menu |

### Save/Load to .oms

```javascript
// Save to container
function buildContainerObject() {
    return {
        // ... other fields ...
        automationData: {
            rotation: automationData.rotation.slice(),
            scale: automationData.scale.slice(),
            tile: automationData.tile.slice(),
            translateX: automationData.translateX.slice(),
            translateY: automationData.translateY.slice(),
            pivotX: automationData.pivotX.slice(),
            pivotY: automationData.pivotY.slice()
        }
    };
}

// Load from container
function applyControlState(state) {
    if (state.automationData) {
        automationData = {
            rotation: (state.automationData.rotation || []).slice(),
            scale: (state.automationData.scale || []).slice(),
            tile: (state.automationData.tile || []).slice(),
            translateX: (state.automationData.translateX || []).slice(),
            translateY: (state.automationData.translateY || []).slice(),
            pivotX: (state.automationData.pivotX || []).slice(),
            pivotY: (state.automationData.pivotY || []).slice()
        };
    }
}
```

---

## 4. BUMPR STICKRS SYSTEM

### Overview (V5_7_02-04, 06)

BumpR StickRs provides video overlay triggering for live performance with nested B+N panel interface.

### State Variables

```javascript
let bumprModeActive = false;          // B key mode
let bumprSelectedSlot = 0;            // Currently selected (0-9)
let bumprSettingsPanelOpen = false;   // N panel visibility
let bumprPlaying = false;             // Video currently playing
let bumprQueuedTimeout = null;        // Pending quantized trigger
let bumprActiveVideoEl = null;        // Current video element
let bumprLastKTapTime = 0;            // K double-tap detection
let bumprDragSourceIndex = -1;        // Drag-drop source

// Per-slot data (10 slots)
let bumprStickrsSlots = [null, null, null, null, null, null, null, null, null, null];
let bumprTimingModes = ['instant', 'instant', 'instant', 'instant', 'instant', 'instant', 'instant', 'instant', 'instant', 'instant'];
let bumprScaleModes = [100, 100, 100, 100, 100, 100, 100, 100, 100, 100];
let bumprAnimateModes = ['none', 'none', 'none', 'none', 'none', 'none', 'none', 'none', 'none', 'none'];
let bumprDurationModes = ['original', 'original', 'original', 'original', 'original', 'original', 'original', 'original', 'original', 'original'];
```

### Z-Index Hierarchy

| Element | Z-Index | Purpose |
|---------|---------|---------|
| B slideout panel | 260 | Always on top |
| N settings panel | 259 | Below B panel |
| BumpR video (playing) | 250 | Above PIPs |
| PIP2/PIP3 | 200 | Camera overlays |
| BumpR video (stopped) | 100 | Below PIPs |

### B+N Nested Panel Pattern (V5_7_02)

```javascript
// B key enters mode and shows B panel
function enterBumprMode() {
    bumprModeActive = true;
    showBumprSlideout();
    updateBumprSlideout();
    showMessage('BumpR: 1-0 trigger, ↑↓ select, N settings, B exit', 'info');
}

// N key toggles settings panel (left of B panel)
function toggleBumprSettingsPanel() {
    bumprSettingsPanelOpen = !bumprSettingsPanelOpen;
    if (bumprSettingsPanelOpen) {
        showBumprSettingsPanel();
        updateBumprSettingsPanel(bumprSelectedSlot);
    } else {
        hideBumprSettingsPanel();
    }
}

// Get DOM elements per tab
function getBumprElements() {
    const isPerform = isPerformTabActive();
    return {
        video: isPerform ? bumprOverlayVideoLive : bumprOverlayVideo,
        panel: isPerform ? bumprSlideoutLive : bumprSlideout,
        settingsPanel: isPerform ? bumprSettingsPanelLive : bumprSettingsPanel
    };
}
```

### JKL Transport in BumpR Mode (V5_7_03)

```javascript
function handleBumprJKL(key) {
    if (!bumprPlaying || !bumprActiveVideoEl) return;
    
    const video = bumprActiveVideoEl;
    const now = Date.now();
    
    switch (key) {
        case 'j':
            if (now - bumprLastJTapTime < OMS_DOUBLE_TAP_MS) {
                video.currentTime -= 5;  // JJ = back 5s
            } else {
                video.currentTime -= 1;  // J = back 1s
            }
            bumprLastJTapTime = now;
            break;
            
        case 'k':
            const kDoubleTap = now - bumprLastKTapTime < OMS_DOUBLE_TAP_MS;
            if (kDoubleTap) {
                // K×2 = swap z-index (BumpR above/below PIPs)
                toggleBumprZIndex();
            } else {
                // K = play/pause
                if (video.paused) video.play();
                else video.pause();
            }
            bumprLastKTapTime = now;
            break;
            
        case 'l':
            if (now - bumprLastLTapTime < OMS_DOUBLE_TAP_MS) {
                video.currentTime += 5;  // LL = forward 5s
            } else {
                video.currentTime += 1;  // L = forward 1s
            }
            bumprLastLTapTime = now;
            break;
    }
}
```

### Drag-Drop Reordering (V5_7_04)

```javascript
function handleBumprDragStart(e, slotIndex) {
    if (!bumprStickrsSlots[slotIndex]) return;
    bumprDragSourceIndex = slotIndex;
    e.dataTransfer.effectAllowed = 'move';
}

function handleBumprDrop(e, targetIndex) {
    if (bumprDragSourceIndex < 0) return;
    swapBumprSlots(bumprDragSourceIndex, targetIndex);
    bumprDragSourceIndex = -1;
}

function swapBumprSlots(a, b) {
    // Swap all 5 arrays
    [bumprStickrsSlots[a], bumprStickrsSlots[b]] = [bumprStickrsSlots[b], bumprStickrsSlots[a]];
    [bumprTimingModes[a], bumprTimingModes[b]] = [bumprTimingModes[b], bumprTimingModes[a]];
    [bumprScaleModes[a], bumprScaleModes[b]] = [bumprScaleModes[b], bumprScaleModes[a]];
    [bumprAnimateModes[a], bumprAnimateModes[b]] = [bumprAnimateModes[b], bumprAnimateModes[a]];
    [bumprDurationModes[a], bumprDurationModes[b]] = [bumprDurationModes[b], bumprDurationModes[a]];
    updateBumprSlideout();
}
```

### A/D Sequential Firing (V5_7_06)

```javascript
function handleBumprADKeys(key) {
    if (!bumprModeActive) return;
    
    if (key === 'd') {
        // Next slot + fire (wraps 10→1)
        bumprSelectedSlot = (bumprSelectedSlot + 1) % 10;
        triggerBumprStickr(bumprSelectedSlot);
        showMessage(`BumpR: Slot ${bumprSelectedSlot + 1}`, 'state');
    } else if (key === 'a') {
        // Previous slot + fire (wraps 1→10)
        bumprSelectedSlot = (bumprSelectedSlot - 1 + 10) % 10;
        triggerBumprStickr(bumprSelectedSlot);
        showMessage(`BumpR: Slot ${bumprSelectedSlot + 1}`, 'state');
    }
}

// A key conflict fix - check bumprModeActive first
document.addEventListener('keydown', (e) => {
    if (e.key === 'a' && bumprModeActive) {
        e.stopImmediatePropagation();  // Prevent Audio Focus from capturing
        handleBumprADKeys('a');
    }
});
```

---

## 5. AUDIO INPUT ROUTING

### Overview (V5_7_10)

Audio Input Routing enables external audio (from Ableton via Loopback) to drive shader modulators.

### State Variables

```javascript
let audioCardInputDevices = [];       // Cached input device list
let audioCardInputStream = null;      // MediaStream
let audioCardInputSource = null;      // MediaStreamAudioSourceNode
let audioCardInputGainNode = null;    // GainNode
let audioCardInputAnalyser = null;    // AnalyserNode
let inputMuted = false;               // Mute state
let inputVolume = 0.75;               // Level 0-1
```

### Signal Flow

```
External Audio (Ableton/DAW)
    ↓
Loopback Virtual Device (macOS)
    ↓
getUserMedia({ audio: { deviceId } })
    ↓
MediaStreamAudioSourceNode (audioCardInputSource)
    ↓
GainNode (audioCardInputGainNode) ← inputVolume, inputMuted
    ↓
AnalyserNode (audioCardInputAnalyser)
    ↓
modA/B/C Highpass Filters → Shader Modulators
```

### Key Functions

```javascript
// Set input device
async function setAudioCardInput(deviceId) {
    // Stop existing stream
    if (audioCardInputStream) {
        audioCardInputStream.getTracks().forEach(t => t.stop());
    }
    
    if (deviceId === 'none') {
        audioCardInputSource = null;
        updateInputStatusHelper();
        return;
    }
    
    // Get stream
    const constraints = {
        audio: deviceId === 'default' 
            ? true 
            : { deviceId: { exact: deviceId } }
    };
    audioCardInputStream = await navigator.mediaDevices.getUserMedia(constraints);
    
    // Create nodes
    audioCardInputSource = audioContext.createMediaStreamSource(audioCardInputStream);
    audioCardInputGainNode = audioContext.createGain();
    audioCardInputAnalyser = audioContext.createAnalyser();
    
    // Connect
    audioCardInputSource.connect(audioCardInputGainNode);
    audioCardInputGainNode.connect(audioCardInputAnalyser);
    
    // Route to modulator filters
    audioCardInputGainNode.connect(modAHighpass);
    audioCardInputGainNode.connect(modBHighpass);
    audioCardInputGainNode.connect(modCHighpass);
    
    updateInputGainValue();
    updateInputStatusHelper();
}

// Update gain value
function updateInputGainValue() {
    if (!audioCardInputGainNode) return;
    const gain = inputMuted ? 0 : inputVolume;
    audioCardInputGainNode.gain.setValueAtTime(gain, audioContext.currentTime);
}
```

### UI Components

**ADMIN → Audio Card → INPUT DEVICES:**
- Scan Devices button
- INPUT ROUTING dropdown (None, System Default, device list)
- Input Level meter (80×12 horizontal, green)
- Status helper: "✓ Input active → Modulators"

**CREATE Header → Input Knob:**
- Green 24×24 rotary knob
- I key to select in Audio Focus
- Drag for coarse, Shift+drag for fine
- Tap to toggle mute
- 4×24 vertical level meter

---

## 6. CHROMA KEY SYSTEM

### Overview (V5_7_06-08, 18)

Chroma Key provides WebGPU-based green screen removal for PIP3 camera.

### State Variables

```javascript
let chromaKeyEnabled = false;
let chromaKeyColor = { r: 0, g: 1, b: 0 };  // Default green
let chromaKeyTolerance = 0.3;   // 0-1
let chromaKeySoftness = 0.1;    // 0-1
let chromaKeySpill = 0.2;       // 0-1

// WebGPU resources
let chromaKeyDevice = null;
let chromaKeyPipeline = null;
let chromaKeyContext = null;
let chromaKeyCanvas = null;
```

### WebGPU Implementation

```javascript
async function initChromaKey() {
    // Separate device for chroma processing
    const adapter = await navigator.gpu.requestAdapter();
    chromaKeyDevice = await adapter.requestDevice();
    
    // Create canvas and context
    chromaKeyCanvas = document.createElement('canvas');
    chromaKeyContext = chromaKeyCanvas.getContext('webgpu');
    chromaKeyContext.configure({
        device: chromaKeyDevice,
        format: 'bgra8unorm',  // Must match main pipeline (V5_7_08 fix)
        alphaMode: 'premultiplied'
    });
    
    // Create pipeline with chroma shader
    chromaKeyPipeline = chromaKeyDevice.createRenderPipeline({
        // ... pipeline config
    });
}
```

### Render Export Fix (V5_7_18)

```javascript
// Keep RAF running during render (don't pause)
function renderFrame() {
    // Process chroma key every frame
    if (chromaKeyEnabled && pip3Visible) {
        processChromaKey();
    }
    
    // ... shader render ...
    
    // Composite overlays (simple bitmap grab from live canvas)
    if (isRendering && pip3Visible && chromaKeyEnabled) {
        const dims = getPipDimensions();
        const x = (canvas.width - dims.w) / 2;
        const y = (canvas.height - dims.h) / 2;
        ctx.drawImage(chromaKeyCanvas, x, y, dims.w, dims.h);
    }
    
    requestAnimationFrame(renderFrame);
}
```

### UI Controls (ADMIN → PIP3 Settings)

| Control | Range | Default |
|---------|-------|---------|
| Chroma Enable | on/off | off |
| Color | color picker | #00ff00 |
| Tolerance | 0-1 | 0.3 |
| Softness | 0-1 | 0.1 |
| Spill | 0-1 | 0.2 |

---

## 7. PIP CAMERA SYSTEM

### Chord Hotkey System

```javascript
let pipPKeyHeld = false;

document.addEventListener('keydown', (e) => {
    if (e.key === 'p' || e.key === 'P') {
        pipPKeyHeld = true;
    }
    
    if (pipPKeyHeld && e.key >= '1' && e.key <= '4') {
        e.preventDefault();
        handlePIPChord(parseInt(e.key));
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

### Mutual Exclusivity

Only one PIP visible at a time. P2 persists across tab switches.

```javascript
function showPIP1() {
    hidePIP2();
    hidePIP3();
    pip1Visible = true;
    // ...
}

function handleTabSwitch(tab) {
    if (pip1Visible) hidePIP1();
    if (pip3Visible) hidePIP3();
    // P2 persists
}
```

---

## 8. AUDIO FOCUS MODE

### Bare A Key Entry

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
    showMessage('Audio Focus: V/M/I/C/E/F panels, JKL transport, Esc exit', 'info');
}
```

### Panel Navigation

| Key | Panel |
|-----|-------|
| V | Volume (audio knob) |
| M | Mic knob |
| I | Input knob (V5_8_0) |
| C | Cue points |
| E | EQ (H/M/L) |
| F | FX (W/D/B) |

### Two-Phase Message Pattern

```javascript
function openAudioPanel(panel) {
    activeAudioPanel = panel;
    
    // Phase 1: Entry message (cyan, with shortcuts)
    const messages = {
        volume: 'Volume (←→ adjust, Enter=kill, Esc exit)',
        mic: 'Mic (←→ adjust, Enter=mute, Esc exit)',
        input: 'Input (←→ adjust, Enter=mute, Esc exit)',
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

---

## 9. JKL TRANSPORT SYSTEM

### Video Editor Controls

```javascript
function handleJKLTransport(key) {
    if (activeFocusMode !== 'audio') return;
    
    const now = Date.now();
    
    switch (key) {
        case 'j':
            if (now - lastJTapTime < OMS_DOUBLE_TAP_MS) {
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
            if (now - lastLTapTime < OMS_DOUBLE_TAP_MS) {
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
}
```

---

## 10. PERFORM TAB SYSTEMS

### Unified Rotary Control System (V5_7_21)

```javascript
// Consistent interaction pattern for all rotary controls
const rotaryBehavior = {
    hover: 'Show help in message bar',
    drag: 'Coarse adjustment (0.01/px)',
    shiftDrag: 'Fine adjustment (0.002/px)',
    click: 'Enter keyboard mode (cyan ring)',
    doubleClick: 'Toggle kill (magenta fill)',
    leftRight: '±5%',
    upDown: '±1%',
    enterDouble: 'Toggle kill'
};
```

### Track/MAIN Selection (V5_7_19-21)

```javascript
let selectedTrack = -1;  // -1 = none, 0-3 = track, 4 = MAIN

function handleTrackSelection(key) {
    switch (key) {
        case '1': case '2': case '3': case '4':
            selectedTrack = parseInt(key) - 1;
            break;
        case '5':
            selectedTrack = 4;  // MAIN
            break;
        case '0':
            selectedTrack = -1;  // Deselect
            break;
    }
    updateSelectionHighlight();
}
```

### Arrow Key Control (V5_7_21)

```javascript
function handlePerformArrowKeys(e) {
    if (selectedTrack < 0) return;
    
    e.stopPropagation();  // Prevent library navigation
    
    const isVisual = e.metaKey || e.ctrlKey;  // Cmd+ for visual
    const delta = (e.key === 'ArrowLeft' || e.key === 'ArrowRight') ? 5 : 1;
    const sign = (e.key === 'ArrowRight' || e.key === 'ArrowUp') ? 1 : -1;
    
    if (selectedTrack < 4) {
        // Track 1-4
        if (isVisual) {
            adjustTrackOpacity(selectedTrack, delta * sign);
        } else {
            adjustTrackVolume(selectedTrack, delta * sign);
        }
    } else {
        // MAIN
        if (isVisual) {
            adjustMainOpacity(delta * sign);
        } else {
            adjustMainVolume(delta * sign);
        }
    }
}
```

### Stop All Button (V5_7_19)

```javascript
let stopAllPrimed = false;
let stopAllPrimedTime = 0;

function handleStopAllClick() {
    const now = Date.now();
    
    if (stopAllPrimed && now - stopAllPrimedTime < OMS_CONFIRM_TIMEOUT_MS) {
        // Second click within timeout - execute
        executeStopAll();
        stopAllPrimed = false;
    } else {
        // First click - prime
        stopAllPrimed = true;
        stopAllPrimedTime = now;
        flashStopButton();
        showMessage('Press again to stop all', 'state');
    }
}

// ESC×1 primes button visually, ESC×2 executes
function handleEscapeKey() {
    if (stopAllPrimed) {
        executeStopAll();
        stopAllPrimed = false;
    } else {
        stopAllPrimed = true;
        stopAllPrimedTime = Date.now();
        flashStopButton();
    }
}
```

### Waveform Grid Alignment Fix (V5_7_20)

```javascript
// CRITICAL: Dynamic playheadX calculation
function getPlayheadX(container) {
    return container.clientWidth / 2;  // Always center (was hardcoded 387.5px)
}

// CRITICAL: Correct BPM fallback
function getTrackBPM(trackIndex) {
    const cell = getCurrentCell(trackIndex);
    if (cell && cell.containerData) {
        return cell.containerData.audioTempo || 120;  // Was incorrectly 128
    }
    return 120;
}
```

---

## 11. RENDER PIPELINE

### Format Selection

```javascript
let renderFormat = 'webm';  // 'webm' or 'mp4'

function startRender() {
    let mimeType;
    
    if (renderFormat === 'mp4') {
        mimeType = 'video/mp4; codecs="avc1.42E01E, mp4a.40.2"';
        // Fallback chain: H.264+AAC → H.264 only → WebM
    } else {
        mimeType = 'video/webm; codecs="vp8, opus"';
    }
    
    // Pre-render frame for MP4 keyframe
    await renderSingleFrame();
    await new Promise(r => setTimeout(r, 50));
    
    mediaRecorder = new MediaRecorder(stream, { mimeType });
}
```

### Z-Index Render Respect (V5_7_06)

```javascript
function compositeOverlaysToCanvas(ctx, canvas) {
    // Respect K toggle state for z-index
    const bumprAbovePIP = bumprZIndexHigh;
    
    if (bumprAbovePIP) {
        // PIP first, then BumpR
        if (pip3Visible) compositePIP3(ctx, canvas);
        if (bumprPlaying) compositeBumpR(ctx, canvas);
    } else {
        // BumpR first, then PIP
        if (bumprPlaying) compositeBumpR(ctx, canvas);
        if (pip3Visible) compositePIP3(ctx, canvas);
    }
}
```

---

## 12. AUDIO ANALYSIS SYSTEM

### Frequency Filter Sync

```javascript
function updateBandpassFilters() {
    const aFreq1 = parseFloat(modAFreq1Input.value) || 30;
    const aFreq2 = parseFloat(modAFreq2Input.value) || 250;
    
    if (modAHighpass) modAHighpass.frequency.value = aFreq1;
    if (modALowpass) modALowpass.frequency.value = aFreq2;
    if (modAHighpass) modAHighpass.Q.value = modAQ;
    // ... B and C
}
```

**Frequency Range:** 1Hz - 20kHz

---

## 13. SIGNAL FLOW DIAGRAMS

### Audio Analysis Chain

```
┌─────────────────────────────────────────────────────────────┐
│                    AUDIO SOURCES                             │
├─────────────────────────────────────────────────────────────┤
│  File Audio       Mic Input        External Input            │
│  (AudioBuffer)    (getUserMedia)   (Loopback) [V5_8_0]       │
│       │                │                │                    │
│       └────────────────┴────────────────┘                    │
│                        │                                     │
│                  GainNode                                    │
│                        │                                     │
│           ┌────────────┼────────────┐                        │
│           │            │            │                        │
│       modAHighpass modBHighpass modCHighpass                 │
│           │            │            │                        │
│       modALowpass  modBLowpass  modCLowpass                  │
│           │            │            │                        │
│       modAAnalyser modBAnalyser modCAnalyser                 │
│           │            │            │                        │
│       peakDetect   peakDetect   peakDetect                   │
│           │            │            │                        │
│       ADSREnvelope ADSREnvelope ADSREnvelope                 │
│           │            │            │                        │
│           └────────────┼────────────┘                        │
│                        │                                     │
│                Shader Uniforms                               │
│           (shaderModAAudio, B, C)                            │
└─────────────────────────────────────────────────────────────┘
```

### WebGPU Render Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│                    SHADER RENDER LOOP                        │
├─────────────────────────────────────────────────────────────┤
│  1. Get automation values (rotation, scale, tile, pos, pivot)│
│       │                                                      │
│  2. Calculate uniforms (time, beat, modulators, loopLength)  │
│       │                                                      │
│  3. Write uniform buffer (112 bytes)                         │
│       │                                                      │
│  4. Execute render pass (fragment shader)                    │
│       │                                                      │
│  5. Apply transforms                                         │
│       │                                                      │
│  6. Composite layers (tracks 1-4 by opacity)                 │
│       │                                                      │
│  7. Apply VFX post-processing                                │
│       │                                                      │
│  8. Process chroma key (if PIP3 + chroma enabled)            │
│       │                                                      │
│  9. Composite overlays (BumpR, PIP) by z-index               │
│       │                                                      │
│  10. Output to canvas / MediaRecorder                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 14. FUNCTION REFERENCE

### Automation Functions

| Function | Purpose |
|----------|---------|
| `toggleAutomationOverlay()` | G key handler |
| `selectAutomationControl(control)` | 1-5 key handler |
| `drawAutomationLines()` | Render curves + nodes |
| `beatToPixelX()` | Beat → X position |
| `getAutomationValue()` | Interpolated playback value |
| `applyCurveFunction()` | Curve type math |

### BumpR Functions

| Function | Purpose |
|----------|---------|
| `enterBumprMode()` | B key enter |
| `exitBumprMode()` | B key exit |
| `toggleBumprSettingsPanel()` | N key toggle |
| `getBumprElements()` | DOM elements per tab |
| `triggerBumprStickr()` | Fire slot |
| `swapBumprSlots()` | Drag-drop swap |
| `handleBumprJKL()` | JKL transport |
| `handleBumprADKeys()` | A/D sequential |

### Audio Input Functions

| Function | Purpose |
|----------|---------|
| `setAudioCardInput()` | Set input device |
| `scanInputDevices()` | Enumerate devices |
| `updateInputGainValue()` | Apply volume/mute |

### Chroma Key Functions

| Function | Purpose |
|----------|---------|
| `initChromaKey()` | Initialize WebGPU |
| `processChromaKey()` | Execute chroma pass |

### PERFORM Functions

| Function | Purpose |
|----------|---------|
| `handleTrackSelection()` | 1-5, 0 keys |
| `handlePerformArrowKeys()` | Arrow key control |
| `executeStopAll()` | Global stop |

---

## 15. STATE VARIABLES REFERENCE

### Automation

```javascript
let automationOverlayVisible = false;
let automationDomain = null;
let selectedAutomationControl = null;
let automationData = { rotation:[], scale:[], tile:[], translateX:[], translateY:[], pivotX:[], pivotY:[] };
```

### BumpR

```javascript
let bumprModeActive = false;
let bumprSelectedSlot = 0;
let bumprSettingsPanelOpen = false;
let bumprPlaying = false;
let bumprZIndexHigh = true;
```

### Audio Input

```javascript
let audioCardInputDevices = [];
let audioCardInputStream = null;
let audioCardInputSource = null;
let inputMuted = false;
let inputVolume = 0.75;
```

### Chroma Key

```javascript
let chromaKeyEnabled = false;
let chromaKeyColor = { r: 0, g: 1, b: 0 };
let chromaKeyTolerance = 0.3;
let chromaKeySoftness = 0.1;
let chromaKeySpill = 0.2;
```

### PERFORM

```javascript
let selectedTrack = -1;
let stopAllPrimed = false;
let tempoKnobActive = false;
```

---

## 16. SCHEMA SPECIFICATIONS

### .oms Container (V5_8_0)

```javascript
{
    // Metadata
    name: "Container Name",
    version: "5.8.0",
    
    // ... existing fields from V5_7_0 ...
    
    // Automation Data (NEW V5_8_0)
    automationData: {
        rotation: [{beat, value, curveOut}, ...],
        scale: [{beat, value, curveOut}, ...],
        tile: [{beat, value, curveOut}, ...],
        translateX: [{beat, value, curveOut}, ...],
        translateY: [{beat, value, curveOut}, ...],
        pivotX: [{beat, value, curveOut}, ...],
        pivotY: [{beat, value, curveOut}, ...]
    }
}
```

### .omsp Project State (V5_8_0)

```javascript
{
    // ... existing fields ...
    
    // Audio Card (UPDATED V5_8_0)
    audioCardSettings: {
        masterOutputId: null,
        cueOutputId: 'none',      // 'none' option added
        cueMix: 0.5,
        inputDeviceId: 'none'     // NEW V5_8_0
    }
}
```

---

## 17. KEYBOARD SYSTEM

### CREATE Tab

| Key | Action |
|-----|--------|
| R | Reactivity Focus |
| T | Transform Focus |
| A | Audio Focus |
| M | Music Timing Focus |
| G | Automation Overlay (V5_8_0) |
| 1-5 | Select automation control (in G mode) |
| B | BumpR mode |
| N | BumpR settings panel (in B mode) |
| A/D | Sequential fire (in B mode) |
| P+1/2/3/4 | PIP chords |

### PERFORM Tab

| Key | Action |
|-----|--------|
| 1-4 | Select track |
| 5 | Select MAIN |
| 0 | Deselect |
| ←/→ | Volume ±5% (or opacity with Cmd) |
| ↑/↓ | Volume ±1% (or opacity with Cmd) |
| Enter×2 | Toggle kill |
| ESC | Prime stop (ESC×2 = execute) |
| B | BumpR mode |

### BumpR Mode (Both Tabs)

| Key | Action |
|-----|--------|
| 1-0 | Trigger slots 1-10 |
| ↑/↓ | Navigate slots |
| N | Toggle settings panel |
| A | Previous + fire |
| D | Next + fire |
| J/K/L | Video transport |
| K×2 | Swap z-index |
| B | Exit mode |

---

## 18. CSS ARCHITECTURE

### OMS Design System Constants

```javascript
// CORTEX timing constants
const OMS_HOLD_TO_EDIT_MS = 500;
const OMS_DOUBLE_TAP_MS = 300;
const OMS_CONFIRM_TIMEOUT_MS = 500;
```

### Z-Index Hierarchy

```css
.bumpr-slideout { z-index: 260; }
.bumpr-settings-panel { z-index: 259; }
.bumpr-video.playing { z-index: 250; }
.pip-container { z-index: 200; }
.bumpr-video.stopped { z-index: 100; }
```

---

**END OF DEVELOPER DOCUMENTATION**

**Version:** Beta V5_8_0  
**Date:** January 22, 2026  
**License:** GPL-3.0
