# OMS SHADER BAKER V3.2.0 - DEVELOPER DOCUMENTATION

**Version:** V3.2.0
**Audience:** Developers, Technical Reference
**Purpose:** Technical architecture and implementation details

---

## ARCHITECTURE OVERVIEW

### Beat Grid System

**Core Components:**
1. **Track Container Data** - Stores audio timing per track
2. **Beat Grid Rendering** - Canvas drawing with bar lines
3. **Master Tempo Sync** - Time-stretching engine
4. **Visual Scaling** - CSS transforms (GPU-accelerated)
5. **Synced Scrolling** - RequestAnimationFrame loop

### Data Flow

```
Container Load → Extract Timing Data → Store in trackContainerData[]
                                    ↓
                          Draw Beat Grid on Canvas
                                    ↓
                          Calculate ScaleX Ratio
                                    ↓
                          Apply CSS Transform
                                    ↓
                          Position at Playhead
                                    ↓
                          Scroll Animation (preserves scaleX)
```

---

## BEAT GRID IMPLEMENTATION

### Track Container Data Storage

**Location:** Line ~2343-2345

```javascript
const trackContainerData = [null, null, null, null]; // 4 tracks
```

**Structure:**
```javascript
trackContainerData[track] = {
    audioBPM: 128,           // Original tempo
    audioStartTime: 0.0,     // Where music begins (seconds)
    audioEndTime: 180.5,     // Where music ends (seconds)
    audioLoopIn: 64.0,       // Loop start (beats, 0-indexed)
    audioLoopOut: 128.0,     // Loop end (beats, 0-indexed)
    name: "Bass_Tunnel"      // Container name
};
```

**Stored When:** Container assigned to cell (`assignContainerToCell` function)

---

### Beat Grid Drawing

**Function:** `drawBeatGridOnWaveform(ctx, containerData, audioBuffer, canvasWidth, canvasHeight)`

**Location:** Lines ~8628-8667

**Algorithm:**
```javascript
// 1. Calculate seconds per bar (4 beats in 4/4 time)
const secondsPerBar = (4 / audioBPM) * 60;

// 2. Calculate pixels per second
const pixelsPerSecond = canvasWidth / audioBuffer.duration;

// 3. Loop through bars from audioStartTime to audioEndTime
let barNumber = 1;
let barTime = audioStartTime;

while (barTime <= audioEndTime) {
    const x = barTime * pixelsPerSecond;
    
    // 4. Draw bar line (cyan or magenta)
    if (barNumber % 4 === 0) {
        // Phrase marker (magenta, 2px, 75% opacity)
        ctx.strokeStyle = 'rgba(255, 0, 255, 0.75)';
        ctx.lineWidth = 2;
    } else {
        // Regular bar (cyan, 1px, 30% opacity)
        ctx.strokeStyle = 'rgba(0, 255, 255, 0.3)';
        ctx.lineWidth = 1;
    }
    
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, canvasHeight);
    ctx.stroke();
    
    // 5. Draw bar number (every 4 bars)
    if (barNumber % 4 === 1) {
        ctx.fillStyle = 'rgba(255, 255, 255, 0.7)';
        ctx.font = '10px monospace';
        ctx.fillText(barNumber.toString(), x + 2, 12);
    }
    
    barNumber++;
    barTime += secondsPerBar;
}
```

**Key Points:**
- Draws at ORIGINAL BPM (not master tempo)
- CSS scaleX transform handles tempo scaling
- Phrase markers every 4 bars (magenta, thicker)
- Bar numbers every 4 bars for navigation

---

### Master Tempo Synchronization

**Audio Playback Rate:**

**Location:** `playCellAudio()` function, lines ~8812-8820

```javascript
const containerData = trackContainerData[track];
if (containerData && containerData.audioBPM) {
    // Calculate playback rate: masterTempo / originalBPM
    const playbackRate = gridState.masterTempo / containerData.audioBPM;
    source.playbackRate.value = playbackRate;
    
    console.log(`Track ${track + 1} playback rate: ${playbackRate.toFixed(4)} (${containerData.audioBPM} → ${gridState.masterTempo} BPM)`);
}
```

**Example:**
- Container: 128 BPM
- Master: 120 BPM
- Playback rate: 120 / 128 = 0.9375 (play 6.25% slower)
- Result: 128 BPM track plays at 120 BPM

---

### Visual Scaling (CSS Transforms)

**Function:** `positionAudioWaveformAtPlayhead(track, audioBuffer)`

**Location:** Lines ~8754-8822

**Key Algorithm:**
```javascript
// 1. Get container data
const containerData = trackContainerData[track - 1];

// 2. Calculate scaleX ratio (INVERTED from audio!)
let scaleX = 1.0;
if (containerData && containerData.audioBPM) {
    scaleX = containerData.audioBPM / gridState.masterTempo;
}
// Example: 128 / 120 = 1.0667 (stretch visual)
// Audio plays SLOWER, visual must STRETCH

// 3. Calculate Bar 1 position (transform origin)
const pixelsPerSecond = canvas.width / audioBuffer.duration;
const bar1PositionPx = audioStartTime * pixelsPerSecond;

// 4. Set transform origin to Bar 1
canvas.style.transformOrigin = `${bar1PositionPx}px center`;

// 5. Apply combined transform
const playheadPositionInCanvasSpace = playheadPosition - canvasLeftOffset;
const offsetNeeded = playheadPositionInCanvasSpace - bar1PositionPx * scaleX;
canvas.style.transform = `translateX(${offsetNeeded}px) scaleX(${scaleX})`;
```

