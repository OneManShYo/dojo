# ONEMANSHYO App [Beta V5_0_0] - COMPLETE CHANGELOG

**Version:** Beta V5_0_0  
**Release Date:** December 21, 2025  
**Base Version:** V4_8_0  
**Development Iterations:** 36 sub-iterations (V4_9_00 through V4_9_35)

---

## EXECUTIVE SUMMARY

Beta V5_0_0 is a major release introducing the TRANSFORMS module, CORTEX shader contract consolidation, and OMS Design System. The CREATE tab now features 5 transform controls (Rotation, Scale, Tile, XY Translate, Pivot) with full keyboard and trackpad support. The shader uniform system was refactored from 6 scattered definitions into 3 CORTEX constants, and a comprehensive design system unifies all UI components.

**Key Achievements:**
1. **Transform uniforms** - 8 new GPU uniforms for geometric manipulation
2. **CORTEX consolidation** - Single source of truth for shader contract (was 6 locations)
3. **Design System** - Universal CSS components eliminating inline style duplication
4. **OmsDropdown** - Unified dropdown component replacing all native selects in CREATE tab

---

## MAJOR FEATURES

### 1. TRANSFORMS Module (V4_9_01-14)

✅ **5 Transform Controls**
| Control | Type | Range | Default |
|---------|------|-------|---------|
| Rotation | Encoder (endless) | 0-360° | 0° |
| Scale | Encoder | 0-4x | 1.0x |
| Tile | Encoder | 1-8 | 1 |
| XY (Translate) | 50×30 Pad | -1 to 1 | 0,0 |
| Pivot | 50×30 Pad | -1 to 1 | 0,0 |

✅ **Keyboard Navigation (Shift+T)**
| Key | Action |
|-----|--------|
| Shift+T | Enter/exit transform mode |
| ← / → | Navigate between controls |
| Shift+← / → | Adjust encoder values |
| Shift+Arrows | Adjust XY/Pivot pads |
| Shift+Enter×2 | Reset to default (double-tap) |
| Escape | Exit transform mode |

✅ **Trackpad Control (V4_9_19-20)**
- Shift + hover + scroll = adjust any control
- Shift + in keyboard mode + scroll = adjust selected control

### 2. CORTEX Shader Contract Consolidation (V4_9_15) ⚠️ CRITICAL

**Problem:** Shader uniform contract was scattered across 6+ locations:
- 3 identical WGSL struct definitions (CREATE, thumbnails, LIVE)
- 4 identical Float32Array write patterns
- Buffer sizes hardcoded in multiple spots

Every uniform addition required hunting through ~20,000 lines finding all locations.

**Solution:** Three CORTEX constants at top of `<script>`:

```javascript
// CORTEX - Single source of truth for shader contract
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

const UNIFORM_BUFFER_SIZE = 112;  // 28 floats × 4 bytes

function createUniformData(p) {
    return new Float32Array([
        p.time ?? 0.0,
        p.tempo ?? 120.0,
        // ... all 28 fields with defaults
    ]);
}
```

**Locations Updated:**
| Location | Before | After |
|----------|--------|-------|
| CREATE struct | 30-line inline | `${SHADER_UNIFORM_STRUCT}` |
| Thumbnails struct | 30-line inline | `${SHADER_UNIFORM_STRUCT}` |
| LIVE container struct | 30-line inline | `${SHADER_UNIFORM_STRUCT}` |
| CREATE buffer size | `size: 112` | `size: UNIFORM_BUFFER_SIZE` |
| Thumbnails buffer size | `size: 96` ❌ BUG | `size: UNIFORM_BUFFER_SIZE` ✓ |
| All Float32Array writes | 30-line inline each | `createUniformData({...})` |

**Bug Fixed:** Thumbnails had 96 bytes when struct needed 112 bytes!

**Adding a New Uniform (Now vs Before):**
- Before: 10 steps across 6+ files, easy to miss one
- After: 4 steps in 1 location

### 3. GPU Uniform Expansion (V4_9_07 → V4_9_13)

