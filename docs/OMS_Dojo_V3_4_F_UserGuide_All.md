# OMS SHADER BAKER - COMPLETE USER GUIDE V3.4.0

**Version:** V3.4.0 Production Release  
**Date:** November 15, 2025  
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
9. [Workflow Examples](#workflow-examples)
10. [Tips & Tricks](#tips--tricks)
11. [Troubleshooting](#troubleshooting)

---

## GETTING STARTED

### First Time Setup

**Step 1: Open the App**
- Double-click `OMS_ShaderBaker_V3_4_0.html`
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

### System Requirements

**Browser:** WebGPU-enabled (Chrome 113+, Edge 113+)  
**OS:** macOS 10.15+, Windows 10/11, Linux (Vulkan)  
**Screen:** 1512×982+ recommended (MacBook Pro 14" optimal)  
**Hardware:** Modern CPU, integrated or dedicated GPU

---

## INTERFACE OVERVIEW

### Layout

```
┌─────────────────────────────────────────────────┐
│ [C] [P] [L] [W] [S] [LIVE]  OMS SHADER BAKER   │ ← Tab Bar
├──────────────┬──────────────────────────────────┤
│              │                                  │
│  Left Panel  │        Preview / Grid            │
│  (400px)     │        (920px)                   │
│              │                                  │
│  - Tab       │  - Main Canvas                   │
│    Content   │  - Shader Preview                │
│              │  - LIVE Performance Grid         │
│  - Shader    │                                  │
│    Output    │                                  │
│   (bottom)   │                                  │
└──────────────┴──────────────────────────────────┘
```

### Tab Functions

- **C = Controls:** Visual controls for shader parameters
- **P = Presets:** Save/load project states (.omsp)
- **L = Library:** Browse and load containers (.oms)
- **W = WGSL:** Code editor for shader programming
- **S = Settings:** Application configuration
- **LIVE:** Performance grid for mixing multiple shaders

### Display Modes

**Creation Tabs (C, P, L, W, S):**
- Large preview canvas (right panel)
- Shader output preview (bottom left, LIVE tab only)
- Tab-specific controls (left panel)

**Performance Tab (LIVE):**
- Performance grid (right panel)
- Mixer controls (scenes, tracks, master)
- Waveform visualizations
- Shader output preview (bottom left)

---

## CONTROLS TAB (C)

### Purpose
Visual interface for shader parameters. No coding required.

### Controls Available

**Shader Parameters:**
- Time offset (shift animation start)
- Speed multiplier (animation speed)
- Color controls (hue, saturation, brightness)
- Pattern controls (scale, rotation, complexity)
- Effect controls (blur, glow, distortion)

**Modulator Frequency Ranges:**
- Mod A (Bass): 30-500 Hz default
- Mod B (Mids): 500-2000 Hz default
- Mod C (Treble): 2000-20000 Hz default

**Audio Controls:**
- Load audio file (MP3, WAV, FLAC)
- BPM detection/manual entry
- Pitch adjustment (-12 to +12 semitones)
- Start/end time markers
- Loop in/out points

**Render Settings:**
- Canvas resolution (width × height)
- Output format (WebM, MP4)
- Frame rate (30, 60 FPS)

### Using Rotary Knobs

**Mouse:**
- Click and drag vertically to adjust
- Hold Shift for fine control
- Double-click to reset to default

**Keyboard:**
- Tab to select control
- Arrow keys to adjust
- Shift+Arrow for fine control

### Saving Your Work

**Save Container (.oms):**
1. Adjust all parameters to taste
2. Click "Save Container" button
3. Enter name (e.g., "bass_reactive_v1")
4. File saved to application folder

**What Gets Saved:**
- Current shader code
- All control values
- Audio file reference (or embedded)
- Frequency range settings

---

## PRESETS TAB (P)

### Purpose
Manage project states for LIVE tab performance grid.

### Project States (.omsp)

**What They Store:**
- All 32 grid cell assignments (4 tracks × 8 scenes)
- Master tempo setting
- Per-track settings (volume, mute, solo)
- Audio file assignments per cell
- Loop points per cell

**Saving Project State:**
1. Arrange containers in LIVE tab grid
2. Click Presets tab (P)
3. Enter project name
4. Click "Save Project State"
5. File saved as `.omsp` in application folder

**Loading Project State:**
1. Click Presets tab (P)
2. Select project from list
3. Click "Load"
4. LIVE tab updates with all assignments

### Use Cases

- **Set List:** Different projects for different songs
- **Versions:** Multiple arrangements of same content
- **Backup:** Save before major changes
- **Templates:** Start new projects from proven layouts

---

## LIBRARY TAB (L)

### Purpose
Browse, organize, and load .oms containers.

### Container Management

**Viewing Containers:**
- Library displays all .oms files in application folder
- Thumbnails show shader preview (static image)
- Metadata shows name, date, audio info

**Loading Container:**
- Click container card to select
- Container loads into:
  - Controls tab (parameters)
  - Code tab (shader source)
  - Audio (if embedded/linked)

**Importing Containers:**
- Click "Import Container" button
- Select .oms file from anywhere
- Copies to application folder
- Now available in library

### Organization Tips

**Naming Conventions:**
```
bass_reactive_v1.oms
tunnel_zoom_purple.oms
kaleidoscope_8way.oms
fractal_bass_heavy.oms
```

**Folder Structure (in your application folder):**
```
MyShaders/
├── bass_shaders/
│   ├── bass_reactive_v1.oms
│   └── sub_bass_tunnel.oms
├── mid_shaders/
│   ├── vocal_visualizer.oms
│   └── synth_kaleidoscope.oms
└── projects/
    ├── set_2024_11_15.omsp
    └── backup_v2.omsp
```

### Shader Output Preview

**On LIVE Tab Only:**
- Preview window at bottom of Library tab
- Shows currently playing shader
- Monitor while browsing library
- Doesn't scroll with container list

---

## CODE TAB (W)

### Purpose
Direct WGSL shader code editing for advanced users.

### Code Editor Features

**Syntax Highlighting:**
- Keywords in purple
- Functions in blue
- Numbers in orange
- Comments in green

**Editor Controls:**
- Full-screen text area
- Monospace font
- Line numbers (coming in future version)
- Auto-indent (browser default)

### Shader Structure

**Required Components:**
```wgsl
// Uniforms
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32;
@group(0) @binding(3) var<uniform> uModB: f32;
@group(0) @binding(4) var<uniform> uModC: f32;

// Vertex shader (don't modify)
@vertex
fn vs_main(@builtin(vertex_index) vertexIndex: u32) -> @builtin(position) vec4<f32> {
    var pos = array<vec2<f32>, 4>(
        vec2(-1.0, -1.0), vec2(1.0, -1.0),
        vec2(-1.0, 1.0), vec2(1.0, 1.0)
    );
    return vec4<f32>(pos[vertexIndex], 0.0, 1.0);
}

// Fragment shader (your code here)
@fragment
fn fs_main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = fragCoord.xy / uResolution;
    
    // Your shader magic here
    var color = vec3<f32>(uv.x, uv.y, 0.5);
    
    return vec4<f32>(color, 1.0);
}
```

### Common Patterns

**See separate guide:**  
📖 OMS_ShaderBaker_V3_4_0_E_Claude_ShaderCreation_Guide.md

### Compilation Errors

**Viewing Errors:**
- Console shows WebGPU errors (F12 → Console)
- Check for missing semicolons
- Verify parentheses match
- Ensure uniform names correct

**Testing:**
1. Edit code
2. Shader recompiles automatically
3. Preview updates in real-time
4. Fix any errors
5. Repeat until perfect

---

## SETTINGS TAB (S)

### Purpose
Application-level configuration.

### Application Folder

**Setting Folder:**
1. Click "Set Application Folder"
2. Browser shows folder picker
3. Select folder (creates if needed)
4. Grant read/write permissions
5. Folder path displayed

**Folder Persistence:**
- Browser remembers folder
- Persists between sessions
- No need to re-select
- Click "Change Folder" to update

**Folder Contents:**
```
MyApplicationFolder/
├── *.oms (containers)
├── *.omsp (project states)
└── audio/ (optional subfolder)
```

### Future Settings

Coming in future versions:
- Default FFT frequency ranges
- Default canvas resolution
- Keyboard shortcuts configuration
- MIDI device mapping
- Performance optimization toggles

---

## LIVE TAB

### Purpose
Performance grid for live mixing of multiple shaders.

### Grid Layout

**4 Tracks × 8 Scenes = 32 Cells**

```
         Scene 1  Scene 2  Scene 3  Scene 4  Scene 5  Scene 6  Scene 7  Scene 8
Track 1  [     ] [     ] [     ] [     ] [     ] [     ] [     ] [     ]
Track 2  [     ] [     ] [     ] [     ] [     ] [     ] [     ] [     ]
Track 3  [     ] [     ] [     ] [     ] [     ] [     ] [     ] [     ]
Track 4  [     ] [     ] [     ] [     ] [     ] [     ] [     ] [     ]
```

### Mixer Section

**Per Track:**
- M button (mute)
- S button (solo)
- Volume knob (rotary, -∞ to 0 dB)
- VU meters (L/R channels)
- Track label (Track 1-4)

**Master:**
- Volume knob
- VU meters
- Master tempo (60-200 BPM)

**Scenes (Zoom):**
- ZOOM label
- Zoom knob (waveform zoom, 10-200 px/sec)

### Waveform Section

**Per Track:**
- Waveform visualization (bar-style, beat-aligned)
- Beat grid overlay (cyan bars every bar, magenta every 4 bars)
- Playhead (current position)
- Control buttons:
  - |< (previous bar)
  - >| (next bar)
  - << (nudge left)
  - >> (nudge right)

**Waveform Features:**
- Synced scrolling (all tracks together)
- Beat grid always aligned (regardless of BPM differences)
- Time-stretched to master tempo
- Professional DJ software style (Serato, Traktor)

### Loading Containers into Grid

**Method 1: Drag (Coming in V3.5.0)**
- Drag container from Library
- Drop onto grid cell
- Container loads into cell

**Method 2: Click (Current)**
1. Click grid cell to select
2. Go to Library tab
3. Click container
4. Returns to LIVE tab with container loaded

### Master Tempo Control

**Purpose:** Sync all tracks to one tempo

**How It Works:**
1. Set master tempo (e.g., 128 BPM)
2. Load tracks with different BPMs (124, 126, 128, 130)
3. All tracks time-stretch to 128 BPM
4. Beat grids align perfectly
5. Waveforms sync visually

**Playback Rate Formula:**
```
playbackRate = masterTempo / trackBPM
```

**Example:**
- Track: 126 BPM
- Master: 128 BPM
- Playback rate: 128/126 = 1.016 (slightly faster)

### Zoom Control

**Purpose:** Adjust waveform display resolution

**Range:** 10-200 pixels per second

**Low Zoom (10-50 px/sec):**
- See entire track at once
- Overview of arrangement
- Plan long mixes

**High Zoom (100-200 px/sec):**
- See individual beats/bars
- Precise cueing
- Beatmatching accuracy

**Adjustment:**
- Drag zoom knob (SCENES column)
- All track waveforms update together
- Maintains sync and alignment

### Performance Workflow

**Preparation:**
1. Load containers into grid cells
2. Set master tempo
3. Adjust zoom for comfortable view
4. Set track volumes
5. Cue to start positions

**During Performance:**
- Launch cells (coming in V3.5.0)
- Adjust volumes on the fly
- Use mute/solo for quick changes
- Navigate waveforms with control buttons
- Change master tempo for creative effects

---

## WORKFLOW EXAMPLES

### Example 1: Create and Save Simple Shader

1. **Open Controls Tab (C)**
2. **Adjust parameters** to taste
3. **Watch live preview** update
4. **Click "Save Container"**
5. **Enter name:** "my_first_shader"
6. **Container appears in Library (L)**

**Time:** 5 minutes  
**Result:** Reusable .oms container

---

### Example 2: Load Audio and Create Audio-Reactive Shader

1. **Controls Tab (C)**
2. **Click "Load Audio File"**
3. **Select MP3/WAV** from your music
4. **Set BPM** (auto-detect or manual)
5. **Adjust Mod A/B/C ranges** for desired frequencies
6. **Watch modulators react** in preview
7. **Adjust shader parameters** for reactivity
8. **Save as Container**

**Time:** 10 minutes  
**Result:** Audio-reactive .oms container

---

### Example 3: Arrange Multi-Track Performance

1. **Library Tab (L)**
2. **Load 4 different containers** (one per track)
3. **Switch to LIVE Tab**
4. **Verify all tracks loaded** in grid
5. **Set Master Tempo** (128 BPM)
6. **Adjust track volumes**
7. **Presets Tab (P):** Save project state
8. **Ready to perform!**

**Time:** 15 minutes  
**Result:** Performance-ready arrangement

---

### Example 4: Export Shader for Video

1. **Controls Tab (C)**
2. **Set render resolution** (1920×1080)
3. **Load audio file**
4. **Set loop points** (if looping)
5. **Click "Render Video"**
6. **Wait for render** (real-time or faster)
7. **Video saved** to downloads
8. **Convert with FFmpeg** (see FFmpeg guide)

**Time:** 5-30 minutes (depends on length)  
**Result:** Silent video file ready for Ableton Live

---

## TIPS & TRICKS

### Performance Optimization

**Shader Complexity:**
- Start simple, add complexity gradually
- Avoid heavy loops in fragment shader
- Use smoothstep() instead of if/else
- Precompute constants outside loops

**Audio Buffers:**
- Load once, reuse
- Don't reload same file multiple times
- Browser caches decoded audio

**Rendering:**
- Close unnecessary browser tabs
- Disable extensions that modify pages
- Use hardware acceleration
- Lower resolution for testing, full res for export

### Creative Techniques

**Layering:**
- Create multiple subtle shaders
- Mix in DAW (Ableton Live + EboSuite)
- Each shader targets different frequencies
- Combined effect is rich and complex

**Beat Matching:**
- Use BPM detection in Controls tab
- Fine-tune by ear if needed
- Master tempo syncs all tracks
- Perfect for DJ sets

**Color Theory:**
- Use complementary colors (blue/orange, purple/yellow)
- Analogous colors for smooth transitions (blue/cyan/green)
- High contrast for visibility on projectors
- Test on actual performance screen

### Keyboard Shortcuts

**Tab Navigation:**
- Numbers 1-6: Switch tabs (C, P, L, W, S, LIVE)
- Space: Play/pause (in LIVE tab)
- M: Toggle mute (selected track)
- S: Toggle solo (selected track)

**Future:** Full keyboard shortcut customization in Settings

### Organization Best Practices

**Naming Conventions:**
- Descriptive names: "bass_tunnel_purple_v2"
- Version numbers for iterations
- Tag by frequency: "bass_", "mid_", "treble_"
- Tag by style: "geometric_", "organic_", "abstract_"

**Folder Structure:**
- Separate folders for bass/mid/treble shaders
- Project folders for specific performances
- Backup folder for old versions
- Templates folder for starting points

---

## TROUBLESHOOTING

### WebGPU Not Supported

**Symptom:** Error message "WebGPU not supported"

**Solutions:**
- Update browser (Chrome 113+, Edge 113+)
- Check OS compatibility (macOS 10.15+, Windows 10/11)
- Enable GPU acceleration in browser settings
- Safari: Use Technology Preview build

### Audio Won't Load

**Symptom:** Audio file fails to load

**Causes:**
- Unsupported format (use MP3, WAV, FLAC)
- File corrupted
- File too large (>100MB may be slow)
- Browser security restrictions

**Solutions:**
- Convert to MP3 or WAV
- Test with smaller file first
- Check browser console for errors
- Try different browser

### Shader Compilation Error

**Symptom:** Red error in Code tab, no preview

**Common Issues:**
- Missing semicolon
- Mismatched parentheses/brackets
- Wrong uniform names
- Syntax error

**Solutions:**
- Check console (F12 → Console)
- Read error message carefully
- Use Claude AI to fix (paste shader + error)
- Start from working template

### Performance Issues (Slow Rendering)

**Symptom:** Choppy preview, low FPS

**Causes:**
- Complex shader
- Multiple heavy calculations
- Old GPU
- Browser resource constraints

**Solutions:**
- Simplify shader (reduce loops, effects)
- Close other apps/tabs
- Lower preview resolution temporarily
- Update GPU drivers
- Use dedicated GPU if available

### Application Folder Not Persisting

**Symptom:** Need to re-select folder every time

**Causes:**
- Browser privacy mode (incognito)
- Browser cleared storage
- Folder permissions changed
- Browser security settings

**Solutions:**
- Use normal (non-incognito) mode
- Don't clear browser storage
- Re-grant folder permissions
- Check browser security settings

### Waveforms Not Aligned

**Symptom:** Beat grids misaligned across tracks

**Causes:**
- Wrong BPM settings
- Audio start time offset
- Master tempo changed after loading

**Solutions:**
- Verify BPM for each track (Controls tab)
- Set audio start time to 0 (or correct offset)
- Reload tracks after master tempo change
- Use BPM detection (usually accurate)

---

## FREQUENTLY ASKED QUESTIONS

**Q: Can I use this for commercial performances?**  
A: Yes! GPL-3.0 allows commercial use. You own your content.

**Q: Do I need to credit OMS Shader Baker?**  
A: Appreciated but not required for performances. Credit if sharing code.

**Q: Can I modify the source code?**  
A: Yes! GPL-3.0 allows modification. Share improvements!

**Q: Will my shaders work in future versions?**  
A: Yes! Backward compatibility is priority. .oms format stable.

**Q: Can I use VST plugins for audio?**  
A: No. Web Audio API only. Export from DAW, import as audio file.

**Q: What about MIDI control?**  
A: Coming in V3.5.0! Web MIDI API integration planned.

**Q: Can I use this on iPad?**  
A: Not yet. WebGPU not widely supported on mobile. Desktop only.

**Q: How do I report bugs?**  
A: YouTube comments on ONEMANSHYO videos, GitHub issues if repo exists.

**Q: Is support available?**  
A: "Ghost protocol" - tools are free, no ongoing support. Community-driven.

---

## NEXT STEPS

**You're Ready!**

- Create your first shader
- Save some containers
- Arrange a performance
- Export a video
- Perform live!

**Resources:**
- FFmpeg Guide: OMS_ShaderBaker_V3_4_0_G_UserGuide_FFmpeg_Video_Guide.md
- Shader Creation: OMS_ShaderBaker_V3_4_0_E_Claude_ShaderCreation_Guide.md
- Developer Docs: OMS_ShaderBaker_V3_4_0_C_Developer_Documentation.md

**Community:**
- YouTube: ONEMANSHYO channel
- Website: https://onemanshyo.com

---

**END OF COMPLETE USER GUIDE**

Version: V3.4.0  
Date: November 15, 2025  
License: GPL-3.0
