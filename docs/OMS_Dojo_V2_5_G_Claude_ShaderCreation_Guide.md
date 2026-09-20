# Claude Shader Creation Guide for ONEMANSHYO Dojo

## Purpose

This guide helps Claude create effective WGSL fragment shaders for ONEMANSHYO Dojo. The tool is designed for music visualization and live performance, so shaders should be visually engaging and audio-reactive when appropriate.

---

## Available Shader Uniforms

```wgsl
@group(0) @binding(0) var<uniform> tempo: f32;           // Shader Tempo
@group(0) @binding(1) var<uniform> modA: f32;            // Bass (0.0-1.0)
@group(0) @binding(2) var<uniform> modB: f32;            // Mid (0.0-1.0)
@group(0) @binding(3) var<uniform> modC: f32;            // Treble (0.0-1.0)
@group(0) @binding(4) var<uniform> resolution: vec2<f32>; // Canvas size
```

**Key Points:**
- `tempo`: Shader Tempo value for timing visual animations
- `modA/B/C`: Audio-reactive modulators (0-1 range), or time-based when no audio
- `resolution`: Canvas dimensions for aspect ratio calculations

---

## Shader Template Structure

```wgsl
@group(0) @binding(0) var<uniform> tempo: f32;
@group(0) @binding(1) var<uniform> modA: f32;
@group(0) @binding(2) var<uniform> modB: f32;
@group(0) @binding(3) var<uniform> modC: f32;
@group(0) @binding(4) var<uniform> resolution: vec2<f32>;

@fragment
fn main(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    // Normalize coordinates (0.0 to 1.0)
    let uv = FragCoord.xy / resolution;
    
    // Center coordinates (-1.0 to 1.0)
    let centered = (uv - 0.5) * 2.0;
    
    // Aspect ratio correction
    let aspect = resolution.x / resolution.y;
    let pos = vec2<f32>(centered.x * aspect, centered.y);
    
    // Your creative shader code here
    var color = vec3<f32>(0.0);
    
    // Example: use modulators
    color.r = modA; // Bass = red
    color.g = modB; // Mid = green
    color.b = modC; // Treble = blue
    
    return vec4<f32>(color, 1.0);
}
```

---

## Design Patterns

### 1. Time-Based Animation

Use `tempo` for rhythm-synced animations:

```wgsl
// Calculate beat from modA (which cycles with tempo)
let beat = modA * 4.0; // 0-4 range over subdivision cycle

// Pulsing circle
let pulse = sin(beat * 3.14159 * 2.0) * 0.5 + 0.5;
let circle = 1.0 - smoothstep(0.3, 0.32, length(pos) * pulse);
```

### 2. Audio-Reactive Colors

Map audio bands to color channels:

```wgsl
// Direct mapping
var color = vec3<f32>(modA, modB, modC);

// Selective mapping
var color = vec3<f32>(0.0);
color.r = modA * 2.0;        // Boost bass in red
color.b = modC * 1.5;        // Boost treble in blue
color.g = (modB + modC) * 0.5; // Blend mid+treble in green
```

### 3. Audio-Reactive Geometry

Use modulators to drive shape parameters:

```wgsl
// Radius modulated by bass
let radius = 0.3 + (modA * 0.2);
let circle = smoothstep(radius + 0.01, radius, length(pos));

// Rotation modulated by mid
let angle = modB * 3.14159 * 2.0;
let rotated = vec2<f32>(
    pos.x * cos(angle) - pos.y * sin(angle),
    pos.x * sin(angle) + pos.y * cos(angle)
);

// Scale modulated by treble
let scale = 1.0 + (modC * 0.5);
let scaled = pos / scale;
```

### 4. Kaleidoscope Effects

```wgsl
// Mirror coordinates for kaleidoscope
fn kaleidoscope(p: vec2<f32>, segments: f32) -> vec2<f32> {
    let angle = atan2(p.y, p.x);
    let sector = 3.14159 * 2.0 / segments;
    let a = mod(angle, sector) - sector * 0.5;
    let r = length(p);
    return vec2<f32>(cos(a) * r, sin(a) * r);
}

// Usage with audio reactivity
let segments = 6.0 + (modA * 6.0); // 6-12 segments
let kalei = kaleidoscope(pos, segments);
```

### 5. Gradient Patterns

```wgsl
// Radial gradient
let dist = length(pos);
let gradient = 1.0 - smoothstep(0.0, 1.0, dist);

// Angular gradient
let angle = atan2(pos.y, pos.x) / (3.14159 * 2.0) + 0.5;

// Audio-reactive gradients
let radial = smoothstep(0.0, modA, dist);
let angular = smoothstep(0.0, modB, angle);
```

---

## Best Practices

### Performance

**DO:**
- Keep shader logic simple and efficient
- Use built-in functions (sin, cos, smoothstep)
- Minimize branching (if statements)
- Reuse calculations in variables

