# OMS SHADER BAKER V3.3.0 - DEVELOPER DOCUMENTATION

**Version:** V3.3.0  
**Date:** November 2025  
**Architecture:** Single-file HTML application  
**License:** GPL-3.0

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [File Structure](#file-structure)
3. [Core Systems](#core-systems)
4. [Beat Grid System](#beat-grid-system)
5. [Audio System](#audio-system)
6. [Shader System](#shader-system)
7. [State Management](#state-management)
8. [Key Algorithms](#key-algorithms)
9. [API Reference](#api-reference)
10. [Development Guidelines](#development-guidelines)

---

## ARCHITECTURE OVERVIEW

### Single-File Philosophy

OMS Shader Baker is a complete WebGPU shader creation and performance tool contained in ONE HTML file. This design choice enables:

- **Zero dependencies:** No build process, no npm packages
- **Instant distribution:** Single file to share
- **Offline capability:** Works without internet
- **Version control:** Complete app state in one file
- **Portability:** Runs anywhere with WebGPU support

### Technology Stack

**Core Technologies:**
- WebGPU (shader rendering)
- WGSL (shader language)
- Web Audio API (audio processing)
- File System Access API (library management)
- Canvas API (waveforms, UI elements)

**Browser Support:**
- Chrome 113+ (full support)
- Safari Technology Preview (WebGPU support required)

---

## FILE STRUCTURE

### HTML Organization

```html
<!DOCTYPE html>
<html>
<head>
    <meta> tags
    <style> CSS (embedded)
</head>
<body>
    <div class="app-wrapper">
        Header
        Tab Navigation
        Tab Content Areas
    </div>
    <script> JavaScript (embedded)
</body>
</html>
```

### Code Sections (10,219 lines total)

**Lines 1-50:** Header, license, version info  
**Lines 51-1550:** CSS styles (complete UI)  
**Lines 1551-1650:** HTML structure  
**Lines 1651-10219:** JavaScript (all functionality)

### JavaScript Organization

1. **Global State** (lines ~1700-1800)
   - `gridState` - UI and grid configuration
   - `trackContainerData` - Multi-track audio data
   - `controlStates` - Shader control values

2. **WebGPU Setup** (lines ~1800-2000)
   - Device initialization
   - Pipeline creation
   - Shader compilation

3. **Audio System** (lines ~2000-3500)
   - Web Audio API setup
   - FFT analysis
   - Multi-track playback
   - Mixer controls

4. **Beat Grid System** (lines ~3500-4500)
   - Visual timing calculations
   - Waveform rendering
   - Scroll synchronization

5. **UI Controls** (lines ~4500-6000)
   - Tab switching
   - Rotary knobs
   - Sliders
   - Button handlers

6. **File System** (lines ~6000-7500)
   - .oms file format
   - .omsp project files
   - Library management

7. **Shader System** (lines ~7500-9000)
   - Preset management
   - Control binding
   - Render loop

8. **LIVE Tab** (lines ~9000-10219)
   - Performance grid
   - Track launching
   - Shader preview (beta)

---

## CORE SYSTEMS

### 1. Beat Grid System (The One Timeline)

**Problem Solved:** Multi-BPM tracks need to align visually while playing at different speeds.

**Solution:** Single master timeline for ALL tracks.

**Core Principle:**
```javascript
// ALL tracks use SAME timeline reference:
const secondsPerBar = (4 / gridState.masterTempo) * 60;
const pixelsPerSecond = gridState.zoomValue; // Common value
const position = time * pixelsPerSecond;      // Same formula
```

**Key Components:**

1. **Master Tempo:** 60-200 BPM (session-wide)
2. **Zoom Value:** 10-200 px/sec (pixels per second of audio)
3. **Tempo Ratio:** `audioBPM / masterTempo` (per-track scaling)

**Visual Elements:**

- **Cyan bars:** Every musical bar (4 beats)
- **Magenta markers:** Every 4 bars (phrase markers)
- **Bar numbers:** 1, 5, 9, 13... (aligned with magenta)
- **Waveform bars:** 1 per beat (beat-aligned)

### 2. Audio System (Web Audio API)

**Architecture:**
```
Audio File → AudioBuffer → BufferSource → GainNode → AnalyserNode → Destination
                                              ↓
                                         Volume Control
                                              ↓
                                         Mute/Solo Logic
                                              ↓
                                         VU Meters
```

**Multi-Track Playback:**
- 4 independent tracks (expandable)
- Per-track volume, mute, solo
- Master tempo time-stretching
- Sample-accurate synchronization

**FFT Analysis:**
- 32768 sample FFT size
- Real-time frequency analysis
- User-definable frequency ranges
- 60Hz update rate

### 3. Shader System (WebGPU)

**Render Pipeline:**
```
WGSL Shader Code → Compile → Pipeline → Render → Canvas
                                  ↑
                              Uniforms (time, audio, controls)
```

**Two Render Contexts:**

1. **Controls Tab:** Main shader creation/editing
2. **LIVE Tab:** Performance preview (beta)

**Independent Devices:**
- `device` - Controls tab WebGPU device
- `liveShaderDevice` - LIVE tab WebGPU device
- Separate pipelines, separate render loops

### 4. State Management

**Three State Systems:**

1. **Grid State** (session-wide UI)
   ```javascript
   gridState = {
       currentTab: 'controls',
       masterTempo: 120,
       zoomValue: 40,
       // ... other UI state
   }
   ```

2. **Track Container Data** (per-track audio)
   ```javascript
   trackContainerData[trackNumber] = {
       audioBuffer: AudioBuffer,
       audioBPM: 128,
       audioStartTime: 32.5,
       scaledStartTime: 30.0, // tempo-adjusted
       // ... other track data
   }
   ```

3. **Control States** (shader controls)
   ```javascript
   controlStates = {
       shaderMod_A: 0.5,
       shaderMod_B: 0.75,
       shaderFreq_Low_Min: 30,
       // ... all control values
   }
   ```

---

## BEAT GRID SYSTEM

### Core Algorithm: One Timeline

**The Problem:**
- Track A: 124 BPM
- Track B: 128 BPM
- Master Tempo: 120 BPM
- How to align visuals?

**The Solution:**
All tracks render at master tempo, time-stretched audio matches.

**Code Flow:**

```javascript
// 1. Calculate master timeline
const secondsPerBar = (4 / gridState.masterTempo) * 60;
const pixelsPerSecond = gridState.zoomValue;

// 2. Calculate tempo scaling for this track
const tempoRatio = trackContainerData[track].audioBPM / gridState.masterTempo;

// 3. Scale track's start time
const originalStartTime = trackContainerData[track].audioStartTime;
const scaledStartTime = originalStartTime * tempoRatio;

// 4. Draw beat grid at master tempo
function drawBeatGridOnWaveform(canvas, track) {
    const totalBars = Math.ceil(audioBuffer.duration / secondsPerBar);
    
    for (let bar = 0; bar < totalBars; bar++) {
        const barTime = scaledStartTime + (bar * secondsPerBar);
        const x = barTime * pixelsPerSecond;
        
        // Draw cyan bar line
        ctx.strokeStyle = 'cyan';
        ctx.beginPath();
        ctx.moveTo(x, 0);
        ctx.lineTo(x, canvas.height);
        ctx.stroke();
        
        // Draw magenta phrase marker
        if ((bar + 1) % 4 === 1) {
            ctx.strokeStyle = 'magenta';
            // ... draw phrase marker
        }
    }
}
```

### Waveform Rendering

**Beat-Aligned Bars:**

```javascript
function drawWaveformFromAudioBuffer(audioBuffer, track) {
    const secondsPerBeat = 60 / gridState.masterTempo;
    const beatCount = Math.ceil(audioBuffer.duration / secondsPerBeat);
    const tempoRatio = audioBPM / gridState.masterTempo;
    
    // Start at same position as beat grid
    let beatTime = scaledStartTime;
    
    for (let beat = 0; beat < beatCount; beat++) {
        // Convert playback time to original sample time
        const originalTime = beatTime / tempoRatio;
        const startSample = Math.floor(originalTime * sampleRate);
        
        // Read samples from ORIGINAL buffer
        const samplesPerBeat = secondsPerBeat * sampleRate / tempoRatio;
        const endSample = Math.floor(startSample + samplesPerBeat);
        
        // Calculate RMS amplitude
        let sum = 0;
        for (let i = startSample; i < endSample && i < channelData.length; i++) {
            sum += channelData[i] * channelData[i];
        }
        const rms = Math.sqrt(sum / (endSample - startSample));
        
        // Store at SCALED playback time
        levels.push({ time: beatTime, amplitude: rms });
        
        beatTime += secondsPerBeat; // Increment at master tempo
    }
    
    // Draw bars at master tempo positions
    levels.forEach(level => {
        const x = level.time * pixelsPerSecond;
        const height = level.amplitude * canvas.height;
        // ... draw bar
    });
}
```

**Key Insight:** Read samples at original tempo, draw at master tempo.

### Scroll Synchronization

**The Critical Discovery:**

```javascript
// WRONG (causes drift):
const audioTimeElapsed = realTimeElapsed * playbackRate;
const scrollOffset = audioTimeElapsed * pixelsPerSecond;

// CORRECT (perfect sync):
const scrollOffset = realTimeElapsed * pixelsPerSecond;
```

**Why:** Time-stretched audio plays at master tempo → real time = playback time

**Implementation:**

```javascript
function startAudioWaveformScroll(track, audioBuffer) {
    const canvas = /* get waveform canvas */;
    const pixelsPerSecond = gridState.zoomValue;
    const startTimestamp = performance.now();
    
    function scroll() {
        const realTimeElapsed = (performance.now() - startTimestamp) / 1000;
        const scrollOffset = realTimeElapsed * pixelsPerSecond;
        
        canvas.style.transform = `translateX(-${scrollOffset}px)`;
        
        // Continue loop
        const requestId = requestAnimationFrame(scroll);
        trackContainerData[track].scrollAnimationId = requestId;
    }
    
    scroll();
}
```

### Tempo Change Handling

**Problem:** Changing master tempo during playback causes drift.

**Solution:** Restart scroll animations when tempo changes.

```javascript
function handleTempoChange(newTempo) {
    gridState.masterTempo = newTempo;
    
    // Update all track playback rates
    for (let track = 1; track <= 4; track++) {
        const source = trackContainerData[track].source;
        if (source) {
            const audioBPM = trackContainerData[track].audioBPM;
            source.playbackRate.value = newTempo / audioBPM;
        }
        
        // Redraw waveforms at new tempo
        const audioBuffer = trackContainerData[track].audioBuffer;
        if (audioBuffer) {
            drawWaveformFromAudioBuffer(audioBuffer, track);
        }
        
        // Restart scroll animation at new speed
        if (trackContainerData[track].scrollAnimationId) {
            stopAudioWaveformScroll(track);
            startAudioWaveformScroll(track, audioBuffer);
        }
    }
}
```

---

## AUDIO SYSTEM

### Multi-Track Architecture

**Per-Track Signal Chain:**

```javascript
// Audio routing for one track
const source = audioContext.createBufferSource();
source.buffer = audioBuffer;
source.playbackRate.value = masterTempo / audioBPM; // Time-stretch

const gainNode = audioContext.createGain();
gainNode.gain.value = volume; // 0.0 to 1.0

const analyserNode = audioContext.createAnalyser();
analyserNode.fftSize = 32768;

// Connect nodes
source.connect(gainNode);
gainNode.connect(analyserNode);
analyserNode.connect(audioContext.destination);

// Store references
trackContainerData[track].source = source;
trackContainerData[track].gainNode = gainNode;
trackContainerData[track].analyserNode = analyserNode;
```

### Mixer Controls

**Volume Control:**
```javascript
function setTrackVolume(track, value) {
    const gainNode = trackContainerData[track].gainNode;
    if (gainNode) {
        gainNode.gain.value = value; // 0.0 to 1.0
    }
    trackContainerData[track].volume = value;
}
```

**Mute Logic:**
```javascript
function toggleMute(track) {
    const isMuted = trackContainerData[track].muted;
    const gainNode = trackContainerData[track].gainNode;
    
    if (gainNode) {
        if (isMuted) {
            // Unmute - restore volume
            gainNode.gain.value = trackContainerData[track].volume;
        } else {
            // Mute - zero volume
            gainNode.gain.value = 0;
        }
    }
    
    trackContainerData[track].muted = !isMuted;
}
```

**Solo Logic:**
```javascript
function toggleSolo(track) {
    const wasSoloed = trackContainerData[track].soloed;
    trackContainerData[track].soloed = !wasSoloed;
    
    // Check if any tracks soloed
    const anyTrackSoloed = [1,2,3,4].some(t => 
        trackContainerData[t].soloed
    );
    
    // Apply solo logic to all tracks
    for (let t = 1; t <= 4; t++) {
        const gainNode = trackContainerData[t].gainNode;
        if (!gainNode) continue;
        
        if (anyTrackSoloed) {
            // Solo mode: only soloed tracks audible
            if (trackContainerData[t].soloed) {
                gainNode.gain.value = trackContainerData[t].volume;
            } else {
                gainNode.gain.value = 0;
            }
        } else {
            // Normal mode: respect mute status
            if (trackContainerData[t].muted) {
                gainNode.gain.value = 0;
            } else {
                gainNode.gain.value = trackContainerData[t].volume;
            }
        }
    }
}
```

### FFT Analysis

**Real-Time Frequency Data:**

```javascript
function updateFFTData() {
    const analyserNode = /* get analyser from active track */;
    if (!analyserNode) return;
    
    const bufferLength = analyserNode.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    
    analyserNode.getByteFrequencyData(dataArray);
    
    // Convert to user-defined frequency ranges
    const freqLow = extractFrequencyRange(dataArray, 
        controlStates.shaderFreq_Low_Min, 
        controlStates.shaderFreq_Low_Max
    );
    
    const freqMid = extractFrequencyRange(dataArray,
        controlStates.shaderFreq_Mid_Min,
        controlStates.shaderFreq_Mid_Max
    );
    
    const freqHigh = extractFrequencyRange(dataArray,
        controlStates.shaderFreq_High_Min,
        controlStates.shaderFreq_High_Max
    );
    
    // Update shader uniforms
    uniforms.audioLow = freqLow / 255.0;
    uniforms.audioMid = freqMid / 255.0;
    uniforms.audioHigh = freqHigh / 255.0;
    
    requestAnimationFrame(updateFFTData);
}
```

---

## SHADER SYSTEM

### WebGPU Pipeline Setup

**Device Initialization:**

```javascript
async function initWebGPU() {
    // Request adapter
    const adapter = await navigator.gpu?.requestAdapter();
    if (!adapter) {
        console.error('WebGPU not supported');
        return false;
    }
    
    // Request device
    const device = await adapter.requestDevice();
    
    // Get canvas context
    const canvas = document.querySelector('#shader-canvas');
    const context = canvas.getContext('webgpu');
    
    // Configure context
    const format = navigator.gpu.getPreferredCanvasFormat();
    context.configure({
        device: device,
        format: format,
        alphaMode: 'premultiplied'
    });
    
    // Store globally
    window.device = device;
    window.context = context;
    window.format = format;
    
    return true;
}
```

**Shader Compilation:**

```javascript
function compileShader(shaderCode) {
    // Create shader module
    const shaderModule = device.createShaderModule({
        label: 'User Shader',
        code: shaderCode
    });
    
    // Create render pipeline
    const pipeline = device.createRenderPipeline({
        label: 'Shader Pipeline',
        layout: 'auto',
        vertex: {
            module: shaderModule,
            entryPoint: 'vs_main'
        },
        fragment: {
            module: shaderModule,
            entryPoint: 'fs_main',
            targets: [{
                format: format
            }]
        },
        primitive: {
            topology: 'triangle-strip'
        }
    });
    
    return pipeline;
}
```

### Uniform Buffer Management

**Structure:**

```javascript
// WGSL uniform definition
struct Uniforms {
    iTime: f32,
    iResolution: vec2<f32>,
    audioLow: f32,
    audioMid: f32,
    audioHigh: f32,
    mod_A: f32,
    mod_B: f32,
    mod_C: f32,
    // ... all control values
}

// JavaScript buffer creation
const uniformBuffer = device.createBuffer({
    size: 256, // Align to 256 bytes
    usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});
```

**Update Uniforms:**

```javascript
function updateUniforms() {
    const uniformData = new Float32Array([
        performance.now() / 1000,     // iTime
        canvas.width, canvas.height,   // iResolution
        audioLow, audioMid, audioHigh, // FFT data
        controlStates.shaderMod_A,     // Modulator A
        controlStates.shaderMod_B,     // Modulator B
        controlStates.shaderMod_C,     // Modulator C
        // ... all other controls
    ]);
    
    device.queue.writeBuffer(uniformBuffer, 0, uniformData);
}
```

### Render Loop

**Controls Tab:**

```javascript
function renderShader() {
    updateUniforms();
    
    const commandEncoder = device.createCommandEncoder();
    const textureView = context.getCurrentTexture().createView();
    
    const renderPass = commandEncoder.beginRenderPass({
        colorAttachments: [{
            view: textureView,
            clearValue: { r: 0, g: 0, b: 0, a: 1 },
            loadOp: 'clear',
            storeOp: 'store'
        }]
    });
    
    renderPass.setPipeline(pipeline);
    renderPass.setBindGroup(0, bindGroup);
    renderPass.draw(4); // Full-screen quad
    renderPass.end();
    
    device.queue.submit([commandEncoder.finish()]);
    
    requestAnimationFrame(renderShader);
}
```

**LIVE Tab (Beta):**

```javascript
function renderLiveShader() {
    // Independent render loop
    // Uses liveShaderDevice, livePipeline, liveBindGroup
    // Same structure as Controls tab render
    // Future: composite 4 track shaders
    
    requestAnimationFrame(renderLiveShader);
}
```

---

## STATE MANAGEMENT

### .oms File Format

**JSON Structure:**

```json
{
    "shaderCode": "/* WGSL shader source */",
    "controlStates": {
        "shaderMod_A": 0.5,
        "shaderMod_B": 0.75,
        "shaderMod_C": 0.25,
        "shaderFreq_Low_Min": 30,
        "shaderFreq_Low_Max": 75,
        "audioStartTime": 32.5,
        "audioEndTime": 180.0,
        "audioBPM": 128.0,
        "audioPitch": 2.5
    }
}
```

**Save .oms:**

```javascript
async function saveOMS(fileHandle) {
    const omsData = {
        shaderCode: editor.getValue(),
        controlStates: { ...controlStates }
    };
    
    const jsonString = JSON.stringify(omsData, null, 2);
    const writable = await fileHandle.createWritable();
    await writable.write(jsonString);
    await writable.close();
}
```

**Load .oms:**

```javascript
async function loadOMS(fileHandle) {
    const file = await fileHandle.getFile();
    const text = await file.text();
    const omsData = JSON.parse(text);
    
    // Restore shader code
    editor.setValue(omsData.shaderCode);
    compileAndRenderShader();
    
    // Restore control states
    Object.assign(controlStates, omsData.controlStates);
    updateAllControls();
}
```

### .omsp Project Format

**Structure:**

```json
{
    "gridState": {
        "masterTempo": 128,
        "zoomValue": 50
    },
    "tracks": {
        "1": {
            "omsPath": "/path/to/preset1.oms",
            "audioPath": "/path/to/track1.mp3",
            "volume": 0.8,
            "muted": false,
            "soloed": false
        },
        "2": { /* ... */ },
        "3": { /* ... */ },
        "4": { /* ... */ }
    }
}
```

---

## KEY ALGORITHMS

### 1. Tempo Scaling

```javascript
function calculateTempoScaling(originalBPM, masterTempo, originalTime) {
    const tempoRatio = originalBPM / masterTempo;
    const scaledTime = originalTime * tempoRatio;
    return { tempoRatio, scaledTime };
}
```

### 2. Sample-to-Pixel Conversion

```javascript
function sampleToPixel(sampleTime, pixelsPerSecond, tempoRatio) {
    const playbackTime = sampleTime * tempoRatio;
    const pixel = playbackTime * pixelsPerSecond;
    return pixel;
}
```

### 3. Frequency Range Extraction

```javascript
function extractFrequencyRange(fftData, minHz, maxHz) {
    const sampleRate = audioContext.sampleRate;
    const fftSize = fftData.length * 2;
    
    const minBin = Math.floor((minHz / sampleRate) * fftSize);
    const maxBin = Math.ceil((maxHz / sampleRate) * fftSize);
    
    let sum = 0;
    let count = 0;
    for (let i = minBin; i <= maxBin && i < fftData.length; i++) {
        sum += fftData[i];
        count++;
    }
    
    return count > 0 ? sum / count : 0;
}
```

### 4. RMS Amplitude Calculation

```javascript
function calculateRMS(samples, startIdx, endIdx) {
    let sum = 0;
    let count = 0;
    
    for (let i = startIdx; i < endIdx && i < samples.length; i++) {
        sum += samples[i] * samples[i];
        count++;
    }
    
    return count > 0 ? Math.sqrt(sum / count) : 0;
}
```

---

## API REFERENCE

### Global Functions

#### Audio Control

```javascript
// Load audio file
async loadAudioFile(track: number, file: File): Promise<void>

// Start playback
startAudioPlayback(track: number): void

// Stop playback
stopAudioPlayback(track: number): void

// Set volume
setTrackVolume(track: number, volume: number): void // 0.0-1.0

// Mute/unmute
toggleMute(track: number): void

// Solo/unsolo
toggleSolo(track: number): void
```

#### Beat Grid

```javascript
// Draw beat grid
drawBeatGridOnWaveform(canvas: HTMLCanvasElement, track: number): void

// Draw waveform
drawWaveformFromAudioBuffer(audioBuffer: AudioBuffer, track: number): void

// Position at playhead
positionAudioWaveformAtPlayhead(track: number): void

// Start scroll animation
startAudioWaveformScroll(track: number, audioBuffer: AudioBuffer): void

// Stop scroll animation
stopAudioWaveformScroll(track: number): void
```

#### Shader Management

```javascript
// Compile shader
compileShader(shaderCode: string): GPURenderPipeline

// Update uniforms
updateUniforms(): void

// Render frame
renderShader(): void
```

#### State Management

```javascript
// Save .oms file
async saveOMS(fileHandle: FileSystemFileHandle): Promise<void>

// Load .oms file
async loadOMS(fileHandle: FileSystemFileHandle): Promise<void>

// Save .omsp project
async saveProject(fileHandle: FileSystemFileHandle): Promise<void>

// Load .omsp project
async loadProject(fileHandle: FileSystemFileHandle): Promise<void>
```

### Global State Objects

```javascript
// Grid and UI state
gridState = {
    currentTab: string,
    masterTempo: number,      // 60-200 BPM
    zoomValue: number,         // 10-200 px/sec
    minZoom: number,
    maxZoom: number,
    libraryFolderHandle: FileSystemDirectoryHandle | null,
    currentPresetHandle: FileSystemFileHandle | null
}

// Per-track audio data
trackContainerData[1..4] = {
    audioBuffer: AudioBuffer | null,
    audioBPM: number,
    audioStartTime: number,          // Original time
    scaledStartTime: number,         // Tempo-adjusted time
    source: AudioBufferSourceNode | null,
    gainNode: GainNode | null,
    analyserNode: AnalyserNode | null,
    volume: number,                  // 0.0-1.0
    muted: boolean,
    soloed: boolean,
    scrollAnimationId: number | null
}

// Shader control values
controlStates = {
    shaderMod_A: number,             // 0.0-1.0
    shaderMod_B: number,
    shaderMod_C: number,
    shaderFreq_Low_Min: number,      // Hz
    shaderFreq_Low_Max: number,
    shaderFreq_Mid_Min: number,
    shaderFreq_Mid_Max: number,
    shaderFreq_High_Min: number,
    shaderFreq_High_Max: number,
    audioStartTime: number,
    audioEndTime: number,
    audioBPM: number,
    audioPitch: number
}
```

---

## DEVELOPMENT GUIDELINES

### Version Naming Convention

**Production Release:**
- Filename: `OMS_ShaderBaker_V3_3_0.html`
- Title: `v3.3.0` (lowercase v)
- Header: `[V3.3.0]` (brackets)

**Development Build:**
- Filename: `OMS_ShaderBaker_V3_3_0_A.html`
- Title: `v3.3.0a` (lowercase v and letter)
- Header: `[V3.3.0_A]` (underscore before letter)

**CRITICAL:** Use underscores not periods: `V3_3_0_A` (not `V3.3.0_A`)

### Iteration Protocol

1. **Alphabetical:** A, B, C... Z, AA, AB, AC...
2. **Sequential:** Never skip letters
3. **One Feature:** One task per iteration

### Code Modification Rules

1. **Surgical Changes:** Modify specific functions only
2. **Preserve Functionality:** Don't break existing features
3. **Test Incrementally:** Validate each change
4. **Single File:** Never split into multiple files

### Testing Checklist

Before releasing any version:

- [ ] All tabs load without errors
- [ ] WebGPU shaders compile and render
- [ ] Audio loads and plays correctly
- [ ] Beat grids align across all tracks
- [ ] Tempo changes work without drift
- [ ] .oms files save and load
- [ ] .omsp projects save and load
- [ ] Mixer controls function properly
- [ ] Zoom control works smoothly
- [ ] Keyboard shortcuts respond

### Performance Targets

- 60fps shader rendering
- < 50ms audio latency
- Smooth scrolling with 4 tracks loaded
- Instant tempo changes (no lag)
- < 500KB file size

---

## DEBUGGING TIPS

### Common Issues

**Beat Grid Not Aligned:**
- Check `scaledStartTime` calculation
- Verify `tempoRatio` is correct
- Ensure all tracks use same `pixelsPerSecond`

**Audio Drift:**
- Check `playbackRate.value = masterTempo / audioBPM`
- Verify scroll uses `realTimeElapsed` not `audioTimeElapsed`
- Ensure scroll animation restarted on tempo change

**Shader Not Rendering:**
- Check WebGPU device initialization
- Verify shader compiles without errors
- Check uniform buffer size (256 byte alignment)

**Waveforms Not Visible:**
- Check canvas size calculation
- Verify audio buffer loaded correctly
- Check `pixelsPerSecond` and zoom value

### Browser Console Commands

```javascript
// Check WebGPU support
navigator.gpu

// View grid state
console.log(gridState)

// View track data
console.log(trackContainerData)

// Check audio context
console.log(audioContext.state)

// Force tempo recalculation
handleTempoChange(gridState.masterTempo)
```

---

## CONTRIBUTING

### Before Modifying Code

1. Read this documentation fully
2. Understand the One Timeline principle
3. Test changes in isolated iterations
4. Follow version naming conventions
5. Update documentation if needed

### Pull Request Guidelines

1. One feature per PR
2. Include iteration notes
3. Test on Chrome and Safari TP
4. Maintain backward compatibility
5. Update changelog

---

## LICENSE

GPL-3.0 - Free and Open Source

Copyright (C) 2024-2025 Wes Jones / ONEMANSHYO

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

---

**END OF DEVELOPER DOCUMENTATION**

**For More Information:**
- Release Notes: OMS_ShaderBaker_V3_3_0_D_Release_Notes.md
- User Guide: OMS_ShaderBaker_V3_3_0_F_UserGuide_All.md
- Claude Instructions: OMS_ShaderBaker_V3_3_0_Z_ClaudeProject_Instructions.md
