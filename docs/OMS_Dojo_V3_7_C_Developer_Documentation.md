# OMS SHADER BAKER V3.7.0 - DEVELOPER DOCUMENTATION

**Version:** V3.7.0  
**Date:** November 16, 2025  
**Purpose:** Technical reference for developers and contributors

---

## ARCHITECTURE OVERVIEW

### Single-File Philosophy

**Design Goal:** Complete functionality in one portable HTML file

**File:** OMS_ShaderBaker_V3_7_0.html  
**Size:** 525KB  
**Lines:** ~11,965  
**Dependencies:** Zero external files

**Benefits:**
- Works offline
- No build process
- No npm/node required
- Copy file = full application
- Version control friendly

---

## CORE SYSTEMS

### 1. Dual-Domain Mixer Architecture (V3.7.0)

**Philosophy:** Independent control of visual and audio domains per track

**State Arrays:**

```javascript
// AUDIO DOMAIN (V3.5.0+)
let trackGainNodes = [null, null, null, null];
let trackMute = [false, false, false, false];
let trackSolo = [false, false, false, false];

// SHADER DOMAIN (V3.7.0+)
let trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];      // Manual values (0-1)
let trackShaderMute = [false, false, false, false];  // Visual mute
let trackShaderSolo = [false, false, false, false];  // Visual solo
let trackShaderManual = [false, false, false, false]; // Override enabled?
```

**Opacity Calculation:**

```javascript
function calculateTrackShaderOpacity(track) {
    // Priority order (highest to lowest):
    
    // 1. Shader Mute
    if (trackShaderMute[track]) {
        return 0;
    }
    
    // 2. Shader Solo (any track soloed = others get 0)
    const anyShaderSoloed = trackShaderSolo.some(s => s);
    if (anyShaderSoloed && !trackShaderSolo[track]) {
        return 0;
    }
    
    // 3. Manual Override
    if (trackShaderManual[track]) {
        return trackShaderOpacity[track];
    }
    
    // 4. Volume-Based (Auto) - V3.6.0 behavior
    if (!trackAudioBuffers[track]) {
        return 0; // Not playing
    }
    
    const volume = trackGainNodes[track].gain.value;
    return Math.min(1.0, volume * 2.0); // Peaks at 50% fader
}
```

**Render Loop Integration:**

```javascript
function renderLiveShader() {
    const sharedTime = (audioContext.currentTime - masterClockStartTime) * audioPlaybackRate;
    
    for (let track = 0; track < 4; track++) {
        if (!liveShaderPipelines[track]) continue;
        
        // Calculate opacity (dual-domain logic)
        const opacity = calculateTrackShaderOpacity(track);
        liveShaderCanvases[track].style.opacity = opacity;
        
        // WebGPU rendering (same as V3.6.0)
        const commandEncoder = liveShaderDevices[track].createCommandEncoder();
        // ... standard WebGPU rendering
    }
    
    requestAnimationFrame(renderLiveShader);
}
```

**Keyboard Controls:**

```javascript
function handleKeyDown(event) {
    // Shader controls use Shift+ modifier
    switch(event.code) {
        case 'Comma':
            if (event.shiftKey) {
                toggleTrackShaderMute(selectedTrack);  // Shader mute
            } else {
                toggleTrackMute(selectedTrack);        // Audio mute
            }
            break;
            
        case 'Period':
            if (event.shiftKey) {
                toggleTrackShaderSolo(selectedTrack);  // Shader solo
            } else {
                toggleTrackSolo(selectedTrack);        // Audio solo
            }
            break;
            
        case 'ArrowUp':
            if (event.shiftKey) {
                adjustShaderOpacity(selectedTrack, 0.02);  // +2% opacity
            } else {
                increaseVolume(selectedTrack);             // Audio volume
            }
            break;
            
        case 'ArrowDown':
            if (event.shiftKey) {
                adjustShaderOpacity(selectedTrack, -0.02); // -2% opacity
            } else {
                decreaseVolume(selectedTrack);             // Audio volume
            }
            break;
    }
}
```

---

### 2. Four-Track Shader Compositing (V3.6.0)

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

**Z-Index Layering:**

```css
#liveShaderCanvas0 { z-index: 4; } /* Track 1 = top */
#liveShaderCanvas1 { z-index: 3; }
#liveShaderCanvas2 { z-index: 2; }
#liveShaderCanvas3 { z-index: 1; } /* Track 4 = bottom */
```

