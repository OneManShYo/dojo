# ONEMANSHYO BETA V4_6_0 - COMPLETE CHANGELOG

**Version:** Beta V4_6_0  
**Release Date:** December 10, 2025  
**Base Version:** V4_5_0  
**Development Iterations:** 31 sub-iterations (V4_6_01 through V4_6_31)

---

## EXECUTIVE SUMMARY

V4_6_0 is a workflow consistency release focused on unifying save paths, standardizing tempo handling, and improving the CREATE tab experience with inline shader/audio selection. The key achievement is `buildContainerObject()` - a single function that ensures all containers are created with consistent, complete data regardless of the save path used.

**Key Achievement:** Predictable, consistent save behavior with auto-populated filenames and project-level defaults that respect existing container data.

---

## MAJOR FEATURES

### 1. Unified Container Builder (V4_6_08)

✅ **Single buildContainerObject() Function**
- Replaces three different container creation paths
- Ensures ALL fields populated consistently
- Smart loop handling (disables if range is zero)

✅ **Consistent Container Structure**
```javascript
{
    id, name, shaderCode, controlStates,
    audioFilename, audioTempo, audioPitch,
    audioStartTime, audioEndTime, audioSnapLastBar,
    audioLoopIn, audioLoopOut, audioLoopEnabled,
    hasAudio, shaderModATimeline, shaderModBTimeline,
    shaderModCTimeline, created
}
```

### 2. Tempo Standardization (V4_6_10-17)

✅ **Unified Naming**
- `audioBPM` → `audioTempo` throughout codebase
- Column headers: "BPM" → "Tempo"
- Info drawer label: "BPM" → "TEMPO"

✅ **Source of Truth**
- UI field `audioTempoField` is authoritative
- Fallback: parse from filename via `parseBPMFromFilename()`
- Immediate update on swap selection

### 3. Shader Name Dropdown (V4_6_18-23)

✅ **Inline Shader Selection**
- Single dropdown in CREATE tab
- Shows current shader name
- Click to see available options

✅ **Dropdown Structure**
```
CurrentShader          ← current (no action)
── Presets ──          ← system + user presets
FlowerPower
Neon Grid
── Library ──          ← containers with shaderCode
Select Folder...       ← if no folder set
Container 1
Container 2
```

✅ **Value Prefixes**
- `_current` - current shader
- `_setfolder` - trigger folder setup
- `sysPreset:index` - system preset
- `userPreset:name` - user preset
- `library:id` - library container

### 4. Audio Swap Controls (V4_6_28-30)

✅ **Header Integration**
- Swap/X buttons in AUDIO CONTROLS header
- Consistent with SHADER CONTROLS pattern
- Dropdown appears inline (absolute positioned)

✅ **Library Audio Picker**
- Groups: "Unused" (orphan files first), "In Use"
- Alphabetical within groups
- Auto-show after folder setup

✅ **Pending Flags**
- `pendingAudioSwapAfterFolderSetup`
- Resume operation after folder selection

### 5. Orientation & Filename (V4_6_24-26)

✅ **Orientation Relocated**
- Moved from OUTPUT to SHADER CONTROLS
- Between Name and Timing sections
- Auto-saves to .omsp on change

✅ **Filename Auto-Population**
- `cleanAudioName()` strips BPM/extensions/suffixes
- Format: `[CleanedAudioName] [ShaderName]`
- Used for both render and save

### 6. Loop Defaults (V4_6_31)

✅ **Project-Level Defaults**
- Saved to .omsp: `defaultLoopIn`, `defaultLoopOut`
- Applied on fresh audio load
- Container values always override (sacred)

✅ **Hierarchy**
1. Container (.oms) - king, never override
2. Project (.omsp) - fallback for fresh audio

### 7. Container Name Editing (V4_6_01)

✅ **Click-to-Edit**
- Info drawer: click name → edit → Enter saves
- Library cards: click name → edit → Enter saves
- Immediate filesystem save via `renameContainer()`

### 8. All Audio Set (V4_6_02)

✅ **New Filter Option**
- ORGANIZE tab: "All Audio" set card
- PERFORM tab: "All Audio" in SET dropdown
- Filters containers with `hasAudio || audioFilename`

---

## ITERATION HISTORY

### V4_6_01-05: ORGANIZE Tab Improvements

**V4_6_01:** Container name click-to-edit
- Info drawer and library card editing
- Immediate filesystem save

**V4_6_02:** "All Audio" system set
- Filter containers with audio
- Added to both ORGANIZE and PERFORM

