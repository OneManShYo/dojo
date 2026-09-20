# ONEMANSHYO Dojo - Developer Documentation

## Architecture Overview

**Application Type:** Single-file HTML application  
**Size:** 142KB (~3,245 lines)  
**Dependencies:** None (completely self-contained)  
**Technologies:** HTML5, CSS3, JavaScript ES6+, WebGPU, WGSL

---

## Core Systems

### 1. WebGPU Rendering Pipeline

**Initialization:**
```javascript
const adapter = await navigator.gpu.requestAdapter();
const device = await adapter.requestDevice();
```

**Shader Compilation:**
- User provides WGSL fragment shader code
- Compiled to GPU pipeline at runtime
- Automatic uniform binding for shader parameters

**Render Loop:**
- 60fps requestAnimationFrame loop
- Real-time uniform updates (tempo, modulators, audio FFT)
- Canvas renders to both preview and video capture

### 2. Audio Analysis System

**Web Audio API Chain:**
```
AudioElement → AnalyserNode → FFT Analysis → Frequency Bands
```

**Frequency Bands:**
- Bass: 20-250 Hz
- Mid: 250-4000 Hz
- Treble: 4000-20000 Hz

**FFT Configuration:**
- FFT Size: 2048 samples
- Smoothing: 0.8
- Update rate: Per animation frame

### 3. Timing Systems

**Two Independent Systems:**

**Shader Tempo (Visual Timing):**
- Controls: modulator subdivision timing
- Default: 120 (represents BPM for visual calculations)
- Used for: shader animation speed, modulator cycles
- Field ID: `tempo`

**Audio Tempo (Musical Timing):**
- Controls: beat positioning, loop calculations
- User-entered: decimal precision
- Used for: Loop In/Out positioning, render duration
- Field ID: `audioTempoField`

**Critical Separation:**
These systems are intentionally independent. Only integration point is:
- Loop In/Out brackets define video render time range
- Audio FFT analysis feeds shader modulators (Bass/Mid/Treble)

### 4. Time Format Conversion

**Functions:**
```javascript
timeStringToSeconds(str)  // "05:30.500" → 330.5
secondsToTimeString(sec)  // 330.5 → "05:30.500"
barBeatToBeats(str)       // "17.1" → 64
beatsToBarBeat(beats)     // 64 → "17.1"
```

**Format Standards:**
- Time fields: MM:SS.mmm (left-to-right parsing)
- Beat fields: bar.beat (1-indexed, 4/4 time)

---

## Key APIs

### Shader Uniform System

**Available Uniforms:**
```wgsl
@group(0) @binding(0) var<uniform> tempo: f32;      // Shader Tempo
@group(0) @binding(1) var<uniform> modA: f32;       // Bass modulator (0.0-1.0)
@group(0) @binding(2) var<uniform> modB: f32;       // Mid modulator (0.0-1.0)
@group(0) @binding(3) var<uniform> modC: f32;       // Treble modulator (0.0-1.0)
@group(0) @binding(4) var<uniform> resolution: vec2<f32>;
```

**Modulator Calculation:**
```javascript
// Subdivision-based oscillation
const cycleLength = 60 / (tempo * subdivision);
const phase = (beat % cycleLength) / cycleLength;
const waveValue = waveFunction(phase); // sine, saw, square, etc.
const modulatorValue = waveValue * gain; // 0.0-1.0 range
```

### Render System API

**Video Export:**
```javascript
startRender()  // Initiates MediaRecorder capture
finishRender() // Stops recording, triggers download
```

**Render Modes:**

**No Audio Mode:**
- Uses Render Duration field (MM:SS.mmm)
- Duration determines video length
- Shader animates based on Shader Tempo

**Audio Mode:**
- Uses Loop In/Out brackets
- Duration = (Loop Out - Loop In beats) × 60 / Audio Tempo
- Audio synced to video from Loop In position

**Output Format:**
- Container: WebM
- Video Codec: VP8
- Resolution: 720p, 1080p, 1440p, 2160p
- Frame Rate: 24, 30, 60 FPS
- Orientation: Portrait, Landscape, Square

---

## File Structure

