# OMS SHADER BAKER V3.5.0 - DEVELOPER DOCUMENTATION

**Version:** V3.5.0 Production Release  
**Date:** November 16, 2025  
**Audience:** Developers, contributors, technical users

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Master Clock System](#master-clock-system)
3. [Keyboard Control System](#keyboard-control-system)
4. [Quantized Launching](#quantized-launching)
5. [State Management](#state-management)
6. [Visual Feedback](#visual-feedback)
7. [Audio System Integration](#audio-system-integration)
8. [File Formats](#file-formats)
9. [Development Workflow](#development-workflow)
10. [Testing Guidelines](#testing-guidelines)

---

## ARCHITECTURE OVERVIEW

### System Components

```
┌─────────────────────────────────────────────────────────┐
│                    OMS Shader Baker V3.5.0              │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │  Keyboard       │  │  Visual         │              │
│  │  Event System   │──│  Feedback       │              │
│  └─────────────────┘  └─────────────────┘              │
│           │                    │                        │
│           ▼                    ▼                        │
│  ┌─────────────────────────────────────┐               │
│  │      State Management              │               │
│  │  - Track selection                 │               │
│  │  - Scene/cell navigation           │               │
│  │  - Master clock reference          │               │
│  └─────────────────────────────────────┘               │
│           │                                             │
│           ▼                                             │
│  ┌─────────────────────────────────────┐               │
│  │     Master Clock System            │               │
│  │  - Timing reference                │               │
│  │  - Bar calculations                │               │
│  │  - Quantization logic              │               │
│  └─────────────────────────────────────┘               │
│           │                                             │
│           ▼                                             │
│  ┌─────────────────────────────────────┐               │
│  │     Web Audio API                  │               │
│  │  - Sample-accurate scheduling      │               │
│  │  - Playback rate adjustment        │               │
│  │  - Multi-track audio               │               │
│  └─────────────────────────────────────┘               │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Design Principles

**1. One Timeline Architecture (V3.3.0 Foundation)**
- All tracks use ONE master timeline reference
- Visual elements calculate from master tempo
- Audio playback rates adjust to match master tempo
- "Draw at target tempo from the start" - no CSS transform corrections

**2. Separation of Concerns**
- Keyboard events → State updates
- State updates → Visual rendering
- State updates → Audio configuration
- Visual rendering is read-only (never modifies state)

**3. Sample-Accurate Timing**
- Web Audio API scheduling for precision
- Bar boundaries calculated exactly
- Playback rate adjustments for tempo matching
- Zero accumulated drift

**4. Progressive Enhancement**
- Mouse controls work (V3.4.0 foundation)
- Keyboard controls add professional workflow (V3.5.0)
- Both methods coexist without conflict

---

## MASTER CLOCK SYSTEM

### Purpose

The master clock provides a **global timing reference** that all clips sync to when launching. This ensures "soldiers at the goalpost" alignment - all Bar 1s perfectly aligned regardless of when clips are triggered.

### Data Structure

```javascript
// Global state (gridState object)
gridState.masterClockStartTime = null;  // Float: AudioContext time when first clip launched
gridState.masterClockBeatIndex = 0;     // Integer: Current beat number (0, 1, 2, 3, 4, ...)
```

### Initialization

```javascript
function setMasterClock(audioContext, scheduledStartTime, trackBPM) {
    if (gridState.masterClockStartTime === null) {
        gridState.masterClockStartTime = scheduledStartTime;
        gridState.masterClockBeatIndex = 0;
        console.log(`[Master Clock] Set at ${scheduledStartTime.toFixed(3)}s, BPM: ${trackBPM}`);
    }
}
```

**When Called:**
- First clip to launch in a session
- After emergency stop (Esc key) that clears master clock
- When starting fresh performance

**Parameters:**
- `audioContext`: Web Audio API context (for time reference)
- `scheduledStartTime`: **ACTUAL** audio start time (currentTime + 0.2s), NOT currentTime
- `trackBPM`: BPM of the track (used for logging only, master tempo used for calculations)

**Critical:** Must pass scheduled start time, not current time, to avoid 200ms timing offset.

### Bar Calculation

```javascript
function getNextBarTime(audioContext, currentTime, trackBPM) {
    if (gridState.masterClockStartTime === null) {
        return currentTime + 0.2; // First clip launches immediately (200ms latency)
    }
    
    const masterTempo = gridState.masterTempo;
    const secondsPerBar = (4 / masterTempo) * 60;
    const elapsedTime = currentTime - gridState.masterClockStartTime;
    const currentBar = Math.floor(elapsedTime / secondsPerBar);
    const nextBar = currentBar + 1;
    const nextBarTime = gridState.masterClockStartTime + (nextBar * secondsPerBar);
    
    console.log(`[Quantize] Current bar: ${currentBar}, Next bar: ${nextBar}, Wait: ${(nextBarTime - currentTime).toFixed(3)}s`);
    
    return nextBarTime;
}
```

**How It Works:**

1. **Check if master clock exists:**
   - If `null`: First clip, launch immediately (200ms latency)
   - If set: Calculate next bar boundary

2. **Calculate seconds per bar:**
   - Formula: `(4 beats/bar ÷ BPM) × 60 seconds/minute`
   - Example: 120 BPM → (4/120) × 60 = 2.0 seconds per bar

3. **Calculate elapsed time:**
   - Formula: `currentTime - masterClockStartTime`
   - Example: Current 10.5s, start 8.3s → elapsed 2.2s

4. **Calculate current bar:**
   - Formula: `Math.floor(elapsedTime / secondsPerBar)`
   - Example: 2.2s / 2.0s per bar → floor(1.1) = bar 1

5. **Calculate next bar:**
   - Formula: `currentBar + 1`
   - Example: bar 1 + 1 = bar 2

6. **Calculate next bar time:**
   - Formula: `startTime + (nextBar × secondsPerBar)`
   - Example: 8.3s + (2 × 2.0s) = 12.3s

**Return Value:** AudioContext time when next bar starts

### Clearing Master Clock

```javascript
// Emergency stop (Esc key)
gridState.masterClockStartTime = null;
gridState.masterClockBeatIndex = 0;
```

**When to Clear:**
- Emergency stop (Esc key)
- Global stop command
- Session reset

**Effect:** Next clip to launch will start immediately and set new reference.

---

## KEYBOARD CONTROL SYSTEM

### Event Listener Architecture

```javascript
document.addEventListener('keydown', (e) => {
    // Prevent default browser behavior for controlled keys
    if (['1','2','3','4','5','Tab','ArrowUp','ArrowDown',' ','Escape',
         ',','.','w','s','a','d','Enter'].includes(e.key)) {
        e.preventDefault();
    }
    
    // Track selection
    if (e.key >= '1' && e.key <= '5') {
        const trackIndex = parseInt(e.key) - 1;
        updateSelectedTrack(trackIndex);
    }
    
    // Tab cycling
    if (e.key === 'Tab') {
        const direction = e.shiftKey ? -1 : 1;
        const newIndex = (gridState.selectedTrackIndex + direction + 5) % 5;
        updateSelectedTrack(newIndex);
    }
    
    // Volume control
    if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
        const delta = e.key === 'ArrowUp' ? 0.05 : -0.05;
        const trackIndex = gridState.selectedTrackIndex;
        const currentVolume = gridState.trackVolumes[trackIndex];
        const newVolume = Math.max(0, Math.min(1, currentVolume + delta));
        updateVolume(trackIndex, newVolume);
    }
    
    // Mute/Solo
    if (e.key === ',') toggleMute(gridState.selectedTrackIndex);
    if (e.key === '.') toggleSolo(gridState.selectedTrackIndex);
    
    // Scene navigation (default mode)
    if (!gridState.cellModeActive) {
        if (e.key === 'w' || e.key === 's') {
            const delta = e.key === 'w' ? -1 : 1;
            const newIndex = (gridState.selectedSceneIndex + delta + 8) % 8;
            updateSelectedScene(newIndex);
        }
        if (e.key === 'Enter') {
            launchScene(gridState.selectedSceneIndex);
        }
    }
    
    // Cell navigation (cell mode)
    if (e.key === 'a' || e.key === 'd') {
        gridState.cellModeActive = true;
        const delta = e.key === 'a' ? -1 : 1;
        gridState.selectedCellCol = (gridState.selectedCellCol + delta + 4) % 4;
        updateCellSelection();
    }
    
    if (gridState.cellModeActive) {
        if (e.key === 'w' || e.key === 's') {
            const delta = e.key === 'w' ? -1 : 1;
            gridState.selectedCellRow = (gridState.selectedCellRow + delta + 8) % 8;
            updateCellSelection();
        }
        if (e.key === 'Enter') {
            const cellIndex = gridState.selectedCellRow * 4 + gridState.selectedCellCol;
            launchTrack(gridState.selectedCellCol, cellIndex);
            gridState.cellModeActive = false;
        }
    }
    
    // Transport controls
    if (e.key === ' ') {
        // Global play/pause
        if (audioContext.state === 'running') {
            audioContext.suspend();
        } else {
            audioContext.resume();
        }
    }
    
    if (e.key === 'Escape') {
        // Emergency stop
        stopAllTracks();
        gridState.masterClockStartTime = null;
        gridState.masterClockBeatIndex = 0;
    }
});
```

### Key Bindings Reference

| Key | Function | Details |
|-----|----------|---------|
| **1-4** | Select Track | Track 1-4 |
| **5** | Select Master | Master track (volume control) |
| **Tab** | Cycle Forward | Next track (wraps) |
| **Shift+Tab** | Cycle Backward | Previous track (wraps) |
| **↑** | Volume Up | +5% (0.05 steps) |
| **↓** | Volume Down | -5% (0.05 steps) |
| **,** (comma) | Toggle Mute | On selected track |
| **.** (period) | Toggle Solo | On selected track |
| **W** | Up | Scene mode: prev scene; Cell mode: prev row |
| **S** | Down | Scene mode: next scene; Cell mode: next row |
| **A** | Left | Activate cell mode, prev column |
| **D** | Right | Activate cell mode, next column |
| **Enter** | Launch | Scene mode: all tracks; Cell mode: single track |
| **Space** | Play/Pause | Global suspend/resume |
| **Esc** | Emergency Stop | Kill all audio, clear master clock |

### Mode System

**Scene Mode (Default):**
- W/S navigate scenes (8 scenes total, wrapping)
- Enter launches entire scene (all 4 tracks)
- Visual: Cyan border on selected scene
- Exit: Never (always active unless cell mode)

**Cell Mode (Activated by A/D):**
- A/D navigate columns (4 tracks, wrapping)
- W/S navigate rows (8 scenes, wrapping)
- Enter launches single track
- Visual: Yellow inset box-shadow on selected cell
- Exit: After Enter (launch) or manual return to scene mode

**Mode Toggle:**
```javascript
// Activate cell mode
if (e.key === 'a' || e.key === 'd') {
    gridState.cellModeActive = true;
}

// Deactivate cell mode (after launch)
if (e.key === 'Enter' && gridState.cellModeActive) {
    launchTrack(...);
    gridState.cellModeActive = false;
}
```

---

## QUANTIZED LAUNCHING

### Launch Flow

```
User triggers launch
     ↓
Check master clock
     ↓
  ┌──────────────────────────┐
  │ Master clock exists?     │
  └──────────────────────────┘
      │                   │
      │ NO               │ YES
      ↓                   ↓
  Set master clock    Calculate next bar
  Launch immediately  Wait for boundary
      ↓                   ↓
      └─────────┬─────────┘
                ↓
     Schedule audio start
                ↓
     Adjust playback rate
                ↓
     Update visual state
                ↓
          PLAYING
```

### Implementation

```javascript
function launchTrack(trackIndex, cellIndex) {
    const cellData = gridState.cells[cellIndex];
    if (!cellData || !cellData.audioBuffer) return;
    
    const audioContext = trackAudioContexts[trackIndex];
    const currentTime = audioContext.currentTime;
    
    // Determine scheduled start time
    const scheduledStartTime = getNextBarTime(audioContext, currentTime, cellData.bpm);
    
    // Set master clock if first clip
    setMasterClock(audioContext, scheduledStartTime, cellData.bpm);
    
    // Create audio source
    const source = audioContext.createBufferSource();
    source.buffer = cellData.audioBuffer;
    source.loop = true;
    
    // Adjust playback rate for tempo matching
    const playbackRate = gridState.masterTempo / cellData.bpm;
    source.playbackRate.value = playbackRate;
    
    // Connect to audio graph
    source.connect(trackGainNodes[trackIndex]);
    
    // Schedule sample-accurate start
    source.start(scheduledStartTime);
    
    // Store reference for later stop
    trackAudioSources[trackIndex] = source;
    
    // Update visual state
    updateTrackPlayingState(trackIndex, cellIndex);
    
    console.log(`[Launch] Track ${trackIndex}, Cell ${cellIndex}, Start: ${scheduledStartTime.toFixed(3)}s, Rate: ${playbackRate.toFixed(3)}`);
}
```

### Timing Precision

**Web Audio API Scheduling:**
- `source.start(scheduledStartTime)` is sample-accurate
- Scheduled events happen EXACTLY at specified time
- No drift, no accumulated error
- Precision: ±0.001ms (one sample at 48kHz)

**Playback Rate Adjustment:**
- Formula: `masterTempo / trackBPM`
- Example: Master 120 BPM, track 128 BPM → rate 0.9375
- Audio plays slower (pitch-shifted down) to match master tempo
- Beat grid spacing unchanged (drawn at master tempo)

**Result:**
- All tracks play at master tempo speed
- All Bar 1s align at playhead ("soldiers at the goalpost")
- Perfect sync regardless of original track BPMs

---

## STATE MANAGEMENT

### Grid State Object

```javascript
const gridState = {
    // Performance grid (4 tracks × 8 scenes = 32 cells)
    cells: Array(32).fill(null),
    
    // Master tempo control
    masterTempo: 120,           // BPM (60-200)
    
    // Zoom control
    zoomValue: 68,              // pixels per second (10-200)
    
    // Track controls
    trackVolumes: [1, 1, 1, 1], // 0.0 to 1.0
    trackMutes: [false, false, false, false],
    trackSolos: [false, false, false, false],
    masterVolume: 1,            // 0.0 to 1.0
    
    // Keyboard control state (NEW in V3.5.0)
    selectedTrackIndex: 0,      // 0-3 = tracks, 4 = master
    selectedSceneIndex: 0,      // 0-7 (wraps around)
    selectedCellRow: 0,         // 0-7 (current scene in cell mode)
    selectedCellCol: 0,         // 0-3 (current track in cell mode)
    cellModeActive: false,      // Scene mode vs cell mode
    
    // Master clock (NEW in V3.5.0)
    masterClockStartTime: null, // AudioContext time reference
    masterClockBeatIndex: 0,    // Current beat number
};
```

### State Update Functions

**Track Selection:**
```javascript
function updateSelectedTrack(trackIndex) {
    gridState.selectedTrackIndex = trackIndex;
    
    // Redraw all volume knobs (to show/hide selection ring)
    for (let i = 0; i < 4; i++) {
        drawVolumeKnob(i, gridState.trackVolumes[i]);
    }
    drawMasterVolumeKnob(gridState.masterVolume);
}
```

**Volume Update:**
```javascript
function updateVolume(trackIndex, newVolume) {
    // Update state
    gridState.trackVolumes[trackIndex] = newVolume;
    
    // Update audio
    trackGainNodes[trackIndex].gain.value = newVolume;
    
    // Update visual
    drawVolumeKnob(trackIndex, newVolume);
}
```

**Scene Selection:**
```javascript
function updateSelectedScene(sceneIndex) {
    gridState.selectedSceneIndex = sceneIndex;
    
    // Update visual (cyan border on selected scene button)
    document.querySelectorAll('.scene-button').forEach((btn, idx) => {
        if (idx === sceneIndex) {
            btn.style.border = '2px solid cyan';
        } else {
            btn.style.border = '1px solid grey';
        }
    });
}
```

**Cell Selection:**
```javascript
function updateCellSelection() {
    const cellIndex = gridState.selectedCellRow * 4 + gridState.selectedCellCol;
    
    // Update visual (yellow inset box-shadow on selected cell)
    document.querySelectorAll('.grid-cell').forEach((cell, idx) => {
        if (idx === cellIndex) {
            cell.style.boxShadow = 'inset 0 0 0 3px rgba(255, 255, 0, 0.6)';
        } else {
            cell.style.boxShadow = 'none';
        }
    });
}
```

### State Persistence

**.omsp Project State Format:**
```javascript
{
    version: "3.5.0",
    gridState: {
        masterTempo: 120,
        zoomValue: 68,
        cells: [/* 32 cells with .oms references */],
        trackVolumes: [1, 1, 1, 1],
        trackMutes: [false, false, false, false],
        trackSolos: [false, false, false, false],
        masterVolume: 1,
        // Keyboard state NOT saved (resets on load)
    },
    trackData: [/* 4 tracks with audio timing data */]
}
```

**Note:** Keyboard selection state (selectedTrackIndex, selectedSceneIndex, etc.) is NOT persisted in .omsp files. Always starts at default (track 0, scene 0) on load.

---

## VISUAL FEEDBACK

### Selection Indicators

**Volume Knob Selection (Cyan Ring):**
```javascript
function drawVolumeKnob(trackIndex, volume) {
    // ... draw knob background, arc, pointer ...
    
    // Selection ring (if selected)
    if (trackIndex === gridState.selectedTrackIndex) {
        ctx.strokeStyle = 'cyan';
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius + 3, 0, Math.PI * 2);
        ctx.stroke();
    }
}
```

**Scene Selection (Cyan Border):**
```css
.scene-button {
    border: 1px solid grey; /* Default */
}

.scene-button.selected {
    border: 2px solid cyan; /* Selected */
}
```

**Cell Selection (Yellow Inset Box-Shadow):**
```css
.grid-cell.selected {
    box-shadow: inset 0 0 0 3px rgba(255, 255, 0, 0.6);
}
```

**Why Inset:** Using `inset` box-shadow instead of `outline` prevents the selection from appearing in the gap between grid cells. The highlight stays INSIDE the cell boundary.

### Visual Update Timing

**Synchronous Updates:**
- All visual feedback updates happen synchronously with keyboard events
- No setTimeout, no requestAnimationFrame for selection highlights
- User sees instant response (<16ms, typically <5ms)

**Example:**
```javascript
// Keydown event
e.key === '2' 
    ↓ (< 1ms)
updateSelectedTrack(1)
    ↓ (< 5ms)
drawVolumeKnob() for all tracks
    ↓ (< 1ms)
Visual update appears on screen
```

Total latency: <10ms (effectively instant to human perception)

---

## AUDIO SYSTEM INTEGRATION

### Multi-Track Architecture

**4 Independent Audio Contexts:**
```javascript
const trackAudioContexts = [
    new AudioContext({ sampleRate: 48000 }),
    new AudioContext({ sampleRate: 48000 }),
    new AudioContext({ sampleRate: 48000 }),
    new AudioContext({ sampleRate: 48000 })
];
```

**Why Separate Contexts:**
- Independent suspend/resume per track (future feature)
- Prevents audio crosstalk
- Better performance isolation
- Matches professional DJ software architecture

**Current Limitation:** Global play/pause (Space key) suspends all contexts simultaneously. Per-track pause planned for future release.

### Audio Graph Structure

**Per Track:**
```
AudioBufferSourceNode
        ↓
    GainNode (track volume)
        ↓
    GainNode (mute state)
        ↓
    GainNode (solo state)
        ↓
    AnalyserNode (for meters)
        ↓
    AudioContext.destination
```

**Master:**
```javascript
// Master volume affects all tracks
trackGainNodes.forEach(gain => {
    gain.gain.value = trackVolume * masterVolume;
});
```

### Playback Rate Adjustment

**Formula:**
```javascript
const playbackRate = masterTempo / trackBPM;
source.playbackRate.value = playbackRate;
```

**Examples:**
- Master 120 BPM, track 120 BPM → rate 1.0 (normal speed)
- Master 120 BPM, track 128 BPM → rate 0.9375 (slower)
- Master 140 BPM, track 120 BPM → rate 1.1667 (faster)

**Effect:**
- Audio pitch shifts (not time-stretched, preserves pitch)
- Beat grid spacing unchanged (drawn at master tempo)
- All tracks sync to master timeline

---

## FILE FORMATS

### .oms Container Format

```javascript
{
    shaderCode: "/* WGSL shader code */",
    audioData: "data:audio/mpeg;base64,...",  // Base64 audio
    audioFileName: "track.mp3",
    controlStates: {
        /* All shader control values */
    },
    bpm: 128,           // Track BPM
    loopStart: 0,       // Loop start in bars
    loopEnd: 8,         // Loop end in bars
    version: "3.5.0"
}
```

**Loading .oms:**
```javascript
async function loadOMSContainer(file) {
    const text = await file.text();
    const data = JSON.parse(text);
    
    // Decode audio
    const audioBlob = await fetch(data.audioData).then(r => r.blob());
    const audioBuffer = await decodeAudioFile(audioBlob);
    
    // Store in cell
    const cellData = {
        shaderCode: data.shaderCode,
        audioBuffer: audioBuffer,
        bpm: data.bpm,
        loopStart: data.loopStart,
        loopEnd: data.loopEnd,
        fileName: file.name
    };
    
    return cellData;
}
```

### .omsp Project State Format

```javascript
{
    version: "3.5.0",
    gridState: {
        masterTempo: 120,
        zoomValue: 68,
        cells: [
            // 32 cells, each either null or:
            {
                omsFileName: "shader1.oms",
                // Audio buffer loaded separately
            }
        ],
        trackVolumes: [1, 1, 1, 1],
        trackMutes: [false, false, false, false],
        trackSolos: [false, false, false, false],
        masterVolume: 1
    },
    trackData: [
        // 4 tracks, each:
        {
            waveformData: [/* Float32Array samples */],
            beatGridData: {
                bpm: 128,
                barPositions: [0, 2.0, 4.0, 6.0, ...],
                phrasePositions: [0, 8.0, 16.0, ...]
            }
        }
    ]
}
```

**Saving .omsp:**
```javascript
function saveProjectState() {
    const state = {
        version: "3.5.0",
        gridState: {
            masterTempo: gridState.masterTempo,
            zoomValue: gridState.zoomValue,
            cells: gridState.cells.map(cell => 
                cell ? { omsFileName: cell.fileName } : null
            ),
            trackVolumes: gridState.trackVolumes,
            trackMutes: gridState.trackMutes,
            trackSolos: gridState.trackSolos,
            masterVolume: gridState.masterVolume
        },
        trackData: extractTrackData()
    };
    
    const blob = new Blob([JSON.stringify(state)], { type: 'application/json' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `project_${Date.now()}.omsp`;
    a.click();
}
```

---

## DEVELOPMENT WORKFLOW

### Code Style

**Naming Conventions:**
- Variables: camelCase (`masterClockStartTime`)
- Functions: camelCase (`launchTrack()`)
- Constants: UPPER_SNAKE_CASE (rare, use const)
- CSS classes: kebab-case (`grid-cell`)

**Function Organization:**
```javascript
// State update functions (modify state + audio)
function updateVolume(trackIndex, newVolume) { }
function updateMute(trackIndex) { }

// Visual rendering functions (read-only)
function drawVolumeKnob(trackIndex, volume) { }
function drawWaveform(trackIndex) { }

// Audio functions
function launchTrack(trackIndex, cellIndex) { }
function stopTrack(trackIndex) { }

// Utility functions
function getNextBarTime(audioContext, currentTime, trackBPM) { }
function setMasterClock(audioContext, scheduledStartTime, trackBPM) { }
```

**Comments:**
```javascript
// Good: Explain WHY, not WHAT
const playbackRate = masterTempo / trackBPM; // Tempo matching

// Bad: Obvious WHAT
const playbackRate = masterTempo / trackBPM; // Calculate playback rate
```

### Iteration Protocol

**One Feature Per Iteration:**
1. Identify single, specific feature
2. Implement minimal code changes
3. Test thoroughly
4. Document in notes file
5. Increment version letter (A → B → C)

**Example:**
- V3_5_0_A: Track selection only
- V3_5_0_B: Volume control only
- V3_5_0_C: Mute/solo only

**Never combine features in one iteration!**

### Testing Checklist

**Per Iteration:**
- [ ] Feature works as specified
- [ ] No regressions in existing features
- [ ] Visual feedback appears correctly
- [ ] Audio responds as expected
- [ ] Keyboard shortcuts don't conflict
- [ ] Console logs helpful for debugging

**Before Production Release:**
- [ ] All keyboard shortcuts work
- [ ] Multi-BPM track alignment test passes
- [ ] Live tempo change test passes
- [ ] Quantized launching precision verified
- [ ] .oms files save/load correctly
- [ ] .omsp files save/load correctly
- [ ] No console errors
- [ ] Performance acceptable (60 FPS)

---

## TESTING GUIDELINES

### Unit Testing Approach

**Manual Testing (Current Method):**
1. Open HTML file in browser
2. Load test .oms containers
3. Execute test sequence
4. Verify expected behavior
5. Check console logs

**Example Test:**
```javascript
// Test: Quantized launching
// Setup: Load 4 tracks (124, 126, 128, 130 BPM), master 120 BPM
// Steps:
1. Press 'w' to select Scene 0
2. Press Enter to launch all 4 tracks
3. Verify all tracks start on same bar
4. Check console logs for timing
5. Visual: All Bar 1s at playhead
6. Audio: All kicks in sync

// Expected results:
- Track 0: Launches immediately (200ms latency)
- Tracks 1-3: Wait for next bar boundary
- All Bar 1s aligned vertically
- No drift during playback
- Console shows calculated wait times
```

### Integration Testing

**Multi-Component Tests:**
```javascript
// Test: Keyboard → State → Visual → Audio chain
// Steps:
1. Press '1' (select Track 1)
   - State: selectedTrackIndex = 0
   - Visual: Cyan ring on Track 1 knob
2. Press ↑ three times (volume up)
   - State: trackVolumes[0] increases by 0.15
   - Audio: Gain node updated
   - Visual: Knob arc updates
3. Press ',' (mute)
   - State: trackMutes[0] = true
   - Audio: Mute gain = 0
   - Visual: Mute button highlighted
4. Verify audio silent for Track 1
5. Verify other tracks unaffected
```

### Performance Testing

**Timing Precision:**
```javascript
// Log actual vs expected start times
console.log(`Expected: ${nextBarTime.toFixed(6)}s`);
console.log(`Actual: ${audioSource.startTime.toFixed(6)}s`);
console.log(`Difference: ${((audioSource.startTime - nextBarTime) * 1000).toFixed(3)}ms`);

// Acceptable: ±1ms (±48 samples at 48kHz)
```

**Frame Rate:**
```javascript
// Monitor FPS during playback
let lastTime = performance.now();
let frameCount = 0;

function render() {
    const now = performance.now();
    frameCount++;
    
    if (now - lastTime >= 1000) {
        console.log(`FPS: ${frameCount}`);
        frameCount = 0;
        lastTime = now;
    }
    
    requestAnimationFrame(render);
}

// Expected: 60 FPS consistent
```

### Regression Testing

**V3.4.0 Features (Must Still Work):**
- [ ] MacBook Pro 14" layout (1320px fits)
- [ ] Single-letter tab names
- [ ] Visual consistency (all knobs match)
- [ ] Settings tab functional
- [ ] .oms save/load
- [ ] .omsp save/load

**V3.3.0 Features (Must Still Work):**
- [ ] Perfect beat grid synchronization
- [ ] Multi-BPM track alignment
- [ ] Master tempo control (60-200 BPM)
- [ ] Time-stretching
- [ ] Bar-style waveforms
- [ ] Zoom control (10-200 px/sec)

**V3.5.0 Features (New):**
- [ ] All keyboard shortcuts
- [ ] Quantized launching
- [ ] Master clock system
- [ ] Visual selection feedback

---

## DEBUGGING TIPS

### Console Logging

**Keyboard Events:**
```javascript
console.log(`[Key] ${e.key} pressed`);
```

**State Changes:**
```javascript
console.log(`[State] selectedTrackIndex: ${gridState.selectedTrackIndex}`);
console.log(`[State] Volume ${trackIndex}: ${newVolume.toFixed(2)}`);
```

**Master Clock:**
```javascript
console.log(`[Master Clock] Set at ${scheduledStartTime.toFixed(3)}s, BPM: ${trackBPM}`);
console.log(`[Quantize] Next bar at ${nextBarTime.toFixed(3)}s, wait ${(nextBarTime - currentTime).toFixed(3)}s`);
```

**Audio Events:**
```javascript
console.log(`[Launch] Track ${trackIndex}, Start: ${scheduledStartTime.toFixed(3)}s, Rate: ${playbackRate.toFixed(3)}`);
```

### Common Issues

**Issue: Clips launch at wrong time**
- Check: Master clock start time logged correctly?
- Check: Using scheduled start time, not current time?
- Check: Bar calculation formula correct?
- Debug: Add console.log in `getNextBarTime()`

**Issue: Visual selection not appearing**
- Check: State variable updated?
- Check: Visual update function called?
- Check: CSS styling correct?
- Debug: Add console.log in update function

**Issue: Audio doesn't play**
- Check: Audio buffer loaded?
- Check: AudioContext not suspended?
- Check: Gain nodes not zero?
- Debug: Check browser console for errors

**Issue: Keyboard shortcuts not working**
- Check: Event listener attached?
- Check: preventDefault() called?
- Check: Key name matches (case-sensitive)?
- Debug: Log e.key in event listener

---

## CONTRIBUTING

### Pull Request Guidelines

1. **One feature per PR**
2. **Follow iteration protocol** (increment version letter)
3. **Test thoroughly** before submitting
4. **Document changes** in notes file
5. **Maintain backward compatibility**
6. **Update developer docs** if architecture changes

### Code Review Checklist

**Functionality:**
- [ ] Feature works as described
- [ ] No regressions
- [ ] Edge cases handled

**Code Quality:**
- [ ] Follows naming conventions
- [ ] Comments explain WHY
- [ ] No console.log in production (except intentional logging)
- [ ] Efficient (no unnecessary loops/calculations)

**Testing:**
- [ ] Manual testing performed
- [ ] Performance acceptable
- [ ] Works on target browsers (Chrome, Edge)

**Documentation:**
- [ ] Notes file created
- [ ] Developer docs updated (if needed)
- [ ] User guide updated (if user-facing)

---

## ARCHITECTURE DECISIONS

### Why Separate Audio Contexts?

**Decision:** Use 4 separate AudioContexts instead of 1 shared context.

**Rationale:**
- Independent suspend/resume per track (future feature)
- Prevents audio crosstalk between tracks
- Better performance isolation
- Matches professional DJ software architecture (Traktor, Serato)

**Trade-off:** Slightly higher resource usage, but negligible on modern hardware.

### Why Inset Box-Shadow for Cell Selection?

**Decision:** Use `box-shadow: inset` instead of `outline` for cell selection.

**Rationale:**
- `outline` draws OUTSIDE element boundary
- Creates visible gap between grid cells
- `inset box-shadow` draws INSIDE element
- Clean, professional appearance

**Alternative Considered:** Border (rejected - changes element size, affects layout)

### Why Master Clock Instead of Per-Track Timing?

**Decision:** Use global master clock reference instead of per-track timing calculations.

**Rationale:**
- Single source of truth for timing
- Prevents timing drift between tracks
- Simpler mental model (one timeline)
- Matches "One Timeline" architecture principle from V3.3.0

**Alternative Considered:** Per-track bar calculations (rejected - drift accumulation, complexity)

### Why Sample-Accurate Scheduling?

**Decision:** Use Web Audio API `source.start(time)` instead of setTimeout/setInterval.

**Rationale:**
- Sample-accurate precision (±0.001ms)
- No drift over time
- Professional timing requirements
- "Soldiers at the goalpost" precision goal

**Trade-off:** Slightly more complex code, but necessary for professional results.

---

## FUTURE DEVELOPMENT

### Planned Features (V3.6.0+)

**Real-Time Shader Rendering:**
- Shader rendering in grid cells (4 simultaneous)
- Master output compositing
- Visual blend modes
- Complete visual performance

**MIDI Control:**
- Web MIDI API integration
- Hardware controller support
- Midifighter Twister mapping
- CC assignments for knobs

**Ableton Link:**
- Network tempo sync
- Multi-device synchronization
- Professional DJ workflow integration

### Architecture Considerations

**Multi-Instance Communication:**
- BroadcastChannel API for inter-instance messaging
- Share tempo, play state across instances
- Coordinate launches between instances

**Recording/Export:**
- Capture audio from master output
- Record shader video during performance
- Export complete audio-visual performance

**Session Management:**
- Auto-save project state
- Undo/redo functionality
- Session templates

---

## REFERENCES

### Web APIs Used

**Web Audio API:**
- Documentation: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
- Spec: https://www.w3.org/TR/webaudio/

**WebGPU:**
- Documentation: https://developer.mozilla.org/en-US/docs/Web/API/WebGPU_API
- Spec: https://www.w3.org/TR/webgpu/

**File System Access API:**
- Documentation: https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API
- Browser compatibility: Chrome 86+, Edge 86+

**Keyboard Events:**
- Documentation: https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent

### Related Projects

**Ableton Live:** https://www.ableton.com/  
**EboSuite:** https://ebosuite.rocks/  
**Serato DJ:** https://serato.com/  
**Traktor:** https://www.native-instruments.com/traktor/

---

**END OF DEVELOPER DOCUMENTATION**

Version: V3.5.0 Production Release  
Date: November 16, 2025  
License: GPL-3.0  
Developer: Wes Jones (ONEMANSHYO)
