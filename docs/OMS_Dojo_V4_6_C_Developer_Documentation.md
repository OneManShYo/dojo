# ONEMANSHYO BETA V4_6_0 - DEVELOPER DOCUMENTATION

**Version:** Beta V4_6_0  
**Date:** December 10, 2025  
**Purpose:** Technical implementation reference for V4_6_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [New Functions](#new-functions)
3. [Modified Functions](#modified-functions)
4. [Data Structures](#data-structures)
5. [Pending Flags Pattern](#pending-flags-pattern)
6. [UI Changes](#ui-changes)
7. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### Key Principle: Unified Container Building

V4_6_0 consolidates three different container creation paths into one:

**Before (V4_5_0):**
```
saveToLibrary() ────────┐
confirmAddToLibrary() ──┼──► Different field sets, inconsistent containers
handleLibraryAudioDrop()┘
```

**After (V4_6_0):**
```
saveToLibrary() ────────┐
confirmAddToLibrary() ──┼──► buildContainerObject() ──► Consistent containers
handleLibraryAudioDrop()┘
```

### Key Principle: Source of Truth Hierarchy

```
Container (.oms)     ← King (never override saved values)
    ↓ (if missing)
Project (.omsp)      ← Fallback (defaults for fresh audio)
    ↓ (if missing)
Hardcoded defaults   ← Last resort (1.1 → 17.1)
```

### Key Principle: UI Field as Authority

```
Input Sources:
  - Filename parsing (current)
  - Tempo detection (future)
  - Manual user entry
        ↓
  audioTempoField (UI)  ← Single source of truth
        ↓
  buildContainerObject() reads from field
        ↓
  container.audioTempo saved to .oms
```

---

## NEW FUNCTIONS

### buildContainerObject(containerName, audioFilenameOverride)

**Purpose:** Create complete container object with all fields.

**Location:** ~Line 7080

**Parameters:**
- `containerName` (string): Name for the container
- `audioFilenameOverride` (string, optional): Override audio filename (for drag-drop)

**Returns:** Complete container object

**Implementation:**
```javascript
function buildContainerObject(containerName, audioFilenameOverride = null) {
    const audioFilename = audioFilenameOverride || 
        (currentAudioFile ? currentAudioFile.name : null);
    
    // Get tempo from UI field, fallback to filename parsing
    let containerAudioTempo = null;
    if (audioFilename) {
        const tempoFieldValue = parseFloat(
            document.getElementById('audioTempoField').value
        );
        if (!isNaN(tempoFieldValue) && tempoFieldValue > 0) {
            containerAudioTempo = tempoFieldValue;
        }
        if (!containerAudioTempo) {
            containerAudioTempo = parseBPMFromFilename(audioFilename);
        }
    }
    
    // Loop points with validation
    let loopIn = 0;   // 1.1
    let loopOut = 64; // 17.1
    let loopEnabled = false;
    
    if (audioLoaded) {
        const uiLoopIn = barBeatToBeats(
            document.getElementById('audioLoopInField').value
        );
        const uiLoopOut = barBeatToBeats(
            document.getElementById('audioLoopOutField').value
        );
        if (uiLoopOut > uiLoopIn) {
            loopIn = uiLoopIn;
            loopOut = uiLoopOut;
            loopEnabled = audioLoopEnabled;
        }
    }
    
    return {
        id: Date.now(),
        name: containerName,
        shaderCode: document.getElementById('shaderCode').value,
        controlStates: getCurrentControlState(),
        audioFilename: audioFilename,
        audioTempo: containerAudioTempo,
        audioPitch: audioPitch,
        audioStartTime: audioStartTime,
        audioEndTime: audioEndTime,
        audioSnapLastBar: audioSnapLastBar,
        audioLoopIn: loopIn,
        audioLoopOut: loopOut,
        audioLoopEnabled: loopEnabled,
        hasAudio: !!audioFilename,
        shaderModATimeline: null,
        shaderModBTimeline: null,
        shaderModCTimeline: null,
        created: new Date().toISOString()
    };
}
```

---

### cleanAudioName(filename)

**Purpose:** Strip non-title elements from audio filenames.

**Location:** ~Line 10761

**Parameters:**
- `filename` (string): Audio filename with extension

**Returns:** Cleaned name string

**Patterns Removed:**
- Extensions: .mp3, .wav, .ogg, .m4a, .aac, .flac
- BPM patterns: "128BPM", "_128_bpm", trailing "_128"
- Suffixes: "_Master", "_Final", "_v2", "_mix"
- Underscores/dashes → spaces

**Examples:**
```javascript
cleanAudioName("BassTrack_128BPM_Final.mp3")  // → "BassTrack"
cleanAudioName("My_Song_140_Master.wav")      // → "My Song"
cleanAudioName("Deep_House_Mix_v2_124.mp3")   // → "Deep House Mix"
```

---

### showCreateAudioSwapDropdown()

**Purpose:** Populate and show audio picker dropdown in CREATE tab.

**Location:** ~Line 8248

**Behavior:**
1. Check if folder is set (show overlay if not)
2. Set pending flag if no folder
3. Scan Library folder for audio files
4. Group: "Unused" (orphan) first, then "In Use"
5. Show dropdown (absolute positioned)

**Audio File Detection:**
```javascript
const audioExtensions = ['.mp3', '.wav', '.ogg', '.m4a', '.aac', '.flac'];
```

---

### onCreateAudioSwapSelect()

**Purpose:** Handle audio selection from CREATE tab dropdown.

**Location:** ~Line 8346

**Behavior:**
1. Get selected filename
2. Hide dropdown
3. Load file from Library folder
4. Trigger loadAudioFile() with file

---

### populateShaderNameSelect()

**Purpose:** Fill shader name dropdown with presets and library containers.

**Location:** ~Line 9680

**Structure:**
```
_current     → Current shader (first option)
── Presets ──
sysPreset:0  → System preset (omsPresets array)
sysPreset:1
userPreset:x → User preset (userPresetsCache)
── Library ──
_setfolder   → "Select Folder..." (if no folder)
library:id   → Container (if folder set)
```

---

### onShaderNameSelect()

**Purpose:** Handle shader selection from dropdown.

**Location:** ~Line 9750

**Routing:**
```javascript
if (value === '_current') return;
if (value === '_setfolder') { /* show folder overlay */ }
if (value.startsWith('sysPreset:')) { /* load from omsPresets */ }
if (value.startsWith('userPreset:')) { /* load from userPresetsCache */ }
if (value.startsWith('library:')) { /* load from libraryContainersCache */ }
```

---

### toggleLibraryView()

**Purpose:** Single button to toggle between list and grid views.

**Location:** ~Line 8150

**Behavior:**
- Toggle `currentLibraryView` between 'list' and 'grid'
- Update button icon (≡ or ▦)
- Update tooltip
- Call `setLibraryView()`

---

## MODIFIED FUNCTIONS

### saveToLibrary()

**Changes:**
- Now calls `buildContainerObject()` instead of inline object creation
- Reads filename from field directly
- Sets `pendingSaveAfterFolderSetup` flag

---

### loadAudioFile()

**Changes:**
- Sets loop defaults from `projectLoopDefaults` at start
- Prepends cleaned audio name to filename field
- Enables waveformRemoveBtn

---

### compileShader()

**Changes:**
- Updates shader name dropdown first option
- Sets filename field with shader name
- Prepends cleaned audio name if audio loaded

---

### getCurrentProjectState()

**New Fields:**
```javascript
orientation: document.getElementById('orientation')?.value || 'portrait',
defaultLoopIn: document.getElementById('audioLoopInField')?.value || '1.1',
defaultLoopOut: document.getElementById('audioLoopOutField')?.value || '17.1'
```

---

### applyProjectState()

**New Handling:**
```javascript
// Orientation
if (projectData.orientation) {
    document.getElementById('orientation').value = projectData.orientation;
    updateCanvasSize();
}

// Loop defaults (store, don't apply to UI)
if (projectData.defaultLoopIn) {
    projectLoopDefaults.loopIn = projectData.defaultLoopIn;
}
if (projectData.defaultLoopOut) {
    projectLoopDefaults.loopOut = projectData.defaultLoopOut;
}
```

---

### setupApplicationFolder()

**New Handling:**
```javascript
// Resume pending save
if (pendingSaveAfterFolderSetup) {
    pendingSaveAfterFolderSetup = false;
    await saveToLibrary();
}

// Resume pending audio swap
if (pendingAudioSwapAfterFolderSetup) {
    pendingAudioSwapAfterFolderSetup = false;
    await showCreateAudioSwapDropdown();
}
```

---

## DATA STRUCTURES

### projectLoopDefaults

**Purpose:** Store project-level loop defaults.

**Location:** ~Line 3642

```javascript
let projectLoopDefaults = {
    loopIn: '1.1',
    loopOut: '17.1'
};
```

### Container Object (Complete)

```javascript
{
    // Identity
    id: 1702234567890,              // timestamp
    name: "BassTrack TronTunnel",   // display name
    created: "2025-12-10T...",      // ISO timestamp
    
    // Shader
    shaderCode: "/* WGSL */",       // fragment shader
    controlStates: {...},           // UI control values
    
    // Audio Reference
    audioFilename: "bass.mp3",      // filename only, not path
    audioTempo: 128,                // BPM (was audioBPM)
    audioPitch: 0,                  // pitch shift %
    hasAudio: true,                 // boolean flag
    
    // Audio Timing
    audioStartTime: 0,              // seconds
    audioEndTime: 300,              // seconds
    audioSnapLastBar: 146,          // last snapped bar
    
    // Loop Points
    audioLoopIn: 0,                 // beats (0 = 1.1)
    audioLoopOut: 64,               // beats (64 = 17.1)
    audioLoopEnabled: false,        // loop toggle
    
    // Timelines (filled by analyzeAudioTimeline)
    shaderModATimeline: [...],      // FFT analysis arrays
    shaderModBTimeline: [...],
    shaderModCTimeline: [...]
}
```

---

## PENDING FLAGS PATTERN

### Purpose
Resume operations after folder setup without requiring user to click again.

### Implementation

**Declaration:**
```javascript
let pendingSaveAfterFolderSetup = false;
let pendingAudioSwapAfterFolderSetup = false;
```

**Setting Flag:**
```javascript
// In operation that needs folder
if (!applicationFolderHandle) {
    pendingSaveAfterFolderSetup = true;  // Set flag
    document.getElementById('appFolderOverlay').classList.add('active');
    return;
}
```

**Checking Flag:**
```javascript
// In setupApplicationFolder() after success
if (pendingSaveAfterFolderSetup) {
    pendingSaveAfterFolderSetup = false;  // Clear flag
    await saveToLibrary();                // Resume operation
}
```

---

## UI CHANGES

### CREATE Tab - SHADER CONTROLS Header

```html
<div style="display: flex; justify-content: space-between; ...">
    <div class="shader-header">SHADER CONTROLS</div>
    <div style="display: flex; gap: 8px;">
        <button onclick="toggleCodeView()">Code</button>
        <button onclick="saveToLibrary()">Save</button>
    </div>
</div>
```

### CREATE Tab - Name/Orientation Section

```html
<!-- Name -->
<label>Name</label>
<select id="shaderNameSelect" onclick="populateShaderNameSelect()" 
        onchange="onShaderNameSelect()">
    <option value="_current">Untitled</option>
</select>

<!-- Orientation -->
<label>Orientation</label>
<select id="orientation" onchange="updateCanvasSize(); autoSaveProject();">
    <option value="landscape">Landscape</option>
    <option value="portrait" selected>Portrait</option>
    <option value="square">Square</option>
</select>
```

### CREATE Tab - AUDIO CONTROLS Header

```html
<div style="display: flex; justify-content: space-between; position: relative;">
    <div class="shader-header">AUDIO CONTROLS</div>
    <select id="createAudioSwapSelect" style="display: none; position: absolute; 
            right: 100px; ..." onchange="onCreateAudioSwapSelect()">
    </select>
    <div style="display: flex; gap: 8px;">
        <button onclick="showCreateAudioSwapDropdown()">Swap</button>
        <button onclick="removeAudio()" id="waveformRemoveBtn" disabled>X</button>
    </div>
</div>
```

---

## TESTING NOTES

### Save Workflow Tests

1. **Fresh save (no folder):** Click Save → folder overlay → set folder → auto-saves
2. **Fresh save (folder set):** Click Save → immediate save → ORGANIZE tab
3. **Drag-drop audio:** Drop on ORGANIZE → container created with BPM

### Tempo Tests

1. **Filename parsing:** "track_128BPM.mp3" → 128 in tempo field
2. **UI override:** Edit tempo field → saved value used
3. **Swap update:** Select audio from dropdown → tempo updates immediately

### Loop Defaults Tests

1. **Fresh audio:** Load audio → uses project defaults
2. **Container load:** Load container → uses container's loop points
3. **Change defaults:** Edit loop → saves to .omsp → persists on reload

### Shader Dropdown Tests

1. **System presets:** Select → loads and compiles
2. **User presets:** Select → loads and compiles
3. **Library containers:** Select → loads shader code only
4. **No folder:** "Select Folder..." → shows overlay

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V4_6_0
Date: December 10, 2025
License: GPL-3.0
