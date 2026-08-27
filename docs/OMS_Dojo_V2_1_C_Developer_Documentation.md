# ONEMANSHYO Dojo - Developer Documentation

**Version:** V2_1  
**Target Audience:** Developers, technical users, contributors  
**Purpose:** Technical architecture, APIs, and extension guide

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [File Structure](#file-structure)
- [UI Architecture](#ui-architecture)
- [WebGPU Integration](#webgpu-integration)
- [Audio System](#audio-system)
- [Modulation System](#modulation-system)
- [Video Export System](#video-export-system)
- [API Reference](#api-reference)
- [Extension Guide](#extension-guide)
- [Performance Notes](#performance-notes)

---

## Architecture Overview

### Single-File Application

**Philosophy:**
- Everything in one HTML file
- No external dependencies (except browser APIs)
- No build process required
- Easy to distribute and run

**Structure:**
```
OMS_Dojo_V2_1.html (~2,680 lines, ~115KB)
├── HTML Structure (lines 1-~900)
│   ├── Meta tags & title
│   ├── CSS (<style> tag)
│   └── DOM structure
├── JavaScript (lines ~900-2680)
│   ├── WebGPU initialization
│   ├── Audio system
│   ├── Modulation engine
│   ├── UI handlers
│   └── Video export
└── Default Shader (embedded in <textarea>)
```

### Technology Stack

**Core Technologies:**
- WebGPU (graphics)
- WGSL (shading language)
- Web Audio API (audio analysis)
- MediaRecorder API (video capture)
- Canvas 2D API (waveform)

**Browser Requirements:**
- Chrome 113+
- Edge 113+
- Safari Technology Preview

---

## File Structure

### HTML Structure (~900 lines)

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ONEMANSHYO Dojo V2_1 - Audio Reactive</title>
  
  <style>
    /* ~400 lines of CSS */
    /* Layout, controls, tabs, canvas styling */
  </style>
</head>

<body>
  <!-- Left Panel -->
  <div class="left-panel">
    <!-- Header with version -->
    <!-- Tab Bar (Controls / Code) -->
    
    <!-- Controls Tab -->
    <div id="controlsTabContent">
      <!-- 5 rows of shader controls -->
    </div>
    
    <!-- Code Tab -->
    <div id="codeTabContent">
      <textarea id="shaderCode">
        <!-- Default shader code -->
      </textarea>
    </div>
  </div>
  
  <!-- Right Panel -->
  <div class="right-panel">
    <!-- Canvas (1920x1080) -->
    <!-- Waveform Section -->
    <!-- Audio Controls Strip -->
    <!-- Status Bar -->
  </div>
  
  <script>
    /* ~1,780 lines of JavaScript */
  </script>
</body>
</html>
```

### JavaScript Structure (~1,780 lines)

**Global Variables:**
```javascript
// WebGPU (lines ~900-950)
let device, context, pipeline, uniformBuffer, uniformBindGroup;

// Audio (lines ~950-1000)
let audioContext, analyserNode, audioElement, audioSource;
let audioLoaded = false;
let bass = 0, mid = 0, treble = 0;

// Modulation (lines ~1000-1050)
let currentBeat = 0;
let modA = 0, modB = 0, modC = 0;

// Waveform (lines ~1050-1100)
let waveformCanvas, waveformCtx, staticWaveformData;

// Video Export (lines ~1100-1150)
let isRendering = false;
let mediaRecorder, recordedChunks;
```

**Function Organization:**
```javascript
// WebGPU System (lines ~1200-1500)
initWebGPU()           // Initialize device
updateCanvasSize()     // Set canvas dimensions
compileShader()        // Compile user shader
render()               // Main render loop

// Audio System (lines ~1500-1800)
loadAudioFile()        // Load and decode audio
analyzeAudio()         // FFT analysis
toggleAudioPlayback()  // Play/pause
stopAudio()            // Stop and reset
removeAudio()          // Clear audio

// Waveform System (lines ~1800-2100)
generateStaticWaveform()  // Create waveform image
drawRealWaveform()        // Render waveform
setupWaveformControls()   // Interaction handlers
drawProgress()            // Update playhead

// Modulation System (lines ~2100-2300)
calculateMod()         // Compute modulator values
updateUniforms()       // Update GPU buffer

// Video Export (lines ~2300-2500)
startRender()          // Begin video capture
finishRender()         // Complete and download

// UI Helpers (lines ~2500-2680)
showStatus()           // Status bar messages
switchTab()            // Tab switching
setupInputHandlers()   // Control validation
```

---

## UI Architecture

### Tab System (V2_1)

**Implementation:**
```javascript
function switchTab(tab) {
    // Update tab buttons
    const tabs = document.querySelectorAll('.tab');
    tabs.forEach(t => t.classList.remove('active'));
    document.getElementById(tab + 'Tab').classList.add('active');
    
    // Update content visibility
    const contents = document.querySelectorAll('.tab-content');
    contents.forEach(c => c.classList.remove('active'));
    document.getElementById(tab + 'TabContent').classList.add('active');
}
```

**CSS Classes:**
```css
.tab {
    /* Tab button styling */
    background: rgba(255,255,255,0.05);
}

.tab.active {
    /* Active tab */
    background: rgba(0,212,255,0.2);
    border-bottom: 2px solid #00d4ff;
}

.tab-content {
    /* Content panel */
    display: none;
}

.tab-content.active {
    /* Visible content */
    display: flex;
}
```

### Canvas Sizing (V2_1)

**Default Canvas:**
```javascript
// 1920x1080 default (Full HD)
const canvas = document.getElementById('canvas');
canvas.width = 1920;
canvas.height = 1080;
```

**Orientation Scaling:**
```javascript
function updateCanvasSize() {
    const orientation = document.getElementById('orientation').value;
    const resolution = parseInt(document.getElementById('resolution').value);
    
    let width, height;
    switch(orientation) {
        case 'landscape':
            width = resolution * (16/9);  // 16:9 aspect
            height = resolution;
            break;
        case 'portrait':
            width = resolution * (9/16);  // 9:16 aspect
            height = resolution;
            break;
        case 'square':
            width = resolution;           // 1:1 aspect
            height = resolution;
            break;
    }
    
    canvas.width = width;
    canvas.height = height;
    // ... update WebGPU context
}
```

### Waveform on Canvas (V2_1)

**Layout:**
```
┌─────────────────────────────────────┐
│  Canvas (1920x1080)                 │
│  WebGPU shader preview              │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│  Waveform (1920x180)                │
│  Audio visualization                │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│  Audio Controls Strip (40px)        │
│  ▶ ⏹ │ IN [64] OUT [128]           │
└─────────────────────────────────────┘
```

**Waveform Sizing:**
```javascript
const waveformCanvas = document.getElementById('waveformCanvas');
waveformCanvas.width = 1920;   // 6x bigger than V2.0.2
waveformCanvas.height = 180;
```

---

## WebGPU Integration

### Initialization

```javascript
async function initWebGPU() {
    // Check WebGPU support
    if (!navigator.gpu) {
        throw new Error('WebGPU not supported');
    }
    
    // Request adapter
    const adapter = await navigator.gpu.requestAdapter();
    if (!adapter) {
        throw new Error('No adapter found');
    }
    
    // Request device
    device = await adapter.requestDevice();
    
    // Setup canvas context
    const canvas = document.getElementById('canvas');
    context = canvas.getContext('webgpu');
    
    const format = navigator.gpu.getPreferredCanvasFormat();
    context.configure({
        device: device,
        format: format,
        alphaMode: 'premultiplied'
    });
    
    // Create uniform buffer
    uniformBuffer = device.createBuffer({
        size: 32,  // 8 floats × 4 bytes
        usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
    });
    
    showStatus('WebGPU Ready', 'success');
}
```

### Shader Compilation

```javascript
function compileShader(showMessage = false) {
    try {
        // Get user shader code
        const shaderCode = document.getElementById('shaderCode').value;
        
        // Auto-generate vertex shader
        const vertexShaderCode = `
            @vertex
            fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
                var pos = array<vec2<f32>, 4>(
                    vec2<f32>(-1.0, -1.0),
                    vec2<f32>( 1.0, -1.0),
                    vec2<f32>(-1.0,  1.0),
                    vec2<f32>( 1.0,  1.0)
                );
                return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
            }
        `;
        
        // Prepend uniform structure
        const fullShaderCode = `
            struct Uniforms {
                time: f32,
                tempo: f32,
                modA: f32,
                modB: f32,
                modC: f32,
                padding: f32,
                resolution: vec2<f32>,
            }
            
            @group(0) @binding(0) var<uniform> uniforms: Uniforms;
            
            ${vertexShaderCode}
            ${shaderCode}
        `;
        
        // Create shader module
        const shaderModule = device.createShaderModule({
            code: fullShaderCode
        });
        
        // Create pipeline
        pipeline = device.createRenderPipeline({
            layout: 'auto',
            vertex: {
                module: shaderModule,
                entryPoint: 'vs_main'
            },
            fragment: {
                module: shaderModule,
                entryPoint: 'fs_main',
                targets: [{
                    format: navigator.gpu.getPreferredCanvasFormat()
                }]
            },
            primitive: {
                topology: 'triangle-strip'
            }
        });
        
        // Create bind group
        uniformBindGroup = device.createBindGroup({
            layout: pipeline.getBindGroupLayout(0),
            entries: [{
                binding: 0,
                resource: {
                    buffer: uniformBuffer
                }
            }]
        });
        
        if (showMessage) {
            showStatus('Shader compiled successfully', 'success');
        }
        
    } catch (error) {
        showStatus(`Shader error: ${error.message}`, 'error');
        console.error(error);
    }
}
```

### Render Loop

```javascript
function render() {
    if (!pipeline) return;
    
    // Calculate time and beat
    const time = (audioLoaded && audioElement && !audioElement.paused) 
        ? audioElement.currentTime 
        : performance.now() / 1000;
    
    const tempo = parseFloat(document.getElementById('tempo').value) || 120;
    const beatsPerSecond = tempo / 60.0;
    currentBeat = time * beatsPerSecond;
    
    // Analyze audio (if playing)
    if (audioLoaded && audioElement && !audioElement.paused) {
        analyzeAudio();
    }
    
    // Calculate modulators
    modA = calculateMod('A', currentBeat) * (1 + bass * bassGain);
    modB = calculateMod('B', currentBeat) * (1 + mid * midGain);
    modC = calculateMod('C', currentBeat) * (1 + treble * trebleGain);
    
    // Update uniform buffer
    const uniformData = new Float32Array([
        time,
        tempo,
        modA,
        modB,
        modC,
        0,  // padding
        canvas.width,
        canvas.height
    ]);
    device.queue.writeBuffer(uniformBuffer, 0, uniformData);
    
    // Render
    const commandEncoder = device.createCommandEncoder();
    const textureView = context.getCurrentTexture().createView();
    
    const renderPassDescriptor = {
        colorAttachments: [{
            view: textureView,
            clearValue: { r: 0, g: 0, b: 0, a: 1 },
            loadOp: 'clear',
            storeOp: 'store'
        }]
    };
    
    const passEncoder = commandEncoder.beginRenderPass(renderPassDescriptor);
    passEncoder.setPipeline(pipeline);
    passEncoder.setBindGroup(0, uniformBindGroup);
    passEncoder.draw(4, 1, 0, 0);
    passEncoder.end();
    
    device.queue.submit([commandEncoder.finish()]);
    
    // Update waveform progress
    if (audioLoaded) {
        drawProgress();
    }
    
    // Continue loop
    requestAnimationFrame(render);
}
```

### Uniform Structure

```wgsl
struct Uniforms {
    time: f32,              // Elapsed time in seconds
    tempo: f32,             // BPM (for shader timing)
    modA: f32,              // Modulator A (0.0-1.0, audio-reactive)
    modB: f32,              // Modulator B (0.0-1.0, audio-reactive)
    modC: f32,              // Modulator C (0.0-1.0, audio-reactive)
    padding: f32,           // Alignment padding
    resolution: vec2<f32>,  // Canvas width, height
}
```

---

## Audio System

### Architecture

```
Audio File → AudioElement → AudioContext → AnalyserNode → FFT
                                        ↓
                                   Destination (speakers)
                                        ↓
                                   MediaStreamDestination (recording)
```

### Loading Audio

```javascript
function loadAudioFile(event) {
    const file = event.target.files[0];
    if (!file) return;
    
    // Create audio element
    audioElement = new Audio();
    audioElement.src = URL.createObjectURL(file);
    
    // Wait for metadata
    audioElement.addEventListener('loadedmetadata', () => {
        // Setup Web Audio API
        if (!audioContext) {
            audioContext = new (window.AudioContext || window.webkitAudioContext)();
        }
        
        // Create audio source
        audioSource = audioContext.createMediaElementSource(audioElement);
        
        // Create analyser
        analyserNode = audioContext.createAnalyser();
        analyserNode.fftSize = 256;
        
        // Connect graph
        audioSource.connect(analyserNode);
        analyserNode.connect(audioContext.destination);
        
        // Generate waveform
        generateStaticWaveform(audioElement);
        
        // Enable controls
        document.getElementById('playBtn').disabled = false;
        document.getElementById('stopBtn').disabled = false;
        
        audioLoaded = true;
        showStatus('Audio loaded', 'success');
    });
}
```

### FFT Analysis

```javascript
function analyzeAudio() {
    if (!analyserNode) return;
    
    const bufferLength = analyserNode.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    analyserNode.getByteFrequencyData(dataArray);
    
    // Calculate frequency bands
    const bassEnd = Math.floor(bufferLength * 0.1);    // 0-10%
    const midEnd = Math.floor(bufferLength * 0.5);     // 10-50%
    const trebleEnd = bufferLength;                    // 50-100%
    
    // Average each band
    let bassSum = 0, midSum = 0, trebleSum = 0;
    
    for (let i = 0; i < bassEnd; i++) bassSum += dataArray[i];
    for (let i = bassEnd; i < midEnd; i++) midSum += dataArray[i];
    for (let i = midEnd; i < trebleEnd; i++) trebleSum += dataArray[i];
    
    const bassAvg = bassSum / bassEnd;
    const midAvg = midSum / (midEnd - bassEnd);
    const trebleAvg = trebleSum / (trebleEnd - midEnd);
    
    // Normalize to 0-1 range
    const bassNorm = bassAvg / 255;
    const midNorm = midAvg / 255;
    const trebleNorm = trebleAvg / 255;
    
    // Exponential smoothing (prevents jitter)
    const smooth = 0.7;
    bass = bass * smooth + bassNorm * (1 - smooth);
    mid = mid * smooth + midNorm * (1 - smooth);
    treble = treble * smooth + trebleNorm * (1 - smooth);
}
```

### Transport Controls (V2_1)

```javascript
function toggleAudioPlayback() {
    if (!audioLoaded || !audioElement) return;
    
    const playBtn = document.getElementById('playBtn');
    
    if (audioElement.paused) {
        audioElement.play();
        playBtn.textContent = '⏸';
        playBtn.classList.add('playing');
    } else {
        audioElement.pause();
        playBtn.textContent = '▶';
        playBtn.classList.remove('playing');
    }
}

function stopAudio() {
    if (!audioLoaded || !audioElement) return;
    
    audioElement.pause();
    audioElement.currentTime = 0;
    
    const playBtn = document.getElementById('playBtn');
    playBtn.textContent = '▶';
    playBtn.classList.remove('playing');
}
```

---

## Modulation System

### Calculation

```javascript
function calculateMod(modName, beat) {
    // Get subdivision
    const subdiv = parseFloat(document.getElementById(`mod${modName}Subdiv`).value);
    
    // Meta-modes (handle separately)
    if (isNaN(subdiv)) {
        // Chaos, Cycler, Mad Max, Psycho
        return handleMetaMode(subdiv, beat);
    }
    
    // Calculate phase (0-1 repeating cycle)
    const frequency = 1 / subdiv;
    const phase = (beat * frequency) % 1.0;
    
    // Get waveform
    const waveform = document.getElementById(`mod${modName}Wave`).value;
    
    // Generate waveform value
    switch(waveform) {
        case 'sine':
            return Math.sin(phase * Math.PI * 2) * 0.5 + 0.5;
        
        case 'cosine':
            return Math.cos(phase * Math.PI * 2) * 0.5 + 0.5;
        
        case 'sawtooth':
            return phase;
        
        case 'triangle':
            return phase < 0.5 ? phase * 2 : 2 - phase * 2;
        
        case 'square':
            return phase < 0.5 ? 0 : 1;
        
        case 'noise':
            return Math.random();
        
        // ... other waveforms
    }
}
```

### Audio Multiplication

```javascript
// In render loop:
const bassGain = parseFloat(document.getElementById('bassGain').value) || 0;
const midGain = parseFloat(document.getElementById('midGain').value) || 0;
const trebleGain = parseFloat(document.getElementById('trebleGain').value) || 0;

modA = calculateMod('A', currentBeat) * (1 + bass * bassGain);
modB = calculateMod('B', currentBeat) * (1 + mid * midGain);
modC = calculateMod('C', currentBeat) * (1 + treble * trebleGain);
```

---

## Video Export System

### Render Process

```javascript
function startRender() {
    // Validate inputs
    const filename = document.getElementById('filename').value || 'shader';
    const resolution = parseInt(document.getElementById('resolution').value);
    const fps = parseInt(document.getElementById('fps').value);
    const inBeat = parseInt(document.getElementById('inBeat').value) || 0;
    const outBeat = parseInt(document.getElementById('outBeat').value) || 128;
    
    // Calculate duration
    const tempo = parseFloat(document.getElementById('tempo').value);
    const beatsPerSecond = tempo / 60;
    const beatCount = outBeat - inBeat;
    const duration = beatCount / beatsPerSecond;
    
    // Setup canvas capture
    const stream = canvas.captureStream();  // No FPS param = capture every frame
    
    // Add audio track if available
    if (audioLoaded && audioElement) {
        const audioCtx = new AudioContext();
        const source = audioCtx.createMediaElementSource(audioElement);
        const dest = audioCtx.createMediaStreamDestination();
        source.connect(dest);
        source.connect(audioCtx.destination);
        
        stream.addTrack(dest.stream.getAudioTracks()[0]);
    }
    
    // Create MediaRecorder
    recordedChunks = [];
    mediaRecorder = new MediaRecorder(stream, {
        mimeType: 'video/webm;codecs=vp8',
        videoBitsPerSecond: 8000000  // 8 Mbps
    });
    
    mediaRecorder.ondataavailable = (e) => {
        if (e.data.size > 0) {
            recordedChunks.push(e.data);
        }
    };
    
    mediaRecorder.onstop = () => {
        finishRender(filename, resolution, fps, duration, beatCount);
    };
    
    // Start recording
    mediaRecorder.start();
    isRendering = true;
    
    // Reset audio to IN beat
    if (audioLoaded) {
        const inTime = inBeat / beatsPerSecond;
        audioElement.currentTime = inTime;
        audioElement.play();
    }
    
    // Stop after duration
    setTimeout(() => {
        mediaRecorder.stop();
        if (audioLoaded) audioElement.pause();
        isRendering = false;
    }, duration * 1000);
}
```

### Filename Generation

```javascript
function finishRender(filename, resolution, fps, duration, beatCount) {
    // Get orientation suffix
    const orientation = document.getElementById('orientation').value;
    const suffix = orientation === 'landscape' ? 'L' 
                 : orientation === 'portrait' ? 'P' 
                 : 'S';  // Square
    
    // Generate filename
    const finalName = `OMS_${filename}_${resolution}p_${fps}fps_${Math.round(duration)}s_${beatCount}beats_${suffix}.webm`;
    
    // Create blob and download
    const blob = new Blob(recordedChunks, { type: 'video/webm' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = finalName;
    a.click();
    
    showStatus('Video saved!', 'success');
}
```

---

## API Reference

### Public Functions

**WebGPU:**
- `initWebGPU()` - Initialize WebGPU device
- `updateCanvasSize()` - Resize canvas based on orientation/resolution
- `compileShader(showMessage)` - Compile user shader code
- `render()` - Main render loop

**Audio:**
- `loadAudioFile(event)` - Load audio from file input
- `analyzeAudio()` - Perform FFT analysis
- `toggleAudioPlayback()` - Play/pause audio
- `stopAudio()` - Stop and reset audio
- `removeAudio()` - Clear loaded audio

**Waveform:**
- `generateStaticWaveform(audioElement)` - Create waveform visualization
- `drawRealWaveform()` - Render waveform to canvas
- `setupWaveformControls()` - Setup interaction handlers
- `drawProgress()` - Update playhead position

**Modulation:**
- `calculateMod(modName, beat)` - Calculate modulator value
- `handleMetaMode(mode, beat)` - Handle Chaos/Cycler/etc

**Video:**
- `startRender()` - Begin video capture
- `finishRender(...)` - Complete and download video

**UI:**
- `showStatus(message, type)` - Update status bar
- `switchTab(tab)` - Switch between Controls/Code tabs
- `setupInputHandlers()` - Initialize control validation

### Global State

```javascript
// WebGPU
device: GPUDevice
context: GPUCanvasContext
pipeline: GPURenderPipeline
uniformBuffer: GPUBuffer
uniformBindGroup: GPUBindGroup

// Audio
audioContext: AudioContext
analyserNode: AnalyserNode
audioElement: HTMLAudioElement
audioSource: MediaElementAudioSourceNode
audioLoaded: boolean
bass: number (0-1)
mid: number (0-1)
treble: number (0-1)

// Modulation
currentBeat: number
modA: number (0-1)
modB: number (0-1)
modC: number (0-1)

// Video
isRendering: boolean
mediaRecorder: MediaRecorder
recordedChunks: Blob[]
```

---

## Extension Guide

### Adding New Orientations

1. Update orientation dropdown:
```html
<select id="orientation">
    <option value="landscape">Landscape</option>
    <option value="portrait">Portrait</option>
    <option value="square">Square</option>
    <option value="ultrawide">Ultrawide</option> <!-- NEW -->
</select>
```

2. Update `updateCanvasSize()`:
```javascript
case 'ultrawide':
    width = resolution * (21/9);
    height = resolution;
    break;
```

3. Update filename generation:
```javascript
const suffix = orientation === 'ultrawide' ? 'U' : ...;
```

### Adding New Resolutions

```html
<select id="resolution">
    <option value="720">720p</option>
    <option value="1080">1080p</option>
    <option value="1440">1440p</option>
    <option value="2160">2160p</option>
    <option value="4320">8K</option> <!-- NEW -->
</select>
```

### Adding New Waveforms

1. Add to dropdown:
```html
<select id="modAWave">
    <!-- existing options -->
    <option value="pulse">Pulse</option> <!-- NEW -->
</select>
```

2. Add case to `calculateMod()`:
```javascript
case 'pulse':
    return phase < 0.1 ? 1 : 0;  // 10% duty cycle
```

### Adding New Controls

1. Add HTML control
2. Create input handler
3. Use value in render loop or modulation
4. Add tooltip for clarity

---

## Performance Notes

### Canvas Size Impact (V2_1)

**Render Performance:**
- 1920x1080 canvas has minimal impact on modern GPUs
- GPU rendering scales well with resolution
- Main bottleneck is typically shader complexity, not canvas size

**Preview Benefits:**
- 1:1 match with 1080p export resolution
- No surprises when rendering
- Better visual feedback during development

### Optimization Tips

**Shader Code:**
- Avoid expensive operations in tight loops
- Use built-in functions (sin, cos, etc.)
- Minimize texture lookups
- Use simpler math where possible

**Audio Analysis:**
- FFT size: 256 is a good balance (fast, adequate resolution)
- Smoothing factor: 0.7 prevents jitter while remaining responsive
- Only analyze when audio is playing

**Waveform:**
- Static waveform cached as ImageData
- Only overlays redraw each frame
- Retina support with 2x rendering

**Video Export:**
- Use `canvas.captureStream()` without FPS parameter
- Captures every rendered frame reliably
- No browser throttling issues

---

## Code Metrics

**Total Lines:** ~2,680  
**HTML/CSS:** ~900 lines  
**JavaScript:** ~1,780 lines

**Function Count:** ~35 major functions  
**Global Variables:** ~25

**Performance:**
- Initialization: <100ms
- Shader compile: <50ms
- Render frame: ~16ms @ 60fps
- FFT analysis: ~1ms per frame

---

**Last Updated:** November 2025  
**Version:** V2_1  
**Status:** Production Release

---

*For user workflows, see UserGuide_All.md*  
*For version history, see B_Complete_Changelog.md*  
*For release overview, see D_Release_Summary.md*
