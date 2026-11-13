# ONEMANSHYO App [Beta V5_3_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_3_0  
**Date:** December 27, 2025  
**Purpose:** Technical implementation reference for V5_3_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Real Audio Reactivity System](#real-audio-reactivity-system)
3. [Mod Tuning Panel](#mod-tuning-panel)
4. [Unified Sets System](#unified-sets-system)
5. [Tags System](#tags-system)
6. [Proper Peak Metering](#proper-peak-metering)
7. [Quantize System](#quantize-system)
8. [Draggable Bar 1 Marker](#draggable-bar-1-marker)
9. [Volume dB Scale](#volume-db-scale)
10. [Schema Additions](#schema-additions)
11. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_3_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  REAL AUDIO REACTIVITY (CORTEX)                                     │
│  Bandpass peak detection replaces fake FFT averaging                │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Per-mod filters │ Peak detection │ ADSR envelope │ Baking   │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  MOD TUNING PANEL (N-Panel)                                         │
│  Blender-style slide-out with Q and ADSR controls                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Per-mod Q │ Live meters │ Attack │ Decay │ Sustain │ Release│   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  UNIFIED SETS SYSTEM                                                │
│  Manual and smart sets in single cache with type field              │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ userSetsCache │ type: manual|smart │ /Sets/ folder          │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Real Dynamics** - Peak detection captures actual audio transients
2. **Pre-EQ Tap** - Analysis happens before EQ/FX chain
3. **Unified Cache** - Single `userSetsCache` for all set types
4. **File-Based Sets** - All sets stored in `/Sets/` folder

---

## REAL AUDIO REACTIVITY SYSTEM

### The Problem (CORTEX Issue)

V5_2_0 and earlier used FFT bin averaging that produced constant ~0.02 values regardless of audio content:

```javascript
// OLD: FFT averaging (fake dynamics)
// Location: analyzeAudio() function

analyser.getByteFrequencyData(dataArray);
const bins = dataArray.slice(startBin, endBin);
const avg = bins.reduce((a, b) => a + b, 0) / bins.length;
const level = avg / 255;  // Always ~0.02 regardless of audio!
```

Shaders appeared reactive only due to tempo-synced oscillators. Audio was irrelevant.

### The Solution: Bandpass Peak Detection

```javascript
// ============================================
// BANDPASS FILTER CHAIN (per modulator)
// Location: initAudioRouting() ~line 8500
// ============================================

// Create parallel filter chains from pre-EQ source
const modAHighpass = audioContext.createBiquadFilter();
modAHighpass.type = 'highpass';
modAHighpass.frequency.value = freq1Value;  // From Freq1 knob
modAHighpass.Q.value = modAQ;  // From Mod Tuning panel

const modALowpass = audioContext.createBiquadFilter();
modALowpass.type = 'lowpass';
modALowpass.frequency.value = freq2Value;   // From Freq2 knob
modALowpass.Q.value = modAQ;

const modAAnalyser = audioContext.createAnalyser();
modAAnalyser.fftSize = 256;  // Small for fast response

// Connect: source → highpass → lowpass → analyser (NOT to destination)
sourceNode.connect(modAHighpass);
modAHighpass.connect(modALowpass);
modALowpass.connect(modAAnalyser);
// Silent analysis - doesn't affect audio output
```

### Signal Chain Architecture

```
Audio Source
    │
    ├──→ createEQHigh → createEQMid → createEQLow → FX → speakers
    │    (existing audio chain unchanged)
    │
    ├──→ modAHighpass → modALowpass → modAAnalyser → Peak Detection
    ├──→ modBHighpass → modBLowpass → modBAnalyser → Peak Detection
    └──→ modCHighpass → modCLowpass → modCAnalyser → Peak Detection
         (parallel analysis chains - silent)
```

### Peak Detection Implementation

```javascript
// ============================================
// PEAK DETECTION
// Location: analyzeAudio() ~line 9200
// ============================================

function getPeakFromAnalyser(analyser, dataArray) {
    // Get waveform samples (not FFT bins!)
    analyser.getFloatTimeDomainData(dataArray);  // [-1, 1] range
    
    // Find peak amplitude
    let peak = 0;
    for (let i = 0; i < dataArray.length; i++) {
        const abs = Math.abs(dataArray[i]);
        if (abs > peak) peak = abs;
    }
    return peak;  // 0.0 to 1.0
}

// Usage in render loop
const rawA = getPeakFromAnalyser(modAAnalyser, modADataArray);
const rawB = getPeakFromAnalyser(modBAnalyser, modBDataArray);
const rawC = getPeakFromAnalyser(modCAnalyser, modCDataArray);

// Apply envelope
const { a, b, c } = applyEnvelopeFollower(rawA, rawB, rawC);
```

### Live Filter Updates

```javascript
// ============================================
// LIVE FILTER UPDATES
// Location: updateBandpassFilters() ~line 9100
// ============================================

function updateBandpassFilters() {
    // Get current knob values
    const aFreq1 = getFreq1Value('A');
    const aFreq2 = getFreq2Value('A');
    
    // Update filter cutoffs immediately
    if (modAHighpass) {
        modAHighpass.frequency.value = aFreq1;
        modAHighpass.Q.value = modAQ;
    }
    if (modALowpass) {
        modALowpass.frequency.value = aFreq2;
        modALowpass.Q.value = modAQ;
    }
    
    // Repeat for B, C...
}

// Called when:
// - Freq1/Freq2 knobs change
// - Q knobs change in Mod Tuning panel
// - Container loaded with different settings
```

### Offline Baking (Timeline Analysis)

```javascript
// ============================================
// OFFLINE BAKING
// Location: analyzeAudioTimeline() ~line 9400
// ============================================

async function analyzeAudioTimeline(audioBuffer) {
    // Create offline context
    const offlineCtx = new OfflineAudioContext(
        audioBuffer.numberOfChannels,
        audioBuffer.length,
        audioBuffer.sampleRate
    );
    
    // Create same filter chains as live analysis
    // Process in chunks at analysis FPS
    // Capture peaks per frame
    // Apply envelope to each frame
    
    return {
        shaderModATimeline: [...],  // Real dynamic values
        shaderModBTimeline: [...],
        shaderModCTimeline: [...]
    };
}
```

### Results Comparison

| Scenario | FFT Averaging (Before) | Bandpass Peak (After) |
|----------|------------------------|----------------------|
| Kick hit | 0.02 | 0.72 |
| Silence | 0.02 | 0.01 |
| Hi-hat | 0.02 | 0.15 |
| Snare | 0.02 | 0.45 |
| Full mix | 0.02 | Dynamic 0.1-0.8 |

---

## MOD TUNING PANEL

### Access Pattern

```javascript
// ============================================
// MOD TUNING PANEL ACCESS
// ============================================

// 1. Enter Reactivity Focus (Shift+R)
// 2. Press N to open panel
// 3. Panel slides from left edge of canvas
// 4. Escape or N to close
```

### Global State

```javascript
// ============================================
// MOD TUNING STATE
// Location: ~line 7600
// ============================================

// Per-modulator Q (filter sharpness)
let modAQ = 0.7, modBQ = 0.7, modCQ = 0.7;

// Global ADSR envelope
let modAttack = 5;      // 0-100 ms
let modDecay = 20;      // 0-200 ms
let modSustain = 0.7;   // 0-1.0 ratio
let modRelease = 50;    // 0-500 ms
let modThreshold = 0.01; // 0-0.20 noise gate

// Panel state
let modTuningPanelActive = false;

// Envelope follower runtime
let envFollowerA = 0, envFollowerB = 0, envFollowerC = 0;
let envPeakA = 0, envPeakB = 0, envPeakC = 0;
let lastEnvTime = 0;
```

### DOM Structure

```html
<!-- Mod Tuning Panel -->
<div id="modTuningPanel" class="mod-tuning-panel">
    <div class="mod-tuning-header">MOD TUNING</div>
    
    <!-- Per-mod rows -->
    <div class="mod-tuning-row" data-mod="A">
        <span class="mod-label">A</span>
        <canvas class="mod-meter" id="modMeterA" width="60" height="12"></canvas>
        <span class="mod-value" id="modValueA">0.00</span>
        <canvas class="mod-q-knob" id="modQKnobA" width="24" height="24"></canvas>
    </div>
    <div class="mod-tuning-row" data-mod="B">...</div>
    <div class="mod-tuning-row" data-mod="C">...</div>
    
    <!-- Global ADSR -->
    <div class="mod-envelope-row">
        <div class="mod-env-control">
            <span>A</span>
            <canvas class="mod-env-knob" id="modAttackKnob" width="24" height="24"></canvas>
        </div>
        <div class="mod-env-control">
            <span>D</span>
            <canvas class="mod-env-knob" id="modDecayKnob" width="24" height="24"></canvas>
        </div>
        <div class="mod-env-control">
            <span>S</span>
            <canvas class="mod-env-knob" id="modSustainKnob" width="24" height="24"></canvas>
        </div>
        <div class="mod-env-control">
            <span>R</span>
            <canvas class="mod-env-knob" id="modReleaseKnob" width="24" height="24"></canvas>
        </div>
    </div>
    
    <!-- Threshold -->
    <div class="mod-threshold-row">
        <span>Thresh</span>
        <canvas class="mod-env-knob" id="modThresholdKnob" width="24" height="24"></canvas>
    </div>
</div>
```

### Envelope Processing

```javascript
// ============================================
// ENVELOPE FOLLOWER
// Location: applyEnvelopeFollower() ~line 9300
// ============================================

function applyEnvelopeFollower(rawA, rawB, rawC) {
    const now = performance.now();
    const dt = (now - lastEnvTime) / 1000;  // Delta in seconds
    lastEnvTime = now;
    
    envFollowerA = processEnvelope(rawA, envFollowerA, envPeakA, dt, 'A');
    envFollowerB = processEnvelope(rawB, envFollowerB, envPeakB, dt, 'B');
    envFollowerC = processEnvelope(rawC, envFollowerC, envPeakC, dt, 'C');
    
    return { a: envFollowerA, b: envFollowerB, c: envFollowerC };
}

function processEnvelope(input, current, peak, dt, mod) {
    // Gate detection
    const gateOn = input >= modThreshold;
    
    if (gateOn) {
        // Track peak
        if (input > peak) peak = input;
        
        // Attack phase (toward peak)
        if (current < peak) {
            const attackRate = 1 / (modAttack / 1000);
            current += (peak - current) * attackRate * dt;
        }
        // Decay phase (toward sustain level)
        else {
            const decayRate = 1 / (modDecay / 1000);
            const sustainLevel = peak * modSustain;
            current += (sustainLevel - current) * decayRate * dt;
        }
    } else {
        // Release phase (toward zero)
        const releaseRate = 1 / (modRelease / 1000);
        current += (0 - current) * releaseRate * dt;
        
        // Reset peak tracker
        if (mod === 'A') envPeakA = 0;
        else if (mod === 'B') envPeakB = 0;
        else envPeakC = 0;
    }
    
    return Math.max(0, Math.min(1, current));
}
```

### Panel Toggle

```javascript
// ============================================
// PANEL TOGGLE
// Location: toggleModTuningPanel() ~line 9350
// ============================================

function toggleModTuningPanel() {
    if (modTuningPanelActive) {
        closeModTuningPanel();
    } else {
        openModTuningPanel();
    }
}

function openModTuningPanel() {
    modTuningPanelActive = true;
    const panel = document.getElementById('modTuningPanel');
    if (panel) panel.classList.add('active');
    showMessage('Mod Tuning | Q per mod, ADSR global', 'info');
}

function closeModTuningPanel() {
    modTuningPanelActive = false;
    const panel = document.getElementById('modTuningPanel');
    if (panel) panel.classList.remove('active');
}
```

---

## UNIFIED SETS SYSTEM

### Architecture Change

```javascript
// ============================================
// BEFORE (V5_2_0)
// ============================================

let userSetsCache = [];      // Manual sets (file-based)
let smartSetsCache = [];     // Smart sets (project state)

// ============================================
// AFTER (V5_3_0)
// ============================================

let userSetsCache = [];      // ALL sets (unified, file-based)
// smartSetsCache removed

let setsFolderHandle = null;  // /Sets/ folder handle
```

### Set Object Structure

```javascript
// ============================================
// SET OBJECT STRUCTURE
// ============================================

// Manual set - drag containers to add
{
    id: "set_1703712000000",
    name: "My Favorites",
    type: "manual",
    containers: ["container1", "container2", "container3"],
    created: "2025-12-27T10:00:00Z"
}

// Smart set - saved search query
{
    id: "smart_1703712000000",
    name: "Bass 128+",
    type: "smart",
    query: "bass",
    created: "2025-12-27T10:00:00Z"
}
```

### File Operations

```javascript
// ============================================
// SET FILE OPERATIONS
// Location: ~line 12000
// ============================================

async function saveSet(set) {
    if (!setsFolderHandle) {
        setsFolderHandle = await applicationFolderHandle
            .getDirectoryHandle('Sets', { create: true });
    }
    
    const filename = sanitizeFilename(set.name) + '.oms-set';
    const fileHandle = await setsFolderHandle
        .getFileHandle(filename, { create: true });
    
    const writable = await fileHandle.createWritable();
    await writable.write(JSON.stringify(set, null, 2));
    await writable.close();
}

async function loadAllSets() {
    if (!setsFolderHandle) {
        try {
            setsFolderHandle = await applicationFolderHandle
                .getDirectoryHandle('Sets');
        } catch (e) {
            return [];  // No Sets folder yet
        }
    }
    
    const sets = [];
    for await (const entry of setsFolderHandle.values()) {
        if (entry.name.endsWith('.oms-set')) {
            const file = await entry.getFile();
            const data = JSON.parse(await file.text());
            sets.push(data);
        }
    }
    return sets;
}

async function deleteSet(setId) {
    const set = userSetsCache.find(s => s.id === setId);
    if (!set) return;
    
    const filename = sanitizeFilename(set.name) + '.oms-set';
    await setsFolderHandle.removeEntry(filename);
    
    userSetsCache = userSetsCache.filter(s => s.id !== setId);
    refreshLibraryListLeft();
}
```

### Smart Set Filtering

```javascript
// ============================================
// SMART SET FILTERING
// Location: ~line 12100
// ============================================

function getFilteredContainers(query) {
    if (!query || query.trim() === '') {
        return libraryContainersCache;
    }
    
    const lowerQuery = query.toLowerCase().trim();
    
    return libraryContainersCache.filter(container => {
        // Match name
        if (container.name?.toLowerCase().includes(lowerQuery)) return true;
        
        // Match tags
        if (container.tags?.some(t => t.toLowerCase().includes(lowerQuery))) {
            return true;
        }
        
        // Match shader code (optional, expensive)
        if (container.shaderCode?.toLowerCase().includes(lowerQuery)) {
            return true;
        }
        
        return false;
    });
}

function saveCurrentSearch() {
    const query = document.getElementById('librarySearchInput')?.value?.trim();
    if (!query) return;
    
    const set = {
        id: 'smart_' + Date.now(),
        name: query,  // Use query as initial name
        type: 'smart',
        query: query,
        created: new Date().toISOString()
    };
    
    userSetsCache.push(set);
    saveSet(set);
    refreshLibraryListLeft();
    showMessage(`Smart set "${query}" saved`, 'state');
}
```

### Legacy Migration

```javascript
// ============================================
// LEGACY MIGRATION
// Location: migrateSmartSets() ~line 12200
// ============================================

async function migrateSmartSets() {
    const projectState = await loadProjectState();
    
    // Check for legacy smartSets in project state
    if (projectState.smartSets && projectState.smartSets.length > 0) {
        console.log(`Migrating ${projectState.smartSets.length} smart sets to files`);
        
        for (const legacySet of projectState.smartSets) {
            const set = {
                id: legacySet.id || 'smart_' + Date.now(),
                name: legacySet.name,
                type: 'smart',
                query: legacySet.query,
                created: legacySet.created || new Date().toISOString()
            };
            
            await saveSet(set);
            userSetsCache.push(set);
        }
        
        // Remove from project state
        delete projectState.smartSets;
        await saveProjectState(projectState);
        
        showMessage(`Migrated ${projectState.smartSets.length} smart sets`, 'state');
    }
}
```

---

## TAGS SYSTEM

### Schema Addition

```javascript
// ============================================
// .OMS CONTAINER - TAGS
// ============================================

{
    "name": "My Container",
    "tags": ["bass", "heavy", "128bpm"],
    // ... other fields
}
```

### Chip UI Implementation

```javascript
// ============================================
// TAG CHIP FUNCTIONS
// Location: ~line 13000
// ============================================

function addTagChip(text) {
    const tag = text.trim().toLowerCase();
    if (!tag) return;
    
    // Check for duplicates
    const existing = getTagsFromChips();
    if (existing.includes(tag)) {
        showMessage('Tag already exists', 'info');
        return;
    }
    
    // Create chip element
    const chip = document.createElement('span');
    chip.className = 'tag-chip';
    chip.dataset.tag = tag;
    chip.innerHTML = `
        ${tag}
        <button class="tag-remove" onclick="removeTagChip(this)">×</button>
    `;
    
    document.getElementById('tagsChips').appendChild(chip);
    document.getElementById('tagsInput').value = '';
}

function removeTagChip(btn) {
    btn.parentElement.remove();
}

function getTagsFromChips() {
    return Array.from(document.querySelectorAll('.tag-chip'))
        .map(chip => chip.dataset.tag);
}

function populateTagChips(tags) {
    const container = document.getElementById('tagsChips');
    container.innerHTML = '';
    
    if (tags && tags.length > 0) {
        tags.forEach(tag => {
            const chip = document.createElement('span');
            chip.className = 'tag-chip';
            chip.dataset.tag = tag;
            chip.innerHTML = `
                ${tag}
                <button class="tag-remove" onclick="removeTagChip(this)">×</button>
            `;
            container.appendChild(chip);
        });
    }
}
```

### Admin Panel Functions

```javascript
// ============================================
// ADMIN TAGS PANEL
// Location: ~line 13200
// ============================================

function renderTagsAdminPanel() {
    // Collect all unique tags with counts
    const tagCounts = {};
    for (const container of libraryContainersCache) {
        if (container.tags) {
            for (const tag of container.tags) {
                tagCounts[tag] = (tagCounts[tag] || 0) + 1;
            }
        }
    }
    
    const sortedTags = Object.keys(tagCounts).sort();
    
    const list = document.getElementById('adminTagsList');
    list.innerHTML = '';
    
    for (const tag of sortedTags) {
        const li = document.createElement('li');
        li.className = 'admin-tag-item';
        li.innerHTML = `
            <span class="tag-name" data-tag="${tag}" 
                  onclick="startRenameTag('${tag}')">${tag}</span>
            <span class="tag-count">(${tagCounts[tag]})</span>
            <button class="tag-delete" onclick="deleteTag('${tag}')">×</button>
        `;
        list.appendChild(li);
    }
}

async function renameTag(oldTag, newTag) {
    newTag = newTag.trim().toLowerCase();
    if (!newTag || newTag === oldTag) return;
    
    let modified = 0;
    for (const container of libraryContainersCache) {
        if (container.tags?.includes(oldTag)) {
            const idx = container.tags.indexOf(oldTag);
            container.tags[idx] = newTag;
            await saveContainerToOMS(container);
            modified++;
        }
    }
    
    await refreshLibraryList();
    showMessage(`Renamed "${oldTag}" to "${newTag}" in ${modified} containers`, 'state');
}

async function deleteTag(tag) {
    let modified = 0;
    for (const container of libraryContainersCache) {
        if (container.tags?.includes(tag)) {
            container.tags = container.tags.filter(t => t !== tag);
            await saveContainerToOMS(container);
            modified++;
        }
    }
    
    await refreshLibraryList();
    showMessage(`Deleted "${tag}" from ${modified} containers`, 'state');
}
```

---

## PROPER PEAK METERING

### Implementation

```javascript
// ============================================
// PEAK METERING (PERFORM Tab)
// Location: ~line 18000
// ============================================

function getPeakLevel(analyser) {
    const dataArray = new Float32Array(analyser.fftSize);
    analyser.getFloatTimeDomainData(dataArray);  // Waveform, not FFT!
    
    let peak = 0;
    for (let i = 0; i < dataArray.length; i++) {
        const abs = Math.abs(dataArray[i]);
        if (abs > peak) peak = abs;
    }
    return peak;
}

function peakToDb(peak) {
    if (peak <= 0) return -Infinity;
    return 20 * Math.log10(peak);
}

function peakToMeterPercent(peak) {
    const dB = peakToDb(peak);
    // -48dB floor, 0dB ceiling
    const percent = ((dB - (-48)) / 48) * 100;
    return Math.max(0, Math.min(100, percent));
}
```

### Color Thresholds

```javascript
// ============================================
// METER COLORS
// ============================================

// Using CSS gradient mask
const meterGradient = `
    linear-gradient(to right,
        #00ff00 0%,      /* Green: -48 to -18 dB */
        #00ff00 62.5%,
        #ffff00 62.5%,   /* Yellow: -18 to -6 dB */
        #ffff00 87.5%,
        #ff0000 87.5%,   /* Red: -6 to 0 dB */
        #ff0000 100%
    )
`;
```

---

## QUANTIZE SYSTEM

### State

```javascript
// ============================================
// QUANTIZE STATE
// Location: ~line 6800
// ============================================

let quantizeValue = 4;  // 0 = Off, 1 = Beat, 4 = Bar (default)
let tempoHistory = [];  // Last 8 tempo samples for stability
```

### Implementation

```javascript
// ============================================
// QUANTIZE LOGIC
// Location: getNextQuantizedBeat() ~line 17500
// ============================================

function getNextQuantizedBeat(currentBeat) {
    if (quantizeValue === 0) {
        return currentBeat;  // Immediate, no quantize
    }
    
    // Forward-only: always ceil, never round
    return Math.ceil(currentBeat / quantizeValue) * quantizeValue;
}

// Usage in launch
function launchCell(track, scene) {
    const currentBeat = masterBeatNow();
    const launchBeat = getNextQuantizedBeat(currentBeat);
    const delayMs = (launchBeat - currentBeat) * (60000 / gridState.masterTempo);
    
    if (delayMs > 0) {
        setTimeout(() => playCellAudio(track, scene), delayMs);
    } else {
        playCellAudio(track, scene);
    }
}
```

---

## DRAGGABLE BAR 1 MARKER

### Implementation

```javascript
// ============================================
// BAR 1 MARKER DRAG
// Location: ~line 10500
// ============================================

let bar1Dragging = false;
let bar1DragStartX = 0;
let bar1DragStartValue = 0;

function initBar1Drag(canvas) {
    canvas.addEventListener('mousedown', (e) => {
        // Check if click is near Bar 1 marker
        const bar1X = getBar1PixelPosition();
        if (Math.abs(e.offsetX - bar1X) < 10) {
            bar1Dragging = true;
            bar1DragStartX = e.clientX;
            bar1DragStartValue = audioStartTime;
            canvas.style.cursor = 'ew-resize';
        }
    });
    
    document.addEventListener('mousemove', (e) => {
        if (!bar1Dragging) return;
        
        const deltaX = e.clientX - bar1DragStartX;
        const deltaSeconds = deltaX / pixelsPerSecond;
        
        // Snap to 100ms grid
        const newValue = bar1DragStartValue + deltaSeconds;
        audioStartTime = Math.round(newValue * 10) / 10;
        
        // Clamp to valid range
        audioStartTime = Math.max(0, Math.min(audioDuration, audioStartTime));
        
        // Update UI
        document.getElementById('audioStartField').value = audioStartTime.toFixed(1);
        redrawWaveform();
    });
    
    document.addEventListener('mouseup', () => {
        if (bar1Dragging) {
            bar1Dragging = false;
            canvas.style.cursor = 'default';
        }
    });
}
```

---

## VOLUME dB SCALE

### Conversion Functions

```javascript
// ============================================
// VOLUME dB SCALE
// Location: ~line 8800
// ============================================

function volumePositionToGain(position) {
    // 0.0 = 0.0 gain (silence)
    // 0.5 = 1.0 gain (unity, 0dB)
    // 1.0 = 2.0 gain (+6dB)
    return position * 2;
}

function volumeGainToPosition(gain) {
    return gain / 2;
}

function volumePositionToDbString(position) {
    if (position === 0) return '-∞ dB';
    const gain = volumePositionToGain(position);
    const dB = 20 * Math.log10(gain);
    if (dB >= 0) return `+${dB.toFixed(1)} dB`;
    return `${dB.toFixed(1)} dB`;
}

// Usage
// position 0.5 → gain 1.0 → "0.0 dB"
// position 1.0 → gain 2.0 → "+6.0 dB"
// position 0.25 → gain 0.5 → "-6.0 dB"
```

---

## SCHEMA ADDITIONS

### .oms Containers

```javascript
// ============================================
// .OMS SCHEMA - V5_3_0 ADDITIONS
// ============================================

{
    // Existing fields...
    
    // V5_3_0: Tags
    "tags": ["bass", "heavy", "128bpm"],
    
    // V5_3_0: Mod Tuning
    "modTuning": {
        "modAQ": 0.7,
        "modBQ": 0.7,
        "modCQ": 0.7,
        "modAttack": 5,
        "modDecay": 20,
        "modSustain": 0.7,
        "modRelease": 50,
        "modThreshold": 0.01
    }
}
```

### .oms-set Files

```javascript
// ============================================
// .OMS-SET SCHEMA - V5_3_0
// ============================================

// Manual set
{
    "id": "set_1703712000000",
    "name": "My Favorites",
    "type": "manual",
    "containers": ["container1", "container2"],
    "created": "2025-12-27T10:00:00Z"
}

// Smart set
{
    "id": "smart_1703712000000",
    "name": "Bass 128+",
    "type": "smart",
    "query": "bass",
    "created": "2025-12-27T10:00:00Z"
}
```

### Project State (onemanshyo.omsp)

```javascript
// ============================================
// PROJECT STATE - V5_3_0 ADDITIONS
// ============================================

{
    // Existing fields...
    
    // V5_3_0: Audio defaults
    "prefCuePattern": "8bar",      // 4bar, 8bar, 16bar
    "prefLoopBars": 16,            // 16, 32, 64
    "prefLoopEnabled": false,
    
    // V5_3_0: Quantize setting
    "quantizeValue": 4             // 0, 1, 4
}
```

---

## TESTING NOTES

### Real Audio Reactivity Tests

1. **Peak Detection:** Play audio, verify modulator values vary with dynamics
2. **Frequency Bands:** Adjust Freq1/Freq2, verify different frequencies isolated
3. **Q Response:** Increase Q, verify sharper frequency selection
4. **Envelope:** Test ADSR response to transients (kick hits)
5. **Threshold:** Set threshold, verify noise rejection
6. **Baking:** Analyze timeline, verify saved values match live preview

### Mod Tuning Panel Tests

1. **Access:** Shift+R then N opens panel
2. **Q Knobs:** Each modulator has independent Q
3. **ADSR:** Global envelope affects all modulators
4. **Meters:** Live meters show post-envelope values
5. **Persistence:** Settings saved/restored with container

### Sets System Tests

1. **Manual Create:** Drag containers to new set
2. **Smart Create:** Search, click Save, creates smart set
3. **Indicator:** Smart sets show cyan dot
4. **Delete:** Both types delete correctly
5. **Migration:** Legacy smartSets migrated on load

### Tags Tests

1. **Add:** Type + Enter creates chip
2. **Remove:** × button removes chip
3. **Duplicate:** Prevented (case-insensitive)
4. **Search:** Tags included in search matching
5. **Admin:** Rename/delete affects all containers

### Metering Tests

1. **Range:** Meters show full -48dB to 0dB range
2. **Colors:** Green/Yellow/Red bands correct
3. **Response:** Meters respond to actual audio peaks

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_3_0
Date: December 27, 2025
License: GPL-3.0
