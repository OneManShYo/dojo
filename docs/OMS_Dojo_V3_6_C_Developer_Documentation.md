# OMS SHADER BAKER V3.6.0 - DEVELOPER DOCUMENTATION

**Version:** V3.6.0  
**Date:** November 16, 2025  
**Purpose:** Technical reference for developers and contributors

---

## ARCHITECTURE OVERVIEW

### Single-File Philosophy

**Design Goal:** Complete functionality in one portable HTML file

**File:** OMS_ShaderBaker_V3_6_0.html  
**Size:** 475KB  
**Lines:** ~11,600  
**Dependencies:** Zero external files

**Benefits:**
- Works offline
- No build process
- No npm/node required
- Copy file = full application
- Version control friendly

---

## CORE SYSTEMS

### 1. Four-Track Shader Compositing

**Arrays Structure:**

```javascript
// WebGPU devices (one per track)
let liveShaderDevices = [null, null, null, null];
let liveShaderPipelines = [null, null, null, null];
let liveShaderUniformBuffers = [null, null, null, null];
let liveShaderBindGroups = [null, null, null, null];

// Canvas elements
let liveShaderCanvases = [null, null, null, null];
let liveShaderContexts = [null, null, null, null];

// Same structure for pop-out window
let popOutDevices = [null, null, null, null];
let popOutPipelines = [null, null, null, null];
// ... etc
```

**Initialization:**

```javascript
async function initLiveMixer() {
    // Create 4 canvases in DOM
    for (let i = 0; i < 4; i++) {
        const canvas = document.getElementById(`liveShaderCanvas${i}`);
        liveShaderCanvases[i] = canvas;
        liveShaderContexts[i] = canvas.getContext('webgpu');
        
        // Request WebGPU device
        const adapter = await navigator.gpu.requestAdapter();
        liveShaderDevices[i] = await adapter.requestDevice();
    }
}
```

**Render Loop:**

```javascript
function renderLiveShader() {
    const sharedTime = (audioContext.currentTime - masterClockStartTime) * audioPlaybackRate;
    
    // Render INSET (DJ view)
    for (let track = 0; track < 4; track++) {
        if (!liveShaderPipelines[track]) continue;
        
        // Volume-based opacity
        const volume = trackGainNodes[track].gain.value;
        const intensity = Math.min(1.0, volume * 2.0); // Peaks at 50%
        liveShaderCanvases[track].style.opacity = intensity;
        
        // WebGPU rendering
        const commandEncoder = liveShaderDevices[track].createCommandEncoder();
        const renderPass = commandEncoder.beginRenderPass({
            colorAttachments: [{
                view: liveShaderContexts[track].getCurrentTexture().createView(),
                loadOp: 'clear',
                storeOp: 'store'
            }]
        });
        
        // Update uniforms
        const uniformData = new Float32Array([
            sharedTime,
            liveShaderCanvases[track].width,
            liveShaderCanvases[track].height,
            currentModA, currentModB, currentModC
        ]);
        liveShaderDevices[track].queue.writeBuffer(
            liveShaderUniformBuffers[track], 0, uniformData
        );
        
        // Draw
        renderPass.setPipeline(liveShaderPipelines[track]);
        renderPass.setBindGroup(0, liveShaderBindGroups[track]);
        renderPass.draw(4);
        renderPass.end();
        
        liveShaderDevices[track].queue.submit([commandEncoder.finish()]);
    }
    
    // Render POP-OUT (audience view)
    if (popOutWindow && !popOutWindow.closed) {
        for (let track = 0; track < 4; track++) {
            // Same rendering to pop-out canvases
        }
    }
    
    requestAnimationFrame(renderLiveShader);
}
```

---

### 2. Pop-Out Window System

**Opening:**

