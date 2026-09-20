# OMS DOJO - DEVELOPER DOCUMENTATION V2_7

**Version:** V2_7  
**Release Date:** November 2025  
**Architecture:** Single-file HTML application  
**Technologies:** WebGPU, WGSL, Web Audio API, LocalStorage

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [WebGPU Rendering Pipeline](#webgpu-rendering-pipeline)
3. [Audio Analysis System](#audio-analysis-system)
4. [Frequency Mapping System](#frequency-mapping-system)
5. [Control States System](#control-states-system)
6. [Module Separation](#module-separation)
7. [Code Structure](#code-structure)
8. [API Reference](#api-reference)
9. [Development Guidelines](#development-guidelines)

---

## ARCHITECTURE OVERVIEW

### Single-File Philosophy

ONEMANSHYO Dojo is a completely self-contained HTML application with no external dependencies. Everything runs client-side in the browser.

**File Structure:**
```
OMS_Dojo_V2_7.html (~215KB)
├── HTML Structure (UI layout)
├── CSS Styling (embedded <style>)
└── JavaScript Logic (embedded <script>)
    ├── WebGPU rendering pipeline
    ├── Audio analysis (Web Audio API)
    ├── Frequency mapping system
    ├── Control States system
    ├── Preset management
    ├── Video export (MediaRecorder)
    └── LocalStorage persistence
```

**Why Single File:**
- Zero installation friction
- Portable across systems
- Offline capability
- No build process
- No dependency hell
- Easy distribution

### Technology Stack

**WebGPU Rendering:**
- Fragment shader execution (WGSL)
- Real-time 60fps canvas updates
- Uniform buffer for shader parameters
- Device/queue/pipeline management

**Audio Analysis:**
- Web Audio API
- AnalyserNode with FFT
- 32768 sample FFT size
- Frequency domain analysis

**Storage:**
- LocalStorage for presets
- Two namespaces: `shaderPresets` and `controlStates`
- JSON serialization

**Export:**
- MediaRecorder API (WebM/VP8)
- Canvas stream capture
- Multiple resolutions/framerates

---

## WEBGPU RENDERING PIPELINE

### Initialization Flow

```javascript
// 1. Get WebGPU adapter and device
const adapter = await navigator.gpu.requestAdapter();
const device = await adapter.requestDevice();

// 2. Configure canvas context
const context = canvas.getContext('webgpu');
const format = navigator.gpu.getPreferredCanvasFormat();
context.configure({
    device: device,
    format: format,
    alphaMode: 'premultiplied'
});

// 3. Create uniform buffer (48 bytes)
uniformBuffer = device.createBuffer({
    size: 48,
    usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});

// 4. Compile shader and create pipeline
// (see shader compilation section)

// 5. Start render loop
requestAnimationFrame(render);
```

### Uniform Buffer Structure (CRITICAL)

**48 bytes total - 16-byte alignment required:**

```javascript
const uniformData = new Float32Array([
    currentTime,      // 0: f32 (4 bytes)
    currentTempo,     // 1: f32 (4 bytes)
    modA,             // 2: f32 (4 bytes)
    modB,             // 3: f32 (4 bytes)
    modC,             // 4: f32 (4 bytes)
    0,                // 5: padding1 (4 bytes)
    canvas.width,     // 6: resolutionX (4 bytes)
    canvas.height,    // 7: resolutionY (4 bytes)
    0, 0, 0, 0        // 8-11: padding2-5 (16 bytes)
]);
// Total: 12 floats × 4 bytes = 48 bytes
```

**Why 48 bytes:**
- WebGPU requires 16-byte alignment for uniform buffers
- Resolution must be separate f32 values (not vec2)
- Padding ensures proper alignment

### WGSL Shader Structure

**Uniform Binding:**
```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    padding1: f32,
    resolutionX: f32,  // NOT vec2!
    resolutionY: f32,
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;
```

**Fragment Shader Entry:**
```wgsl
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Reconstruct resolution
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = fragCoord.xy / resolution;
    
    // Access modulator values
    let bass = uniforms.modA;
    let mid = uniforms.modB;
    let treble = uniforms.modC;
    
    // Time-based animation
    let t = uniforms.time;
    let bpm = uniforms.tempo;
    
    // Your shader code here
    return vec4<f32>(uv.x, uv.y, 0.0, 1.0);
}
```

### Shader Compilation

```javascript
async function compileShader(shaderCode) {
    try {
        // Create shader module
        const shaderModule = device.createShaderModule({
            code: shaderCode
        });
        
        // Create render pipeline
        pipeline = device.createRenderPipeline({
            layout: 'auto',
            vertex: {
                module: shaderModule,
                entryPoint: 'vertexMain'
            },
            fragment: {
                module: shaderModule,
                entryPoint: 'main',
                targets: [{
                    format: navigator.gpu.getPreferredCanvasFormat()
                }]
            },
            primitive: {
                topology: 'triangle-strip'
            }
        });
        
        // Create bind group for uniforms
        bindGroup = device.createBindGroup({
            layout: pipeline.getBindGroupLayout(0),
            entries: [{
                binding: 0,
                resource: {
                    buffer: uniformBuffer
                }
            }]
        });
        
        return true;
    } catch (error) {
        console.error('Shader compilation failed:', error);
        return false;
    }
}
```

### Render Loop

```javascript
function render() {
    // Update time
    const elapsed = (Date.now() - startTime) / 1000.0;
    currentTime = (elapsed * currentTempo) / 60.0; // Convert to beats
    
    // Update modulator values from audio analysis
    updateModulators();
    
    // Update uniform buffer
    const uniformData = new Float32Array([
        currentTime,
        currentTempo,
        modAValue,
        modBValue,
        modCValue,
        0,
        canvas.width,
        canvas.height,
        0, 0, 0, 0
    ]);
    device.queue.writeBuffer(uniformBuffer, 0, uniformData);
    
    // Encode render pass
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
    renderPass.draw(4, 1, 0, 0); // Full-screen quad
    renderPass.end();
    
    device.queue.submit([commandEncoder.finish()]);
    
    requestAnimationFrame(render);
}
```

---

## AUDIO ANALYSIS SYSTEM

### Web Audio API Setup

```javascript
// Initialize audio context and analyser
audioContext = new (window.AudioContext || window.webkitAudioContext)();
audioAnalyser = audioContext.createAnalyser();
audioAnalyser.fftSize = 32768;
audioAnalyser.smoothingTimeConstant = 0.8;

// Connect audio element
const source = audioContext.createMediaElementSource(audioElement);
source.connect(audioAnalyser);
audioAnalyser.connect(audioContext.destination);
```

### FFT Configuration

**V2_7 Settings:**
- FFT Size: 32768 samples
- Frequency Bin Count: 16384 bins
- Sample Rate: 48000 Hz (detected dynamically)
- Nyquist Frequency: 24000 Hz
- Bin Width: ~1.46 Hz/bin

**Calculation:**
```javascript
const binWidth = audioContext.sampleRate / audioAnalyser.fftSize;
// 48000 / 32768 = ~1.46 Hz/bin
```

### Frequency Data Retrieval

```javascript
function updateModulators() {
    if (!audioAnalyser || !audioElement || audioElement.paused) return;
    
    const freqData = new Uint8Array(audioAnalyser.frequencyBinCount);
    audioAnalyser.getByteFrequencyData(freqData);
    
    // freqData[i] = amplitude (0-255) at frequency bin i
    // bin i = i * binWidth Hz
}
```

---

## FREQUENCY MAPPING SYSTEM

### User-Definable Frequency Ranges

**Purpose:** Allow users to target specific frequency ranges for each modulator.

**Default Ranges:**
- Mod A: 30-50 Hz (kick drums)
- Mod B: 50-75 Hz (sub-bass)
- Mod C: 100-500 Hz (lead bass)

### Hz-to-Bin Conversion

```javascript
// Get user-defined frequency range
const freq1 = parseFloat(document.getElementById('modAFreq1').value);
const freq2 = parseFloat(document.getElementById('modAFreq2').value);

// Calculate bin indices
const binWidth = audioContext.sampleRate / audioAnalyser.fftSize;
const startBin = Math.floor(freq1 / binWidth);
const endBin = Math.floor(freq2 / binWidth);

// Average amplitude over frequency range
let sum = 0;
let count = 0;
for (let i = startBin; i <= endBin && i < freqData.length; i++) {
    sum += freqData[i];
    count++;
}
const rawValue = count > 0 ? sum / count / 255 : 0; // Normalize to 0-1
```

### Smoothing and Gain

```javascript
// Exponential smoothing (reduces jitter)
modASmoothed = modASmoothed * 0.7 + rawValue * 0.3;

// Apply user gain
const gain = parseFloat(document.getElementById('modAGain').value);
modAValue = modASmoothed * gain;
```

### Frequency Validation

**Rules:**
1. Freq1 must be ≤ Freq2
2. Both must be in range 0-24000 Hz (Nyquist limit)
3. Push behavior: Freq1 up pushes Freq2 up, Freq2 down pushes Freq1 down

**Implementation:**
```javascript
function validateFrequency(modPrefix) {
    const freq1Field = document.getElementById(modPrefix + 'Freq1');
    const freq2Field = document.getElementById(modPrefix + 'Freq2');
    
    let freq1 = parseFloat(freq1Field.value);
    let freq2 = parseFloat(freq2Field.value);
    
    // Bounds validation
    freq1 = Math.max(0, Math.min(24000, freq1));
    freq2 = Math.max(0, Math.min(24000, freq2));
    
    // Push behavior
    if (freq1 > freq2) {
        if (document.activeElement === freq1Field) {
            freq2 = freq1; // Freq1 up pushes Freq2 up
        } else {
            freq1 = freq2; // Freq2 down pushes Freq1 down
        }
    }
    
    freq1Field.value = freq1;
    freq2Field.value = freq2;
}
```

### Disabled State (No Audio)

When no audio is loaded, frequency fields are disabled with cyan overlay:

```javascript
function updateFrequencyFieldsState(enabled) {
    const freqFields = [
        'modAFreq1', 'modAFreq2',
        'modBFreq1', 'modBFreq2',
        'modCFreq1', 'modCFreq2'
    ];
    
    freqFields.forEach(id => {
        const field = document.getElementById(id);
        field.disabled = !enabled;
        
        // Add/remove cyan overlay
        if (enabled) {
            field.style.background = '';
        } else {
            field.style.background = 'rgba(0, 255, 255, 0.1)';
        }
    });
}
```

---

## CONTROL STATES SYSTEM

### Purpose

Save and load complete render configurations (21 values) with custom names.

### Storage Schema

**LocalStorage Namespace:** `controlStates`

**Data Structure:**
```javascript
{
    "quicktest": {
        name: "Quick Test",
        tempo: 120,
        loopLength: 16,
        renderDuration: "00:30.000",
        modASubdiv: "1/4",
        modAWaveform: "sine",
        modAGain: 1.0,
        modAFreq1: 30,
        modAFreq2: 50,
        modBSubdiv: "1/4",
        modBWaveform: "sine",
        modBGain: 1.0,
        modBFreq1: 50,
        modBFreq2: 75,
        modCSubdiv: "1/4",
        modCWaveform: "sine",
        modCGain: 1.0,
        modCFreq1: 100,
        modCFreq2: 500,
        orientation: "portrait",
        resolution: "720p",
        fps: 24
    },
    "user_preset_123": {
        // User-created preset
    }
}
```

### Library States (Hardcoded)

**5 Presets:**

1. **Quick Test:** 120bpm, Portrait 720p 24fps, 30sec
2. **HD Render:** 120bpm, Landscape 1080p 60fps, 1min
3. **4K Final:** 120bpm, Landscape 2160p 60fps, 30sec
4. **Instagram Story:** 120bpm, Portrait 1080p 30fps, 15sec
5. **TikTok:** 120bpm, Portrait 1080p 60fps, 30sec

**Implementation:**
```javascript
const libraryStates = {
    quicktest: {
        name: "Quick Test",
        tempo: 120,
        loopLength: 16,
        renderDuration: "00:30.000",
        // ... all 21 values
    },
    // ... other 4 library states
};
```

### Save Control State

```javascript
function saveControlState(name) {
    const stateId = name.toLowerCase().replace(/\s+/g, '_');
    
    const state = {
        name: name,
        tempo: parseFloat(document.getElementById('tempo').value),
        loopLength: parseInt(document.getElementById('loopLength').value),
        renderDuration: document.getElementById('renderDuration').value,
        
        modASubdiv: document.getElementById('modASubdiv').value,
        modAWaveform: document.getElementById('modAWaveform').value,
        modAGain: parseFloat(document.getElementById('modAGain').value),
        modAFreq1: parseFloat(document.getElementById('modAFreq1').value),
        modAFreq2: parseFloat(document.getElementById('modAFreq2').value),
        
        // ... repeat for Mod B and Mod C
        
        orientation: document.getElementById('orientation').value,
        resolution: document.getElementById('resolution').value,
        fps: parseInt(document.getElementById('fps').value)
    };
    
    // Get existing states
    let states = {};
    try {
        const stored = localStorage.getItem('controlStates');
        if (stored) states = JSON.parse(stored);
    } catch (e) {}
    
    // Save new state
    states[stateId] = state;
    localStorage.setItem('controlStates', JSON.stringify(states));
}
```

### Load Control State

```javascript
function applyControlState(stateId) {
    // Get state (library or user)
    let state;
    if (libraryStates[stateId]) {
        state = libraryStates[stateId];
    } else {
        const stored = localStorage.getItem('controlStates');
        if (stored) {
            const states = JSON.parse(stored);
            state = states[stateId];
        }
    }
    
    if (!state) return;
    
    // Apply all 21 values
    document.getElementById('tempo').value = state.tempo;
    document.getElementById('loopLength').value = state.loopLength;
    document.getElementById('renderDuration').value = state.renderDuration;
    
    document.getElementById('modASubdiv').value = state.modASubdiv;
    document.getElementById('modAWaveform').value = state.modAWaveform;
    document.getElementById('modAGain').value = state.modAGain;
    document.getElementById('modAFreq1').value = state.modAFreq1;
    document.getElementById('modAFreq2').value = state.modAFreq2;
    
    // ... repeat for Mod B and Mod C
    
    document.getElementById('orientation').value = state.orientation;
    document.getElementById('resolution').value = state.resolution;
    document.getElementById('fps').value = state.fps;
    
    // Update info box after DOM updates
    setTimeout(updateInfoBox, 0);
}
```

### Custom State Tracking

**Auto-switch to Custom when any field changes:**

```javascript
function attachChangeListeners() {
    const fields = [
        'tempo', 'loopLength', 'renderDuration',
        'modASubdiv', 'modAWaveform', 'modAGain', 'modAFreq1', 'modAFreq2',
        'modBSubdiv', 'modBWaveform', 'modBGain', 'modBFreq1', 'modBFreq2',
        'modCSubdiv', 'modCWaveform', 'modCGain', 'modCFreq1', 'modCFreq2',
        'orientation', 'resolution', 'fps'
    ];
    
    fields.forEach(id => {
        const field = document.getElementById(id);
        field.addEventListener('input', () => {
            document.getElementById('controlStateSelect').value = 'custom';
            updateInfoBox();
        });
    });
}
```

### Info Box Display

**Shows current state configuration:**

```javascript
function updateInfoBox() {
    const state = getCurrentControlState();
    const infoBox = document.getElementById('controlStateInfo');
    
    infoBox.innerHTML = `
        <div><strong>${state.name}</strong></div>
        <div>Tempo: ${state.tempo} BPM | Loop: ${state.loopLength} bars | Duration: ${state.renderDuration}</div>
        <div>Mod A: ${state.modASubdiv} ${state.modAWaveform} × ${state.modAGain} | ${state.modAFreq1}-${state.modAFreq2} Hz</div>
        <div>Mod B: ${state.modBSubdiv} ${state.modBWaveform} × ${state.modBGain} | ${state.modBFreq1}-${state.modBFreq2} Hz</div>
        <div>Mod C: ${state.modCSubdiv} ${state.modCWaveform} × ${state.modCGain} | ${state.modCFreq1}-${state.modCFreq2} Hz</div>
        <div>Output: ${state.orientation} ${state.resolution} @ ${state.fps}fps</div>
    `;
}
```

---

## MODULE SEPARATION

### Shader Module vs Audio Module

**CRITICAL:** These are independent systems with limited integration.

### Shader Module (Visual Timing)

**Location:** Controls tab, Rows 1-5

**Controls:**
- Shader Tempo (BPM) - controls visual animation speed
- Loop Length (bars) - visual loop cycle
- Render Duration (MM:SS.mmm) - video length
- Modulators A/B/C with subdivision, waveform, gain

**Purpose:** Controls shader visual timing independent of audio

**Time Calculation:**
```javascript
// Visual time in beats
currentTime = (elapsedSeconds * shaderTempo) / 60.0;
```

### Audio Module (Musical Timing)

**Location:** Audio controls strip below waveform

**Controls:**
- Audio Tempo (BPM) - controls beat positioning in audio
- Loop In/Out (bar.beat) - audio loop range
- Audio Start/End (MM:SS.mmm) - audio file bounds

**Purpose:** Controls audio playback and beat positioning

**Beat Calculation:**
```javascript
// Current beat position in audio
const audioBeatsPerSecond = audioTempo / 60.0;
const currentBeat = (audioElement.currentTime - audioStartTime) * audioBeatsPerSecond;
```

### Integration Points (Only Two)

**1. Loop Brackets → Render Duration**

When audio loaded, Loop In/Out defines render time range:

```javascript
if (audioElement && !audioElement.paused) {
    const inBeat = barBeatToBeats(document.getElementById('inBeat').value);
    const outBeat = barBeatToBeats(document.getElementById('outBeat').value);
    const beatCount = outBeat - inBeat;
    duration = (beatCount * 60) / audioTempo; // Use Audio Module tempo!
}
```

**2. FFT → Modulators**

Audio frequency analysis feeds shader modulators:

```javascript
// Audio Module provides frequency data
audioAnalyser.getByteFrequencyData(freqData);

// Shader Module consumes modulator values
modAValue = calculateModulator(freqData, modAFreq1, modAFreq2, modAGain);
```

---

## CODE STRUCTURE

### File Organization

**Lines 1-35:** Header comments (version, license, conventions)

**Lines 36-500:** HTML Structure
- Tab navigation
- Controls panel (400px fixed width)
- Canvas area (responsive)
- Status bar

**Lines 501-1200:** CSS Styling
- Layout (flexbox)
- Tab system
- Control styling
- Responsive breakpoints

**Lines 1201-5000:** JavaScript Logic
- WebGPU initialization
- Audio system setup
- Frequency mapping
- Control States system
- Preset management
- Video export
- Event handlers

### Key Functions

**WebGPU:**
- `initWebGPU()` - Initialize adapter/device/context
- `compileShader(code)` - Compile WGSL and create pipeline
- `render()` - Main render loop

**Audio:**
- `handleAudioUpload(file)` - Load audio file
- `updateModulators()` - FFT analysis and modulator calculation
- `validateFrequency(modPrefix)` - Frequency field validation

**Control States:**
- `loadControlStatesDropdown()` - Populate dropdown with library + user states
- `saveControlState(name)` - Save current config to localStorage
- `applyControlState(stateId)` - Load state and update all 21 fields
- `deleteControlState(stateId)` - Remove user state
- `updateInfoBox()` - Display current state info

**Presets:**
- `loadPresetLibrary()` - Display library shader cards
- `loadUserPresets()` - Display user shader cards
- `loadPreset(code, name)` - Load shader into Code tab
- `saveCurrentPreset(name)` - Save shader to localStorage
- `deletePreset(name)` - Remove user preset

**Export:**
- `startRender()` - Begin video capture
- `stopRender()` - Finalize and download video

---

## API REFERENCE

### Public Functions

**Shader Compilation:**
```javascript
async compileShader(shaderCode: string): Promise<boolean>
```

**Audio Management:**
```javascript
handleAudioUpload(file: File): void
removeAudio(): void
```

**Control States:**
```javascript
saveControlState(name: string): void
applyControlState(stateId: string): void
deleteControlState(stateId: string): void
getCurrentControlState(): Object
```

**Preset Management:**
```javascript
loadPreset(code: string, name: string): void
saveCurrentPreset(name: string): void
deletePreset(name: string): void
```

**Video Export:**
```javascript
startRender(): void
stopRender(): void
```

### Data Structures

**Control State Object:**
```typescript
interface ControlState {
    name: string;
    tempo: number;
    loopLength: number;
    renderDuration: string;
    modASubdiv: string;
    modAWaveform: string;
    modAGain: number;
    modAFreq1: number;
    modAFreq2: number;
    modBSubdiv: string;
    modBWaveform: string;
    modBGain: number;
    modBFreq1: number;
    modBFreq2: number;
    modCSubdiv: string;
    modCWaveform: string;
    modCGain: number;
    modCFreq1: number;
    modCFreq2: number;
    orientation: string;
    resolution: string;
    fps: number;
}
```

**Shader Preset Object:**
```typescript
interface ShaderPreset {
    name: string;
    code: string;
    thumbnail?: string; // Base64 PNG
}
```

---

## DEVELOPMENT GUIDELINES

### Incremental Development

**One Task Per Iteration:**
- Letter suffixes (A, B, C... Z, AA, AB...)
- Each iteration = one feature or bugfix
- Test before moving to next

**Version String Updates:**
- Line 2: HTML comment
- Title tag
- Version display in UI
- Consistent across all locations

### File Naming Convention

**Build Files:**
```
OMS_Dojo_V2_X_Y_[Letter]_[DescriptiveName].html
```

**Notes Files:**
```
OMS_Dojo_V2_X_Y_[Letter]_[DescriptiveName]_Notes.txt
```

### Testing Checklist

**Before Release:**
- [ ] Shader compilation works
- [ ] Audio loading and analysis functional
- [ ] Frequency validation correct
- [ ] Control States save/load properly
- [ ] Presets system operational
- [ ] Video export successful (all resolutions/fps)
- [ ] No console errors
- [ ] UTF-8 encoding clean (no Unicode corruption)
- [ ] File size reasonable (<250KB)

### Common Pitfalls

**1. Uniform Buffer Alignment**
- MUST be 48 bytes
- Resolution = separate f32 values, NOT vec2
- Padding required for alignment

**2. Module Confusion**
- Shader Tempo ≠ Audio Tempo
- Separate systems, different purposes
- Only 2 integration points

**3. Disabled Field Events**
- Attach listeners when fields enabled (audio load)
- Browser doesn't fire events on disabled fields reliably

**4. DOM Timing**
- Use setTimeout(fn, 0) for post-DOM updates
- Ensures fields populated before reading

**5. Zero Values**
- parseFloat() only, no "|| 0" fallbacks
- Zero is valid (disable modulator)

---

## PERFORMANCE CONSIDERATIONS

### FFT Size Impact

**32768 samples:**
- High precision (~1.46 Hz/bin)
- Handles well on modern hardware (M1 Mac)
- May cause frame drops on older systems

**Alternative (8192 samples):**
- Lower precision (~5.86 Hz/bin)
- Better performance
- Still usable for most bass music

**Recommendation:** Keep 32768 default, document 8192 alternative

### LocalStorage Limits

- Typical limit: 5-10MB per domain
- Current usage: <100KB (presets + states)
- No practical limit concerns

### Canvas Resolution

**Export resolutions:**
- 720p: Fast, good for testing
- 1080p: Standard, recommended
- 1440p: High quality, slower
- 2160p: Maximum quality, slowest

**Real-time preview:** Always 1080p (performance/quality balance)

---

## BROWSER COMPATIBILITY

### WebGPU Requirements

**Supported:**
- Safari Technology Preview 15+ (recommended)
- Chrome Canary with flag enabled
- Edge Canary with flag enabled

**Unsupported:**
- Firefox stable (WebGPU in development)
- Mobile browsers (WebGPU not yet available)

### Feature Detection

```javascript
if (!navigator.gpu) {
    alert('WebGPU not supported. Use Safari Technology Preview or Chrome Canary.');
    return;
}
```

---

## FUTURE DEVELOPMENT

### Potential Enhancements

**Real-Time Integration:**
- Native Mac app with Syphon output
- Ableton Link sync for live performance
- Eliminate pre-rendered video workflow

**Additional Features:**
- MIDI controller mapping
- More modulator waveforms
- Preset categories/tagging
- Cloud preset sharing

**Performance:**
- WebAssembly for FFT processing
- GPU-based audio analysis
- Multi-threaded rendering

---

**END OF DEVELOPER DOCUMENTATION**