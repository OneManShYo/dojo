# ONEMANSHYO BETA V4_4_0 - DEVELOPER DOCUMENTATION

**Version:** Beta V4_4_0  
**Date:** December 6, 2025  
**Purpose:** Technical reference for developers

---

## ARCHITECTURE OVERVIEW

**File:** OMS_Beta_V4_4_0.html  
**Size:** ~700KB  
**Lines:** ~16,000  
**Dependencies:** Zero (single-file application)

---

## LEAF NODE HIERARCHY

### CORTEX (Sacred - Never Touch)

Change = cascade failure. Everything traces back to these.

**File System Handles:**
```javascript
applicationFolderHandle   // Root app folder (set by user)
libraryFolderHandle       // /Library/ subfolder handle
```

**Master Clock / Seconds:**
```javascript
audioContext              // Web Audio API instance
audioContext.currentTime  // Browser clock (seconds)
trackMasterClockStart     // Session start timestamp
```

**Master Clock / Musical Time:**
```javascript
gridState.masterTempo     // BPM value
trackAudioMasterClock()   // Seconds → beats conversion
masterBeatNow()           // Current beat
masterBarNow()            // Current bar
```

**Pixel Formula:**
```javascript
BASE_PIXELS_PER_SECOND = 6.25
getPixelsPerBeat() = BASE_PIXELS_PER_SECOND × (60 / masterTempo) / zoom
getPixelsPerBar() = getPixelsPerBeat() × 4
```

**Engines:**
```javascript
WebGPU device/context
.oms container schema
```

### NODES (Trace Up/Down Before Touching)

Other things depend on these. Trace dependencies before modifying.

**Library System:**
```javascript
libraryContainersCache    // Full container data from folder
librarySortBy             // 'name', 'bpm', or 'audio'
librarySortDir            // 'asc' or 'desc'
selectedLibrarySetId      // 'all' or 'presets'
selectedLiveSetId         // Synced with Library selection
sanitizeFilename()        // Filesystem-safe conversion
```

**Track System:**
```javascript
trackBar1Positions[]      // Per-track bar1X storage
trackAudioPlaying[]       // Playback state
trackShaderOpacity[]      // Visual opacity
trackContainerData[]      // Loaded container data
```

**Audio System:**
```javascript
audioLoopEnabled          // Loop on/off state
audioStartTime            // Green start marker position
audioEndTime              // Green end marker position
audioLoopIn / audioLoopOut // Cyan loop markers (bar.beat)
playbackRate calculation
GainNode routing
Quantization math
```

### LEAFS (Safe to Modify)

End of chain. Nothing depends on these.

**Library UI:**
```javascript
Sort arrow indicators (▼/▲)
Library card styling
Column header text/colors
Button hover states
Thumbnail canvas elements
```

**Visual:**
```javascript
masterGridVisible, trackGridVisible, waveformVisible
Waveform colors
Grid line colors
Playhead color (magenta #ff00ff)
Loop bracket color (cyan #00ffff)
Audio boundary color (green #7fff00)
```

**Controls:**
```javascript
Knob/button styling
Keyboard shortcut mappings
Header text
Tab labels and colors
```

---

## LIBRARY TWO-PANEL ARCHITECTURE

### Panel Structure

**Library Tab Layout:**
```
+------------------+------------------------------------------------+
|   LEFT (400px)   |              RIGHT (920px)                     |
+------------------+------------------------------------------------+
|   librarySets    |   libraryContainersRightPanel                  |
|   TabContent     |   (CONTAINERS + Audio Controls)                |
+------------------+------------------------------------------------+
```

**HTML Elements:**
```javascript
// Left panel (Library tab only)
librarySetsTabContent     // Sets list + shader preview

// Right panel (Library tab only)
libraryContainersRightPanel // Containers + waveform + audio strip

// Left panel (LIVE tab only)
libraryTabContent         // Container list with SET dropdown
```

### switchTab() Visibility Logic

```javascript
case 'library':
    librarySetsTabContent.classList.add('active');
    libraryContainersRightPanel.style.display = 'block';
    previewContainer.style.display = 'none';
    audioWaveform.style.display = 'block';
    audioControlsStrip.style.display = 'flex';
    break;

case 'live':
    libraryTabContent.classList.add('active');
    liveMixerContainer.style.display = 'grid';
    libraryContainersRightPanel.style.display = 'none';
    break;

default: // Controls, Presets, WGSL, etc.
    previewContainer.style.display = 'flex';
    libraryContainersRightPanel.style.display = 'none';
    break;
```

