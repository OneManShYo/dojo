# ONEMANSHYO Dojo - Claude Shader Creation Guide

## Creating Shaders with Claude AI

This guide helps you work with Claude to design custom WGSL shaders for ONEMANSHYO Dojo. Learn the patterns, structure, and best practices for conversational shader development.

---

## Quick Start Example

### User to Claude:
> "Create a shader with rotating hexagons that pulse to bass"

### Claude's Response Structure:

```wgsl
// ONEMANSHYO Dojo V2_6 - Rotating Hexagons
// Bass-driven pulsing hexagon pattern with rotation

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,        // Bass - hexagon size pulse
    modB: f32,        // Mids - rotation speed
    modC: f32,        // Treble - color intensity
    padding1: f32,
    resolutionX: f32,
    resolutionY: f32,
}

@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Build resolution vec2
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    
    // Normalize coordinates
    var uv = (FragCoord.xy * 2.0 - resolution) / resolution.y;
    
    // Apply rotation (modB controls speed)
    let angle = uniforms.time * uniforms.modB * 2.0;
    let rot = mat2x2<f32>(cos(angle), -sin(angle), sin(angle), cos(angle));
    uv = rot * uv;
    
    // Hexagon distance function
    let hex = hexagonSDF(uv, 0.3 + uniforms.modA * 0.2);
    
    // Color with modC intensity
    let col = vec3<f32>(0.0, 0.7, 1.0) * (1.0 - hex) * uniforms.modC;
    
    return vec4<f32>(col, 1.0);
}

fn hexagonSDF(p: vec2<f32>, r: f32) -> f32 {
    let k = vec3<f32>(-0.866025404, 0.5, 0.577350269);
    var p2 = abs(p);
    p2 -= 2.0 * min(dot(k.xy, p2), 0.0) * k.xy;
    p2 -= vec2<f32>(clamp(p2.x, -k.z*r, k.z*r), r);
    return length(p2) * sign(p2.y);
}
```

**User then:** Copy/paste into Code tab → Compile → Preview → Render!

---

## Essential WGSL Structure

### Required Template

Every shader MUST have this structure:

```wgsl
// ONEMANSHYO Dojo V2_6 - [ShaderName]
// Brief description of what shader does

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

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

@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Build resolution vec2 from separate fields
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    
    // YOUR SHADER CODE HERE
    
    return vec4<f32>(finalColor, 1.0);
}
```

### Critical Requirements

**1. Uniform Struct Layout:**
- MUST use separate `resolutionX` and `resolutionY` (not vec2)
- MUST include `padding1` field between modC and resolution
- MUST be in exact order shown above

**2. Comment Header:**
- MUST start with `// ONEMANSHYO Dojo` comment
- Include version (optional but recommended)
- Include shader name after dash
- Example: `// ONEMANSHYO Dojo V2_6 - CoolPattern`

**3. Resolution Reconstruction:**
- ALWAYS build vec2 from separate fields:
  ```wgsl
  let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
  ```

---

## Modulator Assignment Patterns

### Pattern 1: Energy-Based (Bass/Mid/Treble)

**Best for:** Music-reactive visuals

```wgsl
// modA = Bass (20-250Hz) - Large movements, size changes
// modB = Mids (250-4000Hz) - Medium movements, rotation
// modC = Treble (4000-20000Hz) - Small movements, intensity

// Example usage:
let circleSize = 0.3 + uniforms.modA * 0.4;      // Bass drives size
let rotationSpeed = uniforms.modB * 3.0;         // Mids drive rotation
let brightness = 0.5 + uniforms.modC * 0.5;      // Treble drives brightness
```

### Pattern 2: Parameter Control (Scale/Rotate/Intensity)

**Best for:** General visual control

```wgsl
// modA = Scale/Size
// modB = Rotation/Movement
// modC = Brightness/Intensity

// Example usage:
let scale = 0.5 + uniforms.modA * 1.0;           // Scale control
let angle = uniforms.time * uniforms.modB * 2.0; // Rotation control
let intensity = uniforms.modC;                   // Direct intensity
```

