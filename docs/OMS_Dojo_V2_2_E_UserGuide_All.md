# ONEMANSHYO Dojo V2_2 - Complete User Guide

**Version:** V2_2  
**Target Audience:** All users (beginners to advanced)  
**Purpose:** Complete reference for all features and workflows

---

## Table of Contents

- [Getting Started](#getting-started)
- [Interface Overview](#interface-overview)
- [Tab System](#tab-system)
- [Audio System](#audio-system)
- [Shader Parameters](#shader-parameters)
- [Video Export](#video-export)
- [Keyboard Shortcuts](#keyboard-shortcuts)
- [Workflows](#workflows)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## What's New in V2_2

### Automatic BPM Detection
- Audio tempo automatically detected on load
- Range: 60-180 BPM (0.5 step precision)
- Override by typing if needed
- Onset detection algorithm

### Nudge Field
- Fine-tune loop timing by milliseconds
- Range: ±5000ms
- Arrow keys: ±1ms (Up/Down), ±4ms (Left/Right)
- Applied to all audio position jumps

### UI Clarity
- "Audio BPM" → "BPM"
- "IN" → "Loop In"
- "OUT" → "Loop Out"
- Clearer tooltips throughout

### Loop Button
- Toggle between loop IN→OUT vs continuous play
- Visual state indicator
- Default: Loop enabled

### Simplified Arrow Keys
- Always move both Loop In/Out together
- Maintains loop length (slides window)
- Grid button disabled (reserved for future)

---

## Getting Started

### System Requirements

**Browser:**
- Chrome 113+ (recommended)
- Edge 113+
- Safari Technology Preview

**Hardware:**
- Modern GPU with 4GB+ VRAM
- 8GB+ RAM
- Any modern CPU

### First Steps

1. **Open the application:**
   - Double-click `OMS_Dojo_V2_2.html`
   - Opens in your default browser

2. **Verify WebGPU:**
   - Status bar should show "WebGPU Ready"
   - If not, update browser or enable WebGPU flag

3. **Default state:**
   - Canvas shows default "Flower Power" shader
   - Controls Tab is active
   - Ready to use!

---

## Interface Overview

### Layout

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  LEFT PANEL    â”‚  RIGHT PANEL                       â”‚
â”‚                â”‚                                    â”‚
â”‚  Header        â”‚  Canvas (1920x1080)               â”‚
â”‚  [V2_2]      â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚                â”‚  â”‚  Shader Preview (HD)         â”‚ â”‚
â”‚  Tab Bar       â”‚  â”‚                              â”‚ â”‚
â”‚  [Controls]    â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚  [Code]        â”‚                                    â”‚
â”‚                â”‚  Waveform (1920x180)              â”‚
â”‚  Tab Content   â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â” â”‚
â”‚  (5 rows)      â”‚  â”‚  Audio Waveform              â”‚ â”‚
â”‚                â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚
â”‚                â”‚                                    â”‚
â”‚                â”‚  Audio Controls                   â”‚
â”‚                â”‚  [â–¶] [â¹] â”‚ IN [64] OUT [128]      â”‚
â”‚                â”‚                                    â”‚
â”‚                â”‚  Status Bar                        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Left Panel (Controls Tab)

**5 Rows of Shader Controls:**
1. Orientation (Landscape/Portrait/Square)
2. Shader Tempo + Loop Length
3. Modulator A (Subdivision, Waveform, Gain)
4. Modulator B (Subdivision, Waveform, Gain)
5. Modulator C (Subdivision, Waveform, Gain)

**Bottom Row:**
- [C] Compile
- Resolution dropdown
- FPS dropdown
- Filename input
- [R] Render

### Right Panel

**Canvas:**
- 1920x1080 HD preview (default)
- Updates in real-time
- Shows compiled shader

**Waveform:**
- 1920x180 visualization
- Shows when audio is loaded
- Interactive (click/drag to scrub)

**Audio Controls Strip:**
- â–¶ Play/Pause (OMS Pink when playing)
- â¹ Stop
- IN beat field
- OUT beat field

**Status Bar:**
- Shows current state
- Displays bar.beat format when audio playing
- Shows compilation/render messages

---

## Tab System

### Overview

V2_2 introduces a tab system for better workflow organization.

**Controls Tab:**
- All shader parameters
- Video export settings
- Active by default

**Code Tab:**
- WGSL shader editor
- Full-height editing space
- Press [C] to compile and preview

### Switching Tabs

**Click tab buttons:**
- Click "Controls" for parameter controls
- Click "Code" for shader editing

**Workflow:**
1. Edit shader in Code tab
2. Press [C] to compile
3. Switch to Controls tab to see preview
4. Adjust parameters
5. Repeat

---

## Audio System

### Uploading Audio

**Method:**
1. Click anywhere in canvas/waveform area
2. Select audio file (MP3, WAV, OGG, M4A, FLAC)
3. Wait for waveform to generate
4. Audio ready!

**Supported Formats:**
- MP3 (most common)
- WAV (high quality)
- OGG, M4A, FLAC (browser-dependent)

### Waveform (V2_2)

**New in V2_2:**
- 6x bigger (1920px vs 320px)
- Located on canvas (not left panel)
- Better visibility
- Easier interaction

**Features:**
- Real amplitude visualization (not fake bars)
- Cyan shaded region = IN to OUT range
- Magenta playhead = current position
- Black overlay = played portion

**Interaction:**
- Click and drag to scrub
- Arrow keys to jump beats
- Spacebar to play/pause

### Transport Controls (V2_2)

**New in V2_2:**
- Visual playback controls
- OMS Pink (#ff006e) styling
- Below waveform

**Play Button (â–¶/â¸):**
- Click to play/pause
- Icon changes based on state
- Pink glow when playing

**Stop Button (â¹):**
- Click to stop and reset
- Returns to beginning
- Clears play state

**Keyboard Shortcuts:**
- Spacebar: Play/Pause (same as button)
- All shortcuts preserved!

### IN/OUT Beats

**What are beats?**
- IN beat: Where playback/render starts
- OUT beat: Where playback/render ends
- Defines your loop range

**Default Values:**
- IN: 64 (starts past typical intro)
- OUT: 128 (creates 64-beat/16-bar loop)

**Editing:**
1. Type new values in fields
2. Audio jumps to IN beat automatically
3. OUT must be greater than IN

**Navigation:**
- Left Arrow: Jump to previous beat
- Right Arrow: Jump to next beat
- Stays within IN/OUT range

### Audio Reactivity

**How It Works:**
1. Audio analyzed in real-time (FFT)
2. Split into 3 frequency bands:
   - Bass (Low): 0-10% of spectrum
   - Mid: 10-50% of spectrum
   - Treble (Hi): 50-100% of spectrum
3. Each band multiplies a modulator:
   - Bass Ã— Mod A Ã— Gain
   - Mid Ã— Mod B Ã— Gain
   - Treble Ã— Mod C Ã— Gain

**Gain Controls:**
- Value range: 0.0 to 5.0
- Default: 0.1 (subtle)
- Higher = more reactive

**Frequency Bands:**
- **Bass (Low):** Kicks, bass guitars, sub bass
- **Mid:** Vocals, guitars, most instruments
- **Treble (Hi):** Hi-hats, cymbals, high notes

---

## Shader Parameters

### Orientation

**Options:**
- Landscape (16:9) - YouTube videos
- Portrait (9:16) - Instagram Reels, TikTok, YouTube Shorts
- Square (1:1) - Instagram Feed

**Default:** Portrait

### Shader Tempo

**Purpose:** Visual timing (not audio BPM)
**Range:** 1-999 BPM
**Default:** 120 BPM
**Use:** Controls modulator speed and loop timing

### Shader Loop Length

**Purpose:** Pattern repeat duration
**Options:**
- No Loop (continuous)
- 1, 2, 4, 8, 16, 32 Bars
- Whole, Half, 1/4, 1/8, 1/16 notes
- Chaos, Cycler, Mad Max, Psycho (meta-modes)

**Default:** 4 Bars

### Modulators (A, B, C)

**Each modulator has 3 controls:**

**1. Subdivision (Note Duration):**
- Whole note (4 beats)
- Half note (2 beats)
- Quarter note (1 beat)
- Eighth note (0.5 beats)
- Sixteenth note (0.25 beats)
- Chaos, Cycler, Mad Max, Psycho (meta-modes)

**2. Waveform:**
- Sine, Cosine
- Sawtooth, Triangle, Square
- Noise, Random
- Fibonacci, Prime
- Chaos, Cycler, Mad Max, Psycho (meta-modes)

**3. Gain (Audio Multiplication):**
- Range: 0.0 to 5.0
- Default: 0.1
- Mod A = Bass (Low)
- Mod B = Mid
- Mod C = Treble (Hi)

**How They Work:**
- Generate 0.0-1.0 values
- Multiply by audio levels
- Available in shader as uniforms.modA/B/C

---

## Video Export

### Settings

**Resolution:**
- 720p (1280Ã—720 or equivalent)
- 1080p (1920Ã—1080 or equivalent)
- 1440p (2560Ã—1440 or equivalent)
- 2160p (3840Ã—2160 or equivalent)

**Frame Rate:**
- 24 FPS (cinematic)
- 30 FPS (standard)
- 60 FPS (smooth motion)

**Filename:**
- Enter descriptive name
- Auto-generates full filename:
  `OMS_{name}_{res}p_{fps}fps_{dur}s_{beats}beats_{L|P|S}.webm`

**Examples:**
- `OMS_myshader_1080p_30fps_32s_128beats_L.webm` (Landscape)
- `OMS_myshader_1080p_30fps_32s_128beats_P.webm` (Portrait)
- `OMS_myshader_1080p_30fps_32s_128beats_S.webm` (Square)

### Rendering Process

**1. Configure Settings:**
- Set Resolution, FPS, Filename
- Verify IN/OUT beats are correct

**2. Click [R] Button:**
- Rendering begins
- Progress shown in status bar
- **Don't switch tabs!**

**3. Wait for Completion:**
- Audio plays from IN to OUT
- Canvas captures every frame
- Automatic download when done

**4. Find Your Video:**
- Downloads to browser's download folder
- WebM format with embedded audio

### Quality vs Speed

**Fast (Minutes):**
- 720p
- 30 FPS
- Shorter duration

**Quality (Longer):**
- 1080p or higher
- 60 FPS
- Longer duration
- Complex shaders

**Recommendation:**
- Test: 720p @ 30fps
- Production: 1080p @ 30fps
- High-end: 1440p or 2160p @ 60fps

---

## Keyboard Shortcuts

**Audio Playback:**
- **Spacebar:** Play/Pause audio
- **Left Arrow:** Jump to previous beat
- **Right Arrow:** Jump to next beat

**Compilation:**
- Press [C] button (no shortcut yet)

**Navigation:**
- Click tabs to switch between Controls and Code

---

## Workflows

### Quick Start Workflow

1. **Open application** (status shows "WebGPU Ready")
2. **Upload audio** (click canvas area)
3. **Adjust parameters** (tempo, modulators, gains)
4. **Preview** (spacebar to play)
5. **Render** (set resolution/FPS, click [R])

### Shader Creation Workflow

1. **Switch to Code tab**
2. **Edit WGSL shader**
3. **Press [C] to compile**
4. **Switch to Controls tab**
5. **See preview in 1920x1080 HD**
6. **Adjust parameters**
7. **Repeat**

### Social Media Workflow

**Instagram Feed (Square):**
1. Orientation: Square
2. Resolution: 1080p
3. FPS: 30
4. Duration: Under 60 seconds
5. Render and post-process with FFmpeg

**Instagram Reels / TikTok (Portrait):**
1. Orientation: Portrait
2. Resolution: 1080p
3. FPS: 30
4. Duration: 15-60 seconds
5. Render and post-process

**YouTube Videos (Landscape):**
1. Orientation: Landscape
2. Resolution: 1080p or higher
3. FPS: 30 or 60
4. Any duration
5. Can upload WebM directly

### Live Streaming Workflow

1. **Create visuals** (multiple clips)
2. **Render at 1080p30**
3. **Process with FFmpeg** (see FFmpeg guide)
4. **Add to OBS** as media source
5. **Stream to YouTube**

---

## Troubleshooting

### WebGPU Not Available

**Symptom:** Error about WebGPU support

**Solutions:**
1. Update browser (Chrome 113+, Edge 113+)
2. Enable: `chrome://flags/#enable-unsafe-webgpu`
3. Update GPU drivers
4. Try different browser

### Shader Won't Compile

**Symptom:** Error after pressing [C]

**Common Issues:**
- Missing semicolons
- Type mismatch (use 1.0 not 1 for floats)
- Missing return statement
- Undefined variables

**Solution:**
- Read error message carefully
- Check line number mentioned
- Compare with default shader

### Audio Won't Play

**Symptom:** Waveform visible but no sound

**Solutions:**
1. Check system volume
2. Check browser tab not muted
3. Click page to enable audio context
4. Try different audio file

### Video Export Issues

**Symptom:** Render fails or corrupt video

**Solutions:**
1. Don't switch tabs during render
2. Check disk space
3. Try shorter duration
4. Close other programs
5. Refresh and retry

### Performance Issues

**Symptom:** Laggy UI, dropped frames

**Solutions:**
1. Lower resolution (try 720p)
2. Reduce FPS (30 instead of 60)
3. Simplify shader code
4. Close other tabs
5. Update GPU drivers

---

## FAQ

**Q: What's new in V2_2?**  
A: Tab system, 6x bigger canvas (1920x1080), waveform on canvas, audio controls strip with transport buttons, cleaner left panel.

**Q: Does the bigger canvas affect performance?**  
A: Minimal impact on modern hardware. Main bottleneck is shader complexity, not canvas size.

**Q: Can I use this for live streaming?**  
A: Yes! Render clips, process with FFmpeg, use in OBS for YouTube live streams.

**Q: What video format does it export?**  
A: WebM (VP8 codec, 8Mbps). Process with FFmpeg for other formats.

**Q: Why is the default IN beat 64?**  
A: Skips typical 16-bar intro, starts in the "meat" of the song.

**Q: Can I map MIDI controls?**  
A: Not currently. Tool is designed as standalone, not real-time MIDI integration.

**Q: How do I add my own waveforms?**  
A: Modify the JavaScript code (see Developer Documentation).

**Q: Can I use this without audio?**  
A: Yes! Works great for abstract visuals with just modulators and shader tempo.

**Q: What's the difference between Shader Tempo and Audio BPM?**  
A: Shader Tempo controls visual timing (modulators). Audio BPM is from your track. They can match or be different.

**Q: Can I render multiple clips in sequence?**  
A: No, one at a time. But you can concatenate videos with FFmpeg afterward.

---

## Tips & Tricks

**Canvas Preview (V2_2):**
- Preview is 1920x1080 (matches 1080p export exactly)
- What you see is what you get!
- Great for detailed shader work

**Tab Workflow:**
- Use Code tab for editing
- Use Controls tab for parameter tweaking
- Switch frequently for fast iteration

**Audio Reactivity:**
- Start with gains at 0.1
- Increase gradually to find sweet spot
- Bass (Mod A) great for scale/position
- Mid (Mod B) great for color/rotation
- Treble (Mod C) great for accents

**Modulator Meta-Modes:**
- Chaos: Random waveform every bar
- Cycler: Rotates through all waveforms
- Mad Max: Aggressive rotation
- Psycho: Rapid random switching

**Keyboard Efficiency:**
- Spacebar for quick audio preview
- Arrow keys for precise beat navigation
- Press [C] often while editing shaders

**Quality Settings:**
- Test at 720p30 (fast)
- Produce at 1080p30 (standard)
- Archive at 1440p60 or 2160p60 (high-end)

---

**Last Updated:** November 2025  
**Version:** V2_2  
**Status:** Production Release

---

*For FFmpeg workflows, see UserGuide_FFmpeg_Video_Guide.md*  
*For technical details, see C_Developer_Documentation.md*  
*For release notes, see D_Release_Summary.md*