---

## SETS SYSTEM

### State Variables

```javascript
let selectedLibrarySetId = 'presets';  // 'all' or 'presets'
let selectedLiveSetId = 'presets';     // Synced with above
```

### Set Filtering Logic

```javascript
async function refreshLibraryList() {
    if (selectedLibrarySetId === 'presets') {
        // Build from embedded omsPresets array
        libraryContainersCache = omsPresets.map((preset, index) => ({
            id: `preset_${index}`,
            name: preset.name,
            shaderCode: preset.code,
            hasAudio: false,
            isPreset: true
        }));
    } else {
        // Scan Library/ folder for .oms files
        libraryContainersCache = await scanLibraryFolder();
    }
    
    // Sort by current settings
    sortContainersCache();
    renderLibraryList();
}
```

### Selection Function

```javascript
async function selectLibrarySet(setId) {
    selectedLibrarySetId = setId;
    selectedLiveSetId = setId;
    
    // Update UI
    updateSetCardHighlights();
    updateDropdownValue();
    
    // Refresh both panels
    await refreshLibraryList();
    await refreshLibraryListRight();
    
    autoSaveProject();
}
```

### Persistence

```javascript
// In getCurrentProjectState()
selectedSetId: selectedLibrarySetId

// In applyProjectState()
if (projectData.selectedSetId) {
    await selectLibrarySet(projectData.selectedSetId);
}
```

---

## UNIFIED DRAG/DROP SYSTEM

### Handler Registration

```javascript
const appContainer = document.querySelector('.container');

['dragenter', 'dragover', 'dragleave', 'drop'].forEach(eventName => {
    appContainer.addEventListener(eventName, preventDefaults, false);
});

appContainer.addEventListener('dragenter', showDropOverlay);
appContainer.addEventListener('dragleave', hideDropOverlay);
appContainer.addEventListener('drop', handleUnifiedDrop);
```

### External File Detection

```javascript
function isExternalFileDrop(e) {
    return e.dataTransfer && 
           e.dataTransfer.types && 
           e.dataTransfer.types.includes('Files');
}
```

**Critical:** Without this check, internal card drags (to LIVE grid) would trigger the overlay.

### Routing Logic

```javascript
async function handleUnifiedDrop(e) {
    const file = e.dataTransfer.files[0];
    const isLibraryTab = document.getElementById('librarySetsTabContent')
                                 .classList.contains('active');
    
    if (isAudioFile(file)) {
        if (isLibraryTab) {
            await handleLibraryAudioDrop(file);  // Create container
        } else {
            await loadAudioFile(file);           // Just load audio
        }
    } 
    else if (isShaderFile(file)) {
        await loadShaderFile(file);              // Load + compile
    }
    else if (isZipFile(file)) {
        if (isLibraryTab) {
            await handleZipImport(file);         // Import container
        } else {
            showStatus('Switch to Library tab to import .zip', 'error');
        }
    }
}
```

### File Type Detection

```javascript
function isAudioFile(file) {
    return /\.(mp3|wav|ogg|m4a|aac|flac)$/i.test(file.name);
}

function isShaderFile(file) {
    return /\.(txt|wgsl)$/i.test(file.name);
}

function isZipFile(file) {
    return /\.zip$/i.test(file.name);
}
```

---

## SHADER THUMBNAILS

### Rendering Flow

```javascript
async function renderLibraryThumbnails(containers) {
    for (let i = 0; i < containers.length; i++) {
        const container = containers[i];
        const canvas = document.querySelector(
            `.library-thumb[data-container-id="${container.id}"]`
        );
        
        if (canvas && container.shaderCode) {
            try {
                await initThumbnailShader(canvas, container.shaderCode, i);
            } catch (e) {
                // Fallback: gray placeholder
                const ctx = canvas.getContext('2d');
                ctx.fillStyle = '#333';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
            }
        }
        
        // Yield every 5 to avoid UI blocking
        if (i % 5 === 0) {
            await new Promise(r => setTimeout(r, 0));
        }
    }
}
```

### Canvas HTML

