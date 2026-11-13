# CLAUDE SHADER CREATION GUIDE - OMS SHADER BAKER V4.0.0

**Purpose:** Guide for creating WGSL shaders using Claude AI  
**Version:** V4.0.0_D  
**Date:** November 25, 2025

---

## QUICK START

### Basic Workflow

1. **Open OMS Shader Baker** in your browser
2. **Click Code tab (W)** to see shader editor
3. **Copy the current shader code**
4. **Open Claude.ai** in another tab/window
5. **Paste shader code** and describe what you want
6. **Claude generates new shader**
7. **Copy new shader** back to Code tab
8. **See instant preview** in canvas
9. **NEW:** Test audio reactivity in LIVE tab!

**That's it!** No shader programming knowledge required.

---

## WHAT'S NEW IN V4.0.0

**Audio Reactivity in LIVE Tab:**
- Your shaders now react to audio during performance
- Same visual behavior as Controls tab preview
- Timeline-based reactivity (pre-analyzed FFT)
- Perfect synchronization with music

**Better Workflow:**
- Create in Controls tab
- Save with S button (snap to bar)
- Perform in LIVE tab with full reactivity

---

## HOW TO ASK CLAUDE

### Effective Prompts

**Good Examples:**

```
"Make this shader react more to bass frequencies in modA"

"Add a tunneling effect that pulses with the beat"

"Change the colors to purple and gold instead of cyan"

"Make the patterns rotate clockwise with modB"

"Create a kaleidoscope effect with 8-way symmetry"

"Add a vignette effect around the edges"

"Make it pulse harder when the kick drum hits"
```

**Bad Examples (Too Vague):**

```
"Make it better"
"Add effects"
"Make it cooler"
```

### Be Specific About

1. **Visual Elements**
   - Colors (specific hues, RGB values)
   - Shapes (circles, rectangles, lines, fractals)
   - Patterns (grid, spiral, radial, noise)
   - Movement (rotation, oscillation, pulsing)

2. **Audio Reactivity**
   - Which modulator (modA=bass, modB=mids, modC=treble)
   - How to react (scale, brightness, speed, color)
   - Intensity (subtle vs dramatic)

3. **Performance**
   - Complexity (simple vs complex)
   - Smoothness (60fps target)
   - LIVE tab compatibility

---

## SHADER TEMPLATE STRUCTURE

### Standard WGSL Shader

```wgsl
// Uniforms (values from app)
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32; // Bass (30-500 Hz)
@group(0) @binding(3) var<uniform> uModB: f32; // Mids (500-2000 Hz)
@group(0) @binding(4) var<uniform> uModC: f32; // Treble (2000-20000 Hz)

// Vertex shader (full-screen quad)
@vertex
fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    var pos = array<vec2<f32>, 4>(
        vec2(-1.0, -1.0), vec2(1.0, -1.0),
        vec2(-1.0, 1.0), vec2(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

// Fragment shader (your creativity here!)
@fragment
fn fs_main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = fragCoord.xy / uResolution;
    
    // Your shader magic here
    var color = vec3<f32>(0.0);
    
    // Example: Bass-reactive gradient
    color = vec3(uv.x * uModA, uv.y * uModB, 0.5 + uModC * 0.5);
    
    return vec4<f32>(color, 1.0);
}
```

### What You Can Modify

**Safe to Change:**
- Everything inside `fs_main()` fragment shader
- Color calculations
- Pattern generation
- Audio reactivity (using modA, modB, modC)
- Mathematical formulas

**Don't Change:**
- Uniform declarations (`@group`, `@binding`)
- Vertex shader (`vs_main` function)
- Fragment shader signature (`@fragment fn fs_main`)
- Return type (`vec4<f32>`)

---

## V4.0.0 MODULATOR SYSTEM

### Understanding the Two-Layer System

**Layer 1: Timeline Data (Pre-analyzed FFT)**
- Captured during "Add to Library" in Controls tab
- Contains actual audio levels over time
- Already has gains applied
- Stored in .oms container

