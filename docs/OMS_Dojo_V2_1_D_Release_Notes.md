# ONEMANSHYO Dojo V2_1 - Release Notes

**Version:** V2_1  
**Status:** Production Release  
**Published:** November 2025  
**Previous Version:** V2 (November 4, 2025)

**→ START HERE for overview and quick start**

---

## Executive Summary

ONEMANSHYO Dojo V2_1 is a major UI evolution that dramatically improves the creative workflow with a **tab system**, **6x bigger canvas**, and better workspace organization. The canvas now shows your shader in **full 1920x1080 HD** - exactly what your 1080p export will look like.

This release represents a fundamental reimagining of the interface while preserving all functionality from V2.0.2. Everything works the same, but looks and feels much better.

---

## What's New

### 🎯 Tab System
**Controls Tab / Code Tab split-view interface**

Switch between parameter controls and shader editing with a single click. No more scrolling past controls to reach your code - each workflow gets its own dedicated space.

**Benefits:**
- Cleaner interface
- More focused workflows
- Better use of screen space
- Faster navigation

### 📐 6X Bigger Canvas
**From 320x180 to 1920x1080**

The preview canvas is now full HD - 6 times bigger than V2.0.2. What you see is exactly what you'll get when rendering at 1080p.

**Benefits:**
- Crystal-clear preview
- See fine details in shaders
- 1:1 match with 1080p export
- Better visual feedback

### 🎵 Waveform on Canvas
**Moved from left panel to canvas area**

The waveform is now 6x bigger (1920px wide) and lives with the visual output. Better integration, better visibility, better workflow.

**Benefits:**
- Much easier to see audio structure
- More space in left panel
- Natural grouping with canvas
- Better scrubbing experience

### 🎮 Audio Controls Strip
**New control strip below waveform**

Transport controls (▶ Play/Pause and ⏹ Stop) in OMS Pink styling, plus IN/OUT beats always visible.

**Benefits:**
- Visual playback feedback
- Quick access to transport
- IN/OUT beats always in view
- Professional DAW-style layout

### 🎨 Refined Left Panel
**Cleaner 5-row shader controls**

With audio moved to canvas, the left panel is now focused purely on shader parameters - orientation, timing, and modulators.

**Benefits:**
- All shader controls visible at once
- No scrolling required
- Cleaner organization
- Faster parameter adjustments

---

## Quick Start

### 1. Open the Application
- Open `OMS_Dojo_V2_1.html` in Chrome 113+, Edge 113+, or Safari TP
- Status bar should show "WebGPU Ready"

### 2. Upload Audio (Optional)
- Click anywhere in the canvas area to upload audio
- Waveform appears in the waveform section (1920x180)
- Transport controls appear (▶ ⏹) in OMS Pink

### 3. Adjust Shader Parameters (Controls Tab)
- Set Shader Tempo to match your audio BPM
- Adjust Modulators A/B/C for desired visual motion
- Increase Gains for more audio reactivity

### 4. Edit Shader (Code Tab)
- Click "Code" tab to switch to shader editor
- Edit WGSL fragment shader
- Press [C] to compile and preview

### 5. Render Video
- Switch back to "Controls" tab
- Set Resolution, FPS, Filename
- Press [R] to render
- Video downloads automatically with embedded audio

---

## Key Features Summary

### Audio System
- FFT frequency analysis (Bass/Mid/Treble)
- Audio-reactive modulators
- Real-time waveform visualization
- Transport controls (▶ ⏹)
- IN/OUT beat range selection
- Embedded audio in video exports

### Shader System
- WebGPU rendering (modern GPU acceleration)
- WGSL fragment shaders
- 3 independent modulators (A, B, C)
- 10 waveform types + 4 meta-modes
- Tempo-synced subdivisions
- Real-time preview

### Video Output
- Resolutions: 720p, 1080p, 1440p, 2160p
- Frame rates: 24, 30, 60 FPS
- Orientations: Landscape (16:9), Portrait (9:16), Square (1:1)
- Format: WebM (VP8, 8Mbps)
- Automatic filename generation

---

## Migration from V2.0.2

### What Changed

**UI Layout:**
- ✓ Tab system added (Controls / Code)
- ✓ Canvas: 320x180 → 1920x1080 (6x bigger)
- ✓ Waveform: Moved to canvas area (6x wider)
- ✓ Audio controls: New strip below waveform
- ✓ Left panel: Now only shader controls (5 rows)

**What Stayed the Same:**
- ✓ All keyboard shortcuts (Spacebar, Arrows)
- ✓ All rendering functionality
- ✓ All shader parameters
- ✓ All modulators and audio reactivity
- ✓ All file formats and output specs

### Breaking Changes

**None!** This is a UI evolution, not a functional change. Everything you could do in V2.0.2, you can still do in V2_1.

### Learning Curve

**Minimal.** If you're familiar with V2.0.2:
1. Use tabs to switch between controls and code
2. Audio interactions are now on canvas (not left panel)
3. Enjoy the bigger preview!

---

## System Requirements

### Browser Support
- Chrome 113+ (recommended)
- Edge 113+
- Safari Technology Preview
- WebGPU support required

### Hardware
- Modern GPU with 4GB+ VRAM
- 8GB+ system RAM
- Any modern CPU

### APIs Required
- WebGPU
- Web Audio API
- MediaRecorder API
- Canvas 2D API