### Pattern 3: Multi-Parameter (Complex Effects)

**Best for:** Sophisticated visuals

```wgsl
// modA = Primary parameter (main effect strength)
// modB = Secondary parameter (variation/complexity)
// modC = Tertiary parameter (color/atmosphere)

// Example usage:
let waveStrength = uniforms.modA * 0.5;          // Primary wave effect
let waveFrequency = 2.0 + uniforms.modB * 8.0;   // Wave complexity
let colorShift = uniforms.modC * 6.28;           // Color variation
```

---

## Common Shader Patterns

### Pattern: Tunnel Effect

```wgsl
@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = (FragCoord.xy * 2.0 - resolution) / resolution.y;
    
    // Convert to polar coordinates
    let dist = length(uv);
    let angle = atan2(uv.y, uv.x);
    
    // Tunnel effect (modA controls speed)
    let tunnelZ = uniforms.time * (2.0 + uniforms.modA * 3.0);
    let tunnelEffect = dist + tunnelZ;
    
    // Radial grid (modB controls intensity)
    let gridAngle = angle * 8.0;
    let gridDist = fract(tunnelEffect * 4.0);
    let grid = smoothstep(0.48, 0.5, abs(sin(gridAngle)) * abs(sin(gridDist * 3.14159))) * uniforms.modB;
    
    // Color (modC controls brightness)
    let color1 = vec3<f32>(1.0, 0.2, 0.8); // Magenta
    let color2 = vec3<f32>(0.2, 0.8, 1.0); // Cyan
    let mixFactor = sin(tunnelEffect * 0.5) * 0.5 + 0.5;
    var col = mix(color1, color2, mixFactor) * grid * uniforms.modC;
    
    return vec4<f32>(col, 1.0);
}
```

### Pattern: Grid System

```wgsl
@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = FragCoord.xy / resolution;
    
    // Grid size (modA controls cell count)
    let gridSize = 8.0 + uniforms.modA * 24.0;
    let cell = floor(uv * gridSize);
    let cellUV = fract(uv * gridSize);
    
    // Grid lines (modB controls thickness)
    let lineThickness = 0.02 + uniforms.modB * 0.08;
    let gridLines = smoothstep(lineThickness, 0.0, min(cellUV.x, cellUV.y)) + 
                    smoothstep(1.0 - lineThickness, 1.0, max(cellUV.x, cellUV.y));
    
    // Color (modC controls intensity)
    let baseColor = vec3<f32>(0.0, 0.8, 1.0);
    var col = baseColor * gridLines * uniforms.modC;
    
    return vec4<f32>(col, 1.0);
}
```

### Pattern: Particle System

```wgsl
@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = (FragCoord.xy * 2.0 - resolution) / resolution.y;
    
    var col = vec3<f32>(0.0);
    
    // Create multiple particles (modA controls count via movement range)
    for (var i = 0; i < 20; i++) {
        let fi = f32(i);
        
        // Particle position (modA affects movement range)
        let angle = fi * 0.314159 + uniforms.time * (0.5 + uniforms.modA);
        let radius = 0.3 + sin(fi + uniforms.time) * 0.2 * uniforms.modA;
        let pos = vec2<f32>(cos(angle), sin(angle)) * radius;
        
        // Particle size (modB controls size)
        let size = 0.02 + uniforms.modB * 0.05;
        let dist = length(uv - pos);
        let particle = smoothstep(size, 0.0, dist);
        
        // Color variation
        let hue = fi * 0.1;
        let particleColor = vec3<f32>(
            sin(hue) * 0.5 + 0.5,
            sin(hue + 2.0) * 0.5 + 0.5,
            sin(hue + 4.0) * 0.5 + 0.5
        );
        
        col += particleColor * particle;
    }
    
    // Overall intensity (modC)
    col *= uniforms.modC;
    
    return vec4<f32>(col, 1.0);
}
```