**Layer 2: Waveform Oscillation (Live Calculation)**
- Sine, square, triangle, sawtooth, cycler, etc.
- Based on elapsed beats (musical timing)
- Subdivision controls rate
- Calculated each frame

**Final Result:**
```
modA = Timeline[index] × Waveform
     = (Audio Level × Gain) × Oscillator
```

### Using Modulators in Shaders

**Basic Pattern:**

```wgsl
// Scale something with bass
let size = 0.5 + uModA * 0.5; // 0.5 to 1.0

// Change brightness with mids
let brightness = 0.3 + uModB * 0.7; // 0.3 to 1.0

// Flicker with treble
let flicker = 1.0 - uModC * 0.3; // 1.0 to 0.7
```

**Combined Effects:**

```wgsl
// Rotate with bass, scale with mids
let angle = uTime + uModA * 3.14159;
let radius = 0.5 + uModB * 0.5;

// Color based on all three
let red = uModA;
let green = uModB;
let blue = uModC;
let color = vec3(red, green, blue);
```

### Prompting Claude for Audio Reactivity

**Good Examples:**

```
"Make the circles pulse larger when modA (bass) is high"

"Change color from blue to red based on modB (mids)"

"Add rapid flickering to the edges using modC (treble)"

"Use modA to control rotation speed"

"Make brightness follow modB"

"Create impact flash when modA peaks (kick drum)"
```

---

## OPTIMIZING FOR LIVE PERFORMANCE

### V4.0.0 Considerations

**Timeline-Based Reactivity:**
- Modulators in LIVE tab use pre-analyzed data
- Test in Controls tab first (live FFT analysis)
- Save with proper timeline data ("Add to Library")
- LIVE tab will match exactly

**Performance Tips:**
- Target 60fps (check in LIVE tab with 4 tracks)
- Avoid expensive operations in loops
- Use smoothstep() for antialiasing
- Test with actual music, not silence

### Workflow for LIVE Tab

1. **Create shader** in Controls tab
2. **Load audio** and set parameters
3. **Test reactivity** with preview
4. **Click S button** to snap to bars
5. **Add to Library** (analyzes FFT)
6. **Save as .oms** container
7. **LIVE tab** - drag to grid
8. **Launch** - see perfect reactivity!

---

## COMMON PATTERNS

### Pattern 1: Bass-Driven Pulse

```
"Create concentric circles that pulse with bass (modA)"
```

Claude will generate:
```wgsl
let center = vec2(0.5);
let dist = length(uv - center);
let circles = sin(dist * 20.0 - uTime * 2.0 + uModA * 5.0);
let color = vec3(circles * (0.5 + uModA * 0.5));
```

### Pattern 2: Frequency-Split Colors

```
"Make RGB channels respond to bass/mid/treble separately"
```

### Pattern 3: Beat-Synced Rotation

```
"Rotate shapes at tempo, speed up with mids"
```

### Pattern 4: Impact Effects

```
"Create flash/burst effect when bass peaks (kick drum)"
```

---

## TROUBLESHOOTING

### Shader Won't Compile

**Error:** "Failed to compile shader"

**Solutions:**
1. Check for missing semicolons
2. Verify all parentheses match
3. Ensure uniform names are correct (uTime, uResolution, uModA, etc.)
4. Ask Claude: "This shader has a compile error, can you fix it?"

### No Audio Reactivity in LIVE Tab

**Problem:** Shader doesn't react to audio in performance

**Solutions:**
1. Re-save container with "Add to Library" (analyzes FFT)
2. Click S button to snap to bars
3. Verify audio file loaded in Controls tab
4. Check modulator usage in shader code
5. Update old containers (pre-V4)

### Performance Issues

**Slow Rendering:**
- Simplify shader calculations
- Reduce loop iterations
- Test with 4 tracks in LIVE tab
- Ask Claude: "Make this shader more performant for live use"

---

## ADVANCED TECHNIQUES

### Using Time Creatively