**Rendering:**
See V3.6.0 documentation for complete details. V3.7.0 adds opacity calculation but rendering pipeline unchanged.

---

### 3. Master Mixer System (V3.7.0)

**Master Controls:**

```javascript
// Master gain node (V3.5.0+)
let masterGainNode = null;

// Master mute/solo (V3.7.0+)
let masterMute = false;
let masterSolo = false;

function toggleMasterMute() {
    masterMute = !masterMute;
    if (masterMute) {
        masterGainNode.gain.value = 0;
    } else {
        masterGainNode.gain.value = 1.0; // Or last value
    }
    updateMasterMuteButton();
}

function toggleMasterSolo() {
    masterSolo = !masterSolo;
    // Master solo mutes all tracks
    for (let i = 0; i < 4; i++) {
        if (masterSolo) {
            trackGainNodes[i].gain.value = 0;
        } else {
            // Restore previous values
        }
    }
    updateMasterSoloButton();
}
```

**Master UI Structure:**

```html
<div class="mixer-spacer">
    <div class="spacer-label">MASTER</div>
    <div style="display: flex; gap: 10px;">
        <!-- Volume knob (60px) -->
        <canvas class="rotary-knob master-volume-knob" 
                width="60" height="60"></canvas>
        
        <div style="display: flex; flex-direction: column; gap: 5px;">
            <!-- Meters (120px tall) -->
            <div class="level-meters" style="height: 120px;">
                <!-- L/R VU meters -->
            </div>
            
            <!-- M/S Buttons -->
            <div style="display: flex; gap: 5px;">
                <button class="mixer-button master-mute-btn">M</button>
                <button class="mixer-button master-solo-btn">S</button>
            </div>
            
            <!-- Tempo Control -->
            <div class="tempo-display">
                <button class="tempo-btn">−</button>
                <span>120</span>
                <button class="tempo-btn">+</button>
            </div>
        </div>
    </div>
</div>
```

---

### 4. Rotary Knob System

**Dual Sizing (V3.7.0):**

**Track Knobs:** 30px × 30px (volume, shader opacity)  
**Master/Zoom Knobs:** 60px × 60px (global controls)

**Drawing Function:**

```javascript
function drawRotaryKnob(canvas, value, color, size) {
    const ctx = canvas.getContext('2d');
    const centerX = size / 2;
    const centerY = size / 2;
    const radius = (size / 2) - 2;
    
    // Clear
    ctx.clearRect(0, 0, size, size);
    
    // Background arc
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, 0.75 * Math.PI, 2.25 * Math.PI);
    ctx.strokeStyle = '#444';
    ctx.lineWidth = size === 60 ? 3 : 2; // Thicker for larger knobs
    ctx.stroke();
    
    // Value arc
    const startAngle = 0.75 * Math.PI;
    const endAngle = startAngle + (value * 1.5 * Math.PI);
    
    ctx.beginPath();
    ctx.arc(centerX, centerY, radius, startAngle, endAngle);
    ctx.strokeStyle = color;
    ctx.stroke();
    
    // Pointer line
    const angle = startAngle + (value * 1.5 * Math.PI);
    const pointerLength = radius * 0.6;
    const pointerX = centerX + Math.cos(angle) * pointerLength;
    const pointerY = centerY + Math.sin(angle) * pointerLength;
    
    ctx.beginPath();
    ctx.moveTo(centerX, centerY);
    ctx.lineTo(pointerX, pointerY);
    ctx.strokeStyle = color;
    ctx.lineWidth = size === 60 ? 3 : 2;
    ctx.stroke();
}
```

**Mouse Interaction:**

```javascript
// Store knob draw functions for keyboard control
const shaderKnobDrawFunctions = [];

canvas.addEventListener('mousedown', (e) => {
    const rect = canvas.getBoundingClientRect();
    const startY = e.clientY;
    const startValue = trackShaderOpacity[track];
    
    // Enable manual override on first touch
    if (!trackShaderManual[track]) {
        trackShaderManual[track] = true;
    }
    
    function onMouseMove(e) {
        const deltaY = startY - e.clientY;
        const sensitivity = canvas.width === 60 ? 0.003 : 0.006; // Larger = less sensitive
        let newValue = startValue + (deltaY * sensitivity);
        newValue = Math.max(0, Math.min(1, newValue));
        
        trackShaderOpacity[track] = newValue;
        drawRotaryKnob(canvas, newValue, '#9d4edd', canvas.width);
    }
    
    function onMouseUp() {
        document.removeEventListener('mousemove', onMouseMove);
        document.removeEventListener('mouseup', onMouseUp);
    }
    
    document.addEventListener('mousemove', onMouseMove);
    document.addEventListener('mouseup', onMouseUp);
});
```

