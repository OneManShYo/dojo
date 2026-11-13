# OMS SHADER BAKER V3.8.0 - DEVELOPER DOCUMENTATION

**Version:** V3.8.0  
**Date:** November 17, 2025  
**Purpose:** Technical reference for developers

---

## ARCHITECTURE

**File:** OMS_ShaderBaker_V3_8_0.html  
**Size:** 550KB  
**Lines:** ~12,615  
**Dependencies:** Zero

---

## WHAT CHANGED IN V3.8.0

### 1. Header System

**State:**
```javascript
let headerGreeting = "Hi Friend, I'm Wes!";
```

**Storage:**
```javascript
function getCurrentProjectState() {
    return {
        headerGreeting: headerGreeting  // V3.8.0: Added
    };
}
```

### 2. Ergonomic Keyboard

**Updated:**
```javascript
case 'ArrowLeft':  adjustShaderOpacity(selectedTrack, -0.02);
case 'ArrowRight': adjustShaderOpacity(selectedTrack, 0.02);
case 'BracketLeft':  window.updateZoom(-10);
case 'BracketRight': window.updateZoom(10);
```

### 3. Library Auto-Blur

```javascript
function blurLibraryCards() {
    const cards = document.querySelectorAll('.library-card');
    cards.forEach(card => card.blur());
}
```

### 4. Cell Selection Persistence

**Fixed:**
```javascript
// V3.8.0: Removed selectedCell = null
function selectTrack(track) {
    selectedTrack = track;
    // selectedCell = null; ← REMOVED
    updateTrackSelection();
}
```

### 5. Scene Repositioning

**Grid:**
```javascript
// V3.8.0: 5 columns (181px each)
const gridColumns = '181px 181px 181px 181px 181px';
```

### 6. Folder Overlay

```javascript
function checkFolderAccess() {
    if (!directoryHandle) {
        showFolderOverlay();
        return false;
    }
    return true;
}
```

### 7. Trackpad Zoom

```javascript
canvas.addEventListener('wheel', (e) => {
    if (e.ctrlKey) {
        e.preventDefault();
        const delta = e.deltaY > 0 ? -10 : 10;
        window.updateZoom(delta);
    }
}, { passive: false });
```

### 8. Preview Orientation

```javascript
function updateCanvasSize(orientationOverride) {
    const orientation = orientationOverride || 
                       document.getElementById('orientation').value;
    applyOrientation(mainCanvas, orientation);
}
```

---

## CORE SYSTEMS (V3.7.0 Foundation)

### Dual-Domain Mixer

**State:**
```javascript
// AUDIO
let trackGainNodes = [null, null, null, null];
let trackMute = [false, false, false, false];
let trackSolo = [false, false, false, false];

// SHADER
let trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];
let trackShaderMute = [false, false, false, false];
let trackShaderSolo = [false, false, false, false];
let trackShaderManual = [false, false, false, false];
```

---

## PERFORMANCE

**GPU Usage (4 tracks):**
- M1 MacBook Pro: ~30-45%
- Discrete GPU: ~20-30%
- Integrated GPU: ~50-60%

**Frame Rate:** 60 FPS stable  
**Memory:** ~250-350 MB with audio

---

## DEVELOPMENT

### Testing Checklist
1. ✅ WebGPU compilation
2. ✅ Audio upload
3. ✅ Multi-track playback
4. ✅ Keyboard controls
5. ✅ Project save/load
6. ✅ Pop-out window
7. ✅ Zoom (trackpad + brackets)
8. ✅ Cell persistence
9. ✅ Folder overlay

---

## API REFERENCE

**Zoom:**
```javascript
window.updateZoom(delta) // pixels (+/-)
```

**Project State:**
```javascript
getCurrentProjectState() // Returns Object
loadProjectState(state)  // Restores state
```

**Shader Opacity:**
```javascript
adjustShaderOpacity(track, delta)      // -1.0 to 1.0
calculateTrackShaderOpacity(track)     // Returns 0.0-1.0
```

---

**END OF DEVELOPER DOCUMENTATION**

Version: V3.8.0  
License: GPL-3.0
