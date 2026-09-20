# ONEMANSHYO Dojo V2_2 - Release Notes

**Version:** V2_2  
**Status:** Production Release  
**Published:** November 2025  
**Previous Version:** V2_1

---

## Executive Summary

V2_2 enhances the audio workflow with **automatic BPM detection**, **millisecond-precision timing adjustment** (Nudge field), and **clearer UI labeling**. The focus is on making audio-reactive shader creation faster and more precise, while maintaining all the powerful features from V2.1.0.

**TL;DR:**  
✅ BPM auto-detected on audio load  
✅ Nudge field for perfect timing  
✅ Cleaner UI labels (BPM, Loop In, Loop Out)  
✅ Loop button for playback control  
✅ Simplified arrow key behavior  
✅ All V2.1.0 features preserved  

---

## What's New in V2_2

### 1. **Automatic BPM Detection** 🎵

**Before:** Manual BPM entry required  
**Now:** Tempo automatically detected on audio load

**How It Works:**
- Analyzes audio buffer using onset detection
- Detects BPM in range 60-180 (0.5 step precision)
- Displays detected tempo with 1 decimal place
- User can override by typing if needed

**Benefits:**
- Faster workflow (no manual tempo entry)
- More accurate (algorithmic detection)
- Less guesswork (especially for complex music)

**Example:**
```
1. Upload audio file
2. "Detecting..." appears in BPM field
3. "127.5 BPM" auto-fills
4. Done! (or type to override)
```

---

### 2. **Nudge Field for Fine-Tuning** ⏱️

**New Feature:** Millisecond-precision loop timing adjustment

**What It Does:**
- Shifts **entire loop** (both Loop In and Loop Out) by milliseconds
- Does NOT change beat positions
- Applied to all audio jumps (load, stop, play, loop, render)

**Use Cases:**
- **Loop sounds good but starts slightly early** → Nudge -50ms
- **Video render audio/visual slightly off-sync** → Nudge +30ms
- **Fine-tune after finding rough loop position** → Nudge ±10-20ms

**Controls:**
- Range: -5000ms to +5000ms
- Arrow keys: ±1ms (Up/Down), ±4ms (Left/Right)
- Type value directly

**Location:** Audio controls strip (after Loop Out field)

---

### 3. **UI Clarity Improvements** 🔘