### Pattern: Wave Distortion

```wgsl
@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    var uv = FragCoord.xy / resolution;
    
    // Wave distortion (modA controls amplitude)
    let waveAmplitude = uniforms.modA * 0.1;
    let waveFrequency = 8.0 + uniforms.modB * 16.0; // modB controls frequency
    uv.x += sin(uv.y * waveFrequency + uniforms.time * 2.0) * waveAmplitude;
    uv.y += cos(uv.x * waveFrequency + uniforms.time * 2.0) * waveAmplitude;
    
    // Create pattern with distorted UV
    let pattern = sin(uv.x * 10.0) * sin(uv.y * 10.0);
    
    // Color (modC controls intensity)
    let col = vec3<f32>(0.5 + pattern * 0.5) * uniforms.modC;
    
    return vec4<f32>(col, 1.0);
}
```

---

## Tempo Synchronization

### Beat-Based Animation

```wgsl
// Calculate beat position
let beatsPerSecond = uniforms.tempo / 60.0;
let currentBeat = uniforms.time * beatsPerSecond;

// Pulse on each beat
let beatPulse = fract(currentBeat);
let pulse = 1.0 - beatPulse; // Decays from 1 to 0

// Use pulse for effects
let scale = 1.0 + pulse * 0.3; // Size pulse
```

### Bar-Based Animation

```wgsl
// 4 beats per bar (4/4 time)
let beatsPerSecond = uniforms.tempo / 60.0;
let currentBar = (uniforms.time * beatsPerSecond) / 4.0;

// Change every bar
let barPhase = fract(currentBar);
let barIndex = floor(currentBar);

// Alternate effects per bar
let evenBar = f32(i32(barIndex) % 2);
```

---

## Color Schemes

### Cyan-Magenta Gradient

```wgsl
let color1 = vec3<f32>(0.0, 0.8, 1.0);  // Cyan
let color2 = vec3<f32>(1.0, 0.2, 0.8);  // Magenta
let col = mix(color1, color2, someFactor);
```

### Rainbow Cycle

```wgsl
fn hueToRGB(hue: f32) -> vec3<f32> {
    return vec3<f32>(
        sin(hue) * 0.5 + 0.5,
        sin(hue + 2.094) * 0.5 + 0.5,
        sin(hue + 4.189) * 0.5 + 0.5
    );
}

// Usage
let col = hueToRGB(uniforms.time * 2.0);
```

### Neon Glow

```wgsl
let baseColor = vec3<f32>(0.0, 1.0, 0.5); // Bright teal
let glowIntensity = uniforms.modC * 2.0;
let col = baseColor * glowIntensity;
```

---

## Optimization Tips

### Reduce Fragment Calculations

**Bad (calculates for every pixel):**
```wgsl
for (var i = 0; i < 100; i++) {
    // Heavy calculation per pixel
}
```

**Good (limit iterations):**
```wgsl
let iterationCount = i32(10.0 + uniforms.modA * 40.0); // 10-50 iterations
for (var i = 0; i < iterationCount; i++) {
    // Heavy calculation
}
```

### Use Built-In Functions

**Prefer:**
- `smoothstep()` over custom interpolation
- `mix()` over manual lerp
- `length()` over manual distance
- `normalize()` over manual normalization

### Avoid Expensive Operations

**Expensive:**
- `pow()`, `exp()`, `log()`
- Division (use multiplication by inverse)
- Branching (`if` statements in loops)

**Cheaper:**
- Multiplication, addition
- `sin()`, `cos()`
- `abs()`, `fract()`, `floor()`

---

## Testing Checklist

When Claude creates a shader, verify:

- [ ] Comment header includes shader name
- [ ] Uniform struct matches exact template
- [ ] Resolution built from resolutionX/resolutionY
- [ ] All three modulators (A/B/C) used meaningfully
- [ ] Tempo synchronization working (if applicable)
- [ ] Colors appropriate for intended use
- [ ] Shader compiles without errors
- [ ] Preview shows expected visual
- [ ] Modulators respond correctly to slider changes

