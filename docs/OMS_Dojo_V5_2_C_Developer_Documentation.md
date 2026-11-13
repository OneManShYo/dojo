# ONEMANSHYO App [Beta V5_2_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_2_0  
**Date:** December 26, 2025  
**Purpose:** Technical implementation reference for V5_2_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Centralized Focus Mode System](#centralized-focus-mode-system)
3. [Audio Focus Mode](#audio-focus-mode)
4. [EQ3 System](#eq3-system)
5. [DJ FX System](#dj-fx-system)
6. [Cue Points System (Expanded)](#cue-points-system-expanded)
7. [Audio Controls Module](#audio-controls-module)
8. [Per-Track Audio Processing](#per-track-audio-processing)
9. [Tab Isolation Architecture](#tab-isolation-architecture)
10. [Project State Additions](#project-state-additions)
11. [CSS Additions](#css-additions)
12. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_2_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  CENTRALIZED FOCUS MODE (CORTEX)                                    │
│  Single activeFocusMode variable replaces 8+ booleans               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ activeFocusMode │ activeAudioPanel │ Compatibility Layer    │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  AUDIO CONTROLS MODULE (CREATE Tab)                                 │
│  Unified overlay panel for audio control                            │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Volume │ 5 Cues │ 3-Band EQ │ 3 DJ FX │ Audio Header        │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  PER-TRACK AUDIO PROCESSING (PERFORM Tab)                           │
│  Independent audio chain per track                                  │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ Track Modules │ EQ Filters │ FX Nodes │ Cue Points          │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **Centralized Focus Mode** - Single `activeFocusMode` with compatibility layer
2. **Gate + Panel Pattern** - Shift+A/1-4 gate, then V/C/E/F for panels
3. **Tab Isolation** - All keyboard handlers check `isCreateTabActive()` etc.
4. **Per-Track Independence** - Each PERFORM track has own EQ/FX chain

---

## CENTRALIZED FOCUS MODE SYSTEM

### Architecture (CORTEX)

This is **CORTEX** - every keyboard shortcut depends on it.

```javascript
// ============================================
// GLOBAL STATE - CENTRALIZED FOCUS MODE
// Location: ~line 22842
// ============================================

// Single source of truth
let activeFocusMode = 'none';
// Values: 'none', 'name', 'orientation', 'musicTiming', 
//         'transform', 'reactivity', 'audio', 'bumpr'

// Audio sub-panel (only when activeFocusMode === 'audio')
let activeAudioPanel = 'none';
// Values: 'none', 'volume', 'cue', 'eq', 'fx'
```

### Compatibility Layer (Critical)

Old boolean variables still work via `Object.defineProperty`:

```javascript
// ============================================
// COMPATIBILITY LAYER
// Location: ~line 22905
// ============================================

Object.defineProperty(window, 'shaderTransformFocus', {
    get: () => activeFocusMode === 'transform',
    set: (val) => { 
        if (val) activeFocusMode = 'transform'; 
        else if (activeFocusMode === 'transform') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'shaderReactivityFocus', {
    get: () => activeFocusMode === 'reactivity',
    set: (val) => { 
        if (val) activeFocusMode = 'reactivity'; 
        else if (activeFocusMode === 'reactivity') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'shaderMusicTimingFocus', {
    get: () => activeFocusMode === 'musicTiming',
    set: (val) => { 
        if (val) activeFocusMode = 'musicTiming'; 
        else if (activeFocusMode === 'musicTiming') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'shaderNameFocus', {
    get: () => activeFocusMode === 'name',
    set: (val) => { 
        if (val) activeFocusMode = 'name'; 
        else if (activeFocusMode === 'name') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'shaderOrientationFocus', {
    get: () => activeFocusMode === 'orientation',
    set: (val) => { 
        if (val) activeFocusMode = 'orientation'; 
        else if (activeFocusMode === 'orientation') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'createAudioFocus', {
    get: () => activeFocusMode === 'audio',
    set: (val) => { 
        if (val) activeFocusMode = 'audio'; 
        else if (activeFocusMode === 'audio') activeFocusMode = 'none'; 
    }
});

Object.defineProperty(window, 'bumprModeActive', {
    get: () => activeFocusMode === 'bumpr',
    set: (val) => { 
        if (val) activeFocusMode = 'bumpr'; 
        else if (activeFocusMode === 'bumpr') activeFocusMode = 'none'; 
    }
});
```

### Core Functions

```javascript
// ============================================
// FOCUS MODE - CORE FUNCTIONS
// Location: ~line 22846
// ============================================

function exitAllFocusModes() {
    const previousMode = activeFocusMode;
    const previousPanel = activeAudioPanel;
    
    // Mode-specific cleanup (selection rings, counters, etc.)
    switch (activeFocusMode) {
        case 'name':
            document.querySelectorAll('.name-selected').forEach(el => 
                el.classList.remove('name-selected'));
            break;
        case 'orientation':
            document.querySelectorAll('.orientation-selected').forEach(el => 
                el.classList.remove('orientation-selected'));
            break;
        case 'transform':
            transformKeyRepeatCount = 0;
            TRANSFORM_KNOB_IDS.forEach(id => {
                document.getElementById(id)?.classList.remove('mod-selected');
            });
            break;
        case 'reactivity':
            document.querySelectorAll('.mod-selected').forEach(el => 
                el.classList.remove('mod-selected'));
            break;
        case 'audio':
            // Clear audio panel visuals
            document.querySelectorAll('.audio-focus-selected').forEach(el => 
                el.classList.remove('audio-focus-selected'));
            clearEQSelection();
            clearFXSelection();
            clearCueSelection();
            clearVolumeSelection();
            break;
        case 'bumpr':
            const slideout = document.querySelector('.bumpr-slideout');
            if (slideout) slideout.classList.remove('active');
            break;
    }
    
    // Clear state
    activeFocusMode = 'none';
    activeAudioPanel = 'none';
    
    if (previousMode !== 'none') {
        showMessage('', 'info');
    }
}

function isAnyFocusModeActive() {
    return activeFocusMode !== 'none' || activeAudioPanel !== 'none';
}

function isFocusModeActive(mode) {
    return activeFocusMode === mode;
}

function setAudioPanel(panel) {
    // Must be in audio focus mode
    if (activeFocusMode !== 'audio') return;
    
    // Clear previous panel visuals
    if (activeAudioPanel !== 'none') {
        clearPanelSelection(activeAudioPanel);
    }
    
    activeAudioPanel = panel;
    
    // Show panel visuals
    if (panel !== 'none') {
        updatePanelSelection(panel);
    }
}
```

### Escape Handler Pattern

```javascript
// ============================================
// UNIFIED ESCAPE HANDLER
// Location: ~line 24200
// ============================================

if (e.key === 'Escape' && isAnyFocusModeActive()) {
    e.preventDefault();
    
    // Special case: audio panel nesting
    if (activeFocusMode === 'audio' && activeAudioPanel !== 'none') {
        // Exit panel but stay in audio focus
        setAudioPanel('none');
        showMessage('Audio Focus | V=Vol, C=Cue, E=EQ, F=FX, Esc=exit', 'info');
        return;
    }
    
    exitAllFocusModes();
    return;
}
```

---

## AUDIO FOCUS MODE

### Global State

```javascript
// ============================================
// GLOBAL STATE - AUDIO FOCUS
// Location: ~line 7400
// ============================================

// Column selection (12 columns)
let shaderAudioSelectedCol = 0;

// Progressive acceleration
let audioKeyRepeatCount = 0;
let audioLastKeyTime = 0;

// Panel state (via activeAudioPanel)
// let createAudioPanel = 'none';  // Now maps to activeAudioPanel
```

### Column Definitions

```javascript
// ============================================
// AUDIO FOCUS - COLUMNS
// ============================================

// Column 0-4: Cue points 1-5
// Column 5: Audio Start
// Column 6: Audio End
// Column 7: Tempo
// Column 8: Pitch
// Column 9: Loop In
// Column 10: Loop Out
// Column 11: Loop Toggle

const AUDIO_COL_COUNT = 12;
const AUDIO_CUE_COLS = [0, 1, 2, 3, 4];
const AUDIO_VALUE_COLS = [5, 6, 7, 8, 9, 10];
const AUDIO_TOGGLE_COL = 11;
```

### Keyboard Handler

```javascript
// ============================================
// AUDIO FOCUS - KEYBOARD
// Location: ~line 24485
// ============================================

// Entry: Shift+A
if (e.shiftKey && (e.key === 'a' || e.key === 'A')) {
    if (!isCreateTabActive()) return;
    e.preventDefault();
    
    if (createAudioFocus) {
        exitAllFocusModes();
    } else {
        exitAllFocusModes();  // Exit any other mode
        createAudioFocus = true;
        shaderAudioSelectedCol = 0;
        updateAudioSelection();
        showMessage('Audio Focus | V=Vol, C=Cue, E=EQ, F=FX, Esc=exit', 'info');
    }
    return;
}

// Panel keys (V/C/E/F) - only when in audio focus
if (createAudioFocus && activeAudioPanel === 'none') {
    if (e.key === 'v' || e.key === 'V') {
        e.preventDefault();
        setAudioPanel('volume');
        updateVolumeSelection();
        showMessage('Volume | ←→=adjust, Enter=kill, Esc=back', 'info');
        return;
    }
    if (e.key === 'c' || e.key === 'C') {
        e.preventDefault();
        setAudioPanel('cue');
        createCueSelectedRow = 0;
        updateCueSelection();
        showMessage('Cue | ↑↓=select, ←→=adjust, Enter=jump, Esc=back', 'info');
        return;
    }
    if (e.key === 'e' || e.key === 'E') {
        e.preventDefault();
        setAudioPanel('eq');
        createEQSelectedBand = 0;
        updateEQSelection();
        showMessage('EQ | ↑↓=H/M/L, ←→=adjust, Enter=kill, Esc=back', 'info');
        return;
    }
    if (e.key === 'f' || e.key === 'F') {
        e.preventDefault();
        setAudioPanel('fx');
        createFXSelectedSlot = 0;
        updateFXSelection();
        showMessage('FX | ↑↓=Washout/Dubby/Break, ←→=adjust, Enter=kill, Esc=back', 'info');
        return;
    }
}
```

---

## EQ3 SYSTEM

### Global State (CREATE Tab)

```javascript
// ============================================
// GLOBAL STATE - EQ3 (CREATE)
// Location: ~line 7450
// ============================================

// EQ Values (0-1, 0.75 = unity/0dB)
let eqHighValue = 0.75;
let eqMidValue = 0.75;
let eqLowValue = 0.75;

// Kill states
let eqHighKilled = false;
let eqMidKilled = false;
let eqLowKilled = false;

// Audio nodes
let eqHighFilter = null;  // BiquadFilterNode (highshelf)
let eqMidFilter = null;   // BiquadFilterNode (peaking)
let eqLowFilter = null;   // BiquadFilterNode (lowshelf)

// Keyboard navigation
let createEQSelectedBand = 0;  // 0=High, 1=Mid, 2=Low
let eqKeyRepeatCount = 0;
let eqLastKeyTime = 0;

// Drag state (bug fix V5_1_49)
let eqDraggingBand = -1;  // Which band is being dragged (-1 = none)
```

### Audio Routing

```javascript
// ============================================
// EQ3 - AUDIO CHAIN
// ============================================

// CREATE Tab routing:
// audioElement → EQ (High → Mid → Low) → FX → analyser → gainNode → destination

function initEQ3Filters() {
    if (!audioContext) return;
    
    // High shelf: 2500 Hz
    eqHighFilter = audioContext.createBiquadFilter();
    eqHighFilter.type = 'highshelf';
    eqHighFilter.frequency.value = 2500;
    
    // Mid peaking: 1000 Hz, Q=0.7
    eqMidFilter = audioContext.createBiquadFilter();
    eqMidFilter.type = 'peaking';
    eqMidFilter.frequency.value = 1000;
    eqMidFilter.Q.value = 0.7;
    
    // Low shelf: 250 Hz
    eqLowFilter = audioContext.createBiquadFilter();
    eqLowFilter.type = 'lowshelf';
    eqLowFilter.frequency.value = 250;
    
    // Chain: High → Mid → Low
    eqHighFilter.connect(eqMidFilter);
    eqMidFilter.connect(eqLowFilter);
}
```

### Value Mapping

```javascript
// ============================================
// EQ3 - VALUE MAPPING
// ============================================

// Ableton-style: 0.75 = unity (0dB), 1.0 = +6dB, 0.0 = -∞
function knobToDb(value) {
    if (value <= 0) return -Infinity;
    if (value <= 0.75) {
        // 0 to 0.75 → -∞ to 0dB (exponential)
        return 20 * Math.log10(value / 0.75);
    } else {
        // 0.75 to 1.0 → 0dB to +6dB (linear)
        return (value - 0.75) * 24;  // 0.25 range → 6dB
    }
}

function applyEQToFilters() {
    if (!eqHighFilter || !eqMidFilter || !eqLowFilter) return;
    
    // High
    if (eqHighKilled) {
        eqHighFilter.gain.value = -40;  // Effectively mute
    } else {
        eqHighFilter.gain.value = knobToDb(eqHighValue);
    }
    
    // Mid
    if (eqMidKilled) {
        eqMidFilter.gain.value = -40;
    } else {
        eqMidFilter.gain.value = knobToDb(eqMidValue);
    }
    
    // Low
    if (eqLowKilled) {
        eqLowFilter.gain.value = -40;
    } else {
        eqLowFilter.gain.value = knobToDb(eqLowValue);
    }
}
```

### Knob Drawing

```javascript
// ============================================
// EQ3 - KNOB DRAWING
// ============================================

function drawEQKnob(canvas, value, color, killed, selected) {
    const ctx = canvas.getContext('2d');
    const size = canvas.width;
    const cx = size / 2;
    const cy = size / 2;
    const radius = (size / 2) - 2;
    
    ctx.clearRect(0, 0, size, size);
    
    // Background
    ctx.beginPath();
    ctx.arc(cx, cy, radius, 0, Math.PI * 2);
    ctx.fillStyle = '#1a1a1a';
    ctx.fill();
    
    if (killed) {
        // Magenta fill when killed
        ctx.beginPath();
        ctx.arc(cx, cy, radius - 2, 0, Math.PI * 2);
        ctx.fillStyle = '#ff006e';
        ctx.fill();
    } else {
        // Cyan arc showing value
        const startAngle = Math.PI * 0.75;  // 7 o'clock
        const endAngle = startAngle + (value * Math.PI * 1.5);  // 270° range
        
        ctx.beginPath();
        ctx.arc(cx, cy, radius - 2, startAngle, endAngle);
        ctx.strokeStyle = color || '#00d4ff';
        ctx.lineWidth = 3;
        ctx.stroke();
    }
    
    // Selection ring
    if (selected) {
        ctx.beginPath();
        ctx.arc(cx, cy, radius, 0, Math.PI * 2);
        ctx.strokeStyle = '#00d4ff';
        ctx.lineWidth = 2;
        ctx.stroke();
    }
}
```

---

## DJ FX SYSTEM

### Global State (CREATE Tab)

```javascript
// ============================================
// GLOBAL STATE - DJ FX (CREATE)
// Location: ~line 7500
// V5_1_49: Renamed from Reverb/Delay/Wash
// ============================================

// FX Values (0-1, 0 = off, 1 = full wet)
let fxWashoutValue = 0.5;   // Was fxReverbValue
let fxDubbyValue = 0.5;     // Was fxDelayValue
let fxBreakValue = 0.5;     // Was fxWashValue

// Kill states
let fxWashoutKilled = false;
let fxDubbyKilled = false;
let fxBreakKilled = false;

// Audio nodes
let fxWashoutFilter = null;   // BiquadFilterNode (highpass)
let fxDubbyDelay = null;      // DelayNode
let fxDubbyFeedback = null;   // GainNode for feedback
let fxDubbyWet = null;        // GainNode for wet mix

// Keyboard navigation
let createFXSelectedSlot = 0;  // 0=Washout, 1=Dubby, 2=Break

// Drag state (bug fix V5_1_49)
let fxDraggingSlot = -1;  // Which slot is being dragged (-1 = none)
```

### FX Implementations

```javascript
// ============================================
// DJ FX - IMPLEMENTATIONS
// ============================================

// WASHOUT: Highpass filter sweep (20Hz → 2kHz)
function applyWashoutFilter() {
    if (!fxWashoutFilter) return;
    
    if (fxWashoutKilled || fxWashoutValue === 0) {
        fxWashoutFilter.frequency.value = 20;  // Bypass (let all through)
    } else {
        // Value 0→1 maps to 20Hz→2000Hz (exponential)
        const freq = 20 * Math.pow(100, fxWashoutValue);
        fxWashoutFilter.frequency.value = Math.min(freq, 2000);
    }
}

// DUBBY: Dub delay with feedback
function applyDubbyDelay() {
    if (!fxDubbyDelay || !fxDubbyFeedback || !fxDubbyWet) return;
    
    if (fxDubbyKilled) {
        fxDubbyWet.gain.value = 0;
        fxDubbyFeedback.gain.value = 0;
    } else {
        // Delay time: 375ms (3/16 at 120 BPM)
        fxDubbyDelay.delayTime.value = 0.375;
        // Feedback: 0.3 to 0.7 based on value
        fxDubbyFeedback.gain.value = 0.3 + (fxDubbyValue * 0.4);
        // Wet mix
        fxDubbyWet.gain.value = fxDubbyValue;
    }
}

// BREAK: Vinyl stop effect (playbackRate 1.0 → 0.1)
function applyBreakEffect() {
    const audioEl = document.getElementById('audioElement');
    if (!audioEl) return;
    
    if (fxBreakKilled || fxBreakValue === 0) {
        audioEl.playbackRate = 1.0;  // Normal speed
    } else {
        // Value 0→1 maps to playbackRate 1.0→0.1
        const targetRate = 1.0 - (fxBreakValue * 0.9);
        audioEl.playbackRate = Math.max(0.1, targetRate);
    }
}
```

### Audio Chain

```javascript
// ============================================
// DJ FX - AUDIO CHAIN
// ============================================

// CREATE Tab routing:
// audioElement → EQ → Washout → [Dubby dry/wet] → analyser → gain → destination
// Note: Break uses playbackRate directly (no nodes)

function initFXNodes() {
    if (!audioContext) return;
    
    // Washout: Highpass filter
    fxWashoutFilter = audioContext.createBiquadFilter();
    fxWashoutFilter.type = 'highpass';
    fxWashoutFilter.frequency.value = 20;  // Start at bypass
    
    // Dubby: Delay + feedback
    fxDubbyDelay = audioContext.createDelay(1.0);
    fxDubbyDelay.delayTime.value = 0.375;
    
    fxDubbyFeedback = audioContext.createGain();
    fxDubbyFeedback.gain.value = 0;
    
    fxDubbyWet = audioContext.createGain();
    fxDubbyWet.gain.value = 0;
    
    // Feedback loop
    fxDubbyDelay.connect(fxDubbyFeedback);
    fxDubbyFeedback.connect(fxDubbyDelay);
    fxDubbyDelay.connect(fxDubbyWet);
}
```

---

## CUE POINTS SYSTEM (EXPANDED)

### Global State

```javascript
// ============================================
// GLOBAL STATE - CUE POINTS
// Location: ~line 7350
// V5_1_16: Expanded from 3 to 5 cues
// ============================================

// Cue positions (beats relative to audioStartTime)
let audioCuePoints = [0, 64, 128, 192, 256];
// Default: Bar 1.1, 17.1, 33.1, 49.1, 65.1

// Drag state (CREATE tab)
let draggingCueIndex = -1;
let dragStartX = 0;
let cueDragged = false;

// Keyboard navigation
let createCueSelectedRow = 0;  // 0-4

// Constants
const CUE_HIT_TOLERANCE = 20;  // Pixels for click detection
const CUE_COUNT = 5;
```

### .oms Storage

```javascript
// ============================================
// CUE POINTS - .OMS STORAGE
// ============================================

// In buildContainerObject():
{
    "audioCuePoints": [0, 64, 128, 192, 256]
}

// In loadContainer():
function loadContainer(containerId) {
    // ...
    if (container.audioCuePoints) {
        audioCuePoints = [...container.audioCuePoints];
        // Pad legacy 3-cue containers to 5
        while (audioCuePoints.length < 5) {
            audioCuePoints.push(audioCuePoints.length * 64);
        }
    } else {
        // Default for containers without cues
        audioCuePoints = [0, 64, 128, 192, 256];
    }
}
```

### Cue Drawing

```javascript
// ============================================
// CUE POINTS - DRAWING
// ============================================

function drawCueFlags(ctx, canvas) {
    const audioStart = parseFloat(audioStartField.value) || 0;
    const tempo = parseFloat(audioTempoField.value) || 120;
    const pixelsPerBeat = getPixelsPerBeat();
    
    for (let i = 0; i < CUE_COUNT; i++) {
        const cueBeat = audioCuePoints[i];
        const cueX = cueBeat * pixelsPerBeat;
        
        // Orange flag (#ffa500)
        ctx.strokeStyle = '#ffa500';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(cueX, 0);
        ctx.lineTo(cueX, canvas.height);
        ctx.stroke();
        
        // Flag triangle
        ctx.fillStyle = '#ffa500';
        ctx.beginPath();
        ctx.moveTo(cueX, 0);
        ctx.lineTo(cueX + 12, 8);
        ctx.lineTo(cueX, 16);
        ctx.closePath();
        ctx.fill();
        
        // Cue number
        ctx.fillStyle = '#000';
        ctx.font = '10px monospace';
        ctx.fillText(String(i + 1), cueX + 2, 12);
    }
}
```

---

## AUDIO CONTROLS MODULE

### DOM Structure (CREATE Tab)

```html
<!-- ============================================
     AUDIO CONTROLS MODULE
     Location: Right edge of waveform container
     ============================================ -->

<div class="audio-controls-module">
    <!-- Volume Panel -->
    <div class="audio-panel volume-panel" style="display:none;">
        <div class="panel-header">VOLUME</div>
        <canvas id="createAudioVolumeKnob" width="24" height="24"></canvas>
    </div>
    
    <!-- Cue Panel -->
    <div class="audio-panel cue-panel" style="display:none;">
        <div class="panel-header">CUE</div>
        <div class="cue-buttons">
            <button class="cue-btn" data-cue="0">1</button>
            <button class="cue-btn" data-cue="1">2</button>
            <button class="cue-btn" data-cue="2">3</button>
            <button class="cue-btn" data-cue="3">4</button>
            <button class="cue-btn" data-cue="4">5</button>
        </div>
    </div>
    
    <!-- EQ Panel -->
    <div class="audio-panel eq-panel" style="display:none;">
        <div class="panel-header">EQ</div>
        <div class="eq-knobs">
            <div class="eq-row"><span>H</span><canvas id="eqKnobHigh" width="24" height="24"></canvas></div>
            <div class="eq-row"><span>M</span><canvas id="eqKnobMid" width="24" height="24"></canvas></div>
            <div class="eq-row"><span>L</span><canvas id="eqKnobLow" width="24" height="24"></canvas></div>
        </div>
    </div>
    
    <!-- FX Panel -->
    <div class="audio-panel fx-panel" style="display:none;">
        <div class="panel-header">FX</div>
        <div class="fx-knobs">
            <div class="fx-row"><span>W</span><canvas id="fxKnobWashout" width="24" height="24"></canvas></div>
            <div class="fx-row"><span>D</span><canvas id="fxKnobDubby" width="24" height="24"></canvas></div>
            <div class="fx-row"><span>B</span><canvas id="fxKnobBreak" width="24" height="24"></canvas></div>
        </div>
    </div>
</div>
```

### Volume Knob (CREATE Tab)

```javascript
// ============================================
// VOLUME KNOB - CREATE TAB
// Location: ~line 8500
// ============================================

let createAudioVolume = 0.5;
let createAudioVolumeKilled = false;
let createAudioGainNode = null;

function initCreateAudioVolumeKnob() {
    const canvas = document.getElementById('createAudioVolumeKnob');
    if (!canvas) return;
    
    // Mouse wheel
    canvas.addEventListener('wheel', (e) => {
        e.preventDefault();
        const delta = e.deltaY > 0 ? -0.05 : 0.05;
        createAudioVolume = Math.max(0, Math.min(1, createAudioVolume + delta));
        applyCreateAudioVolume();
        drawCreateAudioVolumeKnob();
    });
    
    // Click to kill
    canvas.addEventListener('click', () => {
        toggleCreateAudioVolumeKill();
    });
    
    drawCreateAudioVolumeKnob();
}

function drawCreateAudioVolumeKnob() {
    const canvas = document.getElementById('createAudioVolumeKnob');
    const selected = activeAudioPanel === 'volume';
    drawEQKnob(canvas, createAudioVolume, '#00d4ff', createAudioVolumeKilled, selected);
}

function applyCreateAudioVolume() {
    if (!createAudioGainNode) return;
    
    if (createAudioVolumeKilled) {
        createAudioGainNode.gain.value = 0;
    } else {
        createAudioGainNode.gain.value = createAudioVolume;
    }
}

function toggleCreateAudioVolumeKill() {
    createAudioVolumeKilled = !createAudioVolumeKilled;
    applyCreateAudioVolume();
    drawCreateAudioVolumeKnob();
    showMessage(createAudioVolumeKilled ? 'Volume: KILLED' : `Volume: ${Math.round(createAudioVolume * 100)}%`, 'state');
}
```

---

## PER-TRACK AUDIO PROCESSING

### Global State (PERFORM Tab)

```javascript
// ============================================
// GLOBAL STATE - PER-TRACK AUDIO (PERFORM)
// Location: ~line 7600
// ============================================

// Panel state
let performActivePanel = 'none';  // 'none', 'cue', 'eq', 'fx'
let performEQSelectedBand = 0;    // 0=High, 1=Mid, 2=Low
let performFXSelectedSlot = 0;    // 0=Washout, 1=Dubby, 2=Break
let performCueSelectedRow = 0;    // 0-4

// Per-Track EQ [4 tracks][3 bands]
let trackEQValues = [
    [0.75, 0.75, 0.75],  // Track 1: H, M, L
    [0.75, 0.75, 0.75],  // Track 2
    [0.75, 0.75, 0.75],  // Track 3
    [0.75, 0.75, 0.75]   // Track 4
];
let trackEQKilled = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
    [false, false, false]
];

// Per-Track FX [4 tracks][3 slots]
let trackFXValues = [
    [0.5, 0.5, 0.5],  // Track 1: Washout, Dubby, Break
    [0.5, 0.5, 0.5],  // Track 2
    [0.5, 0.5, 0.5],  // Track 3
    [0.5, 0.5, 0.5]   // Track 4
];
let trackFXKilled = [
    [false, false, false],
    [false, false, false],
    [false, false, false],
    [false, false, false]
];

// Per-Track Cues [4 tracks][5 cues]
let trackCuePoints = [
    [0, 64, 128, 192, 256],  // Track 1
    [0, 64, 128, 192, 256],  // Track 2
    [0, 64, 128, 192, 256],  // Track 3
    [0, 64, 128, 192, 256]   // Track 4
];

// Audio nodes (arrays of 4)
let trackEQFilters = [];       // [{high, mid, low}, ...]
let trackFXWashFilter = [];
let trackFXDelayNodes = [];
let trackFXDelayFeedback = [];
let trackFXDelayWet = [];
```

### Per-Track Audio Routing

```javascript
// ============================================
// PER-TRACK AUDIO ROUTING
// ============================================

function initTrackAudioRouting(trackIndex) {
    // Create EQ filters for this track
    trackEQFilters[trackIndex] = {
        high: audioContext.createBiquadFilter(),
        mid: audioContext.createBiquadFilter(),
        low: audioContext.createBiquadFilter()
    };
    
    // Configure EQ
    trackEQFilters[trackIndex].high.type = 'highshelf';
    trackEQFilters[trackIndex].high.frequency.value = 2500;
    
    trackEQFilters[trackIndex].mid.type = 'peaking';
    trackEQFilters[trackIndex].mid.frequency.value = 1000;
    trackEQFilters[trackIndex].mid.Q.value = 0.7;
    
    trackEQFilters[trackIndex].low.type = 'lowshelf';
    trackEQFilters[trackIndex].low.frequency.value = 250;
    
    // Create FX nodes
    trackFXWashFilter[trackIndex] = audioContext.createBiquadFilter();
    trackFXWashFilter[trackIndex].type = 'highpass';
    trackFXWashFilter[trackIndex].frequency.value = 20;
    
    trackFXDelayNodes[trackIndex] = audioContext.createDelay(1.0);
    trackFXDelayNodes[trackIndex].delayTime.value = 0.375;
    
    trackFXDelayFeedback[trackIndex] = audioContext.createGain();
    trackFXDelayFeedback[trackIndex].gain.value = 0;
    
    trackFXDelayWet[trackIndex] = audioContext.createGain();
    trackFXDelayWet[trackIndex].gain.value = 0;
    
    // Chain: Source → EQ (H→M→L) → Washout → [Dubby] → Analyser → Gain → Master
    // (Connection happens in playCellAudio)
}

// In playCellAudio():
// source → trackEQFilters[t].high → .mid → .low → trackFXWashFilter[t] → 
// [dubby dry/wet split] → trackAnalysers[t] → trackGainNodes[t] → masterGain
```

### PERFORM Keyboard Handler

```javascript
// ============================================
// PERFORM - KEYBOARD (TRACK MODULES)
// Location: ~line 25500
// ============================================

// Track selection: Shift+1-4
if (e.shiftKey && ['1', '2', '3', '4'].includes(e.key)) {
    if (!isPerformTabActive()) return;
    e.preventDefault();
    
    const oldTrack = selectedTrack;
    selectedTrack = parseInt(e.key) - 1;
    
    // Clear panel highlights from old track
    if (performActivePanel !== 'none') {
        clearPerformPanelHighlights(oldTrack);
        updatePerformPanelHighlights();
    }
    
    showMessage(`Track ${selectedTrack + 1} selected`, 'state');
    return;
}

// Cue jump: 1-5 (no gate needed)
if (isPerformTabActive() && ['1', '2', '3', '4', '5'].includes(e.key) && !e.shiftKey) {
    e.preventDefault();
    const cueIndex = parseInt(e.key) - 1;
    jumpToTrackCue(selectedTrack + 1, cueIndex);
    return;
}

// Panel keys: C/E/F (after track selection)
if (isPerformTabActive() && selectedTrack >= 0 && selectedTrack < 4) {
    if (e.key === 'c' || e.key === 'C') {
        e.preventDefault();
        performActivePanel = 'cue';
        performCueSelectedRow = 0;
        updatePerformCueSelection();
        showMessage(`Track ${selectedTrack + 1} Cue | ↑↓=select, Enter=jump, Esc=back`, 'info');
        return;
    }
    if (e.key === 'e' || e.key === 'E') {
        e.preventDefault();
        performActivePanel = 'eq';
        performEQSelectedBand = 0;
        updatePerformEQSelection();
        showMessage(`Track ${selectedTrack + 1} EQ | ↑↓=H/M/L, ←→=adjust, Enter=kill, Esc=back`, 'info');
        return;
    }
    if (e.key === 'f' || e.key === 'F') {
        e.preventDefault();
        performActivePanel = 'fx';
        performFXSelectedSlot = 0;
        updatePerformFXSelection();
        showMessage(`Track ${selectedTrack + 1} FX | ↑↓=W/D/B, ←→=adjust, Enter=kill, Esc=back`, 'info');
        return;
    }
}
```

---

## TAB ISOLATION ARCHITECTURE

### Tab Helper Functions (CORTEX)

```javascript
// ============================================
// TAB HELPERS (CORTEX)
// Location: ~line 7200
// ============================================

function isCreateTabActive() {
    const content = document.getElementById('controlsTabContent');
    return content && content.style.display !== 'none';
}

function isPerformTabActive() {
    const mixer = document.getElementById('liveMixerContainer');
    return mixer && mixer.offsetParent !== null;
}

function isOrganizeTabActive() {
    const content = document.getElementById('libraryTabContent');
    return content && content.style.display !== 'none';
}

function isAdminTabActive() {
    const content = document.getElementById('adminTabContent');
    return content && content.style.display !== 'none';
}
```

### Keyboard Handler Pattern

```javascript
// ============================================
// TAB ISOLATION - PATTERN
// ============================================

// CORRECT: All handlers check tab first
if (e.shiftKey && (e.key === 'a' || e.key === 'A')) {
    if (!isCreateTabActive()) return;  // Gate by tab
    e.preventDefault();
    // ... handle audio focus
}

if (e.key === 'e' || e.key === 'E') {
    if (isCreateTabActive() && createAudioFocus) {
        // CREATE tab EQ panel
    } else if (isPerformTabActive() && selectedTrack >= 0) {
        // PERFORM tab EQ panel
    }
    // Neither active = do nothing
}

// WRONG: No tab check (causes cross-tab interference)
if (e.key === 'e') {
    setAudioPanel('eq');  // Would fire on ANY tab!
}
```

---

## PROJECT STATE ADDITIONS

### onemanshyo.omsp

```javascript
// ============================================
// PROJECT STATE - V5_2_0 ADDITIONS
// ============================================

function getCurrentProjectState() {
    return {
        // ... existing fields ...
        
        // V5_2_0: Per-track EQ/FX (PERFORM)
        trackEQValues: trackEQValues.map(arr => [...arr]),
        trackEQKilled: trackEQKilled.map(arr => [...arr]),
        trackFXValues: trackFXValues.map(arr => [...arr]),
        trackFXKilled: trackFXKilled.map(arr => [...arr]),
        trackCuePoints: trackCuePoints.map(arr => [...arr]),
        
        // V5_2_0: CREATE tab audio state
        createAudioVolume: createAudioVolume,
        createAudioVolumeKilled: createAudioVolumeKilled,
        eqHighValue: eqHighValue,
        eqMidValue: eqMidValue,
        eqLowValue: eqLowValue,
        eqHighKilled: eqHighKilled,
        eqMidKilled: eqMidKilled,
        eqLowKilled: eqLowKilled,
        fxWashoutValue: fxWashoutValue,
        fxDubbyValue: fxDubbyValue,
        fxBreakValue: fxBreakValue,
        fxWashoutKilled: fxWashoutKilled,
        fxDubbyKilled: fxDubbyKilled,
        fxBreakKilled: fxBreakKilled,
    };
}
```

### .oms Containers

```javascript
// ============================================
// .OMS CONTAINER - V5_2_0 CHANGES
// ============================================

{
    "name": "My Container",
    "shaderCode": "...",
    "audioFilename": "track.mp3",
    "audioTempo": 128,
    "audioStartTime": 17.1,
    "audioCuePoints": [0, 64, 128, 192, 256],  // V5_2_0: 5 cues (was 3)
    // EQ/FX not stored in container (session state only)
}
```

---

## CSS ADDITIONS

### Audio Controls Module

```css
/* ============================================
   AUDIO CONTROLS MODULE
   ============================================ */

.audio-controls-module {
    position: absolute;
    right: 0;
    top: 0;
    bottom: 0;
    width: 115px;
    background: rgba(30, 30, 30, 0.95);
    border-left: 1px solid #333;
    display: flex;
    flex-direction: column;
    z-index: 20;
}

.audio-panel {
    padding: 8px;
    border-bottom: 1px solid #333;
}

.panel-header {
    font-size: 9px;
    color: #666;
    text-transform: uppercase;
    margin-bottom: 6px;
}
```

### EQ/FX Knobs

```css
/* ============================================
   EQ/FX KNOB SYSTEM
   ============================================ */

.eq-knobs, .fx-knobs {
    display: flex;
    flex-direction: column;
    gap: 4px;
}

.eq-row, .fx-row {
    display: flex;
    align-items: center;
    gap: 6px;
}

.eq-row span, .fx-row span {
    font-size: 10px;
    color: #888;
    width: 14px;
}

.eq-knob-canvas, .fx-knob-canvas {
    width: 24px;
    height: 24px;
    cursor: pointer;
}

.mod-selected {
    box-shadow: 0 0 0 2px #00d4ff;
    border-radius: 50%;
}
```

### Track Modules (PERFORM)

```css
/* ============================================
   TRACK MODULES (PERFORM)
   ============================================ */

.track-module {
    width: 115px;
    background: rgba(25, 25, 25, 0.95);
    border-left: 1px solid #333;
    display: flex;
    flex-direction: column;
    padding: 4px;
}

.track-cue-row, .track-eq-row, .track-fx-row {
    display: flex;
    align-items: center;
    gap: 4px;
    padding: 2px 0;
}

.track-cue-btn {
    width: 18px;
    height: 18px;
    font-size: 10px;
    background: rgba(255, 165, 0, 0.1);
    border: 1px solid #ffa500;
    color: #ffa500;
    cursor: pointer;
}

.track-cue-btn:hover {
    background: rgba(255, 165, 0, 0.2);
}

.track-cue-btn.selected {
    background: #ffa500;
    color: #000;
}
```

---

## TESTING NOTES

### Centralized Focus Mode Tests

1. **Entry/Exit:** Each Shift+Key enters mode, Escape exits
2. **Mutual Exclusivity:** Entering new mode exits previous
3. **Compatibility:** Old variable names (`shaderTransformFocus`, etc.) still work
4. **Selection Rings:** Only one ring visible at a time
5. **Dropdown Blocking:** Dropdowns don't open during focus modes

### Audio Focus Mode Tests

1. **Shift+A:** Toggles audio focus on/off
2. **V/C/E/F:** Opens sub-panels when in audio focus
3. **Escape Nesting:** Panel → Audio Focus → Normal (two presses)
4. **Navigation:** ←→ navigates columns, ↑↓ navigates rows in panels

### EQ3 Tests

1. **Knob Drag:** Drag ONE knob, only that knob moves
2. **Kill:** Click/Enter kills band (magenta fill)
3. **Audio:** EQ audibly affects playback
4. **Unity:** 0.75 value = no change to audio

### DJ FX Tests

1. **Washout:** Sweeps bass out as value increases
2. **Dubby:** Dub delay trails increase with value
3. **Break:** Audio slows down (vinyl stop effect)
4. **Independence:** Each FX works independently

### Per-Track Tests (PERFORM)

1. **Shift+1-4:** Selects track
2. **C/E/F:** Opens panel for selected track
3. **EQ:** Track EQ affects only that track
4. **FX:** Track FX affects only that track
5. **Cue Jump:** 1-5 keys jump to cue on selected track

### Tab Isolation Tests

1. **CREATE keys:** Don't trigger on PERFORM tab
2. **PERFORM keys:** Don't trigger on CREATE tab
3. **Focus modes:** Exit when switching tabs

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_2_0
Date: December 26, 2025
License: GPL-3.0
