# ONEMANSHYO Dojo - Complete User Guide

## Quick Start

1. Open `OMS_Dojo_V2_5.html` in Chrome 113+ or Edge 113+
2. Verify "WebGPU Ready" appears in status bar
3. Create shader with Claude or use default shader
4. Paste shader into Code tab, click [Compile]
5. Adjust controls and render video

---

## Interface Overview

### Main Sections

**Code Tab**
- Shader code editor (paste WGSL fragment shaders)
- [Compile] button to test shader
- Syntax highlighting (if enabled)

**Controls Tab**
- Row 1: Shader Tempo | Loop Length | Render Duration
- Rows 2-4: Modulator controls (A, B, C)
- Modulator settings: Subdivision, Waveform, Gain

**Output Tab**
- Orientation: Portrait / Landscape / Square
- Resolution: 720p / 1080p / 1440p / 2160p
- Frame Rate: 24 / 30 / 60 FPS
- Filename input
- [R] Render button

**Canvas**
- Live preview of shader output
- Updates in real-time as you adjust controls
- Visible during rendering (watch your video being created)

**Audio Controls Strip** (appears when audio loaded)
- Audio Start / Audio End fields (MM:SS.mmm format)
- Audio Tempo field (manual entry)
- Loop In / Loop Out brackets (bar.beat format)
- Transport: [Play/Pause] [Loop] [Remove Audio]
- Waveform visualization with beat markers

**Status Bar** (bottom)
- System messages and render progress
- WebGPU status indicator

---

## Core Concepts

### Two Rendering Modes

**No Audio Mode**
- Pure shader animation
- Duration controlled by Render Duration field
- Timing based on Shader Tempo
- Use for: backgrounds, loops, non-music content

**Audio Mode**
- Audio-reactive visualization
- Duration controlled by Loop In/Out brackets
- Timing based on Audio Tempo
- Use for: music videos, visualizers, reactive content

### Timing Systems (Independent)

**Shader Tempo**
- Location: Controls tab, Row 1
- Purpose: Controls visual animation speed
- Default: 120
- Controls: Modulator subdivision cycles
- Independent of audio

**Audio Tempo**
- Location: Audio controls strip
- Purpose: Beat positioning and loop calculations
- User-entered: Manual decimal entry
- Controls: Loop In/Out bracket positioning
- Only relevant when audio loaded

### Modulators (Audio-Reactive)

**Three Frequency Bands:**
- **Mod A:** Bass (20-250 Hz)
- **Mod B:** Mid (250-4000 Hz)
- **Mod C:** Treble (4000-20000 Hz)

**Controls per Modulator:**
- **Subdivision:** Timing cycle (1/32 to 4 bars, plus special modes)
- **Waveform:** Shape (Sine, Saw, Square, Triangle, etc.)
- **Gain:** Intensity (0.0 to 1.0 range)

Without audio loaded, modulators cycle based on Shader Tempo.
With audio loaded, modulators respond to audio frequency content.

---

## Detailed Workflows

### Workflow 1: Create Shader Animation (No Audio)

1. **Create Shader**
   - Ask Claude to generate WGSL shader
   - Copy shader code

2. **Paste and Compile**
   - Go to Code tab
   - Paste shader code
   - Click [Compile]
   - Canvas updates with shader preview

3. **Adjust Visual Timing**
   - Set Shader Tempo (60-180 typical range)
   - Adjust Loop Length if using subdivisions
   - Watch canvas for desired speed

4. **Configure Modulators** (optional)
   - Row 2: Mod A (bass frequencies)
   - Row 3: Mod B (mid frequencies)
   - Row 4: Mod C (treble frequencies)
   - Adjust Subdivision, Waveform, Gain per modulator

5. **Set Render Duration**
   - Focus Render Duration field
   - Type duration in MM:SS.mmm format
   - Examples: "00:15.000" (15 sec), "01:30.000" (90 sec)
   - Or use arrow keys: Up/Down ±0.1s, Left/Right ±1.0s

