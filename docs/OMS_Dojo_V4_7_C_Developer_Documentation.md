# ONEMANSHYO BETA V4_7_0 - DEVELOPER DOCUMENTATION

**Version:** Beta V4_7_0  
**Date:** December 12, 2025  
**Purpose:** Technical implementation reference for V4_7_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Message Bar System (CORTEX)](#message-bar-system-cortex)
3. [PERFORM Tab Controls](#perform-tab-controls)
4. [User-Created Sets](#user-created-sets)
5. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### Key Principle: Central Communication

V4_7_0 establishes the Message Bar as a CORTEX-level system:

```
┌─────────────────────────────────────────────────────────────┐
│  ALL MODULES                                                 │
│  (keyboard handlers, mouse handlers, system operations)      │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  showMessage(text, type)  ← SINGLE ENTRY POINT (CORTEX)     │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  <span id="messageBar" class="message-bar info">            │
│    Audio Volume (↑/↓)                                       │
│  </span>                                                    │
└─────────────────────────────────────────────────────────────┘
                      │
                      ▼
                    USER
```

### Key Principle: Semantic Color Coding

| Type | Color | Hex | Meaning |
|------|-------|-----|---------|
| `'info'` | Cyan | #00d4ff | Help text, keyboard shortcuts |
| `'state'` | Orange | #ffa500 | Current values, confirmations |
| `'error'` | Magenta | #ff006e | Failures, problems |

### Key Principle: Instant Feedback

- Keyboard actions → immediate state message
- Mouse hover → immediate info message
- Mouse leave → clear message
- No slow browser tooltips

---

## MESSAGE BAR SYSTEM (CORTEX)

### HTML Element

**Before V4_7_22:**
```html
<span id="shaderStatus" class="status">WebGPU Ready</span>
```

**After V4_7_22:**
```html
<span id="messageBar" class="message-bar">WebGPU Ready</span>
```

### CSS Classes

**Before V4_7_22:**
```css
.status { ... }
.status.info { color: #ffa500; }    /* Orange - wrong! */
.status.success { color: #00d4ff; } /* Cyan - wrong! */
.status.error { color: #ff006e; }
```

**After V4_7_22:**
```css
.message-bar {
    width: 100%;
    flex-shrink: 0;
    height: 30px;
    background: linear-gradient(180deg, #222, #2a2a2a);
    padding: 6px 20px;
    font-size: 13px;
    color: #00d4ff;
    border-top: 1px solid #444;
    text-align: center;
    font-weight: 500;
    display: flex;
    align-items: center;
    justify-content: center;
}

.message-bar.info { color: #00d4ff; }   /* Cyan - help */
.message-bar.state { color: #ffa500; }  /* Orange - state */
.message-bar.error { color: #ff006e; }  /* Magenta - error */
```

### Function

**Before V4_7_22:**
```javascript
function showStatus(message, type = '') {
    const status = document.getElementById('shaderStatus');
    status.textContent = message;
    status.className = 'status ' + type;
}

// Usage
showStatus('Track 1 selected', 'success');
```

**After V4_7_22:**
```javascript
// V4_7_22: Central message bar API - CORTEX
// Types: 'info' (cyan), 'state' (orange), 'error' (magenta)
function showMessage(text, type = '') {
    const bar = document.getElementById('messageBar');
    bar.textContent = text;
    bar.className = 'message-bar ' + type;
}

// Usage
showMessage('Track 1 selected', 'state');
```

### Usage Patterns

**Keyboard Action → State (Orange):**
```javascript
// Track selection
showMessage(`Track ${track} selected`, 'state');

// Value change
showMessage(`Track ${track} Shader: ${opacity}%`, 'state');

// State toggle
showMessage(`Track ${track} Audio: Muted (,)`, 'state');
```

**Mouse Hover → Info (Cyan):**
```javascript
// Knob hover
canvas.addEventListener('mouseenter', () => {
    showMessage('Audio Volume (↑/↓)', 'info');
});
canvas.addEventListener('mouseleave', () => {
    showMessage('', 'info');
});

// Button hover
btn.addEventListener('mouseenter', () => {
    showMessage('Audio Mute - Silence track (,)', 'info');
});
```

**Error → Error (Magenta):**
```javascript
// Operation failed
showMessage('Container not found', 'error');
showMessage('Failed to setup application folder', 'error');
```

---

## PERFORM TAB CONTROLS

### Keyboard Controls (28 Total)

**Track Selection (1-5):**
```javascript
// Line ~16670
if (key === '1' || key === '2' || key === '3' || key === '4') {
    selectedKnobTrack = parseInt(key);
    updateKnobSelection();
    showMessage(`Track ${selectedKnobTrack} selected`, 'state');
}
if (key === '5') {
    selectedKnobTrack = 5; // Master
    updateKnobSelection();
    showMessage('Master selected', 'state');
}
```

**Arrow Keys (Track Selected):**
```javascript
// Shader opacity
showMessage(`Track ${selectedKnobTrack} Shader: ${Math.round(opacity * 100)}%`, 'state');

// Audio volume
showMessage(`Track ${selectedKnobTrack} Volume: ${Math.round(volume * 100)}%`, 'state');
```

**Arrow Keys (Master Selected):**
```javascript
// Tempo
showMessage(`Tempo: ${gridState.masterTempo.toFixed(1)} BPM`, 'state');

// Master volume
showMessage(`Master Volume: ${Math.round(masterVolume * 100)}%`, 'state');
```

**Mute/Solo Keys:**
```javascript
// , key - Audio Mute
showMessage(`Track ${track} Audio: ${isMuted ? 'Muted' : 'Unmuted'} (,)`, 'state');

// . key - Audio Solo
showMessage(`Track ${track} Audio: Solo ${isSolo ? 'ON' : 'OFF'} (.)`, 'state');

// < key (Shift+,) - Shader Mute
showMessage(`Track ${track} Shader: ${isMuted ? 'Muted' : 'Unmuted'} (<)`, 'state');

// > key (Shift+.) - Shader Solo
showMessage(`Track ${track} Shader: Solo ${isSolo ? 'ON' : 'OFF'} (>)`, 'state');
```

**Grid Visibility (Shift+M/T/W):**
```javascript
// V4_7_22: Skip W key if Shift pressed (avoid WASD conflict)
if ((e.key === 'w' || e.key === 'W') && !e.shiftKey) {
    // WASD navigation
}

// Shift+M - Master Grid
showMessage(`Master Grid: ${state} (Shift+M)`, 'state');

// Shift+T - Track Grid
showMessage(`Track Grid: ${state} (Shift+T)`, 'state');

// Shift+W - Waveform
showMessage(`Waveform: ${state} (Shift+W)`, 'state');
```

**Zoom:**
```javascript
// [ key - Zoom out
// ] key - Zoom in
showMessage(`Zoom: ${gridState.zoomValue} ([/])`, 'state');
```

**Navigation:**
```javascript
// WASD - Cell movement
showMessage(`Cell: Track ${track + 1}, Scene ${scene + 1}`, 'state');

// Enter - Launch
showMessage(`Launching Track ${track + 1}, Scene ${scene + 1}`, 'state');

// Space - Play/Pause
showMessage('Paused', 'state');

// Escape - Stop/Scene mode
showMessage('Scene mode', 'state');
```

### Mouse Controls (16 Total)

**Knob Hovers (5):**
| Knob | Message |
|------|---------|
| Track Volume | "Audio Volume (↑/↓)" |
| Track Opacity | "Shader Opacity (←/→)" |
| Master Volume | "Master Volume (↑/↓)" |
| Master Opacity | "Master Shader Opacity" |
| Master Tempo | "Master Tempo (←/→)" |

**Button Hovers (11):**
| Button | Location | Message |
|--------|----------|---------|
| M (purple) | Track | "Audio Mute - Silence track (,)" |
| S (purple) | Track | "Audio Solo - Solo audio (.)" |
| M (cyan) | Track | "Shader Mute - Hide visuals (Shift+,)" |
| S (cyan) | Track | "Shader Solo - Solo visuals (Shift+.)" |
| M (orange) | Master | "Master Mute - Mute all tracks (,)" |
| S (grey) | Master | "Master Solo - Not applicable" |
| M (purple) | Master Shader | "Master Shader Mute - Mute all visuals (Shift+,)" |
| S (grey) | Master Shader | "Master Shader Solo - Not applicable" |
| M | Visibility | "Master Grid - Toggle overlay (Shift+M)" |
| T | Visibility | "Track Grid - Toggle beat markers (Shift+T)" |
| W | Visibility | "Waveform - Toggle amplitude bars (Shift+W)" |

---

## USER-CREATED SETS

### Data Structure

**Set Object:**
```javascript
{
    id: 'set_1702234567890',
    name: 'My Bass Set',
    containerIds: ['container1', 'container2', 'container3'],
    created: '2025-12-12T10:30:00Z'
}
```

**Storage:** Saved in .omsp project file

### Functions

**createSet(name):**
- Creates new set object
- Adds to userSetsCache
- Saves to project

**addToSet(setId, containerId):**
- Adds container to set's containerIds array
- Container can be in multiple sets

**removeFromSet(setId, containerId):**
- Removes container from set
- Doesn't delete container

**deleteSet(setId):**
- Removes set entirely
- Containers remain in library

---

## TESTING NOTES

### Message Bar Tests

1. **Info (Cyan):** Hover any knob or button
2. **State (Orange):** Press any key (1-5, arrows, etc.)
3. **Error (Magenta):** Try loading missing container
4. **Clear:** Mouse leave from control

### Keyboard Tests

1. **Track Selection:** Press 1-5, verify message
2. **Arrow Keys:** Hold arrow, verify continuous feedback
3. **Mute/Solo:** Press , . < >, verify toggle state
4. **Grid Toggles:** Press Shift+M/T/W, verify ON/OFF
5. **WASD:** Verify W still works for navigation

### Shift+W Fix Test

1. Enter PERFORM tab
2. Select a cell with WASD
3. Press Shift+W
4. Verify: Waveform toggles (not cell movement)
5. Press W (no shift)
6. Verify: Cell moves up

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V4_7_0
Date: December 12, 2025
License: GPL-3.0
