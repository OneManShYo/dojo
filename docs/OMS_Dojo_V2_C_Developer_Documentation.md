# ONEMANSHYO Dojo App V2 — Developer Documentation

**Version:** V2  
**Build Date:** November 4, 2025  
**License:** GPL-3.0  
**Target:** Chrome on macOS (WebGPU required)

---

## Architecture Overview

### Single-File Application

The entire app is one HTML file — markup, styles, and logic. No build step, no
package manager, no external dependencies. Open it in a browser and it runs.

**Total:** ~970 lines
- HTML structure: ~250 lines
- CSS: ~330 lines
- JavaScript: ~390 lines

This constraint is deliberate and load-bearing. The file is portable, archivable,
and inspectable — View Source is the complete source code.

### Core Systems

| System | Responsibility |
|--------|----------------|
| WebGPU Pipeline | Device init, shader compilation, render pass |
| Modulation Engine | Beat-derived Mod A/B/C values, 10 waveform generators |
| Timing | Elapsed time → beats via tempo |
| Render Loop | Per-frame uniform update and draw |
| Export | Canvas capture → WebM via MediaRecorder |
| Status | Single-surface user feedback |

---

## File Structure

```
OMS_Dojo_V2.html
├── License header (GPL-3.0, comment block)
├── <head>
│   ├── Title: ONEMANSHYO App [V2]
│   └── <style> — all CSS inline
└── <body>
    ├── .container
    │   ├── .left-panel — controls + shader editor
    │   └── .preview-panel — canvas + progress overlay
    └── <script> — all JS inline
```

### HTML Structure

```
.container
├── .left-panel
│   ├── .header               ONEMANSHYO dojo app [V2]
│   ├── .controls-section
│   │   ├── .control-row      Resolution | FPS | Orientation
│   │   ├── .control-row      Tempo | Duration
│   │   ├── .control-row      Mod A: Subdiv | Wave
│   │   ├── .control-row      Mod B: Subdiv | Wave
│   │   └── .control-row      Mod C: Subdiv | Wave
│   └── .code-section
│       ├── .code-header      Shader Code
│       └── .editor           WGSL textarea
└── .preview-panel
    ├── .preview-container
    │   └── .canvas           WebGPU render target
    └── .progress-overlay     Export progress
```

---

## CSS Architecture

### Layout System

Flexbox throughout. `.container` is a horizontal flex; `.left-panel` is fixed at
400px, `.preview-panel` takes the remainder. Body is `height: 100vh` with
`overflow: hidden` — the app fills the viewport and never scrolls as a page.

### Key Classes

| Class | Purpose |
|-------|---------|
| `.container` | Root flex, full viewport |
| `.left-panel` | 400px fixed control column |
| `.preview-panel` | Flexible render area |
| `.header` | Brand + version bar |
| `.controls-section` | Control group container |
| `.control-row` | Horizontal group of controls |
| `.control-group` | Label + input pair |
| `.editor` | WGSL textarea, monospace |
| `.canvas` | WebGPU render target |
| `.progress-overlay` | Export progress, covers preview |
| `.status` | Compile/init feedback |
| `.tempo-highlight` | Tempo field emphasis |

### Color Convention

- Cyan `#00d4ff` → magenta `#ff006e` gradient on brand
- Orange `#ffa500` on the app-name token
- Grey `#999` on the version badge
- Dark surface: `#0c0c0c` → `#1a1a1a` gradient

---

## JavaScript Architecture

### Global State

```javascript
let device, context, pipeline, canvas;   // WebGPU handles
let uniformBuffer, uniformBindGroup;     // Uniform plumbing
let startTime = Date.now();              // Timing origin
let isRendering = false;                 // Export guard
let animationId;                         // rAF handle
let currentTempo = 120;                  // BPM
let noiseState = { a: 0, b: 0, c: 0 };   // Noise generator state
```