```wgsl
// Slow pulsing
let pulse = sin(uTime * 0.5) * 0.5 + 0.5;

// Fast oscillation
let flicker = sin(uTime * 10.0) * 0.5 + 0.5;

// Wave motion
let wave = sin(uv.x * 10.0 + uTime * 2.0);

// Beat-synced (V4.0.0 - works in LIVE tab!)
let beat = sin(uTime * 2.0) * uModA;
```

### Combining Modulators

```wgsl
// Bass controls size, mids control brightness, treble adds detail
let size = 0.5 + uModA * 0.5;
let brightness = 0.3 + uModB * 0.7;
let detail = sin(uv.x * 50.0) * uModC * 0.2;
let circle = smoothstep(size + 0.01, size, length(uv - 0.5));
let color = vec3(circle * brightness + detail);
```

---

## BEST PRACTICES FOR V4.0.0

### Development Workflow

1. **Start in Controls Tab**
   - Create and test with live FFT
   - Adjust frequency ranges
   - Fine-tune reactivity

2. **Prepare for Performance**
   - Load final audio file
   - Click S button (snap to bars)
   - "Add to Library" (timeline analysis)
   - Save as .oms

3. **Test in LIVE Tab**
   - Drag to grid
   - Launch with Enter
   - Verify reactivity matches
   - Test with other tracks

### Audio Reactivity Balance

- **Bass (modA):** Big movements, main pulse
- **Mids (modB):** Color shifts, secondary motion
- **Treble (modC):** Fine details, sparkle
- Don't overuse - subtlety often better
- Test with your actual performance tracks

---

## EXAMPLE CONVERSATIONS

### Beginner

```
You: "Create pulsing circles that react to bass"
Claude: [generates shader]
You: "Make them pulse more dramatically"
Claude: [increases modA influence]
You: "Perfect! Now add color shift with mids"
Claude: [adds modB to color calculation]
```

### Intermediate

```
You: "Create a tunnel effect synced to kick drums"
Claude: [generates tunnel with modA]
You: "Add rotation controlled by mids"
Claude: [adds modB rotation]
You: "Make the edges flicker with hi-hats"
Claude: [adds modC edge effects]
```

### Advanced

```
You: "Create particle system where bass creates bursts,
      mids control particle color, and treble adds turbulence"
Claude: [generates complex multi-layer system]
You: "Optimize for 60fps with 4 tracks in LIVE tab"
Claude: [simplifies calculations while preserving effect]
```

---

## V4.0.0 SPECIFIC TIPS

**Timeline Analysis:**
- Always use "Add to Library" for timeline data
- Timeline captures actual audio dynamics
- LIVE tab uses this pre-analyzed data
- No live FFT in performance (more efficient)

**S Button Magic:**
- Snaps to complete bars
- Ensures beat grid alignment
- Stores bar count in container
- Critical for synchronized performances

**Enter Key Testing:**
- Test rapid re-cueing with Enter
- Shader should handle sudden restarts
- Bar 1 positioning is critical
- Works like CDJ cue button now

---

## RESOURCES

### WGSL Documentation
- https://www.w3.org/TR/WGSL/
- https://google.github.io/tour-of-wgsl/

### Shader Inspiration
- https://thebookofshaders.com/
- https://www.shadertoy.com/

### Claude AI
- https://claude.ai/
- Ask for shader help anytime!

---

## CONCLUSION

V4.0.0 brings your shaders to life in the LIVE tab with perfect audio reactivity. The workflow is simple:

1. **Create** with Claude's help
2. **Prepare** with timeline analysis
3. **Perform** with full reactivity

No shader programming required - just describe what you want to see, and Claude translates that into WGSL code.

**The power is in iteration:**
- Start simple
- Build complexity
- Test in LIVE tab
- Refine for performance

Now go create something amazing! 🎨✨

---

**END OF CLAUDE SHADER CREATION GUIDE**

Version: V4.0.0_D  
Date: November 25, 2025  
Website: https://itswessmithyo.com  
License: GPL-3.0
