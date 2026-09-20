# OMS DOJO - COMPLETE USER GUIDE V2_7

**Version:** V2_7  
**Release Date:** November 2025  
**Purpose:** Comprehensive guide for all features and workflows

---

## TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [Interface Overview](#interface-overview)
3. [Controls Tab](#controls-tab)
4. [Code Tab](#code-tab)
5. [Presets Tab](#presets-tab)
6. [Audio System](#audio-system)
7. [Control States System](#control-states-system)
8. [Frequency Mapping](#frequency-mapping)
9. [Video Rendering](#video-rendering)
10. [Workflows](#workflows)
11. [Keyboard Shortcuts](#keyboard-shortcuts)
12. [Troubleshooting](#troubleshooting)

---

## GETTING STARTED

### System Requirements

**Browser:**
- Safari Technology Preview 15+ (recommended)
- Chrome Canary with WebGPU enabled
- Edge Canary with WebGPU enabled

**Hardware:**
- Modern GPU with WebGPU support
- 4GB+ RAM recommended
- Fast storage for video export

**Operating System:**
- macOS, Windows, or Linux
- WebGPU support required

### Installation

**No installation required!**

1. Download OMS_Dojo_V2_7.html
2. Open in supported browser
3. Start creating shaders

**First Launch:**
- Canvas displays default shader
- Library presets available immediately
- All features ready to use

### Quick Start (5 Minutes)

**Create Your First Video:**

1. Click **Presets** tab
2. Press Tab to browse library shaders
3. Press Enter to load a shader
4. Click **Controls** tab
5. Select "Quick Test" from Control States dropdown
6. Click **Render** button
7. Wait ~45 seconds
8. Video downloads automatically

**Congratulations!** You've created your first shader video.

---

## INTERFACE OVERVIEW

### Layout

```
┌─────────────────────────────────────────────┐
│  ONEMANSHYO Dojo V2_7                    │
│  [Controls] [Code] [Presets]        [Logo]  │
├──────────────┬──────────────────────────────┤
│              │                              │
│  Left Panel  │        Canvas Area           │
│  (400px)     │      (Live Preview)          │
│              │                              │
├──────────────┴──────────────────────────────┤
│  Status Bar: Messages and state info        │
└─────────────────────────────────────────────┘
```

### Tabs

**Controls Tab:**
- Shader timing controls
- Audio-reactive modulators
- Frequency mapping fields
- Output settings
- Control States system
- Render controls

**Code Tab:**
- WGSL shader code editor
- Compile button
- Save to presets
- Syntax highlighting

**Presets Tab:**
- Library shaders (5 curated)
- User presets (your shaders)
- Visual thumbnails
- Drag-and-drop import
- Quick navigation

### Canvas Area

**Live Preview:**
- Real-time shader rendering at 60fps
- Always 1080p resolution
- Aspect ratio matches selected orientation
- Audio waveform (when audio loaded)
- Loop brackets (when audio loaded)

### Status Bar

**Displays:**
- Current actions
- Error messages
- Render progress
- State changes

---

## CONTROLS TAB

### Section 1: Shader Timing

**Purpose:** Controls visual animation timing (independent of audio).

#### Shader Tempo (BPM)
- **Range:** 20-300 BPM
- **Default:** 120 BPM
- **Controls:** Visual animation speed in beats per minute
- **Note:** Separate from Audio Tempo (if audio loaded)

#### Loop Length (bars)
- **Options:** 1, 2, 4, 8, 16, 32 bars
- **Default:** 16 bars
- **Controls:** Visual loop cycle duration
- **Note:** In 4/4 time (4 beats per bar)

#### Render Duration (MM:SS.mmm)
- **Format:** Minutes:Seconds.milliseconds
- **Example:** 00:30.000 = 30 seconds
- **Controls:** Video length when NO audio loaded
- **Note:** Overridden by Loop In/Out when audio loaded (magenta overlay)

**Example:**
- Shader Tempo: 128 BPM
- Loop Length: 16 bars
- Render Duration: 00:45.000
- Result: 45-second video at 128 BPM visual timing

### Section 2: Shader Reactivity

**Purpose:** Control audio-reactive modulators with user-defined frequency ranges.

#### Modulator A (Mod A)

**Subdivision:**
- Options: 1/64, 1/32, 1/16, 1/8, 1/4, 1/2, 1, 2, 4, 8
- Controls: How often modulator cycles
- Example: 1/4 = cycles every quarter note

**Waveform:**
- Options: sine, triangle, square, sawtooth, noise
- Controls: Wave shape for time-based modulation
- Note: Combined with audio reactivity

**Gain:**
- Range: 0.0-10.0
- Default: 1.0
- Controls: Modulator intensity multiplier
- Zero: Disables modulator entirely

**Freq1 (Hz):**
- Range: 0-24000 Hz
- Default: 30 Hz (kick drum low end)
- Controls: Lower frequency bound
- Disabled: When no audio loaded (cyan overlay)

**Freq2 (Hz):**
- Range: 0-24000 Hz
- Default: 50 Hz (kick drum high end)
- Controls: Upper frequency bound
- Validation: Must be ≥ Freq1

**Typical Ranges:**
- Kick drums: 30-50 Hz
- Sub-bass: 50-75 Hz
- Bass: 80-250 Hz
- Low-mid: 250-500 Hz

#### Modulator B (Mod B)

Same controls as Mod A.

**Default Frequencies:**
- Freq1: 50 Hz (sub-bass low)
- Freq2: 75 Hz (sub-bass high)

#### Modulator C (Mod C)

Same controls as Mod A.

**Default Frequencies:**
- Freq1: 100 Hz (lead bass low)
- Freq2: 500 Hz (lead bass high)

### Section 3: Output Settings

**Orientation:**
- Portrait: 9:16 aspect (vertical, social media)
- Landscape: 16:9 aspect (horizontal, standard)
- Square: 1:1 aspect (Instagram feed)

**Resolution:**
- 720p: 1280×720 or 720×1280 (fast, testing)
- 1080p: 1920×1080 or 1080×1920 (standard, recommended)
- 1440p: 2560×1440 or 1440×2560 (high quality)
- 2160p: 3840×2160 or 2160×3840 (maximum quality, slow)

**FPS:**
- 24: Cinematic, smallest file size
- 30: Standard video, balanced
- 60: Smooth motion, larger file

### Section 4: Control States

**Purpose:** Save and load complete render configurations.

**Dropdown:**
- Library States (5 presets, can't delete)
- User States (your custom configs, can delete)
- Custom (current values, always available)

**Library States:**
1. Quick Test: 120bpm, Portrait 720p 24fps, 30sec
2. HD Render: 120bpm, Landscape 1080p 60fps, 1min
3. 4K Final: 120bpm, Landscape 2160p 60fps, 30sec
4. Instagram Story: 120bpm, Portrait 1080p 30fps, 15sec
5. TikTok: 120bpm, Portrait 1080p 60fps, 30sec

**Save Button:**
- Click to save current state
- Inline input appears
- Enter custom name
- Press Enter to confirm, Escape to cancel

**Delete Button:**
- Removes selected user state
- Library states protected (can't delete)
- No confirmation prompt

**Info Box:**
- Displays current configuration
- Shows all 21 values
- Updates in real-time

### Section 5: Render Controls

**Render Button:**
- Starts video capture
- Button text changes to "Stop Render"
- Progress displays in status bar

**Stop Render Button:**
- Ends capture early
- Finalizes video
- Downloads partial render

---

## CODE TAB

### Code Editor

**Features:**
- Textarea for WGSL shader code
- Monospace font (Courier New)
- Line wrapping enabled
- Full-width editing area

**Required Structure:**
```wgsl
// Uniform struct (DO NOT MODIFY)
struct Uniforms { ... }

// Vertex shader (DO NOT MODIFY)
@vertex fn vertexMain(...) { ... }

// Fragment shader (YOUR CODE HERE)
@fragment fn main(...) { ... }
```

### Compile Button

**Function:**
- Compiles WGSL shader code
- Updates canvas preview
- Shows errors in status bar

**Success:**
- Canvas updates with new shader
- Status: "Shader compiled successfully"

**Error:**
- Canvas shows previous shader
- Status: Error message with line number

### Save to Presets

**Function:**
- Saves current shader to User Presets
- Generates thumbnail automatically
- Stores in browser LocalStorage

**Process:**
1. Write/load shader code
2. Click Compile to test
3. Enter preset name
4. Click Save
5. Preset appears in Presets tab

---

## PRESETS TAB

### Library Section

**5 Curated Shaders:**
1. Pulsing Orb
2. Frequency Bars
3. Kaleidoscope
4. Wave Pattern
5. Grid Tiles

**Features:**
- Visual thumbnails (WebGPU rendered)
- Always available
- Can't be deleted
- Read-only

### User Presets Section

**Your Custom Shaders:**
- Saved from Code tab
- Visual thumbnails
- Editable (load and modify)
- Deletable

**Card Display:**
- Thumbnail preview (240x180px)
- Shader name below
- Delete button (×)
- Hover effect

### Navigation

**Keyboard:**
- Tab: Next preset
- Shift+Tab: Previous preset
- Enter: Load selected preset
- Focused preset has cyan border

**Mouse:**
- Click card to load
- Click × to delete (no confirmation)
- Hover for visual feedback

### Import Shaders

**Drag-and-Drop:**
1. Drag .txt or .wgsl file
2. Drop anywhere in Presets tab
3. Cyan border indicates drop zone
4. Shader imports to User Presets

**File Types:**
- .txt (plain text shader code)
- .wgsl (WGSL shader files)

---

## AUDIO SYSTEM

### Upload Audio

**Button:** "Upload Audio" below canvas

**Supported Formats:**
- MP3 (recommended)
- WAV
- M4A
- OGG

**Process:**
1. Click Upload Audio
2. Select file
3. Wait for load
4. Waveform displays
5. Enter tempo manually

### Audio Controls

**Location:** Strip below waveform (when audio loaded)

**Audio Tempo (BPM):**
- Manual entry required
- Controls beat positioning
- Independent of Shader Tempo

**Audio Start/End (MM:SS.mmm):**
- Song boundaries
- Usually 00:00.000 to song end
- Rarely need to change

**Loop In/Out (bar.beat):**
- Format: 1.1, 2.3, 5.4, etc.
- First number: Bar (1-indexed)
- Second number: Beat (1-4 for 4/4 time)
- Arrow keys: Navigate by beats
- Defines render range when audio loaded

### Waveform Visualization

**Display:**
- Audio amplitude over time
- Beat grid overlay (vertical lines)
- Loop In/Out brackets (cyan)
- Current position marker

**Interaction:**
- Click to seek
- Arrow keys to nudge
- Visual feedback for loop range

### Loop Playback

**Loop Button:**
- Toggles loop mode
- Cyan when active
- Loops between In/Out brackets

**Playback:**
- Play/Pause button
- Audition loop section
- Fine-tune loop points

### Remove Audio

**Button:** "Remove Audio"

**Function:**
- Clears audio file
- Resets to shader-only mode
- Render Duration field re-enabled
- Frequency fields disabled

---

## CONTROL STATES SYSTEM

### What Gets Saved

**21 Values Total:**

**Shader Timing (3):**
- tempo
- loopLength
- renderDuration

**Modulator A (5):**
- modASubdiv
- modAWaveform
- modAGain
- modAFreq1
- modAFreq2

**Modulator B (5):**
- modBSubdiv
- modBWaveform
- modBGain
- modBFreq1
- modBFreq2

**Modulator C (5):**
- modCSubdiv
- modCWaveform
- modCGain
- modCFreq1
- modCFreq2

**Output (3):**
- orientation
- resolution
- fps

### Saving States

**Process:**
1. Adjust all controls to desired values
2. Click Save button
3. Inline input appears
4. Enter name (e.g., "Techno Drop Config")
5. Press Enter to save
6. State appears in dropdown

**Naming Tips:**
- Descriptive: "128 BPM Bass Heavy"
- Track-specific: "Track Name - Drop"
- Output-focused: "4K Landscape Export"
- Avoid generic names: "Config 1"

### Loading States

**Process:**
1. Select state from dropdown
2. All 21 fields update instantly
3. Info box displays configuration
4. Ready to render

**Auto-Load:**
- Last used state loads on page refresh
- Library states always available
- Custom state shows current values

### Custom State

**What It Is:**
- Always present in dropdown
- Shows current control values
- Updated automatically on field change

**When It Appears:**
- Any control field edited
- Indicates unsaved configuration
- Not a saved state (just current values)

### Deleting States

**Process:**
1. Select state from dropdown
2. Click Delete button
3. State removed immediately
4. No confirmation prompt

**Restrictions:**
- Library states can't be deleted
- Delete button disabled for library states
- Only user states deletable

### Info Box

**Displays:**
```
State Name
Tempo: 120 BPM | Loop: 16 bars | Duration: 00:30.000
Mod A: 1/4 sine × 1 | 30-50 Hz
Mod B: 1/4 sine × 1 | 50-75 Hz
Mod C: 1/4 sine × 1 | 100-500 Hz
Output: portrait 720p @ 24fps
```

**Updates:**
- On state load
- On field change
- Real-time display

---

## FREQUENCY MAPPING

### Purpose

Target specific frequency ranges with each modulator for precise audio reactivity.

### How It Works

**1. FFT Analysis:**
- Audio analyzed at 32768 samples
- 16384 frequency bins created
- Each bin ~1.46 Hz wide

**2. User-Defined Range:**
- You set Freq1 (lower bound)
- You set Freq2 (upper bound)
- System calculates which bins to use

**3. Amplitude Extraction:**
- Average amplitude across bins
- Normalize to 0-1 range
- Apply exponential smoothing (reduces jitter)

**4. Gain Application:**
- Multiply by user gain (0-10x)
- Final value sent to shader as modA/modB/modC

### Frequency Ranges by Genre

**House/Techno:**
- Mod A: 30-50 Hz (kick drums)
- Mod B: 50-100 Hz (sub-bass)
- Mod C: 100-500 Hz (bass synths)

**Dubstep:**
- Mod A: 30-60 Hz (deep bass)
- Mod B: 60-150 Hz (wobble bass)
- Mod C: 150-500 Hz (mid-bass)

**Drum & Bass:**
- Mod A: 40-80 Hz (kick/bass)
- Mod B: 100-300 Hz (snares/percussion)
- Mod C: 1000-5000 Hz (hats/cymbals)

### Validation

**Push Behavior:**
- Increase Freq1: Pushes Freq2 up if needed
- Decrease Freq2: Pushes Freq1 down if needed
- Prevents invalid ranges automatically

**Bounds:**
- Minimum: 0 Hz
- Maximum: 24000 Hz (Nyquist frequency)
- Clamped to valid range

**Visual Feedback:**
- Red border on invalid input
- Corrects on blur
- Status bar shows validation

### Arrow Key Navigation

**Fine Adjustment:**
- ↑ Arrow: +5 Hz
- ↓ Arrow: -5 Hz

**Coarse Adjustment:**
- Shift+↑: +50 Hz
- Shift+↓: -50 Hz

**Use Case:**
- Quick frequency tuning
- Live adjustment during testing
- Precise targeting of bass elements

### Disabled State

**When Audio Not Loaded:**
- Frequency fields disabled
- Cyan overlay indicates "Audio Required"
- Values visible but not editable
- Prevents confusion about non-functional controls

**When Audio Loaded:**
- Frequency fields enabled
- Normal background color
- Editable and functional
- FFT analysis active

### Gain Control

**Purpose:** Scale modulator intensity

**Values:**
- 0.0: Disabled (no reactivity)
- 0.3-0.5: Subtle movement
- 0.8-1.2: Normal response
- 1.5-3.0: Strong reaction
- 5.0-10.0: Extreme effect

**Use Cases:**
- 0 gain: Disable specific modulator
- Low gain: Background subtle motion
- High gain: Dominant visual element

---

## VIDEO RENDERING

### Render Process

**1. Setup:**
- Select Control State or adjust fields
- Load shader (if not already loaded)
- Optionally upload audio

**2. Start Render:**
- Click Render button
- Canvas switches to selected resolution
- Recording begins

**3. Progress Display:**
- Status bar shows: "Rendering 00:15/00:30 | Frame 450/900"
- Updates every 5 frames
- Elapsed time and frame count

**4. Completion:**
- Video finalizes automatically
- Browser download dialog
- Filename: OMS_Dojo_[timestamp].webm

### Render Modes

**Shader-Only Mode (No Audio):**
- Duration: Render Duration field
- Output: Silent video (no audio track)
- Use case: Pre-rendered loops for DJ sets

**Audio Mode (With Audio):**
- Duration: Loop In to Loop Out (overrides Render Duration)
- Output: Video with audio track (loop range)
- Use case: Testing with audio, YouTube videos

### Export Settings

**Container:** WebM  
**Video Codec:** VP8  
**Audio Codec:** Opus (when audio included)

**Bitrate:** Variable (quality-based)

**Post-Processing:**
- Convert to ProRes 422 with FFmpeg for Ableton/EboSuite
- See FFmpeg Video Guide for commands

### Render Time Estimates

**M1 Mac (Reference):**
- 720p 24fps 30sec: ~45 seconds
- 1080p 30fps 60sec: ~2 minutes
- 1440p 60fps 30sec: ~4 minutes
- 2160p 60fps 30sec: ~8 minutes

**Factors:**
- Shader complexity (more calculations = slower)
- Resolution (4K is 4x pixels of 1080p)
- FPS (60fps is 2.5x frames of 24fps)
- Hardware (GPU speed matters most)

### Stopping Render Early

**Process:**
1. Click "Stop Render" button
2. Recording ends
3. Partial video downloads

**Use Case:**
- Made mistake in settings
- Shader looks wrong
- Want to test different approach

---

## WORKFLOWS

### Workflow 1: Fast Shader Testing

**Goal:** Quickly preview many shaders to find the best.

**Steps:**
1. Select "Quick Test" Control State
2. Go to Presets tab
3. Tab through shaders
4. Press Enter to load
5. Click Render
6. Review 30sec preview
7. Repeat for next shader

**Time per shader:** ~2-3 minutes

### Workflow 2: Bass-Reactive Video

**Goal:** Create video responsive to bass music elements.

**Steps:**
1. Upload bass-heavy track
2. Enter tempo manually
3. Set Loop In/Out to drop section
4. Configure modulators:
   - Mod A: 30-50 Hz (kicks)
   - Mod B: 50-75 Hz (sub)
   - Mod C: 100-500 Hz (bass)
5. Test with Loop playback
6. Fine-tune gains
7. Save as Control State
8. Select HD Render or 4K Final
9. Render video

**Time:** ~15-20 minutes

### Workflow 3: DJ Set Preparation

**Goal:** Create videos for complete DJ set.

**Steps:**
1. Create shaders for each track (or use library)
2. For each track:
   - Upload audio
   - Set Loop In/Out
   - Define frequency ranges
   - Test reactivity
   - Save Control State (track-specific)
   - Render video
   - Convert to ProRes 422
3. Import all videos to Ableton
4. Load into EboSuite
5. Trigger with MIDI controller during set

**Time per track:** ~20-30 minutes  
**10-track set:** ~4-6 hours total

### Workflow 4: Social Media Content

**Goal:** Create vertical videos for Instagram/TikTok.

**Steps:**
1. Select "Instagram Story" or "TikTok" preset
2. Load vibrant shader
3. Render (no audio needed)
4. Upload directly to platform

**Time:** ~1-2 minutes

### Workflow 5: Custom Shader Development

**Goal:** Create unique shader with Claude AI.

**Steps:**
1. Describe vision to Claude
2. Claude generates WGSL code
3. Paste into Code tab
4. Click Compile
5. Preview on canvas
6. Refine with Claude
7. Repeat until satisfied
8. Save to User Presets

**Time:** ~10-30 minutes (2-5 iterations)

---

## KEYBOARD SHORTCUTS

### Global
- **Ctrl+S:** Save current shader (Code tab)
- **Ctrl+R:** Start/stop render
- **Space:** Play/pause audio (when loaded)

### Frequency Fields (When Focused)
- **↑ Arrow:** +5 Hz
- **↓ Arrow:** -5 Hz
- **Shift+↑:** +50 Hz
- **Shift+↓:** -50 Hz

### Presets Tab
- **Tab:** Next preset
- **Shift+Tab:** Previous preset
- **Enter:** Load selected preset
- **Delete:** Delete selected user preset

### Control States Save Input
- **Enter:** Confirm save
- **Escape:** Cancel save

### Audio Playback
- **Space:** Play/pause
- **←/→ Arrows:** Seek by beat (when focused on Loop In/Out)

---

## TROUBLESHOOTING

### WebGPU Not Supported

**Symptom:** Blank canvas, error message about WebGPU.

**Solution:**
- Use Safari Technology Preview 15+
- Or Chrome Canary with WebGPU flag enabled
- Check chrome://flags or edge://flags
- Enable "Unsafe WebGPU"

### Shader Won't Compile

**Symptom:** Error message in status bar, canvas doesn't update.

**Common Issues:**
1. **Syntax Error:** Check WGSL syntax, missing semicolons
2. **Type Mismatch:** Verify f32, vec2<f32>, etc.
3. **Uniform Struct:** Must match template exactly
4. **Return Statement:** Must return vec4<f32> with alpha = 1.0

**Solution:**
- Read error message carefully
- Check line number
- Compare to working examples
- Ask Claude for fix

### Audio Won't Load

**Symptom:** Upload button does nothing, no waveform.

**Solution:**
- Check file format (MP3, WAV, M4A, OGG)
- Try different audio file
- Check browser console for errors
- Verify audio file not corrupted

### Frequency Fields Not Updating Info Box

**Symptom:** Change freq values, info box doesn't update.

**Solution:**
- This is expected when no audio loaded
- Frequency fields disabled until audio present
- Upload audio track, fields become active
- Info box updates automatically

### Render Progress Stuck

**Symptom:** Render starts, progress doesn't update.

**Solution:**
- Wait longer (complex shaders take time)
- Check browser console for errors
- Stop render and try again
- Reduce resolution/FPS for testing

### Video Download Failed

**Symptom:** Render completes, no download.

**Solution:**
- Check browser download settings
- Allow automatic downloads
- Check disk space
- Try different browser

### Control State Won't Load

**Symptom:** Select state, fields don't update.

**Solution:**
- Clear localStorage: `localStorage.clear()` in console
- Refresh page
- Library states reload automatically
- Recreate user states

### Canvas Shows Black Screen

**Symptom:** Shader compiles, canvas is black.

**Solution:**
- Check shader returns vec4 with alpha = 1.0
- Verify color calculations valid (not NaN)
- Check uniforms accessed correctly
- Test with simple shader (solid color)

### Poor Performance

**Symptom:** Frame drops, stuttering preview.

**Solution:**
- Reduce shader complexity
- Lower preview resolution (browser zoom)
- Close other tabs/applications
- Check GPU usage in Activity Monitor
- Consider reducing FFT size (see Developer Documentation)

---

## TIPS & BEST PRACTICES

### Shader Design

**Start Simple:**
- Begin with basic shapes
- Add complexity incrementally
- Test each addition

**Loop Considerations:**
- Design for seamless looping
- Use modulo for repeating patterns
- Test loop points in preview

**Color Choices:**
- High contrast for visibility
- Complementary colors (cyan/magenta)
- Consider venue lighting

### Audio Setup

**Tempo Entry:**
- Use precise BPM from DAW
- Test with loop playback
- Adjust ±0.01 if drift occurs

**Loop Positioning:**
- Target impactful sections (drops, breakdowns)
- Use arrow keys for fine positioning
- Audition loop multiple times

**Frequency Ranges:**
- Start with defaults (30-50, 50-75, 100-500)
- Adjust based on track characteristics
- Test with different gain settings

### Render Strategy

**Development:**
- Use Quick Test (720p 24fps 30sec)
- Iterate rapidly
- Don't waste time on 4K yet

**Final Export:**
- Use HD Render or 4K Final
- Only render tested shaders
- Convert to ProRes 422 for Ableton

**File Management:**
- Organize renders by track name
- Use consistent naming
- Keep WebM source files
- Archive ProRes conversions

### Control States

**Organization:**
- Create common presets first (test/preview/final)
- Track-specific states for DJ sets
- Social media presets (IG/TikTok)
- Delete unused states regularly

**Naming:**
- Descriptive names
- Include key details (BPM, resolution)
- Track names for DJ workflows

---

## APPENDIX

### Time Format Reference

**MM:SS.mmm:**
- MM: Minutes (00-99)
- SS: Seconds (00-59)
- mmm: Milliseconds (000-999)

**Examples:**
- 00:30.000 = 30 seconds
- 01:15.500 = 1 minute 15.5 seconds
- 00:00.250 = 250 milliseconds

### Bar.Beat Format

**Format:** bar.beat (1-indexed)

**4/4 Time:**
- 1.1 = Bar 1, Beat 1
- 1.4 = Bar 1, Beat 4
- 2.1 = Bar 2, Beat 1
- 5.3 = Bar 5, Beat 3

**Calculation:**
- Bar 1 starts at beat 0.0
- Bar 2 starts at beat 4.0
- Bar 5 starts at beat 16.0

### Resolution Table

**Portrait (9:16):**
- 720p: 720×1280
- 1080p: 1080×1920
- 1440p: 1440×2560
- 2160p: 2160×3840

**Landscape (16:9):**
- 720p: 1280×720
- 1080p: 1920×1080
- 1440p: 2560×1440
- 2160p: 3840×2160

**Square (1:1):**
- 720p: 720×720
- 1080p: 1080×1080
- 1440p: 1440×1440
- 2160p: 2160×2160

### Frequency Bands Reference

**Sub-Bass:** 20-60 Hz  
**Bass:** 60-250 Hz  
**Low-Mid:** 250-500 Hz  
**Mid:** 500-2000 Hz  
**High-Mid:** 2000-6000 Hz  
**Treble:** 6000-20000 Hz

**Musical Elements:**
- Kick drums: 30-80 Hz
- Sub-bass synths: 40-100 Hz
- Bass guitars: 80-200 Hz
- Snares: 150-250 Hz
- Vocals: 300-3000 Hz
- Hi-hats: 5000-15000 Hz

---

## SUPPORT

**Website:** https://www.itswessmithyo.com/  
**License:** GPL-3.0 (free and open source)  
**Documentation:** Complete package included

**For Issues:**
- Check browser console (F12)
- Verify WebGPU enabled
- Try hard refresh (Ctrl+Shift+R)
- Review troubleshooting section

---

**END OF COMPLETE USER GUIDE**
