# OMS SHADER BAKER V3.0.0 - COMPLETE CHANGELOG

**Release Date:** November 2025  
**Development Period:** November 2025  
**Iterations:** A through P (16 iterations)

---

## Overview

Shader Baker V3.0.0 represents the integration of ShaderBakerLive's performance interface as the 5th "LIVE" tab within the main Shader Baker application. This major architectural shift eliminates the need for separate tools and BroadcastChannel communication, creating a unified performance environment that shares the existing audio engine, LIBRARY folder access, and .oms file system.

**Total Development:** 16 iterations  
**Major Achievement:** Complete LIVE tab UI shell integrated  
**File Size:** ~330KB (from V2.9.0's 272KB)  
**Architecture:** Single unified app with 5 tabs

---

## Development Strategy

### The Merge
ShaderBakerLive V1.0.0 (standalone tool) → Shader Baker V3.0.0 LIVE tab (integrated)

**What Changed:**
- Separate HTML file → 5th tab in main app
- BroadcastChannel communication → Direct shared access
- Instance discovery → Direct LIBRARY container loading
- Standalone tool → Integrated performance interface

**What Stayed:**
- Performance grid layout (4 tracks × 8 scenes)
- Hardware mixer controls (rotary knobs, meters)
- Fixed-width design philosophy
- Professional aesthetics

---

## Iteration Summary

### PHASE 1: Foundation (A-B)

**Iteration A: LIVE Tab Shell**
- Added 5th tab button to tab bar
- Created LIVE tab content area
- Reduced tab padding to fit 5 tabs (8px → 16px)
- Added complete LIVE CSS section (~350 lines)
- HTML structure: performance grid + mixer + waveforms
- JavaScript initialization functions
- File size: 7,401 lines (~320KB)

**What Works:**
- All 5 tabs switch properly
- LIVE tab displays performance grid
- Scene triggers clickable (console feedback)
- Rotary knobs functional (drag interaction)
- Mute/Solo buttons toggle states
- Waveforms display placeholder

**Status:** UI shell functional, no integration yet

---

**Iteration B: Layout Architecture Correction**
- LEFT panel: Shows Library cards when LIVE active
- RIGHT panel: Shows LIVE mixer (replaces canvas/audio controls)
- Enables workflow: browse library + mix simultaneously
- Tab bar gap reduced (10px → 5px)
- switchTab() updated for proper element visibility
- liveMixerContainer positioned in preview-panel

**What Works:**
- LIVE tab shows Library on left, mixer on right
- Switching tabs properly hides/shows elements
- Canvas/audio controls hide when LIVE active
- Performance layout fills right panel

**Status:** Proper layout architecture established

---

### PHASE 2: Layout Refinement (C-H)

**Iteration C: Waveforms Fixed**
- Waveforms height: 120px → 180px
- Each track gets ~42px (was ~28px)
- Right panel locked: 1200px fixed width
- All 4 waveform tracks clearly visible
- No responsive resizing
- File size: 7,445 lines

**What Works:**
- All 4 waveform tracks display properly
- Right panel 1200px (no responsive issues)

---

**Iteration D: Width Optimization for MacBook Pro**
- Right panel: 1200px → 1070px
- Total width: 1470px (400px left + 1070px right)
- Fits 14" MacBook Pro perfectly
- Grid adjusted for 5 columns (4 tracks + master)
- Each mixer column: 196px (980px ÷ 5)
- File size: 7,445 lines

**What Works:**
- Fits 14" MacBook Pro without horizontal scrolling
- Space prepared for 5 mixer columns

---

**Iteration E: Grid Column Sizing**
- Fixed pixel widths instead of flexible 1fr units
- grid-headers: 90px 245px 245px 245px 245px
- scenes-grid: matching column widths
- mixer-section: matching column widths
- Total: 90px + (4 × 245px) = 1070px ✓
- All 4 tracks now visible
- File size: 7,445 lines

**What Works:**
- All 4 track columns display properly
- Grid headers, scenes, mixer aligned
- Fixed widths prevent cut-off

---

**Iteration F: Master Column + Waveforms Always Visible**
- Added 5th column (Master) to grid
- Column widths: 245px → 196px (5 columns)
- Total: 90px + (5 × 196px) = 1070px ✓
- Master header, mixer controls, grid cells added
- Waveforms moved INSIDE performance-view (critical fix)
- Waveforms always visible (flex-shrink: 0)
- Clip grid scrolls, headers/mixer/waveforms stay fixed
- File size: 7,552 lines

**What Works:**
- All 5 columns visible (Track 1-4 + Master)
- Master mixer controls present
- Waveforms always visible at bottom
- Clip grid scrolls independently

**Vertical Layout:**
```
performance-view (flex column):
├─ grid-headers (60px, fixed)
├─ scenes-scroll-container (flex: 1, scrolls)
├─ mixer-section (140px, fixed)
└─ waveforms-section (220px, fixed) ← ALWAYS VISIBLE
```

---

**Iteration G: Centered Layout**
- App centered on screen (max-width: 1470px, margin: 0 auto)
- Waveforms height: 180px → 220px (~52px per track)
- Clip grid auto-adjusts (flex: 1)
- Better visual balance on larger monitors
- Professional centered appearance
- File size: 7,554 lines

**What Works:**
- App centers on all screen sizes
- All 4 waveforms fully visible with proper height

---

**Iteration H: Wrapper Fix**
- Added app-wrapper div wrapping entire app
- Header, container, status bar all centered together
- max-width: 1470px on wrapper (removed from container)
- Unified 1470px width block
- Professional appearance
- File size: 7,562 lines

**What Works:**
- Entire app centered as one unit
- Header/status match app width
- Clean, cohesive layout

---

### PHASE 3: Audio Controls (I-N)

**Iteration I: Audio Labels Row**
- Added dedicated label row above audio controls strip
- Labels separated from data fields
- 20px label row + 40px controls strip
- Less crowded, cleaner layout
- Clickable labels preserved
- File size: 7,624 lines

**What Works:**
- Labels in dedicated row above inputs
- Clean separation improves readability

---

**Iteration K: Bordered Audio Cells** [J skipped]
- Redesigned audio controls: 5 bordered cells
- Labels INSIDE each cell (above inputs)
- Height: 40px → 60px (accommodates labels + inputs)
- Cell borders: 1px solid #333
- Transport buttons reduced (28px → 24px height)
- Upload/Remove button swap logic
- File size: ~7,600 lines

**5 Cell Structure:**
1. Transport Controls (100px fixed) - Play, Stop, Loop
2. Audio Start/End (flex: 1, min 200px) - MM:SS.mmm inputs
3. Audio Tempo/Pitch (flex: 1, min 170px) - Decimal inputs
4. Loop In/Out (flex: 1, min 140px) - Bar.Beat inputs
5. Actions (140px fixed) - Grid + Upload/Remove buttons

**What Works:**
- Labels directly above controls (clear association)
- Bordered cells create functional groups
- Professional organization

---

**Iteration L: Horizontal Transport Buttons**
- Transport buttons: vertical stack → horizontal row
- Cell 1 width: 100px → 250px
- Input cells reduced 30%:
  - Cell 2: 200px → 140px
  - Cell 3: 170px → 120px
  - Cell 4: 140px → 100px
- Saved space (150px) redistributed to transport
- 3 buttons fit horizontally (~70px each)
- File size: ~7,600 lines

**What Works:**
- All 3 transport buttons in horizontal row
- Better space utilization
- Input fields still comfortable

---

**Iteration M: Transport Label**
- Added "Audio Transport" label above buttons in Cell 1
- All 5 cells now have labels (except Cell 5 - actions)
- Visual consistency across cells
- Professional labeling
- File size: ~7,653 lines

**What Works:**
- All cells properly labeled
- Consistent visual hierarchy

---

**Iteration N: App Side Borders**
- Added 5px borders left/right to app-wrapper
- Color: #555 (light grey)
- Creates "physical device" frame
- Separates app from browser black background
- No top/bottom borders (seamless vertical)
- File size: ~7,656 lines

**What Works:**
- Side borders visible in header/status
- Physical device aesthetic
- Clear app boundaries

---

### PHASE 4: Final Polish (O-P)

**Iteration O: Audio File Label + UX Polish**
- Added "Audio File" label to drag/drop area
- Various UX improvements
- Preparing for functionality integration
- File size: ~7,665 lines

**What Works:**
- Audio section clearly labeled
- Professional appearance

---

**Iteration P: Fixed Side Borders (Final)**
- Fixed borders not showing full height
- Changed CSS borders → inset box-shadow
- box-shadow: inset 5px 0 0 0 #555, inset -5px 0 0 0 #555
- Borders now render OVER child backgrounds
- Continuous grey lines from top to bottom
- File size: ~7,671 lines

**Technical Fix:**
- CSS border covered by child backgrounds
- Inset box-shadow renders above children
- Same visual result, reliable rendering

**What Works:**
- Borders visible full height continuously
- Physical device frame complete

---

## Technical Achievements (V3.0.0)

### Architecture
- **Single HTML file** (~330KB, up from 272KB)
- **5 tabs total:** Controls, Presets, Library, Code, LIVE
- **No external dependencies**
- **Unified app architecture**

### LIVE Tab Features (UI Shell)
- Performance grid (4 tracks × 8 scenes = 32 cells)
- Fixed track headers (always visible)
- Scrollable scene rows
- Fixed mixer section (140px, always accessible)
- 5 mixer columns (Track 1-4 + Master)
- Rotary volume knobs (canvas-based)
- Mute/Solo buttons per track
- Vertical L/R stereo meters (placeholders)
- Waveform section (220px, 4 tracks, always visible)
- Fixed 1070px width (optimized for 14" MacBook Pro)
- App centered (1470px total with borders)
- Side borders (5px #555, inset box-shadow)

### Audio Controls Refinement
- 5 bordered cells with labels inside
- Horizontal transport buttons
- Optimized spacing (30% input width reduction)
- Professional organization
- Responsive flex layout

### Layout System
- App-wrapper: 1470px centered
- Left panel: 400px (tabs + Library)
- Right panel: 1070px (canvas/LIVE mixer)
- Side borders: 5px inset box-shadow
- Total width: 1480px (app + borders)

---

## What Works in V3.0.0

✅ **All V2.9.0 Features Preserved**
- LIBRARY-first architecture
- .oms file system
- Auto-save workflow
- Keyboard-first navigation
- Active container indicators
- Control States
- Preset system
- WebGPU shader rendering
- FFT audio analysis
- Video export

✅ **LIVE Tab UI Shell**
- 5th tab displays properly
- Performance grid layout (4×8 cells)
- 5 mixer columns (Track 1-4 + Master)
- Rotary knobs (visual interaction)
- Mute/Solo buttons (visual state)
- Waveforms section (placeholder)
- Fixed layout (1070px right panel)
- Centered app (1470px total)
- Side borders (full height)

✅ **Integration Points**
- LIVE tab shares audio engine (code present)
- LIVE tab shares LIBRARY folder access
- LIVE tab shares .oms file system
- Tab switching works properly
- No feature conflicts

---

## What Doesn't Work Yet

❌ **LIVE Tab Functionality (Planned V3.1.0+)**
- No container → grid assignment
- No audio playback
- No shader rendering in cells
- No scene launching
- Meters are placeholders (no animation)
- Controls have no audio effect
- Waveforms are placeholders

❌ **Advanced Features (Planned V3.2.0+)**
- No MIDI control integration
- No Ableton Link sync
- No visual compositing
- No multi-page support

---

## Preserved from V2.9.0

**All features work identically:**
- Controls tab (frequency mapping, modulators, render)
- Presets tab (5 library + user presets)
- Library tab (container management, keyboard navigation)
- Code tab (WGSL shader editor)
- LIBRARY folder system
- .oms file format
- Audio loading from LIBRARY
- Control States as .oms files
- Dirty state tracking
- Active container indicators
- Inline editable names

**Nothing removed, everything enhanced.**

---

## File Size Evolution

**V2.7.0:** 215KB (base)  
**V2.8.0:** 266KB (+BroadcastChannel)  
**V2.9.0:** 272KB (+LIBRARY-first)  
**V3.0.0:** ~330KB (+LIVE tab UI)  

**Growth:** +115KB for complete LIVE tab integration

---

## Browser Compatibility

**Required:**
- Chrome 88+ or Safari 15.2+ (WebGPU + File System Access API)
- CSS Grid, Flexbox, aspect-ratio
- Canvas 2D context
- ES6 JavaScript

**Tested:**
- Chrome 88+ ✓
- Safari 15.2+ ✓
- Edge 88+ ✓
- Firefox: Partial (WebGPU only, no File System Access API)

---

## Detailed Iteration Notes

Complete technical notes for each iteration available in separate files:
- OMS_ShaderBaker_V3_0_0_A_LiveTabShell_Notes.txt
- OMS_ShaderBaker_V3_0_0_B_LiveLayout_Notes.txt
- OMS_ShaderBaker_V3_0_0_C_WaveformsFixed_Notes.txt
- OMS_ShaderBaker_V3_0_0_D_WidthFix_Notes.txt
- OMS_ShaderBaker_V3_0_0_E_GridColumns_Notes.txt
- OMS_ShaderBaker_V3_0_0_F_MasterAndWaveforms_Notes.txt
- OMS_ShaderBaker_V3_0_0_G_Centered_Notes.txt
- OMS_ShaderBaker_V3_0_0_H_WrapperFix_Notes.txt
- OMS_ShaderBaker_V3_0_0_I_AudioLabels_Notes.txt
- OMS_ShaderBaker_V3_0_0_K_AudioControlsCells_Notes.txt [J skipped]
- OMS_ShaderBaker_V3_0_0_L_HorizontalTransport_Notes.txt
- OMS_ShaderBaker_V3_0_0_M_TransportLabel_Notes.txt
- OMS_ShaderBaker_V3_0_0_N_AppBorder_Notes.txt
- OMS_ShaderBaker_V3_0_0_O_AudioFileLabel_Notes.txt
- OMS_ShaderBaker_V3_0_0_P_FixSideBorders_Notes.txt

---

## Next Version (V3.1.0+)

**Functionality Integration:**
- Container → grid cell assignment
- Load .oms files from LIBRARY
- Audio playback per track
- Shader rendering per track (4 simultaneous)
- Real-time meters with FFT data
- Scene launching with crossfades
- Volume/mute/solo audio effects

**Future (V3.2.0+):**
- MIDI control (Midifighter Twister)
- Ableton Link integration
- Visual compositing (blend modes)
- Multi-page support (8+ tracks)
- Recording/export

---

## Development Philosophy

**Iterative Approach:**
- One feature per iteration
- Surgical modifications (not rewrites)
- Comprehensive documentation
- Systematic version control

**Quality Standards:**
- Professional UI/UX
- Clean code organization
- Token-efficient development
- User-friendly workflows

**Integration Success:**
- All V2.9.0 features preserved
- LIVE tab adds value without conflicts
- Single-file architecture maintained
- Performance optimized

---

**End of Changelog**
