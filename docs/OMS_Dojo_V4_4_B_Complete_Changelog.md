# ONEMANSHYO BETA V4_4_0 - COMPLETE CHANGELOG

**Version:** Beta V4_4_0  
**Release Date:** December 6, 2025  
**Base Version:** V4_3_0_A  
**Development Iterations:** 55 sub-iterations (V4_4_01 through V4_4_55)

---

## EXECUTIVE SUMMARY

V4_4_0 is a major UI restructure release that reorganizes the application around a cleaner workflow: **Create → Organize → Perform**. The Library tab gets a complete two-panel layout, a unified drag/drop system replaces scattered import buttons, and shader-only containers now work in the LIVE performance grid.

**Key Achievement:** Professional DAW-style library management with Sets filtering and unified file handling.

---

## MAJOR FEATURES

### 1. Library Tab Two-Panel Layout (sub01-sub07)

✅ **Split Library Tab into Two Columns**
- Left panel (400px): SETS list with shader preview
- Right panel (920px): CONTAINERS list with audio controls

✅ **Sets Panel Features**
- "All" set shows user containers from Library/ folder
- "Presets" set shows embedded system presets (no folder needed)
- Shader preview canvas mirrors main output
- Set cards highlight cyan when selected

✅ **Containers Panel Features**
- Three sortable columns: Name, Audio (Y/N), BPM
- 32x18 shader thumbnails on each card
- Load/Update/Export/Add Current/X toolbar buttons
- Full keyboard navigation

### 2. Unified Drag/Drop System (sub25-sub33)

✅ **Single Drop Handler for All File Types**
- Audio files (.mp3, .wav, etc.): Create container on Library tab, load audio on others
- Shader files (.txt, .wgsl): Load code and compile on any tab
- Container files (.zip): Import on Library tab only

✅ **Visual Feedback**
- Full-screen dark overlay (85% black) on drag
- Cyan upload icon + "Drop files here to add" text
- Only shows for external file drops (not internal card drags)

✅ **Removed UI Elements**
- I (Import) button
- + (Add) button
- Upload button in waveform area
- "Drag Audio Here" text overlay

### 3. Sets System (sub40-sub53)

✅ **Two Built-in Sets**
- Presets: Embedded system presets (works without folder)
- All: User containers from Library/ folder

✅ **State Persistence**
- Selected set saves to .omsp project file
- Restores on app load
- LIVE tab dropdown synced with Library selection

✅ **System Presets Copy**
- On first folder setup, presets copied to Library/
- Saved as editable .oms files with `source: 'system-preset'`

### 4. Shader-Only Container Launch (sub54-sub55)

✅ **LIVE Grid Support**
- Containers without audio now launch properly
- Visual state (playing class, overlay) applies to all
- Compositor renders shader at track opacity

✅ **Detection Fix**
- Changed from audio source check to .playing class
- Status: "Launched: ShaderName (shader only)"

### 5. Audio Workflow Improvements (sub16-sub24)

✅ **Audio Boundary Pinning**
- Click Audio Start/End: Reset to 0 or max
- Shift+click: Pin to current playhead position

