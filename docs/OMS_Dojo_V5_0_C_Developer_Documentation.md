# ONEMANSHYO App [Beta V5_0_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_0_0  
**Date:** December 21, 2025  
**Purpose:** Technical implementation reference for V5_0_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [CORTEX Shader Contract](#cortex-shader-contract)
3. [Uniform Struct (112 bytes)](#uniform-struct-112-bytes)
4. [Transform Module](#transform-module)
5. [Unified Wheel Handler](#unified-wheel-handler)
6. [OMS Design System](#oms-design-system)
7. [OmsDropdown Component](#omsdropdown-component)
8. [Reactivity Keyboard Mode (V4_8_0)](#reactivity-keyboard-mode-v4_8_0)
9. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_0_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  CORTEX SHADER CONTRACT (V4_9_15)                                    │
│  Single source of truth for uniform struct                          │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ SHADER_UNIFORM_STRUCT │ UNIFORM_BUFFER_SIZE │ createUniformData │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
     ┌───────────────────────┼───────────────────────┐
     ▼                       ▼                       ▼
┌──────────┐          ┌──────────┐          ┌──────────┐
│ CREATE   │          │Thumbnails│          │  LIVE    │
│ Renderer │          │ Renderer │          │ Renderer │
└──────────┘          └──────────┘          └──────────┘
```

### Key Principles

1. **CORTEX Consolidation** - Shader contract defined once, used everywhere
2. **Transform Uniforms** - 8 new floats for geometric manipulation
3. **Design System** - Universal CSS components eliminate duplication
4. **Unified Interactions** - Single wheel handler for all controls

---

## CORTEX SHADER CONTRACT

### Problem Solved (V4_9_15)

Before V4_9_15, the shader uniform contract was scattered:
- 3 identical WGSL struct definitions (CREATE, thumbnails, LIVE)
- 4 identical Float32Array write patterns
- Buffer sizes hardcoded in multiple spots
- Adding a uniform required changes in ~6 locations

### Solution: Three CORTEX Constants

**Location:** Top of `<script>`, lines ~100-180

```javascript
// ============================================
// CORTEX - SHADER CONTRACT
// Change = cascade failure across all rendering
// ============================================

// 1. WGSL Uniform Struct Definition
const SHADER_UNIFORM_STRUCT = `
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    beat: f32,
    resolutionX: f32,
    resolutionY: f32,
    shaderModAWave: f32,
    shaderModAGain: f32,
    shaderModAFFT: f32,
    shaderModBWave: f32,
    shaderModBGain: f32,
    shaderModBFFT: f32,
    shaderModCWave: f32,
    shaderModCGain: f32,
    shaderModCFFT: f32,
    transformRotation: f32,
    transformScaleX: f32,
    transformScaleY: f32,
    transformTranslateX: f32,
    transformTranslateY: f32,
    transformPivotX: f32,
    transformPivotY: f32,
    transformTile: f32,
    padding1: f32,
    padding2: f32,
    padding3: f32,
}
@group(0) @binding(0) var<uniform> uniforms: Uniforms;
`;

// 2. Buffer Size (MUST match struct)
const UNIFORM_BUFFER_SIZE = 112;  // 28 floats × 4 bytes

// 3. Factory Function (order MUST match WGSL exactly)
function createUniformData(p) {
    return new Float32Array([
        p.time ?? 0.0,
        p.tempo ?? 120.0,
        p.modA ?? 0.0,
        p.modB ?? 0.0,
        p.modC ?? 0.0,
        p.beat ?? 0.0,
        p.resolutionX ?? 1920,
        p.resolutionY ?? 1080,
        p.shaderModAWave ?? 0.0,
        p.shaderModAGain ?? 0.1,
        p.shaderModAFFT ?? 0.0,
        p.shaderModBWave ?? 0.0,
        p.shaderModBGain ?? 0.1,
        p.shaderModBFFT ?? 0.0,
        p.shaderModCWave ?? 0.0,
        p.shaderModCGain ?? 0.1,
        p.shaderModCFFT ?? 0.0,
        p.transformRotation ?? 0.0,
        p.transformScaleX ?? 1.0,
        p.transformScaleY ?? 1.0,
        p.transformTranslateX ?? 0.0,
        p.transformTranslateY ?? 0.0,
        p.transformPivotX ?? 0.0,
        p.transformPivotY ?? 0.0,
        p.transformTile ?? 1.0,
        0.0, 0.0, 0.0  // padding
    ]);
}
```

### Usage Pattern

```javascript
// CREATE tab shader compilation
const fullShaderCode = SHADER_UNIFORM_STRUCT + userShaderCode;

// Buffer creation (all locations)
const uniformBuffer = device.createBuffer({
    size: UNIFORM_BUFFER_SIZE,
    usage: GPUBufferUsage.UNIFORM | GPUBufferUsage.COPY_DST
});

// Render loop (all locations)
const uniformData = createUniformData({
    time: currentTime,
    tempo: effectiveTempo,
    modA, modB, modC,
    beat: rawBeat,
    resolutionX: canvas.width,
    resolutionY: canvas.height,
    shaderModAWave: modAWave,
    // ... all values
});
device.queue.writeBuffer(uniformBuffer, 0, uniformData);
```

### Adding a New Uniform (4 Steps)

1. Add field to `SHADER_UNIFORM_STRUCT` (in WGSL block)
2. Update `UNIFORM_BUFFER_SIZE` (count floats × 4)
3. Add to `createUniformData()` with default value
4. Update controlStates save/load if persistent

---

## UNIFORM STRUCT (112 BYTES)

### Buffer Size Progression

| Version | Floats | Bytes | Added |
|---------|--------|-------|-------|
| V4_8_0 | 20 | 80 | Base (modulators) |
| V4_9_07 | 24 | 96 | +5 transform, +2 padding |
| V4_9_13 | 28 | 112 | +tile, +3 padding |

### Complete Struct (28 floats)

```wgsl
struct Uniforms {
    // Core Timing (8 floats)
    time: f32,              // 0: Elapsed seconds
    tempo: f32,             // 1: Current BPM
    modA: f32,              // 2: Legacy combined (Wave × Gain × FFT)
    modB: f32,              // 3: Legacy combined
    modC: f32,              // 4: Legacy combined
    beat: f32,              // 5: Tempo-synced beat position
    resolutionX: f32,       // 6: Canvas width
    resolutionY: f32,       // 7: Canvas height
    
    // Decoupled Modulators (9 floats)
    shaderModAWave: f32,    // 8: Waveform A output (0-1)
    shaderModAGain: f32,    // 9: Gain A value (0-1)
    shaderModAFFT: f32,     // 10: FFT A level (0-1)
    shaderModBWave: f32,    // 11: Waveform B output
    shaderModBGain: f32,    // 12: Gain B value
    shaderModBFFT: f32,     // 13: FFT B level
    shaderModCWave: f32,    // 14: Waveform C output
    shaderModCGain: f32,    // 15: Gain C value
    shaderModCFFT: f32,     // 16: FFT C level
    
    // Transform Controls (8 floats)
    transformRotation: f32, // 17: Rotation in RADIANS
    transformScaleX: f32,   // 18: Scale X (default 1.0)
    transformScaleY: f32,   // 19: Scale Y (default 1.0)
    transformTranslateX: f32, // 20: Translate X (-1 to 1)
    transformTranslateY: f32, // 21: Translate Y (-1 to 1)
    transformPivotX: f32,   // 22: Pivot X (-1 to 1)
    transformPivotY: f32,   // 23: Pivot Y (-1 to 1)
    transformTile: f32,     // 24: Tile count (1-8)
    
    // Padding (3 floats)
    padding1: f32,          // 25: GPU alignment
    padding2: f32,          // 26
    padding3: f32,          // 27
}
```

### UI-to-Uniform Conversion

| UI Control | UI Value | Uniform Value |
|------------|----------|---------------|
| Rotation encoder | 0-360° | radians (× π/180) |
| Scale encoder | 0-4x | direct |
| Tile encoder | 1-8 | direct |
| XY pad | -1 to 1 | direct |
| Pivot pad | -1 to 1 | direct |

---

## TRANSFORM MODULE

### HTML Structure

```html
<div class="control-row oms-control-box" id="transformControlsRow">
    <span style="color:#9d4edd;">TRANSFORMS</span>
    
    <!-- Rotation (endless encoder) -->
    <canvas id="transformRotationKnob" width="30" height="30"></canvas>
    <input type="hidden" id="transformRotation" value="0">
    
    <!-- Scale (combo encoder → writes ScaleX + ScaleY) -->
    <canvas id="transformScaleKnob" width="30" height="30"></canvas>
    <input type="hidden" id="transformScaleX" value="1">
    <input type="hidden" id="transformScaleY" value="1">
    
    <!-- Tile (encoder) -->
    <canvas id="transformTileKnob" width="30" height="30"></canvas>
    <input type="hidden" id="transformTile" value="1">
    
    <!-- Translate XY (50×30 pad) -->
    <span>Translate</span>
    <canvas id="transformXYPad" width="50" height="30"></canvas>
    <input type="hidden" id="transformTranslateX" value="0">
    <input type="hidden" id="transformTranslateY" value="0">
    
    <!-- Pivot XY (50×30 pad) -->
    <span>Pivot</span>
    <canvas id="transformPivotPad" width="50" height="30"></canvas>
    <input type="hidden" id="transformPivotX" value="0">
    <input type="hidden" id="transformPivotY" value="0">
</div>
```

### Global State

```javascript
// Transform keyboard mode (Shift+T)
let transformKeyboardMode = false;
let transformSelectedCol = 0;       // 0=ROT, 1=SCALE, 2=TILE, 3=XY, 4=PIVOT
let transformKeyRepeatCount = 0;
let transformLastKeyTime = 0;
let transformLastEnterTime = 0;     // For double-tap reset

// Config arrays
const TRANSFORM_COL_IDS = ['transformRotation', 'transformScale', 'transformTile', 'transformXY', 'transformPivot'];
const TRANSFORM_KNOB_IDS = ['transformRotationKnob', 'transformScaleKnob', 'transformTileKnob', 'transformXYPad', 'transformPivotPad'];
const TRANSFORM_COL_NAMES = ['Rotation', 'Scale', 'Tile', 'Translate', 'Pivot'];
const TRANSFORM_CONFIGS = [
    { min: 0, max: 360, step: 1, default: 0, endless: true },
    { min: 0, max: 4, step: 0.01, default: 1 },
    { min: 1, max: 8, step: 0.1, default: 1 },
    { min: -1, max: 1, step: 0.01, default: 0 },
    { min: -1, max: 1, step: 0.01, default: 0 }
];
```

### Initialization Pattern

```javascript
function initTransformEncoders() {
    const configs = [
        { id: 'transformRotation', knobId: 'transformRotationKnob', 
          min: 0, max: 360, default: 0, step: 1, endless: true, 
          label: 'Rotation', color: '#9d4edd' },
        { id: 'transformScale', knobId: 'transformScaleKnob',
          min: 0, max: 4, default: 1, step: 0.01,
          label: 'Scale', color: '#9d4edd' },
        // ... etc
    ];
    
    configs.forEach(config => {
        const canvas = document.getElementById(config.knobId);
        if (!canvas) return;
        
        let value = config.default;
        const ctx = canvas.getContext('2d');
        
        function drawTransformKnob() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            const normalized = (value - config.min) / (config.max - config.min);
            
            if (config.endless) {
                // Full 360° circle with indicator line
                ctx.strokeStyle = '#333';
                ctx.beginPath();
                ctx.arc(15, 15, 11, 0, Math.PI * 2);
                ctx.stroke();
                
                const angle = Math.PI / 2 + normalized * Math.PI * 2;
                ctx.strokeStyle = config.color;
                ctx.beginPath();
                ctx.moveTo(15, 15);
                ctx.lineTo(15 + Math.cos(angle) * 10, 15 + Math.sin(angle) * 10);
                ctx.stroke();
            } else {
                // Standard arc (7 o'clock to 5 o'clock)
                // ... arc drawing code
            }
        }
        
        // Expose update function globally
        window[`update${config.id}`] = (newVal) => {
            value = config.endless
                ? ((newVal - config.min) % (config.max - config.min) + (config.max - config.min)) % (config.max - config.min) + config.min
                : Math.max(config.min, Math.min(config.max, newVal));
            drawTransformKnob();
            // Update hidden inputs...
        };
    });
}
```

### Keyboard Handler (Shift+T)

```javascript
document.addEventListener('keydown', (e) => {
    // Shift+T: Toggle transform mode
    if (e.shiftKey && (e.key === 't' || e.key === 'T')) {
        e.preventDefault();
        transformKeyboardMode = !transformKeyboardMode;
        if (!transformKeyboardMode) exitTransformMode();
        else updateTransformSelection();
        return;
    }
    
    if (!transformKeyboardMode) return;
    
    // Left/Right: Navigate controls
    if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
        e.preventDefault();
        transformSelectedCol = e.key === 'ArrowLeft'
            ? (transformSelectedCol - 1 + 5) % 5
            : (transformSelectedCol + 1) % 5;
        updateTransformSelection();
        return;
    }
    
    // Shift+Left/Right: Adjust encoder values
    if (e.shiftKey && (e.key === 'ArrowLeft' || e.key === 'ArrowRight')) {
        e.preventDefault();
        const delta = e.key === 'ArrowRight' ? 1 : -1;
        adjustTransformValue(delta);
        return;
    }
    
    // Shift+Arrow for XY/Pivot pads
    if (e.shiftKey && (e.key === 'ArrowUp' || e.key === 'ArrowDown')) {
        // ... XY adjustment code
    }
    
    // Shift+Enter×2: Reset to default
    if (e.key === 'Enter' && e.shiftKey) {
        e.preventDefault();
        const now = Date.now();
        if (now - transformLastEnterTime < 300) {
            resetTransformControl(transformSelectedCol);
            transformLastEnterTime = 0;
        } else {
            transformLastEnterTime = now;
        }
    }
});
```

---

## UNIFIED WHEEL HANDLER

### Architecture (V4_9_19-20, V4_9_35)

Single document-level handler for all scroll controls:

```javascript
// V4_9_35: Unified wheel handler for ALL CREATE tab controls
document.addEventListener('wheel', (e) => {
    // Only in CREATE tab
    const createTab = document.getElementById('controlsTabContent');
    if (!createTab || createTab.style.display === 'none') return;
    
    // Require Shift key
    if (!e.shiftKey) return;
    
    // Don't interfere with inputs
    if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
    
    const target = e.target;
    let activeControlId = null;
    
    // Transform controls
    const transformIds = ['transformRotationKnob', 'transformScaleKnob', 
                          'transformTileKnob', 'transformXYPad', 'transformPivotPad'];
    
    // Reactivity knobs
    const reactivityKnobIds = [
        'modAGainKnob', 'modBGainKnob', 'modCGainKnob',
        'modAFreq1Knob', 'modBFreq1Knob', 'modCFreq1Knob',
        'modAFreq2Knob', 'modBFreq2Knob', 'modCFreq2Knob'
    ];
    
    // Method 1: Direct hover
    if (transformIds.includes(target.id)) {
        activeControlId = target.id;
    } else if (reactivityKnobIds.includes(target.id)) {
        activeControlId = target.id;
    }
    // Method 2: Keyboard mode selection
    else if (transformKeyboardMode) {
        activeControlId = TRANSFORM_KNOB_IDS[transformSelectedCol];
    } else if (modReactivityMode && modSelectedCol >= 2) {
        activeControlId = getModKnobCanvasId(modSelectedRow, modSelectedCol);
    }
    
    if (!activeControlId) return;
    e.preventDefault();
    
    // Route by control type
    const delta = -e.deltaY;  // Scroll up = increase
    
    if (activeControlId === 'transformRotationKnob') {
        // Endless 0-360°
        let val = parseFloat(document.getElementById('transformRotation').value) || 0;
        val = ((val + delta * 0.1) % 360 + 360) % 360;
        window.updatetransformRotation?.(val);
    }
    else if (activeControlId === 'transformScaleKnob') {
        // Linear 0-4
        let val = parseFloat(document.getElementById('transformScaleX').value) || 1;
        val = Math.max(0, Math.min(4, val + delta * 0.001));
        window.updatetransformScale?.(val);
    }
    else if (activeControlId.includes('Gain')) {
        // Linear 0-1
        const inputId = activeControlId.replace('Knob', '');
        let val = parseFloat(document.getElementById(inputId).value) || 0.1;
        val = Math.max(0, Math.min(1, val + delta * 0.002));
        window[`update${inputId}`]?.(val);
    }
    else if (activeControlId.includes('Freq')) {
        // Log scale 20-24000 Hz
        const inputId = activeControlId.replace('Knob', '');
        let val = parseInt(document.getElementById(inputId).value) || 100;
        const ratio = delta > 0 ? 1.05 : 1/1.05;
        val = Math.max(20, Math.min(24000, Math.round(val * ratio)));
        window[`update${inputId}`]?.(val);
    }
    // ... XY pads use deltaX + deltaY
    
}, { passive: false });
```

### Sensitivity Reference

| Control | Input | Sensitivity | Range |
|---------|-------|-------------|-------|
| Rotation | deltaY | 0.1° per unit | 0-360° (wrap) |
| Scale | deltaY | 0.001 per unit | 0-4 |
| Tile | deltaY | 0.01 per unit | 1-8 |
| XY Pads | deltaX + deltaY | 0.002 per unit | -1 to 1 |
| Gain | deltaY | 0.002 per unit | 0-1 |
| Freq | deltaY | ×1.05 ratio | 20-24000 Hz |

---

## OMS DESIGN SYSTEM

### Design Tokens

```css
/* ============================================
   OMS DESIGN SYSTEM - CORTEX VALUES
   Change = cascade to ALL UI elements
   ============================================ */

/* Surfaces */
--oms-surface-bg: rgba(255,255,255,0.05);
--oms-surface-hover: rgba(255,255,255,0.08);  /* +0.03 */

/* Focus */
--oms-focus-ring: 0 0 0 1px rgba(0, 212, 255, 0.5);

/* Sizing */
--oms-control-height: 34px;
--oms-control-padding: 6px 10px;
--oms-border-radius: 0;  /* Sharp corners throughout */
```

### Universal Components

#### `.oms-header-bar`

```css
.oms-header-bar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: rgba(255,255,255,0.05);
    padding: 12px 15px;
    border-radius: 0;
    margin-top: 5px;
    margin-bottom: 15px;
}

.oms-header-bar.no-margin-bottom {
    margin-bottom: 0;
}
```

**Usage:** Section headers (SHADER CONTROLS, AUDIO CONTROLS, SET, LIBRARY, ADMIN)

#### `.oms-field`

```css
.oms-field {
    background: rgba(255,255,255,0.05);
    border: none;
    border-radius: 0;
    color: #fff;
    padding: 6px 10px;
    font-size: 13px;
    min-height: 34px;
}

.oms-field:hover {
    background: rgba(255,255,255,0.08);
}

.oms-field:focus {
    outline: none;
    box-shadow: 0 0 0 1px rgba(0, 212, 255, 0.5);
}
```

**Usage:** Form inputs, native selects

#### `.oms-control-box`

```css
.oms-control-box {
    display: flex;
    align-items: center;
    background: rgba(255,255,255,0.05);
    border: none;
    border-radius: 0;
    padding: 2px 6px;
    gap: 5px;
}
```

**Usage:** Grouped controls (Transforms row)

#### `.mod-knob-box`

```css
.mod-knob-box {
    display: flex;
    gap: 5px;
    background: rgba(255,255,255,0.05);
    border: none;
    border-radius: 0;
    padding: 0 6px;
    height: 34px;
    align-items: center;
}
```

**Usage:** Modulator knobs (Gain/Freq1/Freq2)

---

## OMSDROPDOWN COMPONENT

### V5_0_0 Expansion

| Version | Dropdowns | Notes |
|---------|-----------|-------|
| V4_8_0 | 1 | shaderNameDropdown only |
| V5_0_0 | 11 | All CREATE tab selects |

### Full List

```javascript
// Shader Controls
window.shaderNameDropdown    // With thumbnails
window.orientationDropdown   // Landscape/Portrait/Square

// Timing
window.loopLengthDropdown    // Bar lengths + durations

// Reactivity (×6)
window.modASubdivDropdown
window.modAWaveformDropdown
window.modBSubdivDropdown
window.modBWaveformDropdown
window.modCSubdivDropdown
window.modCWaveformDropdown

// Output
window.resolutionDropdown    // 720p/1080p/1440p/2160p
window.fpsDropdown           // 24/30/60
```

### Initialization Pattern

```javascript
function initCreateTabDropdowns() {
    function createSyncedDropdown(dropdownId, hiddenId, options, onChange) {
        const dropdown = new OmsDropdown(dropdownId, {
            showThumbnails: false,
            onChange: (value, item) => {
                document.getElementById(hiddenId).value = value;
                if (onChange) onChange(value, item);
            }
        });
        dropdown.setItems(options);
        dropdown.setValue(document.getElementById(hiddenId).value);
        return dropdown;
    }
    
    // Orientation
    window.orientationDropdown = createSyncedDropdown('orientationDropdown', 'orientation', [
        { value: 'landscape', label: 'Landscape' },
        { value: 'portrait', label: 'Portrait' },
        { value: 'square', label: 'Square' }
    ], () => updateCanvasSize());
    
    // ... repeat for all dropdowns
}
```

### Keyboard Mode Integration (V4_9_34)

When replacing native `<select>` with OmsDropdown, keyboard handlers must use dropdown API:

```javascript
// OLD (native select)
el.selectedIndex += delta;
el.dispatchEvent(new Event('change'));

// NEW (OmsDropdown)
const dropdown = window.modASubdivDropdown;
const currentValue = dropdown.getValue();
const selectableItems = dropdown.items.filter(item => !item.separator);
let currentIndex = selectableItems.findIndex(item => item.value === currentValue);
let newIndex = (currentIndex + delta + selectableItems.length) % selectableItems.length;
dropdown.setValue(selectableItems[newIndex].value);
```

### Helper Function

```javascript
function getModDropdown(row, col) {
    const rowLetter = ['A', 'B', 'C'][row];
    if (col === 0) return window[`mod${rowLetter}SubdivDropdown`];
    if (col === 1) return window[`mod${rowLetter}WaveformDropdown`];
    return null;
}
```

---

## REACTIVITY KEYBOARD MODE (V4_8_0)

*Inherited from V4_8_0, updated for OmsDropdown.*

### State Variables

```javascript
let modReactivityMode = false;
let modSelectedRow = 0;      // 0=A, 1=B, 2=C
let modSelectedCol = 0;      // 0=Subdiv, 1=Waveform, 2=Gain, 3=Freq1, 4=Freq2
let modKeyRepeatCount = 0;
let modLastKeyTime = 0;
```

### Selection Ring

```javascript
function updateModSelection() {
    // Clear previous
    document.querySelectorAll('.mod-selected').forEach(el => el.classList.remove('mod-selected'));
    
    const col = modSelectedCol;
    const row = modSelectedRow;
    
    if (col <= 1) {
        // OmsDropdown - target header element
        const dropdown = getModDropdown(row, col);
        if (dropdown?.header) dropdown.header.classList.add('mod-selected');
    } else {
        // Knob canvas
        const canvasId = getModKnobCanvasId(row, col);
        const canvas = document.getElementById(canvasId);
        if (canvas) canvas.classList.add('mod-selected');
    }
}
```

---

## TESTING NOTES

### CORTEX Contract Tests

1. **Struct consistency:** All 3 renderers (CREATE, thumbnails, LIVE) should produce identical visuals
2. **Buffer size:** Mismatch causes GPU crash - verify 112 bytes everywhere
3. **Order match:** Float32Array order must exactly match WGSL struct order

### Transform Tests

1. **Rotation wrap:** 359° + 2° = 1° (not 361°)
2. **Scale reset:** Shift+Enter×2 resets to 1.0
3. **Degrees to radians:** UI shows degrees, uniform receives radians
4. **Shift+trackpad:** All 5 controls respond

### OmsDropdown Tests

1. **Keyboard mode:** Shift+R → navigate to dropdown → Shift+Arrow cycles options
2. **Selection ring:** Cyan ring on `.oms-dropdown-header`
3. **Value sync:** Dropdown selection updates hidden input
4. **Height:** All dropdowns 34px

### Wheel Handler Tests

1. **Shift required:** No Shift = no adjustment
2. **Both modes:** Hover + scroll AND keyboard-mode + scroll
3. **All 14 controls:** 5 transforms + 9 reactivity knobs

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_0_0
Date: December 21, 2025
License: GPL-3.0
