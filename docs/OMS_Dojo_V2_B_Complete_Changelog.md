# ONEMANSHYO Dojo App V2 — Complete Changelog

**Version:** V2  
**Release Date:** November 4, 2025  
**License:** GPL-3.0  
**Status:** Foundation release — tempo-synced modulation system

---

## Overview

V2 is the culmination of the V1 development series. Six builds took the app from a bare
WebGPU shader renderer to a tempo-synced visual instrument with three independent
modulation sources driving shader uniforms in real time.

The through-line of the series: **move from time-based animation to musical time.**
Early builds animated on elapsed seconds. V2 animates on beats.

| Build | Lines | Milestone |
|-------|-------|-----------|
| V1.0.0 | 614 | WebGPU render + WebM export |
| V1.0.1 | 631 | Orientation control |
| V1.0.2 | 666 | Export quality pass |
| V1.0.3 | 727 | Duration slider, output filename |
| V1.0.4 | 822 | Tempo (BPM) — musical time arrives |
| V1.0.5 | 970 | Mod A/B/C modulation system |
| **V2** | **970** | **Published as Dojo V2** |

Net growth across the series: 614 → 970 lines, ~58% increase.

---

## V1.0.0 — Foundation

The starting point. A single-file HTML app that compiles WGSL fragment shaders,
renders them to a canvas via WebGPU, and captures the output to WebM.

**Implemented:**
- WebGPU device/context initialization with adapter request
- WGSL fragment shader compilation from a live text editor
- Render pipeline with uniform buffer binding
- Canvas capture via `MediaRecorder` (VP8, WebM container)
- Resolution selection (720p / 1080p)
- Frame rate selection (24 / 30 / 60 FPS)
- Duration control for export length
- Compile status feedback

**Shader interface at this stage:**
```wgsl
struct Uniforms {
    time: f32,
    resolution: vec2<f32>,
}
```

Animation was driven entirely by `uniforms.time` — elapsed seconds since compile.

---

## V1.0.1 — Orientation

**Added:**
- Orientation control (Landscape / Portrait)
- Canvas dimensions recalculated on orientation change
- Render output respects selected orientation

Portrait output made the tool usable for vertical video platforms without
post-processing the export.

---

## V1.0.2 — Export Quality

**Changed:**
- Explicit bitrate on the recorder: `videoBitsPerSecond: 8000000`
- Codec pinned to `video/webm;codecs=vp8`

Exports at 1080p were showing compression artifacts on high-contrast shader
output — hard edges and rapid color transitions are exactly what a low bitrate
handles worst. Raising the ceiling to 8 Mbps resolved it.

---

## V1.0.3 — Duration & Filename

**Added:**
- Duration as a range slider (1–600 seconds) with live value readout
- Output filename field — exports name themselves instead of defaulting to browser naming

**Changed:**
- Duration moved from a number input to a slider for faster coarse adjustment

Small quality-of-life release. Naming exports at author time removed a rename
step from every single render.

---

## V1.0.4 — Tempo Sync

The pivot point of the series.

**Added:**
- Tempo control (BPM) — number input, range 60–200, default 120
- `tempo` exposed to shaders as a uniform

**Shader interface:**
```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    resolution: vec2<f32>,
}
```

Shaders could now derive beat position themselves:

```wgsl
let beatsPerSecond = tempo / 60.0;
let beat = time * beatsPerSecond;
```

**Why this matters:** every prior build animated on wall-clock seconds. Visuals
drifted against any music they were paired with. Exposing tempo let shader authors
write motion that lands on beats and bars — the foundation everything after this
depends on.

---

## V1.0.5 — Modulation System

**Added:**
- Three independent modulation sources: Mod A, Mod B, Mod C
- Per-mod subdivision selector: Whole, Half, 1/4, 1/8, 1/16
- Per-mod waveform selector: 10 generators
- `modA` / `modB` / `modC` exposed as shader uniforms, normalized 0.0–1.0
- `calculateMod()` engine computing all three per frame from beat position

**Shader interface (final V2 contract):**
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

**Waveform generators:**

| Waveform | Behavior |
|----------|----------|
| Sine | Smooth oscillation, phase-mapped to 0–1 |
| Cosine | Sine offset by quarter phase |
| Sawtooth | Linear ramp, hard reset at cycle boundary |
| Triangle | Linear rise and fall |
| Square | Binary gate at half-phase |
| Chaos | Three detuned sines multiplied — non-repeating drift |
| Noise | Hash-based smooth noise |
| Random | Sample-and-hold, new value per beat boundary |
| Fibonacci | Sine scaled by the golden ratio (φ ≈ 1.618) |
| Prime | Sine × cosine at π and e ratios |

**Subdivision math:**

Subdivision is stored as a beat multiplier (Whole = 4, Half = 2, 1/4 = 1, 1/8 = 0.5,
1/16 = 0.25). Phase derives from beat position:

```javascript
const freq = 1.0 / subdivision;
const phase = (beat * freq) % 1.0;
```

A 1/16 subdivision cycles four times per beat; Whole cycles once per bar.

**Defaults shipped:**
- Mod A — 1/4, Sine
- Mod B — 1/8, Sawtooth
- Mod C — 1/16, Square

Three mods at different rates was a deliberate choice. One modulator gives you
pulsing. Three at different subdivisions gives you polyrhythm — layered motion that
reads as musical rather than mechanical.

---

## Published as V2

V1.0.5 ships as **Dojo V2** under the ONEMANSHYO project.

**Applied at publication:**
- GPL-3.0 license header
- ONEMANSHYO branding (`ONEMANSHYO dojo app [V2]`)
- Title tag standardized to `ONEMANSHYO App [V2]`
- Version badge styling aligned to project standard (11px, grey)
- Default shader header comment updated to Dojo naming

No functional changes were made at publication. The app is the V1.0.5 build.

---

## Known Limitations

Documented as the honest state of V2, not as defects:

- **No audio input.** Modulation is tempo-derived, not audio-reactive. Tempo is set
  by hand and the app has no awareness of what's actually playing.
- **No preset system.** Mod configurations are lost on reload.
- **No shader library.** One shader at a time, pasted into the editor.
- **Fixed uniform set.** Six uniforms, not extensible without editing the app.
- **Export is VP8 WebM only.** No ProRes or MP4 path.
- **No transport controls.** Compile starts the render loop; there is no scrub,
  no pause, no loop region.

---

## What Came Next

The V2 series (V2.0.1 → V2.9.0) addressed audio directly — waveform display,
FFT analysis, real audio-reactive modulation, In/Out bar regions, and the
container format that eventually became `.oms`.

---

**End of Changelog**