```
Lines 1-35:     Version format conventions
Lines 36-600:   CSS styling
Lines 601-1400: HTML structure
Lines 1401-end: JavaScript application logic
```

**Major JavaScript Sections:**
- WebGPU initialization (setup)
- Audio processing (FFT, beat detection)
- UI event handlers (controls, file upload)
- Render loop (animation frame)
- Video recording (MediaRecorder)
- Helper functions (time conversion, beat math)

---

## Extension Points

### Adding New Shader Uniforms

1. Add uniform to WGSL shader template
2. Create uniform buffer in WebGPU setup
3. Update uniform values in render loop
4. Add UI control in HTML (if user-facing)

### Adding New Modulator Waveforms

1. Add option to waveform dropdown
2. Implement waveform function:
```javascript
function customWave(phase) {
    // phase: 0.0-1.0
    // return: 0.0-1.0
    return Math.abs(Math.sin(phase * Math.PI * 4));
}
```
3. Add to modulator calculation switch statement

### Adding New Export Formats

MediaRecorder API supports:
- WebM (VP8/VP9)
- MP4 (if browser supports)

Check MIME type support:
```javascript
MediaRecorder.isTypeSupported('video/mp4; codecs=avc1')
```

---

## Browser Compatibility Requirements

**Essential APIs:**
- WebGPU (Chrome 113+, Edge 113+, Safari TP)
- Web Audio API
- MediaRecorder API
- FileReader API
- Canvas API

**Optional APIs:**
- requestAnimationFrame (fallback: setTimeout)

---

## Performance Considerations

**GPU Intensive:**
- Shader complexity directly affects frame rate
- Complex shaders may struggle at 4K 60fps
- Test render at lower resolution first

**Memory Usage:**
- Video recording buffers frames in memory
- Long renders (>60s) may consume significant RAM
- Browser may throttle or crash on extended renders

**Audio FFT:**
- FFT analysis runs every frame (60 times/second)
- Negligible CPU impact with optimized implementation

---

## Common Integration Patterns

### Claude Shader Creation Workflow

1. User describes visual concept to Claude
2. Claude generates WGSL fragment shader with proper uniforms
3. User pastes shader into Code tab
4. Click [Compile] to test
5. Adjust modulators/tempo in real-time
6. Render when satisfied

### Live Performance Workflow

1. Render multiple shader variations (different tempos/colors)
2. Export without audio (no-audio mode)
3. Import videos into Ableton Live
4. Use ebosuite for clip launching
5. Control with Midifighter Twister/Yaeltex controller

### Social Media Workflow

1. Load audio track (TikTok audio, Instagram sound)
2. Set correct Audio Tempo
3. Set Loop In/Out to desired clip range (15s, 30s, 60s)
4. Render with audio at target resolution (1080p Portrait)
5. Upload directly or post-process with FFmpeg

---

## Debug Tips

**WebGPU Not Available:**
- Check chrome://gpu for WebGPU status
- Enable chrome://flags/#enable-unsafe-webgpu
- Update browser to latest version

**Shader Compilation Errors:**
- Check browser console for WGSL error messages
- Verify uniform bindings match shader code
- Test with default shader first

**Audio Not Loading:**
- Check file format (MP3, WAV, OGG supported)
- Verify file isn't corrupted
- Check browser console for CORS errors

**Render Issues:**
- Verify MediaRecorder supports WebM VP8
- Check available disk space
- Monitor browser console during render

---

## Code Modification Guidelines

**Before Making Changes:**
1. Always work on iteration builds (V2.5.0_A, etc.)
2. Test each change independently
3. Document changes in iteration notes file
4. Only merge to production when fully tested

**Version String Updates:**
All version references must be updated consistently:
- HTML comment (line 3): `v2.5.0x`
- Title tag: `v2.5.0x`
- Header display: `[V2.5.0_X]`

**Avoid:**
- Embedding large documentation in HTML comments
- Using Unicode symbols (UTF-8 encoding issues)
- Multiple simultaneous feature changes
- Breaking single-file architecture

---

**Last Updated:** November 2025  
**Current Version:** V2_5  
**Maintainer:** Wes Smith / OneManShYo
