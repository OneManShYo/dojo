# ONEMANSHYO Dojo - Developer Documentation

## Technical Architecture Overview

### System Design Philosophy

**Single-File Application:**
- All functionality in one HTML file (~199KB)
- No external dependencies
- Portable, self-contained
- Runs offline after first load
- No build process required

**Two Independent Modules:**
1. **Shader Module** (Visual Timing System)
2. **Audio Module** (Musical Timing System)

### Core Technologies

- **WebGPU**: Modern graphics API for shader rendering
- **WGSL**: WebGPU Shading Language for fragment shaders
- **Web Audio API**: FFT analysis for audio reactivity
- **LocalStorage**: User preset persistence
- **MediaRecorder**: Video export (WebM/VP8)

---

## Module Architecture

### Shader Module

**Purpose:** Controls visual animation timing in shader code

**UI Location:** Controls tab, Rows 1-5

**Key Components:**
```
Row 1: Shader Tempo (BPM) | Loop Length | Render Duration
Row 2: Modulator A (subdivision, waveform, gain)
Row 3: Modulator B (subdivision, waveform, gain)
Row 4: Modulator C (subdivision, waveform, gain)
Row 5: Output settings (resolution, FPS, orientation)
```

**Shader Uniforms:**
```wgsl
struct Uniforms {
    time: f32,           // Elapsed time in seconds
    tempo: f32,          // Shader Module BPM
    modA: f32,           // Modulator A value (0.0-1.0)
    modB: f32,           // Modulator B value (0.0-1.0)
    modC: f32,           // Modulator C value (0.0-1.0)
    padding1: f32,       // Alignment padding
    resolutionX: f32,    // Canvas width
    resolutionY: f32,    // Canvas height
    padding2-5: f32      // Additional padding (48 bytes total)
}
```

**Critical Note:** Uniform buffer MUST be 48 bytes for proper 16-byte alignment. Struct uses separate resolutionX/Y instead of vec2 to avoid alignment issues.

### Audio Module

**Purpose:** Controls audio loop positioning and beat-based navigation

**UI Location:** Audio controls strip below waveform

**Key Components:**
```
- Audio Tempo (BPM) - manual entry
- Audio Start/End (MM:SS.mmm format)
- Loop In/Out brackets (bar.beat format, 1-indexed)
- Waveform visualization with beat grid
- Transport controls (Play/Pause, Loop, Remove Audio)
```

**Time Format Conversions:**
```javascript
// MM:SS.mmm string to seconds
timeStringToSeconds(timeStr)

// bar.beat to absolute beat number
barBeatToBeats(barBeatStr)

// Seconds to MM:SS.mmm string
secondsToTimeString(seconds)
```

### Module Integration Points

Only two connection points exist between modules:

1. **Loop Brackets → Render Duration**
   - When audio loaded, Loop In/Out define render time range
   - Audio Module sets duration, Shader Module uses it

2. **FFT → Modulators**
   - Audio waveform analysis (Bass/Mid/Treble)
   - Drives Shader Module modulators A/B/C
   - Real-time frequency response

---

## WebGPU Pipeline

### Initialization Sequence

```javascript
1. Request GPU adapter
   const adapter = await navigator.gpu.requestAdapter();

2. Request GPU device
   const device = await adapter.requestDevice();

3. Configure canvas context
   const context = canvas.getContext('webgpu');
   context.configure({
       device: device,
       format: navigator.gpu.getPreferredCanvasFormat(),
       alphaMode: 'premultiplied'
   });

4. Create shader module
   const shaderModule = device.createShaderModule({
       code: vertexShaderCode + fragmentShaderCode
   });

5. Create uniform buffer (48 bytes)
   const uniformBuffer = device.createBuffer({
       size: 48,
       usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
   });

6. Create bind group layout
   const bindGroupLayout = device.createBindGroupLayout({
       entries: [{
           binding: 0,
           visibility: GPUShaderStage.FRAGMENT,
           buffer: { type: 'uniform' }
       }]
   });

7. Create render pipeline
   const pipeline = device.createRenderPipeline({
       layout: device.createPipelineLayout({
           bindGroupLayouts: [bindGroupLayout]
       }),
       vertex: { module: shaderModule, entryPoint: 'vertexMain' },
       fragment: {
           module: shaderModule,
           entryPoint: 'fragmentMain',
           targets: [{ format: navigator.gpu.getPreferredCanvasFormat() }]
       }
   });
```

