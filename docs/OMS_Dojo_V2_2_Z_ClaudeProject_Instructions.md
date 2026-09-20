# ONEMANSHYO Dojo - Claude Project Instructions

**Official Documentation for AI Assistant Context - V2_2**

---

## PROJECT IDENTITY

**Project Name:** ONEMANSHYO Dojo (OneManShYo Dojo)  
**Project Codename:** OMS  
**Current Version:** V2_2 (Published November 2025)  
**Creator:** Wes Smith  
**Project Type:** Audio-Reactive WebGPU Shader Tool  
**Status:** Production Release  
**Lines of Code:** ~3,185 (HTML/CSS/JavaScript single-file application)  
**File Size:** ~150 KB

---

## WHAT'S NEW IN V2_2

### Automatic BPM Detection
- Analyzes uploaded audio buffer automatically
- Detects tempo in 60-180 BPM range (0.5 step precision)
- Onset detection algorithm with autocorrelation
- User can override by typing if needed

### Nudge Field for Fine-Tuning
- Shifts entire loop by milliseconds (±5000ms range)
- Arrow keys: ±1ms (Up/Down), ±4ms (Left/Right)
- Applied to all audio position jumps
- Perfect for video render sync adjustments

### UI Clarity Improvements
- "Audio BPM" → "BPM" (cleaner label)
- "IN" → "Loop In" (clearer purpose)
- "OUT" → "Loop Out" (clearer purpose)
- Updated all tooltips

### Loop Button
- Toggle between loop IN→OUT vs continuous play
- Visual state indicator (active/inactive)
- Default: Loop enabled

### Simplified Arrow Keys
- Always move both Loop In/Out together
- Maintains loop length (slides window)
- Grid button disabled (reserved for future visual grid)

---

## EXECUTIVE OVERVIEW

ONEMANSHYO Dojo is a browser-based audio-reactive visual synthesis tool designed for creating music visualizers and video content. Part of the **OneManShYo Streaming Project** - a one-man operation focused on live DJ performance and streaming workflows.

### Purpose
- Enable creative shader programming with audio-reactive capabilities
- Generate video content for YouTube live streaming and social media
- Provide synth-style musical parameters for visual experimentation
- Offer a standalone tool that doesn't require complex software integration

### Philosophy
**"Go wild, design whatever you want in the shader"** - Creative freedom is paramount.

---

## CORE FEATURES

### UI Layout (V2_2)

**Left Panel:**
- Tab bar (Controls / Code)
- Controls Tab: 6 rows of shader parameters
- Code Tab: WGSL shader editor with management buttons

**Right Panel:**
- Canvas (1920x1080 HD preview)
- Waveform (1920x180)
- Audio controls strip (â–¶ â¹ Loop | BPM | Loop In/Out | Nudge)
- Status bar

### Audio System (V2_2 Enhanced)
- **Automatic BPM detection** on load
- FFT frequency analysis (Bass/Mid/Treble)
- Audio-reactive modulators
- Real-time waveform visualization
- **Loop button** for playback control
- **Nudge field** for fine-timing
- Transport controls

### Shader System
- WebGPU rendering
- WGSL fragment shaders
- 3 independent modulators (A, B, C)
- 10 waveform types + 4 meta-modes
- Tempo-synced subdivisions

### Video Output
- Resolutions: 720p, 1080p, 1440p, 2160p
- Frame rates: 24, 30, 60 FPS
- Orientations: Landscape, Portrait, Square
- Format: WebM (VP8, 8Mbps)

---

## VERSION HISTORY SUMMARY

**V1.0.5:** Pre-audio baseline  
**V2.0.0:** Audio system integration (MAJOR)  
**V2.0.1:** Beat-based navigation  
**V2.0.2:** UI polish & Square orientation  
**V2.1.0:** Tab system & canvas expansion  
**V2_2:** BPM detection & Nudge field (CURRENT)

---

## AI ASSISTANT GUIDELINES

### When Assisting with This Project

