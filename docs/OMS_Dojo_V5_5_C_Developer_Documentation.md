# ONEMANSHYO App [Beta V5_5_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_5_0  
**Date:** January 3, 2026  
**Purpose:** Technical implementation reference for V5_5_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Transform Animation System](#transform-animation-system)
3. [Per-Mod ADSR+T Envelopes](#per-mod-adsrt-envelopes)
4. [PIP Camera System](#pip-camera-system)
5. [ORGANIZE Tab UX](#organize-tab-ux)
6. [PERFORM Keyboard Workflow](#perform-keyboard-workflow)
7. [FX Overlay System](#fx-overlay-system)
8. [Schema Additions](#schema-additions)
9. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_5_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  TRANSFORM ANIMATION SYSTEM                                         │
│  Beat-synced rotation/scale/tile with exponential speed control     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Mode keys │ Speed trackpad │ shaderLoopLength sync │ Persist │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  PER-MOD ADSR+T ENVELOPES                                           │
│  Independent Q/A/D/S/R/T per frequency band (18 parameters)         │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ N-panel sync │ Row selection │ Live readouts │ Container save│   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  PIP CAMERA SYSTEM                                                  │
│  3 PIP types + full-screen overlay with shapes/colors/persistence   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ cam* services │ pip1/2/3 │ O key overlay │ Border-click color│   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  ORGANIZE TAB UX OVERHAUL                                           │
│  Apple Music-style auto-save with tag autocomplete                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Blur auto-save │ Tag suggestions │ Usage counts │ Navigation │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  PERFORM KEYBOARD WORKFLOW                                          │
│  Bare number keys, quick-load, library navigation                   │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ 1-4 select │ SHIFT+Right load │ Arrow nav │ Visual feedback  │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Beat-Synced Transforms** - All animation modes reference shaderLoopLength
2. **Per-Mod Independence** - Each frequency band has its own envelope settings
3. **Shared Camera Services** - Single stream feeds multiple PIP outputs
4. **Auto-Save UX** - No OK/Cancel, changes save on blur
5. **Pre-Fader Tap Preserved** - Quick-load doesn't break existing audio routing

---

## TRANSFORM ANIMATION SYSTEM

### State Structure

```javascript
// ============================================
// TRANSFORM MODES STATE
// Location: ~line 2200
// ============================================

let transformModes = {
    rotation: { mode: 'stop', speed: 1.0 },
    scale: { mode: 'stop', speed: 1.0 },
    tile: { mode: 'stop', speed: 1.0 }
};

// Mode values: 'stop', 'clockwise', 'counterclockwise', 'pingpong'
// Speed range: 0.03 to 256 (exponential via trackpad)
```

### Beat-Synced Animation Calculation

```javascript
// ============================================
// TRANSFORM ANIMATION CALCULATION
// Location: applyTransformAnimation() ~line 9800
// ============================================

function applyTransformAnimation(transform, beat) {
    const config = transformModes[transform];
    if (config.mode === 'stop') return;
    
    // Get loop length in beats
    const loopBeats = getShaderLoopLengthBeats();
    if (loopBeats <= 0) return;
    
    // Calculate position within loop (0-1)
    const adjustedBeat = beat * config.speed;
    const loopPosition = (adjustedBeat % loopBeats) / loopBeats;
    
    let value;
    switch (config.mode) {
        case 'clockwise':  // Sawtooth up
            value = loopPosition;
            break;
        case 'counterclockwise':  // Sawtooth down
            value = 1 - loopPosition;
            break;
        case 'pingpong':  // Triangle wave
            value = loopPosition < 0.5 
                ? loopPosition * 2 
                : 2 - loopPosition * 2;
            break;
    }
    
    // Apply to transform (range depends on transform type)
    applyTransformValue(transform, value);
}

function applyTransformValue(transform, normalized) {
    switch (transform) {
        case 'rotation':
            // 0-1 → 0-360 degrees
            transformRotation = normalized * 360;
            break;
        case 'scale':
            // 0-1 → 0.1-4.0
            transformScaleX = transformScaleY = 0.1 + normalized * 3.9;
            break;
        case 'tile':
            // 0-1 → 1-8 (integer steps)
            transformTile = Math.round(1 + normalized * 7);
            break;
    }
}
```

### Exponential Speed Control

```javascript
// ============================================
// TRACKPAD SPEED ADJUSTMENT
// Location: handleTransformWheel() ~line 9900
// ============================================

function handleTransformWheel(transform, deltaY) {
    const config = transformModes[transform];
    
    // Exponential scaling for huge range
    const factor = Math.pow(1.01, -deltaY);
    let newSpeed = config.speed * factor;
    
    // Clamp to 0.03x - 256x range
    newSpeed = Math.max(0.03, Math.min(256, newSpeed));
    
    config.speed = newSpeed;
    updateTransformSpeedDisplay(transform);
}

// Result: Small scroll = fine adjustment, large scroll = big jumps
// 0.03x = 1 loop takes ~33× longer
// 256x = 256 loops per original loop
```

### Mode Key Handling

```javascript
// ============================================
// MODE KEY HANDLING (Transform Focus)
// Location: handleTransformFocusKey() ~line 10000
// ============================================

function handleTransformModeKey(key) {
    const transform = getSelectedTransform();  // rotation, scale, or tile
    
    const modeMap = {
        '0': 'stop',
        '1': 'clockwise',      // CW / Grow / Increase
        '2': 'counterclockwise', // CCW / Shrink / Decrease
        '3': 'pingpong'
    };
    
    if (modeMap[key]) {
        transformModes[transform].mode = modeMap[key];
        updateTransformModeUI(transform);
        showMessage(`${transform}: ${modeMap[key]}`, 'state');
    }
}
```

### Container Persistence

```javascript
// ============================================
// TRANSFORM MODES IN CONTROL STATES
// Location: buildContainerObject() ~line 5500
// ============================================

// Save
controlStates.transformRotationMode = transformModes.rotation.mode;
controlStates.transformRotationSpeed = transformModes.rotation.speed;
controlStates.transformScaleMode = transformModes.scale.mode;
controlStates.transformScaleSpeed = transformModes.scale.speed;
controlStates.transformTileMode = transformModes.tile.mode;
controlStates.transformTileSpeed = transformModes.tile.speed;

// Load (in loadContainer)
if (controlStates.transformRotationMode) {
    transformModes.rotation.mode = controlStates.transformRotationMode;
    transformModes.rotation.speed = controlStates.transformRotationSpeed || 1.0;
}
// ... same for scale and tile
```

---

## PER-MOD ADSR+T ENVELOPES

### State Structure

```javascript
// ============================================
// PER-MOD ENVELOPE STATE
// Location: ~line 2400
// ============================================

// ModA (Bass) - default values
let modAQ = 0.70;
let modAAttack = 5;      // ms
let modADecay = 20;      // ms
let modASustain = 70;    // %
let modARelease = 50;    // ms
let modAThreshold = 10;  // %

// ModB (Mids)
let modBQ = 0.70;
let modBAttack = 10;
let modBDecay = 30;
let modBSustain = 80;
let modBRelease = 100;
let modBThreshold = 5;

// ModC (Treble)
let modCQ = 0.70;
let modCAttack = 2;
let modCDecay = 15;
let modCSustain = 60;
let modCRelease = 80;
let modCThreshold = 5;
```

### N-Panel Row Synchronization

```javascript
// ============================================
// N-PANEL ROW SYNC
// Location: syncModTuningToRow() ~line 11200
// ============================================

function syncModTuningToRow(row) {
    // row: 0=ModA, 1=ModB, 2=ModC
    const prefix = ['modA', 'modB', 'modC'][row];
    
    // Update panel header
    const modLabels = ['Mod A (Bass)', 'Mod B (Mids)', 'Mod C (Treble)'];
    document.getElementById('modTuningHeader').textContent = modLabels[row];
    
    // Update knob displays
    updateModTuningKnob('Q', window[`${prefix}Q`]);
    updateModTuningKnob('Attack', window[`${prefix}Attack`]);
    updateModTuningKnob('Decay', window[`${prefix}Decay`]);
    updateModTuningKnob('Sustain', window[`${prefix}Sustain`]);
    updateModTuningKnob('Release', window[`${prefix}Release`]);
    updateModTuningKnob('Threshold', window[`${prefix}Threshold`]);
    
    // Update readouts
    updateModTuningReadouts(row);
}

// Called when navigating Reactivity rows with ↑/↓
function handleReactivityRowChange(newRow) {
    createReactivitySelectedRow = newRow;
    updateReactivityRowHighlight();
    
    if (modTuningPanelOpen) {
        syncModTuningToRow(newRow);
    }
}
```

### Envelope Value Update

```javascript
// ============================================
// ENVELOPE VALUE UPDATE
// Location: updateModEnvelopeValue() ~line 11300
// ============================================

function updateModEnvelopeValue(param, value) {
    const row = createReactivitySelectedRow;  // 0, 1, or 2
    const prefix = ['modA', 'modB', 'modC'][row];
    
    // Update the appropriate variable
    window[`${prefix}${param}`] = value;
    
    // Update display
    updateModTuningReadout(param, value);
    
    // Apply to audio processing (if applicable)
    if (param === 'Q') {
        updateBandpassQ(row, value);
    }
}

// Parameter ranges:
// Q: 0.5 - 10.0 (default 0.7)
// Attack: 0 - 100 ms (default 5)
// Decay: 0 - 200 ms (default 20)
// Sustain: 0 - 100 % (default 70)
// Release: 0 - 500 ms (default 50)
// Threshold: 0 - 100 % (default 10)
```

### Container Persistence

```javascript
// ============================================
// PER-MOD ENVELOPES IN CONTROL STATES
// Location: buildContainerObject() ~line 5600
// ============================================

// Save all 18 parameters
controlStates.modAQ = modAQ;
controlStates.modAAttack = modAAttack;
controlStates.modADecay = modADecay;
controlStates.modASustain = modASustain;
controlStates.modARelease = modARelease;
controlStates.modAThreshold = modAThreshold;
// ... same for modB and modC

// Load with fallback to global defaults
if (controlStates.modAQ !== undefined) {
    modAQ = controlStates.modAQ;
    modAAttack = controlStates.modAAttack;
    // ... etc
} else {
    // Fallback to global defaults (backward compatibility)
    modAQ = modBQ = modCQ = 0.70;
    // ... etc
}
```

---

## PIP CAMERA SYSTEM

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│  CAMERA SERVICES (cam*)                                         │
│  Shared resources: stream, settings, device enumeration         │
├─────────────────────────────────────────────────────────────────┤
│  camStream ──┬──→ camOverlay (O key, all tabs, full-screen)    │
│              │                                                   │
│              ├──→ pip1 (P key, PERFORM, slideout)               │
│              │                                                   │
│              ├──→ pip2 (P key, CREATE/ORG/ADMIN, corner)        │
│              │                                                   │
│              └──→ pip3 (Shift+P, CREATE/ORG/ADMIN, center)      │
└─────────────────────────────────────────────────────────────────┘
```

### State Variables

```javascript
// ============================================
// CAMERA SYSTEM STATE
// Location: ~line 2600
// ============================================

// Shared camera services
let camSettings = {
    sourceId: 'default',
    mirror: false
};
let camDevices = [];
let camStream = null;

// Full-screen overlay
let camOverlayVisible = false;

// PIP 1 (PERFORM slideout)
let pip1Visible = false;
let pip1Color = '#00d4ff';
let pip1Shape = 'landscape';
let pip1DefaultLabel = 'PIP 1';

// PIP 2 (Corner)
let pip2Visible = false;
let pip2Color = '#00d4ff';
let pip2Shape = 'landscape';

// PIP 3 (Center)
let pip3Visible = false;
let pip3Color = '#00d4ff';
let pip3Shape = 'square';

// Shape options: 'landscape', 'portrait', 'square', 'circle'
// Color options: 'transparent', '#00d4ff', '#ff00ff', '#ffa500', 
//                '#ffff00', '#00ff00', '#8800ff', '#ff69b4', '#ffffff'
```

### Smart Stream Management

```javascript
// ============================================
// SMART STREAM MANAGEMENT
// Location: ~line 12500
// ============================================

function isCamInUse() {
    return pip1Visible || pip2Visible || pip3Visible || camOverlayVisible;
}

async function startCamStream() {
    if (camStream) return camStream;  // Already running
    
    try {
        const constraints = {
            video: {
                deviceId: camSettings.sourceId !== 'default' 
                    ? { exact: camSettings.sourceId } 
                    : undefined,
                width: { ideal: 1280 },
                height: { ideal: 720 }
            }
        };
        
        camStream = await navigator.mediaDevices.getUserMedia(constraints);
        return camStream;
    } catch (err) {
        console.error('Camera access failed:', err);
        showMessage('Camera access denied', 'error');
        return null;
    }
}

function stopCamStream() {
    // Only stop if no outputs using it
    if (isCamInUse()) return;
    
    if (camStream) {
        camStream.getTracks().forEach(track => track.stop());
        camStream = null;
    }
}
```

### PIP Toggle Functions

```javascript
// ============================================
// PIP TOGGLE FUNCTIONS
// Location: ~line 12600
// ============================================

async function toggleCamOverlay() {
    camOverlayVisible = !camOverlayVisible;
    const overlay = document.getElementById('cam-overlay');
    
    if (camOverlayVisible) {
        const stream = await startCamStream();
        if (stream) {
            overlay.querySelector('video').srcObject = stream;
            overlay.style.display = 'block';
        }
    } else {
        overlay.style.display = 'none';
        stopCamStream();
    }
}

async function togglePip1() {
    pip1Visible = !pip1Visible;
    const container = document.getElementById('pip1-container');
    
    if (pip1Visible) {
        const stream = await startCamStream();
        if (stream) {
            container.querySelector('video').srcObject = stream;
            container.classList.add('visible');
            container.classList.add('genie-in');
        }
    } else {
        container.classList.remove('visible');
        container.classList.add('genie-out');
        stopCamStream();
    }
}

// pip2 and pip3 follow same pattern with different containers
```

### Border-Click Color Picker

```javascript
// ============================================
// BORDER-CLICK COLOR PICKER
// Location: ~line 12800
// ============================================

function initPipBorderClick(pipId) {
    const container = document.getElementById(`${pipId}-container`);
    
    container.addEventListener('click', (e) => {
        // Check if click is on border (not video)
        // Video has pointer-events: none, so clicks pass through to container
        const rect = container.getBoundingClientRect();
        const borderWidth = 2;
        
        const onBorder = (
            e.clientX < rect.left + borderWidth ||
            e.clientX > rect.right - borderWidth ||
            e.clientY < rect.top + borderWidth ||
            e.clientY > rect.bottom - borderWidth
        );
        
        if (onBorder) {
            showPipColorPicker(pipId, e);
        }
    });
}

function showPipColorPicker(pipId, event) {
    const picker = document.getElementById('pip-color-picker');
    
    // Position near click
    picker.style.left = `${event.clientX}px`;
    picker.style.top = `${event.clientY}px`;
    picker.dataset.pip = pipId;
    picker.style.display = 'flex';
    
    // Click outside dismisses
    setTimeout(() => {
        document.addEventListener('click', hidePipColorPicker, { once: true });
    }, 10);
}

function selectPipColor(color) {
    const picker = document.getElementById('pip-color-picker');
    const pipId = picker.dataset.pip;
    
    // Update state
    window[`${pipId}Color`] = color;
    
    // Apply to container
    const container = document.getElementById(`${pipId}-container`);
    container.style.borderColor = color;
    
    hidePipColorPicker();
    saveProjectState();
}
```

### Shape Application

```javascript
// ============================================
// PIP SHAPE APPLICATION
// Location: ~line 12900
// ============================================

const PIP_SHAPES = {
    landscape: { aspect: '16/9', radius: '4px' },
    portrait: { aspect: '9/16', radius: '4px' },
    square: { aspect: '1/1', radius: '4px' },
    circle: { aspect: '1/1', radius: '50%' }
};

function updatePipShape(pipId, shape) {
    window[`${pipId}Shape`] = shape;
    
    const container = document.getElementById(`${pipId}-container`);
    const config = PIP_SHAPES[shape];
    
    container.style.aspectRatio = config.aspect;
    container.style.borderRadius = config.radius;
    
    saveProjectState();
}
```

### Genie Zoom Animation

```css
/* ============================================
   GENIE ZOOM ANIMATIONS
   Location: ~line 1800 (CSS)
   ============================================ */

.pip-container {
    transform: scale(0);
    transition: transform 0.3s ease-out;
}

.pip-container.visible {
    transform: scale(1);
}

/* Transform origins per PIP */
#pip1-container { transform-origin: center; }
#pip2-container { transform-origin: top right; }
#pip3-container { transform-origin: center; }
```

---

## ORGANIZE TAB UX

### Apple Music-Style Auto-Save

```javascript
// ============================================
// AUTO-SAVE ON BLUR
// Location: initInfoDrawerAutoSave() ~line 13200
// ============================================

function initInfoDrawerAutoSave() {
    const fields = ['infoNameInput', 'infoTempoInput', 'infoLoopInInput', 'infoLoopOutInput'];
    
    fields.forEach(fieldId => {
        const field = document.getElementById(fieldId);
        
        field.addEventListener('blur', () => {
            saveCurrentFieldValue(fieldId);
        });
        
        field.addEventListener('keydown', (e) => {
            if (e.key === 'Enter') {
                e.preventDefault();
                saveCurrentFieldValue(fieldId);
                focusNextField(fieldId);
            } else if (e.key === 'Escape') {
                saveCurrentFieldValue(fieldId);
                closeInfoDrawer();
            } else if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
                saveCurrentFieldValue(fieldId);
                navigateToContainer(e.key === 'ArrowUp' ? -1 : 1);
            }
        });
    });
}

async function saveCurrentFieldValue(fieldId) {
    const container = getCurrentDrawerContainer();
    if (!container) return;
    
    const field = document.getElementById(fieldId);
    const value = field.value;
    
    // Apply change based on field
    switch (fieldId) {
        case 'infoNameInput':
            await renameContainer(container.id, value);
            break;
        case 'infoTempoInput':
            container.audioTempo = parseInt(value) || 120;
            await saveContainer(container);
            break;
        // ... other fields
    }
}
```

### Tag Autocomplete

```javascript
// ============================================
// TAG AUTOCOMPLETE SYSTEM
// Location: ~line 13400
// ============================================

let tagAutocompleteIndex = -1;
let tagSuggestions = [];

function showTagSuggestions(query) {
    if (!query || query.length < 1) {
        hideTagSuggestions();
        return;
    }
    
    const container = getCurrentDrawerContainer();
    const existingTags = container.tags || [];
    
    // Get all tags with usage counts
    const allTags = getTagUsageCounts();
    
    // Filter: contains query, not already on container
    tagSuggestions = allTags
        .filter(t => t.tag.toLowerCase().includes(query.toLowerCase()))
        .filter(t => !existingTags.includes(t.tag))
        .sort((a, b) => b.count - a.count)
        .slice(0, 8);  // Limit to 8 suggestions
    
    if (tagSuggestions.length === 0) {
        hideTagSuggestions();
        return;
    }
    
    // Render suggestions
    const dropdown = document.getElementById('tag-suggestions');
    dropdown.innerHTML = tagSuggestions.map((t, i) => `
        <div class="tag-suggestion ${i === tagAutocompleteIndex ? 'selected' : ''}"
             onclick="selectTagSuggestion(${i})">
            ${t.tag} <span class="tag-count">(${t.count})</span>
        </div>
    `).join('');
    
    dropdown.style.display = 'block';
}

function getTagUsageCounts() {
    const counts = {};
    
    libraryContainersCache.forEach(container => {
        (container.tags || []).forEach(tag => {
            counts[tag] = (counts[tag] || 0) + 1;
        });
    });
    
    return Object.entries(counts)
        .map(([tag, count]) => ({ tag, count }))
        .sort((a, b) => b.count - a.count);
}

function navigateTagSuggestions(direction) {
    if (tagSuggestions.length === 0) return;
    
    tagAutocompleteIndex += direction;
    
    // Wrap around
    if (tagAutocompleteIndex < 0) {
        tagAutocompleteIndex = tagSuggestions.length - 1;
    } else if (tagAutocompleteIndex >= tagSuggestions.length) {
        tagAutocompleteIndex = 0;
    }
    
    updateTagSuggestionHighlight();
}

function selectTagSuggestion(index) {
    if (index >= 0 && index < tagSuggestions.length) {
        const tag = tagSuggestions[index].tag;
        addTagToContainer(tag);
        document.getElementById('tag-input').value = '';
        hideTagSuggestions();
    }
}
```

### Selection Visual States

```css
/* ============================================
   SELECTION VISUAL STATES
   Location: ~line 750 (CSS)
   ============================================ */

/* Default state */
.library-card {
    border: 1px solid #333;
    background: rgba(255, 255, 255, 0.03);
}

/* Hover */
.library-card:hover {
    border-color: #00d4ff;
}

/* Selected */
.library-card.selected {
    border-color: #00d4ff;
    background: rgba(0, 212, 255, 0.1);
}

/* Loaded (in CREATE tab) */
.library-card.loaded {
    border-color: #ffcc00;
    background: rgba(255, 204, 0, 0.1);
}

/* Both selected AND loaded */
.library-card.selected.loaded {
    border-color: #ffcc00;
    background: rgba(255, 204, 0, 0.15);
}
```

---

## PERFORM KEYBOARD WORKFLOW

### Track Selection

```javascript
// ============================================
// BARE NUMBER KEY TRACK SELECTION
// Location: handlePerformKeyDown() ~line 14000
// ============================================

function handlePerformKeyDown(e) {
    // Bare number keys (no modifier) for track selection
    if (/^[1-4]$/.test(e.key) && !e.shiftKey && !e.ctrlKey && !e.metaKey) {
        e.preventDefault();
        const trackNum = parseInt(e.key);
        selectTrackByNumber(trackNum);
        return;
    }
    
    // Key 0 to deselect
    if (e.key === '0' && !e.shiftKey && !e.ctrlKey && !e.metaKey) {
        e.preventDefault();
        deselectTrack();
        return;
    }
    
    // SHIFT+Right for quick-load
    if (e.key === 'ArrowRight' && e.shiftKey) {
        e.preventDefault();
        quickLoadToNextCell();
        return;
    }
    
    // SHIFT+Left to return to library
    if (e.key === 'ArrowLeft' && e.shiftKey) {
        e.preventDefault();
        returnToLibraryFocus();
        return;
    }
    
    // Arrow up/down for library navigation (when track selected)
    if ((e.key === 'ArrowUp' || e.key === 'ArrowDown') && selectedTrackIndex !== null) {
        e.preventDefault();
        navigateLibraryFromPerform(e.key === 'ArrowUp' ? -1 : 1);
        return;
    }
}

function selectTrackByNumber(trackNum) {
    selectedTrackIndex = trackNum - 1;  // 0-indexed
    updateTrackSelectionUI();
    showTrackSelectionFeedback(trackNum);
    showMessage(`Track ${trackNum} selected`, 'state');
}

function deselectTrack() {
    selectedTrackIndex = null;
    updateTrackSelectionUI();
    showMessage('Track deselected', 'state');
}
```

### Quick Load Workflow

```javascript
// ============================================
// QUICK LOAD (SHIFT+RIGHT)
// Location: quickLoadToNextCell() ~line 14200
// ============================================

function quickLoadToNextCell() {
    // Get selected library container
    const selectedContainer = getSelectedLibraryContainer();
    if (!selectedContainer) {
        showMessage('No container selected in library', 'error');
        return;
    }
    
    // Find next empty cell
    const cell = findNextEmptyCell();
    if (!cell) {
        showMessage('No empty cells available', 'error');
        return;
    }
    
    // Load container to cell
    assignContainerToCell(cell.track, cell.scene, selectedContainer);
    
    // Advance library selection
    navigateLibraryFromPerform(1);
    
    showMessage(`Loaded to Track ${cell.track + 1} Scene ${cell.scene + 1}`, 'state');
}

function findNextEmptyCell() {
    // Scan order: horizontal by scene
    // Scene 1: T1S1 → T2S1 → T3S1 → T4S1
    // Scene 2: T1S2 → T2S2 → T3S2 → T4S2
    // ... etc
    
    for (let scene = 0; scene < 8; scene++) {
        for (let track = 0; track < 4; track++) {
            if (!gridState.cells[track][scene]) {
                return { track, scene };
            }
        }
    }
    
    return null;  // All cells occupied
}
```

### Critical Audio Timing Bug Fix

```javascript
// ============================================
// AUDIO TIMING BUG FIX (V5_4_17h)
// Location: assignContainerToCell() ~line 16500
// ============================================

function assignContainerToCell(track, scene, containerData) {
    // Store container in grid
    gridState.cells[track][scene] = containerData;
    
    // Update cell UI
    updateCellDisplay(track, scene, containerData);
    
    // CRITICAL FIX: Don't update trackContainerData if track is playing
    // This was causing wrong tempo when loading via SHIFT+Right
    if (!trackAudioPlaying[track]) {
        trackContainerData[track] = containerData;
    }
    // When track IS playing, launchCell() will set trackContainerData
    // from the actual container being launched
}

function launchCell(track, scene) {
    const containerData = gridState.cells[track][scene];
    if (!containerData) return;
    
    // Set trackContainerData from actual container (not grid state)
    // This ensures correct tempo regardless of how container was loaded
    trackContainerData[track] = containerData;
    
    // Continue with launch...
    const tempo = containerData.audioTempo || 120;
    const playbackRate = tempo / gridState.masterTempo;
    // ... etc
}
```

### Visual Feedback

```javascript
// ============================================
// TRACK SELECTION VISUAL FEEDBACK
// Location: ~line 14400
// ============================================

function showTrackSelectionFeedback(trackNum) {
    // Yellow highlight on track header
    const header = document.querySelector(`.track-header[data-track="${trackNum}"]`);
    header.classList.add('selected');
}

function showKnobRing(knobElement, color) {
    // Add ring overlay during adjustment
    const ring = document.createElement('div');
    ring.className = 'knob-ring';
    ring.style.setProperty('--ring-color', color);
    knobElement.parentElement.appendChild(ring);
}

function hideKnobRing(knobElement) {
    const ring = knobElement.parentElement.querySelector('.knob-ring');
    if (ring) ring.remove();
}

// Usage during arrow key holds:
// Hold ←/→: Purple ring on opacity knob
// Hold ↑/↓: Cyan ring on volume knob
```

---

## FX OVERLAY SYSTEM

### Toggle and State

```javascript
// ============================================
// FX OVERLAY STATE
// Location: ~line 2700
// ============================================

let fxVisible = false;
let trackVFXValues = [
    [0, 0, 0],  // Track 1: A, B, C
    [0, 0, 0],  // Track 2
    [0, 0, 0],  // Track 3
    [0, 0, 0]   // Track 4
];

function toggleFX() {
    fxVisible = !fxVisible;
    const overlay = document.getElementById('fx-overlay');
    overlay.style.display = fxVisible ? 'flex' : 'none';
    
    const btn = document.getElementById('fx-toggle-btn');
    btn.classList.toggle('active', fxVisible);
}
```

### Overlay Structure

```html
<!-- ============================================
     FX OVERLAY HTML STRUCTURE
     Location: ~line 3500 (HTML)
     ============================================ -->

<div id="fx-overlay" class="fx-overlay" style="display: none;">
    <!-- Track 1 -->
    <div class="fx-column" data-track="1">
        <div class="meter-module-wrap">
            <div class="meter-module-label">VFX</div>
            <div class="meter-module-knobs">
                <canvas id="track1VfxAKnob" width="20" height="20"></canvas>
                <canvas id="track1VfxBKnob" width="20" height="20"></canvas>
                <canvas id="track1VfxCKnob" width="20" height="20"></canvas>
            </div>
        </div>
        <div class="meter-module-wrap">
            <div class="meter-module-label">FX</div>
            <div class="meter-module-knobs">
                <canvas id="track1FxRKnob" width="20" height="20"></canvas>
                <canvas id="track1FxDKnob" width="20" height="20"></canvas>
                <canvas id="track1FxWKnob" width="20" height="20"></canvas>
            </div>
        </div>
    </div>
    <!-- Tracks 2-4 same structure -->
    <!-- Scenes column spacer -->
    <div class="fx-column scenes-spacer"></div>
</div>
```

### VFX Knob Drawing

```javascript
// ============================================
// VFX KNOB DRAWING (Placeholder)
// Location: drawVFXKnob() ~line 14600
// ============================================

function drawVFXKnob(canvas, value) {
    const ctx = canvas.getContext('2d');
    const w = canvas.width;
    const h = canvas.height;
    const cx = w / 2;
    const cy = h / 2;
    const r = (w - 4) / 2;
    
    ctx.clearRect(0, 0, w, h);
    
    // Background circle
    ctx.beginPath();
    ctx.arc(cx, cy, r, 0, Math.PI * 2);
    ctx.fillStyle = '#1a1a1a';
    ctx.fill();
    
    // Value arc (purple)
    if (value > 0) {
        const startAngle = -Math.PI * 0.75;
        const endAngle = startAngle + (Math.PI * 1.5 * value);
        
        ctx.beginPath();
        ctx.arc(cx, cy, r - 2, startAngle, endAngle);
        ctx.strokeStyle = '#a855f7';  // Purple
        ctx.lineWidth = 2;
        ctx.stroke();
    }
    
    // Center dot
    ctx.beginPath();
    ctx.arc(cx, cy, 2, 0, Math.PI * 2);
    ctx.fillStyle = '#666';
    ctx.fill();
}

// Note: VFX knobs are interactive placeholders
// Backend processing not yet connected
```

---

## SCHEMA ADDITIONS

### Container State (.oms) - V5_5_0 Additions

```javascript
// ============================================
// .OMS SCHEMA - V5_5_0 ADDITIONS
// ============================================

{
    "controlStates": {
        // Existing fields...
        
        // V5_5_0: Transform Animation Modes
        "transformRotationMode": "stop",    // stop|clockwise|counterclockwise|pingpong
        "transformRotationSpeed": 1.0,       // 0.03-256
        "transformScaleMode": "stop",
        "transformScaleSpeed": 1.0,
        "transformTileMode": "stop",
        "transformTileSpeed": 1.0,
        
        // V5_5_0: Per-Mod ADSR+T Envelopes (18 fields)
        "modAQ": 0.70,
        "modAAttack": 5,
        "modADecay": 20,
        "modASustain": 70,
        "modARelease": 50,
        "modAThreshold": 10,
        
        "modBQ": 0.70,
        "modBAttack": 10,
        "modBDecay": 30,
        "modBSustain": 80,
        "modBRelease": 100,
        "modBThreshold": 5,
        
        "modCQ": 0.70,
        "modCAttack": 2,
        "modCDecay": 15,
        "modCSustain": 60,
        "modCRelease": 80,
        "modCThreshold": 5
    }
}
```

### Project State (.omsp) - V5_5_0 Additions

```javascript
// ============================================
// .OMSP SCHEMA - V5_5_0 ADDITIONS
// ============================================

{
    // Existing fields...
    
    // V5_5_0: Video Sources Settings
    "videoSourcesSettings": {
        "cameraSourceId": "default",
        "cameraMirror": false
    },
    
    // V5_5_0: PIP Settings
    "pip1Settings": {
        "visible": false,
        "color": "#00d4ff",
        "label": "PIP 1",
        "shape": "landscape"
    },
    "pip2Settings": {
        "color": "#00d4ff",
        "shape": "landscape"
    },
    "pip3Settings": {
        "color": "#00d4ff",
        "shape": "square"
    }
}
```

---

## TESTING NOTES

### Transform Animation Tests

1. **Mode Keys:** 0/1/2/3 switch modes correctly
2. **Beat-Sync:** Animation stays locked to shaderLoopLength
3. **Speed Range:** Trackpad covers 0.03x-256x smoothly
4. **Persistence:** Modes/speeds saved to .oms and restored
5. **PERFORM Sync:** Container transform modes apply correctly

### Per-Mod Envelope Tests

1. **Row Sync:** N-panel updates when changing Reactivity row
2. **Value Update:** Knob changes update immediately
3. **Readouts:** ms/% values display correctly
4. **Persistence:** All 18 values saved to .oms
5. **Backward Compat:** Old containers use global defaults

### PIP Camera Tests

1. **Stream Start:** Camera permission requested once
2. **Smart Stop:** Stream only stops when all PIPs closed
3. **O Key:** Full-screen overlay works in all tabs
4. **P Key:** Context-aware (pip1 in PERFORM, pip2 elsewhere)
5. **Shift+P:** pip3 shows in CREATE/ORG/ADMIN
6. **Shapes:** All 4 shapes render correctly
7. **Colors:** Border-click picker works
8. **Genie Zoom:** Animations have correct origins
9. **Persistence:** All settings saved to .omsp

### ORGANIZE Tab Tests

1. **Auto-Save:** Changes save on blur
2. **Enter Key:** Saves and moves to next field
3. **Escape:** Saves and closes drawer
4. **Arrow Nav:** Saves and navigates containers
5. **Tag Autocomplete:** Shows matching tags with counts
6. **Selection States:** Cyan/yellow borders correct

### PERFORM Workflow Tests

1. **Bare Keys:** 1-4 select tracks without modifier
2. **Key 0:** Deselects track
3. **Arrow Nav:** ↑/↓ navigate library when track selected
4. **SHIFT+Right:** Loads to next empty cell
5. **SHIFT+Left:** Returns focus to library
6. **Visual Feedback:** Yellow header, colored rings
7. **Audio Timing:** Correct tempo regardless of load method

### FX Overlay Tests

1. **Toggle:** Button shows/hides overlay
2. **Knob Interaction:** Drag works on VFX knobs
3. **Arc Color:** Purple for VFX, cyan for FX
4. **Size:** All knobs 20×20px

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_5_0
Date: January 3, 2026
License: GPL-3.0