6. **Configure Output**
   - Go to Output tab
   - Select Orientation (Portrait for social media)
   - Select Resolution (1080p recommended)
   - Select Frame Rate (30 FPS typical)
   - Enter custom Filename

7. **Render Video**
   - Click [R] button
   - Watch canvas during render (shows actual output)
   - Status bar shows render progress
   - Video downloads automatically when complete

### Workflow 2: Create Music Visualizer (Audio Mode)

1. **Create Shader**
   - Follow same shader creation steps as above

2. **Load Audio**
   - Click [Upload Audio] button
   - Select audio file (MP3, WAV, OGG)
   - Audio waveform appears
   - Loop button defaults to active (cyan)

3. **Enter Audio Tempo**
   - Manually enter BPM in Audio Tempo field
   - Use decimal precision (e.g., 126.53)
   - Arrow keys: Up/Down ±0.01, Left/Right ±1.0

4. **Locate First Downbeat**
   - Use Audio Start field to find first strong beat
   - Type time or use arrow keys
   - This is your musical reference point

5. **Set Loop Brackets**
   - **Loop In:** Start of desired section
   - **Loop Out:** End of desired section
   - Use arrow keys for precise positioning:
     - Loop In: Up/Down ±1/4 note, Left/Right ±1 note
     - Loop Out: Up/Down ±1/32 note, Left/Right ±1/8 note

6. **Audition Loop**
   - Verify Loop button is cyan (active)
   - Click [Play/Pause]
   - Audio plays from Loop In to Loop Out and repeats
   - Fine-tune brackets if needed
   - Audio Tempo corrections: Use arrow keys on Audio Tempo field

7. **Configure Output and Render**
   - Same as No Audio Mode workflow steps 6-7
   - Video includes audio from Loop In to Loop Out

---

## Control Reference

### Shader Tempo
- **Location:** Controls tab, Row 1, Position 1
- **Format:** Numeric (integer)
- **Range:** 30-300 typical
- **Default:** 120
- **Purpose:** Visual animation timing
- **Arrow Keys:** Up/Down ±1, Left/Right ±10

### Loop Length
- **Location:** Controls tab, Row 1, Position 2
- **Options:** 4, 8, 16, 32, 64 bars
- **Default:** 16 bars
- **Purpose:** Modulator cycle length
- **Note:** Only affects modulator timing, not render duration

### Render Duration
- **Location:** Controls tab, Row 1, Position 3
- **Format:** MM:SS.mmm (e.g., "00:30.000")
- **Default:** 00:30.000 (30 seconds)
- **Purpose:** Video length when no audio loaded
- **Arrow Keys:** Up/Down ±0.1s, Left/Right ±1.0s
- **Parsing:** Left-to-right (typing "5" = 5 minutes)

### Modulator Subdivision
- **Options:** 
  - Notes: 1/32, 1/16, 1/8, 1/4, 1/2, 1, 2, 4 bars
  - Special: Chaos, Cycler, Mad Max, Psycho
- **Default:** 1/4 note
- **Purpose:** Cycle timing for modulator waveform

### Modulator Waveform
- **Options:** Sine, Saw Up, Saw Down, Square, Triangle, Noise, Pulse, Random
- **Default:** Sine
- **Purpose:** Shape of modulator oscillation

### Modulator Gain
- **Range:** 0.00 to 1.00
- **Default:** 0.50
- **Purpose:** Modulator intensity
- **Arrow Keys:** Up/Down ±0.01, Left/Right ±0.10

### Audio Start / Audio End
- **Format:** MM:SS.mmm
- **Purpose:** Define musical boundaries of audio file
- **Usage:** Audio Start = first downbeat, Audio End = last downbeat
- **Arrow Keys:** Up/Down ±0.1s, Left/Right ±1.0s

