# OMS DOJO - CLAUDE SHADER CREATION GUIDE V2.7.0

**Version:** V2.7.0  
**Purpose:** Guide for creating WGSL shaders conversationally with Claude AI  
**Audience:** Users creating custom shaders for ONEMANSHYO Dojo

---

## TABLE OF CONTENTS

1. [Introduction](#introduction)
2. [WGSL Shader Structure](#wgsl-shader-structure)
3. [Uniform Buffer Reference](#uniform-buffer-reference)
4. [Audio-Reactive Patterns](#audio-reactive-patterns)
5. [Common Shader Patterns](#common-shader-patterns)
6. [Conversational Prompting](#conversational-prompting)
7. [Debugging & Troubleshooting](#debugging--troubleshooting)
8. [Example Shaders](#example-shaders)

---

## INTRODUCTION

### What This Guide Covers

This guide shows you how to work with Claude AI to create WGSL (WebGPU Shading Language) fragment shaders for ONEMANSHYO Dojo. You don't need to be a shader expert - Claude handles the technical details while you focus on creative vision.

### What You Need to Know

**Minimal Requirements:**
- Describe what you want visually (colors, movement, patterns)
- Basic understanding of music timing (bars, beats)
- Willingness to iterate and refine

**Claude Handles:**
- WGSL syntax and structure
- Uniform buffer integration
- Audio reactivity implementation
- Math and algorithms
- Optimization

### The Conversational Workflow

1. **Describe Vision:** "I want a pulsing circle that grows with bass"
2. **Claude Creates:** Complete WGSL shader code
3. **Test in Baker:** Paste code, compile, preview
4. **Refine:** "Make it more cyan, faster pulse"
5. **Claude Updates:** Modified shader code
6. **Repeat:** Until you're happy

**Key Principle:** You drive the creative vision, Claude handles the technical implementation.

---

## WGSL SHADER STRUCTURE

### Required Template

Every shader must follow this structure:

```wgsl
// Uniform buffer structure (REQUIRED - DO NOT MODIFY)
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

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

// Vertex shader (REQUIRED - DO NOT MODIFY)
@vertex
fn vertexMain(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    var pos = array<vec2<f32>, 4>(
        vec2<f32>(-1.0, -1.0),
        vec2<f32>(1.0, -1.0),
        vec2<f32>(-1.0, 1.0),
        vec2<f32>(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

// Fragment shader (YOUR CODE HERE)
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Reconstruct resolution
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    
    // Normalize coordinates (0.0 to 1.0)
    let uv = fragCoord.xy / resolution;
    
    // Center coordinates (-1.0 to 1.0)
    let center = (uv - 0.5) * 2.0;
    
    // Your shader logic here
    var color = vec3<f32>(0.0, 0.0, 0.0);
    
    // Example: gradient based on UV
    color = vec3<f32>(uv.x, uv.y, 0.5);
    
    return vec4<f32>(color, 1.0);
}
```

### Critical Rules

**1. DO NOT modify uniform struct:**
- Must match exact order and types
- Resolution is separate resolutionX/resolutionY (NOT vec2)
- Padding fields required for alignment

**2. DO NOT modify vertex shader:**
- Fixed full-screen quad
- No vertex manipulation needed

**3. Fragment shader is where you work:**
- All visual logic goes here
- Access uniforms for time/audio/tempo
- Return final color as vec4<f32>

---

## UNIFORM BUFFER REFERENCE

### Available Uniforms

```wgsl
uniforms.time          // f32 - Current time in beats
uniforms.tempo         // f32 - BPM (from Shader Tempo field)
uniforms.modA          // f32 - Audio modulator A (0.0-10.0)
uniforms.modB          // f32 - Audio modulator B (0.0-10.0)
uniforms.modC          // f32 - Audio modulator C (0.0-10.0)
uniforms.resolutionX   // f32 - Canvas width in pixels
uniforms.resolutionY   // f32 - Canvas height in pixels
```

### Time (Beat-Based)

**What It Is:**
- Elapsed time in musical beats (not seconds)
- Controlled by Shader Tempo field (BPM)
- Loops based on Loop Length setting

**Usage:**
```wgsl
let t = uniforms.time;
let cycle = t % 4.0; // Cycles every 4 beats (1 bar in 4/4)
```

**Examples:**
- `t` = current beat (0.0, 0.5, 1.0, 1.5...)
- `t % 1.0` = fraction within current beat (0.0-1.0)
- `t % 4.0` = position in 4-beat cycle
- `t % 16.0` = position in 16-beat cycle (4 bars)

### Tempo (BPM)

**What It Is:**
- Beats per minute from Shader Tempo field
- Useful for tempo-relative timing

**Usage:**
```wgsl
let bpm = uniforms.tempo;
let speed = bpm / 120.0; // Relative to 120 BPM baseline
```

**Example:**
```wgsl
// Pulse faster at higher BPM
let pulseRate = (bpm / 60.0) * uniforms.time;
let pulse = sin(pulseRate * 6.28318) * 0.5 + 0.5;
```

### Modulators (Audio-Reactive)

**What They Are:**
- modA, modB, modC = audio frequency analysis results
- Range: 0.0 (silence) to 10.0 (max gain × max amplitude)
- Controlled by frequency ranges (Freq1/Freq2) and gain settings

**Default Ranges (V2.7.0):**
- modA: 30-50 Hz (kick drums)
- modB: 50-75 Hz (sub-bass)
- modC: 100-500 Hz (lead bass)

**Usage:**
```wgsl
let bass = uniforms.modA;
let mid = uniforms.modB;
let treble = uniforms.modC;

// Scale circle with bass
let radius = 0.3 + (bass * 0.2);
```

**Important:**
- Modulators work WITHOUT audio loaded (default to 0.0)
- Shader timing is independent of audio timing
- Use modulators to enhance visuals, not drive them

### Resolution

**What It Is:**
- Canvas width and height in pixels
- Separate X and Y values (NOT vec2)

**Reconstruction:**
```wgsl
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
```

**Usage:**
```wgsl
// Normalize pixel coordinates to 0.0-1.0
let uv = fragCoord.xy / resolution;

// Aspect ratio correction
let aspect = resolution.x / resolution.y;
let correctedUV = vec2<f32>(uv.x * aspect, uv.y);
```

---

## AUDIO-REACTIVE PATTERNS

### Pattern 1: Pulse Effect

**Description:** Object pulses in size with audio.

**Code:**
```wgsl
let bass = uniforms.modA;
let baseSize = 0.3;
let pulseAmount = bass * 0.2; // 0-2x size change
let size = baseSize + pulseAmount;

// Use size for circle radius, line thickness, etc.
```

### Pattern 2: Color Shift

**Description:** Colors change based on audio frequency.

**Code:**
```wgsl
let bass = uniforms.modA;
let mid = uniforms.modB;
let treble = uniforms.modC;

// RGB driven by frequency bands
let color = vec3<f32>(bass, mid, treble);

// Or: Hue shift based on bass
let hue = bass * 0.3; // 0-3 hue range
```

### Pattern 3: Movement Speed

**Description:** Animation speed driven by audio.

**Code:**
```wgsl
let bass = uniforms.modA;
let baseSpeed = 1.0;
let audioSpeed = bass * 2.0; // 0-2x speed boost
let totalSpeed = baseSpeed + audioSpeed;

let t = uniforms.time * totalSpeed;
let wave = sin(t * 6.28318);
```

### Pattern 4: Distortion/Warping

**Description:** Geometry warps based on audio.

**Code:**
```wgsl
let bass = uniforms.modA;

// Warp UV coordinates
let warpAmount = bass * 0.1;
let warpedUV = uv + vec2<f32>(
    sin(uv.y * 10.0 + uniforms.time) * warpAmount,
    cos(uv.x * 10.0 + uniforms.time) * warpAmount
);
```

### Pattern 5: Particle Count/Density

**Description:** More elements appear with audio intensity.

**Code:**
```wgsl
let bass = uniforms.modA;

// Base count + audio-driven count
let baseCount = 10.0;
let audioCount = bass * 20.0; // Up to 20 more
let totalCount = baseCount + audioCount;

// Use in loop or grid generation
```

### Pattern 6: Brightness/Intensity

**Description:** Overall brightness modulated by audio.

**Code:**
```wgsl
let bass = uniforms.modA;

// Calculate base color
var color = vec3<f32>(0.5, 0.8, 1.0);

// Scale brightness with audio
let brightness = 0.5 + (bass * 0.5); // 0.5-1.0 range
color *= brightness;
```

### Pattern 7: Multi-Frequency Response

**Description:** Different visual elements react to different frequencies.

**Code:**
```wgsl
let bass = uniforms.modA;   // 30-50 Hz
let mid = uniforms.modB;    // 50-75 Hz
let treble = uniforms.modC; // 100-500 Hz

// Red channel = bass, Green = mid, Blue = treble
let color = vec3<f32>(bass * 0.3, mid * 0.5, treble * 0.8);

// Or: Different shapes for each
let circleSize = 0.2 + bass * 0.15;
let squareSize = 0.3 + mid * 0.2;
let triangleSize = 0.25 + treble * 0.18;
```

---

## COMMON SHADER PATTERNS

### Pattern: UV Coordinates

**Description:** Working with normalized screen coordinates.

**Code:**
```wgsl
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
let uv = fragCoord.xy / resolution; // 0.0 to 1.0

// Centered coordinates (-0.5 to 0.5)
let centeredUV = uv - 0.5;

// Centered and aspect-corrected (-1.0 to 1.0)
let aspect = resolution.x / resolution.y;
let correctedUV = vec2<f32>(
    (uv.x - 0.5) * aspect * 2.0,
    (uv.y - 0.5) * 2.0
);
```

### Pattern: Circle

**Description:** Draw a circle at center.

**Code:**
```wgsl
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
let uv = fragCoord.xy / resolution;
let center = (uv - 0.5) * 2.0;

let radius = 0.3;
let dist = length(center);

// Hard edge
let circle = step(dist, radius);

// Soft edge
let circle = smoothstep(radius + 0.01, radius - 0.01, dist);

// Color
let color = vec3<f32>(circle);
```

### Pattern: Gradient

**Description:** Color gradient across screen.

**Code:**
```wgsl
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
let uv = fragCoord.xy / resolution;

// Horizontal gradient
let color = vec3<f32>(uv.x, 0.5, 1.0 - uv.x);

// Radial gradient
let center = uv - 0.5;
let dist = length(center);
let color = vec3<f32>(1.0 - dist);
```

### Pattern: Wave Motion

**Description:** Sine wave animation.

**Code:**
```wgsl
let t = uniforms.time;
let wave = sin(t * 6.28318) * 0.5 + 0.5; // 0.0 to 1.0

// Use wave for anything
let color = vec3<f32>(wave);

// Multiple waves
let wave1 = sin(t * 2.0) * 0.5 + 0.5;
let wave2 = sin(t * 3.0) * 0.5 + 0.5;
let wave3 = sin(t * 5.0) * 0.5 + 0.5;
```

### Pattern: Rotation

**Description:** Rotate UV coordinates around center.

**Code:**
```wgsl
let t = uniforms.time;
let angle = t * 0.5; // Rotation speed

// 2D rotation matrix
let cosA = cos(angle);
let sinA = sin(angle);

let centered = uv - 0.5;
let rotated = vec2<f32>(
    centered.x * cosA - centered.y * sinA,
    centered.x * sinA + centered.y * cosA
);
let finalUV = rotated + 0.5;
```

### Pattern: Grid/Tiles

**Description:** Repeat pattern in grid.

**Code:**
```wgsl
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
let uv = fragCoord.xy / resolution;

// Tile count
let tiles = 8.0;
let tiled = fract(uv * tiles); // 0.0-1.0 per tile

// Use tiled for patterns
let dist = length(tiled - 0.5);
let circle = step(dist, 0.3);
```

### Pattern: Noise (Simple)

**Description:** Pseudo-random pattern.

**Code:**
```wgsl
fn hash(p: vec2<f32>) -> f32 {
    let p3 = fract(vec3<f32>(p.x, p.y, p.x) * 0.13);
    let p3dot = dot(p3, vec3<f32>(p3.y, p3.z, p3.x) + 3.333);
    return fract((p3.x + p3.y) * p3dot);
}

// Usage
let noise = hash(uv * 10.0 + uniforms.time);
let color = vec3<f32>(noise);
```

---

## CONVERSATIONAL PROMPTING

### How to Ask Claude for Shaders

**Good Prompts:**
- "Create a pulsing circle that grows with bass (modA), cyan and magenta colors"
- "Make a kaleidoscope effect that rotates based on time, with 6 symmetry arms"
- "Generate a grid of squares where each square pulses at different times"
- "Build a wave pattern that flows from left to right, driven by mid frequencies (modB)"

**Bad Prompts:**
- "Make something cool" (too vague)
- "Do a shader" (no direction)
- "Like the one on that website" (Claude can't see external references)

**Key Elements to Include:**
1. **Shape/Form:** Circles, lines, grids, waves, etc.
2. **Motion:** Pulse, rotate, flow, expand, etc.
3. **Colors:** Specific colors or palettes
4. **Audio Response:** Which modulator (A/B/C) drives what
5. **Timing:** Bar-based loops, beat-synced changes

### Iteration Pattern

**First Pass:**
```
You: "Create a pulsing circle, cyan color, grows with bass"
Claude: [generates shader]
You: [test in Baker]
```

**Refinement:**
```
You: "Make the pulse stronger, add magenta outline"
Claude: [updates shader]
You: [test again]
```

**Further Refinement:**
```
You: "Add rotation, slower pulse"
Claude: [final version]
You: [test and approve]
```

**Typical Iterations:** 2-5 rounds to perfect a shader

### Common Requests

**"Make it loop every 4 bars":**
```wgsl
let loopDuration = 16.0; // 16 beats = 4 bars in 4/4
let t = uniforms.time % loopDuration;
```

**"Make it more reactive to bass":**
```wgsl
let bass = uniforms.modA * 2.0; // Double sensitivity
// or
let bass = pow(uniforms.modA, 0.5); // Softer response curve
```

**"Change colors to X and Y":**
```wgsl
let color1 = vec3<f32>(0.0, 1.0, 1.0); // Cyan
let color2 = vec3<f32>(1.0, 0.0, 1.0); // Magenta
let finalColor = mix(color1, color2, someValue);
```

**"Speed up the animation":**
```wgsl
let t = uniforms.time * 2.0; // 2x speed
```

**"Make it symmetrical":**
```wgsl
// Mirror horizontally
let uv = fragCoord.xy / resolution;
uv.x = abs(uv.x - 0.5) * 2.0;

// Radial symmetry (8-way)
let angle = atan2(center.y, center.x);
let symAngle = angle * 4.0; // 8 symmetry = multiply by 4
```

---

## DEBUGGING & TROUBLESHOOTING

### Shader Won't Compile

**Error: "Expected identifier"**
- Check uniform struct matches template exactly
- Verify all variable names are valid (no spaces, start with letter)

**Error: "Unknown type"**
- Use `f32` not `float`
- Use `vec2<f32>` not `vec2`
- WGSL is strict about types

**Error: "Binding not found"**
- Verify `@group(0) @binding(0)` matches template
- Don't add extra bindings

### Visual Issues

**Nothing renders (black screen):**
- Check final return statement: `return vec4<f32>(color, 1.0);`
- Verify alpha channel = 1.0
- Check if color calculations are valid (not NaN or inf)

**Colors wrong:**
- Verify color ranges are 0.0-1.0 (not 0-255)
- Check RGB order (red, green, blue)
- Ensure alpha = 1.0 for full opacity

**Pattern doesn't move:**
- Verify using `uniforms.time` not a constant
- Check time is used in calculations
- Confirm shader tempo is not 0

**Audio reactivity not working:**
- Modulators work even without audio (default 0.0)
- Test with audio loaded and playing
- Check gain settings in controls (not 0)
- Verify frequency ranges appropriate for track

### Performance Issues

**Render too slow:**
- Reduce loop iterations
- Simplify calculations
- Avoid expensive functions (pow, exp) in tight loops
- Use lower resolution for testing

**Frame drops during preview:**
- Same as above
- Consider reducing FFT size (see Developer Documentation)

---

## EXAMPLE SHADERS

### Example 1: Bass-Reactive Pulsing Circle

**Description:** Cyan circle pulses with kick drums.

**Code:**
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
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

@vertex
fn vertexMain(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    var pos = array<vec2<f32>, 4>(
        vec2<f32>(-1.0, -1.0),
        vec2<f32>(1.0, -1.0),
        vec2<f32>(-1.0, 1.0),
        vec2<f32>(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = fragCoord.xy / resolution;
    let center = (uv - 0.5) * 2.0;
    
    // Bass reactivity
    let bass = uniforms.modA;
    let radius = 0.3 + (bass * 0.2);
    
    // Circle
    let dist = length(center);
    let circle = smoothstep(radius + 0.02, radius - 0.02, dist);
    
    // Cyan color
    let color = vec3<f32>(0.0, 1.0, 1.0) * circle;
    
    return vec4<f32>(color, 1.0);
}
```

### Example 2: Frequency-Split RGB

**Description:** Red = bass, Green = mid, Blue = treble.

**Code:**
```wgsl
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = fragCoord.xy / resolution;
    
    // Each frequency band drives one color channel
    let bass = uniforms.modA * 0.3;
    let mid = uniforms.modB * 0.5;
    let treble = uniforms.modC * 0.8;
    
    let color = vec3<f32>(bass, mid, treble);
    
    return vec4<f32>(color, 1.0);
}
```

### Example 3: Rotating Kaleidoscope

**Description:** 6-arm radial symmetry, rotates over time.

**Code:**
```wgsl
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = fragCoord.xy / resolution;
    let center = (uv - 0.5) * 2.0;
    
    // Polar coordinates
    let angle = atan2(center.y, center.x);
    let radius = length(center);
    
    // 6-way symmetry
    let symAngle = (angle * 3.0) % 6.28318;
    
    // Rotation
    let rotSpeed = uniforms.time * 0.5;
    let rotatedAngle = symAngle + rotSpeed;
    
    // Pattern
    let pattern = sin(rotatedAngle * 5.0 + radius * 10.0) * 0.5 + 0.5;
    
    // Color
    let color = vec3<f32>(pattern, pattern * 0.5, 1.0 - pattern);
    
    return vec4<f32>(color, 1.0);
}
```

### Example 4: Beat-Synced Flash

**Description:** Screen flashes white on every beat.

**Code:**
```wgsl
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Beat position (0.0-1.0 within each beat)
    let beatFraction = fract(uniforms.time);
    
    // Flash at start of beat (quick decay)
    let flash = exp(-beatFraction * 10.0);
    
    // White flash
    let color = vec3<f32>(flash);
    
    return vec4<f32>(color, 1.0);
}
```

### Example 5: Waveform Visualization

**Description:** Audio waveform represented as vertical bars.

**Code:**
```wgsl
@fragment
fn main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = fragCoord.xy / resolution;
    
    // Frequency bands
    let bass = uniforms.modA;
    let mid = uniforms.modB;
    let treble = uniforms.modC;
    
    // Three vertical bars
    var height = 0.0;
    if (uv.x < 0.33) {
        height = bass * 0.5;
    } else if (uv.x < 0.66) {
        height = mid * 0.5;
    } else {
        height = treble * 0.5;
    }
    
    // Bar visualization
    let bar = step(uv.y, height);
    let color = vec3<f32>(bar);
    
    return vec4<f32>(color, 1.0);
}
```

---

## BEST PRACTICES

### Design Principles

**1. Start Simple**
- Begin with basic shapes (circle, square)
- Add complexity incrementally
- Test each addition

**2. Use Audio Wisely**
- Modulators enhance, not dominate
- Shader should work without audio
- Use appropriate frequency ranges

**3. Loop Considerations**
- Design for seamless looping
- Use modulo for repeating patterns
- Test loop points in preview

**4. Color Choices**
- High contrast for visibility
- Complementary colors (cyan/magenta)
- Consider venue lighting

**5. Performance**
- Keep calculations simple
- Avoid nested loops when possible
- Test on target hardware

### Common Mistakes

**1. Hardcoded Resolution**
```wgsl
// BAD
let uv = fragCoord.xy / vec2<f32>(1920.0, 1080.0);

// GOOD
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
let uv = fragCoord.xy / resolution;
```

**2. Wrong Time Units**
```wgsl
// BAD
let t = uniforms.time / 60.0; // Already in beats!

// GOOD
let t = uniforms.time; // Use directly
```

**3. Missing Aspect Correction**
```wgsl
// BAD (circles become ovals on non-square canvas)
let dist = length(uv - 0.5);

// GOOD (circles stay circular)
let aspect = resolution.x / resolution.y;
let corrected = vec2<f32>((uv.x - 0.5) * aspect, uv.y - 0.5);
let dist = length(corrected);
```

**4. Color Range Confusion**
```wgsl
// BAD (colors out of range)
let color = vec3<f32>(255.0, 128.0, 64.0);

// GOOD (0.0-1.0 range)
let color = vec3<f32>(1.0, 0.5, 0.25);
```

---

## ADVANCED TECHNIQUES

### Signed Distance Fields (SDF)

**Description:** Precise shape rendering with smooth edges.

**Code:**
```wgsl
fn sdCircle(p: vec2<f32>, r: f32) -> f32 {
    return length(p) - r;
}

// Usage
let sdf = sdCircle(center, 0.3);
let circle = smoothstep(0.02, -0.02, sdf);
```

### Smooth Min/Max

**Description:** Blend shapes smoothly.

**Code:**
```wgsl
fn smin(a: f32, b: f32, k: f32) -> f32 {
    let h = max(k - abs(a - b), 0.0) / k;
    return min(a, b) - h * h * k * 0.25;
}
```

### Fractal Brownian Motion (FBM)

**Description:** Layered noise for natural textures.

**Code:**
```wgsl
fn fbm(p: vec2<f32>) -> f32 {
    var value = 0.0;
    var amplitude = 0.5;
    for (var i = 0; i < 4; i++) {
        value += hash(p) * amplitude;
        p *= 2.0;
        amplitude *= 0.5;
    }
    return value;
}
```

---

## RESOURCES

### Mathematical Functions

**Trigonometric:**
- `sin(x)` - Sine wave
- `cos(x)` - Cosine wave
- `atan2(y, x)` - Angle from vector

**Smoothing:**
- `smoothstep(edge0, edge1, x)` - Smooth interpolation
- `mix(a, b, t)` - Linear interpolation
- `clamp(x, min, max)` - Constrain to range

**Geometric:**
- `length(v)` - Vector magnitude
- `normalize(v)` - Unit vector
- `dot(a, b)` - Dot product

**Utility:**
- `fract(x)` - Fractional part
- `mod(x, y)` - Modulo
- `step(edge, x)` - 0 if x < edge, else 1

### Color Theory

**Complementary Pairs:**
- Cyan (0.0, 1.0, 1.0) / Red (1.0, 0.0, 0.0)
- Magenta (1.0, 0.0, 1.0) / Green (0.0, 1.0, 0.0)
- Yellow (1.0, 1.0, 0.0) / Blue (0.0, 0.0, 1.0)

**RGB to HSV Conversion:**
```wgsl
fn hsv2rgb(c: vec3<f32>) -> vec3<f32> {
    let k = vec4<f32>(1.0, 2.0/3.0, 1.0/3.0, 3.0);
    let p = abs(fract(c.xxx + k.xyz) * 6.0 - k.www);
    return c.z * mix(k.xxx, clamp(p - k.xxx, vec3<f32>(0.0), vec3<f32>(1.0)), c.y);
}
```

---

## CONCLUSION

### Key Takeaways

1. **You Control Creative Vision:** Describe what you want, Claude handles implementation
2. **Iterate Freely:** Refine through conversation, no coding expertise needed
3. **Audio Enhances:** Modulators add life, but shaders work independently
4. **Loop Design:** Consider seamless looping from the start
5. **Test Early:** Compile and preview frequently

### Next Steps

1. Start with simple shader (pulsing circle)
2. Test in Dojo
3. Refine with Claude
4. Experiment with audio reactivity
5. Build preset library

**"Go wild, design whatever you want in the shader"**

---

**END OF CLAUDE SHADER CREATION GUIDE**
