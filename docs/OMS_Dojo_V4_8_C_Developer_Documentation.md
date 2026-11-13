# ONEMANSHYO BETA V4_8_0 - DEVELOPER DOCUMENTATION

**Version:** Beta V4_8_0  
**Date:** December 19, 2025  
**Purpose:** Technical implementation reference for V4_8_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Uniform Struct Expansion](#uniform-struct-expansion)
3. [Modulator Knob Controls](#modulator-knob-controls)
4. [Reactivity Keyboard Mode](#reactivity-keyboard-mode)
5. [OmsDropdown Component](#omsdropdown-component)
6. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### Key Principle: Independent Modulator Access

V4_8_0 expands the shader uniform struct to give authors granular access to modulator components:

```
┌─────────────────────────────────────────────────────────────┐
│  MODULATOR UI (CREATE Tab)                                   │
│  Subdiv | Waveform | Gain | Freq1 | Freq2                   │
└─────────────────────┬───────────────────────────────────────┘
                      │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│ Wave Output │ │ Gain Value  │ │ FFT Level   │
│ (0-1)       │ │ (0-1)       │ │ (0-1)       │
└──────┬──────┘ └──────┬──────┘ └──────┬──────┘
       │               │               │
       ▼               ▼               ▼
┌─────────────────────────────────────────────────────────────┐
│  UNIFORM BUFFER (80 bytes)                                   │
│  modAWave | modAGain | modAFFT | modBWave | ...             │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  SHADER (WGSL)                                               │
│  uniforms.shaderModAWave, uniforms.shaderModAGain, ...      │
└─────────────────────────────────────────────────────────────┘
```

### Key Principle: Backward Compatibility

Legacy `modA/B/C` uniforms still work - they contain the combined value (Wave × Gain × FFT). New uniforms provide component access for advanced effects.

### Key Principle: Audio-Aware Controls

Freq knobs disabled when no audio loaded - FFT ranges meaningless without audio source.

---

## UNIFORM STRUCT EXPANSION

### Before V4_8_26 (48 bytes)

```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    padding1: f32,
    resolutionX: f32,
    resolutionY: f32,
    padding2: f32,
    padding3: f32,
    padding4: f32,
    padding5: f32,
}
```

### After V4_8_27 (80 bytes)

```wgsl
struct Uniforms {
    time: f32,              // Elapsed seconds
    tempo: f32,             // Current BPM
    modA: f32,              // Legacy combined (Wave × Gain × FFT)
    modB: f32,              
    modC: f32,              
    beat: f32,              // V4_8_26: Tempo-synced beat position
    resolutionX: f32,
    resolutionY: f32,
    shaderModAWave: f32,    // V4_8_27: Waveform output (0-1)
    shaderModAGain: f32,    // V4_8_27: Gain value (0-1)
    shaderModAFFT: f32,     // V4_8_27: FFT level (0-1, 0 if no audio)
    shaderModBWave: f32,    
    shaderModBGain: f32,    
    shaderModBFFT: f32,     
    shaderModCWave: f32,    
    shaderModCGain: f32,    
    shaderModCFFT: f32,     
    padding1: f32,          
    padding2: f32,          
    padding3: f32,          // 20 floats = 80 bytes
}
```

### Buffer Size Updates

All buffer creations updated from 48 to 80 bytes:

```javascript
// CREATE tab (~line 5423)
const uniformBuffer = device.createBuffer({
    size: 80,  // Was 48
    usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});

// Thumbnail renderer (~line 11403)
// PERFORM inset (~line 15095)
// PERFORM pop-out (~line 15300)
```

### Float32Array Updates

```javascript
// CREATE render loop (~line 5730)
const uniformData = new Float32Array([
    currentTime,           // 0: time
    effectiveTempo,        // 1: tempo
    modA,                  // 2: modA (legacy combined)
    modB,                  // 3: modB
    modC,                  // 4: modC
    rawBeat,               // 5: beat
    canvas.width,          // 6: resolutionX
    canvas.height,         // 7: resolutionY
    modAWave,              // 8: shaderModAWave
    modAGain,              // 9: shaderModAGain
    modAFFT,               // 10: shaderModAFFT
    modBWave,              // 11: shaderModBWave
    modBGain,              // 12: shaderModBGain
    modBFFT,               // 13: shaderModBFFT
    modCWave,              // 14: shaderModCWave
    modCGain,              // 15: shaderModCGain
    modCFFT,               // 16: shaderModCFFT
    0, 0, 0                // 17-19: padding
]);
```

### Beat Calculation (V4_8_26)

```javascript
// Calculate beat based on audio state
let beat;
if (audioLoaded && audioElement && !audioElement.paused) {
    // Audio playing - beat follows audio position
    const audioBeatsPerSecond = audioTempo / 60.0;
    beat = (audioElement.currentTime - audioStartTime) * audioBeatsPerSecond;
} else if (audioLoaded && audioElement) {
    // Audio loaded but paused - use audio tempo
    beat = currentTime * (audioTempo / 60.0);
} else {
    // No audio - use shader tempo
    beat = currentTime * (currentTempo / 60.0);
}
```

---

## MODULATOR KNOB CONTROLS

### HTML Structure

```html
<div class="control-row mod-row">
    <!-- Column 1: Subdivision dropdown -->
    <select id="modASubdiv">...</select>
    
    <!-- Column 2: Waveform dropdown -->
    <select id="modAWaveform">...</select>
    
    <!-- Column 3: Gain knob -->
    <div class="mod-knob-container">
        <canvas id="modAGainKnob" width="30" height="30"></canvas>
        <input type="hidden" id="modAGain" value="0.1">
    </div>
    
    <!-- Column 4: Freq1 knob -->
    <div class="mod-knob-container">
        <canvas id="modAFreq1Knob" width="30" height="30"></canvas>
        <input type="hidden" id="modAFreq1" value="30">
    </div>
    
    <!-- Column 5: Freq2 knob -->
    <div class="mod-knob-container">
        <canvas id="modAFreq2Knob" width="30" height="30"></canvas>
        <input type="hidden" id="modAFreq2" value="120">
    </div>
</div>
```

### CSS Layout

```css
.control-row.mod-row > *:nth-child(1) { flex: 1; }      /* Subdiv */
.control-row.mod-row > *:nth-child(2) { flex: 1; }      /* Waveform */
.control-row.mod-row > *:nth-child(3) { flex: 0 0 36px; } /* Gain */
.control-row.mod-row > *:nth-child(4) { flex: 0 0 36px; } /* Freq1 */
.control-row.mod-row > *:nth-child(5) { flex: 0 0 36px; } /* Freq2 */

.mod-knob-container {
    display: flex;
    justify-content: center;
    align-items: center;
}
```

### Gain Knob Initialization

```javascript
function initGainKnobs() {
    const knobs = [
        { canvas: 'modAGainKnob', input: 'modAGain', label: 'Mod A Gain' },
        { canvas: 'modBGainKnob', input: 'modBGain', label: 'Mod B Gain' },
        { canvas: 'modCGainKnob', input: 'modCGain', label: 'Mod C Gain' }
    ];
    
    knobs.forEach(knob => {
        const canvas = document.getElementById(knob.canvas);
        const input = document.getElementById(knob.input);
        const ctx = canvas.getContext('2d');
        
        let isDragging = false;
        let startY, startValue;
        
        function draw(value) {
            ctx.clearRect(0, 0, 30, 30);
            // Draw knob arc and indicator
            const angle = (value * 270 - 135) * Math.PI / 180;
            ctx.strokeStyle = '#9d4edd';
            ctx.lineWidth = 3;
            ctx.beginPath();
            ctx.arc(15, 15, 12, -225 * Math.PI / 180, angle);
            ctx.stroke();
            // Draw indicator line
            ctx.strokeStyle = '#fff';
            ctx.beginPath();
            ctx.moveTo(15, 15);
            ctx.lineTo(15 + Math.cos(angle) * 10, 15 + Math.sin(angle) * 10);
            ctx.stroke();
        }
        
        canvas.addEventListener('mousedown', (e) => {
            isDragging = true;
            startY = e.clientY;
            startValue = parseFloat(input.value);
        });
        
        document.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            const delta = (startY - e.clientY) * 0.005;
            const newValue = Math.max(0, Math.min(1, startValue + delta));
            input.value = newValue.toFixed(2);
            draw(newValue);
        });
        
        document.addEventListener('mouseup', () => isDragging = false);
        
        canvas.addEventListener('mouseenter', () => {
            showMessage(`${knob.label}: ${input.value} (drag ↑/↓)`, 'info');
        });
        
        // Expose update function
        window[`update${knob.input}`] = (val) => {
            input.value = val.toFixed(2);
            draw(val);
        };
        
        draw(parseFloat(input.value));
    });
}
```

### Freq Knob Log Scale

```javascript
// Frequency to knob position (0-1)
function freqToKnob(freq) {
    const minLog = Math.log(20);
    const maxLog = Math.log(24000);
    return (Math.log(freq) - minLog) / (maxLog - minLog);
}

// Knob position to frequency
function knobToFreq(knob) {
    const minLog = Math.log(20);
    const maxLog = Math.log(24000);
    return Math.round(Math.exp(minLog + knob * (maxLog - minLog)));
}
```

### Freq Knob Disabled State

```javascript
let freqKnobsDisabled = true;
let freqKnobRedrawFunctions = [];

function setFreqKnobsDisabled(disabled) {
    freqKnobsDisabled = disabled;
    freqKnobRedrawFunctions.forEach(fn => fn());
    
    // Update cursor style
    document.querySelectorAll('[id$="Freq1Knob"], [id$="Freq2Knob"]').forEach(el => {
        el.style.cursor = disabled ? 'not-allowed' : 'pointer';
    });
}

// In loadAudioFile():
setFreqKnobsDisabled(false);

// In removeAudio():
setFreqKnobsDisabled(true);
```

---

## REACTIVITY KEYBOARD MODE

### State Variables

```javascript
let modReactivityMode = false;
let modSelectedRow = 0;    // 0=A, 1=B, 2=C
let modSelectedCol = 0;    // 0=Subdiv, 1=Waveform, 2=Gain, 3=Freq1, 4=Freq2
let modKeyRepeatCount = 0;
let modLastKeyTime = 0;
```

### ID Mapping

```javascript
const MOD_ROW_IDS = ['modA', 'modB', 'modC'];
const MOD_ROW_NAMES = ['A', 'B', 'C'];
const MOD_COL_NAMES = ['Subdiv', 'Waveform', 'Gain', 'Freq1', 'Freq2'];

function getModControlId(row, col) {
    const prefix = MOD_ROW_IDS[row];
    switch(col) {
        case 0: return prefix + 'Subdiv';
        case 1: return prefix + 'Waveform';
        case 2: return prefix + 'Gain';
        case 3: return prefix + 'Freq1';
        case 4: return prefix + 'Freq2';
    }
}
```

### Selection Visual

```javascript
function updateModSelection() {
    // Remove previous selection
    document.querySelectorAll('.mod-selected').forEach(el => {
        el.classList.remove('mod-selected');
    });
    
    if (!modReactivityMode) return;
    
    // Add selection to current control
    const id = getModControlId(modSelectedRow, modSelectedCol);
    const el = document.getElementById(id);
    if (el) el.classList.add('mod-selected');
    
    // For knobs, select the canvas
    if (modSelectedCol >= 2) {
        const canvas = document.getElementById(id + 'Knob');
        if (canvas) canvas.classList.add('mod-selected');
    }
    
    showMessage(`Mod ${MOD_ROW_NAMES[modSelectedRow]} ${MOD_COL_NAMES[modSelectedCol]} (↑↓←→ nav, ⇧←→ adjust, Esc exit)`, 'info');
}
```

### Value Adjustment with Progressive Acceleration

```javascript
function adjustModValue(delta) {
    if (!modReactivityMode) return;
    
    const id = getModControlId(modSelectedRow, modSelectedCol);
    const el = document.getElementById(id);
    if (!el) return;
    
    // Progressive acceleration
    const now = Date.now();
    if (now - modLastKeyTime < 150) {
        modKeyRepeatCount = Math.min(modKeyRepeatCount + 1, 20);
    } else {
        modKeyRepeatCount = 0;
    }
    modLastKeyTime = now;
    const accel = 1 + modKeyRepeatCount * 0.5;
    
    // Dropdowns (col 0-1)
    if (modSelectedCol <= 1) {
        const newIndex = el.selectedIndex + delta;
        if (newIndex >= 0 && newIndex < el.options.length) {
            el.selectedIndex = newIndex;
            el.dispatchEvent(new Event('change'));
        }
    }
    // Gain knob (col 2)
    else if (modSelectedCol === 2) {
        let val = parseFloat(el.value);
        if (isNaN(val)) val = 0.1;
        const step = 0.01 * accel;
        val = Math.max(0, Math.min(1, val + delta * step));
        el.value = val.toFixed(2);
        window[`update${id}`]?.(val);
    }
    // Freq knobs (col 3-4)
    else {
        let val = parseInt(el.value) || 100;
        const ratio = 1.1 + (modKeyRepeatCount * 0.05);
        val = delta > 0 
            ? Math.min(24000, Math.round(val * ratio))
            : Math.max(20, Math.round(val / ratio));
        el.value = val;
        window[`update${id}`]?.(val);
        
        // Push behavior
        // ... (Freq1 can't exceed Freq2)
    }
}
```

### Keyboard Handler

```javascript
document.addEventListener('keydown', (e) => {
    if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
    
    // Shift+R: Toggle reactivity mode
    if (e.shiftKey && (e.key === 'r' || e.key === 'R')) {
        e.preventDefault();
        modReactivityMode = !modReactivityMode;
        if (!modReactivityMode) exitReactivityMode();
        else updateModSelection();
        return;
    }
    
    // Escape: Exit mode
    if (e.key === 'Escape' && modReactivityMode) {
        e.preventDefault();
        exitReactivityMode();
        return;
    }
    
    if (!modReactivityMode) return;
    
    // Shift+Left/Right: Adjust value
    if (e.shiftKey && (e.key === 'ArrowLeft' || e.key === 'ArrowRight')) {
        e.preventDefault();
        adjustModValue(e.key === 'ArrowRight' ? 1 : -1);
        return;
    }
    
    // Left/Right: Navigate columns
    if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
        e.preventDefault();
        modSelectedCol = e.key === 'ArrowLeft' 
            ? (modSelectedCol - 1 + 5) % 5 
            : (modSelectedCol + 1) % 5;
        updateModSelection();
        return;
    }
    
    // Up/Down: Navigate rows
    if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
        e.preventDefault();
        modSelectedRow = e.key === 'ArrowUp'
            ? (modSelectedRow - 1 + 3) % 3
            : (modSelectedRow + 1) % 3;
        updateModSelection();
        return;
    }
});
```

---

## OMSDROPDOWN COMPONENT

### Constructor

```javascript
class OmsDropdown {
    constructor(containerId, options = {}) {
        this.container = document.getElementById(containerId);
        this.options = {
            showThumbnails: false,
            thumbnailSize: { w: 32, h: 18 },
            onChange: null,
            onOpen: null,
            ...options
        };
        this.items = [];
        this.selectedValue = null;
        this.isOpen = false;
        this.highlightedIndex = -1;
        this.thumbnailsRendered = false;
        
        this.render();
        this.bindEvents();
    }
}
```

### API Methods

```javascript
// Set items
dropdown.setItems([
    { value: 'id1', label: 'Name 1', shaderCode: '...' },
    { separator: true, label: '── Section ──' },
    { value: 'id2', label: 'Name 2', shaderCode: '...' }
]);

// Get/Set value
dropdown.setValue('id1');
const current = dropdown.getValue();

// Programmatic open/close
dropdown.open();
dropdown.close();
```

### DOM Structure

```html
<div class="oms-dropdown">
    <div class="oms-dropdown-header">
        <canvas class="oms-dropdown-tn" width="32" height="18"></canvas>
        <span class="oms-dropdown-label">Selected Item</span>
        <span class="oms-dropdown-arrow">▼</span>
    </div>
    <div class="oms-dropdown-list">
        <div class="oms-dropdown-sep">── Section ──</div>
        <div class="oms-dropdown-item" data-value="id1">
            <canvas class="oms-dropdown-tn"></canvas>
            <span>Item Name</span>
        </div>
    </div>
</div>
```

### Thumbnail Rendering

```javascript
async renderThumbnails() {
    if (this.thumbnailsRendered) return;
    
    const items = this.container.querySelectorAll('.oms-dropdown-item');
    let count = 0;
    
    for (const item of items) {
        const canvas = item.querySelector('.oms-dropdown-tn');
        const shaderCode = item.dataset.shaderCode;
        
        if (canvas && shaderCode) {
            await initThumbnailShader(canvas, shaderCode);
        }
        
        // Yield every 3 to keep UI responsive
        if (++count % 3 === 0) {
            await new Promise(r => setTimeout(r, 0));
        }
    }
    
    this.thumbnailsRendered = true;
}
```

### onOpen Callback

```javascript
async open() {
    if (this.isOpen) return;
    
    // Call onOpen callback (refresh items)
    if (this.options.onOpen) {
        await this.options.onOpen();
    }
    
    // Reset thumbnail state for fresh render
    this.thumbnailsRendered = false;
    
    this.isOpen = true;
    this.container.classList.add('open');
    
    // Render thumbnails after open
    if (this.options.showThumbnails) {
        this.renderThumbnails();
    }
}
```

---

## TESTING NOTES

### Uniform Tests

1. **uniforms.beat:** Create shader using `fract(uniforms.beat)` - should pulse on beat
2. **Component uniforms:** Use test shader to visualize all 9 new uniforms
3. **Legacy compatibility:** Old shaders using modA/B/C should still work

### Knob Tests

1. **Gain knobs:** Drag up/down, verify 0-1 range
2. **Freq knobs:** Verify log scale (most travel in bass frequencies)
3. **Push behavior:** Turn Freq1 past Freq2, verify Freq2 follows
4. **Disabled state:** Remove audio, verify freq knobs dimmed

### Reactivity Mode Tests

1. **Enter/Exit:** Shift+R toggles mode, Escape exits
2. **Navigation:** Arrow keys move through 3×5 grid
3. **Value adjust:** Shift+Arrow changes values
4. **Acceleration:** Hold key, verify faster changes
5. **Visual:** Cyan ring follows selection

### OmsDropdown Tests

1. **Open/Close:** Click header, click outside
2. **Keyboard:** Arrow keys, Enter, Escape
3. **Thumbnails:** Verify lazy render on open
4. **Refresh:** Add .wgsl to /Presets/, reopen dropdown

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V4_8_0
Date: December 19, 2025
License: GPL-3.0
