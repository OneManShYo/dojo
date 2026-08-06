# ONEMANSHYO Dojo App V2 — User Guide

**Version:** V2  
**License:** GPL-3.0

---

## Table of Contents

1. [Getting Started](#getting-started)
2. [The Interface](#the-interface)
3. [Output Controls](#output-controls)
4. [Tempo](#tempo)
5. [The Modulation System](#the-modulation-system)
6. [Writing Shaders](#writing-shaders)
7. [Exporting](#exporting)
8. [Workflow](#workflow)
9. [Troubleshooting](#troubleshooting)
10. [Appendix A: Waveform Reference](#appendix-a-waveform-reference)
11. [Appendix B: Shader Recipes](#appendix-b-shader-recipes)

---

## Getting Started

Open `OMS_Dojo_V2.html` in Chrome. That's the whole install.

On load the app initializes WebGPU and compiles the default shader — a rotating
spiral driven by all three mods. If you see motion in the preview, everything works.

If you see "WebGPU not supported," you're on a browser without WebGPU or with the
flag disabled.

---

## The Interface

Two panels.

**Left — controls and code**
- Header with brand and version
- Output settings: resolution, frame rate, orientation
- Tempo and duration
- Three modulation rows (A, B, C)
- Shader code editor

**Right — preview**
- Live WebGPU canvas
- Export progress overlay when rendering

---

## Output Controls

**Resolution** — 720p or 1080p. Affects both preview and export. Higher resolution
means more fragment shader work per frame; a heavy shader that runs smoothly at
720p may drop frames at 1080p.

**FPS** — 24, 30, or 60. Applies to export capture.

**Orientation** — Landscape or portrait. Portrait output targets vertical video
without post-processing.

**Duration** — 1 to 600 seconds. Export length only; preview runs indefinitely.

---

## Tempo

Range 60–200 BPM, default 120.

Tempo is the spine of the app. It's passed to shaders as a uniform and it drives
every modulation calculation. Set it to match the track you're scoring and the
visuals land on the beat.

Changing tempo mid-session changes the rate without resetting phase — motion speeds
up or slows down rather than jumping.

---

## The Modulation System

Three independent sources — Mod A, Mod B, Mod C — each producing a value between
0.0 and 1.0, updated every frame from the current beat position.

Each has two settings.

### Subdivision

How often the modulator completes a cycle.

| Setting | Cycle |
|---------|-------|
| Whole | Once per bar (4 beats) |
| Half | Twice per bar |
| 1/4 | Once per beat |
| 1/8 | Twice per beat |
| 1/16 | Four times per beat |

### Waveform

The shape of the cycle. Ten generators — see [Appendix A](#appendix-a-waveform-reference).

### Defaults

| Mod | Subdivision | Waveform |
|-----|-------------|----------|
| A | 1/4 | Sine |
| B | 1/8 | Sawtooth |
| C | 1/16 | Square |

These aren't arbitrary. Three modulators at the same rate produce uniform pulsing —
everything moving together, which reads as mechanical. Three at different
subdivisions produce layered motion: a slow smooth sweep, a mid-rate ramp, and
fast punctuation. That's what makes output feel musical.

### Using Mods

Mods are raw material — they don't do anything until a shader reads them. In WGSL:

```wgsl
let brightness = uniforms.modA;              // direct
let scale = 1.0 + uniforms.modB * 0.5;       // scaled range
let toggle = step(0.5, uniforms.modC);       // gate
```

**Practical guidance:** slow subdivisions on structural parameters (scale,
position, color), fast subdivisions on accents (brightness flashes, edge
highlights). A 1/16 Square driving overall scale looks like a strobe seizure; the
same mod driving a subtle edge glow looks like rhythm.

---

## Writing Shaders

### The Contract

You write only the fragment entry point. The app prepends the uniform struct:

```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    resolution: vec2<f32>,
}
```

Your shader must define:

```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    // your code
}
```

### Normalized Coordinates

```wgsl
let uv = coord.xy / uniforms.resolution;   // 0–1
let center = uv - 0.5;                     // -0.5 to 0.5, origin at center
```

### Beat Position

```wgsl
let beatsPerSecond = uniforms.tempo / 60.0;
let beat = uniforms.time * beatsPerSecond;
```

`beat` counts up continuously — 0, 1, 2, 3... one per beat. For bar position,
`beat / 4.0`. For a repeating cycle, `beat % 4.0`.

### Compiling

Edit the code and compile. Errors surface in the status area with the WGSL
compiler's own message, including line numbers.

Compile resets the timing origin, so motion restarts from beat zero.

---

## Exporting

1. Set resolution, FPS, orientation, and duration
2. Enter a filename
3. Click Render

Capture runs in real time — a 60-second export takes 60 seconds. Progress shows in
the preview overlay. When it finishes the file downloads automatically.

**Filename pattern:**
```
OMS_{yourname}_{resolution}p_{fps}fps_{duration}s_{tempo}bpm.webm
```

Every render parameter is in the name, so an export is self-describing months
later without opening anything.

**Format:** WebM, VP8, 8 Mbps. For editing workflows that want ProRes or MP4,
convert with FFmpeg after export.

---

## Workflow

A typical pass:

1. **Set tempo first.** Match the track. Everything downstream depends on it.
2. **Start with the default shader.** Confirm rendering works before writing code.
3. **Write with one mod.** Get one parameter moving correctly before layering.
4. **Add the second and third.** Vary subdivisions — don't stack three at the same rate.
5. **Watch a full cycle.** Let it run several bars. Motion that looks good for two
   seconds often reveals itself as repetitive over sixteen.
6. **Tune subdivisions before waveforms.** Rate is usually the problem, not shape.
7. **Export.**

---

## Troubleshooting

| Symptom | Likely cause |
|---------|--------------|
| Black canvas | Shader returns zero alpha, or entry point isn't named `fs_main` |
| "WebGPU not supported" | Browser lacks WebGPU or the flag is off |
| Compile error | Read the status message — it's the WGSL compiler's own text with line numbers |
| Nothing appears to move | Subdivision set to Whole at low tempo — slow but working. Also expected for Chaos. |
| Chaos doesn't respond to subdivision | Known behavior — Chaos derives from beat directly and ignores subdivision |
| Frame drops | Shader too heavy for the resolution — drop to 720p or simplify |
| Export downloads nothing | Duration shorter than one frame interval |
| Motion drifts against music | Tempo doesn't match the track |

---

## Appendix A: Waveform Reference

All generators output 0.0–1.0.

| Waveform | Character | Good for |
|----------|-----------|----------|
| **Sine** | Smooth, symmetric | Breathing, gentle sweeps, organic motion |
| **Cosine** | Sine offset a quarter cycle | Pairing with Sine for phase-offset motion |
| **Triangle** | Linear rise and fall | Even sweeps without sine's easing |
| **Sawtooth** | Ramp up, hard reset | Build-and-drop, sweeps that snap back |
| **Square** | Hard on/off | Gates, strobes, binary state changes |
| **Noise** | Smooth randomness | Organic wobble, imperfection |
| **Random** | Sample-and-hold, steps on beats | Hard cuts, glitch, per-beat variation |
| **Chaos** | Non-repeating drift | Long-form evolution that never loops |
| **Fibonacci** | Golden-ratio sine | Slow evolution that rarely repeats |
| **Prime** | π/e ratio product | Complex non-repeating patterns |

**On Sawtooth:** the hard reset at cycle end is a visible snap. That's the point —
it's the shape for tension and release. If the snap reads as a glitch rather than a
hit, the subdivision is probably too fast.

**On Random:** it steps rather than ramps, quantized to beat boundaries. This makes
it the only generator that produces genuinely discrete per-beat variation.

**On Chaos:** ignores subdivision entirely. Use it when you want motion that never
loops, and accept that you can't control its rate.

---

## Appendix B: Shader Recipes

### Beat-Synced Pulse

```wgsl
let beat = uniforms.time * (uniforms.tempo / 60.0);
let pulse = 1.0 - fract(beat);          // 1.0 on beat, decays to 0
let brightness = pulse * 0.8 + 0.2;
```

Decays from each beat rather than pulsing symmetrically — reads as a hit, not a wobble.

### Mod-Driven Rotation

```wgsl
let uv = coord.xy / uniforms.resolution;
let center = uv - 0.5;
let angle = beat * uniforms.modA * 2.0;
let rotated = vec2<f32>(
    center.x * cos(angle) - center.y * sin(angle),
    center.x * sin(angle) + center.y * cos(angle)
);
```

### Layered Motion

```wgsl
let scale  = 1.0 + uniforms.modA * 0.3;   // slow — structure
let warp   = uniforms.modB * 0.5;          // mid  — movement
let accent = step(0.7, uniforms.modC);     // fast — punctuation
```

Assign each mod a role by rate. Slow to structure, fast to accent. Inverting this —
fast mods on structural parameters — is the most common way output starts looking
frantic instead of rhythmic.

### Radial Distance Field

```wgsl
let dist = length(center);
let ring = smoothstep(0.3, 0.31, dist) - smoothstep(0.4, 0.41, dist);
```

`smoothstep` pairs give antialiased edges. Hard comparisons produce jagged edges
that alias badly on export.

---

**End of User Guide**