**Critical Points:**
- ScaleX ratio INVERTED from playback rate
- Audio: `masterTempo / audioBPM` (e.g., 120/128 = 0.9375 compress)
- Visual: `audioBPM / masterTempo` (e.g., 128/120 = 1.0667 stretch)
- Transform origin at Bar 1 (keeps Bar 1 fixed during scaling)
- TranslateX positions canvas so Bar 1 aligns at playhead

---

### Synced Scrolling

**Function:** `startAudioWaveformScroll(track)`

**Location:** Lines ~8940-9020

**Key Fix (V3.2.0_AB + V3.2.0_AE):**
```javascript
// 1. Extract BOTH transforms from current style
const currentTransform = canvas.style.transform || '';
const translateMatch = currentTransform.match(/translateX\(([^)]+)px\)/);
const scaleMatch = currentTransform.match(/scaleX\(([^)]+)\)/);
const initialOffset = translateMatch ? parseFloat(translateMatch[1]) : 0;
const scaleX = scaleMatch ? parseFloat(scaleMatch[1]) : 1.0;

// 2. In animation loop, apply BOTH transforms
function animate() {
    const elapsed = (Date.now() - startTime) / 1000;
    
    // V3.2.0_AE: Scroll speed adjusted by scaleX
    const scrollOffset = elapsed * pixelsPerSecond * scaleX;
    
    // Apply both transforms
    canvas.style.transform = `translateX(${initialOffset - scrollOffset}px) scaleX(${scaleX})`;
    
    requestAnimationFrame(animate);
}
```

**Why This Works:**
- V3.2.0_AB bug: Scroll animation was overwriting scaleX (losing tempo scaling)
- V3.2.0_AE fix: Scroll speed multiplied by scaleX (sync visual movement)
- Stretched waveforms (scaleX > 1) scroll faster
- Compressed waveforms (scaleX < 1) scroll slower
- Result: All tracks scroll at same VISUAL speed

---

## AUDIO DATA STORAGE

### Extended .oms Format

**New Fields (V3.2.0):**
```json
{
  "audioPitch": 2.5,           // Playback speed adjustment (±16%)
  "audioStartTime": 0.0,       // Where musical content begins (seconds)
  "audioEndTime": 180.5,       // Where musical content ends (seconds)
  "audioLoopIn": 64.0,         // Loop start (beats, 0-indexed)
  "audioLoopOut": 128.0        // Loop end (beats, 0-indexed)
}
```

**Save Location:** `confirmAddToLibrary()` and `updateContainer()` functions

**Load Location:** Container loading functions (line ~5201-5241)

**Backward Compatible:** Missing fields use sensible defaults

---

## KEY FUNCTIONS REFERENCE

### Beat Grid Functions

1. **drawBeatGridOnWaveform(ctx, containerData, audioBuffer, width, height)**
   - Draws bar lines and phrase markers on canvas
   - Lines ~8628-8667

2. **drawInfoOverlay(ctx, containerData, canvasHeight)**
   - Draws container name + BPM overlay at bottom
   - Lines ~8803-8818

### Audio Timing Functions

3. **positionAudioWaveformAtPlayhead(track, audioBuffer)**
   - Calculates scaleX ratio
   - Sets transform origin to Bar 1
   - Applies translateX + scaleX transform
   - Lines ~8754-8822

4. **startAudioWaveformScroll(track)**
   - Animates waveform scrolling
   - Preserves scaleX during animation
   - Adjusts scroll speed by scaleX
   - Lines ~8940-9020

### Container Data Functions

5. **assignContainerToCell(containerId, track, scene)**
   - Loads container from LIBRARY
   - Stores audio timing in trackContainerData[]
   - Renders shader thumbnail
   - Pre-loads and decodes audio
   - Draws waveform with beat grid
   - Lines ~8265-8436

6. **playCellAudio(track, scene, cellData, startTime)**
   - Loads pre-loaded audio buffer
   - Applies master tempo playback rate
   - Starts audio playback
   - Lines ~8537-8820

---

## TECHNICAL INSIGHTS

### The Ableton "Start from here" Analogy

**From Debugging Session (V3.2.0_AB):**

Wes's explanation that solved the alignment problem:

> "In Ableton warping 'Start from here' mode: you pin the START (audioStartTime), set the BPM (128), and Ableton just lays out bars from that point forward. No complex math, just 'start here, this tempo, go'."

This is EXACTLY what Shader Baker does:
1. Pin the START (audioStartTime)
2. Set the BPM (audioBPM)
3. Draw beat grid from there
4. Scale from Bar 1 position (CSS transform origin)
5. Bar 1 stays FIXED during scaling
6. TranslateX positions canvas so Bar 1 aligns with playhead

