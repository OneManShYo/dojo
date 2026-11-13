# OMS SHADER BAKER V3.8.0 - COMPLETE CHANGELOG

**Version:** V3.8.0 Production Release  
**Release Date:** November 17, 2025  
**Base Version:** V3.7.0  
**Development Iterations:** 26 (V3_8_0_A through V3_8_0_Z)

---

## MAJOR CHANGES

### User Experience

✅ **Custom Header Greeting**
- ONEMANSHYO branding (24px gradient) + Shader Baker [version] (11px grey)
- Custom greeting text (right side, default: "Hi Friend, I'm Wes!")
- Settings tab input (50 char max)
- Real-time updates, persists in .omsp files

✅ **Single-Line Header**
- Redesigned from stacked (2 lines) → single-line layout
- Horizontal flex layout (baseline aligned)
- 12px gap between brand and app name
- Same 50px height, cleaner appearance

✅ **Ergonomic Keyboard Controls**
- Opacity: ←→ arrows (was Shift+↑↓) ← MAJOR IMPROVEMENT
- Volume: ↑↓ arrows (unchanged)
- No modifier keys needed for performance controls
- Right hand stays in position (vertical + horizontal)
- Left hand: WASD navigation

✅ **Library Auto-Blur**
- Library cards auto-blur during performance
- Prevents scroll interference with arrow keys
- Triggers on: track selection (1-5, Tab), arrow key usage
- Automatic, no user action needed

✅ **Folder Management**
- App-level overlay (single, not per-tab)
- ESC or Cancel to dismiss
- C/W/P tabs work without folder
- L/S/LIVE tabs require folder
- Clean, unified UX

### Navigation & Layout

✅ **Scene Controls Repositioned**
- Moved from LEFT column (Scenes) → RIGHT column (Master)
- Scene triggers align with Master controls
- Left column removed (was empty)

✅ **5-Column Grid**
- Removed empty Scenes column
- Redistributed space: 151px → 181px (30px wider)
- Grid: [Track 1] [Track 2] [Track 3] [Track 4] [Master/Scenes]
- Total: 5 columns × 181px = 905px

✅ **WASD Navigation Updated**
- A key: Scene mode → enters Track 4 (was: did nothing)
- D key: Track 4 → exits to scene mode (new)
- Logic: Scenes on right = enter from right
- Track 1 + A: Does nothing (at leftmost)

✅ **Cell Selection Persistence (ROOT FIX)**
- Cell selection persists when adjusting track volume
- Removed `selectedCell = null` from track selection (1-5 keys, Tab)
- Navigate to cell → select track → adjust volume → cell stays selected
- Enter launches correct cell
- Simpler code, correct behavior

### Zoom Control

✅ **Trackpad Pinch-to-Zoom**
- Continuous zoom with trackpad gestures
- Pinch = zoom out, spread = zoom in
- Natural, hardware-like control

✅ **Bracket Keys Zoom**
- `[` key: Zoom out (10px step)
- `]` key: Zoom in (10px step)
- Discrete steps for precise control
- Global `window.updateZoom` function

✅ **Visual Zoom Control Removed**
- Zoom knob removed from UI (S iteration)
- Functionality preserved (function still exists)
- Cleaner interface

### Content & Organization

✅ **About Tab (NEW)**
- Identity: Title, version, creator, copyright
- ONEMANSHYO: Bio, achievements, links
- TECHNICAL: Browser, shader lang, audio engine, file format
- TOOLS: Claude, WebGPU, Web Audio, File System API
- **INSPIRED BY:** 27 people/companies/communities

✅ **PROJECT STATES Moved**
- Relocated from Controls tab → Settings tab
- Better organization (Settings tab now has: Application, Project States, Greeting, Pop-Out)

✅ **Preview Orientation Fix**
- "Preview As:" dropdown now updates main canvas (not just preset cards)
- Portrait/Square/Landscape orientations
- Independent from render settings (Controls tab)

---

## ITERATION-BY-ITERATION CHANGELOG

### V3_8_0_A-C: Header Greeting System
**Features:**
- Custom greeting text (50 char max)
- Settings tab input
- Real-time updates
- Persists in .omsp files
- Fixed save/load bug in C iteration

---

### V3_8_0_D: PROJECT STATES Organization
Moved PROJECT STATES section from Controls tab → Settings tab