### Render Loop

```javascript
function render() {
    // Calculate uniforms
    const uniformData = new Float32Array([
        currentTime,      // 0: time
        currentTempo,     // 1: tempo
        modA,             // 2: modA
        modB,             // 3: modB
        modC,             // 4: modC
        0,                // 5: padding1
        canvas.width,     // 6: resolutionX
        canvas.height,    // 7: resolutionY
        0, 0, 0, 0        // 8-11: padding2-5 (48 bytes total)
    ]);
    
    // Write to GPU
    device.queue.writeBuffer(uniformBuffer, 0, uniformData);
    
    // Create command encoder
    const commandEncoder = device.createCommandEncoder();
    
    // Get render target
    const textureView = context.getCurrentTexture().createView();
    
    // Begin render pass
    const renderPass = commandEncoder.beginRenderPass({
        colorAttachments: [{
            view: textureView,
            clearValue: { r: 0, g: 0, b: 0, a: 1 },
            loadOp: 'clear',
            storeOp: 'store'
        }]
    });
    
    // Set pipeline and draw
    renderPass.setPipeline(pipeline);
    renderPass.setBindGroup(0, bindGroup);
    renderPass.draw(6); // Full-screen quad (2 triangles)
    renderPass.end();
    
    // Submit commands
    device.queue.submit([commandEncoder.finish()]);
    
    // Update progress counter (if rendering)
    if (isRendering && renderStartTime > 0) {
        const elapsedTime = (Date.now() - renderStartTime) / 1000;
        const currentFrame = Math.floor(elapsedTime * renderFPS);
        
        if (currentFrame % 5 === 0 || currentFrame >= totalFrames - 1) {
            const elapsedText = secondsToTimeString(Math.min(elapsedTime, renderDuration));
            const totalText = secondsToTimeString(renderDuration);
            showStatus(`Rendering ${elapsedText}/${totalText} | Frame ${currentFrame}/${totalFrames}`, 'info');
        }
    }
    
    // Next frame
    animationId = requestAnimationFrame(render);
}
```

---

## Audio Analysis System

### FFT Configuration

```javascript
// Web Audio setup
audioContext = new AudioContext();
analyserNode = audioContext.createAnalyser();
analyserNode.fftSize = 2048;
analyserNode.smoothingTimeConstant = 0.8;

// Connect audio element
sourceNode = audioContext.createMediaElementSource(audioElement);
sourceNode.connect(analyserNode);
analyserNode.connect(audioContext.destination);
```

### Frequency Band Analysis

```javascript
function updateAudioReactivity() {
    const dataArray = new Uint8Array(analyserNode.frequencyBinCount);
    analyserNode.getByteFrequencyData(dataArray);
    
    // Define frequency ranges
    const sampleRate = audioContext.sampleRate;
    const nyquist = sampleRate / 2;
    const binCount = analyserNode.frequencyBinCount;
    
    // Bass: 20-250Hz
    const bassStart = Math.floor((20 / nyquist) * binCount);
    const bassEnd = Math.floor((250 / nyquist) * binCount);
    
    // Mids: 250-4000Hz
    const midStart = bassEnd;
    const midEnd = Math.floor((4000 / nyquist) * binCount);
    
    // Treble: 4000-20000Hz
    const trebleStart = midEnd;
    const trebleEnd = Math.floor((20000 / nyquist) * binCount);
    
    // Calculate averages
    bassLevel = average(dataArray, bassStart, bassEnd) / 255;
    midLevel = average(dataArray, midStart, midEnd) / 255;
    trebleLevel = average(dataArray, trebleStart, trebleEnd) / 255;
}
```