### Audio Tempo
- **Format:** Decimal (e.g., 126.53)
- **Range:** 30-300 typical
- **Purpose:** Beat positioning and loop calculations
- **Arrow Keys:** Up/Down ±0.01, Left/Right ±1.0
- **Note:** Must be manually entered (no auto-detection)

### Loop In / Loop Out
- **Format:** bar.beat (e.g., "17.1" = bar 17, beat 1)
- **Default:** 17.1 to 33.1 (64 beats)
- **Purpose:** Define video render range
- **Arrow Keys:**
  - Loop In: Up/Down ±1/4 note, Left/Right ±1 note
  - Loop Out: Up/Down ±1/32 note, Left/Right ±1/8 note

---

## Tips & Best Practices

### Shader Development
- Start with simple shapes before complex patterns
- Test at 720p before rendering 4K
- Use Shader Tempo to speed up/slow down animation
- Ask Claude for shader variations and refinements

### Audio Tempo Entry
- Use decimal precision for accuracy (126.53 vs 127)
- Fine-tune with arrow keys (±0.01 increments)
- Verify tempo by enabling Loop and checking beat alignment
- Incorrect tempo = Loop Out position will drift

### Loop Bracket Positioning
- Use Audio Start as reference for first downbeat
- Start with coarse positioning (Left/Right arrow keys)
- Fine-tune with Up/Down arrow keys
- Enable Loop button to audition before rendering
- Loop button displays cyan when active

### Render Settings
- **Social Media:** 1080p Portrait, 30 FPS
- **High Quality:** 2160p, 60 FPS (larger file size)
- **Performance:** 720p, 30 FPS (faster render, smaller file)
- **Smooth Motion:** 60 FPS (worth the extra time)

### Filename Conventions
Use descriptive names:
- Good: "sunset_vibes" "geometric_blue" "bass_drop"
- Avoid: "test" "video1" "untitled"

---

## Keyboard Shortcuts

**Global:**
- Tab: Navigate between fields
- Enter: Apply changes / Trigger focused button
- Escape: Cancel / Defocus field

**Arrow Keys (context-dependent):**
- All number fields support arrow key adjustment
- Up/Down: Fine increment
- Left/Right: Coarse increment
- See individual control reference for specific increments

---

## Troubleshooting

**Problem: "WebGPU not supported"**
- Solution: Update browser to Chrome/Edge 113+ or Safari TP
- Solution: Enable chrome://flags/#enable-unsafe-webgpu

**Problem: Shader won't compile**
- Solution: Check browser console for WGSL error
- Solution: Verify uniform bindings in shader code
- Solution: Ask Claude to debug shader

**Problem: Audio won't load**
- Solution: Check file format (MP3, WAV, OGG)
- Solution: Try different audio file
- Solution: Verify file isn't corrupted

**Problem: Loop doesn't play correctly**
- Solution: Verify Loop button is cyan (active)
- Solution: Check Audio Tempo value
- Solution: Adjust Loop Out position with fine arrow keys

**Problem: Render video too short/long**
- With Audio: Check Loop In/Out bracket positions
- With Audio: Verify Audio Tempo is correct
- No Audio: Check Render Duration field value

**Problem: Video quality is poor**
- Solution: Increase resolution (1440p or 2160p)
- Solution: Increase frame rate (60 FPS)
- Solution: Simplify shader if GPU struggling

---

## File Formats

**Input:**
- Audio: MP3, WAV, OGG, M4A (4/4 time signature)
- Shader: WGSL fragment shader code (plain text)

**Output:**
- Video: WebM container, VP8 codec
- Includes audio if loaded
- Filename format: OMS_[name]_[resolution]p_[fps]fps_[orientation].webm

---

## System Requirements

**Browser:**
- Chrome 113+ (recommended)
- Edge 113+
- Safari Technology Preview

**Hardware:**
- GPU with WebGPU support
- 4GB+ RAM (8GB+ recommended)
- 2GB+ free disk space

---

**Last Updated:** November 2025  
**Version:** V2_5  
**Creator:** Wes Smith / OneManShYo