`startTime` is the timing origin for the entire session — every beat calculation
derives from it. It resets on compile, not on tempo change, so changing tempo
mid-session shifts the rate without jumping the phase.

### Initialization Flow

```
Page load
  → initWebGPU()
      → navigator.gpu.requestAdapter()
      → adapter.requestDevice()
      → canvas.getContext('webgpu')
      → context.configure({ device, format })
      → updateCanvasSize()
  → compileShader()
      → build fragment shader (uniform struct + user WGSL)
      → device.createRenderPipeline()
      → create uniform buffer + bind group
  → render()  [loop begins]
```

### Key Functions

| Function | Role |
|----------|------|
| `initWebGPU()` | Adapter/device acquisition, context configure. Returns false and surfaces a status error if WebGPU is unavailable. |
| `compileShader()` | Prepends the uniform struct to user WGSL, builds the pipeline, allocates the uniform buffer and bind group. |
| `calculateMod(beat, subdivision, waveform)` | Returns a 0–1 value for the given beat position, subdivision, and generator. |
| `render()` | Per-frame: compute time and beat, read mod controls, compute three mod values, write the uniform buffer, encode and submit the render pass. |
| `updateCanvasSize()` | Applies resolution and orientation to canvas dimensions. |
| `startRender()` | Begins canvas capture, drives progress, triggers download on completion. |
| `finishRender()` | Stops the recorder, restores UI state. |
| `showStatus(message, type)` | The single user-feedback surface. |

---

## Shader Contract

Every compiled shader receives this uniform struct, prepended automatically:

```wgsl
struct Uniforms {
    time: f32,          // elapsed seconds since compile
    tempo: f32,         // BPM
    modA: f32,          // 0.0–1.0
    modB: f32,          // 0.0–1.0
    modC: f32,          // 0.0–1.0
    resolution: vec2<f32>,
}

@group(0) @binding(0) var<uniform> uniforms: Uniforms;
```

Shader authors write only the fragment entry point:

```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    // ...
}
```

### Buffer Layout

The uniform buffer is a `Float32Array` with explicit padding for `vec2` alignment:

```javascript
[ time, tempo, modA, modB, modC, 0 /* pad */, width, height ]
```

The padding slot at index 5 is required — WGSL aligns `vec2<f32>` to 8 bytes, so
`resolution` must start on an even float boundary. Removing it silently corrupts
the resolution values.

### Deriving Beats in a Shader

```wgsl
let beatsPerSecond = uniforms.tempo / 60.0;
let beat = uniforms.time * beatsPerSecond;
```

---

## Modulation Engine

### Subdivision Model

Subdivision is stored as a beat multiplier, inverted to a frequency:

```javascript
const freq = 1.0 / subdivision;
const phase = (beat * freq) % 1.0;
```

| Label | Value | Cycles |
|-------|-------|--------|
| Whole | 4 | once per bar |
| Half | 2 | twice per bar |
| 1/4 | 1 | once per beat |
| 1/8 | 0.5 | twice per beat |
| 1/16 | 0.25 | four times per beat |

### Generators

All return 0.0–1.0.

| Waveform | Implementation |
|----------|----------------|
| Sine | `sin(phase × 2π) × 0.5 + 0.5` |
| Cosine | `cos(phase × 2π) × 0.5 + 0.5` |
| Sawtooth | `phase` — linear ramp |
| Triangle | Piecewise linear rise/fall at half-phase |
| Square | `phase < 0.5 ? 0.0 : 1.0` |
| Chaos | Three detuned sines (3.7, 7.1, 2.3) multiplied |
| Noise | Hash of scaled beat, fractional part |
| Random | Sample-and-hold — hash of `floor(beat × freq)` |
| Fibonacci | Sine scaled by φ (1.618033988749895) |
| Prime | `sin(beat × freq × π) × cos(beat × freq × e)` |

**Note on Chaos:** it ignores `freq` and derives directly from `beat`. Subdivision
has no effect on Chaos output — this is inherent to the implementation, not a bug
to route around.