---

## Preset System Architecture

### OMS Library (Hardcoded)

**Location:** JavaScript array `omsPresets`

**Structure:**
```javascript
const omsPresets = [
    {
        name: "TronTunnel",
        orientation: "portrait",
        code: "// WGSL shader code..."
    },
    // ... 4 more presets
];
```

**Presets Included:**
1. Tron Tunnel (Portrait) - Pink-blue radial tunnel
2. Tron Grid (Portrait) - Retrowave sun with grid floor
3. MetaBalls (Square) - Lava lamp blobs
4. Flower Power (Square) - Rotating circle pattern
5. Retro Fish Farm (Square) - Animated grid with dots

### User Presets (LocalStorage)

**Storage Key:** `'omsUserPresets'`

**Structure:**
```javascript
[
    {
        name: "MyCustomShader",
        orientation: "portrait",
        code: "// WGSL shader code..."
    },
    // ... more user presets
]
```

**Functions:**
```javascript
// Read from localStorage
function getUserPresets() {
    const stored = localStorage.getItem('omsUserPresets');
    return stored ? JSON.parse(stored) : [];
}

// Write to localStorage
function saveUserPresets(presets) {
    localStorage.setItem('omsUserPresets', JSON.stringify(presets));
}

// Parse shader name from code comment
function parseShaderNameFromCode(shaderCode) {
    // Look for: // ONEMANSHYO Dojo [version] - ShaderName
    // Extract name after last dash
    // Clean special characters
    // Return null if not found
}

// Extract name from filename
function extractShaderName(filename, code) {
    // Try parseShaderNameFromCode first
    // Fall back to filename without extension
    // Default to "Untitled Shader"
}
```

### Thumbnail Rendering System

**Approach:** Offscreen WebGPU → 2D Canvas Display

```javascript
async function initThumbnailShader(displayCanvas, shaderCode, index) {
    try {
        // 1. Create offscreen canvas
        const offscreenCanvas = document.createElement('canvas');
        offscreenCanvas.width = displayCanvas.width;
        offscreenCanvas.height = displayCanvas.height;
        
        // 2. Get WebGPU context on offscreen canvas
        const context = offscreenCanvas.getContext('webgpu');
        
        // 3. Configure WebGPU (same as main shader)
        // 4. Create shader module (vertex + wrapped fragment)
        // 5. Create uniform buffer (48 bytes)
        // 6. Create bind group layout and bind group
        // 7. Create render pipeline
        
        // 8. Render single frame (time = 2.0 seconds)
        const uniformData = new Float32Array([
            2.0,                       // time
            120.0,                     // tempo
            0.5, 0.5, 0.5,            // modA/B/C
            0.0,                       // padding1
            offscreenCanvas.width,     // resolutionX
            offscreenCanvas.height,    // resolutionY
            0.0, 0.0, 0.0, 0.0        // padding2-5
        ]);
        
        device.queue.writeBuffer(uniformBuffer, 0, uniformData);
        
        // Execute render pass
        // Wait for completion
        await device.queue.onSubmittedWorkDone();
        
        // 9. Copy offscreen to display canvas (2D context)
        const ctx2d = displayCanvas.getContext('2d');
        ctx2d.drawImage(offscreenCanvas, 0, 0);
        
        console.log(`Thumbnail ${index} rendered and copied successfully`);
        
    } catch (error) {
        console.error(`Thumbnail ${index} failed:`, error);
        // Draw error indicator on display canvas
        const ctx2d = displayCanvas.getContext('2d');
        ctx2d.fillStyle = '#400';
        ctx2d.fillRect(0, 0, displayCanvas.width, displayCanvas.height);
        ctx2d.fillStyle = '#f00';
        ctx2d.font = '12px monospace';
        ctx2d.fillText('Error', 10, 20);
    }
}
```

