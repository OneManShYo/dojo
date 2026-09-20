# OMS DOJO V2_8-N - USER GUIDE

**Version:** V2_8-N  
**Last Updated:** November 2025  
**Purpose:** Complete user documentation

---

## TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [User Interface Overview](#user-interface-overview)
3. [Controls Tab](#controls-tab)
4. [Code Tab](#code-tab)
5. [Presets Tab](#presets-tab)
6. [Library Tab](#library-tab-new)
7. [BroadcastChannel Features](#broadcastchannel-features-new)
8. [Audio Pitch Control](#audio-pitch-control-new)
9. [Rendering Videos](#rendering-videos)
10. [Keyboard Shortcuts](#keyboard-shortcuts)
11. [Workflows](#workflows)
12. [Troubleshooting](#troubleshooting)

---

## GETTING STARTED

### System Requirements

**Browser:** Chrome 88+, Safari 15+, or Edge 88+ (WebGPU required)  
**OS:** macOS, Windows, or Linux  
**Storage:** ~300MB for application + audio library

### Opening the Application

1. Download `OMS_Dojo_V2_8_N.html`
2. Double-click to open in default browser, or
3. Right-click → Open With → Chrome/Safari

### First Launch

Application opens to Controls tab with default shader loaded. Click **Compile** to see shader in action.

---

## USER INTERFACE OVERVIEW

### Layout

```
┌─────────────┬───────────────────────────────────┐
│  CONTROLS   │                                   │
│  CODE       │        CANVAS                     │
│  PRESETS    │       (Shader Preview)            │
│  LIBRARY    │                                   │
│   (tabs)    │                                   │
├─────────────┼───────────────────────────────────┤
│             │  [Compile]  [Stop]  [Render]      │
│  Left       │                                   │
│  Panel      │  Audio Controls                   │
│             │  Waveform                         │
│             │                                   │
├─────────────┴───────────────────────────────────┤
│  Status Bar                                     │
└─────────────────────────────────────────────────┘
```

### Tabs (Left Panel)

- **Controls** - Timing, reactivity, output settings
- **Code** - Shader editor
- **Presets** - Library shaders + user presets
- **Library** - Container storage (NEW in V2_8)

### Main Buttons

- **Compile** - Compile shader and start preview
- **Stop** - Stop shader preview
- **Render** - Export video

---

## CONTROLS TAB

### Shader Timing Section

**Shader Tempo (BPM)**
- Controls visual animation speed
- Independent from audio tempo
- Range: 60-200 BPM
- Arrow keys: ±0.1 BPM (fine), ±1 BPM (coarse)

**Loop Length (Beats)**
- Duration of visual loop
- Options: 1/4, 1/2, 1, 2, 4, 8, 16, 32, 64 beats
- Dropdown or arrow keys to change

**Render Duration (mm:ss.ms)**
- Length of exported video
- Format: minutes:seconds.milliseconds
- Example: 00:30.000 = 30 seconds

### Shader Reactivity Section

**Only visible when audio loaded.**

**Modulator A/B/C:**
Each modulator has 3 parameters:
- **Subdivision** - Beat division (1/2, 1/4, 1/8, 1/16, 1/32)
- **Waveform** - sine, triangle, saw, square
- **Gain** - Intensity (0.0-1.0)

**Frequency Mapping:**
- **Freq1 / Freq2** - Frequency range in Hz
- Target specific audio frequencies
- Examples:
  - 30-50 Hz: Kick drums
  - 50-75 Hz: Sub-bass
  - 100-500 Hz: Lead bass
  - 500-2000 Hz: Mids
  - 2000-8000 Hz: Highs

**Arrow Key Navigation:**
- Up/Down: ±5 Hz (fine tune)
- Left/Right: ±50 Hz (coarse tune)

### Output Settings Section

**Filename**
- Default: "shader"
- Used for exported video filename

**Resolution**
- Options: 720p, 1080p, 1440p, 2160p (4K)
- Dropdown selection

**Frame Rate (FPS)**
- Options: 24, 30, 60
- Higher FPS = smoother but larger files

**Format**
- WebM VP8 (only option currently)
- Convert to ProRes with FFmpeg for Ableton/EboSuite

### Control States

**Purpose:** Save complete render configurations for recall.

**Features:**
- 5 Library Presets (Quick Test, HD, 4K, Instagram, TikTok)
- Unlimited User Presets
- Info box shows current state

**Save User State:**
1. Configure all settings
2. Type name in input field
3. Press Enter or click ✓
4. State saved to dropdown

**Load State:**
1. Select from dropdown
2. All 21 values apply instantly

**Delete State:**
1. Select state
2. Click Delete button
3. Confirm deletion

**What's Saved (21 values):**
- Shader tempo, loop length, render duration
- All modulator settings (A/B/C)
- Frequency mappings (6 values)
- Audio tempo, pitch
- Output settings (filename, resolution, FPS, format)

---

## CODE TAB

### Shader Editor

**Features:**
- Syntax highlighting (basic)
- Tab indentation support
- Line numbers
- Monospace font

**Writing Shaders:**
- Use WGSL (WebGPU Shading Language)
- Fragment shader only
- See Claude_ShaderCreation_Guide.md for patterns

**Shader Name:**
- First line comment: `// My Shader Name`
- Used in BroadcastChannel announcements
- Shows in DojoLive dropdowns

**Compile Button:**
- Click to compile and preview
- Errors shown in status bar
- Success → animation starts

---

## PRESETS TAB

### Library Presets

**5 Curated Shaders:**
1. Plasma - Flowing colors
2. Geometric - Rotating shapes
3. Reactive Bars - Audio bars
4. Tunnel - Perspective tunnel
5. Wobble - Fluid motion

**Loading:**
- Click thumbnail to load shader
- Automatically compiles
- Ready to customize

### User Presets

**Import Shader:**
1. Click "Import Shader" button
2. Select .txt file with shader code
3. Shader imports and compiles
4. Thumbnail generated automatically

**Export Shader:**
1. Click "Export Shader"
2. Downloads current shader as .txt
3. Share with others

**Removing:**
- Click X on thumbnail to remove
- Confirmation required

---

## LIBRARY TAB (NEW)

### Overview

Store complete shader+audio+state packages (containers) for instant recall.

### Setup Audio Library (One-Time)

**First Time:**
1. Click "Setup Audio Library"
2. File picker opens
3. Navigate to folder with audio files
4. Click "Select"
5. Permission granted (persists forever)

**Status:**
- Info box shows folder name: "✓ Songs"
- Can change folder anytime

**Folder Organization:**
```
~/Music/OMS/
├── kick_loops/
│   ├── kick_124.mp3
│   ├── kick_126.mp3
├── bass/
│   ├── sub_128.mp3
├── full_tracks/
    ├── track1.mp3
```

### Adding Containers

**Workflow:**
1. Create shader in Code tab
2. Load audio from library folder
3. Configure reactivity in Controls tab
4. Library tab → "Add Current"
5. Enter name (or keep default)
6. Press Enter or click ✓
7. Container saved

**What's Saved:**
- Shader code
- Control States (21 values)
- Audio filename (reference, not embedded)
- Audio BPM
- Created timestamp

### Loading Containers

**Grid View:**
- 🎵 icon = has audio
- 🎨 icon = no audio
- Name + BPM displayed
- L/E/X buttons

**Load (L Button):**
1. Click "L" on container
2. Shader loads into editor
3. Audio loads from library folder
4. All states apply
5. Shader compiles automatically

**If Audio Missing:**
- Shader + states still load
- Error: "Audio not found: filename.mp3"
- Add file to library folder or load different audio

### Export/Import

**Export (E Button):**
1. Click "E" on container
2. Downloads as .oms file
3. Audio embedded (for portability)
4. Share file with others

**Import:**
1. Click "Import" button
2. Select .oms file
3. Container added to library
4. New ID generated (prevents collisions)

**Remove (X Button):**
1. Click "X" on container
2. Confirmation dialog
3. Removed from library
4. .oms files unaffected

### Storage Details

**Metadata:** localStorage (~5KB per container)  
**Audio:** Your filesystem (unlimited)  
**Permissions:** IndexedDB (persistent)  
**Capacity:** 100+ containers, 10,000+ audio files

---

## BROADCASTCHANNEL FEATURES (NEW)

### Purpose

Communicate with DojoLive for multi-instance performance workflows.

### How It Works

**Automatic:**
- No setup required
- Each Dojo tab gets unique ID
- Announces presence on page load
- Broadcasts shader name
- Sends canvas snapshots

**Instance ID:**
- Format: `sb-abc123` (9 chars)
- Shows in console log
- Used by DojoLive for track assignment

**Shader Name:**
- Extracted from first comment: `// Bass Tunnel v3`
- Fallback: "Dojo Instance"
- Appears in Live dropdowns

**Canvas Snapshots:**
- Sent on compile (100ms delay)
- Auto-sent on page load (500ms delay)
- JPEG 80% quality (~50-100KB)
- Displayed in DojoLive grid

**Heartbeat:**
- Announces every 5 seconds
- Keeps connection alive
- Live removes stale instances (30 sec timeout)

### Usage with DojoLive

**Setup:**
1. Open DojoLive (V1.0.0-L or later)
2. Open Dojo tab(s)
3. DojoLive auto-discovers instances

**Workflow:**
1. Assign instances to tracks (dropdown)
2. Compile shaders → snapshots appear
3. Mix visuals in Live grid
4. Future: Audio/FFT from Live → Dojo

**Multi-Instance:**
- Open 3-4 Dojo tabs
- Different shaders per tab
- All communicate independently
- Build complete visual set

---

## AUDIO PITCH CONTROL (NEW)

### Purpose

Match audio tempo to session BPM using DJ-style pitch control.

### Pitch Field

**Location:** After Audio Tempo field  
**Range:** -16% to +16%  
**Display:** Percentage with 2 decimals

**Arrow Key Navigation:**
- **Up/Down:** ±0.1% (fine tune)
- **Left/Right:** ±1.0% (coarse tune)
- **Type:** Direct input supported

### How It Works

**Playback Rate:**
- Formula: `playbackRate = 1.0 + (pitch / 100)`
- Example: +3.23% → 1.0323x speed
- Changes both speed and pitch (like vinyl/CDJ)

**BPM Display:**
- Shows adjusted BPM during playback
- Formula: `adjusted = native × playbackRate`
- Example: 124 BPM × 1.0323 = 128 BPM

**Duration Display:**
- Accounts for pitch adjustment
- Faster pitch = shorter duration
- Slower pitch = longer duration
- Example: 198s at +6.9% = 185s

### Workflow

**Matching Tracks to Session Tempo:**

```
Session: 128 BPM

Track 1: Native 124 BPM
→ Set pitch: +3.23%
→ Plays at: 128 BPM ✓

Track 2: Native 130 BPM
→ Set pitch: -1.54%
→ Plays at: 128 BPM ✓
```

**Saving with Control States:**
1. Load audio
2. Set pitch
3. Save Control State
4. Later: Load state → pitch auto-applies

### Future Integration

**DojoLive (V1.1.0+):**
- Live sends pitch commands via BroadcastChannel
- Dojo receives and applies pitch
- Master tempo in Live drives all instances
- Automatic sync across tracks

---

## RENDERING VIDEOS

### Render Process

**Steps:**
1. Configure all settings (Controls tab)
2. Click **Render** button
3. Progress bar shows %
4. Download starts when complete
5. Video saved to Downloads folder

**Filename Format:**
```
[filename]_[timestamp].webm

Example:
shader_20251110_143022.webm
```

### Render Settings

**Duration:**
- Set in Render Duration field
- Format: mm:ss.ms
- Minimum: 00:00.100 (100ms)
- Maximum: 10:00.000 (10 minutes)

**Resolution:**
- 720p: 1280×720 (fast, small files)
- 1080p: 1920×1080 (standard HD)
- 1440p: 2560×1440 (2K)
- 2160p: 3840×2160 (4K, large files)

**Frame Rate:**
- 24 fps: Cinematic
- 30 fps: Standard video
- 60 fps: Smooth motion (2x file size)

### Silent Videos

**Why Silent?**
- Shaders are visual loops
- Audio-reactive modulators baked into animation
- Sync to audio in Ableton/EboSuite during performance
- Keeps files small

**With Audio (Optional):**
- Load audio before rendering
- Reactivity baked into video
- Audio NOT included in export
- This is by design

### ProRes Conversion

**For Ableton/EboSuite:**

WebM → ProRes 422 for optimal playback.

```bash
ffmpeg -i shader.webm -c:v prores_ks -profile:v 3 shader.mov
```

See `UserGuide_FFmpeg_Video_Guide.md` for complete instructions.

---

## KEYBOARD SHORTCUTS

### Global

- **Tab** - Indent in shader editor
- **Shift+Tab** - Outdent in shader editor

### Numeric Fields

**Audio Tempo, Shader Tempo, Pitch:**
- **Up Arrow** - Increase by 0.1
- **Down Arrow** - Decrease by 0.1
- **Right Arrow** - Increase by 1.0
- **Left Arrow** - Decrease by 1.0

**Frequency Fields (Freq1A, Freq2A, etc.):**
- **Up Arrow** - Increase by 5 Hz
- **Down Arrow** - Decrease by 5 Hz
- **Right Arrow** - Increase by 50 Hz
- **Left Arrow** - Decrease by 50 Hz

**Loop Length:**
- **Up Arrow** - Next subdivision (4→8→16→32)
- **Down Arrow** - Previous subdivision (32→16→8→4)

### Container Name Input

- **Enter** - Confirm and save
- **Escape** - Cancel

---

## WORKFLOWS

### Basic Shader Creation

1. **Code Tab:** Write or paste shader
2. Click **Compile**
3. Preview animation
4. **Controls Tab:** Adjust timing
5. Click **Render**
6. Download video

### Audio-Reactive Shader

1. **Code Tab:** Load preset or write shader
2. Click **Compile**
3. Load audio file (Load Audio button)
4. **Controls Tab:** Set frequency mapping
   - Freq1A/Freq2A: 30-50 Hz (kick)
   - Freq1B/Freq2B: 50-75 Hz (sub)
   - Freq1C/Freq2C: 100-500 Hz (bass)
5. Adjust modulator gains
6. Click **Render**
7. Download video
8. Convert to ProRes with FFmpeg
9. Import to Ableton/EboSuite

### Container Library Workflow

**Setup (Once):**
1. **Library Tab:** Setup Audio Library
2. Pick folder with audio files
3. Done

**Daily Use:**
1. Create shader
2. Load audio from library
3. Configure reactivity
4. **Library Tab:** Add Current
5. Name container
6. Container saved

**Recall:**
1. **Library Tab**
2. Click "L" on container
3. Everything loads instantly

**Backup:**
1. Export containers as .oms files
2. Store in cloud/external drive
3. Import if needed

### Multi-Instance Performance (with Live)

**Setup:**
1. Open DojoLive
2. Open 3-4 Dojo tabs
3. Load different containers in each tab
4. DojoLive discovers all instances

**Performance:**
1. Assign instances to tracks
2. DojoLive shows snapshots
3. Build your set
4. Mix visuals live

**Future (V1.2.0):**
- Load audio in DojoLive
- Live sends FFT to Dojo
- Real-time audio-reactive visuals
- Scene launching

---

## TROUBLESHOOTING

### Shader Won't Compile

**Check:**
- Syntax errors in shader code
- Valid WGSL format
- Status bar shows error message

**Solutions:**
- Load library preset (known working)
- Check Claude_ShaderCreation_Guide.md
- Start with simple shader, add complexity

### Audio Won't Load

**Check:**
- File format: MP3, WAV, OGG
- File size: <100MB recommended
- Not DRM-protected

**Solutions:**
- Try different audio file
- Check browser console for errors
- Use supported format

### Audio Library Not Working

**Firefox:**
- File System Access API not supported
- Use Chrome or Safari

**Permission Expired:**
- Re-run Setup Audio Library
- Grant permission again

**File Not Found:**
- Verify file exists in library folder
- Check filename matches exactly
- Case-sensitive on Mac/Linux

### Render Freezes/Crashes

**Causes:**
- Complex shader + high resolution
- Insufficient GPU memory

**Solutions:**
- Lower resolution (1080p instead of 4K)
- Shorter duration
- Simpler shader
- Close other tabs/apps

### Canvas Blank After Compile

**Check:**
- Shader compiled successfully?
- Console errors?
- WebGPU supported?

**Solutions:**
- Refresh page
- Try different browser
- Check WebGPU support: chrome://gpu

### Storage Quota Exceeded

**Cause:**
- localStorage full (~5-10MB)
- Too many containers without audio library

**Solutions:**
- Remove old containers
- Export containers, then delete
- Setup Audio Library (externalizes audio)

---

## TIPS & BEST PRACTICES

### Shader Creation
- Start simple, add complexity gradually
- Test frequently (click Compile often)
- Use library presets as templates
- Save user presets for reusable patterns

### Audio Reactivity
- Target specific frequencies for clean results
- Kick: 30-50 Hz
- Bass: 50-150 Hz
- Mids: 500-2000 Hz
- Don't overlap frequency ranges (muddy)

### Container Library
- Organize audio files in subfolders
- Use descriptive filenames
- Export containers regularly (backup)
- Keep library folder synced (Dropbox/iCloud)

### Performance
- Close unused Dojo tabs
- Limit to 4-8 instances max
- Lower canvas resolution if laggy
- Render at native resolution (don't upscale)

### Workflow Efficiency
- Use Control States for render configs
- Use Containers for complete setups
- Use Presets for shader variations
- Master all three systems

---

## ADVANCED FEATURES

### Manual Beat Grid

**Controls Tab:** Audio Module section

**Loop In/Out Brackets:**
- Format: bar.beat (1.1, 2.3, 4.1)
- Navigate audio by musical position
- Set loop region for rendering

**Beat Grid:**
- Visual beat markers on waveform
- Synced to Audio Tempo
- Manual positioning (not auto-detected)

### Modulator Waveforms

**sine:** Smooth, organic motion  
**triangle:** Linear ramps  
**saw:** Sharp attacks, fast decay  
**square:** Binary on/off

**Use Cases:**
- sine: Breathing effects
- triangle: Predictable motion
- saw: Rhythmic hits
- square: Strobe effects

### Zero Value Frequency Mapping

**Disabling Reactivity:**
- Set Freq1/Freq2 to 0
- Modulator still works (timing-based)
- No audio analysis applied
- Selective reactivity across modulators

---

## FUTURE FEATURES (V2.9.0+)

### Incoming
- Receive FFT/tempo from DojoLive
- Update modulators from external audio
- Scene launch commands
- Remote preset loading

### Planned
- Container thumbnail previews
- Sort/filter library
- Batch operations
- Subfolder support in audio library
- IndexedDB upgrade (more containers)

### Long-Term (V3.0.0)
- Remove all audio features
- Pure visual player
- Receives all data from OMS Session
- Continuous play mode

---

**END OF USER GUIDE**

**See Also:**
- Developer_Documentation.md - Technical reference
- Claude_ShaderCreation_Guide.md - Shader patterns
- FFmpeg_Video_Guide.md - ProRes conversion
- Release_Guide.md - What's new in V2_8