✅ **Loop Controls**
- Cyan brackets (#00ffff) for Loop In/Out
- 25% cyan fill for loop region
- Toggle Loop OFF hides brackets
- Loop state saves to .oms container

✅ **Navigation**
- Arrow keys navigate entire file (not clamped to loop)
- Spacebar plays from clicked position
- Removed "played portion" darkening

### 6. User Presets System (sub34-sub38)

✅ **Folder Storage**
- Presets saved to /Presets/ folder
- File format: .wgsl with orientation comment
- Folder created lazily on first save

✅ **P Tab Sections**
- SYSTEM: Built-in presets (read-only)
- USER: Saved presets with X delete button

### 7. Tab System Reorganization (sub29)

✅ **New Tab Order**
C P W | L LIVE | S U A

✅ **Color Coding**
- Cyan: Creation (Controls, Presets, WGSL) - no folder needed
- Magenta: Library/Performance (Library, LIVE) - folder required
- Yellow: Utility (Settings, Utilities, About)

✅ **Default State**
- Controls tab active on load
- Shader visible immediately
- Out-of-box: load → see shader → create

### 8. Keyboard Shortcuts (sub42)

✅ **Shift+Enter: Shader-Only Load**
- Enter: Full container load (replaces everything)
- Shift+Enter: Load shader only (keep audio)
- Enables fast shader auditioning with your audio

---

## ITERATION HISTORY

### Sub01-07: Library Two-Panel Layout

**sub01:** Initial two-panel split
- Left panel: SETS (140px initially)
- Right panel: CONTAINERS with existing functionality
- Branding update: "ONEMANSHYO" (removed "Shader Baker")

**sub02:** Layout refinement
- Left panel: 400px, right panel: 920px
- Full container list with toolbar

**sub03:** Audio controls integration
- Waveform section below containers
- Audio controls strip visible

**sub04:** Display fix
- Container cards now full-width (display: block, not flex)

**sub05:** Shader preview
- Added shader canvas to left panel
- Mirrors main output in real-time

**sub06:** Refresh fix
- Added refreshLibraryListRight() after container operations
- Active dot now updates properly

**sub07:** Focus fix
- Tab-aware refocus after container load
- Arrow keys continue working after Enter

---

### Sub08-15: LIVE Tab & Drag/Drop Foundation

**sub08:** LIVE tab SET dropdown
- Replaced buttons with dropdown selector
- Clean separation: Library manages, LIVE consumes

**sub09:** Layout consistency
- "Set" as shader-header matching Library styling

**sub10:** Delete refresh fix
- Always rescan folder after delete

**sub11:** Library tab drag/drop fix
- Added listeners to libraryListRight

**sub12:** Audio audition fix
- Play from clicked position (not forced to Loop In)

**sub13:** Unified drag/drop (initial)
- Single handler for Library tab
- Routes audio vs .zip files

**sub14:** Tab reorder (initial)
- L P C W S A U LIVE order
- Library as default tab

**sub15:** Audio controls section container
- Renamed waveform-section → audio-waveform
- Added audio-controls-section wrapper

---

### Sub16-24: Audio Workflow

**sub16:** Arrow key navigation fix
- Navigate full file (0 to end)
- Not clamped to loop boundaries

**sub17:** Loop visual toggle
- Magenta brackets hide when Loop OFF

**sub18:** Audio Start pinning
- Click: reset to 0
- Shift+click: pin to playhead

**sub19:** Magenta playhead
- Matches LIVE tab playhead color

**sub20:** Cyan loop brackets
- Loop In/Out now cyan (#00ffff)
- 25% cyan fill for loop region

**sub21:** Audio End pinning
- Same pattern as Audio Start

**sub22:** Save loop enabled state
- audioLoopEnabled saved to .oms

**sub23:** Remove played darkening
- Cleaner waveform display

**sub24:** Loop off start position
- Container loads to Audio Start when loop disabled

---

### Sub25-33: Unified Drop Handler

**sub25:** Unified Library drag/drop
- Removed I, +, Upload buttons
- Single handler routes by file type

**sub26:** Single drop zone
- Handler on .container (entire app)
- No visual highlights (cleaner)

**sub27:** Drop overlay
- Full-screen dark overlay on drag
- Cyan icon + text

**sub29:** Tab reorder with colors
- C P W | L LIVE | S U A
- Cyan/Magenta/Yellow coding
- Controls as default tab

**sub30:** Unified drop handler (all tabs)
- Routes by file type AND active tab
- Audio: create container (L) or load (others)
- Shader: load code (all tabs)
- ZIP: import (L only)

**sub31:** Preset audio fix
- compileShader(false) prevents audio restart

**sub32:** Delete container + audio
- Deletes both .oms and referenced audio file

**sub33:** Internal drag fix
- isExternalFileDrop() prevents overlay on card drags

---

### Sub34-42: Presets & Features

**sub34:** User presets folder storage
- /Presets/ folder for .wgsl files
- W tab Save button

**sub35:** User presets UX polish
- Styled empty state messages

**sub36:** Library buttons full text
- Load, Update, Export, Add Current, X

**sub37:** User presets style match
- Matches Settings tab styling

**sub38:** Empty user presets grid
- No messages, just empty grid
- Learn on first Save attempt

**sub40:** System set in Library
- Presets shows embedded omsPresets
- Diamond indicator for presets

**sub41:** Presets section in Library
- PRESETS header + SETS section
- System presets copy to Library on folder setup

**sub42:** Shift+Enter shader only
- Load shader code without clearing audio

---

### Sub43-55: Polish & Fixes

**sub43:** Presets as regular set
- Both are set cards (alphabetical)
- Added Audio column (Y/N)

**sub44:** Sortable audio column
- Click Audio header to sort

**sub45:** Sort fix + thumbnails
- Async sort functions
- 32x18 shader thumbnails
- Hidden scrollbar

**sub46:** Set filter fix
- Removed undefined function call
- Active indicator updates on preset load

**sub47:** Arrow nav focus fix
- Centralized refocus in refreshLibraryListRight()

**sub48:** Remove diamond indicator
- Only ● for loaded status

**sub49:** All containers draggable
- Presets draggable to LIVE grid

**sub50:** Refresh on folder set
- Lists load immediately after setup

**sub51:** Save/restore selected set
- selectedSetId in .omsp

**sub53:** Fix set restore
- Removed default assignment that overwrote restore

**sub54:** Shader-only container launch
- Renamed playCellAudio → launchCell
- Audio setup conditional

**sub55:** Shader-only compositor
- Detection via .playing class
- Proper opacity for shader-only containers

---

## ARCHITECTURE CHANGES

### New Global Variables

```javascript
let selectedLibrarySetId = 'presets';     // Current set filter
let selectedLiveSetId = 'presets';        // Synced with Library
let userPresetsCache = [];                // User presets from /Presets/
```

### Modified Functions

```javascript
refreshLibraryList()        // Handles 'presets' vs 'all' sets
refreshLibraryListRight()   // Renders right panel, handles thumbnails
selectLibrarySet()          // Updates both panels + dropdown
loadContainer()             // Handles preset_ prefix
launchCell()                // (renamed from playCellAudio) handles shader-only
calculateTrackShaderOpacity() // Detection via .playing class
handleUnifiedDrop()         // Routes all file drops
```

### New Functions

```javascript
handleLibraryDrop()         // Main drop handler
isExternalFileDrop()        // Distinguishes external vs internal drags
loadShaderOnly()            // Shift+Enter functionality
setAudioStart(event)        // Click/Shift+click handler
setAudioEnd(event)          // Click/Shift+click handler
renderLibraryThumbnails()   // Async thumbnail rendering
saveShaderToPresets()       // W tab Save button
copyPresetsToLibrary()      // System presets copy on folder setup
```

---

## BUG FIXES

| Issue | Cause | Fix |
|-------|-------|-----|
| Active dot not updating | Only refreshLibraryList() called | Added refreshLibraryListRight() |
| Focus lost after load | Wrong panel selector | Tab-aware selector |
| Delete didn't update UI | Cache not rescanned | Always rescan folder |
| Preset selection started audio | compileShader(true) | Changed to compileShader(false) |
| Internal drags triggered overlay | No external file check | Added isExternalFileDrop() |
| Set not restoring from .omsp | Default overwrote restore | Removed default assignment |
| Arrow nav lost after load | No refocus | Centralized in refreshLibraryListRight() |
| Sort not working | Async not awaited | Made functions async |
| Shader-only no compositor | Audio source check | Changed to .playing class check |

---

## DATA STRUCTURE CHANGES

### Container .oms (Updated)

```javascript
{
  // ... existing fields ...
  audioLoopEnabled: true    // NEW: Loop on/off state
}
```

### Project .omsp (Updated)

```javascript
{
  // ... existing fields ...
  selectedSetId: "all"      // NEW: Current set selection
}
```

---

## REMOVED/DEPRECATED

### UI Elements Removed
- I (Import) button
- + (Add) button  
- Upload button in waveform area
- "Drag Audio Here" text overlay
- Diamond (◆) indicator

### Functions Deprecated
- playCellAudio() → renamed to launchCell()

---

## STATISTICS

- **Total Sub-iterations:** 55
- **Lines Changed:** ~1500+
- **Functions Modified:** 15+ core functions
- **Functions Added:** 10+
- **UI Components Changed:** Library tab, audio controls, tabs, overlays

---

## PRODUCTION READINESS

**Overall: ~92%**

**100% Complete:**
- Two-panel Library layout
- Sets system (Presets/All)
- Unified drag/drop
- Shader thumbnails
- Audio workflow improvements
- Shader-only container launch

**Inherited from V4_3_0_A (100%):**
- Folder as source of truth
- Container CRUD
- Grid alignment
- Audio engine
- Time display

**Remaining:**
- MIDI control integration
- Edge case testing
- Final polish

---

**END OF COMPLETE CHANGELOG**

Version: Beta V4_4_0
Date: December 6, 2025
Total Development: 55 sub-iterations
License: GPL-3.0
