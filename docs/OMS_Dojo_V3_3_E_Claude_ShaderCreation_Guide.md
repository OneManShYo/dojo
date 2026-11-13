# OMS SHADER BAKER V3.3.0 - CLAUDE SHADER CREATION GUIDE

**Purpose:** Guide for using Claude AI to create audio-reactive WGSL shaders  
**Version:** V3.3.0  
**Date:** November 2025

---

## OVERVIEW

This guide helps you use Claude AI (or other AI assistants) to create high-quality WGSL fragment shaders for OMS Shader Baker. The goal is to generate shaders that are audio-reactive, visually compelling, and optimized for live DJ performance.

---

## QUICK START PROMPT TEMPLATE

```
Create a WGSL fragment shader for WebGPU audio-reactive visuals.

Requirements:
- Target: @fragment function returning vec4<f32> color
- Vertex shader already provided (full-screen quad)
- Available uniforms:
  - iTime: f32 (elapsed seconds)
  - iResolution: vec2<f32> (canvas width/height)
  - audioLow: f32 (0.0-1.0, bass frequencies)
  - audioMid: f32 (0.0-1.0, mid frequencies) 
  - audioHigh: f32 (0.0-1.0, high frequencies)
  - mod_A, mod_B, mod_C: f32 (0.0-1.0, user controls)

Style: [describe visual style]
Audio reactivity: [describe how audio should affect visuals]
Colors: [describe color palette]

Example: "Flowing liquid metal with bass-reactive ripples, cyan/magenta colors"
```

---

## SHADER STRUCTURE

### Complete WGSL Template

```wgsl
struct VertexOutput {
    @builtin(position) position: vec4<f32>,
    @location(0) uv: vec2<f32>,
}

struct Uniforms {
    iTime: f32,
    iResolution: vec2<f32>,
    audioLow: f32,
    audioMid: f32,
    audioHigh: f32,
    mod_A: f32,
    mod_B: f32,
    mod_C: f32,
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

@vertex
fn vs_main(@builtin(vertex_index) vertex_index: u32) -> VertexOutput {
    var output: VertexOutput;
    let x = f32((vertex_index & 1u) << 1u) - 1.0;
    let y = 1.0 - f32((vertex_index & 2u));
    output.position = vec4<f32>(x, y, 0.0, 1.0);
    output.uv = vec2<f32>((x + 1.0) * 0.5, (1.0 - y) * 0.5);
    return output;
}

@fragment
fn fs_main(input: VertexOutput) -> @location(0) vec4<f32> {
    // Your shader code here
    let uv = input.uv * 2.0 - 1.0;
    let time = uniforms.iTime;
    let bass = uniforms.audioLow;
    let mid = uniforms.audioMid;
    let high = uniforms.audioHigh;
    
    // Example: simple gradient
    let color = vec3<f32>(uv.x, uv.y, sin(time));
    
    return vec4<f32>(color, 1.0);
}
```

---

## AUDIO REACTIVITY PATTERNS

### Bass Reactivity (Kick Drums)

```wgsl
// Scale effect intensity with bass
let scale = 1.0 + uniforms.audioLow * 0.5;
let scaledUV = uv * scale;

// Pulse brightness with bass
let brightness = 1.0 + uniforms.audioLow * 2.0;
let color = baseColor * brightness;

// Trigger events on bass hits
let threshold = 0.7;
if (uniforms.audioLow > threshold) {
    // Flash or change effect
}
```

### Mid Reactivity (Vocals, Synths)

```wgsl
// Control color with mids
let hue = uniforms.audioMid;
let color = hsvToRgb(vec3<f32>(hue, 1.0, 1.0));

// Rotate with mids
let angle = uniforms.audioMid * 6.28318; // 0-2π
let rotatedUV = rotate2D(uv, angle);
```

### High Reactivity (Hi-Hats, Cymbals)

```wgsl
// Add sparkle/noise with highs
let noise = random(uv + uniforms.iTime);
let sparkle = noise * uniforms.audioHigh;
let color = baseColor + vec3<f32>(sparkle);

// Increase detail with highs
let octaves = 3.0 + uniforms.audioHigh * 5.0;
let detail = fbm(uv, octaves);
```

---

## VISUAL PATTERNS

### Geometric Patterns

**Concentric Circles:**
```wgsl
let dist = length(uv);
let circles = sin(dist * 10.0 - uniforms.iTime * 2.0);
let bass_scale = 1.0 + uniforms.audioLow * 0.5;
circles = sin(dist * 10.0 * bass_scale - uniforms.iTime * 2.0);
```

**Radial Lines:**
```wgsl
let angle = atan2(uv.y, uv.x);
let radial = sin(angle * 12.0 + uniforms.iTime);
```

**Grid Pattern:**
```wgsl
let grid = sin(uv.x * 20.0) * sin(uv.y * 20.0);
```

### Flow Fields

**Simplex/Perlin Noise:**
```wgsl
let noise1 = snoise(uv * 5.0 + uniforms.iTime * 0.5);
let noise2 = snoise(uv * 5.0 - uniforms.iTime * 0.3);
let flow = vec2<f32>(noise1, noise2);
```

