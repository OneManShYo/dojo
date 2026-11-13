# ONEMANSHYO App [Beta V5_0_0] - RELEASE NOTES

**Release Date:** December 21, 2025  
**Version:** Beta V5_0_0  
**Type:** Transform Controls & Design System

---

## WHAT'S NEW

### 🎛️ Transform Controls

**Rotate, scale, tile, and position your shaders live!**

The CREATE tab now has 5 transform controls in the TRANSFORMS row:

| Control | Type | Range | Default |
|---------|------|-------|---------|
| Rotation | Endless encoder | 0-360° | 0° |
| Scale | Encoder | 0-4× | 1× |
| Tile | Encoder | 1-8 | 1 |
| Translate | XY Pad (50×30) | -1 to 1 | 0,0 |
| Pivot | XY Pad (50×30) | -1 to 1 | 0,0 |

- **Rotation** wraps endlessly (360° → 0°)
- **Scale** affects both X and Y uniformly
- **Tile** repeats the shader pattern
- **Pivot** sets rotation/scale center point

All controls save with your .oms containers!

---

### ⌨️ Transform Keyboard Mode

**Control transforms without a mouse!**

| Key | Action |
|-----|--------|
| `Shift+T` | Enter/exit transform mode |
| `← →` | Navigate between controls |
| `Shift+←` | Decrease encoder value |
| `Shift+→` | Increase encoder value |
| `Shift+Arrows` | Move XY/Pivot pads |
| `Shift+Enter×2` | Reset to default (double-tap) |
| `Escape` | Exit mode |

Cyan selection ring shows current control. Works alongside Shift+R for reactivity!

---

### 🖱️ Shift+Trackpad Control

**Scroll to adjust any knob!**

Two ways to use:
1. **Hover mode:** Shift + hover over control + scroll
2. **Keyboard mode:** Enter mode, select control, Shift + scroll anywhere

Works for ALL 14 adjustable controls:
- 5 Transform controls (Rotation, Scale, Tile, XY, Pivot)
- 9 Reactivity knobs (Gain×3, Freq1×3, Freq2×3)

Mouse wheel and trackpad both supported!

---

### 🎨 New Visual Style

**Cleaner, more modern interface!**

- **Borderless controls** - No more busy borders
- **Sharp corners** - Modern modular aesthetic
- **Consistent height** - All controls 34px
- **Unified dropdowns** - Custom OmsDropdown everywhere

The new design system ensures visual consistency across all tabs.

---

### 📐 8 New Shader Uniforms

**Control geometry from within your shaders!**

```wgsl
// Transform uniforms
uniforms.transformRotation    // Rotation in radians
uniforms.transformScaleX      // Scale X (1.0 = normal)
uniforms.transformScaleY      // Scale Y (1.0 = normal)
uniforms.transformTranslateX  // Position X (-1 to 1)
uniforms.transformTranslateY  // Position Y (-1 to 1)
uniforms.transformPivotX      // Pivot X (-1 to 1)
uniforms.transformPivotY      // Pivot Y (-1 to 1)
uniforms.transformTile        // Tile count (1-8)
```

Use these in your shader's transform chain for live control!

---

### 🔧 Shader Contract Consolidation

**For developers: One source of truth!**

The uniform struct is now defined in 3 CORTEX constants:
- `SHADER_UNIFORM_STRUCT` - WGSL definition
- `UNIFORM_BUFFER_SIZE` - 112 bytes
- `createUniformData()` - Factory function

Adding a new uniform now takes 4 steps instead of 10!

---

## KEYBOARD QUICK REFERENCE

### Transform Mode (Shift+T to enable)

| Key | Action |
|-----|--------|
| `← →` | Navigate: Rotation ↔ Scale ↔ Tile ↔ Translate ↔ Pivot |
| `Shift+← →` | Adjust encoder values |
| `Shift+↑↓←→` | Move XY/Pivot pads |
| `Shift+Enter×2` | Reset selected to default |
| `Escape` | Exit mode |

### Reactivity Mode (Shift+R to enable)

| Key | Action |
|-----|--------|
| `← →` | Column: Subdiv → Waveform → Gain → Freq1 → Freq2 |
| `↑ ↓` | Row: A ↔ B ↔ C |
| `Shift+← →` | Adjust value |
| `Escape` | Exit mode |

