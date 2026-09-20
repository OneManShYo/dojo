# ONEMANSHYO Dojo - Complete User Guide

## Table of Contents

1. [Getting Started](#getting-started)
2. [Interface Overview](#interface-overview)
3. [Controls Tab](#controls-tab)
4. [Presets Tab](#presets-tab)
5. [Code Tab](#code-tab)
6. [Workflows](#workflows)
7. [Keyboard Shortcuts](#keyboard-shortcuts)
8. [Tips & Tricks](#tips--tricks)

---

## Getting Started

### System Requirements

- **Browser:** Chrome 113+ or Edge 113+ (WebGPU required)
- **RAM:** 4GB minimum, 8GB+ recommended
- **GPU:** Any modern GPU (Intel UHD 620 or better)
- **Storage:** ~200KB for app, varies for videos

### First Launch

1. Open `OMS_Dojo_V2_6.html` in Chrome or Edge
2. Allow microphone access if prompted (for audio features)
3. You'll see the Controls tab with preview canvas
4. Click **Presets** tab to explore example shaders

### Quick Test

1. Click **Presets** tab
2. Click "Tron Tunnel" thumbnail
3. Preview shows animated shader
4. Click **Render** button
5. Video downloads automatically

**Congratulations!** You've created your first shader video.

---

## Interface Overview

### Main Layout

```
┌─────────────────────────────────────────┐
│  [V2_6] OMS DOJO             │
│  [Controls] [Code] [Presets]           │  ← Tab Bar
├──────────────┬──────────────────────────┤
│              │                          │
│  Left Panel  │    Preview Canvas        │
│  (400px)     │    (WebGPU Rendering)    │
│              │                          │
│  • Settings  │    Visual Output         │
│  • Controls  │    60fps Preview         │
│  • Buttons   │                          │
│              │                          │
├──────────────┴──────────────────────────┤
│  Status Bar: Messages & Progress        │
└─────────────────────────────────────────┘
```

### Three Tabs

**Controls:** Settings, modulators, rendering
**Code:** WGSL shader editor
**Presets:** Library + User preset management

### Status Bar

Bottom of screen shows:
- Success messages (green)
- Error messages (red)
- Info messages (blue)
- Render progress during video export

---

## Controls Tab

### Row 1: Core Timing

**Shader Tempo (BPM)**
- Controls visual animation speed
- Default: 120 BPM
- Range: 60-200 typical
- Independent from audio tempo

**Loop Length (Bars)**
- Duration in musical bars (4/4 time)
- Options: 1, 2, 4, 8, 16, 32 bars
- Used when calculating from tempo

**Render Duration (MM:SS.mmm)**
- Direct time entry for shader-only mode
- Format: Minutes:Seconds.Milliseconds
- Example: 00:30.000 = 30 seconds
- Left-to-right parsing: "5" = 5 minutes

**Arrow Key Controls:**
- Up/Down: ±0.1 seconds
- Left/Right: ±1.0 seconds
- Fine control without mouse

### Rows 2-4: Modulators

Each modulator (A, B, C) has three controls:

**Subdivision Dropdown**
- Note values: 1/4, 1/8, 1/16, 1/32
- Controls oscillation rate
- Example: 1/4 = once per quarter note

**Waveform Dropdown**
- Sine: Smooth curve
- Triangle: Linear up/down
- Square: On/off binary
- Sawtooth: Linear rise, instant drop

**Gain Slider (0.0-1.0)**
- Modulator intensity
- 0.0 = no effect
- 1.0 = full effect
- Live-tweakable during preview

**Modulator Assignments:**
- Mod A typically: Bass (20-250Hz)
- Mod B typically: Mids (250-4000Hz)
- Mod C typically: Treble (4000-20000Hz)

### Row 5: Output Settings

**Resolution**
- 720p: 1280×720 (faster renders)
- 1080p: 1920×1080 (standard)
- 1440p: 2560×1440 (high quality)
- 2160p: 3840×2160 (4K, slow renders)

**FPS (Frames Per Second)**
- 24: Cinematic
- 30: Standard (recommended)
- 60: Smooth motion (larger files)

**Orientation**
- Portrait: 9:16 aspect (1080×1920)
- Landscape: 16:9 aspect (1920×1080)
- Square: 1:1 aspect (1080×1080)

**Filename**
- Optional text entry
- Auto-parsed from shader code if empty
- Falls back to "filename" if no name found

### Audio Controls Strip

**Only visible when audio loaded**

**Upload Audio Button**
- Drag audio file onto waveform
- Or click "Upload Audio" button
- Formats: .mp3, .wav, .m4a

**Audio Tempo (BPM)**
- Manual entry required
- Must match track's actual BPM
- Used for beat positioning

**Audio Start/End (MM:SS.mmm)**
- Set first downbeat position
- Used for beat grid alignment
- Format: Minutes:Seconds.Milliseconds

**Loop In/Out Brackets**
- Bar.Beat format (1-indexed)
- Example: 1.1 = Bar 1, Beat 1
- Example: 5.1 = Bar 5, Beat 1
- Arrow keys: ±1 beat increments
- Fine control: ±1/32 or ±1/8 note

**Loop Button**
- Click to enable looping
- Turns cyan when active
- Audio loops from In to Out bracket
- Essential for testing sync

**Play/Pause Button**
- Start/stop audio playback
- Preview shader with audio

**Remove Audio Button**
- Clears loaded audio
- Returns to shader-only mode

### Action Buttons

**Compile Shader**
- Compiles WGSL code from Code tab
- Shows errors if compilation fails
- Updates preview canvas
- Required before rendering

**Render**
- Starts video export
- Progress shown in status bar
- Format: "Rendering 00:15/00:30 | Frame 450/900"
- Downloads WebM file when complete

---

## Presets Tab

### Overview

Two sections:
1. **LIBRARY:** 5 curated shaders (hardcoded)
2. **USER PRESETS:** Your performance library

### Preview As Dropdown

**Controls all thumbnail orientations**
- Portrait: Tall cards (9:16)
- Landscape: Wide cards (16:9)
- Square: Square cards (1:1)
- Changes ALL cards simultaneously

### Library Section

**5 Preset Shaders:**

1. **Tron Tunnel** (Portrait)
   - Pink-blue radial tunnel effect
   - Bass: tunnel speed + grid intensity
   - Mid: color shift (cyan ↔ magenta)
   - Treble: sparkles + center glow

2. **Tron Grid** (Portrait)
   - Retrowave sun with grid floor
   - Bass: grid speed
   - Mid: grid density
   - Treble: glow intensity

3. **MetaBalls** (Square)
   - Lava lamp organic blobs
   - Bass: blob size
   - Mid: movement speed
   - Treble: threshold

4. **Flower Power** (Square)
   - Rotating circle pattern
   - Mod A: base radius
   - Mod B: rotation speed
   - Mod C: brightness

5. **Retro Fish Farm** (Square)
   - Animated grid with dots
   - Bass: dot movement
   - Mid: grid distortion
   - Treble: intensity

**Using Library Presets:**
1. Click any thumbnail
2. Shader loads into Code tab
3. Compiles automatically
4. Preview shows result
5. Stays on Presets tab (browse more if desired)
6. Switch to Controls tab to adjust settings
7. Click Render when ready

### User Presets Section

**Your Performance Library**

**Import Button:**
- Click to open file picker
- Or drag .txt/.wgsl files onto grid
- Select multiple files (20-30 at once!)
- Visual feedback: cyan border during drag
- Thumbnails generate automatically
- Stored in browser LocalStorage

**Delete Button:**
- Click/Tab to select card first
- Then click Delete button
- Immediate deletion (no prompt)
- One-click workflow

**Keyboard Navigation:**
- Tab: Move between cards
- Enter: Load selected card
- Fast browsing workflow

**Drag & Drop:**
1. Select shader .txt files on desktop
2. Drag onto User Presets grid
3. Cyan border appears during drag
4. Drop files
5. Thumbnails render automatically
6. Status: "Imported X shader(s) via drag & drop!"

**Performance Workflow:**
1. Import 20-30 shaders via drag & drop
2. Visual thumbnails show each shader
3. Tab through cards to browse
4. Enter to load shader
5. Render video
6. Tab to next shader
7. Repeat (fast iteration!)

---

## Code Tab

### Shader Editor

**Large textarea for WGSL code**
- Syntax highlighting: None (plain text)
- Write or paste shader code
- Line breaks preserved

### Required Structure

```wgsl
// ONEMANSHYO Dojo V2_6 - [ShaderName]

@group(0) @binding(0) var<uniform> uniforms: Uniforms;

struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    padding1: f32,
    resolutionX: f32,
    resolutionY: f32,
}

@fragment
fn fragmentMain(@builtin(position) FragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    
    // YOUR CODE HERE
    
    return vec4<f32>(finalColor, 1.0);
}
```

### Buttons

**Compile**
- Validates WGSL syntax
- Creates shader module
- Updates preview
- Shows errors if any

**Export .txt**
- Saves shader code as text file
- Filename: OMS_[name]_Shader_V2.6.0.txt
- Name auto-parsed from comment or filename field
- Falls back to "filename" if empty

### Common Uniforms

**uniforms.time**
- Elapsed seconds since start
- Use for animation timing

**uniforms.tempo**
- Current Shader Tempo (BPM)
- Use for beat-synced effects

**uniforms.modA/B/C**
- Modulator values (0.0-1.0)
- Real-time from Controls tab sliders

**uniforms.resolutionX/Y**
- Canvas dimensions
- Reconstruct: `let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);`

---

## Workflows

### Workflow 1: Shader-Only Mode

**Creating visual loops without audio**

1. Click **Code** tab
2. Write or paste WGSL shader
3. Click **Compile** to preview
4. Switch to **Controls** tab
5. Set **Shader Tempo** (e.g., 120 BPM)
6. Set **Render Duration** (e.g., 00:30.000)
7. Adjust **Modulators** A/B/C
8. Click **Render**
9. Watch progress: "Rendering 00:15/00:30 | Frame 450/900"
10. Video downloads as silent WebM

**Use Case:** Building performance video library for EboSuite/Resolume

### Workflow 2: Audio-Reactive Mode

**Syncing visuals to music tracks**

1. **Upload Audio:** Drag .mp3 onto waveform
2. **Set Tempo:** Enter BPM in Audio Tempo field
3. **Set Start:** Find first downbeat (MM:SS.mmm)
4. **Set Loop In:** Arrow keys to position bracket (bar.beat)
5. **Set Loop Out:** Arrow keys to end bracket
6. **Enable Loop:** Click Loop button (turns cyan)
7. **Test:** Click Play, verify loop timing
8. **Adjust Modulators:** Match music energy
9. **Render:** Click Render button
10. **Output:** Video includes audio from Loop In to Out

**Use Case:** Creating synced videos for social media

### Workflow 3: Performance Library Building

**Preparing 20-30 shaders for live DJ set**

1. Click **Presets** tab
2. Collect 20-30 shader .txt files on desktop
3. Drag all files onto **User Presets** grid
4. Visual thumbnails generate (may take 30-60 seconds)
5. **Browse:** Tab through cards to see visuals
6. **Load:** Press Enter on card
7. **Adjust:** Tweak settings in Controls tab if needed
8. **Render:** Click Render button
9. **Next:** Tab to next card, repeat
10. **Result:** Complete video library in 30-60 minutes

**Use Case:** DJ performance prep with visual element

### Workflow 4: Iterative Shader Development

**Refining visuals through multiple passes**

1. Start with basic shader (Claude or manual)
2. **Compile** → Preview
3. Adjust modulator gains
4. Note improvements needed
5. Edit shader code
6. **Compile** → Preview again
7. Repeat until satisfied
8. **Export .txt** to save shader
9. Add to User Presets for future use
10. **Render** final video

**Use Case:** Creating perfect custom shader

---

## Keyboard Shortcuts

### Render Duration Field

- **Up Arrow:** +0.1 seconds
- **Down Arrow:** -0.1 seconds
- **Left Arrow:** +1.0 seconds
- **Right Arrow:** -1.0 seconds

### Loop In/Out Brackets (when audio loaded)

- **Up Arrow:** +1 beat
- **Down Arrow:** -1 beat
- **Shift+Up:** +1/8 note (Loop Out only)
- **Shift+Down:** -1/8 note (Loop Out only)
- **Ctrl+Up:** +1/32 note (Loop Out only)
- **Ctrl+Down:** -1/32 note (Loop Out only)

### Preset Navigation

- **Tab:** Move to next preset card
- **Shift+Tab:** Move to previous card
- **Enter:** Load focused preset
- **Space:** Load focused preset (alternative)

### General

- **Ctrl+S:** (Browser default - save page)
- **F11:** Fullscreen (browser default)
- **Esc:** Exit fullscreen

---

## Tips & Tricks

### Render Progress Tracking

**During render, status bar shows:**
```
Rendering 00:15/00:30 | Frame 450/900
```

- Left time: Elapsed (00:15 = 15 seconds in)
- Right time: Total (00:30 = 30 second render)
- Frame count: Current/Total frames

**Use this to:**
- Time modulator tweaks without audio
- Know when big drop hits
- Gauge how much longer render takes

### Filename Auto-Parsing

**Leave filename field empty to auto-parse:**
1. Shader has comment: `// ONEMANSHYO Dojo - TronTunnel`
2. Filename field empty
3. Export/Render uses "TronTunnel" automatically

**Fallback hierarchy:**
1. User-entered filename (if provided)
2. Parsed from shader comment (if found)
3. "filename" (if neither)

### Preview Orientation

**Preview As dropdown affects:**
- ALL preset thumbnails (Library + User)
- Does NOT affect render orientation
- Use to see how shader looks in different formats

**Render orientation:**
- Set in Controls tab (separate dropdown)
- Affects final video output
- Independent from Preview As

### Modulator Gain Tuning

**Finding optimal gain values:**
1. Set gain to 0.5 (middle)
2. Play audio or watch preview
3. Increase gain if effect too subtle
4. Decrease gain if effect too intense
5. Final range typically 0.3-0.8

### Audio Loop Testing

**Perfect loop setup:**
1. Load audio
2. Set tempo accurately (use BPM tool if unsure)
3. Set Audio Start to exact first downbeat
4. Set Loop In to 1.1 (first beat)
5. Set Loop Out to 5.1 (after 4 bars)
6. Enable Loop button
7. Click Play
8. Should loop seamlessly at Out → In

**If loop drifts:**
- Audio Tempo incorrect (adjust by ±0.1 BPM)
- Audio Start not at downbeat (adjust by ±0.01s)
- Loop brackets not at beat (use arrow keys)

### Render Duration Shortcuts

**Quick entry formats:**
- "5" = 5 minutes (00:05:00)
- "30" = 30 seconds (00:00:30)
- "1:30" = 1 minute 30 seconds
- "00:45.500" = 45.5 seconds (precise)

**Left-to-right parsing means:**
- Assume minutes first if ambiguous
- Use colons for explicit MM:SS format

### Video File Management

**Downloaded files:**
- Format: OMS_[name]_[res]p_[fps]fps_[orient].webm
- Example: OMS_TronTunnel_1080p_30fps_P.webm

**Organizing:**
```
MySet/
├── shaders/        (shader .txt files)
├── videos/         (rendered WebM files)
└── converted/      (H.264 .mp4 files for compatibility)
```

### LocalStorage Persistence

**User Presets saved in browser:**
- Survives browser refresh
- Does NOT sync across devices
- Lost if browser data cleared
- Export important shaders as .txt files!

**Backup strategy:**
1. Import shaders into User Presets
2. Use for session
3. Keep original .txt files safe
4. Re-import if LocalStorage cleared

### Performance Optimization

**Faster renders:**
- Lower resolution (1080p → 720p)
- Lower FPS (60 → 30)
- Shorter duration (60s → 30s)
- Close other applications
- Disable browser extensions

**Smoother preview:**
- Lower resolution preview (automatic)
- Close other tabs
- Reduce modulator complexity
- Simplify shader code

---

## Troubleshooting

### Thumbnails Won't Render

**Problem:** Preset thumbnails show black or blank

**Solutions:**
1. Check browser console (F12) for WebGPU errors
2. Update GPU drivers
3. Try different browser (Chrome vs Edge)
4. Reduce number of User Presets (<20)
5. Clear browser cache
6. Restart browser

### Audio Won't Load

**Problem:** Dragging audio file shows error

**Solutions:**
1. Check file format (.mp3/.wav/.m4a only)
2. Try different audio file
3. Convert with VLC to .mp3
4. Ensure file size < 100MB
5. Check file isn't corrupted

### Shader Won't Compile

**Problem:** Compile button shows error

**Solutions:**
1. Check uniform struct matches template
2. Verify no typos in field names
3. Ensure resolution built from resolutionX/Y
4. Check all functions have return types
5. Look for missing semicolons/braces

### Video Won't Render

**Problem:** Render button does nothing

**Solutions:**
1. Compile shader first
2. Check Render Duration has valid time
3. Try lower resolution
4. Close other GPU applications
5. Restart browser

### Loop Won't Play Back

**Problem:** Audio doesn't loop at Out bracket

**Solutions:**
1. Verify Audio Tempo matches track BPM
2. Check Loop In/Out brackets valid (e.g., 1.1 to 5.1)
3. Ensure Loop button enabled (cyan)
4. Set Audio Start to first downbeat
5. Try wider loop range (4 bars instead of 1)

---

## Appendix: File Formats

### Video Output

**WebM (VP8 Codec)**
- Native browser export format
- Good quality, reasonable file size
- May not play on all devices
- Use FFmpeg to convert to H.264/MP4

### Shader Files

**Plain Text (.txt or .wgsl)**
- WGSL fragment shader code
- Comment header for identification
- Import into User Presets

### Audio Input

**Supported Formats:**
- MP3 (most common)
- WAV (uncompressed)
- M4A (Apple format)
- OGG (less common)

---

**User Guide Version:** V2_6  
**Last Updated:** November 2025  
**Target Audience:** All users (beginners to advanced)

**Questions?** Check Claude ShaderCreation Guide for shader development help!
