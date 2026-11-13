# OMS SHADER BAKER V3.9.0 - DEVELOPER DOCUMENTATION

**Version:** V3.9.0_L_sub20 (Development Build)  
**Date:** November 21, 2025  
**Purpose:** Technical reference for developers

---

## ARCHITECTURE CHANGES

**File:** OMS_ShaderBaker_V3_9_0_L_sub20.html  
**Size:** ~560KB  
**Lines:** ~13,737  
**Dependencies:** Zero

---

## WHAT CHANGED IN V3.9.0

### 1. Master Clock Object (NEW)

**Location:** ~line 8538

**Core Structure:**
```javascript
const masterClock = {
    audioContext: null,              // Set during initAudioRouting()
    
    // Getters (always fresh, no caching)
    get currentTime() {
        return this.audioContext?.currentTime || 0;
    },
    
    get currentTempo() {
        return gridState.masterTempo;
    },
    
    get secondsPerBeat() {
        return 60.0 / this.currentTempo;
    },
    
    get secondsPerBar() {
        return this.secondsPerBeat * 4;
    },
    
    // Methods
    getCurrentBeat() {
        if (!sessionStartTime) return 0;
        const elapsed = this.currentTime - sessionStartTime;
        // Walk tempo history
        let totalBeats = 0;
        let lastTime = sessionStartTime;
        
        for (const change of tempoHistory) {
            if (change.time > this.currentTime) break;
            const duration = change.time - lastTime;
            const bpm = change.previousTempo || this.currentTempo;
            totalBeats += duration / (60.0 / bpm);
            lastTime = change.time;
        }
        
        const finalDuration = this.currentTime - lastTime;
        totalBeats += finalDuration / this.secondsPerBeat;
        return totalBeats;
    },
    
    getCurrentBar() {
        return this.getCurrentBeat() / 4;
    },
    
    getNextBarTime() {
        const currentBeat = this.getCurrentBeat();
        const nextBar = Math.ceil(currentBeat / 4) * 4;
        return sessionStartTime + (nextBar * this.secondsPerBeat);
    },
    
    logState(context) {
        console.log(`[MasterClock] ${context}: {
            currentTime: ${this.currentTime.toFixed(3)},
            currentTempo: ${this.currentTempo},
            secondsPerBeat: ${this.secondsPerBeat.toFixed(3)},
            secondsPerBar: ${this.secondsPerBar.toFixed(3)},
            currentBeat: ${this.getCurrentBeat().toFixed(3)},
            currentBar: ${this.getCurrentBar().toFixed(3)}
        }`);
    }
};
```

**Philosophy:**
- Single source of truth for all timing
- No cached values (getters ensure freshness)
- No local timers (derives from audioContext.currentTime)
- SessionStartTime-based calculations

---

### 2. Beat-Based Scrolling

**New State:**
```javascript
let trackLaunchBeats = [null, null, null, null];  // Beat position when launched
let tempoHistory = [];  // Track tempo changes during session
```

**Scroll Calculation:**
```javascript
function updateAudioWaveformScroll(track) {
    const currentBeat = masterClock.getCurrentBeat();
    const launchBeat = trackLaunchBeats[track];
    
    if (launchBeat === null) return;
    
    const beatsSinceStart = currentBeat - launchBeat;
    const pixelsPerBeat = gridState.pixelsPerBar / 4;
    const scrollOffset = beatsSinceStart * pixelsPerBeat;
    
    const canvas = document.getElementById(`waveform-canvas-${track + 1}`);
    const initialOffset = parseFloat(canvas.dataset.initialOffset || 0);
    
    canvas.style.transform = `translateX(${initialOffset - scrollOffset}px)`;
}
```

**Why Beat-Based:**
- Musical coordinates match Master Grid
- Handles tempo changes correctly
- No clock time → musical time conversion errors
- "One Timeline" architecture

---

### 3. Launch Beat Tracking

**On Launch:**
```javascript
function launchContainer(track, sceneIndex) {
    // ... existing code ...
    
    // V3_9_0: Track launch beat
    trackLaunchBeats[track] = masterClock.getCurrentBeat();
    
    // Log master clock state
    masterClock.logState(`Track ${track + 1} launched`);
    
    // ... continue launch ...
}
```

**On Stop:**
```javascript
function stopTrack(track) {
    // ... existing code ...
    
    // V3_9_0: Clear launch beat
    trackLaunchBeats[track] = null;
    
    // ... continue stop ...
}
```

---

### 4. Tempo History Walker

**Recording Changes:**
```javascript
function setMasterTempo(newTempo) {
    // V3_9_0: Record tempo change
    tempoHistory.push({
        time: masterClock.currentTime,
        previousTempo: gridState.masterTempo,
        newTempo: newTempo
    });
    
    gridState.masterTempo = newTempo;
    
    masterClock.logState('Tempo changed');
    
    // Redraw waveforms
    redrawAllWaveforms();
}
```

**Walking History:**
```javascript
// Inside masterClock.getCurrentBeat()
let totalBeats = 0;
let lastTime = sessionStartTime;

for (const change of tempoHistory) {
    if (change.time > this.currentTime) break;
    
    const duration = change.time - lastTime;
    const bpm = change.previousTempo || this.currentTempo;
    totalBeats += duration / (60.0 / bpm);
    
    lastTime = change.time;
}

// Final segment at current tempo
const finalDuration = this.currentTime - lastTime;
totalBeats += finalDuration / this.secondsPerBeat;

return totalBeats;
```

---

### 5. Waveform Positioning (audioStartTime Removal)