```javascript
function openPopOutWindow() {
    const [width, height] = appSettings.popOutWindowSize.split('x').map(Number);
    
    popOutWindow = window.open(
        '', 
        'shaderOutput',
        `width=${width},height=${height},resizable=yes,menubar=no,toolbar=no`
    );
    
    // Write HTML structure
    popOutWindow.document.write(`
        <!DOCTYPE html>
        <html>
        <head>
            <style>
                body { margin: 0; background: #000; overflow: hidden; }
                .container { width: 100%; height: 100vh; position: relative; }
                canvas { position: absolute; top: 0; left: 0; width: 100%; height: 100%; }
            </style>
        </head>
        <body>
            <div class="container">
                <canvas id="popOutCanvas0"></canvas>
                <canvas id="popOutCanvas1"></canvas>
                <canvas id="popOutCanvas2"></canvas>
                <canvas id="popOutCanvas3"></canvas>
            </div>
        </body>
        </html>
    `);
    
    // Initialize WebGPU for pop-out canvases
    initPopOutShaders();
}
```

**Closing:**

```javascript
function closePopOutWindow() {
    if (popOutWindow && !popOutWindow.closed) {
        // Clean up WebGPU resources
        for (let i = 0; i < 4; i++) {
            if (popOutDevices[i]) {
                popOutDevices[i].destroy();
                popOutDevices[i] = null;
            }
            popOutPipelines[i] = null;
        }
        
        popOutWindow.close();
        popOutWindow = null;
    }
}
```

---

### 3. Waveform Overlay System

**HTML Structure:**

```html
<div class="waveform-canvas-container" style="position: relative;">
    <canvas class="waveform-canvas" id="waveform1"></canvas>
    
    <div class="waveform-overlay" id="waveformOverlay1">
        <canvas class="waveform-overlay-thumbnail" 
                id="waveformOverlayThumb1" 
                width="90" height="51"></canvas>
        <div class="waveform-overlay-name" style="width: 150px;">
            <span class="waveform-overlay-name-text"></span>
        </div>
    </div>
</div>
```

**Update Function:**

```javascript
function updateWaveformOverlay(track) {
    const overlay = document.getElementById(`waveformOverlay${track + 1}`);
    const thumb = document.getElementById(`waveformOverlayThumb${track + 1}`);
    const nameDiv = overlay.querySelector('.waveform-overlay-name');
    const nameTextSpan = overlay.querySelector('.waveform-overlay-name-text');
    
    const container = trackContainers[track];
    if (!container) {
        overlay.classList.remove('visible');
        return;
    }
    
    // Show overlay
    overlay.classList.add('visible');
    
    // Render shader thumbnail
    const thumbCtx = thumb.getContext('2d');
    thumbCtx.clearRect(0, 0, 90, 51);
    
    if (liveShaderCanvases[track]) {
        thumbCtx.drawImage(liveShaderCanvases[track], 0, 0, 90, 51);
    }
    
    // Set name text (duplicated for marquee)
    nameTextSpan.textContent = container.name + ' ' + container.name;
    
    // Enable scrolling if text overflows
    const textWidth = nameTextSpan.scrollWidth / 2;
    const containerWidth = 150 - 24; // Width minus padding
    
    if (textWidth > containerWidth) {
        nameDiv.classList.add('scrolling');
    } else {
        nameDiv.classList.remove('scrolling');
    }
}
```

**CSS Animation:**

```css
@keyframes marquee-scroll {
    0% { transform: translateX(0); }
    100% { transform: translateX(-50%); }
}

.waveform-overlay-name.scrolling .waveform-overlay-name-text {
    animation: marquee-scroll 10s linear infinite;
}
```

---

### 4. Visual Indicators System

**CSS Stacking:**

```css
/* Priority #1: Cyan Cursor (OUTLINE - outermost) */
.clip-slot.selected {
    outline: 1px solid #00d4ff;
    outline-offset: -1px;
}

/* Priority #2: Yellow Playing (BORDER - middle) */
.clip-slot.playing {
    border: 2px solid #ffff00 !important;
    background: rgba(255, 255, 0, 0.15) !important;
}

/* Priority #3: Shader Content (BACKGROUND - innermost) */
.clip-slot canvas {
    /* Normal canvas rendering */
}
```

**JavaScript Classes:**

```javascript
function launchClipAtCell(track, scene) {
    // Add playing class
    const cell = document.querySelector(`[data-track="${track}"][data-scene="${scene}"]`);
    cell.classList.add('playing');
}

function stopTrack(track) {
    // Remove playing class from all cells in track
    const cells = document.querySelectorAll(`[data-track="${track}"]`);
    cells.forEach(cell => cell.classList.remove('playing'));
}
```

---

### 5. Settings Persistence

**Data Structure:**

```javascript
let appSettings = {
    popOutWindowSize: '640x360',  // '640x360' | '1280x720'
    popOutAlwaysOnTop: true        // boolean
};
```

**Save to .omsp:**

```javascript
function getCurrentProjectState() {
    return {
        version: '3.6.0',
        masterTempo: masterTempo,
        gridData: gridData,
        // ... other project data
        settings: {
            popOutWindowSize: appSettings.popOutWindowSize,
            popOutAlwaysOnTop: appSettings.popOutAlwaysOnTop
        }
    };
}
```

**Load from .omsp:**

```javascript
function applyProjectState(projectData) {
    // Load settings
    if (projectData.settings) {
        appSettings.popOutWindowSize = projectData.settings.popOutWindowSize || '640x360';
        appSettings.popOutAlwaysOnTop = projectData.settings.popOutAlwaysOnTop ?? true;
        
        // Update UI
        document.getElementById('popOutSizeSelect').value = appSettings.popOutWindowSize;
        document.getElementById('popOutAlwaysOnTopCheckbox').checked = appSettings.popOutAlwaysOnTop;
    }
    
    // Load grid data
    // ... rest of loading logic
}
```

---

## KEY ALGORITHMS

### Volume-Based Opacity Mixing

**Formula:**

```javascript
const volume = trackGainNodes[track].gain.value; // 0.0 to 1.0
const intensity = Math.min(1.0, volume * 2.0);   // Peaks at 50% volume
canvas.style.opacity = intensity;
```

**Why ×2:**
- Volume at 0% → opacity 0% (invisible)
- Volume at 50% → opacity 100% (fully visible)
- Volume at 100% → opacity 100% (clamped)
- Natural mixing sweet spot at 50% fader position

### Quantized Launching

**Master Clock:**

```javascript
// First clip sets reference
if (masterClockStartTime === null) {
    masterClockStartTime = audioContext.currentTime + 0.2; // 200ms lookahead
    startAudioAtTime(track, masterClockStartTime);
    return;
}

// Subsequent clips wait for bar boundary
const beatDuration = 60 / masterTempo;
const barDuration = beatDuration * 4; // 4 beats per bar

const timeSinceMasterStart = audioContext.currentTime - masterClockStartTime;
const barsElapsed = timeSinceMasterStart / barDuration;
const nextBarStart = Math.ceil(barsElapsed) * barDuration;
const nextBarTime = masterClockStartTime + nextBarStart;

startAudioAtTime(track, nextBarTime);
```

---

## DEBUGGING

### Common Issues

**1. Shader Not Rendering**

Check:
- WebGPU device initialized? (`liveShaderDevices[track] !== null`)
- Pipeline created? (`liveShaderPipelines[track] !== null`)
- Canvas in DOM? (`liveShaderCanvases[track] !== null`)
- Render loop running? (check console for errors)

**2. Pop-Out Window Blank**

Check:
- Window opened successfully? (`popOutWindow && !popOutWindow.closed`)
- Pop-out devices initialized? (`popOutDevices[track] !== null`)
- Render loop rendering both inset AND pop-out?

**3. Visual Indicators Not Showing**

Check:
- CSS classes applied? (inspect element)
- CSS stacking order? (outline > border > content)
- z-index conflicts? (check shader canvas z-index)

**4. Waveform Overlay Not Visible**

Check:
- Track playing? (overlay only shows when playing)
- Canvas dimensions set? (`width` and `height` attributes)
- Overlay class `visible` added?
- Container has shader loaded?

---

## PERFORMANCE OPTIMIZATION

### GPU Usage

**Measured on M1 MacBook Pro:**
- 1 shader: ~8-12% GPU
- 2 shaders: ~16-24% GPU
- 3 shaders: ~24-36% GPU
- 4 shaders: ~32-48% GPU
- Pop-out adds ~5-10%

**Optimization Tips:**
1. Simplify shader code (fewer operations per fragment)
2. Use lower pop-out resolution (640×360 vs 1280×720)
3. Reduce FFT size if audio reactivity less critical
4. Close other GPU-intensive applications
5. Use hardware acceleration in browser settings

### Memory Usage

**Typical Session:**
- 4 loaded containers: ~40-60MB
- WebGPU buffers: ~10-20MB
- Audio buffers (4 tracks): ~40-80MB (depends on file size)
- Total: ~100-200MB

---

## MODIFICATION GUIDE

### Adding New Keyboard Shortcut

```javascript
// In handleKeyDown() function
case 'KeyN': // Example: 'N' key
    if (selectedTrack !== null) {
        // Your functionality here
        console.log('N pressed, selected track:', selectedTrack);
    }
    break;
```

### Adding New Setting

1. Add to `appSettings` object:
```javascript
let appSettings = {
    popOutWindowSize: '640x360',
    popOutAlwaysOnTop: true,
    myNewSetting: defaultValue  // ADD THIS
};
```

2. Add UI in Settings tab HTML

3. Save in `getCurrentProjectState()`:
```javascript
settings: {
    // ... existing settings
    myNewSetting: appSettings.myNewSetting
}
```

4. Load in `applyProjectState()`:
```javascript
if (projectData.settings) {
    // ... existing settings
    appSettings.myNewSetting = projectData.settings.myNewSetting || defaultValue;
}
```

### Changing Shader Canvas Resolution

```javascript
// In initLiveMixer() or similar
for (let i = 0; i < 4; i++) {
    const canvas = liveShaderCanvases[i];
    canvas.width = 1920;  // Change from 920
    canvas.height = 1080; // Change from 518
}
```

**Note:** Higher resolution = more GPU usage

---

## CODE STYLE

### Naming Conventions

**Variables:**
- camelCase: `masterTempo`, `trackGainNodes`
- Arrays end in 's': `liveShaderDevices`, `trackContainers`

**Functions:**
- camelCase: `initLiveMixer()`, `launchClipAtCell()`
- Verb-first: `updateWaveformOverlay()`, `drawZoomKnob()`

**CSS Classes:**
- kebab-case: `.clip-slot`, `.waveform-overlay`
- BEM-ish: `.waveform-overlay-name-text`

### Comments

**Do:**
```javascript
// V3_6_0_B: Four independent WebGPU systems for compositing
let liveShaderDevices = [null, null, null, null];
```

**Don't:**
```javascript
// devices
let liveShaderDevices = [null, null, null, null];
```

---

## TESTING

### Manual Test Checklist

**Multi-Shader Compositing:**
- [ ] Load 4 different shaders
- [ ] All 4 render simultaneously
- [ ] Volume faders adjust opacity
- [ ] Left track renders on top

**Pop-Out Window:**
- [ ] Opens successfully
- [ ] Canvases render in sync with inset
- [ ] Resizing works
- [ ] Always-on-top works (if supported)
- [ ] Closing cleans up resources

**Visual Indicators:**
- [ ] Cyan outline shows selected cell
- [ ] Yellow border shows playing cell
- [ ] Both visible when cell selected + playing
- [ ] Indicators don't conflict

**Waveform Overlays:**
- [ ] Visible when track playing
- [ ] Hidden when track stopped
- [ ] Shader thumbnail updates in real-time
- [ ] Long names scroll

**Keyboard Controls:**
- [ ] All keys work as documented
- [ ] DELETE clears cells completely
- [ ] No key conflicts

---

## CONTRIBUTION GUIDELINES

**Philosophy:** "Ghost Protocol" - no ongoing support

**But if you want to fork/modify:**
1. Maintain single-file architecture
2. Use underscores in version names (V3_6_0_ZA not V3.6.0_AA)
3. One feature per iteration
4. Comment iteration number for changes
5. Test on MacBook Pro 14" (1352px viewport)

---

## LICENSE

GPL-3.0 - See LICENSE section in HTML file

---

**END OF DEVELOPER DOCUMENTATION**

Version: V3.6.0  
Date: November 16, 2025  
License: GPL-3.0