**Context Awareness:**
- V2_2 has automatic BPM detection
- Nudge field for millisecond-precision timing
- Loop button for playback control
- UI labels renamed for clarity
- Arrow keys always move both brackets together
- Grid button disabled (reserved for future)

**Focus Areas:**
- BPM detection algorithm accuracy
- Nudge field workflow integration
- Loop timing improvements
- Audio control strip usability
- Shader performance optimization

**Don't:**
- Suggest complex Ableton integration
- Recommend replacing with TouchDesigner
- Over-complicate the standalone nature
- Assume server-side requirements
- Ignore the automatic BPM detection feature

### Code Architecture Notes

**Single-File Application:**
- HTML + CSS + JavaScript (~3,185 lines)
- No external dependencies except browser APIs
- No build process required

**Key Changes in V2_2:**
- `detectBPM(audioBuffer)` function added
- `nudgeOffset` variable for timing adjustment
- `loopEnabled` variable for loop mode
- Simplified arrow key handler (always coupled)
- Nudge applied to all audio position jumps

---

## FILE STRUCTURE V2_2

```
OMS_Dojo_V2_2.html (~3,185 lines)
├── HTML Structure
│   ├── Full-width Header
│   ├── Container
│   │   ├── Left Panel
│   │   │   ├── Tab Bar (Controls / Code)
│   │   │   ├── Controls Tab (6 rows)
│   │   │   └── Code Tab (shader editor + management buttons)
│   │   └── Right Panel
│   │       ├── Canvas (1920x1080)
│   │       ├── Waveform Section (1920x180)
│   │       └── Audio Controls Strip (â–¶ â¹ Loop | BPM | Loop In/Out | Nudge | Grid)
│   └── Full-width Status Bar
└── JavaScript
    ├── BPM detection algorithm
    ├── Nudge system
    ├── Loop toggle
    ├── WebGPU system
    ├── Audio system
    ├── Modulation system
    ├── Waveform system
    └── Video export
```

---

## DEFAULT VALUES V2_2

**Canvas:**
- Size: 1920x1080 (Full HD)
- Updates with orientation/resolution

**Render Settings:**
- Resolution: 1080p
- Orientation: Portrait
- FPS: 30
- Shader Tempo: 120 BPM

**Audio Controls:**
- BPM: Auto-detected (60-180 range)
- Loop In: 64 beats
- Loop Out: 128 beats
- Nudge: 0 ms
- Loop: Enabled (default)
- Gains: 0.1 for all bands

**Modulators:**
- Mod A: 1/4 subdivision, Sine wave
- Mod B: Whole subdivision, Sawtooth wave
- Mod C: Whole subdivision, Triangle wave

---

## KEYBOARD SHORTCUTS V2_2

**Audio Controls (Global):**
- Spacebar: Play/Pause audio

**When focused on Loop In or Loop Out:**
- Up Arrow: Both +1 beat (shift window right)
- Down Arrow: Both -1 beat (shift window left)
- Left Arrow: Both -4 beats (shift window left 1 bar)
- Right Arrow: Both +4 beats (shift window right 1 bar)

**When focused on Nudge field:**
- Up Arrow: +1ms
- Down Arrow: -1ms
- Left Arrow: -4ms
- Right Arrow: +4ms

---

## AUDIO CONTROLS STRIP LAYOUT

```
[Play ▶] [Stop ■] [Loop] | BPM: [120.0] | Loop In: [64] Loop Out: [128] Nudge: [0 ms] [Grid]
                                                                                      ↑
                                                                                  (disabled)
```

**Components:**
1. **Play** - Play/Pause (Spacebar)
2. **Stop** - Stop and reset to Loop In
3. **Loop** - Toggle loop mode (active/inactive)
4. **BPM** - Auto-detected tempo (type to override)
5. **Loop In** - Loop start beat (click label → jump to 0)
6. **Loop Out** - Loop end beat (click label → jump to end)
7. **Nudge** - Fine-tune timing by milliseconds
8. **Grid** - Disabled (reserved for future visual grid)

