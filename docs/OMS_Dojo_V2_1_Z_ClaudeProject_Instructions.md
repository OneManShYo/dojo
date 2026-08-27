# ONEMANSHYO Dojo - Claude Project Instructions

**Official Documentation for AI Assistant Context - V2_1**

---

## PROJECT IDENTITY

**Project Name:** ONEMANSHYO Dojo (OneManShYo Dojo)  
**Project Codename:** OMS  
**Current Version:** V2_1 (Published November 2025)  
**Creator:** Wes Smith  
**Project Type:** Audio-Reactive WebGPU Shader Tool  
**Status:** Production Release  
**Lines of Code:** ~2,680 (HTML/CSS/JavaScript single-file application)  
**File Size:** ~115 KB

---

## WHAT'S NEW IN V2_1

### Major UI Evolution

**Tab System:**
- Split-view interface (Controls Tab / Code Tab)
- Dedicated workspace for each mode
- Cleaner organization

**6X Bigger Canvas:**
- From 320x180 to 1920x1080 (Full HD preview)
- Exactly matches 1080p export resolution
- Better for detailed shader work

**Waveform on Canvas:**
- Moved from left panel to canvas area
- 1920px wide (6x bigger)
- Better visibility and integration

**Audio Controls Strip:**
- New strip below waveform
- Transport controls: ▶ (Play/Pause) and ⏹ (Stop)
- OMS Pink (#ff006e) styling
- IN/OUT beats always visible

**Refined Left Panel:**
- Now only shader controls (5 rows)
- All parameters visible at once
- No scrolling required

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

### UI Layout (V2_1)

**Left Panel:**
- Tab bar (Controls / Code)
- Controls Tab: 5 rows of shader parameters
- Code Tab: WGSL shader editor

**Right Panel:**
- Canvas (1920x1080 HD preview)
- Waveform (1920x180)
- Audio controls strip (▶ ⏹ IN OUT)
- Status bar

### Audio System
- FFT frequency analysis (Bass/Mid/Treble)
- Audio-reactive modulators
- Real-time waveform visualization
- Transport controls with OMS Pink styling

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
**V2_1:** Tab system & canvas expansion (CURRENT)

---

## AI ASSISTANT GUIDELINES

### When Assisting with This Project

**Context Awareness:**
- V2_1 has tab system and 1920x1080 canvas
- Waveform is on canvas, not in left panel
- Audio controls moved to canvas area
- Left panel is now purely shader controls

**Focus Areas:**
- Tab system functionality
- Canvas size implications
- Waveform integration on canvas
- Transport controls styling (OMS Pink)
- Shader previewing in HD

**Don't:**
- Suggest complex Ableton integration
- Recommend replacing with TouchDesigner
- Over-complicate the standalone nature
- Assume server-side requirements

### Code Architecture Notes

**Single-File Application:**
- HTML + CSS + JavaScript (~2,680 lines)
- No external dependencies except browser APIs
- No build process required

**Key Changes in V2_1:**
- Tab switching JavaScript
- Canvas sizing (1920x1080 default)
- Waveform positioning on canvas
- Transport button styling
- Layout reorganization

---

## FILE STRUCTURE V2_1

```
OMS_Dojo_V2_1.html
├── HTML Structure
│   ├── Left Panel
│   │   ├── Header
│   │   ├── Tab Bar (Controls / Code)
│   │   ├── Controls Tab Content (5 rows)
│   │   └── Code Tab Content (shader editor)
│   └── Right Panel
│       ├── Canvas (1920x1080)
│       ├── Waveform Section (1920x180)
│       ├── Audio Controls Strip
│       └── Status Bar
└── JavaScript
    ├── Tab switching
    ├── WebGPU system
    ├── Audio system
    ├── Modulation system
    ├── Waveform system
    └── Video export
```

---

## DEFAULT VALUES V2_1

**Canvas:**
- Size: 1920x1080 (was 320x180 in V2.0.2)
- Updates with orientation/resolution

**Render Settings:**
- Resolution: 1080p
- Orientation: Portrait
- FPS: 30
- Tempo: 120 BPM

**Audio:**
- IN Beat: 64
- OUT Beat: 128
- Gains: 0.1 for all bands

---

## KEYBOARD SHORTCUTS

**Current:**
- Spacebar: Play/Pause audio
- Left Arrow: Previous beat
- Right Arrow: Next beat

**All preserved from V2.0.2!**

---

## KNOWN LIMITATIONS & FUTURE

### Resolved in V2_1
- ✓ Small canvas preview
- ✓ Tiny waveform
- ✓ Scattered audio controls
- ✓ No transport feedback

### Future Considerations
- Section headers for control groups
- Draggable IN/OUT handles
- Preset system
- Beat grid overlay

---

## PERFORMANCE V2_1

**Canvas Size Impact:**
- 1920x1080 has minimal impact on modern GPUs
- Main bottleneck is shader complexity
- No performance regression from V2.0.2

**Benefits:**
- 1:1 match with 1080p export
- Better visual feedback
- More accurate preview

---

## PRODUCTION RELEASE PACKAGE

**Standard 10-File Package:**
1. OMS_Dojo_V2_1.html
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

## CURRENT STATUS

- **Version:** V2_1
- **Status:** Production Release
- **Published:** November 2025
- **Package:** 10 files (~338 KB total)
- **Development:** 6 iterations (A-F)

---

**Last Updated:** November 2025  
**Version:** V2_1  
**For:** AI Assistants working on ONEMANSHYO Dojo

*This document provides complete context for AI assistants. All technical decisions, workflows, and version history are documented to ensure contextually-aware assistance.*
