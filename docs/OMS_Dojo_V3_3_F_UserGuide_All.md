# OMS SHADER BAKER V3.3.0 - COMPLETE USER GUIDE

**Version:** V3.3.0  
**Date:** November 2025  
**Purpose:** Complete user manual for all features

---

## TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [CONTROLS Tab](#controls-tab)
3. [AUDIO Tab](#audio-tab)
4. [RENDER Tab](#render-tab)
5. [LIBRARY Tab](#library-tab)
6. [LIVE Tab](#live-tab)
7. [Workflows](#workflows)
8. [Tips & Tricks](#tips--tricks)

---

## GETTING STARTED

### System Requirements

**Browser:**
- Chrome 113+ (recommended)
- Safari Technology Preview (WebGPU required)

**Hardware:**
- 8GB RAM minimum (16GB recommended)
- Dedicated GPU recommended
- Audio interface optional

### First Launch

1. Open `OMS_ShaderBaker_V3_3_0.html` in supported browser
2. Grant permissions if prompted (file access, audio)
3. App loads to CONTROLS tab with default shader

---

## CONTROLS TAB

### Purpose
Create and edit WebGPU shaders with real-time preview

### Layout

**Left Side:** Shader editor (WGSL code)  
**Right Side:** Live preview canvas  
**Bottom:** Control panel (modulators, frequency mapping)

### Shader Editor

**Features:**
- Syntax highlighting
- Line numbers
- Real-time compilation
- Error reporting

**Default Shader:** Gradient (always loads on startup)

### Control Panel

**Modulators (Mod A/B/C):**
- Range: 0.0 to 1.0
- Rotary knobs for precise control
- Click-drag to adjust
- Double-click to reset to 0.5

**Frequency Mapping:**
- Low: Default 30-75 Hz (kick drums, sub-bass)
- Mid: Default 75-500 Hz (vocals, synths)
- High: Default 500-5000 Hz (hi-hats, cymbals)
- Editable: Type to change ranges

**Audio Reactivity:**
- FFT analysis (32768 samples)
- Real-time frequency extraction
- Smooth value interpolation

### Workflow

1. Write/edit WGSL shader code
2. Shader auto-compiles on change
3. Preview updates in real-time
4. Adjust modulators for effect
5. Save when satisfied (LIBRARY tab)

---

## AUDIO TAB

### Purpose
Load and configure audio for shader reactivity and rendering

### Audio Loading

**Method 1:** Drag-and-drop audio file onto canvas  
**Method 2:** Click canvas to browse files

**Supported Formats:**
- MP3, WAV, FLAC, M4A
- Any format supported by Web Audio API

### Audio Controls

**Playback:**
- Play/Pause button
- Loop toggle
- Volume slider

**Timing:**
- Start Time: Set loop in point (seconds)
- End Time: Set loop out point (seconds)
- Duration: Auto-calculated

**BPM:**
- Manual entry (30-300 BPM)
- Used for render loop calculation
- Affects render length

**Pitch:**
- Range: -12 to +12 semitones
- For tuning audio to shader tempo
- Visual only (doesn't affect actual playback in LIVE tab)

### Waveform Display

**Features:**
- Full audio waveform visualization
- Click to set playhead position
- Visual loop in/out markers
- Zoom controls

---

## RENDER TAB

### Purpose
Export shader as video file for use in Ableton/EboSuite

### Render Settings

**Duration:**
- Start Time: From AUDIO tab
- End Time: From AUDIO tab
- Calculated: Based on BPM and bar count

**Quality:**
- Width: 1920, 3840, etc.
- Height: 1080, 2160, etc.
- FPS: 30, 60
- Format: WebM (then convert to ProRes)

**Bar-Based:**
- Calculate end time from BPM
- E.g., "16 bars at 128 BPM" = exact length

### Render Process

1. Set start/end times in AUDIO tab
2. Choose resolution and FPS
3. Click "Start Render"
4. Browser captures frames (takes time)
5. Download .webm file
6. Convert to ProRes with FFmpeg

### Post-Processing

See `OMS_ShaderBaker_V3_3_0_G_UserGuide_FFmpeg_Video_Guide.md` for:
- FFmpeg installation
- ProRes conversion commands
- Ableton/EboSuite integration

---

## LIBRARY TAB

### Purpose
Manage .oms shader preset files

### Setup

**First Time:**
1. Click "Set Library Folder"
2. Choose folder containing .oms files
3. Permission granted (persists)

**Folder Structure:**
```
MyShaders/
  ├── Gradient.oms
  ├── Plasma.oms
  ├── Tunnel.oms
  └── ...
```

### Container Library

**Features:**
- Visual thumbnails (64×64 PNG)
- Alphabetical listing
- Click to load preset
- Scroll through collection

**Thumbnail Generation:**
- Auto-created when saving preset
- Captures current shader frame
- Stored in .oms file as base64

### Preset Management

**Save Preset:**
1. Create shader in CONTROLS tab
2. Switch to LIBRARY tab
3. Click "Save Current Preset"
4. Enter filename (e.g., "MyShader.oms")
5. Preset saved to library folder

**Load Preset:**
1. Click thumbnail in Container Library
2. Shader code loads into CONTROLS tab
3. All control states restored
4. Audio reference loaded (if path valid)

### .oms File Format

**Contents:**
- Shader code (WGSL)
- Control states (modulators, frequencies)
- Audio reference path (relative)
- Audio timing (BPM, start/end, pitch)
- Thumbnail (base64 PNG)

**Portability:**
- Share .oms files between users
- Audio paths may need adjustment
- Shaders always portable

---

## LIVE TAB

### Purpose
DJ performance mixing with audio-reactive shaders

### Layout

**Left Sidebar:**
- Container Library (preset selection)
- Shader Preview (16:9 canvas, beta)
- Set Library Folder button

**Center:**
- 4 tracks × 8 scenes grid
- Cell thumbnails
- Track waveforms with beat grids
- Synced scrolling

**Right Sidebar:**
- Master Tempo (60-200 BPM rotary)
- Zoom (10-200 px/sec rotary)
- Mixer per track:
  - Volume knob
  - Mute button (M)
  - Solo button (S)
  - VU meter

**Bottom:**
- Waveform control buttons (Q, |<, |>, <<, >>)

### Performance Grid (4×8)

**Tracks:** Horizontal rows (1-4)  
**Scenes:** Vertical columns (1-8)

**Cell Behavior:**
- Click cell → Load .oms preset to that track/scene
- Thumbnail shows shader preview
- Audio auto-loads if referenced in .oms

### Loading Tracks

**Method 1:** Click cell, choose .oms preset  
**Method 2:** Drag .oms file onto cell  
**Method 3:** Drag audio file onto waveform (no preset)

**Auto-Play:**
- Loading cell with audio starts playback
- Positioned at "Bar 1" (audioStartTime)
- Time-stretched to master tempo

### Beat Grid System

**Visual Elements:**
- Cyan bars: Every musical bar (4 beats)
- Magenta markers: Every 4 bars (phrase markers)
- Bar numbers: 1, 5, 9, 13... (aligned with magenta)
- Waveform bars: 1 per beat (beat-aligned)

**Master Tempo:**
- Controls session-wide tempo (60-200 BPM)
- All tracks time-stretch to master tempo
- Playback rate = masterTempo ÷ trackBPM
- Visual scale = trackBPM ÷ masterTempo

**Perfect Sync:**
- "One Timeline" architecture
- All tracks use same pixels-per-second
- Tempo changes update instantly
- Zero drift during playback

### Zoom Control

**Range:** 10-200 pixels per second  
**Default:** 40 px/sec (30 seconds visible)

**Dynamic Minimum:**
- Automatically set to show entire longest track
- Updates when tracks load/unload
- Can't zoom out past longest track

**Usage:**
- Zoom in: See detail, precise timing
- Zoom out: Overview, song structure

### Mixer Controls

**Volume (Per Track):**
- Rotary knob (0-100%)
- Click-drag to adjust
- Affects only that track

**Mute:**
- Toggle button (gray = normal, red = muted)
- Silences track (volume to 0)
- Solo takes priority over mute

**Solo:**
- Toggle button (gray = normal, yellow = soloed)
- Only soloed tracks audible
- Multiple tracks can be soloed
- Overrides mute state

**VU Meters:**
- Real-time level metering
- Peak hold indicator
- Green/yellow/red zones

### Keyboard Shortcuts

**Transport:**
- Spacebar: Play/Pause all tracks
- Esc: Stop all tracks

**Navigation:**
- Tab: Switch between tabs
- Arrow keys: Navigate grid

**Mixer:**
- M: Mute selected track
- S: Solo selected track

### Shader Preview (BETA)

**Location:** Left sidebar, below Container Library

**Features:**
- Real-time WebGPU rendering
- 16:9 aspect ratio (640×360)
- Independent render loop (60fps)
- Default gradient shader

**Current Status:**
- Shows default gradient only
- No sync with Controls tab yet
- No audio reactivity yet
- Foundation for track compositing

**Future (V3.4.0+):**
- Sync with Controls tab shader
- Audio reactivity (FFT from master output)
- Composite 4 track shaders
- Visual blend modes

### Project States (.omsp)

**Purpose:** Save/load entire LIVE session

**Contents:**
- Grid state (master tempo, zoom)
- All track configurations
- Loaded .oms presets per cell
- Audio file paths
- Mixer settings (volume, mute, solo)

**Save Project:**
1. Configure LIVE session
2. Click "Save Project State"
3. Choose location/filename
4. .omsp file created

**Load Project:**
1. Click "Load Project State"
2. Choose .omsp file
3. Entire session restored
4. Audio/presets reload if paths valid

---

## WORKFLOWS

### Workflow 1: Create Shader + Video

**Goal:** Make audio-reactive shader video for Ableton

1. **AUDIO Tab:**
   - Load audio file
   - Set BPM (e.g., 128)
   - Set start/end times (e.g., 0-120 sec)

2. **CONTROLS Tab:**
   - Write/edit shader
   - Test with audio playback
   - Adjust modulators

3. **LIBRARY Tab:**
   - Save as .oms preset

4. **RENDER Tab:**
   - Set resolution (1920×1080)
   - Set FPS (60)
   - Start render → .webm file

5. **FFmpeg:**
   - Convert to ProRes 422

6. **Ableton:**
   - Load in EboSuite video track
   - Sync with audio

### Workflow 2: DJ Performance

**Goal:** Mix 4 tracks with synchronized visuals

1. **LIBRARY Tab:**
   - Set library folder
   - Ensure .oms presets available

2. **LIVE Tab:**
   - Load tracks (click cells, choose presets)
   - Audio auto-starts

3. **Setup:**
   - Set master tempo (e.g., 128 BPM)
   - Adjust zoom for overview

4. **Mix:**
   - Use volume/mute/solo per track
   - Beat grids keep timing aligned
   - Adjust tempo live (tracks stay synced)

5. **Perform:**
   - Navigate song structure with zoom
   - Visual timing reference (phrase markers)
   - Professional sync like Traktor/Ableton

### Workflow 3: Build Preset Library

**Goal:** Create collection of reusable shaders

1. **Research:**
   - Find shader inspiration
   - Plan visual concepts

2. **Create:**
   - CONTROLS tab: Write shaders
   - Use Claude AI for generation (see Guide E)

3. **Test:**
   - AUDIO tab: Test with different audio
   - Verify reactivity works well

4. **Save:**
   - LIBRARY tab: Save each shader
   - Use descriptive names

5. **Organize:**
   - Folder structure in library
   - Subcategories (geometric, organic, abstract)

6. **Perform:**
   - LIVE tab: Quick access to library
   - Mix and match in performance

---

## TIPS & TRICKS

### Audio Reactivity

**Kick Detection:**
- Set Freq Low: 30-75 Hz
- Use audioLow for punch effects
- Multiply by 2-3x for stronger response

**Vocal Response:**
- Set Freq Mid: 200-800 Hz
- Use audioMid for melodic visuals
- Smooth with moving average

**Hi-Hat Sparkle:**
- Set Freq High: 8000-16000 Hz
- Use audioHigh for detail/noise
- Add to existing effects subtly

### Performance

**Multi-BPM Mixing:**
- Load 124, 126, 128 BPM tracks
- Set master tempo to 126 BPM
- All tracks sync perfectly
- Tempo-stretch happens automatically

**Phrase Alignment:**
- Use magenta markers for drops/builds
- Align phrase markers across tracks
- Easy visual cue for transitions

**Loop Points:**
- Set audioStartTime to intro end
- Set audioEndTime to outro start
- Perfect loops every time

### Shader Development

**Start Simple:**
- Begin with basic gradient
- Add one effect at a time
- Test audio reactivity incrementally

**Use Modulators:**
- Mod_A: Speed/tempo
- Mod_B: Complexity/detail
- Mod_C: Color/hue
- Make everything tweakable

**Optimize:**
- Avoid complex calculations in loops
- Use built-in functions
- Test on target hardware

### File Management

**Library Organization:**
```
ShaderLibrary/
  ├── Geometric/
  │   ├── Tunnel.oms
  │   ├── Kaleidoscope.oms
  │   └── Grid.oms
  ├── Organic/
  │   ├── Plasma.oms
  │   ├── Fluid.oms
  │   └── Flow.oms
  └── Abstract/
      ├── Noise.oms
      └── Particles.oms
```

**Naming Convention:**
- Descriptive: "BluePlasma_BassReactive.oms"
- Categorize: Prefix with style
- Version: Include iteration if needed

**Backup:**
- .oms files are small (< 1MB usually)
- Easy to backup entire library
- Share presets with other users

---

## TROUBLESHOOTING

### Common Issues

**Shader Won't Compile:**
- Check syntax (semicolons, brackets)
- Verify WGSL spec compliance
- Look for typos in function names

**No Audio Reactivity:**
- Check frequency ranges (CONTROLS tab)
- Verify audio is playing
- Increase multiplier in shader code

**Beat Grid Not Aligned:**
- Check audioBPM matches actual track BPM
- Verify audioStartTime is at Bar 1
- Ensure master tempo set correctly

**Drift During Playback:**
- Shouldn't happen in V3.3.0!
- Reload track if occurring
- Report bug with details

**WebGPU Not Available:**
- Update browser (Chrome 113+)
- Enable WebGPU in browser flags
- Try Safari Technology Preview

---

## KEYBOARD SHORTCUTS

**Global:**
- Tab: Cycle through tabs
- Ctrl/Cmd + S: Save (browser download)

**CONTROLS Tab:**
- Ctrl/Cmd + Enter: Compile shader

**LIVE Tab:**
- Spacebar: Play/Pause all
- Esc: Stop all
- M: Mute selected track
- S: Solo selected track
- +/-: Adjust zoom
- [ / ]: Adjust tempo

---

## ADVANCED FEATURES

### .oms File Structure

**Viewing:**
- .oms files are JSON
- Open in text editor to see structure
- Can edit manually (advanced users)

**Sharing:**
- Share .oms files in community
- Audio paths may need updating
- Shaders always work

### Multiple Instances

**Browser Windows:**
- Open multiple OMS Shader Baker windows
- Each independent
- Useful for comparison/testing

**BroadcastChannel:**
- Instances can communicate (future)
- Sync tempo across instances
- Collaborative performance (vision)

---

## FUTURE FEATURES

### V3.4.0 (Planned Q1 2026)

- Scene launching (trigger rows)
- Quantized triggering
- MIDI control integration
- Shader preview sync with Controls tab

### V3.5.0 (Vision Q2 2026)

- Real-time shader rendering per track
- Track compositing (blend 4 shaders)
- Ableton Link synchronization

---

**END OF USER GUIDE**

**For More Help:**
- Developer Documentation: File C
- FFmpeg Guide: File G
- Claude Shader Guide: File E
- Release Notes: File D

**YouTube:** @ONEMANSHYO for live demonstrations