---

## Common Requests

### "Make it more complex"
Add layers, more particles, additional effects, color variations

### "Make it simpler"
Remove effects, reduce particle count, simplify math

### "Sync to tempo better"
Add beat-based pulse, bar transitions, tempo-driven timing

### "Different color scheme"
Change base colors, add gradients, shift hue over time

### "More responsive to bass"
Increase modA multiplier, add bass-driven pulse effects

### "Add rotation"
Apply rotation matrix to UV coordinates with modB control

---

## Example Conversation Flow

**User:** "Create a shader with pulsing circles"

**Claude:** [Provides basic circle shader with pulse]

**User:** "Make the circles move in a spiral pattern"

**Claude:** [Adds spiral positioning to circles]

**User:** "Have the bass control the size and treble control the color brightness"

**Claude:** [Adjusts modA for size, modC for brightness]

**User:** "Perfect! Can you add a grid in the background?"

**Claude:** [Adds grid layer beneath circles]

**User:** "Great! Export this as .txt"

**Result:** Complete custom shader ready for performance use

---

## Advanced Techniques

### SDF (Signed Distance Functions)

```wgsl
// Circle SDF
fn circleSDF(p: vec2<f32>, r: f32) -> f32 {
    return length(p) - r;
}

// Box SDF
fn boxSDF(p: vec2<f32>, b: vec2<f32>) -> f32 {
    let d = abs(p) - b;
    return length(max(d, vec2<f32>(0.0))) + min(max(d.x, d.y), 0.0);
}

// Star SDF (5-pointed)
fn starSDF(p: vec2<f32>, r: f32) -> f32 {
    let k1 = vec2<f32>(0.809, -0.588);
    let k2 = vec2<f32>(-k1.x, k1.y);
    var p2 = vec2<f32>(abs(p.x), p.y);
    p2 -= 2.0 * max(dot(k1, p2), 0.0) * k1;
    p2 -= 2.0 * max(dot(k2, p2), 0.0) * k2;
    p2.x = abs(p2.x);
    p2.y -= r;
    let ba = r * vec2<f32>(-k1.y, k1.x) - vec2<f32>(0.0, 1.0);
    let h = clamp(dot(p2, ba) / dot(ba, ba), 0.0, r);
    return length(p2 - ba * h) * sign(p2.y * ba.x - p2.x * ba.y);
}
```

### Noise Functions

```wgsl
// Hash for pseudo-random
fn hash(p: vec2<f32>) -> f32 {
    var p3 = fract(vec3<f32>(p.xyx) * 0.1031);
    p3 += dot(p3, vec3<f32>(p3.yzx) + 33.33);
    return fract((p3.x + p3.y) * p3.z);
}

// Value noise
fn noise(p: vec2<f32>) -> f32 {
    let i = floor(p);
    let f = fract(p);
    
    let a = hash(i);
    let b = hash(i + vec2<f32>(1.0, 0.0));
    let c = hash(i + vec2<f32>(0.0, 1.0));
    let d = hash(i + vec2<f32>(1.0, 1.0));
    
    let u = f * f * (3.0 - 2.0 * f);
    
    return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}
```

---

## Troubleshooting Shader Issues

### Shader Won't Compile

**Check:**
1. Uniform struct matches template exactly
2. No typos in struct field names
3. All functions have return types
4. No missing semicolons or braces

### Black/Blank Output

**Check:**
1. Final color vec4 alpha is 1.0 (not 0.0)
2. Color values in 0.0-1.0 range
3. No division by zero
4. Resolution vec2 built correctly

### Modulators Don't Affect Visual

**Check:**
1. modA/B/C actually used in calculations
2. Multiplier values not too small (try 0.5-2.0 range)
3. Effect visible in preview (adjust intensity)

---

**Guide Version:** V2_6  
**Last Updated:** November 2025  
**Target Audience:** Users creating shaders with Claude AI

**Happy shader creation!**