**Buffer Size Progression:**
| Version | Floats | Bytes | Added |
|---------|--------|-------|-------|
| V4_8_0 | 20 | 80 | (base) |
| V4_9_07 | 24 | 96 | +5 transform, +2 padding |
| V4_9_13 | 28 | 112 | +tile, +3 padding |

**Final Uniform Struct (28 floats = 112 bytes):**
```wgsl
struct Uniforms {
    // Core (8)
    time: f32,
    tempo: f32,
    modA: f32,              // Legacy combined
    modB: f32,
    modC: f32,
    beat: f32,
    resolutionX: f32,
    resolutionY: f32,
    
    // Decoupled Modulators (9)
    shaderModAWave: f32,
    shaderModAGain: f32,
    shaderModAFFT: f32,
    shaderModBWave: f32,
    shaderModBGain: f32,
    shaderModBFFT: f32,
    shaderModCWave: f32,
    shaderModCGain: f32,
    shaderModCFFT: f32,
    
    // Transforms (8)
    transformRotation: f32,    // radians
    transformScaleX: f32,
    transformScaleY: f32,
    transformTranslateX: f32,
    transformTranslateY: f32,
    transformPivotX: f32,
    transformPivotY: f32,
    transformTile: f32,
    
    // Padding (3)
    padding1: f32,
    padding2: f32,
    padding3: f32,
}
```

### 4. OMS Design System (V4_9_32)

✅ **Universal CSS Components**
| Class | Purpose | Height |
|-------|---------|--------|
| `.oms-header-bar` | Section headers | auto (12px 15px padding) |
| `.oms-field` | Form inputs | 34px |
| `.oms-control-box` | Grouped controls | auto (2px 6px padding) |
| `.oms-dropdown` | Custom dropdown | 34px |
| `.mod-knob-box` | Modulator knobs | 34px |

✅ **Design Tokens**
| Token | Value |
|-------|-------|
| Surface BG | `rgba(255,255,255,0.05)` |
| Hover | `rgba(255,255,255,0.08)` (+0.03) |
| Focus Ring | `rgba(0,212,255,0.5)` |
| Control Height | `34px` |
| Border Radius | `0` (sharp corners) |

✅ **Inline Style Elimination**
- ~2,000+ characters of duplicated inline styles removed
- Single source of truth for visual consistency

### 5. OmsDropdown Expansion (V4_9_33-35)

✅ **CREATE Tab Conversion** (11 dropdowns)
- shaderNameDropdown (with thumbnails)
- orientationDropdown
- loopLengthDropdown
- modA/B/C SubdivDropdown (×3)
- modA/B/C WaveformDropdown (×3)
- resolutionDropdown
- fpsDropdown

✅ **Keyboard Mode Integration** (V4_9_34)
- Shift+R reactivity mode works with OmsDropdown
- Selection ring targets `dropdown.header`
- New `getModDropdown(row, col)` helper function

### 6. Unified Wheel Handler (V4_9_19-20, V4_9_35)

✅ **Shift+Trackpad for All Knobs** (14 controls total)

| Section | Controls | Behavior |
|---------|----------|----------|
| Transforms | Rotation, Scale, Tile | Encoder (deltaY) |
| Transforms | XY, Pivot | XY pad (deltaX + deltaY) |
| Reactivity | Gain ×3 | Linear 0-1 |
| Reactivity | Freq1 ×3, Freq2 ×3 | Log scale 20-24000 Hz |

Single document-level handler, routes by control ID.

---

## ITERATION HISTORY

### V4_9_01-06: TRANSFORMS UI Foundation

**V4_9_01:** HTML structure for TRANSFORMS section (5 canvas placeholders)

**V4_9_02:** Draw function `initTransformEncoders()` with config-driven approach

**V4_9_03:** Drag interaction, message bar feedback, removed labels

**V4_9_04:** Module border/fill styling matching `.control-state-info`

**V4_9_05:** controlStates wiring for save/load with .oms containers

**V4_9_06:** Keyboard mode (Shift+T) matching Reactivity pattern

### V4_9_07-09: GPU Uniforms & Validation

