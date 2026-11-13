# OMS SHADER BAKER - COMPLETE USER GUIDE V3.5.0

**Version:** V3.5.0 Production Release  
**Date:** November 16, 2025  
**Purpose:** Complete guide to all features and workflows

---

## TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [Interface Overview](#interface-overview)
3. [Controls Tab (C)](#controls-tab-c)
4. [Presets Tab (P)](#presets-tab-p)
5. [Library Tab (L)](#library-tab-l)
6. [Code Tab (W)](#code-tab-w)
7. [Settings Tab (S)](#settings-tab-s)
8. [LIVE Tab](#live-tab)
9. [Keyboard Controls](#keyboard-controls) **← NEW IN V3.5.0**
10. [Quantized Launching](#quantized-launching) **← NEW IN V3.5.0**
11. [Workflow Examples](#workflow-examples)
12. [Tips & Tricks](#tips--tricks)
13. [Troubleshooting](#troubleshooting)

---

## GETTING STARTED

### First Time Setup

**Step 1: Open the App**
- Double-click `OMS_ShaderBaker_V3_5_0.html`
- Browser opens (Chrome or Edge recommended)
- Interface loads instantly

**Step 2: Set Application Folder**
- Click Settings tab (S)
- Click "Set Application Folder" button
- Choose a folder for your files
- This folder will store:
  - .oms containers (shader + audio + controls)
  - .omsp project states (LIVE tab arrangements)

**Step 3: You're Ready!**
- Start creating shaders in Controls (C) or Code (W)
- Save containers to Library (L)
- Arrange for performance in LIVE tab
- Use keyboard shortcuts for hands-free control **← NEW**

### System Requirements

**Browser:** WebGPU-enabled (Chrome 113+, Edge 113+)  
**OS:** macOS 10.15+, Windows 10/11, Linux (Vulkan)  
**Screen:** 1512×982+ recommended (MacBook Pro 14" optimal)  
**GPU:** Integrated OK, dedicated recommended for 4K

---

## INTERFACE OVERVIEW

### Tab Navigation

**Single-Letter Tabs:**
- **C** = Controls (shader parameter controls)
- **P** = Presets (project states .omsp)
- **L** = Library (shader containers .oms)
- **W** = WGSL (shader code editor)
- **S** = Settings (application configuration)
- **LIVE** = Performance grid (DJ/performance mode)

**Click tabs or use keyboard:** No keyboard shortcuts for tab switching (use mouse/trackpad)

### Application Sections

**Left Panel (240px):**
- Current tab content (C, P, L, W, or S)
- Shader output canvas (fixed at bottom on LIVE tab)

**Right Panel (Remainder):**
- LIVE tab: Performance grid (4 tracks × 8 scenes)
- Other tabs: Large shader preview canvas

**Total Width:** 1320px (fits MacBook Pro 14" at 1352px viewport)

---

## CONTROLS TAB (C)

### Purpose

Visual controls for shader parameters. No coding required - just adjust sliders and see instant results.

### Main Controls

**Shader Tempo (BPM):**
- Visual timing control (not audio playback)
- Range: 60-200 BPM
- Affects shader animation speed
- Independent from audio BPM

**Render Resolution:**
- 1920×1080 (Full HD, recommended)
- 2560×1440 (2K, higher quality)
- 3840×2160 (4K, maximum quality, slow render)

**Aspect Ratio:**
- 16:9 (widescreen, standard)
- 1:1 (square, Instagram)
- 4:3 (classic)
- 21:9 (ultrawide)

### Audio Reactivity

**Three Modulators:**

**modA (Bass):**
- Default: 30-500 Hz
- Targets: Kick drum, bassline
- Use for: Big movements, impacts, scale

**modB (Mids):**
- Default: 500-2000 Hz
- Targets: Vocals, synths, leads
- Use for: Color changes, brightness

**modC (Treble):**
- Default: 2000-20000 Hz
- Targets: Hi-hats, cymbals, percussion
- Use for: Fast flickering, sparkle

**Customizable Ranges:**
- Click frequency range fields
- Enter custom Hz values
- Adjust to match your music

### Rendering

**Render Video Button:**
1. Click "Render Video"
2. Select duration (or use audio length)
3. Wait for render (real-time or faster)
4. Video saves to Downloads folder
5. Convert to ProRes 422 with FFmpeg
6. Import to Ableton Live + EboSuite

**See:** FFmpeg Video Guide (G file) for conversion instructions

---

## PRESETS TAB (P)

### Purpose

Save and load complete project states (.omsp files). Store your entire LIVE tab arrangement for future performances.

### What's Saved

**Grid State:**
- All 32 cell references (which .oms containers are loaded)
- Master tempo setting
- Zoom value
- Track volumes
- Mute/solo states
- Master volume

**Track Data:**
- Waveform data for each track
- Beat grid positions (bar lines, phrase markers)
- Audio timing information

**NOT Saved:**
- Audio buffers (too large)
- Shader code (stored in .oms containers)
- Keyboard selection state (always resets to defaults)

### Usage

**Save Project State:**
1. Arrange LIVE tab (load containers, set volumes, etc.)
2. Click Presets tab (P)
3. Click "Save Project State"
4. File saves to application folder as `.omsp`

**Load Project State:**
1. Click Presets tab (P)
2. Click project state from list
3. LIVE tab loads arrangement
4. Audio buffers load from original .oms files

**Requirements:**
- Original .oms containers must exist
- Must be in same application folder
- If .oms moved/deleted, cells won't load

---

## LIBRARY TAB (L)

### Purpose

Browse and load .oms containers into LIVE tab grid cells.

### What's Displayed

**Container List:**
- All .oms files in application folder
- Filename
- File size
- Click to load into selected cell (not implemented yet)

**Current Workflow:**
- Containers created in Controls/Code tabs
- Saved via "Save Container" button
- Appear in Library automatically
- Drag to LIVE tab grid cells (manual process)

---

## CODE TAB (W)

### Purpose

Direct WGSL shader code editing for advanced users. Full control over shader behavior.

### Code Editor

**Features:**
- Syntax highlighting (basic)
- Line numbers
- Monospace font
- Real-time preview (updates on focus loss)

**Editing:**
1. Modify shader code
2. Click outside editor (or press Esc)
3. Shader compiles automatically
4. Preview updates
5. Errors show in console

### WGSL Shader Structure

```wgsl
// Uniforms (values from app)
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32;
@group(0) @binding(3) var<uniform> uModB: f32;
@group(0) @binding(4) var<uniform> uModC: f32;

// Vertex shader (full-screen quad)
@vertex
fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    var pos = array<vec2<f32>, 4>(
        vec2(-1.0, -1.0), vec2(1.0, -1.0),
        vec2(-1.0, 1.0), vec2(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

// Fragment shader (your creativity here!)
@fragment
fn fs_main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = fragCoord.xy / uResolution;
    var color = vec3<f32>(uv.x, uv.y, 0.5); // Example gradient
    return vec4<f32>(color, 1.0);
}
```

**See:** Claude Shader Creation Guide (E file) for detailed help

---

## SETTINGS TAB (S)

### Purpose

Application-level configuration. One-time setup, rarely changed.

### Controls

**Set Application Folder:**
- Click button
- Choose folder
- Browser remembers permission
- All .oms and .omsp files stored here

**Folder Info:**
- Shows current folder path
- Shows folder access status
- Click "Set Application Folder" to change

**Why Separate Tab:**
- Settings ≠ Library content
- Clear separation of concerns
- One-time setup vs frequent access

---

## LIVE TAB

### Overview

**Purpose:** Performance interface for DJ sets with audio-reactive shaders.

**Layout:**
- **Left:** SCENES column (8 scene buttons)
- **Middle:** 4 TRACK columns (4 tracks × 8 scenes = 32 cells)
- **Right:** MASTER column (master volume, zoom, shader output)

### Performance Grid

**Grid Structure:**
- 4 tracks (columns) × 8 scenes (rows) = 32 cells
- Each cell can hold one .oms container
- Drag containers from Library to cells
- Click cell to launch track
- Click scene button to launch entire scene (4 tracks) **← NEW**

**Grid Cells:**
- Empty: Grey with dashed border
- Loaded: Shows container filename
- Playing: Highlighted (to be implemented)

### Mixer Controls

**Per Track:**

**Volume Knob (Rotary):**
- Range: 0% to 100%
- Drag to adjust
- Keyboard: ↑↓ on selected track **← NEW**
- Visual: Arc shows level, pointer shows position

**Mute Button:**
- Click to toggle
- Visual: Button highlights when muted
- Keyboard: , (comma) on selected track **← NEW**

**Solo Button:**
- Click to toggle
- Solos this track (mutes all others)
- Visual: Button highlights when solo
- Keyboard: . (period) on selected track **← NEW**

**Peak Meter:**
- Real-time audio level display
- Green = safe, Yellow = loud, Red = clipping
- Peak hold with decay

**Waveform Display:**
- Bar-style waveform (Serato-style)
- Beat grid overlaid (cyan bar lines, magenta phrase markers)
- Scrolls during playback (synced to master tempo)
- Auto-generated from audio file

**Waveform Controls:**
- LOAD: Load audio file for this track
- CLEAR: Clear audio and waveform
- BPM: Set track's original BPM (for time-stretching)

### Master Controls

**Master Volume Knob:**
- Controls overall output level
- Range: 0% to 100%
- Affects all tracks simultaneously
- Keyboard: ↑↓ when Master selected (press 5) **← NEW**

**Zoom Knob:**
- Controls waveform zoom level
- Range: 10-200 pixels per second
- Affects all tracks simultaneously
- Keyboard: ↑↓ when Master selected **← NEW**
- Higher = more zoomed in (fewer seconds visible)
- Lower = more zoomed out (more seconds visible)

**Tempo Control:**
- Master tempo (BPM) for entire session
- Range: 60-200 BPM
- All tracks time-stretch to match this tempo
- Change during performance for dramatic effects

### Scene Buttons

**Purpose:** Launch entire scenes (all 4 tracks simultaneously)

**Usage:**
- Click scene button to launch all 4 tracks
- Keyboard: W/S to select, Enter to launch **← NEW**
- Quantized launching: Clips wait for next bar **← NEW**
- Perfect sync every time

**Visual Feedback:**
- Cyan border on selected scene **← NEW**
- Scene number (0-7)

### Shader Output (Fixed at Bottom)

**Purpose:** Live preview of currently rendering shader

**Location:** Bottom of left panel (doesn't scroll)

**Visibility:** Only visible on LIVE tab (hidden on C, P, L, W, S)

**Usage:**
- Monitor shader while browsing Library
- See real-time shader output during performance
- Verify audio reactivity working

---

## KEYBOARD CONTROLS

**← NEW IN V3.5.0**

### Overview

Complete keyboard-based performance control. No mouse required during performance. Professional DJ workflow like Ableton Live, Traktor, Serato.

### Track Selection

**Keys:**
- **1** = Select Track 1
- **2** = Select Track 2
- **3** = Select Track 3
- **4** = Select Track 4
- **5** = Select Master

**Tab Cycling:**
- **Tab** = Select next track (cycles: 1→2→3→4→M→1)
- **Shift+Tab** = Select previous track (cycles: M→4→3→2→1→M)

**Visual Feedback:**
- Cyan ring around selected volume knob
- 2px stroke, clearly visible

### Volume Control

**Keys:**
- **↑** = Increase volume (+5%, 0.05 steps)
- **↓** = Decrease volume (-5%, 0.05 steps)

**Behavior:**
- Affects currently selected track
- Range: 0.0 to 1.0 (0% to 100%)
- Instant audio and visual feedback
- Knob arc and pointer update in real-time

**Example:**
```
Press '2' (select Track 2)
Press ↑ (volume increases to 0.80)
Press ↑ (volume increases to 0.85)
Press ↓ (volume decreases to 0.80)
```

### Mute/Solo

**Keys:**
- **,** (comma) = Toggle mute on selected track
- **.** (period) = Toggle solo on selected track

**Mute Behavior:**
- Mutes selected track only
- Other tracks unaffected
- Visual: Mute button highlights

**Solo Behavior:**
- Solos selected track (mutes all others)
- Click solo on another track: that track solos instead
- Click solo again: all tracks unmute
- Visual: Solo button highlights

**Example:**
```
Press '1' (select Track 1)
Press ',' (Track 1 mutes)
Press '2' (select Track 2)
Press '.' (Track 2 solos, Tracks 1/3/4 mute)
Press '.' (Track 2 unsolo, all tracks unmute)
```

### Scene Navigation

**Keys:**
- **W** = Select previous scene (up)
- **S** = Select next scene (down)
- **Enter** = Launch entire scene (all 4 tracks)

**Behavior:**
- Wraps around (scene 0→1→...→7→0)
- Scene mode (default, no cell selection)
- Visual: Cyan border on selected scene (2px)

**Quantized Launching:**
- First scene/clip: Launches immediately (200ms latency)
- Subsequent scenes: Wait for next bar boundary
- Perfect sync ("soldiers at the goalpost")

**Example:**
```
Press 'W' (select Scene 0)
Press Enter (launch all 4 tracks in Scene 0)
[wait 8 bars]
Press 'S' (select Scene 1)
Press Enter (Scene 1 launches on next bar, perfectly synced)
```

### Cell Navigation

**Keys:**
- **A** = Move left (activates cell mode)
- **D** = Move right (activates cell mode)
- **W** = Move up (in cell mode)
- **S** = Move down (in cell mode)
- **Enter** = Launch single track

**Mode Activation:**
- Pressing A or D activates cell mode
- Visual: Yellow inset box-shadow on selected cell (3px)
- Mode: Scene mode (cyan) vs Cell mode (yellow)

**Cell Mode Behavior:**
- Navigate 4 columns × 8 rows grid
- Wraps around at edges
- Enter launches SINGLE track (not entire scene)
- After launch: Returns to scene mode automatically

**Example:**
```
Press 'A' (cell mode activates, move left)
Press 'D' (move right to Track 2)
Press 'S' (move down to Scene 1)
Press Enter (launch Track 2, Scene 1 only)
[Cell mode deactivates, returns to scene mode]
```

### Transport Controls

**Keys:**
- **Space** = Global play/pause (suspend/resume all audio)
- **Esc** = Emergency stop (kill all audio, clear master clock)

**Space (Play/Pause):**
- Pauses: `AudioContext.suspend()` (clean pause)
- Resumes: `AudioContext.resume()` (continues from pause point)
- All 4 tracks affected simultaneously
- Visual: No indicator (audio simply pauses/resumes)

**Esc (Emergency Stop):**
- Immediately stops all audio sources
- Clears master clock reference
- Resets timing system
- Use when something goes wrong
- Next clip to launch will set new master clock

**Example:**
```
[Playing 4 tracks]
Press Space (all audio pauses)
Press Space (all audio resumes)
Press Esc (emergency stop, all audio kills, master clock clears)
```

### Keyboard Shortcuts Reference

| Key | Function | Details |
|-----|----------|---------|
| **1-4** | Select Track | Track 1-4 |
| **5** | Select Master | Master volume/zoom |
| **Tab** | Next Track | Cycle forward |
| **Shift+Tab** | Prev Track | Cycle backward |
| **↑** | Volume Up | +5% |
| **↓** | Volume Down | -5% |
| **,** | Mute | Toggle on selected track |
| **.** | Solo | Toggle on selected track |
| **W** | Up | Scene mode: prev scene; Cell mode: prev row |
| **S** | Down | Scene mode: next scene; Cell mode: next row |
| **A** | Left | Activate cell mode, move left |
| **D** | Right | Activate cell mode, move right |
| **Enter** | Launch | Scene mode: all tracks; Cell mode: single track |
| **Space** | Play/Pause | Global pause/resume |
| **Esc** | Stop | Emergency stop, clear master clock |

---

## QUANTIZED LAUNCHING

**← NEW IN V3.5.0**

### Overview

All clips launch **quantized to bar boundaries**. This ensures perfect beat-grid alignment - the "soldiers at the goalpost" precision required for professional DJ mixing.

### How It Works

**Master Clock System:**
1. **First clip launches** → Sets master clock reference time
2. **Subsequent clips wait** → Calculate next bar boundary
3. **Launch on bar** → Perfect alignment with first clip
4. **Maintain sync** → All tracks stay aligned during playback

**Timeline:**
```
Time:        10.0s   10.5s   12.0s   14.0s   16.0s
             │       │       │       │       │
Clip 1:      [START] ······················
                     │       │       │       │
User input:          PRESS   │       │       │
                     ENTER   │       │       │
                             │       │       │
Clip 2:                      [WAIT]  [START] ··
                                     ^
                                     Launches on next bar (12.0s)
```

**Formula:**
```
currentBar = floor((currentTime - masterClockStartTime) / secondsPerBar)
nextBar = currentBar + 1
launchTime = masterClockStartTime + (nextBar × secondsPerBar)
```

### Master Clock Reference

**What It Is:**
- Global timing reference (AudioContext time)
- Set by first clip to launch
- All other clips sync to this reference

**When It's Set:**
- First clip launches in a session
- Emergency stop (Esc) clears it
- Next clip to launch sets new reference

**What It Stores:**
- `masterClockStartTime`: AudioContext time when first clip started
- `masterClockBeatIndex`: Current beat number (0, 1, 2, 3, 4, ...)

**Why It Matters:**
- One source of truth for timing
- Prevents drift between tracks
- Maintains "soldiers at the goalpost" alignment

### Bar Calculation

**Seconds Per Bar:**
```
secondsPerBar = (4 beats/bar ÷ masterTempo) × 60 seconds/minute
```

**Examples:**
- 120 BPM: (4/120) × 60 = 2.0 seconds per bar
- 128 BPM: (4/128) × 60 = 1.875 seconds per bar
- 140 BPM: (4/140) × 60 = 1.714 seconds per bar

**Current Bar:**
```
elapsedTime = currentTime - masterClockStartTime
currentBar = floor(elapsedTime / secondsPerBar)
```

**Next Bar:**
```
nextBar = currentBar + 1
nextBarTime = masterClockStartTime + (nextBar × secondsPerBar)
```

### Timing Precision

**Web Audio API Scheduling:**
- Sample-accurate: `source.start(scheduledTime)`
- Precision: ±0.001ms (one sample at 48kHz)
- No setTimeout/setInterval (those drift)

**Playback Rate Adjustment:**
- All tracks play at master tempo
- Formula: `playbackRate = masterTempo / trackBPM`
- Example: Master 120, track 128 → rate 0.9375 (slower)
- Audio pitch-shifts, beat grid spacing unchanged

**Result:**
- Zero accumulated drift
- Perfect sync regardless of track BPMs
- Professional timing standards

### Visual Alignment

**Beat Grid:**
- **Cyan lines** = Bar lines (every bar)
- **Magenta lines** = Phrase markers (every 4 bars)
- All tracks aligned vertically
- "Soldiers at the goalpost" - Bar 1s at playhead

**Waveform Scrolling:**
- Synced to master tempo
- Scrolls at master tempo speed
- All tracks scroll together
- Visual and audio perfectly matched

### User Experience

**What You Do:**
- Press Enter to launch (scene or cell)
- Clips wait for next bar automatically
- No perfect timing required

**What Happens:**
- Clip schedules itself for next bar
- Launches exactly on bar boundary
- Perfect sync with other clips
- Professional results every time

**Example:**
```
You press Enter at 10.5s (middle of Bar 5)
Current bar: 5
Next bar: 6
Next bar time: 12.0s
Wait: 1.5s
Clip launches: 12.0s (exactly on Bar 6)
Result: Perfect alignment with other tracks
```

---

## WORKFLOW EXAMPLES

### Example 1: Create and Save a Shader

**Goal:** Make a simple pulsing circle shader

**Steps:**
1. Click Controls tab (C)
2. Adjust shader parameters (or use defaults)
3. Load audio file (optional, for reactivity)
4. Set audio BPM if using audio
5. Click "Save Container" button
6. Container saved to application folder as `.oms`
7. Appears in Library tab (L)

**Time:** 2 minutes

---

### Example 2: Arrange a 4-Scene DJ Set

**Goal:** Create performance arrangement

**Steps:**
1. Click Library tab (L)
2. Drag 4 containers to Scene 0 (cells 0-3)
3. Drag 4 containers to Scene 1 (cells 4-7)
4. Drag 4 containers to Scene 2 (cells 8-11)
5. Drag 4 containers to Scene 3 (cells 12-15)
6. Click LIVE tab
7. Set master tempo (e.g., 128 BPM)
8. Adjust track volumes as needed
9. Click Presets tab (P)
10. Click "Save Project State"
11. Project saved as `.omsp` file

**Time:** 10 minutes

---

### Example 3: Keyboard-Only Performance

**Goal:** Perform using only keyboard shortcuts

**Steps:**
1. Load project state (4 scenes arranged)
2. Press 'W' to select Scene 0 (cyan border)
3. Press Enter to launch all 4 tracks (quantized)
4. Press '1' to select Track 1 (cyan ring on knob)
5. Press ↑↑↑ to increase Track 1 volume 15%
6. Press 'S' to select Scene 1
7. Press Enter to launch Scene 1 (waits for next bar)
8. Press '2', ',' to mute Track 2 (breakdown)
9. Press '2', ',' to unmute Track 2 (build back)
10. Press 'S' to select Scene 2 (drop scene)
11. Press Enter to launch Scene 2 (perfectly timed)
12. Continue performing hands-free!

**Time:** 3 minutes per transition

**Result:** Professional DJ set with zero mouse interaction!

---

### Example 4: Multi-BPM Track Mixing

**Goal:** Mix tracks with different BPMs

**Setup:**
- Track 1: 124 BPM
- Track 2: 126 BPM
- Track 3: 128 BPM
- Track 4: 130 BPM
- Master Tempo: 120 BPM

**Steps:**
1. Load all 4 tracks with correct BPMs set
2. Press 'W' to select Scene 0
3. Press Enter to launch all 4 tracks
4. All tracks time-stretch to 120 BPM
5. All Bar 1s align at playhead
6. Change master tempo to 140 BPM during performance
7. All tracks speed up to 140 BPM
8. Perfect sync maintained

**Result:** Mix any BPMs together seamlessly!

---

## TIPS & TRICKS

### Performance Tips

**Practice Offline:**
- Load project state
- Practice keyboard shortcuts
- Build muscle memory
- No audience pressure

**Plan Your Set:**
- Low energy scenes first (0-2)
- Build to high energy (3-5)
- Peak at scene 6
- Outro at scene 7

**Use Mute Creatively:**
- Drop individual tracks for breakdowns
- Build tension by removing elements
- Bring back on next bar for impact

**Master Tempo Strategy:**
- Choose stable tempo for set (120-128 BPM typical)
- Change tempo between scenes, not during
- Practice tempo changes before live

**Emergency Stop:**
- Esc key is your friend
- Use if audio glitches
- Clears master clock
- Start fresh

### Shader Creation Tips

**Start Simple:**
- Basic shapes first (circles, rectangles)
- Add complexity gradually
- Test audio reactivity incrementally

**Use Claude AI:**
- Paste shader code
- Describe what you want
- Claude generates new shader
- Copy back to Code tab

**Audio Reactivity:**
- modA for big movements (kick)
- modB for color changes (synths)
- modC for details (hi-hats)

**Render Quality:**
- Test at 720p first (fast)
- Final export at 1080p or higher
- 4K only if needed (very slow)

### File Management Tips

**Naming Convention:**
```
Genre_Energy_Description_BPM.oms

Examples:
Techno_High_PulsingCircles_128.oms
House_Mid_ColorWaves_124.oms
Ambient_Low_SlowBreathe_90.oms
```

**Organization:**
- Group by BPM (120-129, 130-139, etc.)
- Group by energy level (low, mid, high)
- Group by visual style (geometric, organic, abstract)

**Backup:**
- Copy application folder regularly
- Export .omsp project states
- Keep source .oms files safe

---

## TROUBLESHOOTING

### Audio Issues

**Problem:** Clips not launching

**Solutions:**
- Check: Is master tempo set?
- Check: Are .oms containers loaded in cells?
- Check: Is audio buffer loaded? (console logs)
- Check: Is AudioContext suspended? (press Space)

**Problem:** Clips out of sync

**Solutions:**
- Check: Are track BPMs set correctly?
- Check: Is master clock reference set? (first clip launched)
- Solution: Press Esc to reset, start fresh

**Problem:** No audio playing

**Solutions:**
- Check: Is volume knob at 0?
- Check: Is track muted? (mute button highlighted)
- Check: Are other tracks solo'd? (solo button highlighted)
- Check: Is master volume at 0?
- Check: Browser audio enabled?

### Visual Issues

**Problem:** Shader not updating

**Solutions:**
- Check: WebGPU supported? (Chrome 113+, Edge 113+)
- Check: GPU driver updated?
- Check: Console errors?
- Solution: Refresh page, try again

**Problem:** Selection not showing (cyan ring, yellow box)

**Solutions:**
- Check: Is track selected? (press 1-4)
- Check: Is scene selected? (press W/S)
- Check: Is cell mode active? (press A/D)
- Solution: Click scene/cell, then try keyboard

**Problem:** Waveforms not displaying

**Solutions:**
- Check: Is audio loaded? (LOAD button)
- Check: Is BPM set correctly?
- Check: Console errors during load?
- Solution: Clear and reload audio file

### Keyboard Issues

**Problem:** Shortcuts not working

**Solutions:**
- Check: Is LIVE tab active? (click tab)
- Check: Page has focus? (click canvas)
- Check: Not in address bar? (click page content)
- Check: Caps Lock off? (keys are case-sensitive for some)
- Solution: Click page, try again

**Problem:** Wrong track selected

**Solutions:**
- Check: Press correct number (1-4 for tracks, 5 for master)
- Check: Visual feedback (cyan ring on knob)
- Solution: Press desired track number again

### Performance Issues

**Problem:** Low FPS (stuttering visuals)

**Solutions:**
- Lower render resolution (1080p → 720p)
- Close other browser tabs
- Close other applications
- Disable browser extensions
- Update GPU drivers

**Problem:** High CPU usage

**Solutions:**
- Lower zoom level (fewer pixels drawn)
- Simplify shader code (fewer calculations)
- Reduce FFT size (32768 → 16384, not recommended)

### File Issues

**Problem:** .oms won't load

**Solutions:**
- Check: File in application folder?
- Check: File not corrupted? (try opening in text editor)
- Check: JSON valid? (proper syntax)
- Solution: Re-save from original shader

**Problem:** .omsp project state won't load

**Solutions:**
- Check: Original .oms files exist?
- Check: Files in same application folder?
- Check: File not corrupted?
- Solution: Re-save project state

---

## KEYBOARD SHORTCUTS QUICK REFERENCE

```
┌─────────────────────────────────────────────────────┐
│              KEYBOARD SHORTCUTS                     │
├─────────────────────────────────────────────────────┤
│ Track Selection:                                    │
│   1-4      Select Tracks 1-4                        │
│   5        Select Master                            │
│   Tab      Next Track                               │
│   Shift+Tab Previous Track                          │
├─────────────────────────────────────────────────────┤
│ Volume Control (on selected track):                 │
│   ↑        Increase Volume (+5%)                    │
│   ↓        Decrease Volume (-5%)                    │
├─────────────────────────────────────────────────────┤
│ Mute/Solo (on selected track):                      │
│   ,        Toggle Mute                              │
│   .        Toggle Solo                              │
├─────────────────────────────────────────────────────┤
│ Scene Navigation (scene mode):                      │
│   W        Previous Scene                           │
│   S        Next Scene                               │
│   Enter    Launch All Tracks                        │
├─────────────────────────────────────────────────────┤
│ Cell Navigation (cell mode):                        │
│   A/D      Activate Cell Mode, Move Left/Right     │
│   W/S      Move Up/Down                             │
│   Enter    Launch Single Track                      │
├─────────────────────────────────────────────────────┤
│ Transport:                                          │
│   Space    Play/Pause (global)                      │
│   Esc      Emergency Stop                           │
└─────────────────────────────────────────────────────┘

Visual Feedback:
  Cyan Ring     = Selected Volume Knob
  Cyan Border   = Selected Scene
  Yellow Box    = Selected Cell
```

---

**END OF COMPLETE USER GUIDE**

Version: V3.5.0 Production Release  
Date: November 16, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com