---

### 5. Visual Indicators System (V3.6.0)

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

---

### 6. Settings Persistence (V3.6.0+)

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
        version: '3.7.0',
        masterTempo: masterTempo,
        gridData: gridData,
        
        // V3.7.0: Shader state
        trackShaderOpacity: trackShaderOpacity.slice(),
        trackShaderMute: trackShaderMute.slice(),
        trackShaderSolo: trackShaderSolo.slice(),
        trackShaderManual: trackShaderManual.slice(),
        
        // V3.6.0: Window settings
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
    // Load shader state (V3.7.0)
    if (projectData.trackShaderOpacity) {
        trackShaderOpacity = [...projectData.trackShaderOpacity];
        trackShaderMute = [...projectData.trackShaderMute];
        trackShaderSolo = [...projectData.trackShaderSolo];
        trackShaderManual = [...projectData.trackShaderManual];
        
        // Redraw knobs
        for (let i = 0; i < 4; i++) {
            if (shaderKnobDrawFunctions[i]) {
                shaderKnobDrawFunctions[i]();
            }
        }
    } else {
        // Defaults for old projects
        trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];
        trackShaderMute = [false, false, false, false];
        trackShaderSolo = [false, false, false, false];
        trackShaderManual = [false, false, false, false];
    }
    
    // Load window settings (V3.6.0)
    if (projectData.settings) {
        appSettings.popOutWindowSize = projectData.settings.popOutWindowSize || '640x360';
        appSettings.popOutAlwaysOnTop = projectData.settings.popOutAlwaysOnTop ?? true;
    }
    
    // ... rest of loading logic
}
```

---

## KEY ALGORITHMS

### Manual Override Detection

**When Knob Touched:**

```javascript
canvas.addEventListener('mousedown', (e) => {
    // First touch enables manual mode
    if (!trackShaderManual[track]) {
        trackShaderManual[track] = true;
        console.log(`Track ${track + 1} shader opacity: AUTO → MANUAL`);
    }
    // ... knob interaction code
});
```

**Resetting to Auto:**

Currently requires project reload. Planned for V3.8.0:

```javascript
// Future: Double-click to reset
canvas.addEventListener('dblclick', (e) => {
    trackShaderManual[track] = false;
    trackShaderOpacity[track] = 1.0;
    console.log(`Track ${track + 1} shader opacity: MANUAL → AUTO`);
});
```

---

## DEBUGGING

### Common Issues

**1. Shader Opacity Not Working**

Check:
- Is manual override enabled? (`trackShaderManual[track]`)
- Is shader muted? (`trackShaderMute[track]`)
- Is any track soloed? (`trackShaderSolo.some(s => s)`)
- Is track playing? (`trackAudioBuffers[track]`)

Debug:
```javascript
console.log('Track', track, {
    manual: trackShaderManual[track],
    opacity: trackShaderOpacity[track],
    mute: trackShaderMute[track],
    solo: trackShaderSolo[track],
    calculated: calculateTrackShaderOpacity(track)
});
```

**2. Keyboard Shortcuts Not Working**

Check:
- Is Shift key pressed? (`event.shiftKey`)
- Is track selected? (`selectedTrack !== null`)
- Are you in LIVE tab? (shortcuts disabled in other tabs)

**3. Knob Drawing Issues**

Check:
- Canvas dimensions set? (`canvas.width`, `canvas.height`)
- Canvas size matches code? (30px or 60px)
- Draw function called? (stored in `shaderKnobDrawFunctions[]`)

---

## PERFORMANCE OPTIMIZATION

### GPU Usage

**Measured on M1 MacBook Pro:**
- 1 shader: ~8-12% GPU
- 2 shaders: ~16-24% GPU
- 3 shaders: ~24-36% GPU
- 4 shaders: ~32-48% GPU
- Pop-out adds ~5-10%
- Dual-domain mixer: ~0% (UI only, no shader impact)

**V3.7.0 Performance:**
Same as V3.6.0 - dual-domain mixer is pure UI/state management with no GPU impact.

---

## MODIFICATION GUIDE

### Adding New Keyboard Shortcut

```javascript
// In handleKeyDown() function
case 'KeyN': // Example: 'N' key
    if (event.shiftKey) {
        // Shader-specific function
        console.log('Shift+N pressed for shader control');
    } else {
        // Audio-specific function
        console.log('N pressed for audio control');
    }
    break;