**Fractional Brownian Motion:**
```wgsl
fn fbm(p: vec2<f32>, octaves: f32) -> f32 {
    var value = 0.0;
    var amplitude = 0.5;
    var frequency = 1.0;
    for (var i = 0; i < i32(octaves); i++) {
        value += amplitude * snoise(p * frequency);
        frequency *= 2.0;
        amplitude *= 0.5;
    }
    return value;
}
```

### Color Palettes

**HSV to RGB Conversion:**
```wgsl
fn hsvToRgb(hsv: vec3<f32>) -> vec3<f32> {
    let h = hsv.x * 6.0;
    let s = hsv.y;
    let v = hsv.z;
    let c = v * s;
    let x = c * (1.0 - abs(h % 2.0 - 1.0));
    let m = v - c;
    
    var rgb: vec3<f32>;
    if (h < 1.0) { rgb = vec3<f32>(c, x, 0.0); }
    else if (h < 2.0) { rgb = vec3<f32>(x, c, 0.0); }
    else if (h < 3.0) { rgb = vec3<f32>(0.0, c, x); }
    else if (h < 4.0) { rgb = vec3<f32>(0.0, x, c); }
    else if (h < 5.0) { rgb = vec3<f32>(x, 0.0, c); }
    else { rgb = vec3<f32>(c, 0.0, x); }
    
    return rgb + vec3<f32>(m);
}
```

**Gradient Palettes:**
```wgsl
// Two-color gradient
let color1 = vec3<f32>(0.0, 1.0, 1.0); // Cyan
let color2 = vec3<f32>(1.0, 0.0, 1.0); // Magenta
let t = uv.x * 0.5 + 0.5; // 0.0 to 1.0
let color = mix(color1, color2, t);

// Multi-stop gradient
fn palette(t: f32) -> vec3<f32> {
    let a = vec3<f32>(0.5, 0.5, 0.5);
    let b = vec3<f32>(0.5, 0.5, 0.5);
    let c = vec3<f32>(1.0, 1.0, 1.0);
    let d = vec3<f32>(0.0, 0.33, 0.67);
    return a + b * cos(6.28318 * (c * t + d));
}
```

---

## MODULATOR USAGE

### Mod A, B, C Guidelines

```wgsl
// Mod_A: Often used for speed/tempo
let speed = uniforms.mod_A * 3.0 + 0.1;
let anim = uniforms.iTime * speed;

// Mod_B: Often used for complexity/detail
let complexity = uniforms.mod_B * 10.0 + 1.0;
let pattern = sin(uv.x * complexity);

// Mod_C: Often used for color/hue shift
let hueShift = uniforms.mod_C;
let color = hsvToRgb(vec3<f32>(hueShift, 1.0, 1.0));
```

---

## PERFORMANCE OPTIMIZATION

### Best Practices

**Minimize Calculations:**
```wgsl
// BAD: Calculate same value multiple times
let dist1 = length(uv);
let dist2 = length(uv);
let dist3 = length(uv);

// GOOD: Calculate once, reuse
let dist = length(uv);
let circles = sin(dist * 10.0);
let rings = cos(dist * 5.0);
```

**Avoid Branching:**
```wgsl
// BAD: If statements in tight loops
for (var i = 0; i < 100; i++) {
    if (someCondition) {
        // ...
    }
}

// GOOD: Use step/smoothstep
let mask = step(0.5, someValue);
let result = mix(valueA, valueB, mask);
```

**Use Built-in Functions:**
```wgsl
// GOOD: Fast built-ins
let d = distance(p1, p2);
let n = normalize(v);
let r = reflect(incident, normal);

// AVOID: Manual implementations when built-ins exist
```

---

## EXAMPLE PROMPTS

### Prompt 1: Bass-Reactive Tunnel

```
Create a WGSL shader that renders a tunnel effect with these properties:
- Tunnel rotates based on iTime
- Tunnel expands/contracts with audioLow (bass)
- Colors shift from cyan to magenta based on audioMid
- mod_A controls rotation speed (0.5-3.0)
- mod_B controls tunnel depth/perspective
- High-contrast, suitable for club visuals
```

### Prompt 2: Liquid Plasma

```
Create a flowing plasma effect shader:
- Multiple sine waves creating liquid motion
- audioLow controls wave amplitude (bigger waves on kick)
- audioMid controls color cycling speed
- audioHigh adds turbulence/noise
- mod_A: wave frequency (2-20)
- mod_B: flow speed (0.1-2.0)
- mod_C: color palette rotation (0-1)
- Smooth, organic movement
```

### Prompt 3: Geometric Kaleidoscope

```
Create a kaleidoscope pattern:
- 6-way symmetry (mirror 6 times around center)
- Geometric shapes (circles, lines, polygons)
- audioLow triggers shape pulsing
- audioMid rotates the pattern
- audioHigh controls shape complexity
- mod_A: number of symmetry segments (3-12)
- mod_B: zoom level (0.5-3.0)
- mod_C: color scheme (HSV hue shift)
- Sharp, digital aesthetic
```