**DON'T:**
- Use loops with high iteration counts
- Create deeply nested functions
- Perform expensive operations per pixel
- Use complex ray-marching algorithms

### Visual Design

**DO:**
- Create clear focal points
- Use color contrast effectively
- Sync major visual elements to modA (bass)
- Keep motion smooth and continuous

**DON'T:**
- Create overly chaotic patterns
- Use rapid flashing (epilepsy concern)
- Overwhelm with complexity
- Ignore aspect ratio correction

### Audio Reactivity

**Modulator Guidelines:**
- **modA (Bass):** Major structural changes, scale, primary motion
- **modB (Mid):** Secondary details, rotation, color shifts
- **modC (Treble):** Accents, fine details, sparkle effects

**Example Mapping:**
```wgsl
// Bass drives overall scale
let scale = 1.0 + (modA * 0.3);

// Mid drives rotation
let rotation = modB * 3.14159;

// Treble adds sparkle/glow
let glow = modC * 2.0;
```

---

## Common Patterns Library

### Pattern 1: Centered Circle

```wgsl
let dist = length(pos);
let circle = 1.0 - smoothstep(0.3, 0.32, dist);
let color = vec3<f32>(circle) * vec3<f32>(modA, modB, modC);
```

### Pattern 2: Radial Stripes

```wgsl
let angle = atan2(pos.y, pos.x);
let stripes = sin(angle * 8.0 + modA * 10.0) * 0.5 + 0.5;
let color = vec3<f32>(stripes * modB);
```

### Pattern 3: Grid

```wgsl
let grid = fract(pos * 10.0);
let lines = step(0.95, max(grid.x, grid.y));
let color = vec3<f32>(lines) * vec3<f32>(modA, modB, modC);
```

### Pattern 4: Tunnel Effect

```wgsl
let dist = length(pos);
let tunnel = sin(dist * 10.0 - modA * 20.0) * 0.5 + 0.5;
let color = vec3<f32>(tunnel) * vec3<f32>(0.0, modB, modC);
```

### Pattern 5: Plasma

```wgsl
let plasma = sin(pos.x * 5.0 + modA * 5.0) +
             sin(pos.y * 7.0 + modB * 5.0) +
             sin((pos.x + pos.y) * 6.0 + modC * 5.0);
plasma = plasma / 3.0 * 0.5 + 0.5;
let color = vec3<f32>(plasma);
```

---

## User Request Interpretation

When user asks for shaders, consider:

**"Audio-reactive"** → Use all three modulators (modA/B/C)  
**"Geometric"** → Clean shapes, hard edges, symmetry  
**"Organic"** → Flowing curves, noise functions, smooth transitions  
**"Psychedelic"** → Kaleidoscope, intense colors, complex patterns  
**"Minimal"** → Simple shapes, limited colors, clean composition  
**"Energetic"** → Fast motion, high contrast, strong bass response

---

## Testing Recommendations

Suggest users test with:
1. Default shader first (verify setup works)
2. Simple test shader (verify modulators work)
3. Final creative shader (verify performance)

Always mention:
- Test at 720p first before 4K
- Adjust Shader Tempo for desired speed
- Use Loop button to audition with audio

---

## Example Complete Shader

```wgsl
@group(0) @binding(0) var<uniform> tempo: f32;
@group(0) @binding(1) var<uniform> modA: f32;
@group(0) @binding(2) var<uniform> modB: f32;
@group(0) @binding(3) var<uniform> modC: f32;
@group(0) @binding(4) var<uniform> resolution: vec2<f32>;

@fragment
fn main(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = FragCoord.xy / resolution;
    let centered = (uv - 0.5) * 2.0;
    let aspect = resolution.x / resolution.y;
    let pos = vec2<f32>(centered.x * aspect, centered.y);
    
    // Audio-reactive pulsing circle
    let pulse = modA * 0.3;
    let radius = 0.4 + pulse;
    let dist = length(pos);
    let circle = 1.0 - smoothstep(radius - 0.01, radius + 0.01, dist);
    
    // Rotation based on mid frequencies
    let angle = modB * 3.14159 * 2.0;
    let rotated = vec2<f32>(
        pos.x * cos(angle) - pos.y * sin(angle),
        pos.x * sin(angle) + pos.y * cos(angle)
    );
    
    // Radial pattern
    let radial = sin(atan2(rotated.y, rotated.x) * 8.0) * 0.5 + 0.5;
    
    // Color mixing
    var color = vec3<f32>(0.0);
    color.r = modA;
    color.g = modB;
    color.b = modC;
    
    // Combine elements
    let final = circle * radial * color;
    final += vec3<f32>(modC * 0.2); // Treble glow
    
    return vec4<f32>(final, 1.0);
}
```

This creates a pulsing, rotating, audio-reactive circular pattern with color mapped to audio frequencies.

---

**Last Updated:** November 2025  
**For Version:** V2_5