---

## KNOWN LIMITATIONS & FUTURE

### Resolved in V2_2
- ✓ Manual BPM entry (now auto-detected)
- ✓ No fine-timing control (Nudge field added)
- ✓ Confusing Grid/Free toggle (simplified)
- ✓ Generic field labels (now "Loop In"/"Loop Out")

### Future Considerations
- Visual beat grid overlay (Grid button activation)
- Expanded BPM range (40-240)
- Per-section nudge offsets
- Draggable IN/OUT handles
- Preset system
- Multiple loop regions

---

## PERFORMANCE V2_2

**BPM Detection Impact:**
- Analysis time: 1-2 seconds on audio load (one-time)
- CPU usage: Minimal (single analysis pass)
- No impact on playback or rendering

**Canvas & Rendering:**
- 1920x1080 has minimal impact on modern GPUs
- Main bottleneck: Shader complexity
- No performance regression from V2.1.0

**Benefits:**
- 1:1 match with 1080p export
- Better visual feedback
- Accurate preview

---

## PRODUCTION RELEASE PACKAGE

**Standard 10-File Package:**
1. OMS_Dojo_V2_2.html
2. A_Deliverables.txt
3. B_Complete_Changelog.md
4. C_Developer_Documentation.md
5. D_Release_Summary.md
6. E_Production_Release_Guide.md
7. F_Project_Instructions.md (this file)
8. G_Claude_ShaderCreation_Guide.md
9. UserGuide_All.md
10. UserGuide_FFmpeg_Video_Guide.md

---

## DEVELOPMENT GUIDELINES

### When Assisting with Code

**Assume:**
- User understands shader programming
- Familiarity with audio-reactive visuals
- Knowledge of music production workflows
- Experience with DAWs (Ableton, etc.)

**Focus On:**
- BPM detection accuracy improvements
- Nudge workflow optimization
- Audio analysis enhancements
- UI/UX for audio controls
- Shader performance

**Don't:**
- Suggest complex Ableton integration
- Recommend server-side processing
- Over-complicate standalone nature
- Break single-file architecture

### Code Notes

**BPM Detection:**
- Onset detection with RMS energy analysis
- Autocorrelation scoring for tempo matching
- Range: 60-180 BPM (0.5 step)
- Fallback: 120 BPM if insufficient onsets

**Nudge System:**
- Global offset applied to all audio jumps
- Stored in seconds (converted from milliseconds)
- Range: -5000ms to +5000ms

**Loop System:**
- Boolean flag: `loopEnabled`
- True: Repeats Loop In → Loop Out
- False: Plays continuously

---

## TYPICAL WORKFLOW V2_2

```
1. Upload Audio
   └─ BPM auto-detects → [127.5 BPM]
   └─ Audio jumps to Loop In (64)

2. Find Loop Section
   └─ Arrow keys: Shift Loop In/Out window
   └─ Right Arrow: Try next bar →
   └─ Left Arrow: Try previous bar ←

3. Fine-Tune Timing
   └─ Click Nudge field
   └─ Arrow keys: ±1ms or ±4ms
   └─ Or type value (e.g., -50 for -50ms)

4. Edit Shader
   └─ Switch to Code tab
   └─ Modify WGSL fragment shader
   └─ Click [Compile]

5. Render Video
   └─ Switch to Controls tab
   └─ Set Orientation, Resolution, FPS
   └─ Enter Filename
   └─ Click [R] Render Video
```

---

## CURRENT STATUS

- **Version:** V2_2
- **Status:** Production Release
- **Published:** November 2025
- **Package:** 10 files (~281 KB total)
- **Development:** 5 iterations (A, D, F, G, H)

---

**Last Updated:** November 2025  
**Version:** V2_2  
**For:** AI Assistants working on ONEMANSHYO Dojo

*This document provides complete context for AI assistants. All technical decisions, workflows, and version history are documented to ensure contextually-aware assistance.*