**V4_9_07:** Uniform struct expansion (80 → 96 bytes)
- Added: transformRotation, transformScaleX, transformScaleY, transformTranslateX, transformTranslateY
- Added: padding1, padding2
- Updated all 6 locations manually

**V4_9_08:** Test shader validating all 5 transform uniforms work

**V4_9_09:** ZOOM combo encoder (writes to both ScaleX and ScaleY)
- Layout: ROT | ZOOM | TRN X | TRN Y (4 controls)

### V4_9_10-14: XY Pads & Refinement

**V4_9_10:** XY Translate pad (50×30 landscape)
- Click+drag, trackpad support
- Layout: ROT | ZOOM | XY (3 controls)

**V4_9_11:** Shift+trackpad global control for transforms

**V4_9_12:** PIVOT XY pad
- Fixed Y-axis direction issues
- Layout: ROT | ZOOM | XY | PIVOT (4 controls)

**V4_9_13:** TILE encoder
- Uniform struct expansion (96 → 112 bytes)
- Layout: ROT | ZOOM | TILE | Translate | PIVOT (5 controls)
- Fixed "XY" → "Translate" message text

**V4_9_14:** Reactivity knob boxes (`.mod-knob-box` class)

### V4_9_15: CORTEX Shader Contract Consolidation ⚠️

**V4_9_15:** Major refactor - unified 6 scattered uniform definitions
- Created `SHADER_UNIFORM_STRUCT` constant
- Created `UNIFORM_BUFFER_SIZE` constant
- Created `createUniformData()` factory function
- Fixed thumbnail buffer bug (96 → 112 bytes)
- Fixed preset property mismatch (.code vs .shaderCode)

### V4_9_16-18: Shader Conventions & Fixes

**V4_9_16:** `// @name` shader header convention
- Replaced hardcoded `// OMS Shader Baker` parsing
- New tags: @name, @app, @url, @credit, @creditUrl, @desc

**V4_9_17:** Transform keyboard index fix
- Shift+Arrow for TRANSLATE/PIVOT after TILE was added
- Index 2→3 for Translate, 3→4 for Pivot

**V4_9_18:** Trackpad scroll on transform encoders (per-encoder handlers)

### V4_9_19-23: Interaction Refinement

**V4_9_19:** Unified wheel handler (document-level, routes by target.id)

**V4_9_20:** Two-mode wheel control
- Hover mode: Shift + hover + scroll
- Keyboard mode: Shift+T, select, scroll anywhere

**V4_9_21:** Double-tap Enter to reset (within 300ms)

**V4_9_22:** Standardization
- Reset requires Shift+Enter×2 (was Enter×2)
- Wheel handler requires Shift key
- Zoom → Scale rename (UI label)

**V4_9_23:** transformZoom → transformScale (internal IDs)

### V4_9_24-25: Endless Rotation

**V4_9_24:** Endless rotation encoder (wrapping 0-360°)
- Added `endless: true` config flag
- Modulo wrapping formula for negative values

**V4_9_25:** 360° visual indicator
- Full circle track (was 300° arc)
- Single indicator line from center
- 0° at 6 o'clock, clockwise

### V4_9_26-31: Borderless UI

**V4_9_26:** UI test - sharp corners, dropdown blend (0.05 bg, no border)

**V4_9_27:** Global control styling - all inputs/selects borderless

**V4_9_28:** (Reactivity section polish)

**V4_9_29:** Transforms row - compressed padding (12px 15px → 2px 6px)

**V4_9_30:** Output row spacing - 4px margin between rows

**V4_9_31:** Audio Controls - borderless styling applied

### V4_9_32: Design System Foundation

**V4_9_32:** Universal OMS components
- Created `.oms-header-bar`, `.oms-field`, `.oms-control-box`
- Eliminated ~2,000 chars inline styles
- CORTEX Design System proposal

### V4_9_33-35: OmsDropdown & Wheel Handler

**V4_9_33:** CREATE tab dropdown conversion
- 10 native selects → OmsDropdown (11 total with shaderName)
- Consistent 34px height
- Separator support for grouping

