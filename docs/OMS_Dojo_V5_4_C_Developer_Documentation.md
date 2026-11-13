# ONEMANSHYO App [Beta V5_4_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_4_0  
**Date:** December 31, 2025  
**Purpose:** Technical implementation reference for V5_4_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Audio Card & Cue Bus System](#audio-card--cue-bus-system)
3. [MAIN Column Architecture](#main-column-architecture)
4. [DSP Output Processing](#dsp-output-processing)
5. [VFX Post-Processing Pipeline](#vfx-post-processing-pipeline)
6. [Meter System](#meter-system)
7. [Grid Cell Visual System](#grid-cell-visual-system)
8. [Scene Labels & Colors](#scene-labels--colors)
9. [CSS Architecture](#css-architecture)
10. [Schema Additions](#schema-additions)
11. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_4_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  AUDIO CARD & CUE BUS SYSTEM                                        │
│  Pioneer DJM-style Pre-Fader Listen with separate outputs           │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Device enum │ setSinkId │ PFL routing │ Mix crossfade       │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  MAIN COLUMN ARCHITECTURE                                           │
│  Dedicated output bus controls (Master → MAIN rename)               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 64 renames │ Stereo metering │ 110px column │ DSP/VFX toggles│   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  DSP & VFX SLIDEOUT PANELS                                          │
│  Master bus processing with emerge-from-behind animation            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Compressor │ 5 presets │ WebGPU VFX │ 10 effects            │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  GRID CELL VISUAL OVERHAUL                                          │
│  Half-height cells with split shader/waveform layout                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 32/9 aspect │ Split halves │ Layered states │ Subdued colors│   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Pre-Fader Listen** - Cue bus taps BEFORE volume fader
2. **Pioneer Crossfade** - Mix knob crossfades (not additive)
3. **Master → MAIN** - Clear separation from Master Clock/Grid
4. **Gap-as-Divider** - Blender-style border architecture
5. **Layered States** - Pseudo-elements for independent z-ordering

---

## AUDIO CARD & CUE BUS SYSTEM

### Device Enumeration

```javascript
// ============================================
// AUDIO DEVICE ENUMERATION
// Location: scanAudioDevices() ~line 7200
// ============================================

async function scanAudioDevices() {
    try {
        // Request permission if needed
        await navigator.mediaDevices.getUserMedia({ audio: true });
        
        const devices = await navigator.mediaDevices.enumerateDevices();
        audioCardDevices = devices.filter(d => d.kind === 'audiooutput');
        
        return audioCardDevices;
    } catch (err) {
        console.error('Device enumeration failed:', err);
        return [];
    }
}
```

### Audio Card Settings State

```javascript
// ============================================
// AUDIO CARD STATE
// Location: ~line 7100
// ============================================

let audioCardSettings = {
    masterOutputId: 'default',  // FOH/speakers device ID
    cueOutputId: 'default',     // Headphones device ID
    cueMix: 50                  // 0=full cue, 100=full main
};

let audioCardDevices = [];      // Cached device list

// Per-track cue states
// gridState.trackCues = [false, false, false, false];
```

### Cue Bus Audio Graph

```javascript
// ============================================
// CUE BUS ROUTING
// Location: initAudioRouting() ~line 8500
// ============================================

// New nodes for cue bus
let trackCueGainNodes = [];      // Per-track cue send (0 or 1)
let cueMasterGainNode = null;    // Sum of all cued tracks
let mainToCueGainNode = null;    // Main mix copy for crossfade
let cueOutputGainNode = null;    // Final cue output

function initCueBusRouting() {
    // Create cue master (sums all cued tracks)
    cueMasterGainNode = audioContext.createGain();
    cueMasterGainNode.gain.value = 1.0;
    
    // Create main-to-cue (for crossfade)
    mainToCueGainNode = audioContext.createGain();
    mainToCueGainNode.gain.value = 0.5;  // Default 50% mix
    
    // Create final cue output
    cueOutputGainNode = audioContext.createGain();
    cueOutputGainNode.gain.value = 1.0;
    
    // Connect main analyser to main-to-cue
    trackMainAnalyser.connect(mainToCueGainNode);
    
    // Connect both to cue output
    cueMasterGainNode.connect(cueOutputGainNode);
    mainToCueGainNode.connect(cueOutputGainNode);
    
    // Per-track cue sends (created during track init)
    for (let i = 0; i < 4; i++) {
        trackCueGainNodes[i] = audioContext.createGain();
        trackCueGainNodes[i].gain.value = 0;  // Off by default
        trackCueGainNodes[i].connect(cueMasterGainNode);
    }
}
```

### Signal Chain Architecture

```
Track → EQ ──┬──→ Gain → Analyser → Master Gain → speakers
             │         (POST-FADER)
             │
             └──→ Cue Gain [0/1] ─┐   (PRE-FADER tap from EQ output)
                                  ↓
             All Cue Gains → Cue Master ──┐
                                          │ (crossfade via mix knob)
                                          ↓
Master Analyser ──→ Main-to-Cue Gain ─────┴─→ Cue Output → headphones
```

### Pre-Fader Listen Implementation

```javascript
// ============================================
// PFL ROUTING (PRE-FADER)
// Location: Track initialization ~line 16000
// ============================================

function connectTrackForCue(track, eqOutputNode) {
    // Cue tap is from EQ output (BEFORE volume fader)
    // This means track at 0% volume is still audible in cue
    
    eqOutputNode.connect(trackCueGainNodes[track]);
    // trackCueGainNodes[track] → cueMasterGainNode (set up in init)
}

// Toggle cue for track
function updateTrackCueRouting(track, isCued) {
    gridState.trackCues[track] = isCued;
    
    // Cue gain is binary: 0 (off) or 1 (on)
    trackCueGainNodes[track].gain.value = isCued ? 1 : 0;
}
```

### Pioneer-Style Crossfade

```javascript
// ============================================
// MIX KNOB CROSSFADE
// Location: updateCueMixGains() ~line 7400
// ============================================

function updateCueMixGains() {
    // cueMix: 0=full cue, 50=equal, 100=full main
    const mix = audioCardSettings.cueMix / 100;
    
    // Pioneer-style crossfade (not additive)
    // At mix=0: cue=1.0, main=0.0
    // At mix=50: cue=0.5, main=0.5
    // At mix=100: cue=0.0, main=1.0
    
    cueMasterGainNode.gain.value = 1 - mix;    // Cue amount
    mainToCueGainNode.gain.value = mix;        // Main amount
}
```

### Device Output Routing

```javascript
// ============================================
// DEVICE ROUTING (setSinkId)
// Location: applyMasterOutputDevice() ~line 7500
// ============================================

async function applyMasterOutputDevice() {
    // Master output uses default audio destination
    // (setSinkId not needed for main output)
}

async function applyCueOutputDevice() {
    if (!audioCardSettings.cueOutputId || 
        audioCardSettings.cueOutputId === 'default') {
        return;
    }
    
    // Create MediaStream from cue output
    cueStreamDestination = audioContext.createMediaStreamDestination();
    cueOutputGainNode.disconnect();
    cueOutputGainNode.connect(cueStreamDestination);
    
    // Create audio element and route to device
    cueAudioElement = new Audio();
    cueAudioElement.srcObject = cueStreamDestination.stream;
    
    // setSinkId routes to specific device
    await cueAudioElement.setSinkId(audioCardSettings.cueOutputId);
    cueAudioElement.play();
}
```

### S→C Button Logic

```javascript
// ============================================
// S→C BUTTON TRANSFORMATION
// Location: ~line 15800
// ============================================

function isCueModeEnabled() {
    return audioCardSettings.cueOutputId && 
           audioCardSettings.cueOutputId !== 'default';
}

function updateTrackSoloCueButton(track) {
    const btn = document.getElementById(`trackAudioSoloBtn${track + 1}`);
    
    if (isCueModeEnabled()) {
        btn.textContent = 'C';
        btn.style.background = gridState.trackCues[track] 
            ? '#ffa500'   // Orange when active
            : '#333';
        btn.onclick = () => toggleTrackCue(track);
    } else {
        btn.textContent = 'S';
        btn.style.background = trackAudioSolo[track]
            ? '#00d4ff'   // Cyan when active
            : '#333';
        btn.onclick = () => toggleTrackSolo(track);
    }
}
```

---

## MAIN COLUMN ARCHITECTURE

### Master → MAIN Rename

```javascript
// ============================================
// VARIABLE RENAMES (64 occurrences)
// ============================================

// KEEP these (timing/overlay systems):
masterTempo              // Master clock BPM
masterBarNow()           // Current bar position
masterBeatNow()          // Current beat position
masterGridVisible        // Beat grid overlay toggle
trackMasterClockStart    // Session start timestamp

// RENAME these (output bus):
// Old                   → New
masterShaderOpacity      → trackMainShaderOpacity
masterShaderMute         → trackMainShaderMute
masterVolume             → trackMainVolume
masterAudioMute          → trackMainAudioMute
masterGainNode           → trackMainGainNode
masterAnalyser           → trackMainAnalyser
mixerMaster              → mixerMain
```

### Column Layout Structure

```html
<!-- MAIN Column (110px) -->
<div id="mainColumnContainer" class="main-column-container">
    
    <!-- Top Module (flex: 1, aligns with grid) -->
    <div class="main-top-module">
        <div class="main-header">MAIN</div>
        
        <!-- Two 52px sections side by side -->
        <div class="main-sections-row">
            <!-- Visual Section -->
            <div class="main-column-section">
                <canvas id="trackMainOpacityMeter"></canvas>
                <div class="main-meter-readout" id="trackMainOpacityReadout">100</div>
                <button class="main-btn-wide">M</button>
                <canvas class="main-knob" id="trackMainOpacityKnob"></canvas>
            </div>
            
            <!-- Audio Section -->
            <div class="main-column-section">
                <canvas id="trackMainMeterL"></canvas>
                <canvas id="trackMainMeterR"></canvas>
                <div class="main-meter-readout" id="trackMainAudioReadout">-∞</div>
                <button class="main-btn-wide">M</button>
                <canvas class="main-knob" id="trackMainVolumeKnob"></canvas>
                <canvas class="main-knob" id="trackMainMixKnob"></canvas>
            </div>
        </div>
        
        <!-- Control boxes (5 placeholders) -->
        <div class="main-control-box"></div>
        <div class="main-control-box"></div>
        <div class="main-control-box"></div>
        <div class="main-control-box"></div>
        <div class="main-control-box"></div>
    </div>
    
    <!-- Bottom Module (248px fixed, aligns with waveforms) -->
    <div class="main-bottom-module">
        <div class="main-clock-display">1.1.1</div>
        <select id="quantizeDropdown">...</select>
        <select id="zoomDropdown">...</select>
        <div class="main-tempo-row">
            <span id="tempoDisplay">120</span>
            <canvas id="tempoKnob"></canvas>
        </div>
        <div class="main-mtw-buttons">
            <button id="masterGridBtn">M</button>
            <button id="trackGridBtn">T</button>
            <button id="waveformBtn">W</button>
        </div>
    </div>
</div>
```

### Stereo Metering Implementation

```javascript
// ============================================
// STEREO METERING
// Location: initTrackMainStereoMeters() ~line 8900
// ============================================

let trackMainSplitter = null;
let trackMainAnalyserL = null;
let trackMainAnalyserR = null;

function initTrackMainStereoMeters() {
    // Create channel splitter
    trackMainSplitter = audioContext.createChannelSplitter(2);
    
    // Create per-channel analysers
    trackMainAnalyserL = audioContext.createAnalyser();
    trackMainAnalyserL.fftSize = 256;
    
    trackMainAnalyserR = audioContext.createAnalyser();
    trackMainAnalyserR.fftSize = 256;
    
    // Connect splitter outputs to analysers
    trackMainSplitter.connect(trackMainAnalyserL, 0);  // Left channel
    trackMainSplitter.connect(trackMainAnalyserR, 1);  // Right channel
    
    // Insert splitter after main gain
    trackMainGainNode.connect(trackMainSplitter);
}

function updateMainStereoMeters() {
    const peakL = getPeakLevel(trackMainAnalyserL);
    const peakR = getPeakLevel(trackMainAnalyserR);
    
    drawMeterBar(trackMainMeterLCanvas, peakL);
    drawMeterBar(trackMainMeterRCanvas, peakR);
    
    // Update readout with max of L/R
    const peakDb = peakToDb(Math.max(peakL, peakR));
    trackMainAudioReadout.textContent = peakDb.toFixed(1);
}
```

---

## DSP OUTPUT PROCESSING

### State Variables

```javascript
// ============================================
// DSP STATE
// Location: ~line 7800
// ============================================

let dspPanelOpen = false;
let dspEnabled = true;
let dspCompressor = null;       // DynamicsCompressorNode
let dspOutputGain = null;       // Makeup gain
let dspThreshold = -18;         // -48 to 0 dB
let dspRatio = 4;               // 1 to 20
let dspKnee = 20;               // Fixed
let dspAttack = 0.003;          // Fixed 3ms
let dspRelease = 0.25;          // Fixed 250ms
let dspOutputLevel = 0;         // -12 to +12 dB
let dspCurrentPreset = 'medium';

const DSP_PRESETS = {
    off:        { threshold: 0,   ratio: 1,  output: 0 },
    gentle:     { threshold: -24, ratio: 2,  output: 2 },
    medium:     { threshold: -18, ratio: 4,  output: 4 },
    aggressive: { threshold: -12, ratio: 8,  output: 6 },
    limiter:    { threshold: -6,  ratio: 20, output: 3 }
};
```

### Audio Chain

```javascript
// ============================================
// DSP AUDIO CHAIN
// Location: initDSPProcessing() ~line 7900
// ============================================

function initDSPProcessing() {
    // Create compressor
    dspCompressor = audioContext.createDynamicsCompressor();
    dspCompressor.threshold.value = dspThreshold;
    dspCompressor.ratio.value = dspRatio;
    dspCompressor.knee.value = dspKnee;
    dspCompressor.attack.value = dspAttack;
    dspCompressor.release.value = dspRelease;
    
    // Create makeup gain
    dspOutputGain = audioContext.createGain();
    dspOutputGain.gain.value = Math.pow(10, dspOutputLevel / 20);
    
    // Insert into chain: Master Gain → DSP → Destination
    trackMainGainNode.disconnect();
    trackMainGainNode.connect(dspCompressor);
    dspCompressor.connect(dspOutputGain);
    dspOutputGain.connect(audioContext.destination);
}
```

### Gain Reduction Meter

```javascript
// ============================================
// GR METER
// Location: startDSPReductionMeter() ~line 8000
// ============================================

function startDSPReductionMeter() {
    function updateGR() {
        if (!dspCompressor) return;
        
        // reduction is negative value in dB
        const reduction = dspCompressor.reduction;
        const percent = Math.min(100, Math.abs(reduction) * 5);  // Scale to meter
        
        drawGRMeter(percent);
        
        if (dspPanelOpen) {
            requestAnimationFrame(updateGR);
        }
    }
    updateGR();
}
```

---

## VFX POST-PROCESSING PIPELINE

### State Variables

```javascript
// ============================================
// VFX STATE
// Location: ~line 8100
// ============================================

let vfxDevice = null;           // WebGPU device
let vfxContext = null;          // Canvas context
let vfxPipeline = null;         // Current render pipeline
let vfxSampler = null;          // Texture sampler
let vfxUniformBuffer = null;    // Effect parameters
let vfxInitialized = false;
let vfxCurrentEffect = 'off';
let vfxAmount = 0;              // 0-100%
let vfxPanelOpen = false;

const VFX_EFFECTS = [
    'off', 'rgbsplit', 'glitch', 'pixelate', 'scanlines',
    'feedback', 'mirror', 'kaleidoscope', 'vhs', 'shake'
];
```

### Pipeline Architecture

```javascript
// ============================================
// VFX PIPELINE
// Location: initVFXPipeline() ~line 8200
// ============================================

async function initVFXPipeline() {
    // Get WebGPU device (reuse from shader system)
    vfxDevice = device;  // Already initialized
    
    // Create output canvas context
    const vfxCanvas = document.getElementById('vfxOutputCanvas');
    vfxContext = vfxCanvas.getContext('webgpu');
    vfxContext.configure({
        device: vfxDevice,
        format: 'bgra8unorm',
        alphaMode: 'premultiplied'
    });
    
    // Create sampler for input texture
    vfxSampler = vfxDevice.createSampler({
        magFilter: 'linear',
        minFilter: 'linear'
    });
    
    vfxInitialized = true;
}
```

### Effect Shader Example (RGB Split)

```javascript
// ============================================
// VFX SHADER EXAMPLE
// Location: buildVFXPipeline() ~line 8300
// ============================================

const VFX_SHADER_RGBSPLIT = `
struct Uniforms {
    amount: f32,
    time: f32,
    padding: vec2<f32>
}
@group(0) @binding(0) var<uniform> uniforms: Uniforms;
@group(0) @binding(1) var inputTexture: texture_2d<f32>;
@group(0) @binding(2) var inputSampler: sampler;

@fragment
fn fs_main(@builtin(position) pos: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = pos.xy / vec2<f32>(textureDimensions(inputTexture));
    let offset = uniforms.amount * 0.02;
    
    let r = textureSample(inputTexture, inputSampler, uv + vec2<f32>(offset, 0.0)).r;
    let g = textureSample(inputTexture, inputSampler, uv).g;
    let b = textureSample(inputTexture, inputSampler, uv - vec2<f32>(offset, 0.0)).b;
    
    return vec4<f32>(r, g, b, 1.0);
}
`;
```

### Render Pass

```javascript
// ============================================
// VFX RENDER PASS
// Location: renderVFXPass() ~line 8400
// ============================================

function renderVFXPass() {
    if (!vfxInitialized || vfxCurrentEffect === 'off') return;
    
    // 1. Composite track canvases to texture
    const compositeTexture = compositeTracksToTexture();
    
    // 2. Update uniforms
    const uniformData = new Float32Array([
        vfxAmount / 100,  // Normalize to 0-1
        performance.now() / 1000,
        0, 0  // Padding
    ]);
    vfxDevice.queue.writeBuffer(vfxUniformBuffer, 0, uniformData);
    
    // 3. Execute render pass
    const commandEncoder = vfxDevice.createCommandEncoder();
    const passEncoder = commandEncoder.beginRenderPass({
        colorAttachments: [{
            view: vfxContext.getCurrentTexture().createView(),
            clearValue: { r: 0, g: 0, b: 0, a: 1 },
            loadOp: 'clear',
            storeOp: 'store'
        }]
    });
    
    passEncoder.setPipeline(vfxPipeline);
    passEncoder.setBindGroup(0, vfxBindGroup);
    passEncoder.draw(6);  // Full-screen quad
    passEncoder.end();
    
    vfxDevice.queue.submit([commandEncoder.finish()]);
}
```

---

## METER SYSTEM

### Toggle Overlay

```javascript
// ============================================
// METER OVERLAY
// Location: toggleMeters() ~line 9100
// ============================================

let metersVisible = false;

function toggleMeters() {
    metersVisible = !metersVisible;
    
    const overlay = document.getElementById('metersOverlay');
    overlay.style.display = metersVisible ? 'flex' : 'none';
    
    // Update toggle button appearance
    const btn = document.getElementById('meterToggleCube');
    btn.classList.toggle('active', metersVisible);
    
    if (metersVisible) {
        startMeterAnimation();
    }
}
```

### Per-Track Meter Module

```html
<!-- Track Meter Column -->
<div class="meter-column" data-track="1">
    <div class="meter-module-wrap">
        <!-- Visual (left) -->
        <div class="meter-half visual">
            <canvas class="meter-bar-visual" id="opacityMeter1"></canvas>
            <div class="meter-readout" id="opacityReadout1">100</div>
        </div>
        
        <!-- Audio (right) -->
        <div class="meter-half audio">
            <canvas class="meter-bar-narrow" id="meterL1"></canvas>
            <canvas class="meter-bar-narrow" id="meterR1"></canvas>
            <div class="meter-readout" id="peakReadout1">-∞</div>
        </div>
    </div>
</div>
```

### Animation Loop

```javascript
// ============================================
// METER ANIMATION
// Location: animateMeters() ~line 9200
// ============================================

function animateMeters() {
    if (!metersVisible) return;
    
    for (let i = 0; i < 4; i++) {
        // Visual meter (opacity)
        const opacity = trackShaderOpacity[i];
        updateOpacityMeter(i, opacity);
        
        // Audio meter (peak)
        if (trackAudioPlaying[i] && trackAnalysers[i]) {
            const peak = getPeakLevel(trackAnalysers[i]);
            updateAudioMeter(i, peak);
        }
    }
    
    // MAIN meters (always update)
    updateMainStereoMeters();
    
    requestAnimationFrame(animateMeters);
}
```

---

## GRID CELL VISUAL SYSTEM

### Aspect Ratio Change

```css
/* Half-height cells */
.clip-slot {
    aspect-ratio: 32 / 9;  /* Was 16/9 - now half height */
    position: relative;
    overflow: hidden;
}
```

### Split Layout Structure

```javascript
// ============================================
// CELL UPDATE
// Location: updateCellDisplay() ~line 14000
// ============================================

function updateCellDisplay(cell, containerData) {
    cell.innerHTML = '';
    
    if (!containerData) {
        cell.innerHTML = '<span class="cell-empty-text">Empty</span>';
        return;
    }
    
    // Create split container
    const splitContainer = document.createElement('div');
    splitContainer.className = 'cell-split-container';
    
    // Visual half (left)
    const visualHalf = document.createElement('div');
    visualHalf.className = 'cell-visual-half';
    if (containerData.thumbnail) {
        const canvas = document.createElement('canvas');
        canvas.width = 80;
        canvas.height = 45;
        // Draw thumbnail...
        visualHalf.appendChild(canvas);
    } else {
        visualHalf.style.background = 'rgba(128, 0, 128, 0.3)';  // 30% purple
    }
    
    // Audio half (right)
    const audioHalf = document.createElement('div');
    audioHalf.className = 'cell-audio-half';
    if (containerData.audioBuffer) {
        const canvas = document.createElement('canvas');
        // Draw waveform bars...
        audioHalf.appendChild(canvas);
    } else {
        audioHalf.style.background = 'rgba(0, 212, 255, 0.3)';  // 30% cyan
    }
    
    splitContainer.appendChild(visualHalf);
    splitContainer.appendChild(audioHalf);
    cell.appendChild(splitContainer);
    
    // Name overlay
    const nameOverlay = document.createElement('div');
    nameOverlay.className = 'cell-name-overlay';
    nameOverlay.textContent = containerData.name;
    cell.appendChild(nameOverlay);
}
```

### Layered State Indicators

```css
/* Base pseudo-elements (always present) */
.clip-slot::before,
.clip-slot::after {
    content: '';
    position: absolute;
    inset: 0;
    pointer-events: none;
}

.clip-slot::before { z-index: 5; }   /* Playing layer */
.clip-slot::after { z-index: 10; }   /* Cursor layer (always on top) */

/* Playing state - yellow on ::before */
.clip-slot.playing::before {
    background: rgba(255, 255, 0, 0.15);
    outline: 2px solid #ffff00;
    outline-offset: -2px;
}

/* Cursor selection - cyan on ::after */
.clip-slot.wasd-selected::after {
    outline: 1px solid #00d4ff;
    outline-offset: -1px;
}

/* Hover state - subtle cyan on ::after */
.clip-slot:hover::after {
    background: rgba(0, 212, 255, 0.05);
}
```

### Ghost Cell Cleanup Fix

```javascript
// ============================================
// GHOST CELL FIX
// Location: launchScene() ~line 16500
// ============================================

function launchScene(scene) {
    for (let track = 0; track < 4; track++) {
        const cellData = gridState.cells[track][scene];
        
        // ALWAYS clear .playing from ALL cells in this track FIRST
        document.querySelectorAll(`.clip-slot[data-track="${track}"]`).forEach(cell => {
            cell.classList.remove('playing');
        });
        
        if (!cellData) {
            // Empty cell - stop track if playing
            if (trackAudioSources[track]) {
                stopTrackAudio(track);
            }
            continue;  // Don't add .playing to empty cell
        }
        
        // Launch container...
        const cell = document.querySelector(
            `.clip-slot[data-track="${track}"][data-scene="${scene}"]`
        );
        cell.classList.add('playing');
    }
}
```

---

## SCENE LABELS & COLORS

### State Variables

```javascript
// ============================================
// SCENE LABEL STATE
// Location: ~line 6900
// ============================================

let sceneLabels = ['1', '2', '3', '4', '5', '6', '7', '8'];
let sceneColors = [null, null, null, null, null, null, null, null];

const SCENE_COLOR_PRESETS = [
    { name: 'Default', hex: null },
    { name: 'Cyan', hex: '#00d4ff' },
    { name: 'Magenta', hex: '#ff006e' },
    { name: 'Orange', hex: '#ffa500' },
    { name: 'Yellow', hex: '#ffff00' },
    { name: 'Green', hex: '#00ff88' },
    { name: 'Purple', hex: '#a855f7' },
    { name: 'Red', hex: '#ff4444' }
];
```

### Ticker Animation

```css
/* Ticker scroll for long labels */
.scene-trigger-label {
    overflow: hidden;
    white-space: nowrap;
}

.scene-trigger-label.ticker {
    animation: ticker 6s linear infinite;
}

@keyframes ticker {
    0% { transform: translateX(0); }
    100% { transform: translateX(-50%); }
}
```

```javascript
// ============================================
// TICKER DETECTION
// Location: updateSceneTriggerDisplay() ~line 18000
// ============================================

function updateSceneTriggerDisplay(index) {
    const trigger = document.getElementById(`sceneTrigger${index + 1}`);
    const label = trigger.querySelector('.scene-trigger-label');
    
    const text = sceneLabels[index];
    const displayText = text === String(index + 1) ? `Scene ${text}` : text;
    
    // Check if text overflows
    label.textContent = displayText;
    
    if (label.scrollWidth > label.clientWidth) {
        // Enable ticker - duplicate text for seamless loop
        label.innerHTML = `<span>${displayText}</span><span>${displayText}</span>`;
        label.classList.add('ticker');
    } else {
        label.classList.remove('ticker');
    }
    
    // Apply color
    const color = sceneColors[index];
    trigger.style.background = color || '';
}
```

---

## CSS ARCHITECTURE

### Gap-as-Divider Pattern

```css
/* Parent provides gap, children have no borders */
.parent-container {
    display: flex;
    gap: 1px;
    background: #111;  /* Gap color */
}

.child-element {
    background: #1a1a1a;
    border: none;
}

/* State indicators use outline (doesn't affect layout) */
.child-element.active {
    outline: 1px solid #00d4ff;
    outline-offset: -1px;
}
```

### Box-Sizing Compensation

```css
/* When parent has padding, child needs negative margin */
.preview-panel {
    padding: 3px 7px;  /* Creates offset */
}

.main-column-container {
    margin-left: -3px;   /* Compensate left padding */
    margin-right: -7px;  /* Compensate right padding */
}
```

### Domain Color System

```css
:root {
    /* Visual domain */
    --color-visual: #a855f7;       /* Purple */
    --color-visual-dim: rgba(128, 0, 128, 0.3);
    
    /* Audio domain */
    --color-audio: #00d4ff;        /* Cyan */
    --color-audio-dim: rgba(0, 212, 255, 0.3);
    
    /* Cue domain */
    --color-cue: #ffa500;          /* Orange */
    
    /* States */
    --color-playing: #ffff00;      /* Yellow */
    --color-error: #ff0000;        /* Red */
}
```

---

## SCHEMA ADDITIONS

### Project State (.omsp)

```javascript
// ============================================
// .OMSP SCHEMA - V5_4_0 ADDITIONS
// ============================================

{
    // Existing fields...
    
    // V5_4_0: Audio Card Settings
    "audioCardSettings": {
        "masterOutputId": "default",
        "cueOutputId": "default",
        "cueMix": 50
    },
    
    // V5_4_0: Track Cue States
    "trackCues": [false, false, false, false],
    
    // V5_4_0: Scene Customization
    "sceneLabels": ["1", "2", "3", "4", "5", "6", "7", "8"],
    "sceneColors": [null, null, null, null, null, null, null, null],
    
    // V5_4_0: Tempo Preference
    "audioPrefs": {
        "masterTempo": 120
    },
    
    // V5_4_0: DSP Settings
    "dspSettings": {
        "enabled": true,
        "preset": "medium",
        "threshold": -18,
        "ratio": 4,
        "output": 0
    },
    
    // V5_4_0: VFX Settings
    "vfxSettings": {
        "effect": "off",
        "amount": 0
    }
}
```

---

## TESTING NOTES

### Audio Card Tests

1. **Device Enumeration:** Verify devices listed in dropdowns
2. **Master Output:** Test tone plays through selected device
3. **Cue Output:** Test tone plays through headphone device
4. **Mix Knob:** Crossfade works correctly (not additive)
5. **PFL:** Track at 0% volume still audible in cue
6. **Persistence:** Settings saved/restored with project

### MAIN Column Tests

1. **Stereo Metering:** L/R meters show independent levels
2. **Opacity Control:** Visual mute/knob work correctly
3. **Volume Control:** Audio mute/mix/volume work correctly
4. **Layout:** Aligns with grid (top) and waveforms (bottom)

### DSP Tests

1. **Panel Toggle:** Slideout animates correctly
2. **Presets:** All 5 presets apply correct values
3. **Knobs:** Threshold/Ratio/Output update in real-time
4. **GR Meter:** Shows gain reduction during compression
5. **Bypass:** "Off" preset bypasses processing

### VFX Tests

1. **Panel Toggle:** Slideout animates correctly
2. **Effect Selection:** All 10 effects render correctly
3. **Amount Knob:** 0-100% range with live preview
4. **Reset:** Double-click and "Off" reset amount to 0%
5. **Performance:** 60fps maintained with effects

### Grid Cell Tests

1. **Aspect Ratio:** Cells are half-height (2× more visible)
2. **Split Layout:** Shader left, waveform right
3. **Empty Cells:** Show "Empty" text
4. **Placeholders:** 30% purple/cyan when missing content
5. **Playing State:** Yellow tint + outline visible
6. **Selection State:** Cyan outline ON TOP of yellow
7. **Ghost Cells:** Empty cells don't get .playing

### Scene Label Tests

1. **Custom Labels:** Edit in ADMIN and PERFORM
2. **Colors:** All 8 presets apply correctly
3. **Ticker:** Long labels scroll continuously
4. **Persistence:** Saved/restored with project

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_4_0
Date: December 31, 2025
License: GPL-3.0
