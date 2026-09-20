# ONEMANSHYO Dojo V2_2 - Developer Documentation

**Version:** V2_2  
**Target Audience:** Developers, technical users, contributors  
**Purpose:** Technical architecture, APIs, and extension guide

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [File Structure](#file-structure)
- [Audio System](#audio-system)
- [BPM Detection System](#bpm-detection-system)
- [Nudge System](#nudge-system)
- [Modulation System](#modulation-system)
- [WebGPU Integration](#webgpu-integration)
- [Video Export System](#video-export-system)
- [API Reference](#api-reference)
- [Extension Guide](#extension-guide)
- [Performance Notes](#performance-notes)

---

## Architecture Overview

### Single-File Application

**Philosophy:**
- Everything in one HTML file (~3,185 lines, ~150KB)
- No external dependencies (browser APIs only)
- No build process required
- Easy to distribute and run

**Structure:**
```
OMS_Dojo_V2_2.html
├── HTML Structure (lines 1-~900)
│   ├── Meta tags & title
│   ├── CSS (<style> tag, ~500 lines)
│   └── DOM structure
└── JavaScript (lines ~900-3185)
    ├── WebGPU initialization
    ├── Audio system (including BPM detection)
    ├── Nudge system
    ├── Modulation engine
    ├── UI handlers
    └── Video export
```

### Technology Stack

**Core Technologies:**
- WebGPU (graphics rendering)
- WGSL (WebGPU Shading Language)
- Web Audio API (audio analysis + BPM detection)
- MediaRecorder API (video capture)
- Canvas 2D API (waveform visualization)

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
  <title>ONEMANSHYO Dojo V2_2</title>
  
  <style>
    /* Layout, controls, tabs, canvas styling */
    /* ~500 lines of CSS */
  </style>
</head>

<body>
  <!-- Full-width Header -->
  <div class="header">
    <h1>ONEMANSHYO Dojo</h1>
    <span class="version">[V2_2]</span>
  </div>
  
  <!-- Main Container -->
  <div class="container">
    <!-- Left Panel (400px width) -->
    <div class="left-panel">
      <!-- Tab Bar: Controls / Code -->
      <!-- Controls Tab: 6 rows -->
      <!-- Code Tab: Shader editor -->
    </div>
    
    <!-- Right Panel (flex: 1) -->
    <div class="preview-panel">
      <!-- Canvas (1920x1080) -->
      <!-- Waveform Section (1920x180) -->
      <!-- Audio Controls Strip (40px) -->
    </div>
  </div>
  
  <!-- Full-width Status Bar -->
  <span id="shaderStatus">WebGPU Ready</span>
  
  <script>
    /* All JavaScript (~2,285 lines) */
  </script>
</body>
</html>
```

---

## Audio System

### Audio Variables

```javascript
// Core audio objects
let audioContext = null;          // Web Audio API context
let audioElement = null;           // HTML5 Audio element
let audioSource = null;            // MediaElementSource
let analyser = null;               // AnalyserNode for FFT
let audioDataArray = null;         // Frequency data array
let audioLoaded = false;           // Load state

// Audio properties
let audioTempo = 120;              // Detected/user BPM (for positioning)
let currentTempo = 120;            // Shader visual tempo
let bassLevel = 0, midLevel = 0, trebleLevel = 0;  // Frequency levels
let loopEnabled = true;            // Loop mode toggle
let nudgeOffset = 0;               // Millisecond offset (converted to seconds)

// Audio buffer data
let audioBufferData = null;        // Decoded audio buffer for BPM analysis
```

### Audio Loading Workflow

```javascript
async function loadAudioFile(event) {
    // 1. Create Audio element
    audioElement = new Audio();
    audioElement.src = URL.createObjectURL(file);
    
    // 2. Initialize Web Audio API
    audioContext = new AudioContext();
    audioSource = audioContext.createMediaElementSource(audioElement);
    analyser = audioContext.createAnalyser();
    analyser.fftSize = 256;
    
    // 3. Connect audio graph
    audioSource.connect(analyser);
    analyser.connect(audioContext.destination);
    
    // 4. Load and decode for BPM detection
    const arrayBuffer = await file.arrayBuffer();
    audioBufferData = await audioContext.decodeAudioData(arrayBuffer);
    
    // 5. Detect BPM
    const detectedBPM = detectBPM(audioBufferData);
    audioTempo = detectedBPM;
    
    // 6. Generate waveform
    generateStaticWaveform();
    
    // 7. Jump to Loop In beat (with nudge offset)
    const inBeat = parseInt(document.getElementById('inBeat').value);
    const beatsPerSecond = audioTempo / 60.0;
    audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
    
    audioLoaded = true;
}
```

### Frequency Analysis

```javascript
function analyzeAudio() {
    if (!analyser || !audioDataArray) return { bass: 0, mid: 0, treble: 0 };
    
    analyser.getByteFrequencyData(audioDataArray);
    
    const fftSize = analyser.fftSize;
    const bufferLength = analyser.frequencyBinCount;
    
    // Split spectrum into bass/mid/treble
    const bassEnd = Math.floor(bufferLength * 0.1);
    const midEnd = Math.floor(bufferLength * 0.5);
    
    // Calculate average amplitude per band
    let bassSum = 0, midSum = 0, trebleSum = 0;
    
    for (let i = 0; i < bassEnd; i++) bassSum += audioDataArray[i];
    for (let i = bassEnd; i < midEnd; i++) midSum += audioDataArray[i];
    for (let i = midEnd; i < bufferLength; i++) trebleSum += audioDataArray[i];
    
    const bass = (bassSum / bassEnd) / 255.0;
    const mid = (midSum / (midEnd - bassEnd)) / 255.0;
    const treble = (trebleSum / (bufferLength - midEnd)) / 255.0;
    
    // Exponential smoothing (0.7 factor)
    bassLevel = bassLevel * 0.7 + bass * 0.3;
    midLevel = midLevel * 0.7 + mid * 0.3;
    trebleLevel = trebleLevel * 0.7 + treble * 0.3;
    
    return { bass: bassLevel, mid: midLevel, treble: trebleLevel };
}
```

---

## BPM Detection System

### Algorithm Overview

**Method:** Onset detection with autocorrelation scoring

**Steps:**
1. Calculate RMS energy for each frame
2. Detect onsets (energy peaks above threshold)
3. Calculate intervals between onsets
4. Score BPM candidates (60-180 range, 0.5 step)
5. Return highest-scoring BPM

### Implementation

```javascript
function detectBPM(audioBuffer) {
    const sampleRate = audioBuffer.sampleRate;
    const channelData = audioBuffer.getChannelData(0);  // Mono analysis
    const bufferSize = 4096;
    const hopSize = 2048;
    
    // Step 1: Calculate RMS energy for each frame
    const energies = [];
    for (let i = 0; i < channelData.length - bufferSize; i += hopSize) {
        let sum = 0;
        for (let j = 0; j < bufferSize; j++) {
            sum += channelData[i + j] ** 2;
        }
        energies.push(Math.sqrt(sum / bufferSize));
    }
    
    // Step 2: Detect onsets (energy peaks)
    const onsets = [];
    const threshold = 0.15;  // Energy threshold
    const minTimeBetweenOnsets = 0.1;  // 100ms minimum gap
    
    for (let i = 1; i < energies.length - 1; i++) {
        // Peak detection: higher than neighbors + above threshold
        if (energies[i] > energies[i - 1] && 
            energies[i] > energies[i + 1] && 
            energies[i] > threshold) {
            
            const time = (i * hopSize) / sampleRate;
            
            // Enforce minimum gap between onsets
            if (onsets.length === 0 || 
                time - onsets[onsets.length - 1] >= minTimeBetweenOnsets) {
                onsets.push(time);
            }
        }
    }
    
    // Require minimum number of onsets
    if (onsets.length < 8) {
        console.warn('Beat detection: Not enough onsets detected');
        return 120.0;  // Default fallback
    }
    
    // Step 3: Calculate intervals between onsets
    const intervals = [];
    for (let i = 1; i < onsets.length; i++) {
        intervals.push(onsets[i] - onsets[i - 1]);
    }
    
    // Step 4: Score BPM candidates
    const bpmCandidates = [];
    
    for (let testBPM = 60; testBPM <= 180; testBPM += 0.5) {
        const testInterval = 60 / testBPM;  // Seconds per beat
        let score = 0;
        
        // Check how well this BPM matches detected intervals
        for (let interval of intervals) {
            // Try different multipliers (1x, 2x, 4x beats)
            for (let mult of [1, 2, 4]) {
                const expected = testInterval * mult;
                const diff = Math.abs(interval - expected);
                
                // Award points if interval matches expected (within 50ms)
                if (diff < 0.05) {
                    score += 1 / mult;  // Lower multipliers score higher
                }
            }
        }
        
        bpmCandidates.push({ bpm: testBPM, score: score });
    }
    
    // Step 5: Return highest-scoring BPM
    bpmCandidates.sort((a, b) => b.score - a.score);
    const detectedBPM = bpmCandidates[0].bpm;
    
    console.log(`Detected ${detectedBPM} BPM (${onsets.length} onsets)`);
    return detectedBPM;
}
```

### Performance Characteristics

**Analysis Time:**
- ~1-2 seconds for typical 3-5 minute song
- Depends on song length (linear scaling)
- Only runs once on audio load

**Accuracy:**
- Works best with clear beats (percussion, kicks, hi-hats)
- May struggle with:
  - Ambient/classical music (few onsets)
  - Very fast music (>180 BPM gets detected as half-time)
  - Very slow music (<60 BPM gets detected as double-time)

**Fallbacks:**
- Minimum 8 onsets required (returns 120 BPM if insufficient)
- User can always override by typing

---

## Nudge System

### Purpose

Fine-tune loop timing by shifting the entire loop (both Loop In and Loop Out) by milliseconds without changing beat positions.

### Implementation

```javascript
// Global variable
let nudgeOffset = 0;  // In seconds (converted from ms input)

// Applied to all audio position jumps
function jumpToPosition(beat) {
    const beatsPerSecond = audioTempo / 60.0;
    const baseTime = beat / beatsPerSecond;
    audioElement.currentTime = baseTime + nudgeOffset;  // ADD NUDGE
}

// Updated on Nudge field change
document.getElementById('nudgeField').addEventListener('input', (e) => {
    const ms = parseInt(e.target.value) || 0;
    nudgeOffset = ms / 1000;  // Convert ms to seconds
});
```

### Where Nudge is Applied

**1. Audio Load:**
```javascript
audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
```

**2. Stop Button:**
```javascript
audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
```

**3. Play (if outside loop range):**
```javascript
if (currentAudioBeat < inBeat || currentAudioBeat >= outBeat) {
    audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
}
```

**4. Loop (return to IN):**
```javascript
if (loopEnabled && currentAudioBeat >= outBeat) {
    audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
}
```

**5. Compile Shader (reset audio):**
```javascript
audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
```

**6. Video Render Start:**
```javascript
audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
```

**7. IN Beat Field Change:**
```javascript
audioElement.currentTime = (inBeat / beatsPerSecond) + nudgeOffset;
```

**8. Arrow Key Navigation:**
```javascript
audioElement.currentTime = (newIn / beatsPerSecond) + nudgeOffset;
```

### Arrow Keys in Nudge Field

```javascript
nudgeInput.addEventListener('keydown', (e) => {
    if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight'].includes(e.key)) {
        e.preventDefault();
        
        const currentNudge = parseInt(nudgeInput.value) || 0;
        let delta = 0;
        
        if (e.key === 'ArrowUp') delta = 1;        // +1ms
        else if (e.key === 'ArrowDown') delta = -1; // -1ms
        else if (e.key === 'ArrowRight') delta = 4;  // +4ms
        else if (e.key === 'ArrowLeft') delta = -4;  // -4ms
        
        const newNudge = Math.max(-5000, Math.min(5000, currentNudge + delta));
        nudgeInput.value = newNudge;
        nudgeOffset = newNudge / 1000;  // Convert to seconds
    }
});
```

---

## Modulation System

### Modulation Calculation

```javascript
function calculateMod(beat, subdivision, waveform) {
    const freq = 1.0 / subdivision;  // Convert subdivision to frequency
    const phase = (beat * freq) % 1.0;
    
    // Generate waveform value (0.0-1.0)
    switch (waveform) {
        case 'Sine': return (Math.sin(phase * Math.PI * 2) + 1) / 2;
        case 'Cosine': return (Math.cos(phase * Math.PI * 2) + 1) / 2;
        case 'Sawtooth': return phase;
        case 'Triangle': return phase < 0.5 ? phase * 2 : (1 - phase) * 2;
        case 'Square': return phase < 0.5 ? 1.0 : 0.0;
        // ... other waveforms
    }
}

// In render loop
let modA = calculateMod(beat, subdivisionA, waveformA);
let modB = calculateMod(beat, subdivisionB, waveformB);
let modC = calculateMod(beat, subdivisionC, waveformC);

// Multiply by audio levels (if audio loaded)
if (audioLoaded) {
    const audioLevels = analyzeAudio();
    modA *= (audioLevels.bass * bassGain);
    modB *= (audioLevels.mid * midGain);
    modC *= (audioLevels.treble * trebleGain);
}
```

---

## WebGPU Integration

### Uniforms Structure

```wgsl
struct Uniforms {
    time: f32,              // Elapsed time in seconds
    tempo: f32,             // Shader tempo (BPM)
    modA: f32,              // Modulator A (0-1, bass audio-reactive)
    modB: f32,              // Modulator B (0-1, mid audio-reactive)
    modC: f32,              // Modulator C (0-1, treble audio-reactive)
    padding: f32,           // Alignment padding
    resolution: vec2<f32>,  // Canvas width, height
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;
```

### User Shader Requirements

```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    // User shader code here
    return vec4<f32>(r, g, b, a);
}
```

---

## Video Export System

### MediaRecorder Setup

```javascript
// Capture canvas stream (no FPS param - captures every frame)
const canvasStream = canvas.captureStream();

// Add audio track if loaded
if (audioLoaded) {
    const audioDest = audioContext.createMediaStreamDestination();
    audioSource.disconnect();
    audioSource.connect(audioDest);
    audioSource.connect(analyser);
    
    const audioTrack = audioDest.stream.getAudioTracks()[0];
    canvasStream.addTrack(audioTrack);
}

// Create MediaRecorder
const recorder = new MediaRecorder(canvasStream, {
    mimeType: 'video/webm;codecs=vp8',
    videoBitsPerSecond: 8000000  // 8 Mbps
});
```

### Filename Format

```
OMS_{name}_{res}p_{fps}fps_{dur}s_{beats}beats_{orientation}.webm

Examples:
  OMS_myshader_1080p_30fps_32s_128beats_L.webm  (Landscape)
  OMS_myshader_1080p_30fps_32s_128beats_P.webm  (Portrait)
  OMS_myshader_1080p_30fps_32s_128beats_S.webm  (Square)
```

---

## API Reference

### Key Functions

```javascript
// Audio System
loadAudioFile(event)          // Load and analyze audio file
toggleAudioPlayback()         // Play/pause audio
stopAudio()                   // Stop audio and reset to Loop In
removeAudio()                 // Remove audio and reset UI
analyzeAudio()                // Get bass/mid/treble levels
detectBPM(audioBuffer)        // Automatic tempo detection

// Nudge System
// (handled via event listeners on nudgeField input)

// Loop Mode
toggleLoop()                  // Toggle loop IN→OUT vs continuous

// Waveform
generateStaticWaveform()      // Create cached waveform
drawRealWaveform()            // Render waveform to canvas
drawProgress()                // Draw playhead and IN/OUT indicators
setupWaveformControls()       // Initialize click/drag handlers

// WebGPU
initWebGPU()                  // Initialize WebGPU device
updateCanvasSize()            // Update canvas dimensions
compileShader()               // Compile user WGSL code
render()                      // Main rendering loop

// Video Export
startRender()                 // Begin video capture
finishRender()                // Complete and download video

// UI
switchTab(tabName)            // Switch between Controls/Code tabs
showStatus(message, type)     // Update status bar
showLoopDuration()            // Show beat count in status
```

---

## Extension Guide

### Adding New Waveform Types

1. Add option to dropdown:
```html
<option value="MyWave">MyWave</option>
```

2. Implement in `calculateMod()`:
```javascript
case 'MyWave':
    return /* your calculation */;
```

### Adding New Frequency Bands

1. Modify `analyzeAudio()`:
```javascript
const subBassEnd = Math.floor(bufferLength * 0.05);
// Calculate subBass average...
```

2. Add to modulator multiplication
3. Add UI control for gain

### Custom BPM Detection

Replace `detectBPM()` with your algorithm:
```javascript
function detectBPM(audioBuffer) {
    // Your custom analysis
    return detectedBPM;
}
```

---

## Performance Notes

### Bottlenecks

**1. Shader Complexity:**
- Main performance factor
- Complex shaders can drop below 60fps
- Solution: Optimize shader code

**2. BPM Detection:**
- 1-2 second analysis on load (one-time)
- No impact on playback/rendering
- Runs in main thread (could be moved to Web Worker)

**3. Canvas Size:**
- 1920x1080 has minimal GPU impact on modern hardware
- Well within capabilities of 4GB+ GPUs

**4. FFT Analysis:**
- ~1ms per frame (negligible)
- 256 FFT size for real-time performance

### Optimization Tips

**Shader:**
- Use `let` for reused calculations
- Minimize `sqrt()`, `pow()`, trigonometry
- Use lookup tables for complex functions

**Audio:**
- Keep FFT size at 256 (balance of resolution/speed)
- Exponential smoothing prevents jitter

**Video:**
- 8 Mbps bitrate is high-quality, adjust if needed
- Lower FPS (24/30) reduces file size

---

## Variables Reference

### Global State

```javascript
// WebGPU
let device, context, pipeline, canvas;
let uniformBuffer, uniformBindGroup;

// Timing
let startTime = Date.now();
let isRendering = false;
let animationId;

// Tempo
let currentTempo = 120;    // Shader visual tempo
let audioTempo = 120;      // Audio reference tempo

// Audio
let audioContext = null;
let audioElement = null;
let audioSource = null;
let analyser = null;
let audioDataArray = null;
let audioLoaded = false;
let audioBufferData = null;

// Frequency levels
let bassLevel = 0, midLevel = 0, trebleLevel = 0;

// Loop & Nudge
let loopEnabled = true;    // Loop mode toggle
let nudgeOffset = 0;       // Millisecond offset (in seconds)

// Modulation
let noiseState = { a: 0, b: 0, c: 0 };
```

---

**Last Updated:** November 2025  
**Version:** V2_2  
**Status:** Production Release  
**Documentation:** Complete

---

*For user workflows, see D_Release_Summary.md*  
*For version history, see B_Complete_Changelog.md*  
*For production deployment, see E_Production_Release_Guide.md*