```html
<canvas class="library-thumb" 
        data-container-id="${id}" 
        width="32" 
        height="18" 
        style="width: 32px; height: 18px; margin-right: 8px;">
</canvas>
```

---

## SHADER-ONLY CONTAINER LAUNCH

### Detection Change

**Before (broken for shader-only):**
```javascript
const isPlaying = trackAudioSources[track] !== null;
```

**After (works for all):**
```javascript
const playingCell = document.querySelector(
    `.clip-slot[data-track="${track}"].playing`
);
const isPlaying = playingCell !== null;
```

### Launch Function

```javascript
async function launchCell(track, sceneIndex) {
    const cell = getCellElement(track, sceneIndex);
    const containerData = trackContainerData[track];
    const hasAudio = containerData?.audioFilename;
    
    // Visual state (all containers)
    cell.classList.add('playing');
    updateWaveformOverlay(track);
    
    // Audio setup (only if has audio)
    if (hasAudio) {
        await setupAudioPlayback(track, containerData);
        showStatus(`Playing: ${containerData.audioFilename}`);
    } else {
        showStatus(`Launched: ${containerData.name} (shader only)`);
    }
    
    // Shader always launches
    loadTrackShader(track, containerData.shaderCode);
}
```

### Compositor Opacity

```javascript
function calculateTrackShaderOpacity(track) {
    const playingCell = document.querySelector(
        `.clip-slot[data-track="${track}"].playing`
    );
    const isPlaying = playingCell !== null;
    
    if (!isPlaying) return 0;
    
    // Shader-only fallback
    if (!trackAudioSources[track]) {
        return trackShaderOpacity[track] || 1.0;
    }
    
    // Normal audio-reactive calculation
    return calculateAudioReactiveOpacity(track);
}
```

---

## AUDIO WORKFLOW SYSTEM

### Boundary Pinning

```javascript
function setAudioStart(event) {
    if (event.shiftKey) {
        // Pin to current playhead
        audioStartTime = audioElement.currentTime;
    } else {
        // Reset to start
        audioStartTime = 0;
    }
    drawWaveform();
    showStatus(`Audio Start: ${formatTime(audioStartTime)}`);
}

function setAudioEnd(event) {
    if (event.shiftKey) {
        // Pin to current playhead
        audioEndTime = audioElement.currentTime;
    } else {
        // Reset to end
        audioEndTime = audioElement.duration;
    }
    drawWaveform();
    showStatus(`Audio End: ${formatTime(audioEndTime)}`);
}
```

### Loop State Persistence

```javascript
// In container save
{
    audioLoopEnabled: audioLoopEnabled,
    audioLoopIn: document.getElementById('audioLoopIn').value,
    audioLoopOut: document.getElementById('audioLoopOut').value
}

// In container load
audioLoopEnabled = containerData.audioLoopEnabled ?? true;
updateLoopButtonVisual();
if (!audioLoopEnabled) {
    audioElement.currentTime = audioStartTime;
}
```

### Waveform Color Constants

```javascript
const PLAYHEAD_COLOR = '#ff00ff';       // Magenta
const LOOP_BRACKET_COLOR = '#00ffff';   // Cyan
const LOOP_FILL_COLOR = 'rgba(0,255,255,0.25)';
const AUDIO_BOUNDARY_COLOR = '#7fff00'; // Green
```

---

## KEYBOARD SHORTCUTS

### Shift+Enter: Shader-Only Load

```javascript
// In library card keydown handler
if (e.key === 'Enter') {
    if (e.shiftKey) {
        loadShaderOnly(containerId);
    } else {
        loadContainer(containerId);
    }
}
```

```javascript
async function loadShaderOnly(containerId) {
    const container = getContainerById(containerId);
    if (!container) return;
    
    // Load ONLY shader code
    document.getElementById('shaderCode').value = container.shaderCode;
    
    // Compile without touching audio
    compileShader(false);
    
    showStatus(`Shader: ${container.name}`);
}
```

---

## TAB SYSTEM

### Order and Colors

```javascript
const TAB_CONFIG = {
    controls: { label: 'C', color: 'cyan', needsFolder: false },
    presets:  { label: 'P', color: 'cyan', needsFolder: false },
    code:     { label: 'W', color: 'cyan', needsFolder: false },
    library:  { label: 'L', color: 'magenta', needsFolder: true },
    live:     { label: 'LIVE', color: 'magenta', needsFolder: true },
    settings: { label: 'S', color: 'yellow', needsFolder: true },
    utilities:{ label: 'U', color: 'yellow', needsFolder: true },
    about:    { label: 'A', color: 'yellow', needsFolder: false }
};
```