**Why This Works:**
- WebGPU may not display correctly in all contexts
- 2D canvas is universally reliable for display
- Offscreen render isolates WebGPU complexity
- drawImage() copies pixels to display canvas
- Bulletproof approach across browsers

---

## Video Export System

### MediaRecorder Configuration

```javascript
function startRender() {
    // Get canvas stream
    const stream = canvas.captureStream(fps);
    
    // Add audio track if loaded
    if (audioLoaded && audioElement.srcObject) {
        const audioTracks = audioElement.srcObject.getAudioTracks();
        audioTracks.forEach(track => stream.addTrack(track));
    }
    
    // Configure MediaRecorder
    mediaRecorder = new MediaRecorder(stream, {
        mimeType: 'video/webm;codecs=vp8',
        videoBitsPerSecond: 10000000 // 10 Mbps
    });
    
    // Collect chunks
    chunks = [];
    mediaRecorder.ondataavailable = (e) => {
        if (e.data.size > 0) {
            chunks.push(e.data);
        }
    };
    
    // Handle completion
    mediaRecorder.onstop = () => {
        const blob = new Blob(chunks, { type: 'video/webm' });
        const url = URL.createObjectURL(blob);
        
        // Trigger download
        const a = document.createElement('a');
        a.href = url;
        a.download = generateFilename();
        a.click();
        
        URL.revokeObjectURL(url);
        finishRender('Video rendered successfully!');
    };
    
    // Start recording
    mediaRecorder.start();
}
```

### Filename Generation

```javascript
function generateFilename() {
    // Get user-entered filename
    let filename = document.getElementById('filename').value.trim();
    
    // Parse from shader code if empty
    if (!filename) {
        const shaderCode = document.getElementById('shaderCode').value;
        const parsedName = parseShaderNameFromCode(shaderCode);
        filename = parsedName || 'filename';
    }
    
    // Get render settings
    const resolution = document.getElementById('resolution').value;
    const fps = document.getElementById('fps').value;
    const orientation = document.getElementById('orientation').value;
    
    // Orientation suffix
    const orientationSuffix = {
        'portrait': '_P',
        'landscape': '_L',
        'square': '_S'
    }[orientation];
    
    // Format: OMS_[name]_[resolution]p_[fps]fps_[orientation].webm
    return `OMS_${filename}_${resolution}p_${fps}fps${orientationSuffix}.webm`;
}
```

---

## Critical Implementation Details

### WebGPU Uniform Buffer Alignment

**CRITICAL:** Buffer must be 48 bytes with proper padding

```javascript
// WRONG (32 bytes - causes validation error)
const uniformData = new Float32Array([
    time, tempo, modA, modB, modC,
    canvas.width, canvas.height, 0
]); // 8 floats = 32 bytes - TOO SMALL!

// CORRECT (48 bytes - proper alignment)
const uniformData = new Float32Array([
    time,           // 0
    tempo,          // 1
    modA,           // 2
    modB,           // 3
    modC,           // 4
    0,              // 5: padding1
    canvas.width,   // 6: resolutionX
    canvas.height,  // 7: resolutionY
    0, 0, 0, 0      // 8-11: padding2-5
]); // 12 floats = 48 bytes - CORRECT!
```

**Why 48 Bytes:**
- vec2<f32> requires 8-byte alignment
- Struct size must be multiple of 16 bytes
- Minimum required: 40 bytes (calculated by WebGPU)
- Rounded up to 16-byte multiple: 48 bytes

### WGSL Struct Layout

**Use separate f32 fields instead of vec2:**

```wgsl
// WRONG (causes alignment issues)
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    resolution: vec2<f32>,  // Requires 8-byte alignment, causes offset mismatch
}

// CORRECT (explicit layout control)
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    padding1: f32,
    resolutionX: f32,
    resolutionY: f32,
}

// Reconstruct vec2 in shader code
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
```

### Loop Playback Logic

