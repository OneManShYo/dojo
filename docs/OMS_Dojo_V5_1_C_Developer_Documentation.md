# ONEMANSHYO App [Beta V5_1_0] - DEVELOPER DOCUMENTATION

**Version:** Beta V5_1_0  
**Date:** December 24, 2025  
**Purpose:** Technical implementation reference for V5_1_0 changes

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [BumpR StickRs Module](#bumpr-stickrs-module)
3. [Cue Points System](#cue-points-system)
4. [PERFORM Keyboard Reorganization](#perform-keyboard-reorganization)
5. [Container Output Refinements](#container-output-refinements)
6. [Project State Additions](#project-state-additions)
7. [CSS Additions](#css-additions)
8. [Testing Notes](#testing-notes)

---

## ARCHITECTURE OVERVIEW

### V5_1_0 Key Changes

```
┌─────────────────────────────────────────────────────────────────────┐
│  BUMPR STICKRS MODULE                                               │
│  Video overlay system with alpha channel support                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ File Handling │ Playback │ Timing │ Slideout UI │ Keyboard  │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  CUE POINTS SYSTEM                                                  │
│  CDJ-style hot cues with visual flags                               │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │ CREATE Flags │ Drag/Jump │ .oms Storage │ PERFORM Jump      │   │
│  └─────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

### Key Principles

1. **BumpR Mode (Shift+B)** - Dedicated keyboard mode for video triggering
2. **CDJ Cue Behavior** - Retrigger replaces current (no toggle)
3. **Tab-Aware Timing** - Quantization uses correct clock source per tab
4. **Mode Mutual Exclusivity** - Only one keyboard mode active at a time

---

## BUMPR STICKRS MODULE

### Folder Structure

```
[Application Folder]/
├── Library/
├── Presets/
├── BumpRStickRs/         ← NEW (auto-created on first drop)
│   ├── logo.webm
│   ├── brand_hit.webm
│   └── outro.webm
└── onemanshyo.omsp
```

### Global State Variables

```javascript
// ============================================
// GLOBAL STATE - BUMPR STICKRS
// Location: ~line 7316
// ============================================

// Folder handle
let bumprStickrsFolderHandle = null;

// Slot data (10 slots)
let bumprStickrsSlots = [null, null, null, null, null, null, null, null, null, null];
// Each slot: { filename: string, thumbnail: dataURL }

// Per-slot settings (10 arrays)
let bumprTimingModes = ['instant', 'instant', 'instant', 'instant', 'instant', 
                        'instant', 'instant', 'instant', 'instant', 'instant'];
let bumprScaleModes = [100, 100, 100, 100, 100, 100, 100, 100, 100, 100];
let bumprAnimateModes = ['none', 'none', 'none', 'none', 'none', 
                         'none', 'none', 'none', 'none', 'none'];
let bumprDurationModes = ['original', 'original', 'original', 'original', 'original',
                          'original', 'original', 'original', 'original', 'original'];

// Keyboard mode state
let bumprModeActive = false;
let bumprSelectedSlot = 0;  // 0-9

// Playback state
let bumprPlaying = false;
let bumprQueuedTimeout = null;
let bumprActiveVideoEl = null;
```

### DOM Elements

```html
<!-- CREATE Tab - Video overlay -->
<div id="preview-container">
    <canvas id="canvas">...</canvas>
    <video id="bumprOverlayVideo" 
           style="position:absolute; top:0; left:0; width:100%; height:100%; 
                  object-fit:contain; z-index:100; pointer-events:none; display:none;">
    </video>
    <div class="bumpr-slideout">...</div>
</div>

<!-- PERFORM Tab - Video overlay -->
<div id="liveShaderOutputContainer">
    <canvas id="liveShaderCanvas0">...</canvas>
    <!-- ... canvases 1-3 ... -->
    <video id="bumprOverlayVideoLive" 
           style="position:absolute; top:0; left:0; width:100%; height:100%; 
                  object-fit:contain; z-index:100; pointer-events:none; display:none;">
    </video>
</div>
```

### File Handling Functions

```javascript
// ============================================
// BUMPR - FILE HANDLING
// ============================================

async function handleBumprSlotDrop(slotIndex, event) {
    // Validate WebM
    const file = event.dataTransfer.files[0];
    if (!file.name.toLowerCase().endsWith('.webm')) {
        showMessage('BumpR StickRs: WebM files only', 'error');
        return;
    }
    
    // Create folder if needed
    if (!bumprStickrsFolderHandle) {
        bumprStickrsFolderHandle = await applicationFolderHandle.getDirectoryHandle(
            'BumpRStickRs', { create: true }
        );
    }
    
    // Copy file to folder
    const fileHandle = await bumprStickrsFolderHandle.getFileHandle(
        file.name, { create: true }
    );
    const writable = await fileHandle.createWritable();
    await writable.write(file);
    await writable.close();
    
    // Generate thumbnail (160×90)
    const thumbnail = await generateBumprThumbnail(file);
    
    // Update state
    bumprStickrsSlots[slotIndex] = {
        filename: file.name,
        thumbnail: thumbnail
    };
    
    updateBumprSlotUI(slotIndex);
    autoSaveProjectState();
}

async function generateBumprThumbnail(file) {
    return new Promise((resolve) => {
        const video = document.createElement('video');
        const url = URL.createObjectURL(file);
        video.src = url;
        video.muted = true;
        
        video.addEventListener('loadeddata', () => {
            // Seek to 10% or 0.5s
            video.currentTime = Math.min(video.duration * 0.1, 0.5);
        });
        
        video.addEventListener('seeked', () => {
            const canvas = document.createElement('canvas');
            canvas.width = 160;
            canvas.height = 90;
            const ctx = canvas.getContext('2d');
            
            // Dark background for alpha visibility
            ctx.fillStyle = '#111';
            ctx.fillRect(0, 0, 160, 90);
            ctx.drawImage(video, 0, 0, 160, 90);
            
            URL.revokeObjectURL(url);
            resolve(canvas.toDataURL('image/png'));
        });
    });
}

function updateBumprSlotUI(slotIndex) {
    const slot = bumprStickrsSlots[slotIndex];
    const thumbEl = document.getElementById(`bumprThumbnail${slotIndex + 1}`);
    const nameEl = document.getElementById(`bumprFilename${slotIndex + 1}`);
    const removeBtn = document.getElementById(`bumprRemove${slotIndex + 1}`);
    
    if (slot) {
        thumbEl.style.backgroundImage = `url(${slot.thumbnail})`;
        thumbEl.textContent = '';
        nameEl.textContent = slot.filename;
        removeBtn.disabled = false;
    } else {
        thumbEl.style.backgroundImage = '';
        thumbEl.textContent = 'Empty';
        nameEl.textContent = '—';
        removeBtn.disabled = true;
    }
}

function removeBumprSlot(slotIndex) {
    bumprStickrsSlots[slotIndex] = null;
    updateBumprSlotUI(slotIndex);
    autoSaveProjectState();
}
```

### Playback Functions

```javascript
// ============================================
// BUMPR - PLAYBACK
// ============================================

function triggerBumprStickr(slotIndex) {
    const slot = bumprStickrsSlots[slotIndex];
    if (!slot) {
        showMessage(`BumpR Slot ${slotIndex + 1}: Empty`, 'error');
        return;
    }
    
    // CDJ cue behavior: stop current, start new
    if (bumprPlaying || bumprQueuedTimeout) {
        stopBumprVideo();
    }
    
    const timing = bumprTimingModes[slotIndex];
    if (timing === 'instant') {
        playBumprVideo(slotIndex);
    } else {
        // Quantized timing
        const delayMs = getBumprDelay(slotIndex);
        if (delayMs > 0) {
            showMessage(`BumpR: Queued (${timing})`, 'state');
            bumprQueuedTimeout = setTimeout(() => {
                bumprQueuedTimeout = null;
                playBumprVideo(slotIndex);
            }, delayMs);
        } else {
            playBumprVideo(slotIndex);
        }
    }
}

function getBumprDelay(slotIndex) {
    const timing = bumprTimingModes[slotIndex];
    if (timing === 'instant') return 0;
    
    let tempo, currentBeat;
    
    // Tab-aware timing source
    const isLiveTab = document.getElementById('liveTabContent')?.classList.contains('active');
    
    if (isLiveTab) {
        // PERFORM: Use master clock
        if (!trackMasterClockStart) {
            showMessage('Start playback for quantized timing', 'info');
            return 0;
        }
        tempo = gridState.masterTempo || 120;
        currentBeat = trackAudioMasterClock();
    } else {
        // CREATE: Use audio playback
        const audioEl = document.getElementById('audioElement');
        if (!audioEl || audioEl.paused) {
            showMessage('Start audio for quantized timing', 'info');
            return 0;
        }
        const audioTempo = parseFloat(document.getElementById('audioTempoField').value) || 120;
        const audioStart = parseFloat(document.getElementById('audioStartField').value) || 0;
        tempo = audioTempo;
        currentBeat = (audioEl.currentTime - audioStart) * (tempo / 60);
    }
    
    // Calculate next boundary
    const beatsPerBar = 4;
    let targetBeat;
    
    if (timing === 'beat') {
        targetBeat = Math.ceil(currentBeat);
    } else {  // 'bar'
        targetBeat = Math.ceil(currentBeat / beatsPerBar) * beatsPerBar;
    }
    
    const beatsUntilTarget = targetBeat - currentBeat;
    const secondsUntilTarget = beatsUntilTarget * (60 / tempo);
    
    return Math.max(0, secondsUntilTarget * 1000);
}

async function playBumprVideo(slotIndex) {
    const slot = bumprStickrsSlots[slotIndex];
    if (!slot) return;
    
    bumprPlaying = true;
    
    // Get video file
    const fileHandle = await bumprStickrsFolderHandle.getFileHandle(slot.filename);
    const file = await fileHandle.getFile();
    const url = URL.createObjectURL(file);
    
    // Select correct video element
    const isLiveTab = document.getElementById('liveTabContent')?.classList.contains('active');
    const videoEl = isLiveTab 
        ? document.getElementById('bumprOverlayVideoLive')
        : document.getElementById('bumprOverlayVideo');
    
    bumprActiveVideoEl = videoEl;
    
    // Configure playback
    videoEl.src = url;
    videoEl.style.display = 'block';
    
    // Apply scale
    const scale = bumprScaleModes[slotIndex] / 100;
    videoEl.style.setProperty('--bumpr-scale', scale);
    videoEl.style.transform = scale === 1 ? '' : `scale(${scale})`;
    
    // Apply animation
    const animate = bumprAnimateModes[slotIndex];
    if (animate !== 'none') {
        const animConfigs = {
            bounce: { duration: '0.5s', iteration: 'infinite' },
            breathe: { duration: '4s', iteration: 'infinite' },
            crazy: { duration: '0.8s', iteration: 'infinite' },
            drift: { duration: '4s', iteration: 'infinite' },
            drop: { duration: '0.6s', iteration: '1', fill: 'forwards' },
            float: { duration: '2s', iteration: 'infinite' },
            glitch: { duration: '0.2s', iteration: 'infinite' },
            pop: { duration: '0.4s', iteration: '1', fill: 'forwards' },
            pulse: { duration: '1s', iteration: 'infinite' },
            shake: { duration: '0.3s', iteration: 'infinite' },
            spin: { duration: '1s', iteration: 'infinite' },
            spiral: { duration: '2s', iteration: 'infinite', direction: 'alternate' },
            strobe: { duration: '0.1s', iteration: 'infinite' },
            wobble: { duration: '0.5s', iteration: 'infinite' },
            zoom: { duration: '2s', iteration: 'infinite' }
        };
        const config = animConfigs[animate];
        videoEl.style.animation = `bumpr-${animate} ${config.duration} ease-in-out ${config.iteration}`;
        if (config.fill) videoEl.style.animationFillMode = config.fill;
        if (config.direction) videoEl.style.animationDirection = config.direction;
    }
    
    // Apply loop
    const duration = bumprDurationModes[slotIndex];
    videoEl.loop = (duration === 'loop');
    
    // Play
    await videoEl.play();
    showMessage(`BumpR: ${slot.filename}`, 'state');
    
    // Duration cutoff (if not loop or original)
    if (duration !== 'original' && duration !== 'loop') {
        const cutoffMs = calculateDurationMs(slotIndex);
        setTimeout(() => {
            if (bumprActiveVideoEl === videoEl) {
                cleanupBumprVideo(videoEl, url);
            }
        }, cutoffMs);
    }
    
    // Cleanup on end
    videoEl.onended = () => {
        if (!videoEl.loop) {
            cleanupBumprVideo(videoEl, url);
        }
    };
    
    videoEl.onerror = () => {
        cleanupBumprVideo(videoEl, url);
        showMessage('BumpR: Playback error', 'error');
    };
}

function stopBumprVideo() {
    if (bumprQueuedTimeout) {
        clearTimeout(bumprQueuedTimeout);
        bumprQueuedTimeout = null;
        showMessage('BumpR: Cancelled', 'state');
    }
    
    if (bumprActiveVideoEl) {
        const videoEl = bumprActiveVideoEl;
        videoEl.onended = null;
        videoEl.onerror = null;
        videoEl.pause();
        videoEl.removeAttribute('src');
        videoEl.style.display = 'none';
        videoEl.style.transform = '';
        videoEl.style.animation = '';
        videoEl.style.animationFillMode = '';
        videoEl.style.animationDirection = '';
        videoEl.loop = false;
        bumprActiveVideoEl = null;
        bumprPlaying = false;
    }
}

function cleanupBumprVideo(videoEl, url) {
    videoEl.onended = null;
    videoEl.onerror = null;
    videoEl.pause();
    videoEl.removeAttribute('src');
    videoEl.style.display = 'none';
    videoEl.style.transform = '';
    videoEl.style.animation = '';
    videoEl.style.animationFillMode = '';
    videoEl.style.animationDirection = '';
    videoEl.loop = false;
    URL.revokeObjectURL(url);
    bumprActiveVideoEl = null;
    bumprPlaying = false;
}
```

### Keyboard Mode Functions

```javascript
// ============================================
// BUMPR - KEYBOARD MODE (Shift+B)
// ============================================

function enterBumprMode() {
    // Exit other modes (mutual exclusivity)
    if (modReactivityMode) exitReactivityMode();
    if (transformKeyboardMode) exitTransformMode();
    
    bumprModeActive = true;
    bumprSelectedSlot = 0;
    
    // Show slideout
    const slideout = document.querySelector('.bumpr-slideout');
    if (slideout) slideout.classList.add('active');
    
    updateBumprSlideout();
    showMessage('BumpR Mode: ↑/↓ nav, Enter/1-0 trigger, Esc exit', 'info');
}

function exitBumprMode() {
    bumprModeActive = false;
    
    // Hide slideout
    const slideout = document.querySelector('.bumpr-slideout');
    if (slideout) slideout.classList.remove('active');
    
    showMessage('', 'info');
}

function updateBumprSlideout() {
    const container = document.querySelector('.bumpr-slideout-slots');
    if (!container) return;
    
    container.innerHTML = '';
    
    for (let i = 0; i < 10; i++) {
        const slot = bumprStickrsSlots[i];
        const keyLabel = i === 9 ? '0' : (i + 1).toString();
        
        const row = document.createElement('div');
        row.className = `bumpr-slot-row ${slot ? '' : 'empty'} ${i === bumprSelectedSlot ? 'selected' : ''}`;
        row.onclick = () => {
            if (slot) {
                bumprSelectedSlot = i;
                triggerBumprStickr(i);
            }
        };
        row.onmouseenter = () => showBumprSlotInfo(i);
        row.onmouseleave = () => clearBumprSlotInfo();
        
        row.innerHTML = `
            <div class="bumpr-slot-key">${keyLabel}</div>
            <div class="bumpr-slot-thumb" style="${slot ? `background-image:url(${slot.thumbnail})` : ''}">
                ${slot ? '' : '—'}
            </div>
        `;
        
        container.appendChild(row);
    }
}

function showBumprSlotInfo(slotIndex) {
    const slot = bumprStickrsSlots[slotIndex];
    const keyLabel = slotIndex === 9 ? '0' : (slotIndex + 1).toString();
    
    if (!slot) {
        showMessage(`[${keyLabel}] Empty`, 'info');
        return;
    }
    
    const timing = bumprTimingModes[slotIndex];
    const scale = bumprScaleModes[slotIndex];
    const animate = bumprAnimateModes[slotIndex];
    const duration = bumprDurationModes[slotIndex];
    
    const timingLabel = timing.charAt(0).toUpperCase() + timing.slice(1);
    const animLabel = animate === 'none' ? 'No Anim' : animate.charAt(0).toUpperCase() + animate.slice(1);
    const durLabel = duration === 'original' ? 'Full' : duration;
    
    showMessage(`[${keyLabel}] ${slot.filename} | ${timingLabel}, ${scale}%, ${animLabel}, ${durLabel}`, 'info');
}
```

### Keyboard Handler

```javascript
// ============================================
// BUMPR - KEYBOARD HANDLER
// Location: In main keydown listener
// ============================================

// Shift+B: Toggle BumpR mode
if (e.shiftKey && (e.key === 'b' || e.key === 'B')) {
    // Skip if in text field
    const activeEl = document.activeElement;
    if (activeEl?.tagName === 'INPUT' || activeEl?.tagName === 'TEXTAREA') return;
    
    // Skip if in ADMIN tab (allow typing capital B)
    const adminTab = document.getElementById('adminTabContent');
    if (adminTab?.classList.contains('active')) return;
    
    e.preventDefault();
    if (bumprModeActive) {
        exitBumprMode();
    } else {
        enterBumprMode();
    }
    return;
}

// BumpR mode active handlers
if (bumprModeActive) {
    // 1-9, 0: Trigger slots
    if (e.code >= 'Digit1' && e.code <= 'Digit9') {
        e.preventDefault();
        const slot = parseInt(e.code.slice(5)) - 1;  // Digit1 → 0
        bumprSelectedSlot = slot;
        updateBumprSlideout();
        triggerBumprStickr(slot);
        return;
    }
    if (e.code === 'Digit0') {
        e.preventDefault();
        bumprSelectedSlot = 9;
        updateBumprSlideout();
        triggerBumprStickr(9);
        return;
    }
    
    // Up/Down: Navigate
    if (e.key === 'ArrowUp') {
        e.preventDefault();
        bumprSelectedSlot = (bumprSelectedSlot - 1 + 10) % 10;
        updateBumprSlideout();
        showBumprSlotInfo(bumprSelectedSlot);
        return;
    }
    if (e.key === 'ArrowDown') {
        e.preventDefault();
        bumprSelectedSlot = (bumprSelectedSlot + 1) % 10;
        updateBumprSlideout();
        showBumprSlotInfo(bumprSelectedSlot);
        return;
    }
    
    // Enter: Trigger selected
    if (e.key === 'Enter') {
        e.preventDefault();
        triggerBumprStickr(bumprSelectedSlot);
        return;
    }
    
    // Escape: Exit
    if (e.key === 'Escape') {
        e.preventDefault();
        exitBumprMode();
        return;
    }
}
```

---

## CUE POINTS SYSTEM

### Global State

```javascript
// ============================================
// GLOBAL STATE - CUE POINTS
// ============================================

// Cue positions (beats relative to audioStartTime)
let audioCuePoints = [0, 64, 128];  // Default: 1.1, 17.1, 33.1

// Drag state
let draggingCueIndex = -1;     // -1 = not dragging, 0-2 = which cue
let dragStartX = 0;            // Start position for click vs drag detection
let cueDragged = false;        // True if mouse moved during interaction
const CUE_HIT_TOLERANCE = 20;  // Pixels for click detection
```

### Cue Drawing (CREATE Tab)

```javascript
// ============================================
// CUE POINTS - DRAWING
// In drawProgress() function
// ============================================

function drawCueFlags(ctx, canvas) {
    const audioStart = parseFloat(document.getElementById('audioStartField').value) || 0;
    const audioTempo = parseFloat(document.getElementById('audioTempoField').value) || 120;
    const beatsPerSecond = audioTempo / 60;
    const pixelsPerSecond = 50;  // Standard waveform zoom
    
    ctx.save();
    
    for (let i = 0; i < audioCuePoints.length; i++) {
        const cueBeat = audioCuePoints[i];
        const cueSeconds = cueBeat / beatsPerSecond;
        const cueX = (audioStart + cueSeconds) * pixelsPerSecond;
        
        // Line
        ctx.strokeStyle = 'rgba(255, 165, 0, 0.8)';  // Orange
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.moveTo(cueX, 0);
        ctx.lineTo(cueX, canvas.height);
        ctx.stroke();
        
        // Flag (triangle pointing right)
        ctx.fillStyle = 'rgba(255, 165, 0, 0.9)';
        ctx.beginPath();
        ctx.moveTo(cueX, 0);
        ctx.lineTo(cueX + 16, 12);
        ctx.lineTo(cueX, 24);
        ctx.closePath();
        ctx.fill();
        
        // Number
        ctx.fillStyle = '#000';
        ctx.font = 'bold 12px sans-serif';
        ctx.fillText((i + 1).toString(), cueX + 4, 16);
    }
    
    ctx.restore();
}
```

### Cue Drag/Jump Handlers

```javascript
// ============================================
// CUE POINTS - MOUSE HANDLERS
// In waveform mouse event listeners
// ============================================

function getCueAtPosition(x) {
    const audioStart = parseFloat(document.getElementById('audioStartField').value) || 0;
    const audioTempo = parseFloat(document.getElementById('audioTempoField').value) || 120;
    const beatsPerSecond = audioTempo / 60;
    const pixelsPerSecond = 50;
    
    for (let i = 0; i < audioCuePoints.length; i++) {
        const cueBeat = audioCuePoints[i];
        const cueSeconds = cueBeat / beatsPerSecond;
        const cueX = (audioStart + cueSeconds) * pixelsPerSecond;
        
        if (Math.abs(x - cueX) < CUE_HIT_TOLERANCE) {
            return i;
        }
    }
    return -1;
}

// mousedown handler
const cueIndex = getCueAtPosition(clickX);
if (cueIndex >= 0) {
    draggingCueIndex = cueIndex;
    dragStartX = e.clientX;
    cueDragged = false;
    e.preventDefault();
    return;
}

// mousemove handler
if (draggingCueIndex >= 0) {
    if (Math.abs(e.clientX - dragStartX) > 3) {
        cueDragged = true;
    }
    if (cueDragged) {
        updateCuePosition(e);
    }
}

// mouseup handler
if (draggingCueIndex >= 0) {
    if (!cueDragged) {
        // Click = jump to cue
        const cueBeat = audioCuePoints[draggingCueIndex];
        const audioStart = parseFloat(document.getElementById('audioStartField').value) || 0;
        const audioTempo = parseFloat(document.getElementById('audioTempoField').value) || 120;
        const cueSeconds = audioStart + (cueBeat * 60 / audioTempo);
        
        audioElement.currentTime = cueSeconds;
        showMessage(`Jump to Cue ${draggingCueIndex + 1} (${beatsToBarBeat(cueBeat)})`, 'state');
    } else {
        // Drag = show final position
        showMessage(`Cue ${draggingCueIndex + 1} set to ${beatsToBarBeat(audioCuePoints[draggingCueIndex])}`, 'state');
    }
    draggingCueIndex = -1;
    cueDragged = false;
}

function updateCuePosition(e) {
    const rect = waveformCanvas.getBoundingClientRect();
    const x = e.clientX - rect.left;
    
    const audioStart = parseFloat(document.getElementById('audioStartField').value) || 0;
    const audioTempo = parseFloat(document.getElementById('audioTempoField').value) || 120;
    const pixelsPerSecond = 50;
    
    const seconds = x / pixelsPerSecond;
    const beats = (seconds - audioStart) * (audioTempo / 60);
    
    // Snap to bar (4 beats)
    const snappedBeats = Math.round(beats / 4) * 4;
    
    // Clamp to valid range
    audioCuePoints[draggingCueIndex] = Math.max(0, snappedBeats);
    
    drawProgress();  // Redraw
}
```

### .oms Storage

```javascript
// ============================================
// CUE POINTS - .OMS STORAGE
// ============================================

// In buildContainerObject()
audioCuePoints: [...audioCuePoints],

// In loadContainer()
if (containerData.audioCuePoints && Array.isArray(containerData.audioCuePoints)) {
    audioCuePoints = [...containerData.audioCuePoints];
} else {
    audioCuePoints = [0, 64, 128];  // Legacy default
}

// In updateContainer()
audioCuePoints: audioExplicitlyRemoved ? [0, 64, 128] : 
    (audioLoaded ? [...audioCuePoints] : 
    (existingContainer.audioCuePoints || [0, 64, 128])),

// In removeAudio()
audioCuePoints = [0, 64, 128];
```

### PERFORM Tab Cue Jump

```javascript
// ============================================
// CUE POINTS - PERFORM TAB
// Keyboard handler for 1/2/3 keys
// ============================================

// In PERFORM keydown handler
if (!e.shiftKey && (e.key === '1' || e.key === '2' || e.key === '3')) {
    e.preventDefault();
    const cueIndex = parseInt(e.key) - 1;
    
    // Determine track
    let track = -1;
    if (selectedKnobIndex >= 0 && selectedKnobIndex <= 3) {
        track = selectedKnobIndex;
    } else if (selectedCell?.track >= 0) {
        track = selectedCell.track;
    }
    
    if (track < 0 || track > 3) {
        showMessage('Select a track first (Shift+1-4)', 'info');
        return;
    }
    
    // Get container data
    let containerData = trackContainerData[track];
    if (!containerData) {
        // Try to get from gridState for non-playing track
        const cellIndex = track * 8 + (selectedCell?.scene || 0);
        const cellData = gridState.cells[cellIndex];
        if (cellData?.id) {
            containerData = getLibraryContainers().find(c => c.id === cellData.id);
        }
    }
    
    if (!containerData?.audioCuePoints) {
        showMessage(`Track ${track + 1}: No cue points`, 'info');
        return;
    }
    
    const cueBeat = containerData.audioCuePoints[cueIndex] || 0;
    
    // Jump to cue
    if (trackAudioPlaying[track]) {
        // Stop and restart at cue position
        // ... implementation details
    } else {
        // Position waveform at cue
        const pixelsPerBar = getPixelsPerBar();
        const cuePixelX = (cueBeat / 4) * pixelsPerBar;
        const canvas = document.getElementById(`waveformCanvas${track + 1}`);
        const playheadX = canvas.parentElement.clientWidth / 2;
        canvas.style.transform = `translateX(${playheadX - cuePixelX}px)`;
    }
    
    showMessage(`Track ${track + 1}: Cue ${cueIndex + 1} (${beatsToBarBeat(cueBeat)})`, 'state');
}
```

---

## PERFORM KEYBOARD REORGANIZATION

### Changed Key Mappings

```javascript
// ============================================
// PERFORM TAB - KEYBOARD CHANGES
// ============================================

// BEFORE (V5_0_0)
// 1-5 = Select Track/Master

// AFTER (V5_1_0)
// Shift+1-5 = Select Track/Master (uses e.code for Shift compatibility)
// 1-3 = Jump to Cue (plain keys, no modifier)

// Track selection (now with Shift)
if (e.shiftKey && e.code >= 'Digit1' && e.code <= 'Digit5') {
    e.preventDefault();
    const index = parseInt(e.code.slice(5)) - 1;  // 0-4
    selectTrackOrMaster(index);
    return;
}

// Cue jump (plain keys)
if (!e.shiftKey && (e.key === '1' || e.key === '2' || e.key === '3')) {
    e.preventDefault();
    jumpToCue(parseInt(e.key) - 1);
    return;
}
```

### Why e.code Instead of e.key

```javascript
// Problem: Shift+1 produces '!' not '1'
// e.key === '!' when user presses Shift+1
// e.code === 'Digit1' regardless of Shift state

// WRONG
if (e.shiftKey && e.key >= '1' && e.key <= '5')  // Fails: '!' !== '1'

// CORRECT
if (e.shiftKey && e.code >= 'Digit1' && e.code <= 'Digit5')  // Works!
```

---

## CONTAINER OUTPUT REFINEMENTS

### Section Rename

```html
<!-- BEFORE -->
<div class="oms-header-bar">
    <span>OUTPUT</span>
</div>

<!-- AFTER -->
<div class="oms-header-bar">
    <span>CONTAINER OUTPUT</span>
    <div>
        <button id="saveContainerBtn">Save</button>
        <button id="updateContainerBtn" style="display:none">Update</button>
    </div>
</div>
```

### Save vs Update Logic

```javascript
// ============================================
// CONTAINER OUTPUT - SAVE/UPDATE
// ============================================

// Save = always creates NEW container
document.getElementById('saveContainerBtn').onclick = async () => {
    const name = document.getElementById('filename').value.trim();
    if (!name) {
        showMessage('Enter a container name', 'error');
        return;
    }
    
    const container = buildContainerObject(name);
    await saveContainerToOMS(container);
    
    // Set ID for future updates
    currentLoadedContainerId = name;
    document.getElementById('updateContainerBtn').style.display = '';
    
    // Switch to ORGANIZE tab
    switchTab('library');
    refreshLibraryList();
    selectContainerInLibrary(name);
    showMessage(`Saved: ${name}`, 'state');
};

// Update = saves to existing container
document.getElementById('updateContainerBtn').onclick = async () => {
    if (!currentLoadedContainerId) {
        showMessage('No container loaded', 'error');
        return;
    }
    
    await updateContainer(currentLoadedContainerId);
    showMessage(`Updated: ${currentLoadedContainerId}`, 'state');
    // Stays on CREATE tab (no navigation)
};

// Show Update button when container loaded
function loadContainer(containerId) {
    // ... load logic ...
    currentLoadedContainerId = containerId;
    document.getElementById('updateContainerBtn').style.display = '';
    document.getElementById('filename').value = containerId;
}
```

### CORTEX Hover Messages

```javascript
// ============================================
// CONTAINER OUTPUT - HOVER MESSAGES
// ============================================

// Header
document.querySelector('.container-output-header').onmouseenter = () => {
    showMessage('Container = Shader + Audio + Settings', 'info');
};

// Save button
document.getElementById('saveContainerBtn').onmouseenter = () => {
    showMessage('Save as NEW container to Library', 'info');
};

// Update button
document.getElementById('updateContainerBtn').onmouseenter = () => {
    showMessage('Update loaded container (keeps name)', 'info');
};

// Resolution dropdown
document.getElementById('resolutionDropdown').onmouseenter = () => {
    showMessage('Video resolution for render', 'info');
};

// FPS dropdown
document.getElementById('fpsDropdown').onmouseenter = () => {
    showMessage('Frame rate for render', 'info');
};

// Filename field
document.getElementById('filename').onmouseenter = () => {
    showMessage('Name for container and video file', 'info');
};

// Render button
document.getElementById('renderBtn').onmouseenter = () => {
    showMessage('Render video file', 'info');
};
```

---

## PROJECT STATE ADDITIONS

### onemanshyo.omsp

```javascript
// ============================================
// PROJECT STATE - NEW FIELDS
// ============================================

function getCurrentProjectState() {
    return {
        // ... existing fields ...
        
        // BumpR StickRs (V5_1_0)
        bumprStickrsSlots: bumprStickrsSlots.slice(),
        bumprTimingModes: bumprTimingModes.slice(),
        bumprScaleModes: bumprScaleModes.slice(),
        bumprAnimateModes: bumprAnimateModes.slice(),
        bumprDurationModes: bumprDurationModes.slice(),
    };
}

function applyProjectState(state) {
    // ... existing restore ...
    
    // BumpR StickRs (V5_1_0)
    if (state.bumprStickrsSlots) {
        bumprStickrsSlots = [...state.bumprStickrsSlots];
        // Pad to 10 if saved with fewer
        while (bumprStickrsSlots.length < 10) bumprStickrsSlots.push(null);
    }
    if (state.bumprTimingModes) {
        bumprTimingModes = [...state.bumprTimingModes];
        while (bumprTimingModes.length < 10) bumprTimingModes.push('instant');
    }
    if (state.bumprScaleModes) {
        bumprScaleModes = [...state.bumprScaleModes];
        while (bumprScaleModes.length < 10) bumprScaleModes.push(100);
    }
    if (state.bumprAnimateModes) {
        bumprAnimateModes = [...state.bumprAnimateModes];
        while (bumprAnimateModes.length < 10) bumprAnimateModes.push('none');
    }
    if (state.bumprDurationModes) {
        bumprDurationModes = [...state.bumprDurationModes];
        while (bumprDurationModes.length < 10) bumprDurationModes.push('original');
    }
    
    // Update BumpR UI
    for (let i = 0; i < 10; i++) {
        updateBumprSlotUI(i);
    }
}
```

### .oms Containers

```javascript
// ============================================
// .OMS CONTAINER - NEW FIELD
// ============================================

// audioCuePoints: Array of 3 beat positions
{
    "name": "My Container",
    "shaderCode": "...",
    "audioFilename": "track.mp3",
    "audioTempo": 128,
    "audioStartTime": 17.1,
    "audioCuePoints": [0, 64, 128],  // NEW (V5_1_0)
    // ...
}
```

---

## CSS ADDITIONS

### Animation Keyframes

```css
/* ============================================
   BUMPR STICKRS - ANIMATION KEYFRAMES
   ============================================ */

@keyframes bumpr-bounce {
    0%, 100% { transform: scale(var(--bumpr-scale, 1)); }
    50% { transform: scale(calc(var(--bumpr-scale, 1) * 1.15)); }
}

@keyframes bumpr-breathe {
    0%, 100% { transform: scale(var(--bumpr-scale, 1)); }
    50% { transform: scale(calc(var(--bumpr-scale, 1) * 1.05)); }
}

@keyframes bumpr-crazy {
    0% { transform: scale(var(--bumpr-scale, 1)) rotate(0deg) translate(0, 0); opacity: 1; }
    25% { transform: scale(calc(var(--bumpr-scale, 1) * 1.2)) rotate(5deg) translate(10px, -10px); opacity: 0.8; }
    50% { transform: scale(calc(var(--bumpr-scale, 1) * 0.9)) rotate(-3deg) translate(-15px, 5px); opacity: 1; }
    75% { transform: scale(calc(var(--bumpr-scale, 1) * 1.1)) rotate(2deg) translate(5px, 10px); opacity: 0.9; }
    100% { transform: scale(var(--bumpr-scale, 1)) rotate(0deg) translate(0, 0); opacity: 1; }
}

/* ... 12 more keyframes ... */
```

### BumpR Dropdown System

```css
/* ============================================
   BUMPR STICKRS - DROPDOWN SYSTEM
   ============================================ */

.bumpr-dropdown {
    position: relative;
    display: inline-block;
    cursor: pointer;
}

.bumpr-dropdown.scale { width: 50px; }
.bumpr-dropdown.timing { width: 55px; }
.bumpr-dropdown.animate { width: 65px; }
.bumpr-dropdown.duration { width: 60px; }

.bumpr-dropdown-label {
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: rgba(255,255,255,0.05);
    padding: 2px 6px;
    border-radius: 3px;
    font-size: 10px;
    height: 20px;
}

.bumpr-dropdown-options {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    background: #1a1a1a;
    border: 1px solid #333;
    z-index: 100;
    max-height: 200px;
    overflow-y: auto;
    min-width: 100%;
}

.bumpr-dropdown.open .bumpr-dropdown-options {
    display: block;
}

.bumpr-dropdown-item {
    padding: 4px 8px;
    font-size: 10px;
    cursor: pointer;
}

.bumpr-dropdown-item:hover {
    background: rgba(0, 212, 255, 0.1);
}
```

### Slideout Panel

```css
/* ============================================
   BUMPR STICKRS - SLIDEOUT PANEL
   ============================================ */

.bumpr-slideout {
    position: absolute;
    top: 0;
    bottom: 0;
    right: -85px;
    width: 80px;
    background: rgba(20, 20, 20, 0.95);
    border-left: 1px solid #333;
    z-index: 50;
    transition: right 0.2s ease;
    display: flex;
    flex-direction: column;
    padding: 8px 6px;
}

.bumpr-slideout.active {
    right: 0;
}

.bumpr-slideout-header {
    font-size: 8px;
    text-transform: uppercase;
    color: #666;
    margin-bottom: 4px;
    text-align: center;
}

.bumpr-slideout-slots {
    display: flex;
    flex-direction: column;
    gap: 3px;
    flex: 1;
}

.bumpr-slot-row {
    display: flex;
    align-items: center;
    gap: 4px;
    cursor: pointer;
    padding: 2px;
    border-radius: 2px;
}

.bumpr-slot-row:hover {
    background: rgba(255,255,255,0.05);
}

.bumpr-slot-row.selected .bumpr-slot-thumb {
    border-color: #00d4ff;
}

.bumpr-slot-row.empty {
    opacity: 0.3;
    cursor: default;
}

.bumpr-slot-key {
    font-size: 9px;
    color: #00d4ff;
    width: 10px;
    text-align: center;
}

.bumpr-slot-thumb {
    width: 68px;
    height: 32px;
    background-color: #222;
    background-size: cover;
    background-position: center;
    border: 1px solid #444;
    border-radius: 2px;
    font-size: 8px;
    color: #666;
    display: flex;
    align-items: center;
    justify-content: center;
}
```

---

## TESTING NOTES

### BumpR StickRs Tests

1. **File drop:** Drop WebM on slot → creates folder, copies file, shows thumbnail
2. **Keyboard mode:** Shift+B → slideout appears, 1-0 trigger slots
3. **CDJ behavior:** Trigger slot A, then B → A stops, B plays
4. **Quantized timing:** Set to "Bar", trigger → waits for next bar
5. **Duration cutoff:** Set to "2" bars → video stops after 2 bars
6. **Loop mode:** Set to "Loop" → video repeats until stopped
7. **Animation:** Set animation, trigger → animation plays
8. **Scale:** Set to 50% → video plays at half size
9. **Persistence:** Refresh page → slots and settings restored

### Cue Points Tests

1. **Visual flags:** Load audio → 3 orange flags at 1.1, 17.1, 33.1
2. **Click jump:** Click flag without dragging → playhead jumps
3. **Drag reposition:** Drag flag → moves with mouse, snaps to bar
4. **Storage:** Save container → reload → cue positions preserved
5. **PERFORM display:** Load to grid → flags visible on waveform
6. **PERFORM jump:** Select track (Shift+1), press 1 → jumps to cue

### Keyboard Reorganization Tests

1. **Shift+1-5:** Selects track/master in PERFORM
2. **1-3:** Jumps to cue on selected track
3. **Detection:** Shift+1 works (e.code not e.key)

### Mode Mutual Exclusivity

1. Enter Reactivity (Shift+R)
2. Press Shift+B → Reactivity exits, BumpR enters
3. Press Shift+T → BumpR exits, Transform enters

---

**END OF DEVELOPER DOCUMENTATION**

Version: Beta V5_1_0
Date: December 24, 2025
License: GPL-3.0