**Result:** "Goal post" alignment - all Bar 1s line up at playhead

### ScaleX Ratio (The Rubber Band)

**Audio playback rate vs Visual scaleX:**

```
Audio: masterTempo / audioBPM
Visual: audioBPM / masterTempo (INVERTED!)
```

**Why inverted:**
- 128 BPM track at 120 BPM master
- Audio plays SLOWER (0.9375x speed)
- Visual must STRETCH to match (1.0667x wider)
- Like stretching a rubber band

**The logic:**
- Faster original BPM → plays slower → visual stretches
- Slower original BPM → plays faster → visual compresses

### Transform Origin is Critical

**Wrong approach:**
```javascript
canvas.style.transformOrigin = 'left center'; // Scales from left edge
// Result: Bar 1 moves during scaling
```

**Correct approach:**
```javascript
const bar1Px = audioStartTime * pixelsPerSecond;
canvas.style.transformOrigin = `${bar1Px}px center`; // Scales from Bar 1
// Result: Bar 1 stays fixed, everything else scales uniformly
```

### GPU Acceleration

**CSS Transforms are GPU-accelerated:**
- No canvas redrawing when tempo changes
- Instant visual updates
- Smooth 60fps scrolling
- Low CPU overhead

**Implementation:**
```javascript
canvas.style.transform = `translateX(${x}px) scaleX(${ratio})`;
canvas.style.transformOrigin = `${originX}px center`;
```

Both properties handled by GPU compositor.

---

## DEBUGGING TIPS

### Console Logging

**Added in V3.2.0_AB:**
Extensive logging in `positionAudioWaveformAtPlayhead()`:

```javascript
console.log(`positionAudioWaveformAtPlayhead called for track ${track}`);
console.log(`Track ${track} containerData:`, containerData);
console.log(`Track ${track} scaleX = ${audioBPM} / ${masterTempo} = ${scaleX.toFixed(4)}`);
console.log(`Track ${track} Bar 1 at ${bar1PositionPx.toFixed(2)}px`);
console.log(`Track ${track} positioning + scaling:`, {
    bar1PositionPx,
    scaleX,
    transformOrigin,
    offsetNeeded,
    finalTransform
});
```

**Look for:**
- Missing containerData (null check failures)
- Wrong scaleX values (check ratio calculation)
- Wrong Bar 1 position (check audioStartTime)
- Transform origin issues (check pixel calculation)

### Common Bugs

**1. getBoundingClientRect() after transform:**
Returns transformed position, not original CSS position.
Solution: Use `canvas.offsetLeft` instead

**2. Scroll animation losing scaleX:**
Only setting translateX in animation loop.
Solution: Extract and preserve scaleX from initial transform

**3. Double scaling:**
Drawing beat grid at master tempo, then CSS scaling again.
Solution: Draw at original BPM, CSS handles all scaling

**4. Array lookup instead of parameter:**
Function receives `audioBuffer` but looks up `trackAudioBuffers[track]`.
Solution: Use parameter directly

---

## PERFORMANCE CONSIDERATIONS

### Canvas Drawing

**Beat grid drawn once:**
- On container assignment (drag to cell)
- Not redrawn during tempo changes
- CSS transform handles visual scaling

**Waveform drawn once:**
- Bar-style rendering (RMS-based)
- ~200-500 bars depending on duration
- Single pass drawing

### Memory Usage

**Per Track:**
- trackContainerData: ~200 bytes
- AudioBuffer: ~50MB (decoded audio)
- Canvas: ~2MB (waveform + beat grid)
- Total: ~52MB per track

**4 Tracks Active:**
- Total memory: ~210MB
- Acceptable for modern systems

### GPU Utilization

**CSS Transforms:**
- Transform operations on GPU
- Compositor handles all scaling
- No CPU canvas redrawing

**Scroll Animation:**
- 60fps requestAnimationFrame
- Only updating transform property
- GPU handles rendering

---

## CODE LOCATIONS

**Key Files:**
- OMS_ShaderBaker_V3_2_0.html (9,783 lines)

**Key Functions (Line Numbers):**
- trackContainerData: ~2343
- assignContainerToCell: ~8265
- drawBeatGridOnWaveform: ~8628
- drawInfoOverlay: ~8803
- positionAudioWaveformAtPlayhead: ~8754
- startAudioWaveformScroll: ~8940
- playCellAudio: ~8537

**Key Data Structures:**
- trackContainerData[4]: Array of timing objects
- gridState.masterTempo: Number (60-200)
- gridState.cells[32]: Array of cell assignments

---

## FUTURE ENHANCEMENTS

**V3.3.0 Planned:**
- Scene launching (quantized to bars)
- Loop bracket visualization
- Time signature support (beyond 4/4)
- Waveform zoom controls

**V3.4.0 Planned:**
- MIDI control integration
- Ableton Link synchronization
- Beat phase alignment
- Advanced transport controls

---

**END OF DEVELOPER DOCUMENTATION**

For implementation questions, see Complete_Changelog.md (all 31 iterations with technical details).