**V4_9_34:** Keyboard mode fix for OmsDropdown
- Shift+R works with converted dropdowns
- Selection ring on `.oms-dropdown-header`
- New `getModDropdown(row, col)` helper
- Fixed height mismatch (28px → 34px)

**V4_9_35:** Reactivity wheel handler
- Extended unified handler to 9 Reactivity knobs
- Gain (linear), Freq1/Freq2 (log scale)
- Total: 14 controls with Shift+trackpad support

---

## CORTEX CONSTANTS & GLOBAL VARIABLES ⚠️ CRITICAL

### CORTEX Constants (V4_9_15) - SHADER CONTRACT

These three constants define the shader contract. **Change = cascade failure across all rendering.**

```javascript
// ============================================
// CORTEX - SHADER CONTRACT (V4_9_15)
// Location: Top of <script>, lines ~100-150
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

// 2. Buffer Size (MUST match struct: 28 floats × 4 bytes)
const UNIFORM_BUFFER_SIZE = 112;

// 3. Factory Function (order MUST match WGSL struct exactly)
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
        0.0,  // padding1
        0.0,  // padding2
        0.0,  // padding3
    ]);
}
```

**Adding a New Uniform (4-step checklist):**
1. Add field to `SHADER_UNIFORM_STRUCT` (in WGSL block)
2. Update `UNIFORM_BUFFER_SIZE` (count floats × 4)
3. Add to `createUniformData()` with default value
4. Update controlStates save/load if persistent

### Global Variables - Transform Module (V4_9_06, V4_9_21)

```javascript
// ============================================
// GLOBAL STATE - TRANSFORM KEYBOARD MODE
// Location: ~line 19354
// ============================================

let transformKeyboardMode = false;  // Shift+T mode active
let transformSelectedCol = 0;       // 0=ROT, 1=SCALE, 2=TILE, 3=XY, 4=PIVOT
let transformKeyRepeatCount = 0;    // Progressive acceleration counter
let transformLastKeyTime = 0;       // Timestamp for acceleration calc
let transformLastEnterTime = 0;     // V4_9_21: For double-tap reset detection
```

### Global Variables - OmsDropdown Instances (V4_9_33)

```javascript
// ============================================
// GLOBAL STATE - OMSDROPDOWN INSTANCES
// Created in initCreateTabDropdowns()
// ============================================

// Shader Controls
window.orientationDropdown      // Landscape/Portrait/Square

// Timing
window.loopLengthDropdown       // Bar lengths + note durations

// Reactivity (6 dropdowns)
window.modASubdivDropdown       // Subdivision A
window.modAWaveformDropdown     // Waveform A
window.modBSubdivDropdown       // Subdivision B
window.modBWaveformDropdown     // Waveform B
window.modCSubdivDropdown       // Subdivision C
window.modCWaveformDropdown     // Waveform C

// Output
window.resolutionDropdown       // 720p/1080p/1440p/2160p
window.fpsDropdown              // 24/30/60
```

### Config Arrays - Transform Controls (V4_9_06)

```javascript
// ============================================
// CONFIG - TRANSFORM CONTROL MAPPING
// Used by keyboard navigation and wheel handler
// ============================================

const TRANSFORM_COL_IDS = [
    'transformRotation',
    'transformScale',
    'transformTile',
    'transformXY',
    'transformPivot'
];

const TRANSFORM_KNOB_IDS = [
    'transformRotationKnob',
    'transformScaleKnob',
    'transformTileKnob',
    'transformXYPad',
    'transformPivotPad'
];

const TRANSFORM_COL_NAMES = ['Rotation', 'Scale', 'Tile', 'Translate', 'Pivot'];

const TRANSFORM_CONFIGS = [
    { min: 0, max: 360, step: 1, default: 0, endless: true },   // Rotation
    { min: 0, max: 4, step: 0.01, default: 1 },                 // Scale
    { min: 1, max: 8, step: 0.1, default: 1 },                  // Tile
    { min: -1, max: 1, step: 0.01, default: 0 },                // XY
    { min: -1, max: 1, step: 0.01, default: 0 }                 // Pivot
];
```

