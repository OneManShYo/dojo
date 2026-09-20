# ONEMANSHYO Dojo - Complete Changelog

## Version 2.6.0 (November 2025)
**Production Release - 20 Iterations**

Complete rewrite of presets system with user library management, render progress tracking, and intelligent filename handling.

---

### V2_6 Development Cycle

#### Iteration A - Presets Tab Structure
**File:** `OMS_Dojo_V2_6_A_PresetsTab_Notes.txt`

- Added new Presets tab to navigation
- Created collapsible sections (LIBRARY, USER PRESETS)
- Implemented preset grid layout system
- Added placeholder action buttons
- Foundation for preset management UI

#### Iteration B - Preset Data & Cards
**File:** `OMS_Dojo_V2_6_B_PresetData_Notes.txt`

- Created omsPresets array with 4 hardcoded shaders:
  - Tron Tunnel (Portrait)
  - Tron Grid (Portrait)
  - MetaBalls (Square)
  - Flower Power (Square)
- Added orientation preview control
- Implemented dynamic card generation
- Added loadPreset() functionality

#### Iteration C - Live Canvas Thumbnails
**File:** `OMS_Dojo_V2_6_C_LiveThumbnails_Notes.txt`

- Replaced static icons with live WebGPU canvas thumbnails
- Real-time animated shader previews
- Orientation-based aspect ratios (9:16, 16:9, 1:1)
- Tooltip system (hover for preset names)
- Simultaneous 4-context rendering

#### Iteration D - Shrink Cards & Error Handling
**File:** `OMS_Dojo_V2_6_D_FixThumbnails_Notes.txt`

- Reduced card sizes by ~40% (100px → 60px)
- Reduced canvas sizes by 50-75%
- Added shader compilation error handling
- Improved console logging for debugging
- Reduced GPU load significantly

#### Iteration E - Static Single-Frame Thumbnails
**File:** `OMS_Dojo_V2_6_E_StaticRender_Notes.txt`

- Replaced animation loops with single-frame renders
- Fixed time uniform at 2.0 seconds
- Eliminated ongoing GPU usage
- Removed requestAnimationFrame loops
- ~75% reduction in resource usage

#### Iteration F - Offscreen Render to 2D Canvas
**File:** `OMS_Dojo_V2_6_F_ImageRender_Notes.txt`

- Complete rendering architecture rewrite
- Offscreen WebGPU canvas with 2D display copy
- Bulletproof display via proven 2D canvas
- Fixed blank thumbnail issue
- Reliable cross-browser rendering

#### Iteration G - WebGPU Uniform Buffer Alignment
**File:** `OMS_Dojo_V2_6_G_BufferFix_Notes.txt`

- Fixed critical buffer size error (32 → 48 bytes)
- Added proper 16-byte alignment padding
- Fixed both thumbnail and main shader buffers
- Resolved WebGPU validation errors
- Proper struct memory layout

#### Iteration H - WGSL Struct Layout Fix
**File:** `OMS_Dojo_V2_6_H_StructAlignment_Notes.txt`

- Split resolution vec2 into separate resolutionX/Y
- Fixed vec2 alignment mismatch
- Updated all 5 shaders (default + 4 presets)
- Explicit memory layout control
- Resolved black thumbnail rendering

#### Iteration I - Array Order & Shader Replacement
**File:** `OMS_Dojo_V2_6_I_ArrayFix_Notes.txt`

- Fixed thumbnail uniform array order
- Matched struct layout exactly
- Replaced incorrect TronTunnel shader
- Added proper pink-blue tunnel effect
- Fixed resolution data reading

#### Iteration J - Tab Behavior & Panel Width
**File:** `OMS_Dojo_V2_6_J_TabFixes_Notes.txt`

- Fixed left panel width (locked at 400px)
- Removed auto-switch to Controls tab
- Added min/max/flex-shrink constraints
- Smooth preset browsing workflow
- No layout jumps when switching tabs

#### Iteration L - Real-Time Render Progress
**File:** `OMS_Dojo_V2_6_L_RenderCounter_Notes.txt`

- Added live frame counter during rendering
- Format: "Rendering [duration] | Frame X/Y"
- Updates every 5 frames (performance optimized)
- Essential for timing modulator tweaks
- Works for both audio and shader-only modes

#### Iteration M - Elapsed Time Counter
**File:** `OMS_Dojo_V2_6_M_TimeCounter_Notes.txt`

- Enhanced progress with elapsed/total time
- Format: "Rendering 00:15/00:30 | Frame 450/900"
- Dual progress indicators (time + frames)
- Math.min() prevents overshoot display
- Instant visual feedback on progress

#### Iteration N - Version in Filename
**File:** `OMS_Dojo_V2_6_N_VersionFilename_Notes.txt`

- Added V2_6 to rendered video filenames
- Format: OMS_[name]_V2.6.0_[resolution]p_[fps]fps_[orientation].webm
- Version tracking for troubleshooting
- Easy to identify app version per video

#### Iteration O - Fifth Preset Shader
**File:** `OMS_Dojo_V2_6_O_FifthPreset_Notes.txt`