---

## DEBUGGING SHADERS

### Common Issues

**Shader Won't Compile:**
- Check all semicolons (`;`)
- Verify function signatures match WGSL spec
- Ensure all variables are declared with `var` or `let`
- Check uniform binding syntax

**Black Screen:**
```wgsl
// Return a test color to verify shader is running
return vec4<f32>(1.0, 0.0, 1.0, 1.0); // Magenta test
```

**Colors Wrong:**
```wgsl
// Clamp values to 0-1 range
let color = clamp(rawColor, vec3<f32>(0.0), vec3<f32>(1.0));
```

**Flashing/Unstable:**
```wgsl
// Add smoothstep to transitions
let smooth = smoothstep(0.4, 0.6, uniforms.audioLow);
```

---

## ADVANCED TECHNIQUES

### Signed Distance Fields (SDF)

```wgsl
fn sdCircle(p: vec2<f32>, r: f32) -> f32 {
    return length(p) - r;
}

fn sdBox(p: vec2<f32>, b: vec2<f32>) -> f32 {
    let d = abs(p) - b;
    return length(max(d, vec2<f32>(0.0))) + min(max(d.x, d.y), 0.0);
}
```

### Domain Repetition

```wgsl
// Repeat space infinitely
fn repeat(p: vec2<f32>, spacing: vec2<f32>) -> vec2<f32> {
    return (p % spacing) - spacing * 0.5;
}
```

### Ray Marching

```wgsl
fn rayMarch(ro: vec3<f32>, rd: vec3<f32>) -> f32 {
    var t = 0.0;
    for (var i = 0; i < 64; i++) {
        let p = ro + rd * t;
        let d = sceneSDF(p);
        if (d < 0.001) { break; }
        t += d;
    }
    return t;
}
```

---

## BEST PRACTICES

### Do's

✅ **Use descriptive variable names**
```wgsl
let bassIntensity = uniforms.audioLow;
let rotationAngle = uniforms.iTime * 0.5;
```

✅ **Comment complex math**
```wgsl
// Convert UV from 0-1 to -1 to 1 centered space
let uv = input.uv * 2.0 - 1.0;
```

✅ **Normalize UV by aspect ratio**
```wgsl
let aspect = uniforms.iResolution.x / uniforms.iResolution.y;
let uv = vec2<f32>(input.uv.x * aspect, input.uv.y);
```

✅ **Use smooth transitions**
```wgsl
let transition = smoothstep(0.4, 0.6, someValue);
```

### Don'ts

❌ **Don't use undefined functions**
```wgsl
// BAD: fract() doesn't exist in WGSL
let f = fract(x);
// GOOD: Use frac()
let f = frac(x);
```

❌ **Don't exceed shader complexity**
```wgsl
// Avoid 1000+ line shaders
// Break into helper functions
```

❌ **Don't ignore audio normalization**
```wgsl
// BAD: Audio values can be very small
let effect = uniforms.audioLow; // Often < 0.1

// GOOD: Amplify and clamp
let effect = clamp(uniforms.audioLow * 3.0, 0.0, 1.0);
```

---

## PROMPT REFINEMENT PROCESS

### Iteration Pattern

1. **Initial Generation:** Ask for basic shader
2. **Test:** Load in OMS Shader Baker
3. **Refine:** Request specific changes:
   - "Make bass reactivity more pronounced"
   - "Add color cycling with mod_C"
   - "Smooth out the transitions"
4. **Optimize:** Request performance improvements
5. **Polish:** Final visual tweaks

### Example Refinement Conversation

```
You: Create a flowing liquid shader with bass reactivity

Claude: [generates basic liquid shader]

You: Make the bass response 3x stronger, and add color cycling

Claude: [updates shader with stronger bass and HSV color cycling]

You: Perfect! Now add mod_A to control flow speed from 0.1 to 2.0

Claude: [adds speed control]
```

---

## INTEGRATION WORKFLOW

### From Prompt to Performance

1. **Generate:** Use Claude to create WGSL shader
2. **Test:** Paste into CONTROLS tab in OMS Shader Baker
3. **Iterate:** Refine with Claude based on visual results
4. **Tune:** Adjust mod_A/B/C values for desired effect
5. **Save:** LIBRARY tab → Save as .oms preset
6. **Perform:** Load in LIVE tab, sync with audio

---

## RESOURCES

### WGSL Reference

- WGSL Spec: https://www.w3.org/TR/WGSL/
- WebGPU Fundamentals: https://webgpufundamentals.org/
- Shader Toy (GLSL, adapt to WGSL): https://www.shadertoy.com/

### Shader Inspiration

- The Book of Shaders: https://thebookofshaders.com/
- Inigo Quilez Articles: https://iquilezles.org/articles/
- Shader Park: https://shaderpark.com/

---

**END OF CLAUDE SHADER CREATION GUIDE**

**Remember:** Start simple, iterate, and let audio drive the visuals!
