# OMS SHADER BAKER - DEVELOPER DOCUMENTATION V3.4.0

**Version:** V3.4.0 Production Release  
**Date:** November 15, 2025  
**License:** GPL-3.0  
**Repository:** Single-file HTML architecture

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Development Philosophy](#development-philosophy)
3. [Code Structure](#code-structure)
4. [Layout System](#layout-system)
5. [Shader System](#shader-system)
6. [Audio System](#audio-system)
7. [Performance Grid](#performance-grid)
8. [File Formats](#file-formats)
9. [Development Guidelines](#development-guidelines)
10. [Testing Protocols](#testing-protocols)

---

## ARCHITECTURE OVERVIEW

### Single-File Philosophy

OMS Shader Baker uses a single HTML file architecture (~445KB, 10,207 lines) with NO external dependencies.

**Benefits:**
- Complete portability (works offline)
- No build process or compilation
- Easy version control (one file to track)
- No dependency hell
- Fast loading (< 1 second)
- Works in any WebGPU-enabled browser

**Components Within Single File:**
- HTML structure (~2,000 lines)
- CSS styling (~1,500 lines)
- JavaScript logic (~6,700 lines)
- WGSL shader templates (embedded)
- Default shader presets (embedded)

### Technology Stack

**Core Technologies:**
- **WebGPU:** GPU-accelerated shader rendering (WGSL shaders)
- **Web Audio API:** Multi-track audio, FFT analysis, tempo scaling
- **File System Access API:** Application folder management (.oms, .omsp files)
- **Canvas API:** Waveform visualization, rotary knob drawing, meters
- **Local Storage:** Settings persistence, container handle storage
- **Flexbox/Grid:** Responsive layout system

**No External Libraries:**
All functionality implemented in vanilla JavaScript. No frameworks, no bundlers, no npm packages.

---

## DEVELOPMENT PHILOSOPHY

### Core Principles

1. **Shader-First Design**
   - Audio is optional enhancement, not requirement
   - Every feature must work without audio loaded
   - Shader creation is primary workflow

2. **Professional Standards**
   - Sample-accurate timing (Web Audio API)
   - 60 FPS rendering (requestAnimationFrame)
   - Professional DJ software paradigms (Serato, Ableton)
   - Hardware mixer aesthetic

3. **One Man Show Philosophy**
   - Solo creators can achieve professional results
   - No teams required
   - No expensive software required
   - Open source, free forever (GPL-3.0)

4. **Surgical Development**
   - One feature per iteration
   - Alphabetical iteration naming (A, B, C... Z, AA, AB...)
   - Never skip iteration letters
   - Comprehensive notes per iteration

### Development Protocols

**Before Implementation:**
1. User describes feature/issue
2. Claude investigates and discusses
3. User may send multiple messages (wait for full context)
4. Claude asks: "Ready to build? yes go?"
5. User responds: "yes go"
6. THEN Claude implements

**Critical Rules:**
- NEVER build without explicit "yes go"
- ALWAYS use underscores in versions (V3_4_0_A not V3.4.0_A)
- NEVER include version history in HTML file
- ALWAYS update 3 places: HTML comment, title tag, UI header

---

## CODE STRUCTURE

### File Organization

```
Lines 1-20:    HTML header, meta, title
Lines 21-1550: CSS (all styles)
Lines 1551-2100: HTML structure (UI layout)
Lines 2101-4000: Tab content (Controls, Presets, Library, Code, Settings, LIVE)
Lines 4001-10200: JavaScript (all logic)
```

### JavaScript Architecture

**Global State Objects:**
```javascript
// Control states
const controlStates = {};

// Grid state (LIVE tab)
const gridState = {
    masterTempo: 120,
    zoomValue: 68,
    isPlaying: false,
    cells: [/* 4 tracks × 8 scenes = 32 cells */],
    tracks: [/* 4 track audio contexts */]
};

// Track container data (audio timing)
const trackContainerData = [null, null, null, null]; // 4 tracks

// Application folder handle (File System Access API)
let applicationFolderHandle = null;
```

**Module Pattern:**
Functions organized by concern:
- Shader rendering: `renderShader()`, `initWebGPU()`, `compileShader()`
- Audio system: `loadAudioFile()`, `analyzeAudio()`, `setupAudioContext()`
- Controls: `updateControl()`, `drawRotaryKnob()`, `saveControlState()`
- File I/O: `saveOmsContainer()`, `loadOmsContainer()`, `saveProjectState()`
- LIVE grid: `renderCell()`, `drawWaveformFromAudioBuffer()`, `syncWaveforms()`

### CSS Organization

**Structure:**
```css
/* Base styles (1-50) */
* { margin: 0; padding: 0; box-sizing: border-box; }

/* Layout (51-200) */
.app-wrapper, .left-panel, .preview-panel, .tab-bar, .tab-content

/* Controls (201-800) */
.controls-section, .control-row, .control-group, .rotary-knob

/* LIVE tab (801-1400) */
.preview-panel, .grid-headers, .scenes-grid, .mixer-section, .waveforms-section

/* Waveform controls (1401-1500) */
.track-waveform-control, .track-control-btn

/* Theme colors (throughout) */
--primary: #00d4ff (cyan)
--bg-dark: #0c0c0c
--bg-panel: #1a1a1a
--border: #444
```

---

## LAYOUT SYSTEM

### V3.4.0 Layout Architecture

**Total Width:** 1320px (fits 1352px viewport with 32px breathing room)

```
Viewport: 1352px
├── Gap: 6px
├── Border left: 10px (grey #666)
├── Content: 1320px
│   ├── Left panel: 400px
│   │   ├── Tab bar: 40px height (fixed)
│   │   ├── Tab content: flex: 1 (scrollable)
│   │   └── Shader output: flex: 0 0 auto (fixed at bottom)
│   └── Preview panel: 920px
│       ├── Padding left: 1px
│       ├── Grid content: 918px
│       │   ├── Scenes: 153px
│       │   ├── Track 1: 153px
│       │   ├── Track 2: 153px
│       │   ├── Track 3: 153px
│       │   ├── Track 4: 153px
│       │   └── Master: 153px
│       └── Padding right: 4px
├── Border right: 10px (grey #666)
└── Gap: 6px
```

**MacBook Pro 14" Optimization:**
- Logical resolution: 1512×982
- Effective viewport: ~1352px width
- All 6 mixer columns visible
- Perfect fit at default scaling

### Flexbox Patterns

**Left Panel Hierarchy:**
```html
<div class="left-panel"> <!-- flex column -->
    <div class="tab-bar"></div> <!-- flex: 0 0 auto (fixed) -->
    <div class="tab-content"></div> <!-- flex: 1, overflow-y: auto (scrollable) -->
    <div id="liveShaderOutputContainer"></div> <!-- flex: 0 0 auto (fixed) -->
</div>
```

**Critical CSS Property:**
```css
.controls {
    flex: 1;
    min-height: 0; /* Allows flex item to shrink below content size */
    overflow-y: auto; /* Enables nested scrolling */
}
```

Without `min-height: 0`, nested scrolling doesn't work in flex containers.

**Library Tab with Shader Output:**
```html
<div class="controls-section"> <!-- flex column -->
    <div class="controls"> <!-- flex: 1, min-height: 0 -->
        <div class="library-list"></div> <!-- flex: 1, overflow-y: auto → SCROLLS -->
    </div>
    <div id="liveShaderOutputContainer"></div> <!-- flex-shrink: 0 → FIXED -->
</div>
```

### Grid System (LIVE Tab)

**Grid Template Columns:**
```css
.grid-headers,
.scenes-grid,
.mixer-section {
    grid-template-columns: 153px 153px 153px 153px 153px 153px;
    /* Scenes + Track1 + Track2 + Track3 + Track4 + Master */
}
```

**Equal Column Widths = Consistent Flex Behavior**

V3.4.0 uses uniform 153px columns (6 × 153px = 918px in 920px container).
This ensures consistent flexbox behavior across all mixer controls.

---

## SHADER SYSTEM

### WebGPU Architecture

**Initialization:**
```javascript
async function initWebGPU() {
    if (!navigator.gpu) {
        throw new Error("WebGPU not supported");
    }
    const adapter = await navigator.gpu.requestAdapter();
    const device = await adapter.requestDevice();
    const canvas = document.getElementById('shaderCanvas');
    const context = canvas.getContext('webgpu');
    const format = navigator.gpu.getPreferredCanvasFormat();
    context.configure({ device, format });
    return { device, context, format };
}
```

**Shader Compilation:**
```javascript
function compileShader(device, shaderCode, format) {
    const shaderModule = device.createShaderModule({ code: shaderCode });
    const pipeline = device.createRenderPipeline({
        layout: 'auto',
        vertex: { module: shaderModule, entryPoint: 'vs_main' },
        fragment: {
            module: shaderModule,
            entryPoint: 'fs_main',
            targets: [{ format }]
        },
        primitive: { topology: 'triangle-strip' }
    });
    return pipeline;
}
```

**Render Loop:**
```javascript
function renderShader() {
    if (!isWebGPUReady) return;
    
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
    renderPass.draw(4);
    renderPass.end();
    
    device.queue.submit([commandEncoder.finish()]);
    requestAnimationFrame(renderShader);
}
```

### WGSL Shader Structure

**Standard Template:**
```wgsl
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32; // Bass
@group(0) @binding(3) var<uniform> uModB: f32; // Mids
@group(0) @binding(4) var<uniform> uModC: f32; // Treble

@vertex
fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    // Full-screen quad
    var pos = array<vec2<f32>, 4>(
        vec2(-1.0, -1.0), vec2(1.0, -1.0),
        vec2(-1.0, 1.0), vec2(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

@fragment
fn fs_main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = fragCoord.xy / uResolution;
    // Shader logic here
    return vec4<f32>(color.rgb, 1.0);
}
```

### Modulator System

**Three-Modulator Architecture:**
- **modA:** Bass frequencies (30-500Hz default, user-definable)
- **modB:** Mid frequencies (500-2000Hz default, user-definable)
- **modC:** Treble frequencies (2000-20000Hz default, user-definable)

**FFT Analysis:**
```javascript
function analyzeAudio(audioBuffer) {
    const fftSize = 32768; // High resolution
    const analyser = audioContext.createAnalyser();
    analyser.fftSize = fftSize;
    const frequencyData = new Uint8Array(analyser.frequencyBinCount);
    analyser.getByteFrequencyData(frequencyData);
    
    // Calculate modulator values from frequency ranges
    const modA = getAverageFrequency(frequencyData, 30, 500);
    const modB = getAverageFrequency(frequencyData, 500, 2000);
    const modC = getAverageFrequency(frequencyData, 2000, 20000);
    
    return { modA, modB, modC };
}
```

---

## AUDIO SYSTEM

### Web Audio API Architecture

**Audio Context per Track:**
```javascript
const trackContexts = [
    null, // Track 1
    null, // Track 2
    null, // Track 3
    null  // Track 4
];

function loadAudioForTrack(trackIndex, audioBuffer) {
    const context = new AudioContext();
    const source = context.createBufferSource();
    source.buffer = audioBuffer;
    
    // Gain control
    const gainNode = context.createGain();
    gainNode.gain.value = 0.8; // -10dB default
    
    // Connect
    source.connect(gainNode);
    gainNode.connect(context.destination);
    
    trackContexts[trackIndex] = {
        context,
        source,
        gainNode,
        buffer: audioBuffer
    };
}
```

### Tempo Scaling (Time-Stretching)

**Master Tempo Control:**
```javascript
function setMasterTempo(newTempo) {
    gridState.masterTempo = newTempo;
    
    // Update playback rates for all tracks
    for (let i = 0; i < 4; i++) {
        if (trackContainerData[i]) {
            const trackBPM = trackContainerData[i].bpm;
            const playbackRate = newTempo / trackBPM;
            if (trackContexts[i]?.source) {
                trackContexts[i].source.playbackRate.value = playbackRate;
            }
        }
    }
    
    // Redraw waveforms at new tempo
    redrawAllWaveforms();
}
```

**Playback Rate Formula:**
```javascript
playbackRate = masterTempo / trackBPM
```

Examples:
- Track at 128 BPM, master at 120 BPM: `120 / 128 = 0.9375` (slower)
- Track at 124 BPM, master at 128 BPM: `128 / 124 = 1.032` (faster)

### Audio Timing Data Storage

**trackContainerData Structure:**
```javascript
trackContainerData[trackIndex] = {
    bpm: 128.0,              // Original track BPM
    startTime: 0.0,          // Audio start offset (seconds)
    endTime: 180.5,          // Audio end point (seconds)
    loopIn: 64.0,            // Loop in point (seconds)
    loopOut: 128.0,          // Loop out point (seconds)
    buffer: audioBuffer,     // Web Audio API buffer
    waveformData: [],        // Analyzed waveform samples
    beatGrid: []             // Beat positions (bars, beats)
};
```

---

## PERFORMANCE GRID

### Grid Architecture

**4 Tracks × 8 Scenes = 32 Cells**

```
     Scene1  Scene2  Scene3  Scene4  Scene5  Scene6  Scene7  Scene8
Track1 [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]
Track2 [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]
Track3 [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]
Track4 [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]    [  ]
```

**Cell Data Structure:**
```javascript
gridState.cells = [
    // Track 1 (8 cells)
    { container: null, audio: null, shader: null, controls: {} },
    // ... 7 more cells
    
    // Track 2 (8 cells)
    // ... 8 cells
    
    // Track 3, Track 4 (16 more cells)
];
```

### Beat Grid System (V3.3.0)

**The One Timeline Architecture:**

All tracks use a SINGLE master timeline for visual alignment:

```javascript
const secondsPerBar = (4 / gridState.masterTempo) * 60;
const pixelsPerSecond = gridState.zoomValue;
const position = time * pixelsPerSecond;
```

**Key Principle:** Don't use CSS transforms (scaleX) to fix timing. Render everything at master tempo from the start.

**Visual Elements:**
- Cyan bars: Every musical bar (4 beats)
- Magenta markers: Every 4 bars (phrase markers)
- Bar numbers: 1, 5, 9, 13... (aligned with magenta)
- Waveform bars: 1 per beat (beat-aligned)

### Waveform Rendering

**drawWaveformFromAudioBuffer():**
```javascript
function drawWaveformFromAudioBuffer(canvas, audioBuffer, trackData) {
    const ctx = canvas.getContext('2d');
    const width = canvas.width;
    const height = canvas.height;
    
    // Calculate scaled duration
    const trackBPM = trackData.bpm;
    const tempoRatio = gridState.masterTempo / trackBPM;
    const scaledDuration = audioBuffer.duration * tempoRatio;
    
    // Pixels per second (zoom)
    const pixelsPerSecond = gridState.zoomValue;
    const totalWidth = scaledDuration * pixelsPerSecond;
    
    // Sample the audio buffer
    const channelData = audioBuffer.getChannelData(0);
    const samplesPerPixel = Math.floor(channelData.length / totalWidth);
    
    // Draw waveform bars (1 per beat)
    const beatsPerSecond = gridState.masterTempo / 60;
    const pixelsPerBeat = pixelsPerSecond / beatsPerSecond;
    
    for (let x = 0; x < totalWidth; x += pixelsPerBeat) {
        const sampleIndex = Math.floor(x * samplesPerPixel);
        const amplitude = Math.abs(channelData[sampleIndex]);
        const barHeight = amplitude * height;
        
        ctx.fillStyle = '#00d4ff';
        ctx.fillRect(x, (height - barHeight) / 2, pixelsPerBeat - 2, barHeight);
    }
}
```

---

## FILE FORMATS

### .oms Container Format

**OMS = One Man Show Container**

JSON structure containing shader + audio + control states:

```json
{
    "version": "3.4.0",
    "shaderCode": "WGSL shader code here...",
    "controlStates": {
        "shaderModAFreqMin": 30,
        "shaderModAFreqMax": 500,
        "shaderModBFreqMin": 500,
        "shaderModBFreqMax": 2000,
        "shaderModCFreqMin": 2000,
        "shaderModCFreqMax": 20000,
        "audioFileName": "track.mp3",
        "audioBPM": 128.0,
        "audioPitch": 2.5,
        "audioStartTime": 0.0,
        "audioEndTime": 180.5,
        "audioLoopIn": 64.0,
        "audioLoopOut": 128.0
    },
    "audioData": "base64-encoded-audio-file (optional)",
    "timestamp": "2025-11-15T12:00:00Z",
    "metadata": {
        "name": "My Shader",
        "artist": "ONEMANSHYO",
        "tags": ["bass", "reactive", "geometric"]
    }
}
```

**Save/Load Functions:**
```javascript
async function saveOmsContainer(name, shaderCode, controlStates, audioFile) {
    const container = {
        version: "3.4.0",
        shaderCode,
        controlStates,
        timestamp: new Date().toISOString(),
        metadata: { name }
    };
    
    if (audioFile) {
        const arrayBuffer = await audioFile.arrayBuffer();
        const base64 = btoa(String.fromCharCode(...new Uint8Array(arrayBuffer)));
        container.audioData = base64;
    }
    
    const json = JSON.stringify(container, null, 2);
    const blob = new Blob([json], { type: 'application/json' });
    const fileName = `${name}.oms`;
    
    // Save to application folder
    await saveFileToApplicationFolder(fileName, blob);
}
```

### .omsp Project State Format

**OMSP = One Man Show Project**

JSON structure for LIVE tab grid state:

```json
{
    "version": "3.4.0",
    "masterTempo": 120,
    "tracks": [
        {
            "index": 0,
            "volume": 0.8,
            "mute": false,
            "solo": false,
            "scenes": [
                { "containerName": "bass_reactive.oms", "loopIn": 0, "loopOut": 64 },
                { "containerName": null },
                // ... 6 more scenes
            ]
        },
        // ... 3 more tracks
    ],
    "timestamp": "2025-11-15T12:00:00Z"
}
```

---

## DEVELOPMENT GUIDELINES

### Code Style

**JavaScript:**
- Use `const` and `let` (never `var`)
- Descriptive function names (verb + noun: `loadAudioFile`, `drawWaveform`)
- Camel case for variables: `trackIndex`, `audioBuffer`
- Clear comments for complex logic
- Avoid nested callbacks (use async/await)

**CSS:**
- BEM-like naming: `.track-control-btn`, `.mixer-layout`
- Logical grouping (layout, controls, LIVE tab)
- Consistent units (px for fixed, % for fluid)
- Comments for non-obvious values

**HTML:**
- Semantic structure (`<button>`, `<label>`, `<canvas>`)
- Descriptive IDs (`rotaryVolume1`, `waveformCanvas1`)
- Inline styles only when absolutely necessary (V3_4_0_Z: gap: 0)

### Performance Optimization

**Critical Paths:**
1. Shader compilation (use cached pipelines)
2. Waveform rendering (debounce on zoom/tempo changes)
3. Audio analysis (use Web Workers for FFT if possible)
4. Canvas operations (batch draw calls)

**60 FPS Rendering:**
```javascript
let lastFrameTime = 0;
const targetFrameTime = 1000 / 60; // 16.67ms

function renderFrame(timestamp) {
    if (timestamp - lastFrameTime < targetFrameTime) {
        requestAnimationFrame(renderFrame);
        return;
    }
    lastFrameTime = timestamp;
    
    // Render logic here
    renderShader();
    updateMeters();
    
    requestAnimationFrame(renderFrame);
}
```

### Memory Management

**Audio Buffers:**
- Decode once, reuse
- Store in trackContainerData
- Clean up on unload: `buffer = null`

**Canvas Contexts:**
- Reuse contexts (don't recreate)
- Clear before redrawing: `ctx.clearRect(0, 0, width, height)`

**WebGPU Resources:**
- Dispose pipelines on shader change: `pipeline.destroy()`
- Release texture views after use

---

## TESTING PROTOCOLS

### Manual Testing Checklist

**Layout (MacBook Pro 14"):**
- [ ] Open at 1352px viewport width
- [ ] All 6 mixer columns visible
- [ ] No horizontal scrollbar
- [ ] Grey borders visible on left/right
- [ ] Professional appearance

**Tab Navigation:**
- [ ] All 6 tabs switch correctly (C, P, L, W, S, LIVE)
- [ ] Single-letter labels correct
- [ ] Shader canvas only visible on LIVE tab
- [ ] Library scrolls independently of shader output
- [ ] No console errors

**Waveform Controls:**
- [ ] 70px wide containers aligned with SCENES column
- [ ] Sharp corners, no gaps
- [ ] Buttons flush against waveform canvas
- [ ] All buttons clickable and functional

**Zoom Knob:**
- [ ] ZOOM label aligned with M/S buttons
- [ ] Knob aligned with volume knobs
- [ ] Cyan arc + white line indicator
- [ ] Dragging adjusts zoom (10-200 px/sec)
- [ ] Waveforms redraw on zoom change

**Audio System:**
- [ ] Load audio file (MP3, WAV)
- [ ] FFT analysis generates modulators
- [ ] Playback at correct tempo
- [ ] Volume control works
- [ ] Mute/solo functional

**File I/O:**
- [ ] Save .oms container
- [ ] Load .oms container
- [ ] Save .omsp project state
- [ ] Load .omsp project state
- [ ] Application folder persists between sessions

### Automated Testing (Future)

Consider implementing:
- Unit tests for critical functions (audio analysis, tempo scaling)
- Integration tests for file I/O
- Visual regression tests for layout
- Performance benchmarks (60 FPS rendering, audio latency)

---

## CONTRIBUTING

### Pull Request Guidelines

1. **One Feature Per PR**
   - Clear, focused changes
   - Comprehensive testing
   - Updated documentation

2. **Code Quality**
   - Follow style guidelines
   - Add comments for complex logic
   - No console.log() in production code

3. **Version Naming**
   - Use alphabetical iterations (A, B, C...)
   - Never skip letters
   - Update all 3 version references

4. **Documentation**
   - Create iteration notes file
   - Update changelog
   - Update relevant guides

### Issue Reporting

**Bug Reports:**
- Browser and OS version
- Steps to reproduce
- Expected vs actual behavior
- Console errors (if any)
- Screenshots/videos (if applicable)

**Feature Requests:**
- Use case description
- Proposed implementation
- Impact on existing features
- Performance considerations

---

## ARCHITECTURE DECISIONS

### Why Single HTML File?

**Pros:**
- No build process
- No dependency management
- Complete portability
- Works offline
- Easy version control
- Fast deployment

**Cons:**
- Large file size (~445KB)
- Harder to navigate
- No module system
- Manual organization required

**Decision:** Benefits outweigh drawbacks for this project. The "ghost protocol" philosophy values simplicity and accessibility over enterprise architecture.

### Why WebGPU Over WebGL?

**Pros:**
- Modern API (closer to Metal, Vulkan, DirectX 12)
- Better performance
- Compute shaders (future)
- WGSL is cleaner than GLSL
- Future-proof

**Cons:**
- Limited browser support (Chrome 113+)
- Newer technology, less mature
- Some users can't run it

**Decision:** WebGPU is the future. Supporting legacy WebGL would double complexity. Users with older browsers can use V2.x versions.

### Why No TypeScript?

**Pros of TypeScript:**
- Type safety
- Better IDE support
- Catches errors at compile time

**Cons of TypeScript:**
- Requires build process
- Adds complexity
- Breaks single-file philosophy

**Decision:** Single-file philosophy is core. TypeScript would require compilation, defeating the purpose.

---

## COMMON PATTERNS

### Rotary Knob Drawing

**Standard Pattern:**
```javascript
function drawRotaryKnob(canvas, value, minValue, maxValue) {
    const ctx = canvas.getContext('2d');
    const centerX = 30, centerY = 30, radius = 28;
    
    // Background
    ctx.fillStyle = '#1a1a1a';
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, 0, Math.PI * 2);
    ctx.fill();
    
    // Border
    ctx.strokeStyle = '#444';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Normalize value
    const normalized = (value - minValue) / (maxValue - minValue);
    
    // Calculate angle
    const startAngle = (-150 * Math.PI / 180) + Math.PI / 2 + Math.PI;
    const endAngle = (150 * Math.PI / 180) + Math.PI / 2 + Math.PI;
    const currentAngle = startAngle + (endAngle - startAngle) * normalized;
    
    // Cyan arc
    ctx.strokeStyle = '#00d4ff';
    ctx.lineWidth = 3;
    ctx.beginPath();
    ctx.arc(centerX, centerY, 22, startAngle, currentAngle);
    ctx.stroke();
    
    // White indicator line
    ctx.strokeStyle = '#fff';
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(centerX, centerY);
    const lineX = centerX + Math.cos(currentAngle) * 18;
    const lineY = centerY + Math.sin(currentAngle) * 18;
    ctx.lineTo(lineX, lineY);
    ctx.stroke();
}
```

### Tab Switching

**Standard Pattern:**
```javascript
function switchTab(tabName) {
    // Hide all tab content
    document.getElementById('controlsTabContent').classList.remove('active');
    document.getElementById('presetsTabContent').classList.remove('active');
    document.getElementById('libraryTabContent').classList.remove('active');
    document.getElementById('codeTabContent').classList.remove('active');
    document.getElementById('settingsTabContent').classList.remove('active');
    
    // Deactivate all tab buttons
    document.getElementById('controlsTab').classList.remove('active');
    document.getElementById('presetsTab').classList.remove('active');
    document.getElementById('libraryTab').classList.remove('active');
    document.getElementById('codeTab').classList.remove('active');
    document.getElementById('settingsTab').classList.remove('active');
    
    // Show selected tab
    if (tabName === 'controls') {
        document.getElementById('controlsTabContent').classList.add('active');
        document.getElementById('controlsTab').classList.add('active');
        shaderCanvas.style.display = 'block';
        liveShaderOutputContainer.style.display = 'none';
    }
    // ... other tabs
}
```

### File System Access

**Standard Pattern:**
```javascript
async function setupApplicationFolder() {
    try {
        const dirHandle = await window.showDirectoryPicker({
            mode: 'readwrite',
            startIn: 'documents'
        });
        
        // Store handle in IndexedDB
        await storeApplicationFolderHandle(dirHandle);
        
        // Update UI
        document.getElementById('applicationFolderName').textContent = dirHandle.name;
        document.getElementById('applicationFolderInfo').style.display = 'block';
        
        // Scan for containers
        await scanApplicationFolder();
    } catch (err) {
        if (err.name !== 'AbortError') {
            console.error('Failed to setup application folder:', err);
        }
    }
}
```

---

## FUTURE DEVELOPMENT

### Planned Architecture Changes

**V3.5.0:**
- Scene launching system (grid cell triggering)
- Web MIDI API integration
- Hardware controller mapping
- Ableton Link synchronization

**V4.0.0:**
- Complete performance environment
- Recording/export from LIVE tab
- Advanced session management
- Plugin architecture for extensions?

### Scalability Considerations

**Current Limits:**
- 4 tracks (could expand to 8 or 16)
- 8 scenes (could expand to 16 or 32)
- Single shader per cell (could support multiple layers)
- No crossfading (coming in V3.5.0)

**Performance Targets:**
- 60 FPS rendering (currently achieved)
- < 10ms audio latency (Web Audio API limits)
- < 1 second file operations (File System Access API limits)

---

## RESOURCES

### WebGPU Documentation
- https://www.w3.org/TR/webgpu/
- https://developer.chrome.com/docs/web-platform/webgpu/

### Web Audio API
- https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- https://webaudio.github.io/web-audio-api/

### File System Access API
- https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API
- https://web.dev/file-system-access/

### WGSL Specification
- https://www.w3.org/TR/WGSL/

---

**END OF DEVELOPER DOCUMENTATION**

For user guides, see: OMS_ShaderBaker_V3_4_0_F_UserGuide_All.md  
For release notes, see: OMS_ShaderBaker_V3_4_0_D_Release_Notes.md
