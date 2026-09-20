# OMS DOJO V2_9 - DEVELOPER DOCUMENTATION

**Version:** V2_9  
**Date:** November 2025  
**Audience:** Developers, contributors, AI development assistants

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [LIBRARY Folder System](#library-folder-system)
3. [.oms File Format](#oms-file-format)
4. [Key Functions](#key-functions)
5. [State Management](#state-management)
6. [Keyboard Navigation](#keyboard-navigation)
7. [Development Guidelines](#development-guidelines)

---

## ARCHITECTURE OVERVIEW

### Core Systems

**1. Rendering System**
- WebGPU/WGSL fragment shaders
- 720p-2160p resolutions
- 24/30/60 FPS support
- Real-time preview + video export

**2. Audio System**
- Web Audio API
- FFT analysis (32768 samples, ~1.46Hz precision)
- User-definable frequency mapping
- 3 independent modulators (A/B/C)

**3. File System (NEW IN V2_9)**
- File System Access API
- LIBRARY folder for persistent storage
- .oms container format
- Auto-save workflow

**4. UI System**
- 4 tabs: Controls, Presets, Library, Code
- Keyboard-first navigation
- Dirty state tracking
- Active container indicators

---

## LIBRARY FOLDER SYSTEM

### File System Access API

**Permission Flow:**
```javascript
// One-time setup
async function setAudioLibrary() {
    audioLibraryHandle = await window.showDirectoryPicker();
    await storeLibraryHandle(audioLibraryHandle); // IndexedDB
}

// Persist permission
async function storeLibraryHandle(handle) {
    const db = await openDatabase();
    await db.put('settings', handle, 'audioLibraryHandle');
}

// Load on startup
async function loadLibraryHandle() {
    const db = await openDatabase();
    audioLibraryHandle = await db.get('settings', 'audioLibraryHandle');
}
```

**File Operations:**
```javascript
// Write .oms file
async function saveContainerToOMS(container) {
    const filename = container.name + '.oms';
    const fileHandle = await audioLibraryHandle.getFileHandle(filename, { create: true });
    const writable = await fileHandle.createWritable();
    await writable.write(JSON.stringify(container, null, 2));
    await writable.close();
}

// Read .oms file
async function loadOmsFile(filename) {
    const fileHandle = await audioLibraryHandle.getFileHandle(filename);
    const file = await fileHandle.getFile();
    const text = await file.text();
    return JSON.parse(text);
}

// Delete .oms file
async function deleteOmsFile(filename) {
    await audioLibraryHandle.removeEntry(filename);
}
```

### Two-Mode Operation

**Mode 1: LIBRARY Set (Persistent)**
- audioLibraryHandle stored in IndexedDB
- All saves go to LIBRARY folder as .oms files
- Survives browser sessions
- Professional workflow

**Mode 2: No LIBRARY (Temp Session)**
- audioLibraryHandle = null
- No saves possible
- Work + render only
- Close browser = lose everything

**Check Function:**
```javascript
async function checkLibrarySet(autoOpen = false) {
    if (!audioLibraryHandle) {
        showStatus('Set Library folder first to save', 'error');
        if (autoOpen) {
            await setAudioLibrary();
            return !!audioLibraryHandle;
        }
        return false;
    }
    return true;
}
```

---

## .OMS FILE FORMAT

### Container .oms Structure

```json
{
  "id": "1731234567890",
  "name": "MyShader",
  "shaderCode": "// Tron Tunnel v2.8.3c\n...",
  "controlStates": {
    "tempo": 120,
    "loopLength": 4,
    "modASubdiv": 1,
    "modAWaveform": "sine",
    "modAGain": 0.1,
    "modAFreq1": 30,
    "modAFreq2": 50,
    "modBSubdiv": 16,
    "modBWaveform": "sawtooth",
    "modBGain": 0.1,
    "modBFreq1": 50,
    "modBFreq2": 75,
    "modCSubdiv": 16,
    "modCWaveform": "triangle",
    "modCGain": 0.1,
    "modCFreq1": 100,
    "modCFreq2": 500,
    "renderDuration": "00:30.000",
    "orientation": "portrait",
    "resolution": "1080",
    "fps": "30",
    "audioPitch": 0
  },
  "audioFilename": "track01.mp3",
  "audioBPM": 128.0,
  "hasAudio": true,
  "created": "2025-11-11T20:55:54.123Z",
  "updated": "2025-11-11T21:30:12.456Z"
}
```

### Control State .oms Structure

```json
{
  "id": "1731234567890",
  "name": "QuickTest",
  "type": "controlState",
  "controlStates": {
    // Same 21 values as container
  },
  "created": "2025-11-11T20:55:54.123Z"
}
```

### Filename Convention

**Container:**
- Name: "MyShader"
- File: "MyShader.oms"
- 1:1 match (predictable)

**Control State:**
- Name: "QuickTest"
- File: "QuickTest_ControlState.oms"
- Suffix distinguishes from containers

**Audio:**
- Original filename preserved in .oms JSON
- Example: "track01.mp3", "AintNoTrick_WesSmith&BumprStickr_128BPM_2024.mp3"

---

## KEY FUNCTIONS

### Container Management

**Add Container:**
```javascript
async function confirmAddToLibrary() {
    // Check LIBRARY (auto-open picker if needed)
    if (!await checkLibrarySet(true)) return;
    
    // Get current editor state
    const shaderCode = document.getElementById('shaderCode').value;
    const states = getCurrentControlState();
    
    // Create container
    const container = {
        id: Date.now().toString(),
        name: suggestedName,
        shaderCode,
        controlStates: states,
        audioFilename: currentAudioFile?.name || null,
        audioBPM: audioLoaded ? audioTempo : null,
        hasAudio: !!currentAudioFile,
        created: new Date().toISOString()
    };
    
    // Save to LIBRARY
    await saveContainerToOMS(container);
    
    // Update UI
    refreshLibraryList();
    isDirty = false;
}
```

**Load Container:**
```javascript
async function loadContainer(containerId) {
    // Clear existing audio
    removeAudio();
    currentAudioFile = null;
    
    // Get container from localStorage
    const containers = getLibraryContainers();
    const container = containers.find(c => c.id === containerId);
    
    // Read .oms file if LIBRARY set
    let containerData = container;
    if (audioLibraryHandle) {
        const omsFilename = container.name + '.oms';
        try {
            containerData = await loadOmsFile(omsFilename);
        } catch (error) {
            console.log('Using localStorage data (no .oms file)');
        }
    }
    
    // Load shader code
    document.getElementById('shaderCode').value = containerData.shaderCode;
    
    // Apply control states
    try {
        applyControlState(containerData.controlStates);
    } catch (error) {
        console.error('Error applying control states:', error);
    }
    
    // Load audio from LIBRARY
    if (containerData.audioFilename && audioLibraryHandle) {
        await getAudioFromLibrary(containerData.audioFilename);
    }
    
    // Compile shader
    await compileShader();
    
    // Update tracking
    currentLoadedContainerId = containerId;
    isDirty = false;
    
    // Refresh UI
    refreshLibraryList();
    
    // Re-focus card
    setTimeout(() => {
        const card = document.querySelector(`.library-card[data-container-id="${containerId}"]`);
        if (card) card.focus();
    }, 100);
}
```

**Update Container:**
```javascript
async function updateContainer(containerId) {
    const containers = getLibraryContainers();
    const containerIndex = containers.findIndex(c => c.id === containerId);
    const existingContainer = containers[containerIndex];
    
    // Get current editor state
    const shaderCode = document.getElementById('shaderCode').value;
    const states = getCurrentControlState();
    
    // Determine audio info
    let audioFilename = null;
    let audioBPM = null;
    
    if (audioLoaded && currentAudioFile) {
        // Audio in editor → use it
        audioFilename = currentAudioFile.name;
        audioBPM = audioTempo;
    } else if (existingContainer.audioFilename) {
        // No audio in editor, preserve existing
        audioFilename = existingContainer.audioFilename;
        audioBPM = existingContainer.audioBPM;
    }
    
    // Update container
    const updatedContainer = {
        ...existingContainer,
        shaderCode,
        controlStates: states,
        audioFilename,
        audioBPM,
        hasAudio: !!audioFilename,
        updated: new Date().toISOString()
    };
    
    // Save to localStorage + LIBRARY
    containers[containerIndex] = updatedContainer;
    saveLibraryContainers(containers);
    await saveContainerToOMS(updatedContainer);
    
    // Update tracking
    currentLoadedContainerId = containerId;
    isDirty = false;
    
    // Refresh UI + re-focus
    refreshLibraryList();
    setTimeout(() => {
        const card = document.querySelector(`.library-card[data-container-id="${containerId}"]`);
        if (card) card.focus();
    }, 100);
}
```

**Rename Container:**
```javascript
async function renameContainer(containerId, newName) {
    const containers = getLibraryContainers();
    const containerIndex = containers.findIndex(c => c.id === containerId);
    const oldName = containers[containerIndex].name;
    
    // Delete old .oms file
    if (audioLibraryHandle) {
        try {
            await audioLibraryHandle.removeEntry(oldName + '.oms');
        } catch (error) {
            console.log('Old .oms file not found');
        }
    }
    
    // Update name
    containers[containerIndex].name = newName;
    saveLibraryContainers(containers);
    
    // Save new .oms file
    await saveContainerToOMS(containers[containerIndex]);
    
    refreshLibraryList();
}
```

**Remove Container:**
```javascript
async function removeContainer(containerId) {
    if (!confirm('Remove this container?')) return;
    
    const containers = getLibraryContainers();
    const container = containers.find(c => c.id === containerId);
    
    // Delete .oms file
    if (audioLibraryHandle) {
        try {
            await audioLibraryHandle.removeEntry(container.name + '.oms');
        } catch (error) {
            console.error('Error deleting .oms file:', error);
        }
    }
    
    // Remove from localStorage
    const filtered = containers.filter(c => c.id !== containerId);
    saveLibraryContainers(filtered);
    
    // Clear active if deleting active container
    if (currentLoadedContainerId === containerId) {
        currentLoadedContainerId = null;
    }
    
    refreshLibraryList();
}
```

---

## STATE MANAGEMENT

### Control States (21 Values)

```javascript
function getCurrentControlState() {
    return {
        tempo: parseFloat(document.getElementById('shaderTempo').value),
        loopLength: parseFloat(document.getElementById('loopLength').value),
        modASubdiv: parseFloat(document.getElementById('modASubdiv').value),
        modAWaveform: document.getElementById('modAWaveform').value,
        modAGain: parseFloat(document.getElementById('modAGain').value),
        modAFreq1: parseFloat(document.getElementById('modAFreq1').value),
        modAFreq2: parseFloat(document.getElementById('modAFreq2').value),
        modBSubdiv: parseFloat(document.getElementById('modBSubdiv').value),
        modBWaveform: document.getElementById('modBWaveform').value,
        modBGain: parseFloat(document.getElementById('modBGain').value),
        modBFreq1: parseFloat(document.getElementById('modBFreq1').value),
        modBFreq2: parseFloat(document.getElementById('modBFreq2').value),
        modCSubdiv: parseFloat(document.getElementById('modCSubdiv').value),
        modCWaveform: document.getElementById('modCWaveform').value,
        modCGain: parseFloat(document.getElementById('modCGain').value),
        modCFreq1: parseFloat(document.getElementById('modCFreq1').value),
        modCFreq2: parseFloat(document.getElementById('modCFreq2').value),
        renderDuration: document.getElementById('renderDuration').value,
        orientation: document.getElementById('orientation').value,
        resolution: document.getElementById('resolution').value,
        fps: document.getElementById('fps').value,
        audioPitch: audioLoaded ? audioPitch : 0
    };
}
```

### Dirty State Tracking

```javascript
let isDirty = false; // Tracks unsaved changes

// Track shader code changes
document.getElementById('shaderCode').addEventListener('input', () => {
    isDirty = true;
});

// Track control changes
function markAsCustomState() {
    isDirty = true;
    // ... update UI
}

// Clear on save/load
function clearDirty() {
    isDirty = false;
}

// Warn on load
function warnIfDirty() {
    if (isDirty) {
        showStatus('⚠ Unsaved changes - loading anyway', 'error');
    }
}
```

---

## KEYBOARD NAVIGATION

### Auto-Focus on Tab Switch

```javascript
function showTab(tabName) {
    // ... tab switching logic
    
    if (tabName === 'library') {
        // Auto-focus first card
        setTimeout(() => {
            const firstCard = document.querySelector('.library-card');
            if (firstCard) firstCard.focus();
        }, 50);
    }
}
```

### Card Navigation

```javascript
card.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowDown' || e.key === 'ArrowUp') {
        e.preventDefault();
        const cards = Array.from(document.querySelectorAll('.library-card'));
        const currentIndex = cards.indexOf(e.target);
        
        if (e.key === 'ArrowDown' && currentIndex < cards.length - 1) {
            cards[currentIndex + 1].focus();
            cards[currentIndex + 1].scrollIntoView({ behavior: 'smooth', block: 'nearest' });
        } else if (e.key === 'ArrowUp' && currentIndex > 0) {
            cards[currentIndex - 1].focus();
            cards[currentIndex - 1].scrollIntoView({ behavior: 'smooth', block: 'nearest' });
        }
    } else if (e.key === 'Enter' && e.target === card) {
        const containerId = card.dataset.containerId;
        if (containerId) loadContainer(containerId);
    }
});

// Button Enter key handling
const buttons = card.querySelectorAll('button.btn');
buttons.forEach(button => {
    button.addEventListener('keydown', (e) => {
        if (e.key === 'Enter') {
            e.preventDefault();
            button.click();
        }
    });
});
```

---

## DEVELOPMENT GUIDELINES

### Version Numbering

**Production Release:**
- Filename: `OMS_Dojo_V2.9.0.html` (periods, no letter)
- Title: `V2_9` (lowercase v, no letter)
- Header: `[V2_9]` (periods, brackets, no letter)

**Development Build:**
- Filename: `OMS_Dojo_V2_9_A.html` (underscores, letter suffix)
- Title: `v2_9_0a` (lowercase, letter suffix)
- Header: `[V2_9_0_A]` (underscores, letter suffix)

### Iteration Process

1. **One feature per iteration** (surgical changes)
2. **Alphabetical naming** (A, B, C... Z, AA, AB, AC...)
3. **Comprehensive notes** for each iteration
4. **Test thoroughly** before proceeding
5. **Never skip letters**

### File Management

**Single-File Philosophy:**
- All functionality in one HTML file
- No external dependencies
- Portable, offline-capable
- ~272KB total size

**Deliverables Package (9 files):**
1. Main HTML application
2. Deliverables inventory
3. Complete changelog
4. Developer documentation
5. Release notes
6. Claude shader guide
7. User guide
8. FFmpeg video guide
9. Project instructions

### Browser Compatibility

**Full Support:**
- Chrome 88+ (recommended)
- Safari 15.2+
- Edge 88+

**Partial Support:**
- Firefox (WebGPU only, no File System Access API)

**Required APIs:**
- WebGPU (shader rendering)
- Web Audio API (FFT analysis)
- File System Access API (LIBRARY folder)
- IndexedDB (permission storage)

---

## FUTURE DEVELOPMENT (V2.10.0+)

### LIVE Tab Integration

**Planned Features:**
- 5th LIVE tab
- Performance grid (4 tracks × 8 scenes)
- Mixer section (4 tracks + master)
- Real-time shader rendering (4 simultaneous)
- Real-time audio mixing
- Visual compositing

**Architecture:**
- Uses existing audio engine
- Uses existing LIBRARY system
- Shares .oms files
- 4 WebGPU render pipelines
- 4 Web Audio source nodes

### Roadmap

**V2.10.0:** LIVE tab UI shell + LIBRARY integration  
**V2.11.0:** Audio routing + shader rendering  
**V2.12.0:** Mixer controls + scene launching  
**V3.0.0:** MIDI control + advanced features

---

**END OF DEVELOPER DOCUMENTATION**