```

### Adding New Shader State

1. Add state array:
```javascript
let trackNewState = [false, false, false, false];
```

2. Save in project state:
```javascript
function getCurrentProjectState() {
    return {
        // ... existing
        trackNewState: trackNewState.slice()
    };
}
```

3. Load from project state:
```javascript
function applyProjectState(projectData) {
    if (projectData.trackNewState) {
        trackNewState = [...projectData.trackNewState];
    } else {
        trackNewState = [false, false, false, false]; // Defaults
    }
}
```

### Adding Mixer Button

```html
<!-- In track-controls div -->
<button class="mixer-button new-button" 
        onclick="handleNewButton(TRACK_INDEX)">
    X
</button>
```

```css
.new-button.active {
    background: #ff00ff; /* Your color */
}
```

```javascript
function handleNewButton(track) {
    trackNewState[track] = !trackNewState[track];
    updateNewButton(track);
}

function updateNewButton(track) {
    const btn = document.querySelector(`[data-track="${track}"] .new-button`);
    if (trackNewState[track]) {
        btn.classList.add('active');
    } else {
        btn.classList.remove('active');
    }
}
```

---

## CODE STYLE

### Naming Conventions

**Variables:**
- camelCase: `masterTempo`, `trackGainNodes`
- Arrays end in 's': `liveShaderDevices`, `trackContainers`
- Shader-specific prefix: `trackShader*` (opacity, mute, solo, manual)

**Functions:**
- camelCase: `initLiveMixer()`, `launchClipAtCell()`
- Verb-first: `calculateTrackShaderOpacity()`, `toggleTrackShaderMute()`
- Domain-specific: `trackShader*()` vs `track*()` (audio)

**CSS Classes:**
- kebab-case: `.clip-slot`, `.waveform-overlay`
- BEM-ish: `.mixer-button`, `.master-volume-knob`
- Descriptive: `.shader-opacity-knob`, `.audio-volume-knob`

### Comments

**Do:**
```javascript
// V3_7_0_J: Dual-domain mixer - independent shader/audio controls
let trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];
```

**Don't:**
```javascript
// opacity
let trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];
```

---

## TESTING

### Manual Test Checklist

**Dual-Domain Mixer:**
- [ ] Shader opacity knob adjusts visual only (audio unchanged)
- [ ] Audio volume fader adjusts audio only (visual unchanged when manual)
- [ ] Shader M/S buttons work independently from audio M/S
- [ ] Keyboard shortcuts work (Shift+ for shader, normal for audio)
- [ ] Manual override persists in project save/load
- [ ] Auto mode (volume-based) works when manual not enabled

**Master Mixer:**
- [ ] Master M/S buttons control audio output
- [ ] Tempo control adjusts BPM
- [ ] Master meters show correct levels
- [ ] Layout symmetrical with tracks

**Visual Hierarchy:**
- [ ] Track knobs are 30px
- [ ] Master/zoom knobs are 60px
- [ ] Headers all 43px tall
- [ ] Borders clean (2px modules, 0.5px internal)
- [ ] No labels visible (clean interface)

---

## CONTRIBUTION GUIDELINES

**Philosophy:** "Ghost Protocol" - no ongoing support

**But if you want to fork/modify:**
1. Maintain single-file architecture
2. Use underscores in version names (V3_7_0_ZA not V3.7.0_AA)
3. One feature per iteration
4. Comment iteration number for changes
5. Test dual-domain independence (shader vs audio)
6. Preserve backwards compatibility

---

## LICENSE

GPL-3.0 - See LICENSE section in HTML file

---

**END OF DEVELOPER DOCUMENTATION**

Version: V3.7.0  
Date: November 16, 2025  
License: GPL-3.0