---

## NEW FUNCTIONS

### Transform Module
```javascript
initTransformEncoders()          // Initialize 5 transform controls
updatetransformRotation(val)     // Update rotation + uniform
updatetransformScale(val)        // Update scale (writes ScaleX + ScaleY)
updatetransformXY(x, y)          // Update translate XY pad
updatetransformPivot(x, y)       // Update pivot XY pad
updatetransformTile(val)         // Update tile encoder
adjustTransformValue(delta)      // Keyboard value adjustment
adjustXYValue(deltaX, deltaY)    // Keyboard XY adjustment
adjustPivotValue(deltaX, deltaY) // Keyboard pivot adjustment
```

### Design System
```javascript
class OmsDropdown                // Custom dropdown component
initCreateTabDropdowns()         // Initialize 10 standard dropdowns
getModDropdown(row, col)         // Get OmsDropdown for reactivity grid
```

### State Variables
```javascript
let transformKeyboardMode        // Shift+T mode active
let transformSelectedCol         // 0-4 (ROT/SCALE/TILE/XY/PIVOT)
let transformLastEnterTime       // For double-tap reset
```

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| `adjustModValue()` | Uses OmsDropdown API instead of native select |
| `updateModSelection()` | Targets `.oms-dropdown-header` for selection ring |
| `applyControlState()` | Syncs OmsDropdown instances + transform values |
| All uniform writes | Now use `createUniformData()` factory |

---

## CSS CHANGES

### Universal Components (V4_9_32)
```css
.oms-header-bar { background: rgba(255,255,255,0.05); padding: 12px 15px; }
.oms-field { min-height: 34px; padding: 6px 10px; border: none; }
.oms-control-box { padding: 2px 6px; border: none; }
.oms-dropdown-header { min-height: 34px; padding: 6px 10px; }
.mod-knob-box { height: 34px; padding: 0 6px; }
```

### Borderless Updates (V4_9_26-31)
```css
/* All interactive elements */
background: rgba(255,255,255,0.05);
border: none;
border-radius: 0;
```

---

## BUG FIXES

| Issue | Version | Fix |
|-------|---------|-----|
| Thumbnail buffer size | V4_9_15 | 96 → 112 bytes |
| Preset .code vs .shaderCode | V4_9_15 | Standardized property name |
| Transform Y-axis inverted | V4_9_12 | Corrected shader coords |
| Shift+arrows keyboard indices | V4_9_17 | Updated after TILE added |
| OmsDropdown keyboard mode | V4_9_34 | New getModDropdown() helper |
| Dropdown height mismatch | V4_9_34 | 28px → 34px |

---

## STATISTICS

- **Total Sub-iterations:** 36
- **Lines Changed:** ~3,000+
- **CORTEX Constants Added:** 3
- **New Uniforms:** 8 (transform controls)
- **CSS Components:** 5 universal classes
- **Dropdowns Converted:** 11
- **Controls with Shift+trackpad:** 14

---

## DESIGN CONSTRAINT ACHIEVED

### 14" MacBook Pro Viewport
- Effective width: 1352px
- Goal: CREATE tab fully visible without vertical scroll
- **Result:** Unified 34px control height, tighter layout = achieved

---

## PRODUCTION READINESS: ~98%

**100% Complete:**
- Transform controls (all 5)
- CORTEX shader contract consolidation
- GPU uniforms (112 bytes, 28 floats)
- Design System components
- OmsDropdown conversion (11 total)
- Keyboard navigation (Shift+T, Shift+R)
- Trackpad control (unified handler, 14 controls)

**Inherited from V4_8_0 (100%):**
- Modulator knobs (9 total)
- Independent shader uniforms
- Reactivity keyboard mode
- ID3 BPM detection

**Remaining:**
- MIDI integration
- PERFORM EQ controls

---

**END OF COMPLETE CHANGELOG**

Version: Beta V5_0_0
Date: December 21, 2025
Total Development: 36 sub-iterations
License: GPL-3.0