**Renamed Labels:**
- `"Audio BPM"` → `"BPM"` (cleaner, less redundant)
- `"IN"` → `"Loop In"` (clearer that it's a loop boundary)
- `"OUT"` → `"Loop Out"` (clearer that it's a loop boundary)

**Updated Tooltips:**
- All tooltips updated to match new labels
- More descriptive and helpful

**Why:** Clearer labels reduce confusion and improve usability, especially for new users.

---

### 4. **Loop Button** 🔁

**New Feature:** Toggle audio playback mode

**Two Modes:**
- **Loop (Active - Default):** Audio repeats Loop In → Loop Out
- **Continuous (Inactive):** Audio plays without looping

**Visual Indicator:**
- Active: Standard button styling
- Inactive: 40% opacity (grayed out)

**Use Cases:**
- Loop ON: Testing short sections repeatedly
- Loop OFF: Previewing long sections or full songs

---

### 5. **Simplified Arrow Key Behavior** 🔒

**Before:** Grid/Free toggle controlled bracket behavior  
**Now:** Arrow keys ALWAYS move both brackets together

**New Behavior (Loop In or Loop Out focused):**
- **Up:** Both +1 beat (shift window right)
- **Down:** Both -1 beat (shift window left)
- **Left:** Both -4 beats / 1 bar (shift window left)
- **Right:** Both +4 beats / 1 bar (shift window right)

**Result:** Maintains loop length, shifts entire window - simpler and more predictable!

**Grid Button:** Disabled (grayed out) - reserved for future visual grid feature

---

## Audio Controls Strip (New Layout)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ [Play ▶] [Stop ■] [Loop] │ BPM: [120.0] │ Loop In: [64] Loop Out: [128] │
│                                              Nudge: [0 ms]  [Grid]        │
└──────────────────────────────────────────────────────────────────────────┘
                                                               ↑
                                                           (disabled)
```

**Left to Right:**
1. **Play ▶** - Play/Pause (Spacebar)
2. **Stop ■** - Stop and reset to Loop In
3. **Loop** - Toggle loop mode (active/inactive)
4. **BPM** - Auto-detected tempo (type to override)
5. **Loop In** - Loop start beat (click label → jump to 0)
6. **Loop Out** - Loop end beat (click label → jump to end)
7. **Nudge** - Fine-tune timing by milliseconds
8. **Grid** - Disabled (reserved for future)

---

## Typical Workflow (V2_2)

### Quick Start

1. **Upload Audio**
   - Drag file onto waveform or click Upload
   - BPM auto-detects → "127.5 BPM"
   - Audio jumps to Loop In beat (64)

2. **Find Loop Section**
   - Use arrow keys to shift Loop In/Out window
   - Right Arrow: Try next bar →
   - Left Arrow: Try previous bar ←
   - Maintains loop length

3. **Fine-Tune Timing**
   - Click Nudge field
   - Arrow keys: ±1ms or ±4ms
   - Or type value (e.g., -50 for -50ms)

4. **Edit Shader**
   - Switch to Code tab
   - Modify WGSL fragment shader
   - Click [Compile]

5. **Render Video**
   - Switch to Controls tab
   - Set Orientation, Resolution, FPS
   - Enter Filename
   - Click [R] Render Video
   - Video downloads as WebM

---

## Preserved Features from V2.1.0

**Tab System:**
- ✓ Controls / Code tabs
- ✓ Bigger shader editor
- ✓ Focused workflows

**HD Canvas:**
- ✓ 1920x1080 preview
- ✓ Matches 1080p export
- ✓ Better visual feedback

**Waveform on Canvas:**
- ✓ 1920px wide
- ✓ 6x bigger than V2.0.2
- ✓ Real amplitude analysis

**Shader Management:**
- ✓ Compile button in Code tab
- ✓ Copy Code to clipboard
- ✓ Export .txt with version

**Drag & Drop:**
- ✓ Audio files onto waveform
- ✓ Shader files (.txt, .wgsl) onto editor

**Transport Controls:**
- ✓ Play/Stop buttons
- ✓ Visual state feedback
- ✓ Spacebar shortcut

---

## Technical Improvements

### BPM Detection Algorithm

**Method:** Onset detection with autocorrelation
- Buffer size: 4096 samples
- Hop size: 2048 samples
- Energy threshold: 0.15 (RMS-based)
- Min onset gap: 0.1 seconds
- BPM range: 60-180 (0.5 step)
- Scoring: Multiple multipliers (1x, 2x, 4x)

**Accuracy:** Works best with clear beats (percussion, kicks)

### Nudge System

**Implementation:** Global offset applied to all audio position jumps
```javascript
let nudgeOffset = 0; // milliseconds (converted to seconds)
audioElement.currentTime = (beat / beatsPerSecond) + nudgeOffset;
```

**Applied to:**
- Audio load
- Stop button
- Play (if outside loop range)
- Loop (return to IN)
- Compile shader
- Video render start

### Performance

**BPM Detection Impact:**
- Analysis time: 1-2 seconds on load (one-time)
- CPU usage: Minimal (only runs once)
- No impact on playback or rendering

**Nudge Offset Impact:**
- Computational cost: None (simple addition)
- Applied at jump time only (not every frame)

---

## Breaking Changes

**None!** V2_2 is fully backward compatible with V2.1.0.

**What Changed:**
- UI labels (cosmetic only)
- Grid button disabled (reserved for future)
- Arrow keys always move both brackets (simplified)

**What Didn't Change:**
- All keyboard shortcuts work the same
- All video rendering works the same
- All shader compilation works the same
- All modulation system works the same

---

## Migration from V2.1.0

**If you're upgrading from V2.1.0:**

✅ **No changes needed** - Open V2_2 and start using  
✅ **Same workflows** - Everything you know still works  
✅ **New features** - BPM detection, Nudge, Loop button  
✅ **Clearer labels** - "BPM", "Loop In", "Loop Out"  

**Things to try:**
1. Upload audio and watch BPM auto-detect
2. Use Nudge field for fine-timing
3. Toggle Loop button for playback modes
4. Enjoy simpler arrow key behavior

---

## Known Limitations

### BPM Detection
- Range limited to 60-180 BPM (expandable in future)
- Works best with clear beats (percussion)
- May struggle with ambient/classical music
- User can override if detection fails

### Nudge Field
- Range limited to ±5000ms (10 seconds total)
- Affects all audio jumps globally
- Cannot set different nudge per section (yet)

### Grid Button
- Disabled in V2_2
- Reserved for future visual grid overlay
- No functionality currently

---

## Future Roadmap

### Planned
- Visual beat grid overlay (Grid button activation)
- Expanded BPM range (40-240)
- Per-section nudge offsets
- Draggable IN/OUT handles on waveform

### Under Consideration
- Multiple loop regions / cue points
- Preset system for modulators
- Waveform zoom controls
- Volume control slider

### Not Planned
- Direct Ableton integration (use TouchDesigner)
- Real-time MIDI control
- Multiple audio tracks

---

## System Requirements

**Browser:**
- Chrome 113+ (recommended)
- Edge 113+
- Safari Technology Preview

**Hardware:**
- Modern GPU with 4GB+ VRAM
- 8GB+ system RAM
- Any modern CPU

**APIs:**
- WebGPU
- Web Audio API
- MediaRecorder API
- Canvas 2D API

---

## File Information

**Filename:** OMS_Dojo_V2_2.html  
**Lines:** ~3,185  
**Size:** ~150 KB  
**Single-file application:** No dependencies, no build process

---

## Next Steps

1. **Read A_Deliverables.txt** for quick start
2. **Read B_Complete_Changelog.md** for full version history
3. **Read C_Developer_Documentation.md** for technical details
4. **Open OMS_Dojo_V2_2.html** and start creating!

---

## Credits

**Created by:** Wes Smith (OneManShYo)  
**Project:** OneManShYo Streaming Project  
**Status:** Production Release  
**License:** Free for personal, educational, and creative use

---

*Welcome to V2_2 - Automatic BPM detection and fine-tuning make audio-reactive shader creation faster and more precise than ever!*