---

## Performance Notes

### Canvas Size Impact

**Common Question:** "Won't a 1920x1080 canvas be slower?"

**Answer:** No significant impact on modern hardware.

**Why:**
- Modern GPUs handle 1080p easily
- The render loop is the same
- Video export renders at target resolution anyway (always did)
- Main performance factor is shader complexity, not canvas size

**Benefit:**
- What you see is what you get (1:1 match with 1080p export)
- No surprises when rendering

---

## Use Cases

### Primary (Creator's Intent)
- YouTube live streaming visuals
- DJ set visualizations
- Music video production

### Social Media
- **Instagram Feed:** Square (1:1)
- **Instagram Reels:** Portrait (9:16)
- **TikTok:** Portrait (9:16)
- **YouTube Shorts:** Portrait (9:16)
- **YouTube Videos:** Landscape (16:9)

### Creative
- Shader programming playground
- Generative art experiments
- Audio-reactive explorations
- VJ loop pre-rendering

---

## Documentation Package

This release includes 10 files:

**Application:**
1. `OMS_Dojo_V2_1.html` - The tool

**Core Documentation:**
2. `A_Deliverables.txt` - Package manifest (you've read this)
3. `B_Complete_Changelog.md` - Version history
4. `C_Developer_Documentation.md` - Technical details
5. `D_Release_Summary.md` - This file (overview)

**Extended Documentation:**
6. `E_Production_Release_Guide.md` - Production workflow
7. `F_Project_Instructions.md` - AI assistant context
8. `G_Claude_ShaderCreation_Guide.md` - Shader creation guide

**User Guides:**
9. `UserGuide_All.md` - Complete user manual
10. `UserGuide_FFmpeg_Video_Guide.md` - Post-processing

**Total Package Size:** ~338 KB

---

## Version History at a Glance

| Version | Date | Focus | Canvas Size |
|---------|------|-------|-------------|
| V1.0.5 | Sep 2025 | Pre-audio | 320x180 |
| V2.0.0 | Oct 2025 | Audio system | 320x180 |
| V2.0.1 | Oct-Nov 2025 | Beat navigation | 320x180 |
| V2.0.2 | Nov 5, 2025 | UI polish | 320x180 |
| **V2_1** | **Nov 2025** | **Tab system & canvas** | **1920x1080** |

---

## Development Iterations

V2_1 went through 6 iterations (A-F):

- **V2_1_A:** Tab system implementation
- **V2_1_B:** Canvas expansion (6x bigger)
- **V2_1_C:** Waveform moved to canvas
- **V2_1_D:** Audio controls strip
- **V2_1_E:** Transport controls (OMS Pink)
- **V2_1_F:** Filename field fix
- **V2_1:** Production release (this version)

---

## Known Issues

### Resolved in V2_1
- ✓ Small canvas preview (now 1920x1080)
- ✓ Waveform too small (now 6x bigger)
- ✓ Audio controls scattered (now in strip)
- ✓ No visual transport feedback (now has buttons)

### Currently Open
- None reported

---

## Future Considerations

**Potential enhancements** (not committed):
- Section headers for control groups
- Draggable IN/OUT handles on waveform
- Preset system for modulators
- Visual beat grid overlay
- Loop button for audio

**Not planned:**
- Direct Ableton integration (use TouchDesigner)
- Real-time MIDI control (standalone philosophy)
- Multiple audio tracks
- Server-side processing

---

## Getting Help

### Documentation
- **Complete User Guide:** `UserGuide_All.md`
- **Technical Docs:** `C_Developer_Documentation.md`
- **Version History:** `B_Complete_Changelog.md`
- **FFmpeg Guide:** `UserGuide_FFmpeg_Video_Guide.md`

### Resources
- **WebGPU:** https://gpuweb.github.io/gpuweb/
- **Web Audio API:** https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- **WGSL:** https://gpuweb.github.io/gpuweb/wgsl/

---

## What's Next?

### Recommended Reading Order

**New Users:**
1. This file (Release Summary) ✓
2. `UserGuide_All.md` - Learn the interface
3. `UserGuide_FFmpeg_Video_Guide.md` - Post-processing
4. Start creating!

**Upgrading from V2.0.2:**
1. This file (Release Summary) ✓
2. `B_Complete_Changelog.md` - See what changed
3. `UserGuide_All.md` - Updated workflows
4. Explore the new UI!

**Developers:**
1. This file (Release Summary) ✓
2. `C_Developer_Documentation.md` - Architecture
3. `B_Complete_Changelog.md` - API changes
4. Extend the tool!

---

## Conclusion

V2_1 is about **better workflows**. The tab system, bigger canvas, and reorganized controls make the creative process smoother and more enjoyable. Everything you loved about V2.0.2 is still here - now with a much better interface.

**Key Improvements:**
- 🎯 Tab system for focused workflows
- 📐 6x bigger canvas (1920x1080 HD preview)
- 🎵 6x bigger waveform (1920px wide)
- 🎮 OMS Pink transport controls
- 🎨 Cleaner 5-row control panel

**Try it now:** Open `OMS_Dojo_V2_1.html` and experience the difference!

---

**Project Creator:** Wes Smith (OneManShYo)  
**Status:** Production Release  
**Version:** V2_1  
**Published:** November 2025

---

*For complete details, see the full documentation package included with this release.*