```javascript
function updateLoop() {
    if (!loopEnabled || !audioLoaded) return;
    
    // Calculate current beat position
    const audioBeatsPerSecond = audioTempo / 60.0;
    const currentAudioBeat = (audioElement.currentTime - audioStartTime) * audioBeatsPerSecond;
    
    // Get Loop Out beat
    const outBeat = barBeatToBeats(document.getElementById('outBeat').value);
    
    // Loop back with 0.15 beat threshold
    if (currentAudioBeat >= (outBeat - 0.15)) {
        const inBeat = barBeatToBeats(document.getElementById('inBeat').value);
        const inBeatTime = inBeat / audioBeatsPerSecond;
        audioElement.currentTime = audioStartTime + inBeatTime;
    }
}
```

---

## Performance Considerations

### Thumbnail Rendering

**Optimization Strategies:**
- Static single-frame render (no animation loops)
- Offscreen canvas + 2D display copy
- Minimal resource usage after render
- Canvas sizes: Portrait 90x160, Landscape 160x90, Square 120x120

**Memory Usage:**
- 4-5 WebGPU contexts (Library + User Presets)
- Each context: ~10-20MB GPU memory
- Total: ~50-100MB during thumbnail generation
- Zero ongoing usage (no animation loops)

### Video Rendering

**Resource Requirements:**
- Canvas capture stream: ~100-200MB RAM
- MediaRecorder buffer: ~100-500MB RAM
- Total: ~200-700MB during render
- Released immediately after completion

**Frame Rate Impact:**
- 24fps: Lighter load, good for most shaders
- 30fps: Standard, recommended default
- 60fps: Heavier load, smooth motion

---

## Error Handling

### WebGPU Initialization

```javascript
async function initWebGPU() {
    if (!navigator.gpu) {
        showStatus('WebGPU not supported', 'error');
        return false;
    }
    
    try {
        adapter = await navigator.gpu.requestAdapter();
        if (!adapter) {
            showStatus('Could not get GPU adapter', 'error');
            return false;
        }
        
        device = await adapter.requestDevice();
        return true;
    } catch (error) {
        showStatus(`WebGPU initialization failed: ${error.message}`, 'error');
        return false;
    }
}
```

### Shader Compilation

```javascript
async function compileShader(silent = false) {
    try {
        const shaderModule = device.createShaderModule({
            code: vertexShaderCode + fragmentShaderCode
        });
        
        // Check for compilation warnings
        const compilationInfo = await shaderModule.getCompilationInfo();
        for (const message of compilationInfo.messages) {
            if (message.type === 'error') {
                console.error('Shader error:', message);
                showStatus('Shader compilation error', 'error');
                return false;
            }
        }
        
        if (!silent) {
            showStatus('Shader compiled successfully');
        }
        return true;
    } catch (error) {
        showStatus(`Shader compilation failed: ${error.message}`, 'error');
        return false;
    }
}
```

---

## File Size Management

**V2_6 Size: ~199KB**

**Optimization Strategies:**
1. No embedded changelog (external notes files)
2. No large comment blocks
3. Minified shader presets
4. Efficient function naming
5. Shared code reuse

**What NOT to Embed:**
- Version history (external)
- Large documentation blocks
- Redundant comments
- Debug logging code
- Unused functions

---

## Development Workflow

### Adding New Iteration

1. Copy previous iteration HTML
2. Update version string (e.g., V2_6 → V2_6)
3. Make single focused change
4. Test thoroughly
5. Create iteration notes file
6. Document changes completely

### Testing Checklist

- [ ] WebGPU initialization
- [ ] Shader compilation
- [ ] Preset loading
- [ ] Thumbnail rendering
- [ ] Video export
- [ ] Audio sync (if applicable)
- [ ] LocalStorage persistence
- [ ] Keyboard navigation
- [ ] Cross-browser compatibility

---

**Documentation Version:** V2_6  
**Last Updated:** November 2025  
**Target Audience:** Developers extending or debugging the application
