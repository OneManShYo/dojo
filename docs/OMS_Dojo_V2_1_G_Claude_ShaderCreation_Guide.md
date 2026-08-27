# ONEMANSHYO Dojo - Claude Shader Creation Guide

**Version:** V2_1  
**Target Audience:** AI Assistants creating shader code  
**Purpose:** Guide for generating high-quality WGSL shaders

---

## Overview

This guide helps AI assistants create effective WGSL fragment shaders for ONEMANSHYO Dojo V2_1. Focus on creating visually interesting, audio-reactive shaders that work well with the modulation system.

---

## Available Uniforms

```wgsl
struct Uniforms {
    time: f32,              // Elapsed time in seconds
    tempo: f32,             // Beats per minute (shader timing)
    modA: f32,              // Modulator A (0.0-1.0, bass audio-reactive)
    modB: f32,              // Modulator B (0.0-1.0, mid audio-reactive)
    modC: f32,              // Modulator C (0.0-1.0, treble audio-reactive)
    padding: f32,           // Alignment padding
    resolution: vec2<f32>,  // Canvas width, height (1920x1080 default)
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;
```

---

## Shader Structure

**Required signature:**
```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    // Your code here
    return vec4<f32>(r, g, b, a);  // Must return color
}
```

---

## Coordinate Normalization

**Always normalize coordinates:**
```wgsl
let resolution = uniforms.resolution;
let uv = (coord.xy / min(resolution.x, resolution.y)) * 2.0 - 1.0;
```

**Why:** Ensures consistent zoom across Portrait/Landscape/Square orientations.

---

## Using Modulators

**Modulator values are 0.0-1.0:**
```wgsl
let modA = uniforms.modA;  // Bass-reactive
let modB = uniforms.modB;  // Mid-reactive
let modC = uniforms.modC;  // Treble-reactive
```

**Common patterns:**
```wgsl
// Scale by modulator
let radius = 0.5 * (1.0 + modA);

// Use as color
let color = vec3<f32>(modA, modB, modC);

// Animate position
let offset = vec2<f32>(modA * 0.5, modB * 0.5);

// Control rotation
let angle = modC * 6.28318;  // 0-2π
```

---

## Tempo & Time

**Beat-synced animations:**
```wgsl
let tempo = uniforms.tempo;
let beat = (uniforms.time * tempo) / 60.0;
let phase = fract(beat / 4.0);  // 4-beat cycle
```

**Smooth vs stepped:**
```wgsl
// Smooth progression
let smooth = sin(beat * 6.28318) * 0.5 + 0.5;

// Stepped/quantized
let stepped = floor(beat) % 4.0;
```

---

## Common Patterns

### Rotating Shape
```wgsl
let angle = uniforms.time + modA * 3.14159;
let rot = mat2x2<f32>(
    cos(angle), -sin(angle),
    sin(angle), cos(angle)
);
let rotated_uv = rot * uv;
```

### Pulsing Circle
```wgsl
let dist = length(uv);
let radius = 0.3 + modA * 0.2;
let circle = smoothstep(radius + 0.01, radius, dist);
```

### Audio-Reactive Grid
```wgsl
let grid_uv = fract(uv * 10.0);
let grid = step(0.5, grid_uv.x) * step(0.5, grid_uv.y);
let color = vec3<f32>(grid * modB);
```

### Kaleidoscope
```wgsl
let angle = atan2(uv.y, uv.x);
let r = length(uv);
let a = abs(angle % 1.047) - 0.523;  // 6-fold symmetry
let kaleid_uv = vec2<f32>(cos(a), sin(a)) * r;
```

---

## Color Techniques

### RGB from Modulators
```wgsl
let color = vec3<f32>(modA, modB, modC);
```

### HSV-style
```wgsl
let hue = modA;
let sat = modB;
let val = modC;
// (implement HSV to RGB conversion)
```

### Gradient
```wgsl
let gradient = mix(
    vec3<f32>(1.0, 0.0, 0.0),
    vec3<f32>(0.0, 0.0, 1.0),
    modA
);
```

---

## Best Practices

**Performance:**
- Avoid expensive operations in loops
- Use built-in functions (sin, cos, length, etc.)
- Minimize texture lookups (if using textures)
- Keep complexity reasonable for 60fps

**Visual Design:**
- Test at different modulator values (0, 0.5, 1.0)
- Consider tempo-synced animations
- Use modulators creatively (not just scaling)
- Balance complexity with performance

**Audio Reactivity:**
- modA (bass) - good for scale, position, intensity
- modB (mid) - good for color, rotation, secondary motion
- modC (treble) - good for accents, rapid changes, sparkles

---

## Example Shaders

### Simple Pulsing Circle
```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = uniforms.resolution;
    let uv = (coord.xy / min(resolution.x, resolution.y)) * 2.0 - 1.0;
    
    let dist = length(uv);
    let radius = 0.3 + uniforms.modA * 0.3;
    let circle = smoothstep(radius + 0.02, radius, dist);
    
    let color = vec3<f32>(uniforms.modA, uniforms.modB, uniforms.modC);
    
    return vec4<f32>(color * circle, 1.0);
}
```

### Rotating Flower
```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = uniforms.resolution;
    let uv = (coord.xy / min(resolution.x, resolution.y)) * 2.0 - 1.0;
    
    let angle = atan2(uv.y, uv.x) + uniforms.time + uniforms.modB * 3.14159;
    let r = length(uv);
    let petals = 8.0;
    let shape = cos(angle * petals) * 0.5 + 0.5;
    let thickness = 0.3 + uniforms.modA * 0.2;
    
    let dist = abs(r - thickness);
    let flower = smoothstep(0.1, 0.05, dist) * shape;
    
    let color = mix(
        vec3<f32>(1.0, 0.0, 1.0),
        vec3<f32>(0.0, 1.0, 1.0),
        uniforms.modC
    );
    
    return vec4<f32>(color * flower, 1.0);
}
```

---

## Testing Checklist

When creating shaders, verify:
- [ ] Works at 1920x1080 (V2_1 default canvas)
- [ ] Works in all orientations (L/P/S)
- [ ] Responds to modA (bass)
- [ ] Responds to modB (mid)
- [ ] Responds to modC (treble)
- [ ] Looks good with audio (not just silent)
- [ ] Runs at 60fps (no stuttering)
- [ ] Compiles without errors

---

## Common Mistakes

**Coordinate normalization:**
```wgsl
// ❌ Wrong - doesn't adapt to orientation
let uv = coord.xy / uniforms.resolution;

// ✓ Correct - consistent across orientations
let uv = (coord.xy / min(uniforms.resolution.x, uniforms.resolution.y)) * 2.0 - 1.0;
```

**Modulator range:**
```wgsl
// ❌ Wrong - assumes 0-255 range
let color = vec3<f32>(uniforms.modA * 255.0);

// ✓ Correct - already 0-1 range
let color = vec3<f32>(uniforms.modA, uniforms.modB, uniforms.modC);
```

**Missing return:**
```wgsl
// ❌ Wrong - no return statement
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let color = vec3<f32>(1.0);
}

// ✓ Correct - returns vec4
return vec4<f32>(color, 1.0);
```

---

## V2_1 Considerations

**Bigger Canvas:**
- Preview is 1920x1080 (6x bigger than V2.0.2)
- Better detail visibility
- Performance impact is minimal

**Tab System:**
- Users can quickly switch to Code tab to edit
- Encourage frequent compilation ([C] button)
- Preview immediately visible in Controls tab

---

**Last Updated:** November 2025  
**Version:** V2_1  
**For:** AI Assistants creating shaders for ONEMANSHYO Dojo