- Added "Retro Fish Farm" as 5th preset
- Square orientation (16×16 grid + animated dots)
- Modulator assignments: dots (A), grid distortion (B), intensity (C)
- Completes 5-shader library row
- Retro/vaporwave aesthetic

#### Iteration P - Filename Format Corrections
**File:** `OMS_Dojo_V2_6_P_FilenameFix_Notes.txt`

- Reverted video filename (removed version)
- Added version to code export with "Shader" identifier
- Format: OMS_[name]_Shader_V2.6.0.txt
- Separate formats for video vs code
- Clear file type distinction

#### Iteration Q - Fallback Filename Consistency
**File:** `OMS_Dojo_V2_6_Q_FallbackFilename_Notes.txt`

- Changed code export fallback from timestamp to "filename"
- Matches video render default behavior
- Removed timestamp generation code
- Cleaner, simpler filenames
- Consistent across all exports

#### Iteration R - Parse Shader Name from Code
**File:** `OMS_Dojo_V2_6_R_ParseShaderName_Notes.txt`

- Added parseShaderNameFromCode() function
- Extracts name from "// ONEMANSHYO Dojo - [Name]" comments
- Fallback hierarchy: user → parsed → "filename"
- Automatic meaningful defaults
- Preserves shader identity

#### Iteration S - User Presets System
**File:** `OMS_Dojo_V2_6_S_UserPresets_Notes.txt`

- Complete user presets implementation
- Import multiple .txt/.wgsl shader files
- LocalStorage persistence
- Visual thumbnail grid with WebGPU rendering
- Keyboard navigation (Tab/Enter)
- Delete functionality
- Performance library workflow optimized

#### Iteration T - Clean UI
**File:** `OMS_Dojo_V2_6_T_CleanUI_Notes.txt`

- Removed "Save Current" button (no use case)
- Removed "Export" button (exists in Code tab)
- Removed placeholder messages
- Removed stub functions
- Streamlined interface (Import + Delete only)

#### Iteration U - Drag & Drop + Smart Delete
**File:** `OMS_Dojo_V2_6_U_DragDelete_Notes.txt`

- Drag-and-drop shader files onto grid
- Visual feedback (cyan border + glow)
- Focus tracking system for deletion
- One-click delete (no prompt!)
- Multiple file drop support
- Clean deletion workflow

---

## Version 2.5.0 (November 2025)
**24 Iterations - File Optimization & Render Accuracy**

Major achievements:
- 90% file size reduction (1.4MB → 142KB)
- Perfect audio/visual sync
- Accurate loop playback
- Clean UTF-8 encoding throughout
- Industry-standard terminology

Key features:
- Render Duration field for shader-only mode
- Fixed render visual timing
- Removed embedded changelog
- Fixed UTF-8 corruption
- Loop button cyan indicator

---

## Version 2.4.0 (November 2025)
**12 Iterations - Manual Beatgrid System**

Introduced Song Start/End fields with visual grid overlay for manual beat alignment. Replaced automatic BPM detection with user-controlled manual workflow.

---

## Version 2.3.0 (November 2025)
**10 Iterations - Module Separation**

Established Shader/Audio Module architecture with independent tempo controls. Manual tempo entry workflow, removed automatic BPM detection.

---

## Version History Summary

- **V2_6** (Nov 2025): Presets + Progress + UX - 20 iterations
- **V2.5.0** (Nov 2025): File optimization + sync fixes - 24 iterations  
- **V2.4.0** (Nov 2025): Manual beatgrid system - 12 iterations
- **V2.3.0** (Nov 2025): Module separation - 10 iterations
- **V2.2.0** (2025): Audio controls refinement
- **V2.1.0** (2025): Loop system implementation
- **V2.0.0** (2025): WebGPU foundation

---

## Development Methodology

**Incremental Approach:**
- One iteration per change (prevents crashes)
- Each iteration has dedicated notes file
- Version string updated throughout file
- Comprehensive testing per iteration
- Documentation before moving forward

**File Management:**
- Single HTML file architecture
- No embedded changelog (external notes)
- UTF-8 encoding verification
- File size monitoring
- ~200KB target maintained

**Quality Standards:**
- No breaking changes between iterations
- Backward compatibility preserved
- Professional code standards
- Comprehensive error handling
- User feedback integration

---

## Looking Forward

**Potential Future Iterations:**
- Preset search/filter functionality
- Drag-to-reorder presets
- Preset tags/categories
- Batch export selected presets
- Custom preset thumbnails
- Preset sharing/import URLs
- Real-time Ableton Link integration
- Native Mac app with Syphon
- TouchDesigner integration

**Core Principles Moving Forward:**
- Single-file philosophy maintained
- Open source GPL license
- Community-focused (not commercial)
- Professional standards
- Incremental development
- User-driven features

---

**Changelog Version:** V2_6  
**Last Updated:** November 2025  
**Total Iterations This Version:** 20 (A-U, skipping K)  
**File Size:** ~199KB  
**Status:** Production Release