**V4_6_03:** BPM column white
- Changed from grey (#666) to white (#fff)

**V4_6_04:** Tooltip fixes
- Show actual keyboard shortcuts
- Remove fake hotkey references

**V4_6_05:** View toggle consolidation
- Single button instead of two
- Icon shows current state

### V4_6_06-09: Save Workflow

**V4_6_06:** Save button in CREATE tab
- Added to SHADER CONTROLS header
- Orchestrates full save flow

**V4_6_07:** Remove Code view Save
- Single Save entry point
- Identified consolidation needs

**V4_6_08:** Unified container builder
- `buildContainerObject()` function
- Single source for all save paths

**V4_6_09:** BPM/Loop defaults fix
- Parse BPM from filename
- Default 1.1 → 17.1 (16 bars)

### V4_6_10-17: Tempo Standardization

**V4_6_10:** Loop math fix
- 0 beats = 1.1, 64 beats = 17.1

**V4_6_11:** Naming standardization
- `audioBPM` → `audioTempo`

**V4_6_12:** Resume save after folder
- `pendingSaveAfterFolderSetup` flag

**V4_6_13:** Tempo from UI field
- Single source of truth

**V4_6_14:** Filename parsing fallback
- Try UI field, then parse filename

**V4_6_15:** Tempo on swap
- Parse when swapping audio in info drawer

**V4_6_16:** Immediate tempo update
- Update displays on dropdown select

**V4_6_17:** Label alignment
- Match label widths to input widths

### V4_6_18-23: Shader Name Dropdown

**V4_6_18:** Name row with Swap button
- Initial implementation

**V4_6_19:** Single dropdown
- Consolidated field + button

**V4_6_20:** Library containers
- Show all containers with shaderCode

**V4_6_21:** Presets + Library sections
- Value prefixes for routing

**V4_6_22:** Standardize preset.shaderCode
- Consistent property naming

**V4_6_23:** System + User presets
- Combined in dropdown

### V4_6_24-31: Final Features

**V4_6_24:** Orientation relocation
- Move to SHADER CONTROLS
- Add to project state

**V4_6_25:** Filename auto-population
- `cleanAudioName()` function
- Auto-build working title

**V4_6_26:** Orientation auto-save
- onchange → autoSaveProject()

**V4_6_27:** SET dropdown alphabetical
- All, All Audio, Presets order

**V4_6_28:** Waveform Swap/X controls
- Initial placement in waveform area

**V4_6_29:** Audio controls header
- Moved to header (consistent pattern)

**V4_6_30:** Dropdown fixes
- Absolute positioning
- Auto-show after folder setup

**V4_6_31:** Loop defaults
- Project-level defaults in .omsp
- Container values sacred

---

## NEW FUNCTIONS

### Container Building

```javascript
buildContainerObject(containerName, audioFilenameOverride)
// Returns complete container with ALL fields
// Used by: saveToLibrary(), confirmAddToLibrary(), handleLibraryAudioDrop()
```

### Audio Handling

```javascript
cleanAudioName(filename)
// Strips: extensions, BPM patterns, _Master/_Final/_v2 suffixes
// Converts underscores/dashes to spaces

showCreateAudioSwapDropdown()
// Populates dropdown from Library folder
// Groups: Unused first, then In Use

onCreateAudioSwapSelect()
// Loads selected audio file immediately
```

### Shader Selection

```javascript
populateShaderNameSelect()
// Fills dropdown: current + presets + library
// Value prefixes for routing

onShaderNameSelect()
// Handles selection based on prefix
// Loads and compiles shader code
```

### View Controls

```javascript
toggleLibraryView()
// Single button toggles list/grid
// Updates icon and tooltip
```

---

## MODIFIED FUNCTIONS

### saveToLibrary()
- Now calls `buildContainerObject()`
- Reads filename from field directly
- Handles folder setup flow

### loadAudioFile()
- Sets loop defaults from project
- Prepends cleaned audio name to filename
- Enables Swap/X buttons

### compileShader()
- Updates shader name dropdown
- Sets filename field with shader name

### getCurrentProjectState()
- Added: orientation, defaultLoopIn, defaultLoopOut

### applyProjectState()
- Restores orientation, updates canvas
- Stores loop defaults (doesn't apply to UI)

---

## CSS CHANGES

### Audio Controls Header

```css
/* Swap dropdown absolute positioning */
#createAudioSwapSelect {
    position: absolute;
    right: 100px;
    top: 50%;
    transform: translateY(-50%);
    z-index: 10;
}
```

### Column Styling

```css
/* Tempo column white text */
.library-container-tempo {
    color: #fff;  /* was #666 */
}
```

---

## BUG FIXES

| Issue | Cause | Fix |
|-------|-------|-----|
| BPM not saved | audioLoaded false during build | Read from UI field, fallback to filename |
| Loop 1.1→1.1 | Wrong beat calculation | 0 beats = 1.1, fixed math |
| Tempo not updating | Only updated on OK | Immediate update on dropdown select |
| Header shift on dropdown | Inline flex item | Absolute positioning |
| Save requires folder twice | No resume logic | pendingSaveAfterFolderSetup flag |

---

## STATISTICS

- **Total Sub-iterations:** 31
- **Lines Changed:** ~1500+
- **Functions Added:** 6+
- **Functions Modified:** 8+
- **New .omsp Fields:** 3

---

## PRODUCTION READINESS

**Overall: ~95%**

**100% Complete:**
- Unified container builder
- Tempo standardization
- Shader name dropdown
- Audio swap controls
- Filename auto-population
- Loop defaults
- Container name editing

**Inherited from V4_5_0 (100%):**
- 4-tab workflow
- Code view toggle
- ADMIN consolidation
- Library toolbar
- Info drawer

**Remaining:**
- MIDI integration
- Advanced ADMIN settings
- Final polish

---

**END OF COMPLETE CHANGELOG**

Version: Beta V4_6_0
Date: December 10, 2025
Total Development: 31 sub-iterations
License: GPL-3.0