**Note on Random:** sample-and-hold quantizes to beat boundaries via `floor()`, so
it steps rather than ramps. This is what makes it useful for hard cuts.

---

## Render Loop

```javascript
function render() {
    if (!device || !pipeline) { requestAnimationFrame(render); return; }

    const currentTime = (Date.now() - startTime) / 1000.0;
    const beat = currentTime * (currentTempo / 60.0);

    // read mod controls, compute modA/B/C
    // write uniform buffer
    // encode render pass, draw 3 vertices (fullscreen triangle)
    // submit

    requestAnimationFrame(render);
}
```

Mod controls are read from the DOM every frame rather than cached. At 60fps that's
six `getElementById` calls per frame — negligible here, but it does mean control
changes take effect immediately with no state synchronization needed.

---

## Export Pipeline

```
startRender()
  → canvas.captureStream(fps)
  → new MediaRecorder(stream, { mimeType: 'video/webm;codecs=vp8',
                                videoBitsPerSecond: 8000000 })
  → collect chunks via ondataavailable
  → after duration: mediaRecorder.stop()
  → onstop: Blob → object URL → programmatic download
```

**Output filename pattern:**
```
OMS_{filename}_{resolution}p_{fps}fps_{duration}s_{tempo}bpm.webm
```

Encoding the render parameters into the filename means an exported file is
self-describing — you can tell what tempo and frame rate produced it months later
without opening anything.

**Real-time capture:** export runs at wall-clock speed. A 60-second render takes
60 seconds. There is no offline/faster-than-realtime path.

---

## Extension Points

### Adding a Waveform

Add a case to `calculateMod()` returning 0–1, then add a matching `<option>` to
each of the three waveform selects. Nothing else needs to change — the generator
name is the select value.

### Adding a Uniform

Three coordinated edits:
1. Add the field to the WGSL `struct Uniforms` in `compileShader()`
2. Add the value to the `Float32Array` in `render()` — mind `vec2` alignment
3. Increase the uniform buffer size allocation

### Adding a Fourth Mod

Add subdivision/waveform selects with `modD*` IDs, add a `calculateMod()` call in
`render()`, then follow the uniform-addition steps above.

---

## Browser Compatibility

**Requires WebGPU.** The app checks `navigator.gpu` on load and surfaces an error
if absent.

| Browser | Status |
|---------|--------|
| Chrome 113+ | Supported — primary target |
| Edge 113+ | Should work, untested |
| Safari 18+ | Should work, untested |
| Firefox | Not supported at build time |

Also requires `MediaRecorder` with VP8 support and `HTMLCanvasElement.captureStream()`.

---

## Performance Notes

### File Size
~35KB uncompressed. No external requests — the app is fully offline-capable after
first load.

### Rendering
A fullscreen triangle (3 vertices) per frame. All cost is in the fragment shader,
so performance is entirely a function of what the user writes. A heavy raymarching
loop at 4K will drop frames; a simple gradient will not.

### Memory
Recorded chunks accumulate in memory during export. Long renders at high bitrate
grow the array — a 10-minute 1080p capture at 8 Mbps is roughly 600MB held before
the blob is assembled.

---

## Debugging

**Shader compile errors** surface through `showStatus()` with the WGSL error text.
The message is the compiler's, unmodified.

**Common issues:**

| Symptom | Cause |
|---------|-------|
| Black canvas | Shader compiled but returns zero alpha, or fragment entry not named `fs_main` |
| "WebGPU not supported" | Non-WebGPU browser, or flag disabled |
| Distorted output | `resolution` misread — check uniform padding |
| Mods appear frozen | Subdivision set to Whole at low tempo; also expected for Chaos |
| Export downloads 0 bytes | Duration shorter than one frame interval |

---

## Code Style

- 4-space indentation
- `camelCase` for functions and variables
- Kebab-case CSS classes
- Comments explain intent, not mechanics
- No external dependencies — ever

---

**End of Developer Documentation**
