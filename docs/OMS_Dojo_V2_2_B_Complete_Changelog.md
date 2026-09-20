# ONEMANSHYO Dojo - Complete Changelog

**Version:** V2_2  
**Published:** November 2025  
**Previous Version:** V2_1

---

## Table of Contents

- [V2_2 - Current Release](#v220---current-release-november-2025)
- [V2.1.0 - Previous Release](#v210---previous-release-november-2025)
- [Migration Guides](#migration-guides)
- [Statistics](#statistics)
- [Known Issues](#known-issues)
- [Future Roadmap](#future-roadmap)

---

## V2_2 - CURRENT RELEASE (November 2025)

**Status:** Production Release  
**Lines:** ~3,185  
**Size:** ~150 KB  
**Focus:** Audio BPM Detection, UI Clarity, Fine-Tuning Workflow

### Major Features

**🎵 Automatic BPM Detection:**
- Analyzes uploaded audio buffer on load
- Onset detection algorithm (RMS-based energy analysis)
- Detects BPM in range 60-180 (0.5 step resolution)
- User can override detected tempo by typing
- Uses autocorrelation scoring for accuracy
- Displays detected tempo with 1 decimal precision
- "BPM" field updates automatically

**⏱️ Nudge Field for Fine-Tuning:**
- New field for millisecond-precision loop timing
- Shifts entire loop (both Loop In and Loop Out) by time offset
- Range: -5000ms to +5000ms
- Arrow keys: ±1ms (Up/Down), ±4ms (Left/Right)
- Applied to all audio position jumps (load, stop, play, loop, compile, render)
- Perfect for video render sync adjustments

**🔘 UI Clarity Improvements:**
- Renamed "Audio BPM" → "BPM"
- Renamed "IN" → "Loop In"
- Renamed "OUT" → "Loop Out"
- Updated tooltips for clarity

**🔒 Simplified Arrow Key Behavior:**
- Arrow keys ALWAYS move both Loop In/Out together
- Maintains loop length (slides window)
- Grid button disabled (reserved for future visual grid feature)
- Removed Grid/Free mode toggle complexity

**🔁 Loop Mode Toggle:**
- Loop button added to transport controls
- Active (default): Audio repeats Loop In → Loop Out
- Inactive: Audio plays continuously
- Visual state indicator (standard inactive styling)

### Development Iterations

#### V2_2-A (BPM Detection)
- Added automatic BPM detection on audio load
- Onset detection algorithm with autocorrelation
- "Audio BPM" field with auto-update
- Detects 60-180 BPM range (0.5 resolution)
- Uses `audioTempo` variable (separate from shader tempo)
- ~3,100 lines (estimated)

#### V2_2-D (Grid/Free Mode)
- Added Grid/Free mode toggle button
- Grid mode: Brackets move together
- Free mode: Brackets adjust independently
- Arrow key behavior respects mode
- Foundation for simplified workflow
- ~3,120 lines (estimated)

#### V2_2-F (Loop & Styling)
- Added Loop button to transport controls
- Toggle loop IN→OUT vs continuous playback
- Standard inactive button styling introduced
- Applied to both Grid and Loop buttons
- Consistent visual language
- ~3,140 lines (estimated)

#### V2_2-G (Load Behavior)
- Audio jumps to Loop In beat on load (not 0)
- Better workflow: start at loop position immediately
- Uses audio BPM for positioning
- Reduces manual navigation
- ~3,155 lines (estimated)

#### V2_2-H (UI Polish & Nudge)
- Renamed controls for clarity (BPM, Loop In, Loop Out)
- Grid button disabled (reserved for future)
- Arrow keys always shift both brackets
- **NEW:** Nudge field for millisecond adjustments
- Nudge applied to all audio jumps
- Simplified workflow (removed Grid/Free complexity)
- ~3,185 lines

#### V2_2 (Published Release)
- All V2_2-H features finalized
- Complete BPM detection system
- Nudge field for fine-tuning
- Cleaner UI labels
- Simplified controls
- Production-ready
- ~3,185 lines

### Technical Changes

**Variables Added:**
```javascript
let nudgeOffset = 0; // Millisecond offset (converted to seconds)
let loopEnabled = true; // Loop mode toggle
```

**Variables Removed:**
```javascript
// let gridMode = true; // No longer used in V2_2-H
```

**Functions Added:**
- `detectBPM(audioBuffer)` - Automatic tempo detection
- `toggleLoop()` - Loop mode toggle
- Nudge field event listeners (input, blur, keydown)

**Functions Modified:**
- `loadAudioFile()` - BPM detection + jump to Loop In + apply nudge
- `stopAudio()` - Apply nudge offset
- `toggleAudioPlayback()` - Apply nudge when jumping to IN
- `compileShader()` - Apply nudge on reset
- `render()` - Apply nudge when looping
- `startRender()` - Apply nudge on video render start
- `handleArrowKeys()` - Simplified (always move both brackets)

**Functions Disabled:**
- `toggleGridFree()` - Grid button no longer functional

**UI Elements Added:**
```html
<!-- Nudge field -->
<label for="nudgeField">Nudge</label>
<input type="number" id="nudgeField" value="0" style="width: 70px;">
<span>ms</span>

<!-- Loop button -->
<button id="loopBtn" onclick="toggleLoop()">Loop</button>
```

### Audio Controls Strip Layout

**Before (V2.1.0):**
```
[Play ▶] [Stop ■] | Audio BPM: [120.0] | IN: [64] OUT: [128] [Grid]
```

**After (V2_2):**
```
[Play ▶] [Stop ■] [Loop] | BPM: [120.0] | Loop In: [64] Loop Out: [128] Nudge: [0 ms] [Grid]
                                                                                      ↑
                                                                                  (disabled)
```

### Breaking Changes

**None!** All V2.1.0 features preserved.

### Migration Notes (V2.1.0 → V2_2)

**For Users:**
1. **BPM auto-detected** - No need to manually enter tempo
2. **Labels renamed** - "BPM", "Loop In", "Loop Out" are clearer
3. **Loop button** - Toggle between loop mode and continuous play
4. **Nudge field** - Fine-tune timing by milliseconds
5. **Arrow keys** - Always shift both brackets together now
6. **Grid button** - Disabled (reserved for future visual grid)

**For Developers:**
- `gridMode` variable removed
- `nudgeOffset` variable added
- `loopEnabled` variable added
- Nudge offset applied to all audio position assignments
- Arrow key handler simplified (no Grid/Free logic)
- BPM detection runs automatically on load

---

## V2.1.0 - PREVIOUS RELEASE (November 2025)

**Status:** Superseded by V2_2  
**Lines:** ~2,680  
**Size:** ~115 KB  
**Focus:** Tab System & Canvas Expansion

### Key Features

**Tab System:**
- Controls tab / Code tab split-view
- Bigger shader editor (2.6x more vertical space)
- Cleaner interface organization
- Tab switching via JavaScript

**Canvas Expansion:**
- Canvas: 320x180 → 1920x1080 (6x bigger)
- Waveform: 320x180 → 1920x180 (6x wider)
- HD preview matches 1080p export resolution
- Better visual feedback for shader work

**Layout Reorganization:**
- Waveform moved to canvas area (was in left panel)
- Audio controls moved to canvas strip (was scattered)
- Transport buttons added (Play ▶ / Stop ■) in OMS Pink
- Left panel now purely shader controls (5 rows)

**Shader Management:**
- Compile button moved to Code tab
- Copy Code button added
- Export .txt button added (includes OMS version)
- Better workflow: write → compile → copy/export

**Drag & Drop:**
- Audio files: Drag onto waveform area
- Shader files: Drag onto code editor (.txt, .wgsl)
- Visual feedback (cyan border highlight)

**Navigation Enhancements:**
- Arrow keys shift both IN/OUT together (maintains loop length)
- Clickable IN/OUT labels (jump to start/end)
- Left/Right arrows jump audio to new position

### Development Iterations (V2.1.0)

- V2.1.0-A: Right panel restructure
- V2.1.0-B: Waveform full-width rendering
- V2.1.0-C: Tab system implementation
- V2.1.0-D: Status bar to left panel
- V2.1.0-E: Transport controls (OMS Pink)
- V2.1.0-F: Filename field fix

All features from V2.1.0 are preserved in V2_2!

---

## Migration Guides

### V2.1.0 → V2_2

**What Changed:**
- BPM field renamed ("Audio BPM" → "BPM")
- IN/OUT renamed ("Loop In"/"Loop Out")
- Loop button added
- Nudge field added
- Grid button disabled
- Arrow keys always move both brackets

**What Stayed The Same:**
- ✓ Tab system
- ✓ 1920x1080 canvas
- ✓ Waveform on canvas
- ✓ Transport controls
- ✓ Drag & drop
- ✓ Shader management buttons
- ✓ All keyboard shortcuts (Spacebar, arrows)
- ✓ Video rendering workflow
- ✓ Modulation system
- ✓ Frequency analysis

**Workflow Changes:**
1. Upload audio → BPM auto-detects (no manual entry needed)
2. Arrow keys always shift both brackets (simpler)
3. Use Nudge for fine-timing (new capability)
4. Toggle Loop button for playback mode (new)

---

## Statistics

### Version Progression

| Version | Lines | Size | Major Features |
|---------|-------|------|----------------|
| V1.0.5  | ~970  | ~40KB | Basic modulation |
| V2.0.0  | ~1,800 | ~75KB | Audio system |
| V2.0.1F | ~2,096 | ~88KB | Beat navigation |
| V2.0.2  | ~2,189 | ~93KB | UI refinement |
| V2.1.0  | ~2,680 | ~115KB | Tab system & canvas |
| **V2_2**  | **~3,185** | **~150KB** | **BPM detection & nudge** |

### Development Timeline

- **V1.0.5:** September 2025
- **V2.0.0:** October 2025 (MAJOR - Audio)
- **V2.0.1A-F:** October-November 2025 (6 iterations)
- **V2.0.2A-G:** November 2025 (7 iterations)
- **V2.1.0A-F:** November 2025 (6 iterations)
- **V2.2.0A-H:** November 2025 (5 iterations, D-H)
- **V2_2 Published:** November 2025

### Growth Metrics

- **Lines of Code:** +228% (970 → 3,185)
- **File Size:** +275% (40KB → 150KB)
- **Major Versions:** 2 (V1 → V2)
- **Minor Releases:** 3 (V2.0 → V2.1 → V2.2)
- **Patch Releases:** 2 (V2.0.0 → V2.0.1 → V2.0.2)
- **Total Iterations:** 29 (A-H across multiple versions)

---

## Known Issues

### Resolved in V2_2
- ✓ Manual BPM entry (now auto-detected)
- ✓ Confusing Grid/Free toggle (simplified)
- ✓ No fine-tuning for sync (Nudge field added)
- ✓ Generic field labels (now "Loop In"/"Loop Out")

### Still Open
- None reported

---

## Future Roadmap

### Potential Enhancements (Not Committed)
- Visual grid overlay (Grid button reserved for this)
- Draggable IN/OUT handles on waveform
- Multiple loop regions / cue points
- Preset system for modulators
- Expanded BPM detection range (40-240)
- Volume control slider
- Waveform zoom controls

### Not Planned
- Direct Ableton integration (use TouchDesigner for that)
- Real-time MIDI control
- Multiple audio tracks
- Server-side processing

---

## Performance Comparison

### Audio System Impact

**V2.1.0 (Pre-BPM Detection):**
- Load time: Fast
- Audio startup: Instant

**V2_2 (With BPM Detection):**
- Load time: +1-2 seconds for BPM analysis
- Audio startup: Instant (jumps to Loop In)
- CPU load: Minimal (analysis only on load)

**Recommendation:** V2_2's BPM detection adds minimal overhead and provides significant workflow improvement.

---

## Breaking Changes History

### V2_2
- None! (UI labels and simplified workflow only)

### V2.1.0
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
**Current Version:** V2_2  
**Status:** Production Release  
**Documentation:** Complete

---

*For detailed technical documentation, see C_Developer_Documentation.md*  
*For user workflows, see D_Release_Summary.md*  
*For production deployment, see E_Production_Release_Guide.md*
