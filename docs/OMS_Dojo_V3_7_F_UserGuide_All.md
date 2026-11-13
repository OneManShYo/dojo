# OMS SHADER BAKER V3.7.0 - COMPLETE USER GUIDE

**Version:** V3.7.0  
**Date:** November 16, 2025  
**Purpose:** Complete guide to all features and workflows

---

## TABLE OF CONTENTS

1. [Quick Start](#quick-start)
2. [Interface Overview](#interface-overview)
3. [Shader Creation](#shader-creation)
4. [Audio Reactivity](#audio-reactivity)
5. [Live Performance](#live-performance)
6. [Four-Track Compositing](#four-track-compositing)
7. [Dual-Domain Mixer](#dual-domain-mixer)
8. [Pop-Out Window](#pop-out-window)
9. [Keyboard Controls](#keyboard-controls)
10. [Settings & Preferences](#settings--preferences)
11. [File Management](#file-management)
12. [Export Workflow](#export-workflow)
13. [Tips & Tricks](#tips--tricks)

---

## QUICK START

### First Time Setup

1. **Open Application**
   - Double-click `OMS_ShaderBaker_V3_6_0.html`
   - Use Chrome, Edge, or Safari Technology Preview
   - WebGPU must be supported

2. **Set Application Folder**
   - Click Settings tab (S)
   - Click "Set Application Folder"
   - Choose folder for .oms and .omsp files
   - This is where all your work saves

3. **Create Your First Shader**
   - Click Controls tab (C)
   - Adjust sliders to see instant preview
   - Or click Code tab (W) to write WGSL
   - Canvas shows real-time preview

4. **Save Your Work**
   - Click "Save as .oms" in Controls tab
   - Name your shader
   - Appears in Library tab (L)

5. **Perform Live**
   - Click LIVE tab
   - Drag containers from library to grid
   - Click "Pop Out ↗" for projector
   - Press Space to play!

---

## INTERFACE OVERVIEW

### Header

Shows version and status information

### Left Panel (400px)

**Tab Bar:**
- **C (Controls):** Visual shader editor
- **P (Presets):** Coming soon
- **L (Library):** Container management
- **W (WGSL):** Code editor
- **S (Settings):** Application preferences
- **LIVE:** Performance interface

**Content Area:**
Scrollable content for each tab

**Shader Output (LIVE tab only):**
Fixed at bottom, shows current shader rendering

### Right Panel (920px)

**Preview Canvas:**
Real-time shader rendering with audio reactivity

**LIVE Tab Grid:**
4 tracks × 8 scenes performance grid

---

## SHADER CREATION

### Using Visual Controls (Controls Tab)

**Available Parameters:**
- **Palette:** Color scheme selection
- **Symmetry:** Kaleidoscope effect (2-8 way)
- **FFT Settings:** Audio frequency ranges
  - A (Bass): 30-500 Hz default
  - B (Mids): 500-2000 Hz default
  - C (Treble): 2000-20000 Hz default

**Workflow:**
1. Adjust sliders
2. See instant preview
3. Fine-tune settings
4. Save as .oms container

### Using Code Editor (Code Tab)

**WGSL Shader Language:**
- WebGPU Shading Language
- Similar to GLSL/HLSL
- GPU-accelerated

**Uniforms Available:**
```wgsl
@group(0) @binding(0) var<uniform> uTime: f32;        // Elapsed time
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>; // Canvas size
@group(0) @binding(2) var<uniform> uModA: f32;        // Bass 0-1
@group(0) @binding(3) var<uniform> uModB: f32;        // Mids 0-1
@group(0) @binding(4) var<uniform> uModC: f32;        // Treble 0-1
```

**Workflow:**
1. Edit WGSL code
2. Click "Apply Shader" or press Cmd/Ctrl+Enter
3. See preview update
4. Save as .oms container

---

## AUDIO REACTIVITY

### Loading Audio

1. Click "Load Audio" in Controls tab
2. Select MP3, WAV, or FLAC file
3. Set BPM accurately (critical for sync)
4. Set loop points if desired

### FFT Analysis

**32,768 sample FFT** analyzes audio in real-time

**Modulators:**
- **modA (Bass):** Low frequencies, kicks, bass
- **modB (Mids):** Vocals, synths, leads
- **modC (Treble):** Hi-hats, cymbals, percussion

**Customizing Ranges:**
1. Expand FFT Settings in Controls tab
2. Adjust frequency sliders
3. Test with audio playback
4. Save in .oms container

### Using Modulators in Shaders

**Example:**
```wgsl
// Pulse size with bass
let size = 0.5 + uModA * 0.5; // 0.5 to 1.0

// Change brightness with mids
let brightness = 0.3 + uModB * 0.7;

// Flicker with treble
let flicker = 1.0 - uModC * 0.3;
```

---

## LIVE PERFORMANCE

### Performance Grid

**Structure:**
- 4 tracks (columns)
- 8 scenes (rows)
- 32 total cells

**Loading Containers:**
- Drag from Library (L tab) to grid cells
- Or double-click cell → select container
- Empty cells = no container loaded

**Visual Feedback:**
- **Cyan outline:** Selected cell
- **Yellow border:** Playing cell
- **Both:** Selected + playing

### Scene Launching

**Scene = Row (all 4 tracks):**
1. Navigate with W/S keys
2. Press Enter to launch all 4 tracks
3. Quantized to bar boundary (after first clip)

**Empty Scene:**
Stops all 4 tracks (clip stop)

### Cell Launching

**Cell = Individual Track:**
1. Press D to enter track columns
2. Navigate with A/D (horizontal) and W/S (vertical)
3. Press Enter to launch single track
4. Press A or Esc to return to scene column

**Empty Cell:**
Stops that track only

---

## FOUR-TRACK COMPOSITING

### How It Works

**Independent Shaders:**
Each track has its own WebGPU device and pipeline

**Volume = Opacity:**
- Volume fader at 0% → shader invisible
- Volume fader at 50% → shader fully visible (100% opacity)
- Volume fader at 100% → shader fully visible (clamped)

**Layering:**
- Track 1 (left) = top layer (z-index 4)
- Track 2 = layer 3
- Track 3 = layer 2
- Track 4 (right) = bottom layer (z-index 1)

### Mixing Workflow

**Example - Two Shaders Crossfade:**
1. Load shader A into Track 1
2. Load shader B into Track 2
3. Launch both clips
4. Track 1 volume at 100%, Track 2 at 0% → see only A
5. Fade Track 1 down, Track 2 up → crossfade A to B
6. Track 1 at 0%, Track 2 at 100% → see only B

**Example - Layer 4 Shaders:**
1. Load different shader in each track
2. Launch all 4 clips
3. Set volumes: T1=100%, T2=30%, T3=20%, T4=50%
4. Shaders composite based on volume + z-index
5. Adjust volumes to taste

---

## DUAL-DOMAIN MIXER

### Overview

**V3.7.0 introduces independent shader and audio control per track!**

Each track now has TWO separate control domains:

```
┌─────────────────────────────────┐
│         [Track N]               │
├──────────────┬──────────────────┤
│   SHADER     │      AUDIO       │
│              │                  │
│  [vM] [vS]   │   [🎚️ volume]    │
│  [🎚️ opacity] │   [▮▮] [▮▮]      │
│              │   [L]   [R]      │
│              │   [aM] [aS]      │
└──────────────┴──────────────────┘
```

### Shader Domain (Left Side)

**Purple controls for VISUAL mixing:**

**Shader Opacity Knob (30px):**
- Range: 0-100%
- Color: Purple (#9d4edd)
- Default: Auto mode (follows audio volume × 2)
- Touch knob = enable manual override

**Shader Mute Button [vM]:**
- Purple when active
- Hides shader (opacity = 0)
- Audio continues playing normally
- Independent from audio mute

**Shader Solo Button [vS]:**
- Cyan when active
- Shows only this shader (others invisible)
- Audio continues playing normally on all tracks
- Independent from audio solo

**Keyboard Shortcuts:**
- `Shift+,` - Toggle shader mute
- `Shift+.` - Toggle shader solo
- `Shift+↑` - Increase opacity (+2%)
- `Shift+↓` - Decrease opacity (-2%)

### Audio Domain (Right Side)

**Standard controls for AUDIO mixing:**

**Audio Volume Knob (30px):**
- Range: 0-100%
- Color: Standard
- Controls audio output level only
- Does NOT affect shader when manual override enabled

**Level Meters (40px height):**
- L/R stereo meters
- VU-style visualization
- Shows current audio levels

**Audio Mute Button [aM]:**
- Standard appearance when active
- Silences audio (volume = 0)
- Shader continues rendering normally
- Independent from shader mute

**Audio Solo Button [aS]:**
- Standard appearance when active
- Mutes all other tracks
- Shader continues rendering normally on all tracks
- Independent from shader solo

**Keyboard Shortcuts:**
- `,` (comma) - Toggle audio mute
- `.` (period) - Toggle audio solo
- `↑` - Increase volume
- `↓` - Decrease volume

### Opacity Modes

**AUTO Mode (Default):**
- Shader opacity = audio volume × 2
- Opacity reaches 100% at 50% fader position
- Same behavior as V3.6.0
- No visual indication (default state)

**MANUAL Mode:**
- Enabled by touching shader opacity knob
- Opacity independent from audio volume
- Persists until project reload
- No visual indication (planned for V3.8.0)

**Priority Logic:**
1. Shader Mute → opacity 0 (highest priority)
2. Shader Solo → opacity 0 if not soloed
3. Manual Override → use manual value
4. Auto Mode → volume × 2
5. Not Playing → opacity 0 (lowest priority)

### Workflow Examples

**Example 1: Visual Build Without Audio Change**

Build visuals during breakdown without touching audio:

1. Audio stays at current mix (don't touch volume)
2. Touch shader opacity knobs (enable manual)
3. Fade in visuals using `Shift+↑`
4. Gradually add layers on tracks 1, 2, 3
5. Audio mix unchanged, visuals build smoothly

**Example 2: Audio Drop With Visual Hold**

Drop audio but keep visuals present:

1. Touch all shader opacity knobs (manual mode)
2. Set all shader opacity to 100%
3. Drop audio volume faders to 0%
4. Result: Visuals stay, audio silent

**Example 3: Visual Solo for Highlight**

Show only one shader while playing all audio:

1. Press `Shift+.` on track 1 (shader solo)
2. Shaders 2-4 invisible
3. All audio still playing (normal mix)
4. Perfect for highlighting specific element

**Example 4: Independent Crossfade**

Crossfade visuals while audio stays constant:

1. Tracks 1-2 playing (audio at 75% each)
2. Touch both shader opacity knobs (manual)
3. Track 1 opacity 100% → 0% (fade out)
4. Track 2 opacity 0% → 100% (fade in)
5. Audio levels unchanged throughout

### Master Mixer

**Master section has parity with tracks (V3.7.0):**

**Master Volume Knob (60px):**
- 2x size of track knobs
- Visual prominence for global control
- Controls final output level

**Master Meters (120px height):**
- 3x height of track meters
- Shows final mix levels
- Clear visual hierarchy

**Master Mute/Solo Buttons:**
- Below volume knob
- Symmetric with track layout
- Control global audio output

**Tempo Control:**
- Below M/S buttons
- Current BPM display
- ± adjustment buttons
- Integrated with master section

### Tips

**When to use Manual Override:**
- Visual builds without audio changes
- Audio drops while keeping visuals
- Independent visual/audio crossfades
- Complex layering effects

**When to keep Auto Mode:**
- Simple performances (visual follows audio)
- Quick setup (no extra control needed)
- V3.6.0-style workflow preferred
- Learning/testing phases

**Resetting to Auto:**
- Currently: Reload project
- V3.8.0 planned: Double-click opacity knob

---

## POP-OUT WINDOW

### Opening

1. Click "Pop Out ↗" button (above shader output)
2. Window opens at default size (640×360)
3. Drag to second monitor/projector
4. Resize as needed

### Size Presets

**Settings Tab:**
- 640×360 (default, lower GPU usage)
- 1280×720 (higher quality, more GPU)

**Changing:**
1. Go to Settings tab (S)
2. Select size from dropdown
3. Close and reopen pop-out to apply
4. Preference saved in .omsp files

### Always On Top

**Settings Tab:**
- Checkbox: "Always on Top"
- Keeps window above other applications
- Useful for performance setups
- May not work on all browsers/OS

### Closing

Click "Close Pop-Out ✕" button

**Note:** Pop-out closes automatically if main window closes

---

## KEYBOARD CONTROLS

### Transport

| Key | Action |
|-----|--------|
| Space | Global play/pause (all tracks) |
| Esc | Emergency stop + return to scene column |

### Track Selection

| Key | Action |
|-----|--------|
| 1-4 | Select tracks 1-4 |
| 5 | Select master track |
| Tab | Next track (cycle) |
| Shift+Tab | Previous track (cycle) |

### Navigation

**Scene Column (Default):**
| Key | Action |
|-----|--------|
| W | Previous scene (up) |
| S | Next scene (down) |
| Enter | Launch scene (all 4 tracks) |
| D | Enter Track 1 |

**Track Columns:**
| Key | Action |
|-----|--------|
| W | Previous scene (up) |
| S | Next scene (down) |
| A | Previous track (left) |
| D | Next track (right) |
| Enter | Launch cell (single track) |
| Esc | Return to scene column |

### Audio Volume Control

| Key | Action |
|-----|--------|
| ↑ | Increase selected track audio volume |
| ↓ | Decrease selected track audio volume |

### Shader Opacity Control (V3.7.0)

| Key | Action |
|-----|--------|
| Shift+↑ | Increase selected track shader opacity (+2%) |
| Shift+↓ | Decrease selected track shader opacity (-2%) |

### Audio Mute/Solo

| Key | Action |
|-----|--------|
| , (comma) | Toggle audio mute on selected track |
| . (period) | Toggle audio solo on selected track |

### Shader Mute/Solo (V3.7.0)

| Key | Action |
|-----|--------|
| Shift+, (comma) | Toggle shader mute on selected track |
| Shift+. (period) | Toggle shader solo on selected track |

### Cell Management

| Key | Action |
|-----|--------|
| Delete | Clear selected cell (removes container) |
| Backspace | Clear selected cell (Mac) |

---

## SETTINGS & PREFERENCES

### Application Folder

**Purpose:** Where .oms and .omsp files are stored

**Setting Up:**
1. Settings tab (S)
2. Click "Set Application Folder"
3. Choose folder
4. All saves/loads use this folder

### Pop-Out Window Size

**Options:**
- 640×360 (default, better performance)
- 1280×720 (higher quality)

**When to use each:**
- 640×360: Most performances, lower GPU usage
- 1280×720: High-quality capture, powerful GPU

### Always On Top

**Purpose:** Keep pop-out window above other apps

**Use cases:**
- Projector output while controlling in main window
- Second monitor display during DJ set
- Recording with OBS while mixing

---

## FILE MANAGEMENT

### .oms Containers

**Contents:**
- Shader code (WGSL)
- Audio file reference (path only, not embedded)
- FFT settings
- Control settings
- Shader Tempo (visual timing)

**Creating:**
1. Create shader in Controls or Code tab
2. Optionally load audio
3. Click "Save as .oms"
4. Enter name
5. Saved to Application Folder

**Loading:**
1. Library tab (L)
2. Click container
3. Loads into Controls/Code tab

### .omsp Project States

**Contents:**
- Grid layout (which containers in which cells)
- Master tempo
- Track volumes, mutes, solos
- Current playback state
- **V3.6.0:** Pop-out window settings

**Creating:**
1. Arrange grid in LIVE tab
2. Click "Save Project State"
3. Enter name
4. Saved to Application Folder

**Loading:**
1. Library tab (L)
2. Click "Load Project State"
3. Select .omsp file
4. Entire session restored

### Auto-Save

**Project states auto-save:**
- When changing settings
- When changing window size
- When toggling always-on-top

---

## EXPORT WORKFLOW

### Render Video

**Purpose:** Export shader as silent video for Ableton Live

**Workflow:**
1. Create and test shader
2. Load audio (optional, for reactivity)
3. Click "Render Video" in Controls tab
4. Wait for render (real-time or faster)
5. Video saves to Downloads folder

**Format:** WebM or MP4 (browser dependent)

### Convert to ProRes

**Required:** FFmpeg installed

**Command:**
```bash
ffmpeg -i shader_output.webm -c:v prores_ks -profile:v 3 output.mov
```

**See:** FFmpeg Video Guide (separate document)

### Import to Ableton

**Required:** Ableton Live + EboSuite

**Workflow:**
1. Open Ableton Live project
2. Drag ProRes video onto audio track
3. EboSuite video track appears
4. Video plays in sync with audio
5. Perform live!

---

## TIPS & TRICKS

### Performance Optimization

**For Smooth 60fps:**
1. Close other GPU apps
2. Use 640×360 pop-out (not 1280×720)
3. Simplify shader code if needed
4. Enable hardware acceleration in browser

### Mixing Techniques

**Crossfading:**
- Use volume faders to fade between shaders
- 50% fader = full visibility (intensity × 2 formula)

**Layering:**
- Stack complementary shaders (e.g., background + foreground)
- Left tracks dominate (z-index priority)

**Building/Dropping:**
- Start with minimal shaders (tracks 3-4)
- Add layers gradually (tracks 2, then 1)
- Remove layers for drops

### Shader Design

**For Live Performance:**
- High contrast (visible from audience)
- Audio-reactive (responds to music)
- Not too busy (overwhelming)
- Test at performance resolution

**For Video Export:**
- Higher complexity acceptable (not real-time)
- Optimize for single-shader rendering
- Test full duration before final render

### Workflow Tips

**Organize Containers:**
- Name descriptively ("Bass Drop Red", "Ambient Blue")
- Create variations (same shader, different colors)
- Test before performance

**Rehearse Navigation:**
- Practice keyboard shortcuts
- Know your grid layout
- Prepare scene order

**Backup Projects:**
- Save .omsp files regularly
- Export important .oms containers
- Keep Application Folder backed up

---

## TROUBLESHOOTING

### Shader Won't Load

**Check:**
- Application Folder set?
- Container file exists?
- WebGPU supported?
- Browser console for errors

### Audio Won't Play

**Check:**
- Audio file path still valid?
- BPM set correctly?
- Volume not at 0%?
- Track not muted?

### Pop-Out Window Blank

**Check:**
- Shaders loaded in grid?
- Clips launched?
- Volume not at 0%?
- Browser allowed pop-up?

### Performance Issues

**Solutions:**
- Lower pop-out resolution
- Simplify shaders
- Close other apps
- Use dedicated GPU (if available)

### Visual Indicators Not Showing

**Check:**
- Selected cell?
- Clip playing?
- Browser zoom at 100%?
- CSS not overridden by extension?

---

## KEYBOARD SHORTCUT CHEAT SHEET

**Print this for performances!**

```
TRANSPORT
Space     - Play/Pause
Esc       - Emergency Stop

TRACKS
1-4       - Select Track
5         - Select Master
Tab       - Next Track
Shift+Tab - Previous Track

NAVIGATION (Scene Column)
W         - Previous Scene
S         - Next Scene
Enter     - Launch Scene
D         - Enter Tracks

NAVIGATION (Track Columns)
W/S       - Vertical
A/D       - Horizontal
Enter     - Launch Cell
Esc       - Back to Scenes

AUDIO VOLUME
↑         - Volume Up
↓         - Volume Down

AUDIO MUTE/SOLO
,         - Toggle Audio Mute
.         - Toggle Audio Solo

SHADER OPACITY (V3.7.0)
Shift+↑   - Opacity Up (+2%)
Shift+↓   - Opacity Down (-2%)

SHADER MUTE/SOLO (V3.7.0)
Shift+,   - Toggle Shader Mute
Shift+.   - Toggle Shader Solo

CELL
Delete    - Clear Cell
```

---

**END OF USER GUIDE**

Version: V3.7.0  
Date: November 16, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com

Enjoy creating! 🎨✨
