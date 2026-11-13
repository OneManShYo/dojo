# CLAUDE SHADER CREATION GUIDE - OMS SHADER BAKER V3.4.0

**Purpose:** Guide for creating WGSL shaders using Claude AI  
**Version:** V3.4.0  
**Date:** November 15, 2025

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

**That's it!** No shader programming knowledge required.

---

## HOW TO ASK CLAUDE

### Effective Prompts

**Good Examples:**

```
"Make this shader react more to bass frequencies in modA"

"Add a tunneling effect that pulses with the beat"

"Change the colors to purple and gold instead of cyan"

"Make the patterns rotate clockwise"

"Create a kaleidoscope effect with 8-way symmetry"

"Add a vignette effect around the edges"
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
   - Smoothness (60fps vs artistic glitches)

---

## SHADER TEMPLATE STRUCTURE

### Standard WGSL Shader

```wgsl
// Uniforms (values from app)
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32; // Bass
@group(0) @binding(3) var<uniform> uModB: f32; // Mids
@group(0) @binding(4) var<uniform> uModC: f32; // Treble

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
    
    // Example: Simple gradient
    color = vec3(uv.x, uv.y, 0.5);
    
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

## MODULATOR SYSTEM

### Understanding Modulators

**modA = Bass Frequencies**
- Default: 30-500 Hz
- Targets: Kick drum, bassline, sub-bass
- Use for: Big movements, impacts, scale

**modB = Mid Frequencies**
- Default: 500-2000 Hz
- Targets: Vocals, synths, leads
- Use for: Color changes, brightness, detail

**modC = Treble Frequencies**
- Default: 2000-20000 Hz
- Targets: Hi-hats, cymbals, percussion
- Use for: Fast flickering, sparkle, texture

### Using Modulators

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
```

---

## COMMON PATTERNS

### Pattern 1: Concentric Circles

```
"Create concentric circles that pulse with bass (modA)"
```

Claude will generate something like:

```wgsl
let center = vec2(0.5);
let dist = length(uv - center);
let circles = sin(dist * 20.0 - uTime * 2.0 + uModA * 5.0);
let color = vec3(circles);
```

### Pattern 2: Rotating Shapes

```
"Create 6 triangles rotating clockwise that speed up with mids (modB)"
```

### Pattern 3: Audio-Reactive Colors

```
"Make background shift from dark blue to bright red based on bass (modA)"
```

### Pattern 4: Kaleidoscope

```
"Create 8-way kaleidoscope symmetry with fractal patterns"
```

### Pattern 5: Tunnel Effect

```
"Create perspective tunnel that zooms in time with the beat using modA"
```

---

## ITERATION WORKFLOW

### Start Simple, Build Complexity

**Iteration 1: Basic Shape**
```
"Create a simple circle in the center"
```

**Iteration 2: Add Movement**
```
"Make the circle pulse with bass (modA)"
```

**Iteration 3: Add Color**
```
"Change color from cyan to magenta based on mids (modB)"
```

**Iteration 4: Add Complexity**
```
"Duplicate the circle 5 times in a ring pattern"
```

**Iteration 5: Add Detail**
```
"Add glowing edges that flicker with treble (modC)"
```

### Testing Tips

1. **Start without audio** - Verify visual works
2. **Load audio** - Test reactivity
3. **Adjust frequency ranges** - Fine-tune response
4. **Export video** - Test at full resolution

---

## TROUBLESHOOTING

### Shader Won't Compile

**Error:** "Failed to compile shader"

**Solutions:**
1. Check for missing semicolons
2. Verify all parentheses match
3. Ensure uniform names are correct (uTime, uResolution, uModA, etc.)
4. Ask Claude: "This shader has a compile error, can you fix it?"

### Shader Compiles But Looks Wrong

**Black Screen:**
- Check color values (should be 0.0 to 1.0)
- Verify return statement exists
- Make sure color has 3 or 4 components

**Too Dark/Bright:**
- Adjust color multipliers
- Add brightness control
- Ask Claude: "Make this shader brighter"

**Not Reacting to Audio:**
- Check modulator usage
- Verify audio is loaded
- Adjust FFT frequency ranges in Controls tab
- Ask Claude: "Make the audio reactivity more obvious"

### Performance Issues

**Slow Rendering:**
- Simplify shader calculations
- Reduce loop iterations
- Ask Claude: "Make this shader more performant"

**Glitchy Output:**
- Check for division by zero
- Verify smooth() instead of step() for antialiasing
- Ask Claude: "Smooth out the edges and remove aliasing"

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
```

### Combining Modulators

```wgsl
// Bass controls size, mids control brightness
let size = 0.5 + uModA * 0.5;
let brightness = 0.3 + uModB * 0.7;
let circle = smoothstep(size + 0.01, size, length(uv - 0.5));
let color = vec3(circle * brightness);
```

### Smooth Transitions

