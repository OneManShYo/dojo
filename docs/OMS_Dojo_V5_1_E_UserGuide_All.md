# ONEMANSHYO App [Beta V5_1_0] - USER GUIDE

**Version:** Beta V5_1_0
**Date:** December 24, 2025
**License:** GPL-3.0

---

## Table of Contents

1. Introduction
2. Instant Gratification
3. Using the Application
   - 3.2.5.4 Cue Points (V5_1_0)
   - 3.4.10 Cue Points in PERFORM (V5_1_0)
   - 3.5.7 BumpR StickRs (V5_1_0)
4. Support
5. Credits

APPENDIX A: DJ 101 Foundation
APPENDIX B: ONEMANSHYO Origin Story
APPENDIX C: Shader Creation Guide
APPENDIX D: FFmpeg Video Conversion

INDEX A: Visual Vocabulary

---

## 1. Introduction

### 1.1 What is OMS Beta?

OMS Beta is a browser-based tool for DJing with live reactive visuals. No installation, no licenses - just open the HTML file. You create audio-reactive shaders, organize them in a library, and perform live with a 4-track mixer that combines audio and visuals.

**Platform:** macOS only  
**Tested on:** M1 Pro MacBook Pro 14" (2021)  
**Browser:** Chrome (WebGPU support required)  
**Screen:** 1352×761 effective viewport (14" MacBook Pro)

**File:** `OMS_App_Beta_V5_1_0.html`

---

## 2. Instant Gratification

### 2.1 Quick Visual Content

If you want to create visuals based on music timing without setting up a library:

**Audio-Reactive Visuals (reacts to your music):**
1. Open `OMS_App_Beta_V5_1_0.html`
2. Drag your audio file (.mp3, .wav) onto the waveform
3. Visuals automatically react to bass, mids, treble
4. Click [Render Video]
5. Get SILENT video with pre-baked reactivity

**Tempo-Synced Visuals (animated to BPM, no audio needed):**
1. Open `OMS_App_Beta_V5_1_0.html`
2. Set tempo (e.g., 128 BPM)
3. Set duration (e.g., 03:30.000)
4. Click [Render Video]
5. Get animated visual

**What you get:**
- WebM video file (convert with FFmpeg - see APPENDIX D)
- SILENT video (audio reactivity is pre-baked into visuals)
- Ready for social media, streaming, or professional playback

**Use it for:**
- Instagram, TikTok, YouTube content
- DJ visual content
- Live stream backgrounds
- Convert to ProRes for Ableton/EboSuite

### 2.2 Library Setup

To save containers and use performance features:
1. Open OMS Beta
2. Go to ADMIN tab
3. Click "Set Application Folder"
4. Select a folder on your computer
5. OMS creates subfolders: /Library/, /Presets/, /Exports/, /BumpRStickRs/
6. Now you can save containers and perform live

---

## 3. Using the Application

### 3.1 Navigation

**Tab Bar** (top of window)

Four tabs available:

| Tab | Label | Color | Purpose |
|-----|-------|-------|---------|
| 1 | CREATE | Cyan | Make shaders |
| 2 | ORGANIZE | Yellow | Manage library |
| 3 | PERFORM | Magenta | DJ live |
| 4 | ADMIN | Grey | Settings |

**Switching tabs:**
- Click tab name
- Keyboard: Ctrl+1, Ctrl+2, Ctrl+3, Ctrl+4

---

### 3.2 CREATE

#### 3.2.1 Overview

The CREATE tab is where you build shaders. When you open OMS Beta, you start here with a shader already running.

**What you can do:**
- Edit shader code
- Configure timing and modulators
- Load audio for reactivity
- Render video output
- Add containers to library

---

#### 3.2.2 Core Concepts

##### 3.2.2.1 Shaders

A **shader** is GPU code that renders visuals in real-time.

**Language:** WGSL (WebGPU Shading Language)

**What it receives (uniforms - auto-injected by app):**
- `uniforms.time` - seconds since start
- `uniforms.tempo` - beats per minute
- `uniforms.beat` - tempo-synced beat position (V4_8_0)
- `uniforms.modA`, `uniforms.modB`, `uniforms.modC` - combined modulators (0.0-1.0)
- `uniforms.resolutionX`, `uniforms.resolutionY` - canvas dimensions

**Independent modulator components (V4_8_0):**
- `uniforms.shaderModAWave`, `shaderModBWave`, `shaderModCWave` - waveform output (0-1)
- `uniforms.shaderModAGain`, `shaderModBGain`, `shaderModCGain` - gain value (0-1)
- `uniforms.shaderModAFFT`, `shaderModBFFT`, `shaderModCFFT` - FFT level (0-1)

**Transform uniforms (V5_0_0):**
- `uniforms.transformRotation` - rotation in radians
- `uniforms.transformScaleX`, `transformScaleY` - scale factors
- `uniforms.transformTranslateX`, `transformTranslateY` - position offset
- `uniforms.transformPivotX`, `transformPivotY` - rotation center
- `uniforms.transformTile` - tile repeat count

**What it does:**
- Runs once per pixel
- Calculates color based on position and time
- Returns RGBA color

**What it returns:**
- Color for each pixel: `vec4<f32>(r, g, b, a)`

**Example:**
```wgsl
@fragment
fn fs_main(@builtin(position) pos: vec4<f32>) -> @location(0) vec4<f32> {
    let pulse = sin(uniforms.time * uniforms.tempo / 60.0) * 0.5 + 0.5;
    return vec4<f32>(pulse, 0.0, uniforms.modA * 0.2, 1.0);
}
```

**For complete shader creation guide, see APPENDIX C.**

---

##### 3.2.2.2 Modulators

**Modulators** are values (0.0-1.0) that change over time to drive visual effects.

**Three modulators:**

**Modulator A (Bass)**
- Frequency range: 30-500 Hz
- Reacts to: Kick drums, sub bass
- Best for: Big movements, main pulse

**Modulator B (Mids)**
- Frequency range: 500-2000 Hz
- Reacts to: Snares, vocals, synths
- Best for: Color shifts, secondary motion

**Modulator C (Treble)**
- Frequency range: 2000-20000 Hz
- Reacts to: Hi-hats, cymbals
- Best for: Fine details, sparkle

---

##### 3.2.2.3 Two-Layer System

**Layer 1: Waveform**
- Tempo-synced oscillator
- Types: None, Static, Sine, Square, Triangle, Sawtooth, Cycler, etc.
- Subdivision: 1/4, 1/2, Whole, 2 Bars, etc.
- Runs even without audio loaded

**Layer 2: Audio (optional)**
- FFT analysis of loaded audio
- Extracts frequency data in real-time
- Multiplies waveform value

**Final calculation:**
```
modulator = waveform × audioLevel × gain
```

**Without audio:** Pure waveform (visual timing)  
**With audio:** Reactive waveform (responds to music)

---

##### 3.2.2.4 Tempo

**Two independent tempo settings:**

**Shader Tempo** (60-200 BPM)
- Controls visual animation speed
- Drives modulator waveforms
- Example: 128 BPM = moderately fast animation

**Audio BPM**
- The tempo of your audio file
- Used for beatgrid alignment
- Used for time-stretching in PERFORM tab

**They are NOT the same:**
- Shader animates at 128 BPM
- Audio track is 124 BPM
- Both work independently
- Visual speed ≠ audio speed

---

#### 3.2.3 Interface Layout

**From top to bottom:**

1. **Preview Canvas** - Large shader display (1352×761 on MacBook Pro 14")
2. **SHADER CONTROLS Header** - [Code] and [Save] buttons on right
3. **Controls View** OR **Code View** (same space, toggled)
4. **AUDIO CONTROLS Header** - [Swap] and [X] buttons on right
5. **Audio Section** - Waveform, transport, timing controls

---

#### 3.2.4 Controls View

##### 3.2.4.1 Name Section (V4_7_0)

**Shader Name** (dropdown)
- Shows current shader name
- Click to see available shaders
- Options include: System presets, User presets, Library containers
- Select to load shader code immediately

---

##### 3.2.4.2 Orientation Section (V4_7_0)

**Orientation** (dropdown)
- Options: Landscape, Portrait, Square
- Landscape = 16:9 (1920×1080)
- Portrait = 9:16 (1080×1920)
- Square = 1:1 (1080×1080)
- Auto-saves to project file

**Note:** Moved from OUTPUT section - orientation is fundamental to shader design.

---

##### 3.2.4.3 Timing Section

**Shader Tempo** (dropdown: 60-200 BPM)
- Controls visual animation speed
- NOT audio BPM
- Higher = faster animation
- **Disabled when audio loaded** (V4_8_0) - Audio Tempo becomes THE tempo

**Duration** (text field: MM:SS.mmm)
- Render length when no audio loaded
- Example: `03:30.000` = 3 minutes 30 seconds
- Only used for shader-only renders

---

##### 3.2.4.4 Reactivity Section

**Modulator Controls** (5 columns × 3 rows)

Each modulator (A, B, C) has:

**Column 1: Subdivision** (dropdown)
- Options: None, 1/4, 1/2, Whole, 2 Bars, 4 Bars, 8 Bars, 16 Bars
- How often waveform repeats
- Example: "Whole" = once per bar
- **None** = modulator disabled (outputs 0)

**Column 2: Waveform** (dropdown)
- Options: None, Static, Sine, Square, Triangle, Sawtooth, Cycler, Chaosmode, Madmax, Psycho
- Shape of oscillation
- Sine = smooth, Square = sharp
- **None** = modulator disabled (outputs 0)
- **Static** = no oscillation, gain controls value directly

**Modulator Modes Explained:**

| Mode | Output | Use Case |
|------|--------|----------|
| **None** | Always 0 | Modulator disabled, shader ignores it |
| **Static** | Gain value (0-1) | Manual control, no animation |
| **Waveform** | Oscillating × Gain | Animated (sine, square, etc.) |

**Performance Progression:** Start with None (static visual), move to Static (manual control), then add Waveforms (animated).

**Column 3: Gain** (rotary knob: 0.0-1.0) (V4_8_0)
- Visual knob control (drag up/down)
- Amplitude multiplier
- 0.1 = default, 1.0 = maximum strength
- With Static waveform: directly controls modulator value

**Column 4: Freq1** (rotary knob: 20-24000 Hz) (V4_8_0)
- Visual knob control (drag up/down)
- Audio frequency range start (log scale)
- **Disabled when no audio loaded** - FFT ranges meaningless without audio
- Default: modA=30, modB=50, modC=100

**Column 5: Freq2** (rotary knob: 20-24000 Hz) (V4_8_0)
- Visual knob control (drag up/down)
- Audio frequency range end (log scale)
- **Disabled when no audio loaded**
- Default: modA=120, modB=500, modC=8000

---

##### 3.2.4.5 TRANSFORMS Section (V5_0_0)

Controls to transform the shader output in real-time. All transforms save with your .oms containers.

**ROTATION** (endless encoder)
- Drag up/down to rotate (0° to 360°, wraps continuously)
- Full 360° circle visual with indicator line
- Double-tap Shift+Enter to reset to 0°

**SCALE** (encoder)
- Drag up/down to scale (0× to 4×)
- Affects both X and Y uniformly
- Double-tap Shift+Enter to reset to 1.0×

**TILE** (encoder)
- Drag up/down to repeat pattern (1 to 8)
- Double-tap Shift+Enter to reset to 1

**TRANSLATE** (XY pad, 50×30)
- Drag to move shader position (-1 to +1)
- Double-tap Shift+Enter to reset to center (0, 0)

**PIVOT** (XY pad, 50×30)
- Sets rotation/scale center point (-1 to +1)
- Double-tap Shift+Enter to reset to center (0, 0)

**Keyboard Mode (Shift+T):**

| Key | Action |
|-----|--------|
| `Shift+T` | Enter/exit transform mode |
| `← →` | Navigate between controls |
| `Shift+← →` | Adjust encoder values |
| `Shift+↑↓←→` | Move XY/Pivot pads |
| `Shift+Enter×2` | Reset selected to default |
| `Escape` | Exit transform mode |

**Shift+Trackpad:** Hover over any control + Shift + scroll to adjust. Also works in keyboard mode - select control, then Shift + scroll anywhere.

---

##### 3.2.4.6 CONTAINER OUTPUT Section (V5_1_0)

**Section renamed from "OUTPUT"** to clarify dual purpose: Container = Shader + Audio + Settings.

**Header Buttons:**

| Button | Action | Visibility |
|--------|--------|------------|
| [Save] | Creates NEW container in Library | Always visible |
| [Update] | Updates currently loaded container | Only when container loaded |

**Save vs Update:**
- **Save** = Always creates a new container with the current name
- **Update** = Overwrites the loaded container, keeps the name
- After Save, Update button appears (you now have a loaded container)
- Update stays on CREATE tab (no navigation)

**Hover Messages:** All controls show info messages on hover:
- Header: "Container = Shader + Audio + Settings"
- Save: "Save as NEW container to Library"
- Update: "Update loaded container (keeps name)"
- Resolution: "Video resolution for render"
- FPS: "Frame rate for render"
- Filename: "Name for container and video file"
- R button: "Render video file"

**Filename** (text field)
- Auto-populated with shader name + audio name (V4_7_0)
- Result: `OMS_[filename]_[resolution]_[fps]fps.webm`
- Edit anytime, used for both render and save

**Resolution** (dropdown)
- Options: 720p, 1080p, 1440p, 2160p
- Recommended: 1080p (1920×1080)

**FPS** (dropdown)
- Options: 24, 30, 60
- 60 = smooth, 30 = smaller files

**[Render Video] Button**
- Start recording
- Progress bar shows status
- Output: Downloads folder, WebM format
- **Note:** Video is SILENT (reactivity pre-baked)

---

#### 3.2.5 Audio Section

##### 3.2.5.0 Audio Controls Header (V4_7_0)

**Header Controls:**
- **[Swap]** - Open Library audio picker dropdown
- **[X]** - Remove loaded audio (disabled when no audio)

**Swap Dropdown:**
- Shows audio files from your Library folder
- Grouped: "Unused" files first, then "In Use"
- Select to load audio immediately
- Requires folder to be set

---

##### 3.2.5.1 Loading Audio

**Drag & Drop**
- Drag .mp3 or .wav onto waveform area
- Loads automatically

**Swap Button (V4_7_0)**
- Click Swap in header
- Select from Library audio files
- Loads immediately

**What happens:**
- Waveform displays (blue bars)
- Audio controls appear
- Beatgrid calculates
- Filename field updates with audio name

---

##### 3.2.5.2 Waveform Display

**Visual elements:**
- Blue bars = amplitude
- Cyan/magenta lines = beat markers
- Magenta vertical line = playhead

**When empty:**
- Shows "Drag/Drop Audio" placeholder

**Interaction:**
- Click waveform = jump playhead
- Spacebar = play/pause
- Arrow keys (← →) = navigate by beat

---

##### 3.2.5.3 Audio Controls Strip

| Control | Function |
|---------|----------|
| [Play] | Play/pause (Spacebar also works) |
| Audio Start | Beatgrid start (Click: reset, Shift+Click: pin to playhead) |
| Audio End | Container end (Click: reset, Shift+Click: pin to playhead) |
| Tempo | Track tempo (was "Audio BPM") |
| Pitch | (future feature - not functional) |
| Loop In | Loop start (bar.beat format, e.g., "1.1") |
| Loop Out | Loop end (bar.beat format, e.g., "65.1") |
| [Loop] | Enable/disable looping |
| [G] | Toggle beatgrid visibility |
| [S] | Snap Audio End to last bar |
| Last Bar | Total bars (read-only) |

**Loop Defaults (V4_7_0):**
- Change Loop In/Out → saves to project file
- Fresh audio uses your defaults
- Existing containers keep their own values

**In/Out Labels:**
- Click "In" label = jump playhead to In point
- Click "Out" label = jump playhead to Out point

---

##### 3.2.5.4 Cue Points (V5_1_0)

CDJ-style hot cues for instant navigation. 3 cue points per container, displayed as orange flags on the waveform.

**Default Positions:**

| Cue | Position | Beats | Typical Use |
|-----|----------|-------|-------------|
| 1 | Bar 1.1 | 0 | Intro |
| 2 | Bar 17.1 | 64 | First drop |
| 3 | Bar 33.1 | 128 | Second drop |

**Visual Appearance:**
- Orange vertical line (#ffa500)
- Orange triangle flag pointing right
- Number (1, 2, or 3) inside flag

**Interaction:**

| Action | Result |
|--------|--------|
| Click flag (no drag) | Jump playhead to cue position |
| Drag flag | Move cue to new position |
| Release drag | Cue snaps to nearest bar |

**Bar Snap:** Cue positions always snap to bar boundaries (every 4 beats).

**Storage:** Cue points save with your .oms container. Legacy containers default to 1.1, 17.1, 33.1.

**Workflow:**
1. Load audio
2. Play through to find key moments
3. Drag cue flags to those positions
4. Save container - cues are stored!
5. In PERFORM tab, press 1/2/3 to jump to cues

---

#### 3.2.6 Code View

**Access:** Click [Code] button in tab header

**Interface:**
- Large textarea with WGSL shader code
- Four buttons below

**Buttons:**

| Button | Function |
|--------|----------|
| [Compile] | Compile and preview shader |
| [Copy] | Copy code to clipboard |
| [Save] | Save to /Presets/ folder as .wgsl |
| [Export] | Download as .wgsl file |

**Toggle back:** Click [Code] button again

---

#### 3.2.7 Adding to Library

**[Add to Library] Button** (bottom of page)

**Process:**
1. Click button
2. Enter container name
3. Click OK

**What happens:**
1. Analyzes audio FFT (entire file)
2. Creates timeline arrays
3. Captures shader code
4. Captures control settings
5. Saves .oms file to /Library/
6. Container appears in ORGANIZE tab

**Timeline analysis:**
- OMS analyzes entire audio file
- Performs FFT at regular intervals
- Extracts frequency levels for each modulator's range
- Stores as timeline arrays in .oms file
- Enables playback in PERFORM tab without re-analysis
- Pixel-perfect reproduction of CREATE tab preview

---

#### 3.2.8 CREATE Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Spacebar | Play/pause audio |
| ← | Navigate one beat left |
| → | Navigate one beat right |
| Shift+< | Jump to previous grid line (4 bars) |
| Shift+> | Jump to next grid line (4 bars) |
| G | Toggle beatgrid |

**Reactivity Mode (V4_8_0):**

| Shortcut | Action |
|----------|--------|
| Shift+R | Enter/exit reactivity mode |
| ← → | Navigate columns (Subdiv/Waveform/Gain/Freq1/Freq2) |
| ↑ ↓ | Navigate rows (A/B/C) |
| Shift+← | Decrease value |
| Shift+→ | Increase value |
| Escape | Exit reactivity mode |

**Note:** Hold Shift+Arrow for progressive acceleration (faster changes).

**Transform Mode (V5_0_0):**

| Shortcut | Action |
|----------|--------|
| Shift+T | Enter/exit transform mode |
| ← → | Navigate controls (Rotation/Scale/Tile/XY/Pivot) |
| Shift+← → | Adjust encoder values |
| Shift+↑↓←→ | Move XY/Pivot pads |
| Shift+Enter×2 | Reset selected to default |
| Escape | Exit transform mode |

**Shift+Trackpad (V5_0_0):**

Both Transform and Reactivity knobs support Shift+scroll:
- Hover over control + Shift + scroll = adjust
- In keyboard mode: select control + Shift + scroll anywhere

---

### 3.3 ORGANIZE

#### 3.3.1 Overview

The ORGANIZE tab manages your container library.

**Requirements:** Application folder must be set (see Section 2.2)

**What you can do:**
- Browse containers
- Load containers into CREATE
- Update existing containers
- Delete containers
- Export containers
- Manage audio files
- Create collections

---

#### 3.3.2 Core Concepts

##### 3.3.2.1 Containers

A **container** is a complete package saved to /Library/ folder.

**Contains:**
- Shader code (WGSL)
- Timing settings (Shader Tempo, modulators)
- Audio reference (filename, BPM, loop points)
- Timeline arrays (pre-analyzed FFT data)
- Control states (all settings)
- Thumbnail (32×18 preview image)

**File format:** `.oms`

**Storage:** `/Library/[name].oms`

**Audio handling:**
- .oms references audio filename
- Audio file stored separately in /Library/
- If audio missing, container loads shader-only

---

##### 3.3.2.2 Projects

A **project** saves PERFORM tab session state.

**Contains:**
- Grid layout (32 cells, which containers loaded where)
- Master settings (tempo, zoom)
- Track states (volume, opacity, mute, solo)
- Selected set filter

**File format:** `.omsp`

**Storage:** `/Library/[projectName].omsp`

**Auto-save:** `default.omsp` (saves after every action)

---

##### 3.3.2.3 Collections

A **collection** bundles multiple containers for distribution.

**Contains:**
- Array of .oms container data
- Set metadata (name, author, description)

**File format:** `.oms-set`

**Storage:** `/Library/[setName].oms-set`

**Use cases:**
- Share curated preset packs
- Backup entire library
- Distribute themed collections

---

#### 3.3.3 Interface Layout

**Two-panel design:**

**Left Panel: Sets** (narrow column)
- [All] - Your containers
- [Presets] - System + user presets

**Right Panel: Containers** (wide column)
- Toolbar (top)
- Container list (below)
- Info drawer (slide-out)

---

#### 3.3.4 Sets Panel

**[All]**
- Containers from /Library/ folder
- Everything you've created

**[All Audio]** (V4_7_0)
- Containers that have audio attached
- Quick filter for audio-enabled content

**[Presets]**
- Built-in system presets
- User presets from /Presets/ folder

**Click set name** to filter right panel

---

#### 3.3.5 Toolbar

**Search Field**
- Filter by: name, BPM, audio, shader code
- Real-time filtering

**Action Buttons:**

| Button | Action | Shortcut |
|--------|--------|----------|
| [L] | Load container | Cmd+L |
| [U] | Update loaded | Cmd+U |
| [E] | Export as ZIP | Cmd+E |
| [+] | Add current state | - |

**View Controls:**

| Button | Action |
|--------|--------|
| [≡] | List view (active) |
| [ℹ] | Toggle info drawer (Cmd+I) |
| [X] | Delete (requires confirmation) |

---

#### 3.3.6 Container List

**Column Headers** (click to sort):

| Column | Content | Sort |
|--------|---------|------|
| NAME | Container name | A→Z or Z→A |
| TEMPO | Audio tempo (was "BPM") | Low→High or High→Low |
| AUDIO | Y/N indicator | Y first or N first |

**Sort arrows:**
- ▼ = descending
- ▲ = ascending

**Container Cards:**

Each shows:
- Shader thumbnail (32×18 preview)
- Container name (click to edit - V4_7_0)
- Tempo value (white text)
- Audio indicator (Y/N)

**Click-to-Edit Names (V4_7_0):**
- Click container name in list
- Edit inline
- Press Enter to save
- Changes saved immediately to disk

**Visual states:**
- Yellow border + tint = Loaded in CREATE
- Cyan border = Selected

**Interaction:**
- Click = select
- Double-click = load
- ↑/↓ arrows = navigate
- Enter = load selected

---

#### 3.3.7 Info Drawer

**Access:** Click [ℹ] or press Cmd+I

**Fields:**

| Field | Type | Content |
|-------|------|---------|
| NAME | Text (editable) | Container name |
| BPM | Read-only | Audio tempo |
| AUDIO | Dropdown | Audio file selection |
| DURATION | Read-only | Total length |
| LOOP | Read-only | In → Out points |
| CREATED | Read-only | Creation date |

**Audio Management:**
- Dropdown shows /Library/ audio files
- Orphan files (unused) listed first
- [Swap] button = change audio
- [Remove] button = delete reference

**Buttons:**

| Button | Action |
|--------|--------|
| [OK] | Apply changes |
| [Cancel] | Discard changes |
| [X] | Close drawer |

**Pending changes:** Edits don't apply until OK clicked

---

#### 3.3.8 Delete Confirmation

**Status bar pattern** (no browser dialog)

**Process:**
1. Select container
2. Press Delete (or click [X])
3. Status: "Press Delete again to confirm removal"
4. Press Delete within 3 seconds
5. Container deletes

**If you wait 3+ seconds:** "Delete cancelled"

---

#### 3.3.9 Drag & Drop

**Drag audio file (.mp3, .wav) onto ORGANIZE tab:**
1. File drops
2. Copies to /Library/
3. Creates container with current shader
4. Appears in list

---

#### 3.3.10 ORGANIZE Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| Cmd+L | Load container |
| Cmd+U | Update loaded |
| Cmd+E | Export container |
| Cmd+I | Toggle info drawer |
| Cmd+S | Swap shader |
| Cmd+Shift+S | Remove shader |
| Cmd+A | Swap audio |
| Cmd+Shift+A | Remove audio |
| ↑ | Navigate up |
| ↓ | Navigate down |
| Enter | Load selected |
| Delete | Delete (with confirm) |

---

### 3.4 PERFORM

#### 3.4.1 Overview

The PERFORM tab is a 4-track DJ mixer for live performance with reactive visuals.

**Requirements:** Application folder must be set

**What you can do:**
- Mix 4 tracks simultaneously
- Launch containers quantized to beat
- Control audio and shader independently
- DJ with live reactive visuals

---

#### 3.4.2 Core Concepts

##### 3.4.2.1 Master Clock

**First Launch:**
- Sets session start time
- All timing references this point
- Tracks sync to master timeline

**How it works:**
- First track that launches sets beat 0
- All subsequent tracks reference this start
- Master Tempo controls playback speed
- Tracks time-stretch to match master

**Emergency Stop:**
- Press Escape
- Stops all tracks
- Resets session clock

**Restart:**
- Next launch starts new session

---

##### 3.4.2.2 Quantization

**All launches snap to next bar boundary:**
- Press Enter → launches at next bar
- Keeps all tracks perfectly synchronized
- No manual timing needed

**How it works:**
1. Calculate current beat position
2. Find next bar (always forward, never backward)
3. Schedule launch at that time
4. Audio starts exactly on the bar

---

##### 3.4.2.3 Time-Stretching

**All tracks play at Master Tempo:**
- Master Tempo = 120 BPM
- Container audio = 124 BPM
- Playback rate = 124 / 120 = 1.033 (plays 3.3% faster)

**Why:**
- Keeps all tracks synchronized
- No tempo drift
- Professional DJ workflow

**Quality:**
- Web Audio API handles time-stretching
- High quality algorithms
- Real-time processing

---

##### 3.4.2.4 Timeline Arrays

**Pre-analyzed FFT data from CREATE tab:**
- Timeline arrays stored in .oms file
- PERFORM reads values by beat position
- No real-time FFT needed
- Identical to CREATE preview (pixel-perfect)

**Performance benefits:**
- Faster (no FFT calculation)
- More reliable (consistent results)
- Lower CPU usage

---

#### 3.4.3 Interface Layout

**From left to right:**

1. **Scene Launch Column** - 8 scene buttons (launch all 4 tracks)
2. **Track 1-4 Columns** - Performance grid (4 tracks × 8 scenes = 32 cells)
3. **Master Column** - Global controls
4. **Waveform Panel** - Visual display (below grid)

---

#### 3.4.4 Performance Grid

**Grid Structure:**
- 4 tracks (columns)
- 8 scenes (rows)
- 32 cells total

**Cell States:**
- Empty = grey
- Loaded = shows container name
- Playing = green highlight
- Selected = cyan border

**Loading Containers:**
- Drag from ORGANIZE tab
- Double-click cell = browse library
- Drop .oms file directly

**Launching:**
- Double-click cell
- Press Enter (selected cell)
- Click scene button (launch all 4)

**Quantization:**
- Launches snap to next bar
- All tracks stay synchronized

---

#### 3.4.5 Track Modules

**Each track (1-4) has two sections:**

**Shader Controls:**
- Opacity knob (0-100%)
- [M] Mute button (purple)
- [S] Solo button (cyan)

**Audio Controls:**
- Volume knob (0-100%)
- [M] Mute button (orange)
- [S] Solo button (green)

**Independent control:**
- Shader and audio mix separately
- Can mute audio, keep visuals
- Can mute shader, keep audio

---

#### 3.4.6 Master Module

**Scene Launch Buttons (1-8)**
- Launch all 4 tracks in scene
- Quantized to next bar

**Shader Controls:**
- [M] Master shader mute
- [S] Master shader solo

**Audio Controls:**
- Master volume knob (60px)
- [M] Master audio mute

**Global Controls:**
- Master Tempo knob (60-200 BPM)
- Zoom knob (waveform scale)

---

#### 3.4.7 Waveform Panel

**Display:**
- 4 tracks stacked vertically
- Blue amplitude bars
- Cyan/magenta beat grid
- Magenta playhead (center)

**Behavior:**
- Bar 1 positions at playhead on launch
- Waveform scrolls left during playback
- Master Grid overlays (toggle with M)

**Overlays:**
- Container name
- Shader thumbnail (32×18)

---

#### 3.4.8 PERFORM Keyboard Shortcuts

**Track Selection (V5_1_0 changed):**

| Shortcut | Action |
|----------|--------|
| Shift+1 | Select track 1 |
| Shift+2 | Select track 2 |
| Shift+3 | Select track 3 |
| Shift+4 | Select track 4 |
| Shift+5 | Select master |
| Tab | Next track |
| Shift+Tab | Previous track |

**Cue Jump (V5_1_0 new):**

| Shortcut | Action |
|----------|--------|
| 1 | Jump to Cue 1 on selected track |
| 2 | Jump to Cue 2 on selected track |
| 3 | Jump to Cue 3 on selected track |

**Why the change:** Cue jumping is urgent during performance (needs fast access). Track selection is a setup task (less urgent). Plain number keys now do the most time-critical action.

**Grid Navigation:**

| Shortcut | Action |
|----------|--------|
| W | Up (previous scene) |
| A | Left (previous track) |
| S | Down (next scene) |
| D | Right (next track) |

**Playback:**

| Shortcut | Action |
|----------|--------|
| Enter | Launch selected cell |
| Space | Global play/pause |
| Esc | Emergency stop all |
| Delete | Clear selected cell |

**Mixer:**

| Shortcut | Action |
|----------|--------|
| ↑ | Volume up (audio) |
| ↓ | Volume down (audio) |
| ← | Opacity down (shader) |
| → | Opacity up (shader) |
| , | Audio mute toggle |
| . | Audio solo toggle |
| Shift+, | Shader mute toggle |
| Shift+. | Shader solo toggle |

**Zoom:**

| Shortcut | Action |
|----------|--------|
| [ | Zoom in |
| ] | Zoom out |

**Visibility:**

| Shortcut | Action |
|----------|--------|
| Shift+M | Master Grid toggle |
| Shift+T | Track Grid toggle |
| Shift+W | Waveform toggle |

#### 3.4.9 Message Bar Feedback (V4_7_0)

The message bar at the bottom of the screen provides instant feedback:

**Color Meanings:**

| Color | Type | Example |
|-------|------|---------|
| Cyan | Help | "Audio Volume (↑/↓)" |
| Orange | State | "Track 1 Shader: 80%" |
| Magenta | Error | "Container not found" |

**How it works:**
- **Hover** any control → cyan help message
- **Press** any key → orange state message  
- **Leave** control → message clears
- **Error** occurs → magenta error message

**Examples:**
- Hover volume knob: "Audio Volume (↑/↓)"
- Press `1`: "Track 1 selected"
- Press `←`: "Track 1 Shader: 78%"
- Press `,`: "Track 1 Audio: Muted (,)"

---

#### 3.4.10 Cue Points in PERFORM (V5_1_0)

Jump to saved cue positions during live performance.

**Visual Display:**
- Orange flags appear on track waveforms
- Same positions saved in CREATE tab
- Flags visible even when track not playing

**Keyboard Jump:**

| Key | Action |
|-----|--------|
| 1 | Jump to Cue 1 |
| 2 | Jump to Cue 2 |
| 3 | Jump to Cue 3 |

**Requirements:**
- Track must be selected (Shift+1-4)
- Container must have cue points (all do by default)

**Behavior:**
- If track playing: Jumps immediately (audio skips to cue position)
- If track stopped: Positions waveform at cue (ready to launch)

**Workflow:**
1. Select track: `Shift+1`
2. Launch container: `Enter`
3. Jump to drop: `2` (cue 2 = bar 17 by default)
4. Jump to breakdown: `3` (cue 3 = bar 33 by default)

---

### 3.5 ADMIN

#### 3.5.1 Overview

The ADMIN tab contains settings, utilities, and application info.

**Three sections:**
- Settings - Application configuration
- Utilities - Tools and batch operations
- About - Credits and license

---

#### 3.5.2 Interface Layout

**Left Navigation (alphabetized):**
- [Application]
- [BumpR StickRs] (V5_1_0)
- [Create]
- [Organize]
- [Perform]
- [Utilities]
- [About]

Click item to view content in right panel.

---

#### 3.5.3 Settings

**Application Folder**

| Control | Function |
|---------|----------|
| [Set Folder] | Select application folder |
| [Change Folder] | Choose different folder |
| [Close Folder] | Disconnect from folder |

**Display Settings**

| Control | Function |
|---------|----------|
| Master Grid | ON/OFF toggle |
| Track Grid | ON/OFF toggle |
| Waveform | ON/OFF toggle |

**Project State**

| Control | Function |
|---------|----------|
| [Save Project] | Save .omsp file |
| [Load Project] | Load .omsp file |
| [Delete Project] | Remove .omsp file |

Auto-saves to `default.omsp` after each action.

---

#### 3.5.4 Utilities

**Batch Operations**

| Operation | Function |
|-----------|----------|
| [Export All] | Export all containers as ZIP |
| [Analyze Audio] | Re-analyze all timeline arrays |
| [Validate Library] | Check for broken references |

**Import/Export**

| Operation | Function |
|-----------|----------|
| [Import Set] | Load .oms-set file |
| [Export Set] | Save selection as .oms-set |

---

#### 3.5.5 About

**Information displayed:**
- Version number
- Release date
- License (GPL-3.0)
- Creator credits
- Technology stack
- GitHub repository link

---

#### 3.5.7 BumpR StickRs (V5_1_0)

Video overlay system for personal branding during live performance. Fire WebM video clips with transparency over your shader canvas.

##### 3.5.7.1 Overview

**What it is:**
- 10-slot video overlay system
- WebM VP9 with alpha channel (transparency)
- Keyboard-triggered during performance
- Per-slot settings for Scale, Timing, Animation, Duration

**Location:** ADMIN → BumpR StickRs

**Folder:** `/BumpRStickRs/` (auto-created on first drop)

##### 3.5.7.2 Setting Up Slots

**Adding Videos:**
1. Go to ADMIN → BumpR StickRs
2. Drag WebM file onto any slot
3. File copies to /BumpRStickRs/ folder
4. Thumbnail generates automatically

**Requirements:**
- WebM format only
- VP9 codec recommended
- Alpha channel for transparency

**Creating WebM with Alpha:**
```bash
ffmpeg -i input.mov -c:v libvpx-vp9 -pix_fmt yuva420p output.webm
```

##### 3.5.7.3 Per-Slot Settings

Each of the 10 slots has 4 independent dropdowns:

**Scale:**

| Option | Result |
|--------|--------|
| 25% | Quarter size |
| 50% | Half size |
| 75% | Three-quarter size |
| 100% | Full size (default) |

**Timing:**

| Option | Behavior |
|--------|----------|
| Instant | Plays immediately (default) |
| Beat | Waits for next beat |
| Bar | Waits for next bar |

*Note: Beat/Bar timing requires audio playback. Tab-aware: PERFORM uses master clock, CREATE uses audio playback.*

**Animate (16 effects):**

| Effect | Duration | Loop | Style |
|--------|----------|------|-------|
| None | - | - | No animation (default) |
| Bounce | 0.5s | ∞ | Pulsing scale |
| Breathe | 4s | ∞ | Slow gentle pulse |
| Crazy | 0.8s | ∞ | Everything at once! |
| Drift | 4s | ∞ | Slow diagonal float |
| Drop | 0.6s | 1× | Fall and bounce |
| Float | 2s | ∞ | Gentle bob up/down |
| Glitch | 0.2s | ∞ | Rapid position jitter |
| Pop | 0.4s | 1× | Scale entrance |
| Pulse | 1s | ∞ | Fading in/out |
| Shake | 0.3s | ∞ | Quick wiggle |
| Spin | 1s | ∞ | Continuous rotation |
| Spiral | 2s | ∞ | Spin while scaling |
| Strobe | 0.1s | ∞ | Rapid flash |
| Wobble | 0.5s | ∞ | Tilting left/right |
| Zoom | 2s | ∞ | Scaling in and out |

**Duration:**

| Option | Behavior |
|--------|----------|
| Original | Play video once (default) |
| Loop | Repeat until stopped/replaced |
| 1/4 bar | Cut after 1/4 bar |
| 1/2 bar | Cut after 1/2 bar |
| 1 bar | Cut after 1 bar |
| 2 bars | Cut after 2 bars |
| 4 bars | Cut after 4 bars |
| 8 bars | Cut after 8 bars |
| 1s | Cut after 1 second |
| 2s | Cut after 2 seconds |
| 3s | Cut after 3 seconds |
| 5s | Cut after 5 seconds |
| 10s | Cut after 10 seconds |

##### 3.5.7.4 Keyboard Mode (Shift+B)

**Entering Mode:**
- Press `Shift+B` to enter BumpR mode
- Slideout panel appears showing all 10 slots
- Message bar shows "BumpR Mode: ↑/↓ nav, Enter/1-0 trigger, Esc exit"

**Triggering Videos:**

| Key | Action |
|-----|--------|
| 1-9 | Trigger slots 1-9 |
| 0 | Trigger slot 10 |
| Enter | Trigger selected slot |

**Navigation:**

| Key | Action |
|-----|--------|
| ↑ | Move selection up |
| ↓ | Move selection down |

**Exiting:**

| Key | Action |
|-----|--------|
| Escape | Exit BumpR mode |
| Shift+B | Toggle mode off |

##### 3.5.7.5 CDJ Cue Behavior

BumpR triggers work like CDJ hot cues:
- **Always fires** - Keys trigger immediately (no toggle)
- **Retrigger replaces** - Pressing same slot restarts video
- **Switch replaces** - Pressing different slot stops current, plays new

This matches professional DJ equipment behavior.

##### 3.5.7.6 Slideout Panel

When BumpR mode is active:
- Panel slides in from right edge of canvas
- Shows all 10 slots with thumbnails
- Empty slots dimmed (not clickable)
- Selected slot has cyan border
- Hover shows slot info in message bar

##### 3.5.7.7 Use Cases

**Logo Hits:**
- Short logo animation on drops
- Set to "Bar" timing + "Pop" animation
- Fire on beat 1 of chorus

**Brand Watermarks:**
- Subtle corner logo
- Set to "Loop" duration + 25% scale
- Fire at start of set

**Transitions:**
- Full-screen wipe or flash
- Set to "Instant" + "1 bar" duration
- Fire during breakdowns

**Hype Moments:**
- Animated text or effects
- Set to "Beat" timing + "Crazy" animation
- Fire on big moments

---

#### 3.5.8 ADMIN Keyboard Shortcuts

**BumpR Mode (V5_1_0):**

| Shortcut | Action |
|----------|--------|
| Shift+B | Enter/exit BumpR mode |
| 1-9, 0 | Trigger slots 1-10 |
| ↑ / ↓ | Navigate slots |
| Enter | Trigger selected |
| Escape | Exit mode |

No other keyboard shortcuts for ADMIN tab.

---

## 4. Support

**@itswessmithyo**  
Social / YouTube

**https://onemanshyo.com**

---

## 5. Credits

**Created by:** Wes Smith / ONEMANSHYO  
**Version:** Beta V5_0_0  
**Date:** December 17, 2025

**Technology:**
- WebGPU / WGSL (shaders)
- Web Audio API (audio processing)
- File System Access API (storage)
- Web MIDI API (controller support)

**License:** GPL-3.0  
Open source, free to use and modify.

**Community:**
- @itswessmithyo (social/YouTube)
- https://onemanshyo.com

---

## APPENDIX A: DJ 101 Foundation

Understanding what DJing is and how it evolved from analog to digital performance.

---

### A.1 The Analog Foundation

#### A.1.1 Basic DJing: Two Turntables and a Mixer

The original DJing setup involves:

**Two Turntables:** Play vinyl records. Each turntable allows the DJ to start, stop, and manipulate a record independently.

**A Mixer:** Combines signals from the two turntables. The mixer typically has volume faders (one per channel), EQ controls (bass, mid, treble), a crossfader for smooth transitions, and a master volume control.

---

#### A.1.2 The Core Skill: Selecting Songs

The most fundamental skill in DJing is selecting the right song at the right time. Before any technical skills matter, a DJ must understand:
- What the crowd responds to
- Building energy throughout a set
- Reading the room
- Music theory and song structure

---

#### A.1.3 Beatmatching and Mixing

Once song selection is mastered, the technical skills come into play:

**Beatmatching:** Adjusting the tempo of the incoming track to match the playing track using the pitch slider.

**Mixing:** Using the crossfader and EQs to smoothly transition between tracks, often swapping bass frequencies between the two tracks.

**Phrasing:** Aligning the musical phrases (typically 8, 16, or 32 bars) so transitions happen at natural musical points.

---

#### A.1.4 The Role of the Microphone

While the turntables handle the music, the microphone adds a live, interactive element. In hip-hop's origins, the microphone was crucial for the MC to hype up block parties. In a club setting, it might be used more for simple announcements. Learning to speak clearly and at the right volume over the music, without feedback, is an additional skill.

Mastering this setup requires consistent practice to train your ear and develop the physical "touch" needed for handling vinyl records.

---

#### A.1.5 The Technics 1200 Era: The Gold Standard

The transition into the era of technical DJing as we know it today was defined by one piece of equipment: the Technics SL-1200 turntable. This period solidified "beatmatching" as the foundational skill of DJing, moving beyond simple track selection into seamless, continuous performance.

**The Technics SL-1200** (and its successor, the 1210) was introduced by Panasonic/Technics and became the industry standard in the 1970s, but it defined DJ culture through the 80s and 90s.

**Direct Drive Motor:** Unlike belt-drive turntables which wobble and slow down when touched, the 1200's powerful direct-drive motor provided consistent, instant torque and speed stability. This was crucial for beatmatching and scratching.

**Pitch Control:** The reliable pitch slider allowed DJs to precisely adjust the playback speed (BPM) of a record by up to ±8%, enabling them to match the tempo of one song to another.

**Durability:** The robust build quality and vibration-dampening base made them perfect for loud, vibrating club environments.

---

#### A.1.6 The Art of Manual Beatmatching

In this analog era, DJing was a purely auditory skill. There were no BPM counters, screens, or sync buttons. The DJ's process was entirely hands-on:

**1. Listening in Headphones (Cueing):** The DJ would use headphones to listen to the next track on the inactive turntable, finding the starting point ("the one" beat).

**2. Using the Pitch Fader:** They would then adjust the pitch slider to match the perceived tempo of the track currently playing through the main speakers.

**3. Nudging and Braking:** The DJ used their hands on the record platter itself—gently slowing it down with a finger (braking) or pushing the center spindle slightly forward (nudging)—while listening intently to ensure the kick drums and snares of both tracks clicked perfectly into phase, creating a single unified rhythm.

**4. The Blend:** Once the two tracks were locked in time ("in the mix"), the DJ would use the mixer's faders to transition smoothly from the outgoing track to the incoming one, focusing on EQ adjustments to keep the mix clean and balanced.

---

#### A.1.7 The Mixers of the Era: 2-Channel Simplicity

Mixers in this era were utilitarian tools, often two-channel (one channel per turntable) and favoring clean sound and reliability:

**Rane TTM Series/DJM Series:** As house music, techno, and hip-hop evolved, specific mixers became standard. The Rane TTM series (especially the TTM 56/57) became popular for scratch DJs due to their durable faders, while Pioneer's DJM series started to add basic effects loops.

**Rotary Mixers (Return):** The clean, uncolored sound and smooth, long-throw rotary knobs of vintage mixers like the UREI 1620 made a resurgence in the techno/house scenes, prioritizing the quality of the sound over aggressive cutting.

This era required significant practice and an acute sense of rhythm to master, establishing the manual dexterity that is still highly respected today as a core DJ skill.

---

### A.2 The DVS Transition

#### A.2.1 DJing in the Digital Era

DJing in the digital era, particularly with a Digital Vinyl System (DVS), combines the classic, tactile feel of traditional hardware (turntables/CDJs) with the power and flexibility of modern DJ software and digital music files.

---

#### A.2.2 The DVS Concept Explained

Instead of music being physically pressed into the grooves of a record, a DVS setup uses special "timecode" vinyl records or CDs. These records contain a continuous, high-pitched control signal (which sounds like a constant tone to the human ear) that is fed into your computer via a dedicated audio interface or a DVS-enabled mixer.

The DJ software interprets this timecode signal to determine the exact speed, direction, and position of the record's playback. It then applies these physical manipulations to a digital audio file (e.g., an MP3 or WAV file) loaded within the software, as if the digital track were physically on the platter. The processed digital audio is then sent back out to the mixer and speakers.

---

#### A.2.3 The Main DJ Software Platforms

When DJing with a DVS, the computer and software become the "brain" of the operation:

**Serato DJ Pro:** Perhaps the most dominant DVS platform, Serato is known for its reliability, low-latency performance, and tight integration with certified hardware.

**Traktor Pro (Native Instruments):** A highly flexible and feature-rich platform favored by many techno and tech-house DJs.

**rekordbox DJ (Pioneer):** Pioneer's own software, tightly integrated with their CDJ and mixer ecosystem.

---

#### A.2.4 Hybrid Setups: CDJs and Media Players

CDJs evolved from playing physical CDs to becoming multi-format digital media players. Modern CDJs (like the Pioneer CDJ-3000) can play from USB drives, SD cards, and stream from rekordbox.

**HID Mode:** When connected via USB to a computer, CDJs can control DJ software directly (HID = Human Interface Device), providing even tighter integration than timecode.

---

### A.3 The USB Era

#### A.3.1 The Standalone Revolution

The USB era represents DJing without a laptop. Modern media players like CDJs and all-in-one units can read music directly from USB drives.

**Preparation:** Music is prepared in software (rekordbox, Engine DJ, Serato) which analyzes BPM, creates waveforms, and sets cue points. This data travels with the files on the USB.

**Performance:** The DJ walks into a club with just USB drives. No laptop, no audio interface, no cables. Plug in and play.

**Benefits:**
- Reliability (no laptop crashes)
- Simplicity (less gear to carry)
- Industry standard (clubs have CDJs)

---

### A.4 The DAW Era

#### A.4.1 Ableton Live as a DJ Platform

Ableton Live revolutionized DJing by bringing DAW (Digital Audio Workstation) capabilities to live performance. Unlike traditional DJ software, Ableton treats clips as building blocks.

**Session View:** A grid of clips (loops, stems, tracks) that can be triggered in any combination.

**Warping:** Ableton's time-stretching algorithm automatically matches any audio to the session tempo.

**Effects and Processing:** Full studio-quality effects chains, sidechaining, and sound design in real-time.

**Hybrid Performance:** Combine DJing with live production—add drums, synths, and original elements on top of existing tracks.

---

#### A.4.2 Why DJs Choose Ableton

**Creative Freedom:** Go beyond playing tracks—remix, layer, and create unique versions live.

**Stem Control:** Separate vocals, drums, bass, and melodic elements for surgical mixing.

**Visual Integration:** Connect to visual software for synchronized audio-visual performance.

**Production to Performance:** Use the same software for making and playing music.

---

### A.5 Controllers

#### A.5.1 Dedicated Ableton Controllers

**1. Akai APC40 / APC40 mkII**

**Integration:** Built specifically for Ableton Live's Session View, providing instant tactile control.

**Key Features:** Grid of backlit, pressure-sensitive pads for clip launching, dedicated faders for mixer control, and assignable encoders for device parameters.

**Performer's Use:** Enables live triggering of clips, real-time effect manipulation, and intuitive navigation of complex Ableton sessions without looking at the screen.

---

**2. Novation Launchpad**

**Integration:** Also primarily designed for Ableton Live's Session View, though it can be MIDI-mapped to other software.

**Key Features:** Compact, portable, affordable grid controller focused on launching clips and navigating the Session View. The pads light up in RGB colors that correspond to clip colors.

**Performer's Use:** Excellent for DJ-hybrid sets where the focus is triggering loops and samples quickly, or as a secondary controller for effects.

---

#### A.5.2 Specialized & Boutique Controllers

**3. MIDI Fighter (by DJ TechTools)**

**Integration:** Highly customizable, rugged controller mappable to any MIDI-compatible software.

**Key Features:** High-quality arcade buttons (Sanwa brand) that are incredibly responsive for technical finger drumming, rapid-fire cue juggling, and scratching routines.

**Performer's Use:** Popular with "controllerists" who specialize in highly technical, visually impressive routines.

---

**4. Yaeltex (Custom & Boutique Controllers)**

**Integration:** Custom-built, boutique hardware solutions built exactly to a performer's needs.

**Key Features:** Complete customization—choose exact layout, fader types, knob count, pad types, size, color, and firmware.

**Performer's Use:** High-end studio setups or touring artists who need unique hardware for their specific live show.

---

### A.6 Visuals

#### A.6.1 VJing: The Visual Performance Art

The world of VJing (Video Jockeying) transforms a musical performance into an immersive audiovisual spectacle. It's the art of curating, creating, and manipulating visual imagery in real-time, synchronized with the music.

---

#### A.6.2 The Role of the VJ

A VJ is analogous to a DJ: where a DJ mixes audio tracks, a VJ mixes visual content. The performance is improvised and responsive, reacting to the energy of the crowd and the flow of the music. The goal is to enhance the mood, create a unique visual identity, and engage the audience on a multi-sensory level.

---

#### A.6.3 Bringing Visuals into the Mix

The VJ setup has evolved from early analog hardware to powerful digital systems:

**Content Library:** VJs use pre-recorded video loops, still images, graphics, and live camera feeds. Many VJs create their own unique content.

**Synchronization:** Visuals are synchronized to the music's BPM and energy levels, either manually or using software features like Ableton Link.

**Real-Time Manipulation:** VJs use software and MIDI controllers to apply effects, transitions, masks, and blending modes in real-time.

**Output and Mapping:** Video signals go to LED screens, projectors, or plasma displays. Advanced VJs use projection mapping to wrap visuals around physical objects or architecture.

---

#### A.6.4 Key VJ Software

**Resolume (Avenue/Arena):** Industry standard with flexible layer-based interface, extensive effects, and projection mapping.

**VDMX:** Highly modular macOS-only platform with deep audio analysis for audio-reactive visuals.

**TouchDesigner:** Node-based procedural platform for generative visuals and interactive systems.

**HeavyM:** User-friendly interface for projection mapping, popular for beginners.

---

#### A.6.5 The DAW as a Visual Engine

Using a DAW for visuals leverages precise timing, MIDI mapping, and audio processing. The core technology is Max for Live (M4L), which allows custom visual tools within Ableton.

**EboSuite:** Max for Live extension that turns Ableton Live into a VJ studio. Treat video clips like audio clips within Session or Arrangement view.

**Max for Live Devices:** Community-created instruments and effects that bridge audio and video, including basic video players, mixers, and generative art tools.

---

**END OF APPENDIX A**

---
---

## APPENDIX B: ONEMANSHYO Origin Story

### B.1 What ONEMANSHYO Is

Wes Smith performs as **ONEMANSHYO** - a solo DJ/visual artist project. The original setup centered around:
- **Ableton Live** (audio engine for DJing)
- **EboSuite** (Max for Live plugin for video mixing in Ableton)
- **MIDI Controller** (Midifighter Twister or Yaeltex for control)

**What ONEMANSHYO Does:**
- DJs audio (mixing tracks, building sets, live performance)
- Runs live reactive visuals simultaneously
- One person controls both audio AND visuals
- Visuals react to audio in real-time (not pre-rendered canned videos)
- **This is DJing with a visual element, NOT VJing**

### B.2 Critical Distinction: DJing vs VJing

**This is NOT VJing:**
- VJing = mixing pre-made video clips (video DJing)
- VJs mix visuals like DJs mix audio
- Separate role from audio DJ

**ONEMANSHYO = DJing WITH Visuals:**
- Primary role: DJ (audio mixing and performance)
- Secondary role: Visual control (shaders react to music)
- One person, integrated workflow
- Audio-first, visuals respond

### B.3 Why OMS Was Created

**Professional Tools Available:**
- Resolume, ShowSync, TouchDesigner, EboSuite, etc.
- All excellent, professional-grade platforms
- Industry standard tools

**But They're Designed For:**
- Broad use cases (installations, theater, A/V productions, etc.)
- Team workflows (VJ + DJ + technical crew + operators)
- Complex multi-purpose pipelines
- Generic video mixing capabilities
- Multiple vendors, multiple products

**The Old Workflow Challenge:**
- Multiple vendor products (Ableton + EboSuite + hardware)
- Static workflow with extensive prep work
- Complex multi-part setup requiring technical knowledge
- Fragmented ecosystem from different vendors
- Expensive software licenses
- Requires assembling pieces that weren't designed to work together

### B.4 The OMS Vision

**OMS Takes a Different Approach:**
- Skip generic pipeline
- Laser-focus on ONE specific workflow: solo DJ + live visuals
- Single-source solution (not assembling pieces)
- Web-native using modern technology (WebGPU, Web Audio, Web MIDI)
- Integrated ecosystem (all parts designed to work together)
- Open source (GPL-3.0) - freely accessible to anyone
- No installation required - runs in browser

OMS isn't a theoretical experiment or academic project. It's **codifying Wes's proven ONEMANSHYO workflow** (years of real performances) into a unified, browser-based solution purpose-built for solo DJ + live visuals.

**Not Replacing Professional Platforms:**
- Not competing with Resolume or TouchDesigner
- Creating specialized tool for this specific use case
- Making proven workflow accessible to anyone

**"Anyone can be a OneManShYo" means:**
- Anyone can DJ with live reactive visuals
- No expensive software licenses required
- No complex multi-vendor setup needed
- No team or crew needed
- Proven workflow, accessible tools, free and open

**END OF APPENDIX B**


## APPENDIX C: Shader Creation Guide

**Purpose:** Everything needed to create WGSL shaders for OMS  
**Usage:** Drop this entire APPENDIX into Claude to enable shader creation

---

### C.1 Technical Reference

*For humans: the must-know contract, syntax, and troubleshooting*

---

#### C.1.1 Technical Contract (REQUIRED)

**Language:** WGSL (WebGPU Shading Language) - NOT GLSL

**Function Signature (exact):**
```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    // Your code here
    return vec4<f32>(r, g, b, 1.0);
}
```

**CRITICAL RULES:**
- Function MUST be named `fs_main` (not `main`)
- Struct is AUTO-INJECTED by app - do NOT include `struct Uniforms`
- Do NOT include `@group(0) @binding(0)` - app handles this
- Return alpha MUST be 1.0
- Use `vec4<f32>` not `vec4f` (explicit types)

**Shader Header Template (REQUIRED):**

Every shader file MUST include these comment tags at the top:
```wgsl
// @name [Display Name - no version]
// @app ONEMANSHYO
// @appVersion BETA V5_0_0
// @url http://www.itswessmithyo.com
// @credit [Inspiration source or empty]
// @creditUrl [URL or empty]
// @desc [One-line description]
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
```

- `@name` is parsed by app for dropdown display
- Keep all fields even if empty
- Update `@appVersion` to match current app version


---

#### C.1.2 Available Uniforms

The app auto-injects these. Just use them directly:

**Core Uniforms:**

| Uniform | Type | Range | Purpose |
|---------|------|-------|---------|
| `uniforms.time` | f32 | 0.0+ | Elapsed seconds |
| `uniforms.tempo` | f32 | 60-200 | BPM for timing |
| `uniforms.beat` | f32 | 0.0+ | Tempo-synced beat position (V4_8_0) |
| `uniforms.modA` | f32 | 0.0-1.0 | Bass reactive (combined) |
| `uniforms.modB` | f32 | 0.0-1.0 | Mids reactive (combined) |
| `uniforms.modC` | f32 | 0.0-1.0 | Treble reactive (combined) |
| `uniforms.resolutionX` | f32 | pixels | Canvas width |
| `uniforms.resolutionY` | f32 | pixels | Canvas height |

**Independent Modulator Components (V4_8_0):**

| Uniform | Type | Range | Purpose |
|---------|------|-------|---------|
| `uniforms.shaderModAWave` | f32 | 0.0-1.0 | Mod A waveform output |
| `uniforms.shaderModAGain` | f32 | 0.0-1.0 | Mod A gain value |
| `uniforms.shaderModAFFT` | f32 | 0.0-1.0 | Mod A FFT level (0 if no audio) |
| `uniforms.shaderModBWave` | f32 | 0.0-1.0 | Mod B waveform output |
| `uniforms.shaderModBGain` | f32 | 0.0-1.0 | Mod B gain value |
| `uniforms.shaderModBFFT` | f32 | 0.0-1.0 | Mod B FFT level |
| `uniforms.shaderModCWave` | f32 | 0.0-1.0 | Mod C waveform output |
| `uniforms.shaderModCGain` | f32 | 0.0-1.0 | Mod C gain value |
| `uniforms.shaderModCFFT` | f32 | 0.0-1.0 | Mod C FFT level |

**Legacy vs Independent:**
- `modA/B/C` = combined value (Wave × Gain × FFT) - backward compatible
- Independent uniforms let you access each component separately for custom combinations

**Example - using independent uniforms:**
```wgsl
// Custom combination: wave controls size, FFT controls color
let size = 0.3 + uniforms.shaderModAWave * 0.2;
let colorIntensity = uniforms.shaderModAFFT;
```

**Transform Uniforms (V5_0_0):**

| Uniform | Type | Range | Purpose |
|---------|------|-------|---------|
| `uniforms.transformRotation` | f32 | radians | Rotation angle |
| `uniforms.transformScaleX` | f32 | 0.1-4.0 | Horizontal scale |
| `uniforms.transformScaleY` | f32 | 0.1-4.0 | Vertical scale |
| `uniforms.transformTranslateX` | f32 | -1.0 to 1.0 | Horizontal offset |
| `uniforms.transformTranslateY` | f32 | -1.0 to 1.0 | Vertical offset |
| `uniforms.transformPivotX` | f32 | -1.0 to 1.0 | Pivot X position |
| `uniforms.transformPivotY` | f32 | -1.0 to 1.0 | Pivot Y position |
| `uniforms.transformTile` | f32 | 1-8 | Tile repeat count |

**Note:** Transform uniforms reflect UI control values. Shaders can read these to create transform-aware effects.

---

#### C.1.3 Modulator Behavior

User controls each modulator (A/B/C) via dropdown in the app:

| Mode | Output | Use Case |
|------|--------|----------|
| **None** | Always 0 | Modulator disabled |
| **Static** | Gain value directly (0-1) | Manual control, no animation |
| **Waveform** | Oscillating × Gain | Animated (sine, square, etc.) |

**Design for this:** User may set any modulator to None/Static/Waveform. Shader should handle modA/B/C being 0.

**Example - handling zero gracefully:**
```wgsl
// Safe: works when modA is 0
let size = 0.3 + uniforms.modA * 0.2;

// Dangerous: division by modA
let bad = 1.0 / uniforms.modA;  // Crashes when modA = 0!
```

---

#### C.1.4 Standard Setup Template

```wgsl
// Shader Name - Description
// uniforms: time, tempo, modA/B/C (0-5), resolutionX/Y

@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = coord.xy / resolution;                    // 0.0 to 1.0
    let centered = uv - 0.5;                           // -0.5 to 0.5
    let aspect = resolution.x / resolution.y;
    let p = vec2<f32>(centered.x * aspect, centered.y); // Aspect-corrected
    
    // Your effect code here
    var color = vec3<f32>(0.0);
    
    return vec4<f32>(color, 1.0);
}
```

---

#### C.1.5 WGSL vs GLSL Differences

| GLSL (Shadertoy) | WGSL (OMS) |
|------------------|------------|
| `vec2`, `vec3`, `vec4` | `vec2<f32>`, `vec3<f32>`, `vec4<f32>` |
| `iTime` | `uniforms.time` |
| `iResolution` | `vec2<f32>(uniforms.resolutionX, uniforms.resolutionY)` |
| `fragCoord` | `coord.xy` (from function parameter) |
| `void main()` | `fn fs_main(...) -> @location(0) vec4<f32>` |
| `gl_FragColor = ` | `return vec4<f32>(...)` |
| `mod(x, y)` | `x % y` or `fract(x/y)*y` |
| `mix(a, b, t)` | `mix(a, b, t)` ✓ same |
| `texture2D()` | Not available (no textures) |

---

#### C.1.6 Troubleshooting

**"Failed to compile shader"**
- Check function is named `fs_main` not `main`
- Don't include struct Uniforms (app injects it)
- Use `vec4<f32>` not `vec4f`
- Check all parentheses match

**Black output**
- Is color calculation producing values > 0?
- Return alpha must be 1.0
- Test with: `return vec4<f32>(1.0, 0.0, 0.0, 1.0);` (should show red)

**No audio reactivity**
- Are you using `uniforms.modA/B/C`?
- Is audio loaded in CREATE tab?
- Is modulator mode set to something other than None?

---

#### C.1.7 Quick Reference

```wgsl
// Resolution
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);

// Normalized UV (0-1)
let uv = coord.xy / resolution;

// Centered (-0.5 to 0.5)
let centered = uv - 0.5;

// Aspect-corrected
let aspect = resolution.x / resolution.y;
let p = vec2<f32>(centered.x * aspect, centered.y);

// Distance from center
let dist = length(p);

// Audio reactive (already 0-1, no normalization needed)
let bass = uniforms.modA;
let mids = uniforms.modB;
let treble = uniforms.modC;

// Beat-synced animation (V4_8_0)
let pulse = fract(uniforms.beat);        // 0→1 each beat
let barPulse = fract(uniforms.beat / 4.0); // 0→1 each bar

// Transform values (V5_0_0) - read from UI controls
let rotation = uniforms.transformRotation;  // radians
let scale = vec2<f32>(uniforms.transformScaleX, uniforms.transformScaleY);
let translate = vec2<f32>(uniforms.transformTranslateX, uniforms.transformTranslateY);
let tile = uniforms.transformTile;  // 1-8

// Always return with alpha 1.0
return vec4<f32>(color, 1.0);
```


---

#### C.1.8 Standard Transform Chain (V5_0_0)

When implementing transforms in your shader, use this standard chain:

```wgsl
// === FULL TRANSFORM CHAIN ===
let minDim = min(resolution.x, resolution.y);
var uv = (coord.xy - resolution * 0.5) / minDim * 2.0;

// Apply pivot offset
uv.x -= uniforms.transformPivotX;
uv.y += uniforms.transformPivotY;

// Rotate around pivot
let cosR = cos(uniforms.transformRotation);
let sinR = sin(uniforms.transformRotation);
uv = vec2<f32>(uv.x * cosR - uv.y * sinR, uv.x * sinR + uv.y * cosR);

// Remove pivot, apply translation
uv.x += uniforms.transformPivotX - uniforms.transformTranslateX;
uv.y -= uniforms.transformPivotY - uniforms.transformTranslateY;

// Scale (both axes uniform from Scale control)
let scale = max(uniforms.transformScaleX, 0.01);
uv = uv / scale;

// Tile
let tile = max(uniforms.transformTile, 1.0);
uv = fract(uv * tile * 0.5 + 0.5) * 2.0 - 1.0;
```

**Order matters:** Pivot → Rotate → Translate → Scale → Tile

**Notes:**
- Rotation is in radians (UI shows degrees, shader receives radians)
- Scale is uniform (X and Y always equal from the single Scale control)
- Tile wraps UV space for seamless repetition
- Transform values come directly from CREATE tab controls

---

### C.2 Development Workflow

*For Claude: iterative shader building patterns and mappings*

---

#### C.2.1 Modulator Mapping Guidelines

| Modulator | Frequency | Character | Best For |
|-----------|-----------|-----------|----------|
| `uniforms.modA` | Bass | Punchy, rhythmic | Size, pulse, impacts, zoom |
| `uniforms.modB` | Mids | Melodic, sustained | Rotation, color shift, movement |
| `uniforms.modC` | Treble | Crisp, transient | Sparkle, edges, fine detail |

**Recommended patterns:**

```wgsl
// modA (Bass) → Size/Impact (values already 0-1)
let size = 0.3 + uniforms.modA * 0.3;
let zoom = 1.0 + uniforms.modA * 0.4;

// modB (Mids) → Movement/Color
let rotation = uniforms.time * (1.0 + uniforms.modB);
let hue = uniforms.modB * 0.5;

// modC (Treble) → Detail/Sparkle
let sparkle = step(0.97, noise) * uniforms.modC;
let edge = smoothstep(0.5 - uniforms.modC * 0.2, 0.5, pattern);
```

---

#### C.2.2 Working Example

```wgsl
// Pulsing Rings - Bass=size, Mids=color, Treble=sparkle

@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = (coord.xy - resolution * 0.5) / min(resolution.x, resolution.y);
    
    let dist = length(uv);
    let rings = sin(dist * 20.0 - uniforms.time * 3.0) * 0.5 + 0.5;
    
    // Bass pulses ring intensity (modA is 0-1)
    let intensity = rings * (0.3 + uniforms.modA * 0.7);
    
    // Mids shift color
    let hue = uniforms.modB * 0.4;
    
    // Treble adds sparkle
    let sparkle = fract(sin(dot(uv, vec2<f32>(12.9898, 78.233)) + uniforms.time) * 43758.5453);
    let sparkleAmount = step(0.97, sparkle) * uniforms.modC * 0.4;
    
    var color = vec3<f32>(intensity * (1.0 - hue), intensity * hue, intensity * 0.5);
    color = color + vec3<f32>(sparkleAmount);
    
    return vec4<f32>(color, 1.0);
}
```

---

#### C.2.3 Shader Development Process

**Recommended approach when creating shaders:**

1. **Start with None** - Set all modulators to None
   - Get the basic visual working first
   - No animation, just the pattern

2. **Add Static control** - Switch one modulator to Static
   - Use gain slider to test parameter ranges
   - Verify the effect looks good at min/max values

3. **Add Waveforms** - Switch to animated waveforms
   - Choose appropriate waveform type
   - Set subdivision for timing

4. **Test edge cases:**
   - All modulators at 0
   - All modulators at 5
   - Mixed values

---

#### C.2.4 Common Patterns

**Distance from center:**
```wgsl
let dist = length(p);
```

**Angle from center:**
```wgsl
let angle = atan2(p.y, p.x);
```

**Circle:**
```wgsl
let circle = smoothstep(radius + 0.01, radius, dist);
```

**Sine wave:**
```wgsl
let wave = sin(value * freq + uniforms.time * speed) * 0.5 + 0.5;
```

**Tempo-synced pulse:**
```wgsl
let beat = uniforms.time * uniforms.tempo / 60.0;
let pulse = sin(beat * 3.14159 * 2.0) * 0.5 + 0.5;
```

**Simple noise:**
```wgsl
let noise = fract(sin(dot(uv, vec2<f32>(12.9898, 78.233))) * 43758.5453);
```

---

### C.3 Recipes

*Real examples showing the iterative flow in action*

---

#### C.3.1 Recipe #1: "No AI Problema"

*How to work with Claude to develop shaders iteratively*

---

##### STEP 1: THE RECIPE

**USER:** I want a shader inspired by Art of the Problem YouTube channel. Like a bicycle wheel with squiggly spokes. The concept:
- **White** = all the world's knowledge
- **Green** = humans reaching out with curiosity  
- **Purple** = AI extending knowledge (but not questioning)

**CLAUDE:** *[asks about distribution pattern, what modA should control]*

---

##### STEP 2: FIRST LAYER (WHITE)

**USER:** Golden angle distribution, modA controls strand count 144 to 244

**CLAUDE:** *[delivers static white squiggly spokes]*

---

##### STEP 3: SECOND LAYER (GREEN)

**USER:** Add green spokes, same curved style. modB controls count 12 to 144

**CLAUDE:** *[delivers green on top of white]*

---

##### STEP 4: THIRD LAYER (PURPLE)

**USER:** Add purple on modC from 0 to 48

**CLAUDE:** *[delivers three layers: modA=White, modB=Green, modC=Purple]*

---

##### STEP 5: ADD MOTION

**USER:** Ideas to make it cooler?

**CLAUDE:** *[offers options: rotation, pulsing, traveling waves, glow...]*

**USER:** No pulsing, annoying. Focus on squiggles.

**CLAUDE:** *[delivers squiggles animating outward along spokes]*

---

##### STEP 6: FILL THE CANVAS

**USER:** The shader is round in the canvas. I'd like it to fill the canvas.

**CLAUDE:** *[offers: extend radius, remove outer limit, scale coordinates]*

**USER:** Go forever - no outer limit.

**CLAUDE:** *[delivers strands extending to canvas edges]*

---

**END OF APPENDIX C**

---

## APPENDIX D: FFmpeg Video Conversion

### D.1 Overview

OMS Beta exports WebM format. Use FFmpeg to convert for different uses.

**FFmpeg Installation (macOS):**
```bash
brew install ffmpeg
```

---

### D.2 Common Conversions

#### D.2.1 H.264 (Social Media)

**Standard quality:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k output.mp4
```

**High quality:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset slow -crf 18 -c:a aac -b:a 192k output.mp4
```

**Instagram/TikTok:**
```bash
ffmpeg -i input.webm -c:v libx264 -preset fast -crf 23 -maxrate 5M -bufsize 10M -c:a aac -b:a 128k output.mp4
```

---

#### D.2.2 ProRes (Professional)

**ProRes 422:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 -pix_fmt yuv422p10le output.mov
```

**ProRes 422 HQ:**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 4 -pix_fmt yuv422p10le output.mov
```

**Use cases:**
- Ableton Live + EboSuite
- Video editing (Final Cut Pro, Premiere)
- Professional playback systems

---

#### D.2.3 H.265 (Archive)

**Standard:**
```bash
ffmpeg -i input.webm -c:v libx265 -preset medium -crf 28 -c:a aac -b:a 128k output.mp4
```

**High quality:**
```bash
ffmpeg -i input.webm -c:v libx265 -preset slow -crf 23 -c:a aac -b:a 192k output.mp4
```

**Benefits:**
- 50% smaller files than H.264
- Same visual quality
- Better for archiving

---

### D.3 Batch Conversion

**Bash Script (macOS):**

```bash
#!/bin/bash
# Convert all WebM files to H.264

for file in *.webm; do
    output="${file%.webm}.mp4"
    ffmpeg -i "$file" -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k "$output"
done
```

**Usage:**
1. Save as `convert.sh`
2. Make executable: `chmod +x convert.sh`
3. Run in folder with WebM files: `./convert.sh`

---

### D.4 Advanced Options

#### D.4.1 Add Audio Track

OMS exports silent video. To add audio:

```bash
ffmpeg -i video.webm -i audio.mp3 -c:v copy -c:a aac -b:a 192k output.mp4
```

#### D.4.2 Loop Video

Create 10-minute loop from short video:

```bash
ffmpeg -stream_loop 10 -i input.webm -c copy output.webm
```

#### D.4.3 Resize

Scale to 720p:

```bash
ffmpeg -i input.webm -vf scale=1280:720 -c:v libx264 -crf 23 output.mp4
```

---

### D.5 Quick Reference

| Use Case | Command |
|----------|---------|
| Social media | `-c:v libx264 -crf 23` |
| Professional | `-c:v prores_ks -profile:v 3` |
| Archive | `-c:v libx265 -crf 23` |
| Add audio | `-i video -i audio -c:v copy` |

---

**END OF APPENDIX D**

---

## INDEX A: Visual Vocabulary

A reference guide for shader effects, patterns, and terminology used in OMS.

**Note:** All code examples use the OMS shader contract from APPENDIX C. Function must be `fs_main`, struct is auto-injected.

---

### E.1 Fundamental Shapes

Basic geometric primitives that form the foundation of shader effects.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Circle** | Round shape from center | `length(uv - center) < radius` |
| **Ring** | Hollow circle | `abs(length(uv) - radius) < thickness` |
| **Rectangle** | Box shape | `step()` on x and y bounds |
| **Line** | Thin stripe | `abs(uv.x - position) < thickness` |
| **Triangle** | Three-sided polygon | Barycentric coordinates or SDF |
| **Polygon** | N-sided shape | Angle-based segmentation |
| **Grid** | Repeating cells | `fract(uv * count)` |
| **Dot Pattern** | Array of circles | Grid + circle per cell |

#### WGSL Examples

**Circle:**
```wgsl
let dist = length(p);
let circle = smoothstep(0.31, 0.3, dist);  // Soft edge
```

**Ring:**
```wgsl
let dist = length(p);
let ring = smoothstep(0.31, 0.3, dist) - smoothstep(0.29, 0.28, dist);
```

**Grid:**
```wgsl
let grid = fract(uv * 10.0);  // 10x10 grid
let cell = step(0.1, grid.x) * step(0.1, grid.y);  // Grid lines
```

---

### E.2 Motion Types

Ways that visual elements move or transform over time.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Rotation** | Spinning around center | `mat2x2(cos, -sin, sin, cos)` |
| **Translation** | Moving position | `uv + vec2(offset)` |
| **Scale/Zoom** | Size change | `uv * scale` or `uv / zoom` |
| **Oscillation** | Back-and-forth motion | `sin(time)` |
| **Orbit** | Circular path movement | `vec2(cos(t), sin(t)) * radius` |
| **Drift** | Slow continuous movement | `uv + time * speed` |
| **Pulse** | Rhythmic size change | `1.0 + sin(time) * amount` |
| **Wave** | Undulating distortion | `sin(uv.x * freq + time)` |
| **Spiral** | Rotation + radial motion | `angle + dist * twist` |

#### WGSL Examples

**Rotation:**
```wgsl
let angle = uniforms.time;
let c = cos(angle);
let s = sin(angle);
let rotated = vec2<f32>(p.x * c - p.y * s, p.x * s + p.y * c);
```

**Pulse:**
```wgsl
let pulse = 1.0 + sin(uniforms.time * 4.0) * 0.2;
let size = baseSize * pulse;
```

**Spiral:**
```wgsl
let dist = length(p);
let angle = atan2(p.y, p.x);
let spiral = sin(angle * 5.0 + dist * 10.0 - uniforms.time * 2.0);
```

---

### E.3 Color Operations

Methods for generating and manipulating colors.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Gradient** | Smooth color transition | `mix(colorA, colorB, factor)` |
| **HSV/HSL** | Hue-based color model | Convert functions |
| **Rainbow** | Full spectrum cycle | `hsv2rgb(vec3(time, 1, 1))` |
| **Palette** | Predefined color set | Index into color array |
| **Invert** | Negative colors | `1.0 - color` |
| **Contrast** | Intensity difference | `(color - 0.5) * contrast + 0.5` |
| **Saturation** | Color intensity | HSV manipulation |
| **Brightness** | Light level | `color * brightness` |
| **Hue Shift** | Color rotation | Add to H in HSV |

#### WGSL Examples

**Gradient:**
```wgsl
let gradient = mix(vec3<f32>(1.0, 0.0, 0.0), vec3<f32>(0.0, 0.0, 1.0), uv.x);
```

**HSV to RGB:**
```wgsl
fn hsv2rgb(c: vec3<f32>) -> vec3<f32> {
    let p = abs(fract(vec3<f32>(c.x + 1.0, c.x + 2.0/3.0, c.x + 1.0/3.0)) * 6.0 - 3.0);
    return c.z * mix(vec3<f32>(1.0), clamp(p - 1.0, vec3<f32>(0.0), vec3<f32>(1.0)), c.y);
}
```

**Rainbow:**
```wgsl
let hue = fract(uniforms.time * 0.1);
let color = hsv2rgb(vec3<f32>(hue, 1.0, 1.0));
```

---

### E.4 Pattern Generation

Techniques for creating repeating or structured visual patterns.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Tiling** | Repeating pattern | `fract(uv * count)` |
| **Checkerboard** | Alternating squares | `mod(floor(x) + floor(y), 2)` |
| **Stripes** | Parallel lines | `sin(uv.x * freq)` or `step` |
| **Waves** | Sinusoidal pattern | `sin(uv.x * freq + phase)` |
| **Brick** | Offset tiling | Offset every other row |
| **Hexagonal** | Hex grid pattern | Complex UV transformation |
| **Voronoi** | Cell-based pattern | Nearest point distance |
| **Truchet** | Rotated tile variations | Random rotation per tile |

#### WGSL Examples

**Checkerboard:**
```wgsl
let check = (floor(uv.x * 8.0) + floor(uv.y * 8.0)) % 2.0;
```

**Stripes:**
```wgsl
let stripes = sin(uv.x * 30.0 + uniforms.time * 2.0) * 0.5 + 0.5;
```

---

### E.5 Noise & Randomness

Methods for introducing controlled chaos and organic textures.

| Term | Description | Core Math |
|------|-------------|-----------|
| **White Noise** | Pure random | `fract(sin(dot(uv, vec2)) * 43758.5453)` |
| **Value Noise** | Smooth random | Interpolated grid noise |
| **Perlin Noise** | Gradient noise | Dot products + interpolation |
| **Simplex Noise** | Optimized Perlin | Triangular grid |
| **FBM** | Fractal Brownian Motion | Layered noise octaves |
| **Turbulence** | Absolute FBM | `abs(noise)` layers |
| **Worley/Cellular** | Cell-based noise | Distance to random points |

#### WGSL Examples

**Simple Hash (White Noise):**
```wgsl
fn hash(p: vec2<f32>) -> f32 {
    return fract(sin(dot(p, vec2<f32>(12.9898, 78.233))) * 43758.5453);
}
```

**Value Noise:**
```wgsl
fn valueNoise(p: vec2<f32>) -> f32 {
    let i = floor(p);
    let f = fract(p);
    let u = f * f * (3.0 - 2.0 * f);  // Smoothstep
    return mix(
        mix(hash(i), hash(i + vec2<f32>(1.0, 0.0)), u.x),
        mix(hash(i + vec2<f32>(0.0, 1.0)), hash(i + vec2<f32>(1.0, 1.0)), u.x),
        u.y
    );
}
```

---

### E.6 Distortion Effects

Techniques that warp or bend the visual space.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Ripple** | Concentric wave distortion | `sin(length(uv) * freq)` offset |
| **Twist** | Rotational distortion | Angle varies with distance |
| **Bulge** | Center magnification | UV scaled by distance |
| **Pinch** | Center compression | Inverse bulge |
| **Wave Distort** | Sinusoidal UV offset | `uv.x + sin(uv.y * freq)` |
| **Noise Distort** | Random UV offset | `uv + noise(uv) * amount` |
| **Barrel** | Lens distortion | Radial UV scaling |
| **Fisheye** | Extreme wide-angle | Spherical projection |

#### WGSL Examples

**Ripple:**
```wgsl
let dist = length(p);
let ripple = sin(dist * 20.0 - uniforms.time * 5.0) * 0.02;
let distorted = p + normalize(p) * ripple;
```

**Twist:**
```wgsl
let dist = length(p);
let twist = dist * 3.0 + uniforms.time;
let c = cos(twist);
let s = sin(twist);
let twisted = vec2<f32>(p.x * c - p.y * s, p.x * s + p.y * c);
```

---

### E.7 Blend Modes

Methods for combining multiple visual layers.

| Term | Description | Formula |
|------|-------------|---------|
| **Add** | Brightening blend | `a + b` |
| **Multiply** | Darkening blend | `a * b` |
| **Screen** | Inverse multiply | `1 - (1-a) * (1-b)` |
| **Overlay** | Contrast blend | Multiply or screen based on base |
| **Difference** | Absolute difference | `abs(a - b)` |
| **Mix/Lerp** | Linear interpolation | `a * (1-t) + b * t` |
| **Max** | Lighten | `max(a, b)` |
| **Min** | Darken | `min(a, b)` |

#### WGSL Examples

**Screen:**
```wgsl
let screened = 1.0 - (1.0 - colorA) * (1.0 - colorB);
```

**Overlay:**
```wgsl
fn overlay(base: f32, blend: f32) -> f32 {
    if (base < 0.5) {
        return 2.0 * base * blend;
    } else {
        return 1.0 - 2.0 * (1.0 - base) * (1.0 - blend);
    }
}
```

---

### E.8 Classic Effects

Well-known visual effects commonly used in shader art.

| Term | Description |
|------|-------------|
| **Tunnel** | Infinite depth illusion using polar coords |
| **Kaleidoscope** | Mirrored angular segments |
| **Plasma** | Colorful interference patterns |
| **Fire** | Animated flame simulation |
| **Water** | Rippling reflection effect |
| **Starfield** | Parallax star layers |
| **Matrix Rain** | Falling character columns |
| **Metaballs** | Organic blob merging |

#### WGSL Examples

**Tunnel:**
```wgsl
let dist = length(p);
let angle = atan2(p.y, p.x);
let tunnel = fract(1.0 / dist - uniforms.time);
let stripes = sin(angle * 8.0) * 0.5 + 0.5;
let color = vec3<f32>(tunnel * stripes);
```

**Kaleidoscope:**
```wgsl
let angle = atan2(p.y, p.x);
let segments = 6.0;
let kaleidoAngle = abs(((angle / 3.14159) * segments) % 2.0 - 1.0);
```

**Plasma:**
```wgsl
let plasma = sin(p.x * 10.0 + uniforms.time)
           + sin(p.y * 10.0 + uniforms.time * 1.1)
           + sin((p.x + p.y) * 10.0 + uniforms.time * 0.5)
           + sin(length(p) * 10.0);
let color = hsv2rgb(vec3<f32>(plasma * 0.1, 1.0, 1.0));
```

---

### E.9 Post-Processing

Effects typically applied to the final image.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Vignette** | Edge darkening | `1.0 - length(uv - 0.5)` |
| **Glow/Bloom** | Bright area expansion | Blur + add |
| **Film Grain** | Noise overlay | Random per pixel per frame |
| **Scanlines** | CRT-style lines | `sin(y * freq)` |
| **Chromatic Aberration** | Color channel offset | Separate RGB samples |
| **Pixelate** | Low resolution | `floor(uv * pixels) / pixels` |
| **CRT Effect** | Old monitor look | Barrel + scanlines + glow |

#### WGSL Examples

**Vignette:**
```wgsl
let vignette = 1.0 - smoothstep(0.3, 0.7, length(uv - 0.5));
color = color * vignette;
```

**Film Grain:**
```wgsl
let grain = hash(uv * 1000.0 + uniforms.time * 100.0) * 0.1;
let color = baseColor + vec3<f32>(grain - 0.05);
```

**Scanlines:**
```wgsl
let scanline = sin(coord.y * 3.14159) * 0.5 + 0.5;
let color = baseColor * (0.8 + scanline * 0.2);
```

---

### E.10 Depth/3D Illusions

Techniques that create the impression of three-dimensional space.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Parallax** | Layered depth motion | Multiple layers, different speeds |
| **Raymarching** | 3D SDF rendering | Distance field stepping |
| **Fog** | Distance-based fade | `exp(-dist * density)` |
| **Shadow** | Darkened regions | Secondary ray or fake |
| **Reflection** | Mirror effect | Flipped coordinates |

#### WGSL Examples

**Parallax Layers:**
```wgsl
let layer1 = pattern(uv + vec2<f32>(uniforms.time * 0.1, 0.0));  // slow
let layer2 = pattern(uv + vec2<f32>(uniforms.time * 0.3, 0.0));  // medium
let layer3 = pattern(uv + vec2<f32>(uniforms.time * 0.5, 0.0));  // fast
let color = mix(mix(layer1, layer2, 0.5), layer3, 0.5);
```

**Fog:**
```wgsl
let fogDensity = 0.1;
let fogColor = vec3<f32>(0.5, 0.6, 0.7);
let fog = 1.0 - exp(-distance * fogDensity);
let color = mix(objectColor, fogColor, fog);
```

---

### E.11 Audio-Reactive Mappings

Best practices for connecting audio analysis to visual parameters.

#### Modulator Characteristics

| Modulator | Frequency Range | Character | Best For |
|-----------|-----------------|-----------|----------|
| `uniforms.modA` | 30-500 Hz (Bass) | Punchy, rhythmic | Big movements, main pulse, impacts |
| `uniforms.modB` | 500-2000 Hz (Mids) | Melodic, sustained | Color shifts, rotation, secondary motion |
| `uniforms.modC` | 2000-20000 Hz (Treble) | Crisp, transient | Fine detail, shimmer, edge effects |

#### Recommended Mappings

**uniforms.modA (Bass) → Structure**
```wgsl
// Size/scale (modA is 0-1)
let size = 0.5 + uniforms.modA * 0.3;

// Brightness flash
let brightness = baseColor + vec3<f32>(uniforms.modA * 0.5);

// Zoom punch
let zoom = 1.0 + uniforms.modA * 0.4;
let punchedUV = (uv - 0.5) / zoom + 0.5;

// Ring expansion (shockwave)
let ringRadius = uniforms.modA * 0.5;
```

**uniforms.modB (Mids) → Movement**
```wgsl
// Rotation speed
let angle = uniforms.time * (1.0 + uniforms.modB * 2.0);

// Hue shift
let hue = fract(baseHue + uniforms.modB * 0.2);

// Wave amplitude
let wave = sin(uv.x * 10.0) * uniforms.modB * 0.2;
```

**uniforms.modC (Treble) → Detail**
```wgsl
// Noise intensity
let noise = valueNoise(uv * 10.0) * uniforms.modC;

// Edge sharpness
let edge = smoothstep(0.5 - uniforms.modC * 0.1, 0.5, pattern);

// Flicker/sparkle
let sparkle = step(0.95, hash(uv + uniforms.time)) * uniforms.modC;
```

#### Compound Effects

**All Three Combined:**
```wgsl
// Bass controls overall intensity (modA is 0-1)
let intensity = 0.5 + uniforms.modA * 0.5;

// Mids control color
let hue = fract(uniforms.time * 0.1 + uniforms.modB * 0.3);
let color = hsv2rgb(vec3<f32>(hue, 0.8, intensity));

// Treble adds sparkle
let sparkle = step(0.98, hash(uv * 500.0 + uniforms.time)) * uniforms.modC;
let finalColor = color + vec3<f32>(sparkle);
```

---

### E.12 Combining Patterns

Strategies for layering and mixing effects.

#### Layering Techniques

| Technique | Method | Use Case |
|-----------|--------|----------|
| **Additive** | `color1 + color2` | Glows, light effects |
| **Multiply** | `color1 * color2` | Shadows, darkening |
| **Screen** | `1 - (1-a)*(1-b)` | Bright highlights |
| **Mix** | `mix(a, b, factor)` | Smooth transitions |
| **Max/Min** | `max(a, b)` | Hard unions |
| **Mask** | `a * mask` | Selective reveal |

#### Example: Multi-Layer Composition

```wgsl
@fragment
fn fs_main(@builtin(position) coord: vec4<f32>) -> @location(0) vec4<f32> {
    let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
    let uv = coord.xy / resolution;
    let p = (uv - 0.5) * vec2<f32>(resolution.x / resolution.y, 1.0);
    
    // Layer 1: Background gradient
    let bg = mix(vec3<f32>(0.1, 0.0, 0.2), vec3<f32>(0.0, 0.1, 0.3), uv.y);
    
    // Layer 2: Tunnel structure (bass reactive, modA is 0-1)
    let dist = length(p);
    let angle = atan2(p.y, p.x);
    let tunnel = fract(1.0 / (dist + 0.1) - uniforms.time + uniforms.modA * 0.5);
    let tunnelColor = vec3<f32>(tunnel * 0.5, tunnel * 0.3, tunnel);
    
    // Layer 3: Rotating pattern (mid reactive)
    let rotAngle = uniforms.time + uniforms.modB * 2.0;
    let pattern = sin(angle * 8.0 + rotAngle) * 0.5 + 0.5;
    let patternColor = vec3<f32>(0.2, 0.5, 0.8) * pattern * uniforms.modB;
    
    // Layer 4: Sparkle (treble reactive)
    let sparkle = step(0.97, fract(sin(dot(uv, vec2<f32>(12.9898, 78.233)) + floor(uniforms.time * 30.0)) * 43758.5453)) * uniforms.modC;
    
    // Composite
    var color = bg;
    color = mix(color, tunnelColor, 0.6);
    color = color + patternColor;
    color = color + vec3<f32>(sparkle);
    
    // Post-processing: vignette
    let vignette = 1.0 - smoothstep(0.3, 0.7, dist);
    color = color * vignette;
    
    // Bass flash
    color = color + vec3<f32>(uniforms.modA * uniforms.modA * 0.2);
    
    return vec4<f32>(color, 1.0);
}
```

---

### E.13 Quick Reference Cheat Sheet

#### Essential Functions

```wgsl
// Distance from center
let dist = length(uv - vec2<f32>(0.5));

// Angle from center
let angle = atan2(uv.y - 0.5, uv.x - 0.5);

// Repeat/tile
let tiled = fract(uv * 4.0);

// Smooth threshold
let edge = smoothstep(0.4, 0.5, value);

// Rotation
let c = cos(angle);
let s = sin(angle);
let rotated = vec2<f32>(p.x * c - p.y * s, p.x * s + p.y * c);

// Mix colors
let blended = mix(colorA, colorB, factor);
```

#### Common Value Ranges

| Parameter | Typical Range | Notes |
|-----------|---------------|-------|
| UV coordinates | 0.0 - 1.0 | Screen space |
| Colors | 0.0 - 1.0 | Per channel |
| Modulators | 0.0 - 1.0 | Audio input (already normalized) |
| Time | 0.0 - ∞ | Seconds |
| Angles | -π to π | Radians |
| Distances | 0.0 - ~0.7 | From center |

#### Quick Pattern Starters

```wgsl
// Circles
sin(length(uv - 0.5) * 20.0 - uniforms.time * 2.0)

// Stripes
sin(uv.x * 30.0 + uniforms.time)

// Checkerboard
(floor(uv.x * 8.0) + floor(uv.y * 8.0)) % 2.0

// Radial segments
step(0.5, fract(atan2(uv.y-0.5, uv.x-0.5) * 4.0 / 6.28))

// Moving dots
length(fract(uv * 8.0) - 0.5)
```

---

### E.14 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | December 2025 | Initial vocabulary catalog |
| 1.1.0 | December 2025 | Updated for OMS shader contract (fs_main, auto-injected struct) |
| 2.0.0 | December 2025 | Added Transform chain (V5_0_0), 112-byte uniform buffer |
| 2.1.0 | December 2025 | Updated for V5_1_0 (Cue Points, BumpR StickRs) |

---

**END OF INDEX A**

---

**END OF USER GUIDE**

Version: Beta V5_1_0  
Date: December 24, 2025  
License: GPL-3.0