### Trackpad (Both modes)

| Action | Result |
|--------|--------|
| `Shift + hover + scroll` | Adjust hovered control |
| `Shift + scroll` (in mode) | Adjust selected control |

---

## TRANSFORM CHAIN FOR SHADERS

Standard transform chain to use in your shaders:

```wgsl
// === FULL TRANSFORM CHAIN ===
let minDim = min(resolution.x, resolution.y);
var uv = (coord.xy - resolution * 0.5) / minDim * 2.0;

// Apply pivot offset
uv.x -= uniforms.transformPivotX;
uv.y += uniforms.transformPivotY;

// Rotate around pivot
let cosR = cos(uniforms.transformRotation);
let sinR = sin(uniforms.transformRotation);
uv = vec2<f32>(uv.x * cosR - uv.y * sinR, uv.x * sinR + uv.y * cosR);

// Remove pivot, apply translation
uv.x += uniforms.transformPivotX - uniforms.transformTranslateX;
uv.y -= uniforms.transformPivotY - uniforms.transformTranslateY;

// Scale
uv.x /= max(uniforms.transformScaleX, 0.01);
uv.y /= max(uniforms.transformScaleY, 0.01);

// Tile
let tile = max(uniforms.transformTile, 1.0);
uv = fract(uv * tile * 0.5 + 0.5) * 2.0 - 1.0;
```

---

## DESIGN SYSTEM COMPONENTS

New universal CSS classes for consistent styling:

| Class | Purpose |
|-------|---------|
| `.oms-header-bar` | Section headers |
| `.oms-field` | Input fields |
| `.oms-control-box` | Grouped controls |
| `.oms-dropdown` | Custom dropdowns |
| `.mod-knob-box` | Modulator knobs |

All use 34px height, borderless style, sharp corners.

---

## OMSDROPDOWN EVERYWHERE

All CREATE tab dropdowns now use the custom OmsDropdown component:

- **Shader Name** (with thumbnails)
- **Orientation** (Landscape/Portrait/Square)
- **Loop Length** (bars and durations)
- **Subdivision** (×3 for A/B/C)
- **Waveform** (×3 for A/B/C)
- **Resolution** (720p to 2160p)
- **FPS** (24/30/60)

Benefits:
- Consistent styling
- Keyboard navigation (↑↓ Enter Escape)
- Works with Shift+R reactivity mode

---

## BUG FIXES

- ✅ Thumbnail buffer size corrected (was 96, now 112 bytes)
- ✅ Preset property name (.code vs .shaderCode) standardized
- ✅ Transform Y-axis direction fixed
- ✅ Keyboard indices updated after Tile control added
- ✅ OmsDropdown height matches other controls (34px)
- ✅ Selection ring targets dropdown header correctly

---

## WORKFLOW TIPS

### Quick Transform Reset

1. Press `Shift+T` to enter transform mode
2. Navigate to control with `← →`
3. Double-tap `Shift+Enter` to reset to default
4. Press `Escape` when done

### Live Rotation with Trackpad

1. Hover over Rotation encoder
2. Hold `Shift` + scroll
3. Shader rotates in real-time!

### Combine with Reactivity

- `Shift+T` for transforms
- `Shift+R` for reactivity
- Both work independently
- Both support Shift+trackpad

---

## COMPATIBILITY

✅ Fully compatible with V4_8_0  
✅ All .oms containers work  
✅ All .omsp projects work  
✅ Old shaders work unchanged  
✅ Transform uniforms default to neutral values

---

## INHERITED FROM V4_8_0

Everything from V4_8_0 is still here:
- 9 Modulator knobs (Gain, Freq1, Freq2 × 3)
- Reactivity keyboard mode (Shift+R)
- Independent shader uniforms (Wave/Gain/FFT)
- Beat uniform for tempo-sync
- ID3 BPM detection
- WGSL preset support
- Custom shader dropdown with thumbnails

---

## WHAT'S NEXT

Coming in future versions:
- **MIDI Integration** - Map controllers to transforms
- **PERFORM EQ** - Kill/adjust frequency bands live
- **More transform modes** - Kaleidoscope, mirror, warp

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://onemanshyo.com

---

**END OF RELEASE NOTES**

Version: Beta V5_0_0
Date: December 21, 2025
License: GPL-3.0

*"Transform your vision"*