### CSS Color Mapping

```css
.tab[data-color="cyan"]:hover { border-color: #00ffff; }
.tab[data-color="magenta"]:hover { border-color: #ff00ff; }
.tab[data-color="yellow"]:hover { border-color: #ffff00; }

.tab.active[data-color="cyan"] { border-color: #00ffff; }
.tab.active[data-color="magenta"] { border-color: #ff00ff; }
.tab.active[data-color="yellow"] { border-color: #ffff00; }
```

---

## USER PRESETS SYSTEM

### Folder Structure

```
[App Folder]/
  └── Presets/
      ├── My Shader.wgsl
      └── Cool Effect.wgsl
```

### Save Function

```javascript
async function saveShaderToPresets() {
    if (!applicationFolderHandle) {
        showFolderOverlay();
        return;
    }
    
    const name = prompt('Preset name:');
    if (!name) return;
    
    // Ensure Presets folder exists
    const presetsFolder = await applicationFolderHandle
        .getDirectoryHandle('Presets', { create: true });
    
    // Add orientation comment if missing
    let code = document.getElementById('shaderCode').value;
    if (!code.includes('// orientation:')) {
        code = `// orientation: ${currentOrientation}\n${code}`;
    }
    
    // Save file
    const filename = sanitizeFilename(name) + '.wgsl';
    const fileHandle = await presetsFolder.getFileHandle(filename, { create: true });
    const writable = await fileHandle.createWritable();
    await writable.write(code);
    await writable.close();
    
    await refreshUserPresets();
    showStatus(`Saved: ${name}`);
}
```

---

## PROJECT STATE (.omsp)

### Updated Structure

```javascript
{
    version: "V4_4_0",
    gridState: { ... },
    trackSettings: { ... },
    librarySort: {
        sortBy: "name",
        sortDir: "asc"
    },
    selectedSetId: "all",    // NEW in V4_4_0
    // ... other fields
}
```

---

## TESTING CHECKLIST

### Library Two-Panel
- [ ] Library tab shows two panels
- [ ] Left panel: Sets + shader preview
- [ ] Right panel: Containers + audio controls
- [ ] Switching tabs hides/shows correct panels

### Sets System
- [ ] "Presets" shows embedded presets
- [ ] "All" shows folder containers
- [ ] Selection persists across refresh
- [ ] LIVE dropdown syncs with Library

### Drag/Drop
- [ ] Audio on Library → creates container
- [ ] Audio on Controls → loads audio
- [ ] Shader file → loads code
- [ ] ZIP on Library → imports
- [ ] Internal drags don't trigger overlay

### Shader-Only Launch
- [ ] Shader-only container launches
- [ ] Shows in compositor
- [ ] Status shows "(shader only)"

### Audio Workflow
- [ ] Click Audio Start → resets to 0
- [ ] Shift+click → pins to playhead
- [ ] Loop OFF hides cyan brackets
- [ ] Arrow keys navigate full file

### Thumbnails
- [ ] Cards show shader previews
- [ ] No UI blocking during render

---

## DEBUGGING

### Console Commands

```javascript
// Check set state
console.log('Set:', selectedLibrarySetId, selectedLiveSetId);

// Check cache
console.log('Cache:', libraryContainersCache.length, 'items');
console.log('First:', libraryContainersCache[0]);

// Check playing state
document.querySelectorAll('.clip-slot.playing').forEach(c => 
    console.log('Playing:', c.dataset.track, c.dataset.scene)
);

// Force refresh
await refreshLibraryList();
await refreshLibraryListRight();
```

### Common Issues

**"Shader-only not showing in compositor":**
- Check .playing class on cell
- Verify calculateTrackShaderOpacity uses class check

**"Overlay appears on internal drag":**
- Check isExternalFileDrop() returns false
- Verify dataTransfer.types doesn't include 'Files'

**"Set not restoring":**
- Check .omsp has selectedSetId
- Verify no default assignment after restore

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V4_4_0
Date: December 6, 2025
License: GPL-3.0