**Before (V3.8.0):**
```javascript
const audioStartTime = containerData.audioStartTime || 0;
const bar1PositionPx = audioStartTime * pixelsPerSecond;
const offsetNeeded = playheadInCanvasSpace - bar1PositionPx;
```

**After (V3.9.0):**
```javascript
// Bar 1 always at playhead, period
const offsetNeeded = playheadInCanvasSpace;
```

**Why:**
- Master Grid has no concept of audioStartTime
- Visual Bar 1 should always be at playhead
- Audio playback still trims via `source.start(when, audioStartTime)`
- Separates audio domain (playback) from visual domain (display)

---

### 6. Pop-Out Window Fix

**Container Data Enhancement:**
```javascript
function loadContainerToGridCell(track, scene, container) {
    trackContainerData[track][scene] = {
        name: container.name,
        shaderCode: container.shaderCode,      // V3_9_0_L: Store shader code
        audioFile: container.audioFile,
        audioBuffer: audioBuffer,
        // ... other properties ...
    };
}
```

**Pop-Out Shader Retrieval:**
```javascript
function compileShaderForTrack(track) {
    const containerData = trackContainerData[track][currentSceneIndex];
    const shaderCode = containerData?.shaderCode;  // V3_9_0_L: Retrieve stored code
    
    if (!shaderCode) {
        console.error(`No shader code for track ${track + 1}`);
        return;
    }
    
    // Compile shader...
}
```

---

## CORE SYSTEMS (Unchanged from V3.8.0)

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

## TESTING CHECKLIST

### V3.9.0 Specific Tests

**Master Clock:**
1. ✅ AudioContext connection verified
2. ✅ Beat/bar calculations accurate
3. ✅ Tempo changes logged
4. ✅ No cached values (getters work)

**Beat-Based Scrolling:**
1. ⚠️ Waveforms scroll at master tempo
2. ⚠️ Tempo changes handled correctly
3. ⚠️ Multi-track sync maintained
4. ⚠️ Launch beat tracking accurate

**Master Grid Alignment:**
1. ⚠️ Track bars align with Master Grid
2. ⚠️ No uniform offset
3. ⚠️ Alignment persists during playback
4. ⚠️ Tempo changes don't break alignment

**Pop-Out Window:**
1. ✅ Shaders compile in pop-out
2. ✅ All 4 tracks render correctly
3. ✅ Shader code retrieved from container data
4. ✅ No compilation errors

---

## PERFORMANCE

**GPU Usage (4 tracks):**
- M1 MacBook Pro: ~30-45%
- Discrete GPU: ~20-30%
- Integrated GPU: ~50-60%

**Frame Rate:** 60 FPS stable  
**Memory:** ~250-350 MB with audio  
**File Size:** ~560KB (+10KB from V3.8.0)

---

## API REFERENCE

### Master Clock

```javascript
// Properties (getters)
masterClock.currentTime         // audioContext.currentTime
masterClock.currentTempo        // gridState.masterTempo
masterClock.secondsPerBeat      // 60.0 / tempo
masterClock.secondsPerBar       // secondsPerBeat * 4

// Methods
masterClock.getCurrentBeat()    // Beat position (walks tempo history)
masterClock.getCurrentBar()     // Bar position (beats / 4)
masterClock.getNextBeatTime()   // Next beat boundary time
masterClock.getNextBarTime()    // Next bar boundary time
masterClock.logState(context)   // Debug logging
```

### Launch Beat Tracking

```javascript
trackLaunchBeats[track]         // Beat when track launched (null if stopped)
tempoHistory[]                  // Array of tempo changes
```

---

## MIGRATION NOTES

**From V3.8.0 to V3.9.0:**

**No Breaking Changes:**
- All V3.8.0 features preserved
- Backward compatible
- .oms and .omsp files work unchanged

**New Requirements:**
- None (timing improvements are internal)

**Optional Enhancements:**
- Test tempo changes during performance
- Verify Master Grid alignment
- Validate multi-track synchronization

---

## DEBUGGING

**Console Logging:**

**Master Clock State:**
```javascript
masterClock.logState('Custom context');
```

**Output:**
```
[MasterClock] Custom context: {
    currentTime: 15.832,
    currentTempo: 120,
    secondsPerBeat: 0.500,
    secondsPerBar: 2.000,
    currentBeat: 31.664,
    currentBar: 7.916
}
```

**Launch Beat Tracking:**
```javascript
console.log('Launch beats:', trackLaunchBeats);
console.log('Tempo history:', tempoHistory);
```

---

## KNOWN ISSUES

⚠️ **Master Grid Alignment:** Under testing, may need refinement  
⚠️ **Tempo Change Handling:** Beat-based system needs validation  
⚠️ **Multi-Track Launch:** Sync accuracy being verified

---

## ROADMAP

**Immediate (V3.9.0 completion):**
- Finalize Master Grid alignment
- Validate tempo change handling
- Test multi-track scenarios
- Performance optimization

**Near-Term (V3.10.0 or V4.0.0):**
- Quantization options (1 bar, 1/2 bar, 1 beat, None)
- Visual launch countdown (armed state)
- Shader parameter controls in LIVE tab
- Master Grid toggle (G key)

**Long-Term:**
- MIDI integration (Web MIDI API)
- Ableton Link
- Tap tempo
- Advanced waveform analysis

---

**END OF DEVELOPER DOCUMENTATION**

Version: V3.9.0_L_sub20 (Development Build)  
Date: November 21, 2025  
License: GPL-3.0