```wgsl
// Smooth interpolation
let t = smoothstep(0.0, 1.0, uModA);
let color = mix(vec3(0.0, 0.0, 1.0), vec3(1.0, 0.0, 0.0), t);
```

### Fractal Patterns

```
"Create a Mandelbrot fractal zooming pattern"
"Generate Perlin noise texture"
"Make a Julia set visualization"
```

---

## EXAMPLE PROMPTS

### Beginner Level

```
"Create pulsing circles that get bigger with bass"

"Make a grid of squares that change color with the music"

"Create a simple gradient that shifts with audio"

"Make concentric rings that expand with the beat"
```

### Intermediate Level

```
"Create 6 rotating triangles, each reacting to different frequencies"

"Make a kaleidoscope effect that changes symmetry with bass"

"Create flowing particles that follow sine waves"

"Make a tunnel effect with perspective distortion"
```

### Advanced Level

```
"Create a fractal tree structure that grows with audio intensity"

"Make a plasma effect with three-way color mixing based on modulators"

"Create particle system with gravitational attraction to bass peaks"

"Generate Voronoi cells that morph shape with frequency analysis"
```

---

## BEST PRACTICES

### Performance

1. **Avoid expensive operations in loops**
2. **Use smoothstep() for antialiasing**
3. **Precompute constants**
4. **Limit texture sampling**
5. **Target 60 FPS**

### Visual Design

1. **Start simple, add complexity**
2. **Use complementary colors**
3. **Balance audio reactivity (not too sensitive)**
4. **Consider viewing distance**
5. **Test at performance resolution**

### Audio Reactivity

1. **Bass for big movements**
2. **Mids for color/brightness**
3. **Treble for details/flickering**
4. **Don't overuse - subtle is often better**
5. **Test with actual music, not silence**

---

## EXPORTING FOR PERFORMANCE

### Workflow

1. **Create and test shader** in Code tab
2. **Save as .oms container**
3. **Click "Render Video"** in Controls tab
4. **Use FFmpeg** to convert to ProRes 422:

```bash
ffmpeg -i shader_output.webm -c:v prores_ks -profile:v 3 \
       -vendor apl0 -pix_fmt yuv422p10le output.mov
```

5. **Import into Ableton Live** + EboSuite
6. **Perform live** with audio-reactive visuals

---

## RESOURCES

### WGSL Documentation
- https://www.w3.org/TR/WGSL/
- https://google.github.io/tour-of-wgsl/

### Shader Math
- https://thebookofshaders.com/
- https://www.shadertoy.com/

### Claude AI
- https://claude.ai/
- Ask for shader help anytime!

---

## TIPS FOR WORKING WITH CLAUDE

### Communication

**Do:**
- Be specific about visual goals
- Reference colors, shapes, movements
- Mention which modulator (modA/B/C)
- Ask for explanations if curious
- Iterate in small steps

**Don't:**
- Be vague ("make it cool")
- Ask for impossible effects
- Expect perfection on first try
- Forget to test before asking for changes

### Iteration

1. Start with a goal
2. Get first version
3. Test it
4. Request specific changes
5. Repeat until satisfied

**Example Conversation:**

```
You: "Create pulsing circles reacting to bass"
Claude: [generates shader]
You: "Make them pulse faster and change color to purple"
Claude: [updates shader]
You: "Perfect! Now add 5 circles in a ring"
Claude: [adds complexity]
```

---

## CREATIVE INSPIRATION

### Concepts to Try

- **Geometric:** Circles, triangles, spirals, fractals
- **Organic:** Waves, flow fields, particle systems
- **Abstract:** Color fields, noise textures, glitch effects
- **Mathematical:** Polar coordinates, sine waves, trigonometry
- **Symmetry:** Kaleidoscopes, mirroring, tessellation

### Color Schemes

- **Monochrome:** Black to white gradients
- **Analogous:** Blue to green, red to orange
- **Complementary:** Orange/blue, purple/yellow
- **Triadic:** Red/green/blue, cyan/magenta/yellow
- **Neon:** Bright cyan, magenta, yellow against black

### Movement Styles

- **Pulsing:** Expand/contract with beat
- **Rotation:** Spin clockwise/counterclockwise
- **Oscillation:** Back and forth motion
- **Waves:** Sine/cosine animations
- **Zoom:** Perspective changes, tunnels

---

## CONCLUSION

Creating shaders with Claude is **conversation, not programming**. Describe what you want to see, and Claude translates that into WGSL code.

**The power is in iteration:**
- Start simple
- Build complexity
- Test frequently
- Refine based on results

**No shader experience required** - just creativity and clear communication.

Now go create something amazing! 🎨✨

---

**END OF CLAUDE SHADER CREATION GUIDE**

Version: V3.4.0  
Date: November 15, 2025  
Website: https://onemanshyo.com  
License: GPL-3.0