---

### V3_8_0_E-H: Folder Management & Overlays
**E-H iterations created:**
- App-level overlay (not per-tab)
- ESC/Cancel to dismiss
- C/W/P tabs work without folder
- L/S/LIVE tabs require folder

---

### V3_8_0_I: Ergonomic Opacity Control
Remapped shader opacity from Shift+↑↓ to ←→ arrows

**Keyboard Layout:**
- Left hand (WASD): Scene/cell navigation
- Right hand: ↑↓ (volume), ←→ (opacity)
- No modifier keys needed

---

### V3_8_0_J: Library Scroll Fix
Library cards auto-blur during performance

---

### V3_8_0_K: Single-Line Header
Redesigned header from stacked → single-line layout

---

### V3_8_0_L-N: Zoom Control Suite
**Three zoom methods:**
1. Trackpad gestures (continuous)
2. Bracket keys (discrete)
3. Zoom knob visual (removed in S)

---

### V3_8_0_O: Enter Key Band-Aid (OBSOLETE)
Root problem fixed in R, this removed

---

### V3_8_0_P-R: Code Cleanup & Root Fix
**R: WORKING ROOT FIX**
- Removed `selectedCell = null` from track selection
- Cell position persists when selecting tracks
- Enter launches correct cell
- Simpler code, correct behavior

---

### V3_8_0_S: Scene Controls Repositioned
Moved scene triggers from LEFT → RIGHT column

---

### V3_8_0_T: WASD Navigation Fix
Fixed WASD after scene triggers moved right

---

### V3_8_0_U: 5-Column Layout
Removed empty Scenes column, 30px wider cells

---

### V3_8_0_V: Master Controls Fix
Removed mixer-spacer div

---

### V3_8_0_W: Preview Orientation Fix
"Preview As:" updates main canvas

---

### V3_8_0_X: About Tab
Added About tab between Settings and LIVE

---

### V3_8_0_Y: About Tab Updates
Fixed URLs, name, added INSPIRED BY credits

---

### V3_8_0_Z: About Tab Alignment Fix
Fixed text alignment using inline styles

---

## KEYBOARD SHORTCUTS (UPDATED)

### Shader Controls (Right Hand - Horizontal) ← UPDATED
- `→`: Opacity up ← CHANGED from Shift+↑
- `←`: Opacity down ← CHANGED from Shift+↓
- `Shift+,`: Shader mute/unmute
- `Shift+.`: Shader solo/unsolo

### Zoom (Either Hand) ← NEW
- `[`: Zoom out (10px step)
- `]`: Zoom in (10px step)
- Trackpad: Pinch/spread gestures

### Audio Controls (Right Hand - Vertical)
- `↑`: Volume up
- `↓`: Volume down
- `,`: Mute/unmute
- `.`: Solo/unsolo

---

## BUG FIXES

🛠️ **Greeting Save/Load** - Added to project state  
🛠️ **Presets Folder** - Removed requirement  
🛠️ **Enter Key** - Root cause fixed  
🛠️ **Bracket Key Scope** - Made global  
🛠️ **WASD Navigation** - Updated for right-side scenes  
🛠️ **Master Controls** - Removed spacer div  
🛠️ **Preview Orientation** - Refactored function  
🛠️ **About Tab** - Fixed URLs and alignment

---

## PERFORMANCE

- 4 shaders: ~30-45% GPU (same as V3.7.0)
- 60fps stable
- V3.8.0 adds 0% GPU overhead (UI only)
- File size: 550KB (+25KB)

---

## COMPATIBILITY

✅ Fully backward compatible with V3.7.0  
✅ No migration required  
✅ All features preserved

---

## TAB ORGANIZATION (UPDATED)

**C (Controls):** Shader creation *(no folder)*  
**P (Presets):** OMS library *(no folder)*  
**L (Library):** .oms containers *(folder required)*  
**W (Code):** Shader editor *(no folder)*  
**S (Settings):** Application config *(folder required)*  
**A (About):** Project info *(no folder)* ← NEW  
**LIVE:** Performance mixer *(folder required)*

---

**END OF COMPLETE CHANGELOG**

Version: V3.8.0  
Date: November 17, 2025  
Iterations: 26 (A-Z)  
License: GPL-3.0
