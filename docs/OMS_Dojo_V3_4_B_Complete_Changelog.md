# OMS SHADER BAKER - COMPLETE CHANGELOG V3.4.0

**Version:** V3.4.0 Production Release  
**Date:** November 15, 2025  
**Previous Version:** V3.3.0  
**Development Iterations:** 31 (A through AE)

---

## OVERVIEW

V3.4.0 is a **UI/UX refinement release** with no new functional features. The development focused on:
- Layout optimization for MacBook Pro 14" displays
- Visual consistency across all mixer controls
- Professional polish and modular aesthetic
- Code quality improvements and naming clarity

**Total Changes:** 31 iterations, ~440 lines modified  
**Development Time:** 1 day (November 15, 2025)  
**Approach:** Surgical changes, one improvement per iteration

---

## MAJOR CHANGES SUMMARY

### Layout & Display
- Optimized for 1352px viewport (MacBook Pro 14" default scaling)
- Reduced total layout from 1470px to 1320px
- All 6 mixer columns now visible with breathing room
- Equal column widths (153px) for consistent flexbox behavior
- Asymmetric padding (1px left, 4px right) for visual balance

### Visual Consistency
- Zoom knob matches volume knobs exactly (cyan arc, white line, border)
- Waveform controls use modular design (sharp corners, no gaps, perfect alignment)
- Grey borders use CSS border property (not pseudo-elements)
- Hidden scrollbars for cleaner appearance
- Professional hardware mixer aesthetic throughout

### Tab Organization
- Single-letter tab names: C, P, L, W, S, LIVE
- New Settings tab (S) for application-level controls
- Cleaner tab headers: "APPLICATION", "Containers"
- Library folder button moved to Settings tab
- Shader output only visible on LIVE tab

### Code Quality
- Refactored naming: audioLibrary → applicationFolder (complete)
- 42 variable renames, 4 function renames, 3 ID updates
- Fixed 2 broken function calls
- Cleaner CSS architecture (proper box model, no hacks)
- Better separation of concerns

---

## DETAILED CHANGELOG BY ITERATION

### V3_4_0_A - Initial Layout Reduction
**Focus:** MacBook Pro 14" viewport optimization

**Changes:**
- Scenes column: 90px → 70px (-20px)
- Mixer columns: 196px → 170px (-26px each, 4 columns = -104px total)
- Total grid width: 1070px → 920px (-150px)
- Total layout: 1470px → 1320px (fits 1352px viewport with 32px breathing room)
- Updated 4 CSS locations: preview-panel, grid-headers, scenes-grid, mixer-section

**Files Modified:** 8 lines
**Rationale:** Master column was completely cut off on MacBook Pro 14" at default scaling

---

### V3_4_0_B - CSS Borders Implementation
**Focus:** Replace pseudo-elements with proper CSS borders

**Changes:**
- Deleted ::before and ::after pseudo-elements (24 lines removed)
- Added `border-left: 10px solid #666` to .app-wrapper
- Added `border-right: 10px solid #666` to .app-wrapper
- Updated max-width: 1470px → 1340px
- Removed position: relative (not needed)
- Removed z-index: 9999 and pointer-events: none (not needed)

**Files Modified:** 30 lines
**Rationale:** Borders are outside content-box (no overlap). Cleaner code, proper box model.

---

### V3_4_0_C - Single-Letter Tab Names
**Focus:** Simplify tab navigation with single-letter labels

**Changes:**
- Controls → C
- Presets → P
- Library → L
- Code → W (WGSL - WebGPU Shading Language)
- Added Settings → S (new tab with empty panel)
- LIVE unchanged
- Updated switchTab() function to handle new Settings tab

**Files Modified:** 15 lines
**Rationale:** Cleaner UI, more space for content. Tab meanings clear in context.

---

### V3_4_0_D - Library Folder Button Relocation
**Focus:** Move application folder setup to Settings tab

**Changes:**
- Removed "Set Library Folder" button from Library tab (L)
- Removed library info box from Library tab
- Added button to Settings tab (S)
- Added info box to Settings tab
- Preserved all IDs - no JavaScript changes needed

**Files Modified:** 20 lines
**Rationale:** Settings tab for app-level config. Library tab for content management.

---

### V3_4_0_E - Naming Refactor: audioLibrary → applicationFolder
**Focus:** Clarify naming to reflect actual folder purpose

**Changes:**
- Variable renames: 42 occurrences
- Function renames: 4 occurrences (setup, store, load)
- Element ID renames: 3 occurrences
- UI text updates: "Set Application Folder", "Change Folder", "Application Folder"
- Error message updates: All references updated
- Bug fixes: Fixed 2 broken function calls (setAudioLibrary → setupApplicationFolder)

**Files Modified:** 60 lines
**Rationale:** Folder stores ALL app data (.oms, .omsp, audio), not just audio files.

---

### V3_4_0_F - Tab Header Updates & Canvas Visibility
**Focus:** Cleaner tab labels and proper shader canvas visibility

**Changes:**
- Settings tab header: "SETTINGS" → "APPLICATION"
- Library tab section header: "Container Library" → "Containers"
- liveShaderCanvas now hidden on C, P, L, W, S tabs
- liveShaderCanvas only shows on LIVE tab
- Updated switchTab() function with visibility toggling (7 locations)

**Files Modified:** 10 lines
**Rationale:** Tab headers match content. Shader canvas only needed during LIVE performance.

---

### V3_4_0_G - Waveform Button Simplification
**Focus:** Remove unnecessary quantize buttons, set uniform width

**Changes:**
- Removed Q (quantize) buttons from all 4 tracks
- Set all waveform control buttons to 70px width
- Button layout: 2 buttons per row (|<, >|) and (<<, >>)
- Added width: 70px to .track-control-btn CSS

**Files Modified:** 15 lines
**Rationale:** Everything always quantized. 70px matches SCENES column width.

---

### V3_4_0_H - Perfect 70px Alignment
**Focus:** Make waveform controls exactly 70px wide to align with SCENES

**Changes:**
- Container width: 70px
- All buttons uniform: 32px each
- Container padding: 2px 3px → 2px
- Button row gap: 3px → 2px
- Math: 2 + 32 + 2 + 32 + 2 = 70px ✓
- Removed all width overrides (.quantize, .bar-left, .bar-right, .nudge-left, .nudge-right)

**Files Modified:** 30 lines
**Rationale:** Perfect pixel alignment with SCENES column above.

---

### V3_4_0_I - Modular Look (Sharp & Tight)
**Focus:** Create professional modular hardware aesthetic

**Changes:**
- Container padding: 2px → 0 (all sides)
- Container border-radius: 2px → 0 (sharp corners)
- Container gap: 2px → 0 (no vertical gap)
- Button row gap: 2px → 0 (no horizontal gap)
- Button border-radius: 2px → 0 (sharp corners)
- Button width: 32px → 35px (2 × 35px = 70px perfect fill)
- Fixed button label: |> → >| (correct advance direction)

**Files Modified:** 20 lines
**Rationale:** Modular, sharp, professional - no rounded corners, no gaps, perfect fill.

---

### V3_4_0_J - Flush Against Waveform
**Focus:** Eliminate ALL extra space between controls and waveform

**Changes:**
- waveform-track gap: 2px → 0 (controls flush against canvas)
- waveform-track column: 90px → 70px (matches control width)
- Container justify-content: center → flex-start (no vertical centering)
- Container height: 100% (fills track height)
- Button rows flex: 1 (each row fills 50% height)
- Button height: 24px → 100% (fills row height)

**Files Modified:** 15 lines
**Rationale:** NO extra space anywhere. Tight, clean, professional.

---

### V3_4_0_K - Perfect Alignment (No Bleeding)
**Focus:** Prevent waveform from showing through control gaps

**Changes:**
- Container align-items: center → stretch
- Button rows now fill full 70px width
- No space on left or right sides
- No waveform bleeding through
- Perfect alignment with SCENES column

**Files Modified:** 5 lines
**Rationale:** Solid background blocks waveform. Perfect horizontal alignment.

---

### V3_4_0_L - Waveforms Section Padding Removal
**Focus:** Edge-to-edge waveforms with visual track separation

**Changes:**
- Removed padding: 5px from .waveforms-section
- Kept gap: 2px (visual separation between tracks)
- Waveforms now extend to full width

**Files Modified:** 3 lines
**Rationale:** No edge padding. Track gaps provide sufficient separation.

---

### V3_4_0_M - Shader Output Visibility Control
**Focus:** Show shader output only on LIVE tab

**Changes:**
- Added liveShaderOutputContainer ID
- Set default display: none
- Updated switchTab() to show/hide based on active tab
- Hidden on: C, P, L, W, S tabs
- Visible on: LIVE tab only

**Files Modified:** 10 lines
**Rationale:** Shader output preview specific to LIVE performance interface.

---

### V3_4_0_N - Zoom Knob Label & Readout Hiding
**Focus:** Match zoom knob to volume knob minimalism

**Changes:**
- Hidden .zoom-label (display: none)
- Hidden .zoom-value readout (display: none)
- Attempted minimalist look

**Files Modified:** 5 lines
**Note:** Later reverted in V3_4_0_P - label needed for clarity

---

### V3_4_0_O - Vertical Space Recovery
**Focus:** Maximize canvas display on MacBook Pro 14"

**Changes:**
- Removed "LIVE SHADER OUTPUT" label above canvas
- Recovered ~23px vertical space (13px text + 10px margin-bottom)
- Canvas maintains 16:9 aspect ratio

**Files Modified:** 5 lines
**Rationale:** Critical for full canvas display on smaller laptop screens.

---

### V3_4_0_P - Exact Visual Match to Volume Knobs
**Focus:** Pixel-perfect zoom knob matching volume knobs

**Changes:**
- Copied EXACT drawing code from drawVolumeKnob() to drawZoomKnob()
- Same angle range: -150° to +150° (300° total sweep)
- Same Math.PI rotation flip
- Same cyan arc: radius 22, lineWidth 3, #00d4ff
- Same white center line indicator: lineWidth 2, #fff
- Same border: #444, lineWidth 2
- Restored "ZOOM" label (kept visible)
- Removed small dot indicator
- Added proper background circle with border

**Files Modified:** 40 lines
**Rationale:** "Don't look like a junk shop" - professional consistency required.

---

### V3_4_0_Q - Shader Output Outside Scrollable Area
**Focus:** Fix shader output at bottom of left panel

**Changes:**
- Restructured library tab layout completely
- Moved liveShaderOutputContainer OUTSIDE .controls div
- Made it sibling of .controls within .controls-section
- Added flex-shrink: 0 to shader output container
- Added min-height: 0 to .controls (critical for nested scrolling)
- Container list scrolls independently
- Shader output always visible at bottom

**Files Modified:** 50 lines
**Rationale:** Monitor shader while browsing containers. Proper flexbox positioning.

---

### V3_4_0_R - Zoom Mixer Layout Structure
**Focus:** Use same container structure as volume knobs

**Changes:**
- Replaced custom .zoom-control with .mixer-layout + .mixer-left structure
- Same flexbox layout as volume knobs
- Removed .zoom-value display element
- Kept .zoom-label
- Updated JavaScript to remove zoomValue display updates

**Files Modified:** 10 lines
**Rationale:** Structural consistency required for proper alignment.

---

### V3_4_0_S - Zoom Two-Row Structure
**Focus:** Match track controls' two-row layout

**Changes:**
- Added .track-control-row for ZOOM label (top row)
- Kept .mixer-layout for zoom knob (bottom row)
- Row 1: ZOOM label (aligned with M/S buttons)
- Row 2: Zoom knob (aligned with volume knobs)

**Files Modified:** 15 lines
**Rationale:** Perfect horizontal alignment at both row levels.

---

### V3_4_0_T - Container CSS Match
**Focus:** Make .mixer-spacer CSS identical to .track-controls

**Changes:**
- Added padding: 10px to .mixer-spacer
- Added display: flex
- Added flex-direction: column
- Added gap: 8px
- Removed margin: 0 0

**Files Modified:** 8 lines
**Rationale:** Same CSS properties = same layout behavior. Critical fix.

---

### V3_4_0_U - Flex Constraint Experiment
**Focus:** Prevent mixer-layout vertical expansion

**Changes:**
- Added flex: 0 inline style to zoom's .mixer-layout
- Attempted to match tight container height

**Files Modified:** 3 lines
**Result:** Wrong approach - knob pushed to top instead of centered

---

### V3_4_0_V - Flex Expansion Restoration
**Focus:** Allow natural vertical centering

**Changes:**
- Removed flex: 0 inline style
- Let .mixer-layout expand (flex: 1 from CSS)
- Allows .mixer-left to center knob vertically via justify-content: center

**Files Modified:** 3 lines
**Result:** Better but alignment still not perfect

---

### V3_4_0_W - Horizontal Centering
**Focus:** Center zoom controls horizontally

**Changes:**
- Added justify-content: center to .track-control-row
- Added justify-content: center to .mixer-layout
- ZOOM label centered in top row
- Zoom knob centered in bottom row

**Files Modified:** 5 lines
**Result:** Centered but still fighting CSS behavior

---

### V3_4_0_X - Height Constraint
**Focus:** Constrain mixer-left height

**Changes:**
- Added height: auto to .mixer-left
- Added max-height: 60px to .mixer-left
- Attempted to prevent vertical expansion

**Files Modified:** 3 lines
**Result:** Still not matching volume knob positioning

---

### V3_4_0_Y - Strip All Inline Overrides
**Focus:** Trust CSS classes, remove inline styles

**Changes:**
- Removed ALL inline styles from zoom control HTML
- No justify-content: center
- No height: auto; max-height: 60px
- Let CSS classes handle layout naturally
- Same approach as volume knobs (zero inline styles)

**Files Modified:** 8 lines
**Rationale:** Inline styles override CSS and break designed layout. Let CSS work.

---

### V3_4_0_Z - Remove Gap for No Meters
**Focus:** Final alignment fix

**Changes:**
- Added gap: 0 inline style to zoom's .mixer-layout
- Track controls need gap (knob + space + meters)
- Zoom has no meters, so gap creates unwanted empty space

**Files Modified:** 3 lines
**Result:** PERFECT alignment achieved! Final iteration of alignment saga.

---

### V3_4_0_AA - Equal Column Widths
**Focus:** Uniform mixer column widths for consistent flex behavior

**Changes:**
- Calculated uniform width: 920px ÷ 6 = 153px per column
- Updated grid-template-columns (3 declarations):
  - grid-headers: 153px × 6
  - scenes-grid: 153px × 6
  - mixer-section: 153px × 6
- All columns now 153px (scenes, tracks 1-4, master)
- 918px total + 2px buffer = 920px

**Files Modified:** 5 lines
**Rationale:** CSS flexbox behavior depends on container width. Equal widths ensure consistency.

---

### V3_4_0_AB - Centered Zoom Controls
**Focus:** Horizontal centering within equal-width column

**Changes:**
- Added justify-content: center to zoom's .track-control-row
- Added justify-content: center to zoom's .mixer-layout
- ZOOM label centered in 153px column (top row)
- Zoom knob centered in 153px column (bottom row)

**Files Modified:** 5 lines
**Rationale:** Now that columns are equal width, centering works properly.

---

### V3_4_0_AC - Fixed Shader Output (Flexbox)
**Focus:** Shader output at bottom of left panel using proper flexbox

**Changes:**
- Moved liveShaderOutputContainer to direct child of .left-panel
- Positioned after all tab-content divs
- Changed flex-shrink: 0 to flex: 0 0 auto (more explicit)
- Library content scrolls BEHIND fixed shader output
- Proper flexbox positioning hierarchy

**Files Modified:** 30 lines
**Rationale:** Clean flexbox solution. Shader output escapes scrollable area.

---

### V3_4_0_AD - Hidden Scrollbar
**Focus:** Clean UI with hidden scrollbar (scroll still works)

**Changes:**
- Added scrollbar hiding CSS to .tab-content:
  - Chrome/Safari: ::-webkit-scrollbar { display: none; }
  - Firefox: scrollbar-width: none;
  - IE/Edge: -ms-overflow-style: none;
- Scroll functionality intact
- Cross-browser compatible

**Files Modified:** 10 lines
**Rationale:** Modern web apps hide scrollbars for cleaner appearance. Visual polish.

---

### V3_4_0_AE - Asymmetric Padding
**Focus:** Balanced black space between content and grey borders

**Changes:**
- Added padding-left: 1px to .preview-panel
- Added padding-right: 4px to .preview-panel
- Creates visual breathing room on right side
- More balanced black space distribution

**Files Modified:** 5 lines
**Rationale:** User preference for more space on right, less on left.

---

## VERSION COMPARISON

### V3.3.0 (Previous)
- Total layout: 1470px wide
- Mixer columns: 90px scenes, 196px tracks
- Rounded corners on waveform controls
- Pseudo-element grey borders
- "Audio Library" naming
- Zoom knob different style from volume knobs
- Scrollbar visible
- Container Library header

### V3.4.0 (Current)
- Total layout: 1320px wide (-150px)
- Mixer columns: 153px uniform across all 6
- Sharp corners on waveform controls (modular look)
- CSS border property (proper box model)
- "Application Folder" naming
- Zoom knob matches volume knobs exactly
- Scrollbar hidden
- Containers header

---

## FILES CHANGED SUMMARY

**Total Lines Modified:** ~440 across 31 iterations

**Major File Sections:**
- CSS (lines 71-1477): Layout, styling, borders, waveform controls, scrollbars
- HTML Structure (lines 1978-8382): Tab content, shader output, zoom control
- JavaScript (lines 4447-8417): Tab switching, zoom knob drawing, naming refactor

**Version References Updated:**
- Line 3: HTML comment
- Line 10: Version comment
- Line 17: Title tag
- Line 1543: UI header display

---

## BREAKING CHANGES

**None.** All changes are backward compatible.

- .oms file format unchanged
- .omsp file format unchanged
- All existing shaders continue to work
- All existing project states load correctly
- localStorage keys unchanged (applicationFolderHandle)
- IndexedDB structure unchanged

---

## UPGRADE PATH

No special upgrade steps required. Simply replace V3.3.0 HTML file with V3.4.0 HTML file.

Your existing:
- Application folder location (persists via browser storage)
- .oms container files (compatible)
- .omsp project state files (compatible)
- Shader code (unchanged)
- Audio files (compatible)

All continue to work without modification.

---

## KNOWN ISSUES

**None reported.**

All 31 iterations tested incrementally during development.
Final V3.4.0 production build tested on MacBook Pro 14" at default scaling.

---

## NEXT RELEASE

**V3.5.0 - Planned Features:**
- Scene launching (trigger entire rows in performance grid)
- Quantized launching (snap to bars/phrases)
- Loop bracket visualization
- Crossfading between cells
- Web MIDI API integration
- Hardware controller support (Midifighter Twister)
- Ableton Link synchronization

V3.4.0 provides the refined foundation for these upcoming performance features.

---

**END OF COMPLETE CHANGELOG**

For user-facing release notes, see: OMS_ShaderBaker_V3_4_0_D_Release_Notes.md  
For developer documentation, see: OMS_ShaderBaker_V3_4_0_C_Developer_Documentation.md
