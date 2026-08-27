# ONEMANSHYO Dojo - Complete Changelog

**Version:** V2_1  
**Status:** Production Release  
**Published:** November 2025  
**Lines of Code:** ~2,680  
**File Size:** ~115 KB

---

## Table of Contents

- [V2_1 - Current Release](#v210---current-release-november-2025)
- [V2.0.2 - Previous Release](#v202---previous-release-november-5-2025)
- [V2.0.1 Series](#v201-series-october-november-2025)
- [V2.0.0 - Major Release](#v200---major-release-october-2025)
- [V1.0.5 - Pre-Audio Baseline](#v105---pre-audio-baseline)
- [Migration Guide](#migration-guide)
- [Statistics](#statistics)

---

## V2_1 - CURRENT RELEASE (November 2025)

**Status:** Production Release  
**Lines:** ~2,680  
**Size:** ~115 KB  
**Focus:** Tab System & Canvas Expansion

### Major Features

**1. TAB SYSTEM**
- Split-view interface: Controls Tab / Code Tab
- Switch between parameter controls and shader editing
- Dedicated workspace for each workflow
- Clean separation of concerns

**2. 6X BIGGER CANVAS**
- Canvas expanded from 320x180 to 1920x1080
- Full HD real-time preview
- See exactly what your render will look like
- Better for detailed shader work

**3. WAVEFORM ON CANVAS**
- Moved from left panel to canvas area
- 6x larger display (easier to see and interact with)
- More prominent visual feedback
- Better integration with canvas workflow

**4. AUDIO CONTROLS STRIP**
- New dedicated control strip below waveform
- Transport controls: ▶ (Play/Pause) and ⏹ (Stop)
- IN/OUT beats always visible
- OMS Pink (#ff006e) styling for transport

**5. REFINED LEFT PANEL**
- Cleaner 5-row layout
- Removed audio row (moved to canvas)
- All shader parameters in one view
- Better workflow organization

### Development Iterations

**V2_1_F (Filename Field Fix)**
- Fixed filename input field (removed default "filename" text)
- Empty field shows only placeholder "Filename"
- Cleaner user experience
- Final polish iteration
- ~2,676 lines

**V2_1_E (Transport Controls)**
- Added Play button (▶/⏸) with OMS Pink styling
- Added Stop button (⏹) with OMS Pink styling
- Buttons in audio-controls-strip below waveform
- Pink glow effect when playing
- Hover effects and disabled states
- Keyboard shortcuts preserved (Spacebar, Arrows)
- ~2,675 lines

**V2_1_D (Audio Controls Strip)**
- Created audio-controls-strip element below waveform
- IN/OUT beats moved from left panel to canvas
- Always visible with audio controls
- Improved accessibility and workflow
- ~2,650 lines

**V2_1_C (Waveform on Canvas)**
- Moved waveform from left panel to right canvas area
- Waveform now 6x bigger (1920px vs 320px)
- Better visual feedback
- More space in left panel
- ~2,610 lines

**V2_1_B (Canvas Expansion)**
- Canvas size: 320x180 → 1920x1080 (6x bigger!)
- Full HD preview in real-time
- Better for detailed work
- Matches standard render resolution
- ~2,580 lines

**V2_1_A (Tab System)**
- Added Controls Tab / Code Tab split-view
- Tab switching functionality
- Dedicated workspace for each mode
- Cleaner interface organization
- ~2,550 lines

### What Changed

**UI Layout:**
- Left panel: Now only shader controls (5 rows)
- Right panel: Canvas + waveform + audio controls
- Tab system: Controls / Code switching

**Removed from Left Panel:**
- Audio upload button (moved to canvas interaction)
- Waveform display (moved to canvas)
- IN/OUT beats (moved to audio-controls-strip)

**Added to Canvas Area:**
- Full-size waveform display
- Audio controls strip
- Transport controls (▶ ⏹)
- IN/OUT beat fields

**Keyboard Shortcuts (Preserved):**
- Spacebar: Play/Pause
- Left Arrow: Previous beat
- Right Arrow: Next beat
- All functionality maintained

### Technical Changes

**Canvas:**
- Default size: 1920x1080 (was 320x180)
- Scales with orientation and resolution
- Better GPU utilization

**Waveform:**
- Size: 1920x180 (was 320x180)
- Same resolution, 6x wider
- Better visibility of audio structure

**Audio Controls:**
- New DOM elements for transport
- OMS Pink styling (#ff006e)
- State management for play/stop

**Tab System:**
- JavaScript tab switching
- Active tab tracking
- Content show/hide logic

### Breaking Changes

**None!**
- All V2.0.2 features preserved
- All keyboard shortcuts work
- All rendering functionality identical
- Only UI layout changed

### Migration from V2.0.2

**For Users:**
1. UI layout is different but workflow is same
2. Audio upload now on canvas (not left panel)
3. Use tabs to switch between Controls and Code
4. Transport buttons provide visual feedback
5. Everything else works identically

**For Developers:**
- No API changes
- All uniform structures unchanged
- All JavaScript functions unchanged
- CSS structure reorganized (tabs, layout)

---

## V2.0.2 - PREVIOUS RELEASE (November 5, 2025)

**Status:** Superseded by V2_1  
**Lines:** ~2,189  
**Size:** ~93 KB  
**Focus:** UI Clarity & Square Orientation

### Key Features

**7-Row Optimized Layout:**
- Row 1: Video orientation (Landscape/Portrait/Square)
- Row 2: Shader tempo + loop length
- Rows 3-5: Modulator controls (A, B, C)
- Row 6: Audio upload/waveform + IN/OUT beats
- Row 7: Compile + Resolution + FPS + Filename + Render

**Square Orientation Support:**
- 1:1 aspect ratio for Instagram Feed
- Scales with resolution (1080x1080, 1440x1440, 2160x2160)
- Proper filename generation with '_S' suffix

**Higher Resolution Support:**
- 1440p (2K) option
- 2160p (4K) option
- Better quality for large displays

**Professional UI Polish:**
- Consistent tooltip naming (Shader/Video prefixes)
- Capitalized tooltip text throughout
- Logical workflow ordering (C → Resolution → FPS → Filename → R)

### Development Iterations (A-G)

Detailed in V2.0.2 documentation package.

---

## V2.0.1 Series (October-November 2025)

**Focus:** Beat-Based Navigation & DAW-Style UX

### V2.0.1G (Video Render Bug Fix)
- Fixed video render freeze bug
- Canvas.captureStream() without FPS parameter
- Captures every rendered frame reliably
- ~2,107 lines

### V2.0.1F (Default Loop Position)
- Default IN beat: 0 → 64
- Starts in "meat" of song (past intro)
- More practical for music visualization
- ~2,096 lines

### V2.0.1E (Default Orientation)
- Default: Landscape → Portrait
- Better for social media (Instagram, TikTok, YouTube Shorts)
- ~2,089 lines

### V2.0.1D (DAW-Style Status Bar)
- Status bar shows bar.beat format
- Industry-standard notation (4.2 bars = 4 bars + 2 beats)
- Clarified IN/OUT tooltips
- ~2,085 lines

### V2.0.1C (Coupled Arrow Key Navigation)
- Arrow keys on IN/OUT fields move both values
- Maintains loop length (DAW-style)
- ~1,962 lines

### V2.0.1B (Validation & Status Bar)
- Fixed OUT field validation
- Status bar improvements
- ~1,943 lines

### V2.0.1A (Beat-Based Navigation)
- Changed from bars to beats
- Arrow keys jump 1 beat (was 4 beats)
- Duration indicator
- ~1,897 lines

---

## V2.0.0 - MAJOR RELEASE (October 2025)

**Status:** Foundation for all V2.x releases  
**Lines:** ~1,800  
**Size:** ~75 KB  
**Focus:** Complete Audio Integration

### Major Features

**Audio System:**
- File upload and decoding
- Real-time FFT frequency analysis
- Bass/Mid/Treble band splitting
- Audio-reactive modulation
- Gain controls (0-5x multipliers)
- Web Audio API integration

**Waveform Visualization:**
- Real waveform analysis (RMS-based)
- 200-bar resolution
- Interactive scrubbing
- Play/Pause/Stop controls
- Progress indicator
- Time display (MM:SS format)

**Visual Indicators:**
- IN/OUT beat indicators (cyan)
- Playhead position (magenta)
- Played portion overlay
- Time display

**Audio Export:**
- Video rendering with embedded audio
- Synchronized audio/video capture
- MediaRecorder API integration

**Modulation System:**
- 3 independent modulators (A, B, C)
- Audio multiplication per modulator
- Mod A × Bass, Mod B × Mid, Mod C × Treble

### Technical Architecture

**Frequency Analysis:**
- AnalyserNode (256 FFT size)
- Bass: 0-10% of spectrum
- Mid: 10-50% of spectrum
- Treble: 50-100% of spectrum
- Exponential smoothing (0.7 factor)

**Audio Routing:**
- AudioContext → AnalyserNode → Destination
- MediaElementSource for playback
- MediaStreamDestination for recording

**Performance:**
- ~16ms budget @ 60fps
- Static waveform cached
- Only overlays redraw each frame
- FFT analysis: ~1ms per frame

---

## V1.0.5 - Pre-Audio Baseline

**Status:** Deprecated (superseded by V2.0.0)  
**Lines:** ~970  
**Size:** ~40 KB

### Features

- Basic modulation system
- Tempo-synced generators
- Multiple waveform types
- WebGPU rendering
- Video export (no audio)

---

## Migration Guide

### V2.0.2 → V2_1

**What's New:**
- Tab system (Controls / Code)
- 6x bigger canvas (1920x1080)
- Waveform on canvas
- Audio controls strip
- Transport buttons (OMS Pink)

**What Changed:**
- Left panel: Only shader controls (5 rows)
- Right panel: Canvas + waveform + audio controls
- Audio upload interaction on canvas

**What Stayed the Same:**
- All keyboard shortcuts
- All rendering functionality
- All shader parameters
- All modulators and audio reactivity
- All file formats and output

**Recommended Actions:**
1. Familiarize yourself with tab system
2. Enjoy bigger canvas preview
3. Use transport buttons for quick playback
4. All workflows function identically

### V2.0.0 → V2_1

**Major Changes:**
- Everything from V2.0.2 plus V2_1 features
- UI completely redesigned
- Much better preview workflow

**Workflow Changes:**
- Use tabs to switch between controls and code
- Waveform and audio controls on canvas
- Bigger preview means better visual feedback

---

## Statistics

### Version Progression

| Version | Lines | Size | Major Features |
|---------|-------|------|----------------|
| V1.0.5  | ~970  | ~40KB | Basic modulation |
| V2.0.0  | ~1,800 | ~75KB | Audio system |
| V2.0.1F | ~2,096 | ~88KB | Beat navigation |
| V2.0.2  | ~2,189 | ~93KB | UI refinement |
| V2_1  | ~2,680 | ~115KB | Tab system & canvas |

### Development Timeline

- **V1.0.5:** September 2025
- **V2.0.0:** October 2025 (MAJOR - Audio)
- **V2.0.1A-F:** October-November 2025 (6 iterations)
- **V2.0.2A-G:** November 2025 (7 iterations)
- **V2.0.2 Published:** November 5, 2025
- **V2_1A-F:** November 2025 (6 iterations)
- **V2_1 Published:** November 2025

### Growth Metrics

- **Lines of Code:** +175% (970 → 2,680)
- **File Size:** +187% (40KB → 115KB)
- **Major Versions:** 2 (V1 → V2)
- **Minor Releases:** 2 (V2.0 → V2.1)
- **Patch Releases:** 3 (V2.0.0 → V2.0.1 → V2.0.2)
- **Development Iterations:** 19 total (A-G across multiple versions)

---

## Known Issues

### Resolved in V2_1
- ✓ Small canvas preview (now 1920x1080)
- ✓ Waveform too small (now 6x bigger)
- ✓ Audio controls scattered (now in strip)
- ✓ No visual transport feedback (now has buttons)

### Resolved in V2.0.2
- ✓ Square orientation filename generation
- ✓ Blank tooltip bugs
- ✓ Inconsistent capitalization

### Still Open
- None reported

---

## Future Roadmap

### Potential Enhancements (Not Committed)
- Section headers for control groups
- Draggable IN/OUT handles on waveform
- Multiple render regions (cue points)
- Preset system for modulators
- Visual beat grid overlay
- Loop button for audio playback
- Volume control slider

### Not Planned
- Direct Ableton integration (use TouchDesigner)
- Real-time MIDI control
- Multiple audio tracks
- Server-side processing

---

## Performance Comparison

### Canvas Size Impact

**V2.0.2 (320x180):**
- Render time: Fast
- Preview quality: Limited
- GPU load: Light

**V2_1 (1920x1080):**
- Render time: Same (renders at export resolution anyway)
- Preview quality: Excellent (1:1 with 1080p export)
- GPU load: Moderate (well within modern GPU capabilities)

**Recommendation:** V2_1's bigger canvas has negligible performance impact on modern hardware while providing significantly better preview quality.

---

## Breaking Changes History

### V2_1
- None! (UI layout only)

### V2.0.2
- None (all V2.0.0 features preserved)

### V2.0.1F
- Default IN beat changed: 0 → 64
- Default OUT beat unchanged: 128

### V2.0.1E
- Default orientation: Landscape → Portrait

### V2.0.1A
- IN/OUT changed from bars to beats
- Arrow key behavior (1 beat vs 4 beats)

### V2.0.0
- Complete rewrite from V1.0.5
- Audio system added (~830 lines)
- All V1.0.5 features preserved

---

**Last Updated:** November 2025  
**Current Version:** V2_1  
**Status:** Production Release  
**Documentation:** Complete

---

*For detailed technical documentation, see C_Developer_Documentation.md*  
*For user workflows, see UserGuide_All.md*  
*For migration guides, see D_Release_Summary.md*
