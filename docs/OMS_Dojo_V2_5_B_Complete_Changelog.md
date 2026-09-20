# ONEMANSHYO Dojo - Version History

## V2_5 (PRODUCTION RELEASE) ← CURRENT VERSION - November 2025

**24 Development Iterations: A through X**

Complete detailed notes for all 24 iterations available in separate files:
- OMS_Dojo_V2_5_A_RenderSync_Notes.txt through
- OMS_Dojo_V2_5_X_LoopButtonCyan_Notes.txt

### Major Feature Additions

**Render Duration Controls (Iterations A-J)**
- Added Render Duration field for no-audio mode rendering
- MM:SS.mmm time format (matches Audio Start field)
- Left-to-right parsing (5 = 5 minutes, not 5 seconds)
- Arrow key controls: Up/Down ±0.1s, Left/Right ±1.0s
- Default: 00:30.000 (30 seconds)

**File Optimization (Iterations M, N, S)**
- Removed 505KB embedded changelog (Iteration M)
- Fixed 100KB corrupted UTF-8 separators (Iteration N)
- Cleaned 1.2MB corrupted comment blocks (Iteration S)
- Final size: 142KB (down from 1.4MB - 90% reduction!)

**Terminology Updates (Iteration U)**
- Changed "BPM" to "Audio Tempo" throughout UI
- Matches industry standard (Ableton Live, Logic Pro, etc.)
- Clear separation: "Shader Tempo" vs "Audio Tempo"

**UX Improvements (Iterations Q, R, X)**
- Render progress moved to status bar (canvas stays visible)
- Simplified filename format (removed duration/beats metadata)
- Loop button visual feedback (cyan when active)

### Critical Bug Fixes

**Audio/Visual Sync (Iterations A, O, P)**
- Fixed render visual timing to sync with Loop In/Out positions
- Fixed render using Audio Tempo instead of Shader Tempo
- Fixed bar.beat format conversion (was using parseInt incorrectly)

**Render Duration Accuracy (Iterations H, T)**
- Removed Math.ceil rounding that caused overshoot
- Removed +1 second buffer causing extra length
- Renders now stop precisely at Loop Out position

**Loop Playback (Iterations T, V, W)**
- Added 0.15 beat threshold to prevent Loop Out overshoot
- Fixed loop-back calculation using correct Audio Tempo
- Perfect tight looping within In/Out brackets

### Technical Changes

**Shader Tempo / Audio Tempo Separation**
- Independent systems as designed
- Shader Tempo: visual modulator timing
- Audio Tempo: beat positioning and loop calculations
- Integration point: Loop In/Out brackets define render range

**Time Format Consistency**
- Render Duration now uses MM:SS.mmm (matches Audio Start)
- Uses shared timeStringToSeconds() and secondsToTimeString() functions
- Consistent parsing and display across all time fields

---

## V2.4.0 (November 2025)

Manual beatgrid system implementation with visual feedback.

**Major Features:**
- Song Start/End fields for first downbeat alignment
- Visual beat grid overlay on waveform
- Manual BPM entry workflow
- Grid verification and alignment tools
- Fixed loop playback functionality

**Development Iterations:** 12 iterations (A through L)
*Refer to V2.4.0 deliverables package for detailed iteration notes*

---

## V2.3.0 (November 2025)

Critical bug fixes and UI improvements.

**Major Features:**
- Shader Tempo / Audio BPM field separation
- Manual BPM entry with arrow key navigation (±0.01, ±1.0)
- Fine precision Loop Out control (1/32 note resolution)
- UI improvements: Fixed button layouts, removed Nudge field
- Removed automatic BPM detection

**Development Iterations:** 10 iterations (A through J)
*Refer to V2.3.0 deliverables package for detailed iteration notes*

---

## Version Numbering Convention

**Production Releases:**
- V2_5 (no letter = final release)
- V2.4.0, V2.3.0, V2.2.0, etc.

**Development Iterations:**
- V2.5.0_A through V2.5.0_X (letters = work in progress)
- Each letter represents a complete work session

---

## Development Approach

Starting with V2.3.0, all iterations are:
1. Developed incrementally (one feature/fix per iteration)
2. Documented in individual notes files
3. Tested independently to prevent system crashes
4. Merged into production release when cycle complete

This incremental approach prevents token limit issues and maintains system stability throughout development.

---

**Last Updated:** November 2025  
**Current Version:** V2_5  
**File Format:** Single HTML file, no external dependencies
