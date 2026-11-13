# OMS BETA V4_7_0 - USER GUIDE

**Version:** Beta V4_7_0  
**Date:** December 12, 2025  
**License:** GPL-3.0

---

## Table of Contents

1. Introduction
2. Instant Gratification
3. Using the Application
4. Support
5. Credits

INDEX A: DJ 101 Foundation
INDEX B: Claude Shader Creation
INDEX C: FFmpeg Video Conversion
INDEX D: Visual Vocabulary

---

## 1. Introduction

### 1.1 What is OMS Beta?

OMS Beta is a browser-based tool for DJing with live reactive visuals. No installation, no licenses - just open the HTML file. You create audio-reactive shaders, organize them in a library, and perform live with a 4-track mixer that combines audio and visuals.

**Platform:** macOS only  
**Tested on:** M1 Pro MacBook Pro 14" (2021)  
**Browser:** Chrome (WebGPU support required)  
**Screen:** 1352×761 effective viewport (14" MacBook Pro)

**File:** `OMS_Beta_V4_7_0.html`

---

## 2. Instant Gratification

### 2.1 Quick Visual Content

If you want to create visuals based on music timing without setting up a library:

**Audio-Reactive Visuals (reacts to your music):**
1. Open `OMS_Beta_V4_7_0.html`
2. Drag your audio file (.mp3, .wav) onto the waveform
3. Visuals automatically react to bass, mids, treble
4. Click [Render Video]
5. Get SILENT video with pre-baked reactivity

**Tempo-Synced Visuals (animated to BPM, no audio needed):**
1. Open `OMS_Beta_V4_7_0.html`
2. Set tempo (e.g., 128 BPM)
3. Set duration (e.g., 03:30.000)
4. Click [Render Video]
5. Get animated visual

**What you get:**
- WebM video file (convert with FFmpeg - see INDEX C)
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
5. OMS creates subfolders: /Library/, /Presets/, /Exports/
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

**What it receives (uniforms):**
- `uTime` - seconds since start
- `uTempo` - beats per minute
- `uModA`, `uModB`, `uModC` - modulators (0.0-1.0)
- `uResolution` - canvas dimensions

**What it does:**
- Runs once per pixel
- Calculates color based on position and time
- Returns RGBA color

**What it returns:**
- Color for each pixel: `vec4f(r, g, b, a)`

**Example:**
```wgsl
@fragment
fn main(@builtin(position) pos: vec4f) -> @location(0) vec4f {
    let pulse = sin(uTime * uTempo / 60.0) * 0.5 + 0.5;
    return vec4f(pulse, 0.0, uModA, 1.0);
}
```

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
- Types: Sine, Square, Triangle, Sawtooth, Cycler, etc.
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

**Duration** (text field: MM:SS.mmm)
- Render length when no audio loaded
- Example: `03:30.000` = 3 minutes 30 seconds
- Only used for shader-only renders

---

##### 3.2.4.4 Reactivity Section

**Modulator Controls** (5 columns × 3 rows)

Each modulator (A, B, C) has:

**Column 1: Subdivision** (dropdown)
- Options: 1/4, 1/2, Whole, 2 Bars, 4 Bars, 8 Bars, 16 Bars
- How often waveform repeats
- Example: "Whole" = once per bar

**Column 2: Waveform** (dropdown)
- Options: Sine, Square, Triangle, Sawtooth, Cycler, Chaosmode, Madmax, Psycho
- Shape of oscillation
- Sine = smooth, Square = sharp

**Column 3: Gain** (number field: 0.0-2.0)
- Amplitude multiplier
- 1.0 = normal, 2.0 = double strength

**Column 4: Freq1** (number field: Hz)
- Audio frequency range start
- Used during timeline analysis
- Default: modA=30, modB=50, modC=100

**Column 5: Freq2** (number field: Hz)
- Audio frequency range end
- Defines frequency band
- Default: modA=50, modB=75, modC=500

---

##### 3.2.4.5 OUTPUT Section

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
| G | Toggle beatgrid |

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

**Track Selection:**

| Shortcut | Action |
|----------|--------|
| 1-4 | Select track |
| 5 | Select master |
| Tab | Next track |
| Shift+Tab | Previous track |

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

### 3.5 ADMIN

#### 3.5.1 Overview

The ADMIN tab contains settings, utilities, and application info.

**Three sections:**
- Settings - Application configuration
- Utilities - Tools and batch operations
- About - Credits and license

---

#### 3.5.2 Interface Layout

**Left Navigation:**
- [Settings]
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

#### 3.5.6 ADMIN Keyboard Shortcuts

No keyboard shortcuts for ADMIN tab.

---

## 4. Support

**@itswessmithyo**  
Social / YouTube

**https://onemanshyo.com**

---

## 5. Credits

**Created by:** Wes Smith / ONEMANSHYO  
**Version:** Beta V4_7_0  
**Date:** December 12, 2025

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

## INDEX A: DJ 101 Foundation

Understanding what DJing is and how it evolved from analog to digital performance.

---

### A.1 The Analog Foundation

#### A.1.1 Basic DJing: Two Turntables and a Mixer

Basic DJing with two turntables and a microphone involves a classic setup for seamlessly blending music and engaging a crowd. The core skills revolve around equipment operation, beatmatching, smooth transitions, and using the microphone for announcements or as part of a performance.

---

#### A.1.2 Essential Equipment

The "two turntables and a microphone" setup requires specific gear connected to a central mixer.

**Two DJ Turntables:** These are the playback devices, ideally direct-drive models with pitch controls to adjust the playback speed (BPM).

**DJ Mixer:** The central hub where both turntables and the microphone are connected. It allows you to control the volume, EQ (highs, mids, lows), and transition (fade) between the two music sources.

**Microphone:** Used for speaking to the audience, making announcements, or for an MC (Master of Ceremonies) to rap or "toast" over the music.

**DJ Headphones:** Essential for cueing (listening to the next track privately) without the audience hearing it through the main speakers.

**Speakers/Monitors:** To output the final, mixed sound to the audience.

---

#### A.1.3 Core DJ Techniques

The primary skills for this setup involve manipulating the records to create a continuous flow of music.

**Cueing:** Using your headphones to find the starting point (often the first beat or "the one") of the next track on the inactive turntable. You pull the record back and forth slightly to mark the spot.

**Beatmatching:** The fundamental skill of adjusting the pitch (tempo fader) of the incoming track to match the beats per minute (BPM) of the track currently playing. You use your ears to ensure the kick drums and snares of both tracks align perfectly in rhythm.

**Phrasing:** Understanding the structure of songs (usually in groups of 32 beats or 8 bars) helps you start your mix at a musically appropriate point, ensuring a smooth transition where the music "feels" right.

**Mixing/Blending:** Gradually transitioning between the two tracks using the mixer's channel faders or the crossfader, often adjusting the EQs (equalizers) to make the blend seamless. For example, you might lower the bass on the incoming track to avoid a muddy sound when both are playing.

**Using the Microphone:** The microphone input is usually a separate channel on the mixer. You control its volume (gain) and can use it to interact with the crowd (call-and-response), introduce songs, or provide live commentary.

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

**END OF INDEX A**

---
## INDEX B: Claude Shader Creation

**Purpose:** Guide for creating WGSL shaders using Claude AI  
**Updated for:** V4_7_0  
**Date:** December 12, 2025

---

### B.1 Quick Start

**Basic Workflow:**

1. **Open OMS Beta** in your browser
2. **Click [Code] button** in CREATE tab to see shader editor
3. **Copy the current shader code**
4. **Open Claude.ai** in another tab/window
5. **Paste shader code** and describe what you want
6. **Claude generates new shader**
7. **Copy new shader** back to Code view
8. **See instant preview** in canvas
9. **Test in PERFORM tab** with perfect grid alignment

**That's it!** No shader programming knowledge required.

---

### B.2 How to Ask Claude

#### B.2.1 Effective Prompts

**Good Examples:**

```
"Make this shader react more to bass frequencies in modA"

"Add a tunneling effect that pulses with the beat"

"Change the colors to purple and gold instead of cyan"

"Make the patterns rotate clockwise with modB"

"Create a kaleidoscope effect with 8-way symmetry"

"Add a vignette effect around the edges"

"Make it pulse harder when the kick drum hits"
```

**Bad Examples (Too Vague):**

```
"Make it better"
"Add effects"
"Make it cooler"
```

#### B.2.2 Be Specific About

**Visual Elements:**
- Colors (specific hues, RGB values)
- Shapes (circles, rectangles, lines, fractals)
- Patterns (grid, spiral, radial, noise)
- Movement (rotation, oscillation, pulsing)

**Audio Reactivity:**
- Which modulator (modA=bass, modB=mids, modC=treble)
- How to react (scale, brightness, speed, color)
- Intensity (subtle vs dramatic)

**Performance:**
- Complexity (simple vs complex)
- Smoothness (60fps target)
- PERFORM tab compatibility

---

### B.3 Shader Template Structure

#### B.3.1 Standard WGSL Shader

\`\`\`wgsl
// Uniforms (values from app)
@group(0) @binding(0) var<uniform> uTime: f32;
@group(0) @binding(1) var<uniform> uResolution: vec2<f32>;
@group(0) @binding(2) var<uniform> uModA: f32; // Bass (30-500 Hz)
@group(0) @binding(3) var<uniform> uModB: f32; // Mids (500-2000 Hz)
@group(0) @binding(4) var<uniform> uModC: f32; // Treble (2000-20000 Hz)

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
    
    // Your shader magic here
    var color = vec3<f32>(0.0);
    
    // Example: Bass-reactive gradient
    color = vec3(uv.x * uModA, uv.y * uModB, 0.5 + uModC * 0.5);
    
    return vec4<f32>(color, 1.0);
}
\`\`\`

#### B.3.2 What You Can Modify

**Safe to Change:**
- Everything inside \`fs_main()\` fragment shader
- Color calculations
- Pattern generation
- Audio reactivity (using modA, modB, modC)
- Mathematical formulas

**Don't Change:**
- Uniform declarations (\`@group\`, \`@binding\`)
- Vertex shader (\`vs_main\` function)
- Fragment shader signature (\`@fragment fn fs_main\`)
- Return type (\`vec4<f32>\`)

---

### B.4 Modulator System

#### B.4.1 Understanding Modulators

**modA (Bass):** 30-500 Hz
- Kick drums, sub bass, bass drops
- Best for: Big movements, main pulse, impact effects

**modB (Mids):** 500-2000 Hz
- Vocals, synths, guitars, snare hits
- Best for: Color shifts, secondary motion, texture

**modC (Treble):** 2000-20000 Hz
- Hi-hats, cymbals, sparkle, air
- Best for: Fine details, flickering, shimmer effects

#### B.4.2 Using Modulators in Shaders

**Basic Pattern:**

\`\`\`wgsl
// Scale something with bass
let size = 0.5 + uModA * 0.5; // 0.5 to 1.0

// Change brightness with mids
let brightness = 0.3 + uModB * 0.7; // 0.3 to 1.0

// Flicker with treble
let flicker = 1.0 - uModC * 0.3; // 1.0 to 0.7
\`\`\`

**Combined Effects:**

\`\`\`wgsl
// Rotate with bass, scale with mids
let angle = uTime + uModA * 3.14159;
let radius = 0.5 + uModB * 0.5;

// Color based on all three
let red = uModA;
let green = uModB;
let blue = uModC;
let color = vec3(red, green, blue);
\`\`\`

#### B.4.3 Prompting Claude for Audio Reactivity

**Good Examples:**

```
"Make the circles pulse larger when modA (bass) is high"

"Change color from blue to red based on modB (mids)"

"Add rapid flickering to the edges using modC (treble)"

"Use modA to control rotation speed"

"Make brightness follow modB"

"Create impact flash when modA peaks (kick drum)"
```

---

### B.5 Optimizing for Live Performance

#### B.5.1 Performance Tips

- Target 60fps (check in PERFORM tab with 4 tracks)
- Avoid expensive operations in loops
- Use smoothstep() for antialiasing
- Test with actual music, not silence

#### B.5.2 Workflow for PERFORM Tab

1. **Create shader** in CREATE tab
2. **Load audio** and set parameters
3. **Add to Library** (analyzes FFT)
4. **Save as .oms** container
5. **PERFORM tab** - drag to grid
6. **Launch** - see perfect alignment!
7. **Press M** - verify with Master Grid

---

### B.6 Common Patterns

#### B.6.1 Bass-Driven Pulse

**Prompt:**
```
"Create concentric circles that pulse with bass (modA)"
```

**Claude generates:**
\`\`\`wgsl
let center = vec2(0.5);
let dist = length(uv - center);
let circles = sin(dist * 20.0 - uTime * 2.0 + uModA * 5.0);
let color = vec3(circles * (0.5 + uModA * 0.5));
\`\`\`

#### B.6.2 Frequency-Split Colors

**Prompt:**
```
"Make RGB channels respond to bass/mid/treble separately"
```

#### B.6.3 Beat-Synced Rotation

**Prompt:**
```
"Rotate shapes at tempo, speed up with mids"
```

#### B.6.4 Impact Effects

**Prompt:**
```
"Create flash/burst effect when bass peaks (kick drum)"
```

---

### B.7 Troubleshooting

#### B.7.1 Shader Won't Compile

**Error:** "Failed to compile shader"

**Solutions:**
1. Check for missing semicolons
2. Verify all parentheses match
3. Ensure uniform names are correct (uTime, uResolution, uModA, etc.)
4. Ask Claude: "This shader has a compile error, can you fix it?"

#### B.7.2 No Audio Reactivity in PERFORM Tab

**Problem:** Shader doesn't react to audio in performance

**Solutions:**
1. Re-save container with "Add to Library" (analyzes FFT)
2. Verify audio file loaded in CREATE tab
3. Check modulator usage in shader code

#### B.7.3 Performance Issues

**Slow Rendering:**
- Simplify shader calculations
- Reduce loop iterations
- Test with 4 tracks in PERFORM tab
- Ask Claude: "Make this shader more performant for live use"

---

### B.8 Advanced Techniques

#### B.8.1 Using Time Creatively

\`\`\`wgsl
// Slow pulsing
let pulse = sin(uTime * 0.5) * 0.5 + 0.5;

// Fast oscillation
let flicker = sin(uTime * 10.0) * 0.5 + 0.5;

// Wave motion
let wave = sin(uv.x * 10.0 + uTime * 2.0);

// Beat-synced (works in PERFORM tab!)
let beat = sin(uTime * 2.0) * uModA;
\`\`\`

#### B.8.2 Combining Modulators

\`\`\`wgsl
// Bass controls size, mids control brightness, treble adds detail
let size = 0.5 + uModA * 0.5;
let brightness = 0.3 + uModB * 0.7;
let detail = sin(uv.x * 50.0) * uModC * 0.2;
let circle = smoothstep(size + 0.01, size, length(uv - 0.5));
let color = vec3(circle * brightness + detail);
\`\`\`

---

### B.9 Example Conversations

#### B.9.1 Beginner

```
You: "Create pulsing circles that react to bass"
Claude: [generates shader]
You: "Make them pulse more dramatically"
Claude: [increases modA influence]
You: "Perfect! Now add color shift with mids"
Claude: [adds modB to color calculation]
```

#### B.9.2 Intermediate

```
You: "Create a tunnel effect synced to kick drums"
Claude: [generates tunnel with modA]
You: "Add rotation controlled by mids"
Claude: [adds modB rotation]
You: "Make the edges flicker with hi-hats"
Claude: [adds modC edge effects]
```

#### B.9.3 Advanced

```
You: "Create particle system where bass creates bursts,
      mids control particle color, and treble adds turbulence"
Claude: [generates complex multi-layer system]
You: "Optimize for 60fps with 4 tracks in PERFORM tab"
Claude: [simplifies calculations while preserving effect]
```

---

### B.10 Resources

**WGSL Documentation:**
- https://www.w3.org/TR/WGSL/
- https://google.github.io/tour-of-wgsl/

**Shader Inspiration:**
- https://thebookofshaders.com/
- https://www.shadertoy.com/

**Claude AI:**
- https://claude.ai/
- Ask for shader help anytime!

---

### B.11 Conclusion

Combined with Claude AI for shader creation, you can:

1. **Create** stunning audio-reactive shaders
2. **Prepare** with bar-aligned timeline analysis
3. **Perform** with confidence knowing grids align
4. **Verify** alignment with M key anytime

No shader programming required - just describe what you want, and Claude translates that into WGSL code.

**The power is in iteration:**
- Start simple
- Build complexity
- Test in PERFORM tab
- Press M to verify alignment
- Refine for performance

---

**END OF INDEX B**

---

## INDEX C: FFmpeg Video Conversion

### C.1 Overview

OMS Beta exports WebM format. Use FFmpeg to convert for different uses.

**FFmpeg Installation (macOS):**
```bash
brew install ffmpeg
```

---

### C.2 Common Conversions

#### C.2.1 H.264 (Social Media)

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

#### C.2.2 ProRes (Professional)

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

#### C.2.3 H.265 (Archive)

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

### C.3 Batch Conversion

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
3. Run: `./convert.sh`

---

### C.4 Understanding Settings

#### C.4.1 CRF (Quality)

**Constant Rate Factor:**
- Lower = better quality, larger file
- Higher = lower quality, smaller file

**Recommendations:**
- CRF 18 = visually lossless
- CRF 23 = high quality (default)
- CRF 28 = acceptable quality

---

#### C.4.2 Presets (Speed)

**Encoding speed:**
- ultrafast = fastest, largest files
- fast = quick, good quality
- medium = balanced (default)
- slow = slower, better compression
- veryslow = slowest, best compression

**Recommendations:**
- `fast` for quick previews
- `medium` for general use
- `slow` for final renders

---

#### C.4.3 Audio Bitrates

**Common values:**
- 128k = acceptable
- 192k = good
- 256k = excellent
- 320k = maximum

---

### C.5 Troubleshooting

**"Unknown encoder 'libx264'"**
- Install FFmpeg: `brew install ffmpeg`

**"Conversion failed"**
- Check input file isn't corrupted
- Try: `ffmpeg -i input.webm -c copy test.webm`

**"Output too large"**
- Increase CRF value (lower quality)
- Use faster preset
- Reduce resolution

**"Playback stutters"**
- Convert to ProRes for editing
- Use H.264 for final delivery

---

### C.6 Quick Reference

**Social media (H.264):**
```bash
ffmpeg -i input.webm -c:v libx264 -preset fast -crf 23 -c:a aac -b:a 128k output.mp4
```

**Professional (ProRes):**
```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

**Archive (H.265):**
```bash
ffmpeg -i input.webm -c:v libx265 -preset medium -crf 28 -c:a aac -b:a 128k output.mp4
```

---

**END OF USER GUIDE**
## INDEX D: Visual Vocabulary


---


### D.1 Spatial Structures

Large-scale spatial arrangements that define the overall visual environment.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Tunnel** | Infinite depth illusion, camera moving forward | `1.0 / dist` with scrolling `uTime` |
| **Vortex** | Spiral pulling toward center | `atan2` + radial distance |
| **Grid** | Repeating cells/tiles | `fract(uv * count)` |
| **Warp Grid** | Distorted grid (curved space) | Grid + sine displacement |
| **Lattice** | 3D grid structure | Raymarched boxes |
| **Corridor** | Rectangular tunnel | Box SDF with depth |
| **Sphere Field** | Array of spheres in space | Raymarched with `mod` repetition |
| **Hyperspace** | Streaking lines toward viewer | Radial lines + depth fade |
| **Wormhole** | Twisted tunnel with rotation | Tunnel + angular distortion |
| **Matrix** | Falling columns of symbols | Column-based animation |

#### WGSL Examples

**Basic Tunnel:**
```wgsl
let center = uv - vec2(0.5);
let dist = length(center);
let angle = atan2(center.y, center.x);
let tunnel = fract(1.0 / dist - uTime);
```

**Infinite Grid:**
```wgsl
let gridSize = 10.0;
let grid = fract(uv * gridSize);
let lines = step(0.95, max(grid.x, grid.y));
```

**Warp Grid:**
```wgsl
let warp = sin(uv.x * 10.0 + uTime) * 0.1;
let warped = vec2(uv.x, uv.y + warp);
let grid = fract(warped * 8.0);
```

---

### D.2 Radial Patterns

Patterns that emanate from or rotate around a center point.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Ripple** | Concentric rings expanding outward | `sin(dist - uTime)` |
| **Pulse** | Single expanding ring | `smoothstep` ring with time |
| **Starburst** | Lines radiating from center | `atan2` segmentation |
| **Kaleidoscope** | Mirrored angular segments | `mod(angle, PI/segments)` |
| **Mandala** | Symmetric radial pattern | Kaleidoscope + layered shapes |
| **Radar Sweep** | Rotating line/wedge | `atan2` comparison with time |
| **Shockwave** | Single sharp expanding ring | Thin `smoothstep` ring |
| **Sunburst** | Alternating radial segments | `step` on angle divisions |
| **Spiral** | Rotating radial pattern | `angle + dist` |
| **Bullseye** | Static concentric rings | `fract(dist * count)` |

#### WGSL Examples

**Ripple:**
```wgsl
let center = uv - vec2(0.5);
let dist = length(center);
let ripple = sin(dist * 20.0 - uTime * 4.0) * 0.5 + 0.5;
```

**Kaleidoscope (8-way):**
```wgsl
let center = uv - vec2(0.5);
let angle = atan2(center.y, center.x);
let segments = 8.0;
let kaleidoAngle = abs(((angle / 3.14159) * segments) % 2.0 - 1.0);
```

**Shockwave:**
```wgsl
let dist = length(uv - vec2(0.5));
let ringPos = fract(uTime * 0.5);  // expanding ring
let ring = smoothstep(ringPos - 0.02, ringPos, dist) 
         - smoothstep(ringPos, ringPos + 0.02, dist);
```

**Spiral:**
```wgsl
let center = uv - vec2(0.5);
let dist = length(center);
let angle = atan2(center.y, center.x);
let spiral = sin(angle * 5.0 + dist * 20.0 - uTime * 3.0);
```

---

### D.3 Flow Fields

Patterns suggesting movement, fluid dynamics, or particle systems.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Flow Field** | Directional streaming motion | Noise-based UV displacement |
| **Particle Stream** | Dots moving along paths | Hash + velocity integration |
| **Smoke/Fluid** | Organic flowing shapes | Layered noise with advection |
| **Aurora** | Vertical color ribbons | Sine waves + gradient |
| **Plasma** | Classic demo effect, shifting blobs | `sin(x) + sin(y) + sin(x+y)` |
| **Lava Lamp** | Blobby morphing shapes | Metaballs or smooth noise |
| **Fire** | Upward turbulent motion | Noise scrolling up + gradient |
| **Water** | Reflective surface ripples | Layered sine distortion |
| **Clouds** | Soft billowing shapes | FBM noise |
| **Wind Lines** | Streaming directional marks | Elongated noise |

#### WGSL Examples

**Classic Plasma:**
```wgsl
let plasma = sin(uv.x * 10.0 + uTime)
           + sin(uv.y * 10.0 + uTime * 1.1)
           + sin((uv.x + uv.y) * 10.0 + uTime * 1.2)
           + sin(length(uv - 0.5) * 10.0 - uTime);
let color = plasma * 0.25 + 0.5;
```

**Aurora Effect:**
```wgsl
let wave1 = sin(uv.x * 3.0 + uTime) * 0.2;
let wave2 = sin(uv.x * 5.0 - uTime * 1.3) * 0.15;
let aurora = smoothstep(0.3 + wave1 + wave2, 0.5, uv.y);
```

**Simple Fire:**
```wgsl
// Requires noise function
let fireUV = vec2(uv.x, uv.y + uTime * 0.5);
let n = noise(fireUV * 5.0);
let fire = smoothstep(0.5, 0.0, uv.y + n * 0.3);
```

---

### D.4 Geometric Primitives

Basic shapes used as building blocks for complex patterns.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Circle** | Basic ring or filled disc | `length(uv - center)` |
| **Ring** | Hollow circle | `abs(dist - radius)` |
| **Box** | Rectangle/square | `max(abs(uv.x), abs(uv.y))` |
| **Rounded Box** | Rectangle with soft corners | Box SDF with corner radius |
| **Triangle** | Three-sided shape | Half-plane intersection |
| **Polygon** | N-sided regular shape | `atan2` + `cos` |
| **Cross** | Plus sign shape | `min(abs(x), abs(y))` |
| **Star** | Pointed star shape | Polygon with alternating radii |
| **Line** | Straight segment | Point-to-line distance |
| **Arc** | Curved segment | Angle-limited ring |

#### WGSL Examples

**Circle (SDF):**
```wgsl
fn sdCircle(p: vec2<f32>, r: f32) -> f32 {
    return length(p) - r;
}

// Usage
let d = sdCircle(uv - vec2(0.5), 0.3);
let circle = smoothstep(0.01, 0.0, d);
```

**Ring:**
```wgsl
let dist = length(uv - vec2(0.5));
let ring = smoothstep(0.01, 0.0, abs(dist - 0.3) - 0.02);
```

**Box (SDF):**
```wgsl
fn sdBox(p: vec2<f32>, b: vec2<f32>) -> f32 {
    let d = abs(p) - b;
    return length(max(d, vec2(0.0))) + min(max(d.x, d.y), 0.0);
}
```

**Regular Polygon:**
```wgsl
fn sdPolygon(p: vec2<f32>, r: f32, n: f32) -> f32 {
    let angle = atan2(p.y, p.x);
    let segment = 6.28318 / n;
    let a = ((angle + segment * 0.5) % segment) - segment * 0.5;
    return length(p) * cos(a) - r;
}
```

---

### D.5 Fractal/Complex

Self-similar and mathematically complex patterns.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Fractal Noise** | Self-similar layered noise | FBM (Fractional Brownian Motion) |
| **Mandelbrot** | Classic fractal zoom | `z = z² + c` iteration |
| **Julia Set** | Mandelbrot variant | Fixed c, varying start |
| **Sierpinski** | Triangular fractal | Folding operations |
| **Kaleido-Fractal** | Fractal with angular symmetry | Kaleidoscope + iteration |
| **Voronoi** | Cell-based pattern | Nearest point calculation |
| **Worley Noise** | Cellular texture | Distance to random points |
| **Domain Warp** | Recursively distorted space | Noise feeding into noise |
| **IFS** | Iterated Function System | Repeated transforms |
| **Truchet** | Tile-based maze pattern | Random tile rotation |

#### WGSL Examples

**FBM (Fractal Brownian Motion):**
```wgsl
fn fbm(p: vec2<f32>) -> f32 {
    var value = 0.0;
    var amplitude = 0.5;
    var pos = p;
    for (var i = 0; i < 5; i++) {
        value += amplitude * noise(pos);
        pos *= 2.0;
        amplitude *= 0.5;
    }
    return value;
}
```

**Simple Voronoi:**
```wgsl
fn voronoi(p: vec2<f32>) -> f32 {
    let cell = floor(p);
    let local = fract(p);
    var minDist = 1.0;
    for (var y = -1; y <= 1; y++) {
        for (var x = -1; x <= 1; x++) {
            let neighbor = vec2(f32(x), f32(y));
            let point = hash2(cell + neighbor);  // random point in cell
            let diff = neighbor + point - local;
            minDist = min(minDist, length(diff));
        }
    }
    return minDist;
}
```

**Domain Warp:**
```wgsl
let warpedUV = uv + vec2(
    noise(uv * 4.0 + uTime),
    noise(uv * 4.0 + 100.0 + uTime)
) * 0.2;
let color = noise(warpedUV * 8.0);
```

---

### D.6 Motion Types

Categories of movement and animation behaviors.

| Term | Description | Application |
|------|-------------|-------------|
| **Oscillate** | Back and forth | `sin(uTime)` |
| **Rotate** | Spinning motion | `mat2` rotation matrix |
| **Zoom** | Scale in/out | Multiply UV by time factor |
| **Pan** | Lateral drift | Add time to UV |
| **Breathe** | Gentle scale pulse | `1.0 + sin(uTime) * 0.1` |
| **Strobe** | Hard on/off flash | `step(0.5, fract(uTime * freq))` |
| **Flutter** | Rapid small variations | High-freq noise or sine |
| **Drift** | Slow wandering | Low-freq noise offset |
| **Bounce** | Elastic rebound | `abs(sin(uTime))` |
| **Ease** | Smooth acceleration | Cubic/sine easing functions |
| **Jitter** | Random micro-movement | Per-frame noise |
| **Orbit** | Circular path around point | `sin`/`cos` position |

#### WGSL Examples

**Rotation Matrix:**
```wgsl
fn rotate2D(angle: f32) -> mat2x2<f32> {
    let c = cos(angle);
    let s = sin(angle);
    return mat2x2<f32>(c, -s, s, c);
}

// Usage
let rotatedUV = rotate2D(uTime) * (uv - 0.5) + 0.5;
```

**Breathing Scale:**
```wgsl
let breath = 1.0 + sin(uTime * 2.0) * 0.1;
let scaledUV = (uv - 0.5) / breath + 0.5;
```

**Smooth Strobe:**
```wgsl
let strobe = smoothstep(0.4, 0.5, fract(uTime * 4.0));
```

**Orbit:**
```wgsl
let orbitRadius = 0.2;
let orbitSpeed = 2.0;
let orbitPos = vec2(
    cos(uTime * orbitSpeed) * orbitRadius,
    sin(uTime * orbitSpeed) * orbitRadius
);
let center = vec2(0.5) + orbitPos;
```

---

### D.7 Transitions/Events

Effects triggered by audio or time, often used for emphasis.

| Term | Description | Trigger |
|------|-------------|---------|
| **Flash** | Full-screen brightness spike | `uModA` peak (kick) |
| **Impact** | Shockwave from center | `uModA` threshold |
| **Glitch** | Digital corruption effect | Random UV displacement |
| **Invert** | Color/brightness flip | `1.0 - color` |
| **Bloom** | Glow bleeding outward | Blur + add |
| **Vignette** | Edge darkening | Radial falloff |
| **Fade** | Opacity transition | Linear interpolation |
| **Cut** | Hard switch between states | Conditional branch |
| **Wipe** | Directional reveal | Moving threshold |
| **Dissolve** | Noise-based transition | Noise threshold |
| **Shake** | Camera vibration | UV offset by noise |
| **Zoom Punch** | Quick zoom in/out | Scale spike on beat |

#### WGSL Examples

**Bass Flash:**
```wgsl
let flash = uModA * uModA;  // squared for punch
let color = baseColor + vec3(flash * 0.5);
```

**Impact Shockwave:**
```wgsl
let dist = length(uv - vec2(0.5));
let shockRadius = uModA * 0.5;  // expands with bass
let shock = smoothstep(shockRadius - 0.02, shockRadius, dist)
          - smoothstep(shockRadius, shockRadius + 0.02, dist);
```

**Glitch:**
```wgsl
let glitchStrength = step(0.8, uModA);  // trigger on loud bass
let offset = hash(floor(uv.y * 20.0 + uTime * 100.0)) * 0.1 * glitchStrength;
let glitchedUV = vec2(uv.x + offset, uv.y);
```

**Vignette:**
```wgsl
let dist = length(uv - vec2(0.5));
let vignette = 1.0 - smoothstep(0.3, 0.7, dist);
let color = baseColor * vignette;
```

**Zoom Punch:**
```wgsl
let punch = 1.0 + uModA * 0.2;  // scale up on bass
let punchedUV = (uv - 0.5) / punch + 0.5;
```

---

### D.8 Color Operations

Techniques for manipulating and generating color.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Gradient** | Smooth color blend | `mix(colorA, colorB, t)` |
| **Palette Cycle** | Rotating through colors | `sin` RGB with phase offsets |
| **Hue Shift** | Rotate color wheel | RGB→HSV→rotate→RGB |
| **Saturation** | Color intensity | HSV S channel |
| **Posterize** | Reduce color levels | `floor(color * levels) / levels` |
| **Duotone** | Two-color mapping | Luminance → two colors |
| **Rainbow** | Full spectrum sweep | `uv.x` or `dist` → hue |
| **Complement** | Opposite color | `1.0 - color` or hue + 0.5 |
| **Triadic** | Three evenly-spaced hues | Hue + 0.33, Hue + 0.66 |
| **Temperature** | Warm/cool color shift | Blue-orange gradient |

#### WGSL Examples

**Palette Cycling:**
```wgsl
fn palette(t: f32) -> vec3<f32> {
    let a = vec3(0.5, 0.5, 0.5);
    let b = vec3(0.5, 0.5, 0.5);
    let c = vec3(1.0, 1.0, 1.0);
    let d = vec3(0.0, 0.33, 0.67);
    return a + b * cos(6.28318 * (c * t + d));
}

// Usage
let color = palette(uTime * 0.1 + length(uv - 0.5));
```

**HSV to RGB:**
```wgsl
fn hsv2rgb(hsv: vec3<f32>) -> vec3<f32> {
    let h = hsv.x * 6.0;
    let s = hsv.y;
    let v = hsv.z;
    let c = v * s;
    let x = c * (1.0 - abs(h % 2.0 - 1.0));
    let m = v - c;
    var rgb: vec3<f32>;
    if (h < 1.0) { rgb = vec3(c, x, 0.0); }
    else if (h < 2.0) { rgb = vec3(x, c, 0.0); }
    else if (h < 3.0) { rgb = vec3(0.0, c, x); }
    else if (h < 4.0) { rgb = vec3(0.0, x, c); }
    else if (h < 5.0) { rgb = vec3(x, 0.0, c); }
    else { rgb = vec3(c, 0.0, x); }
    return rgb + vec3(m);
}
```

**Rainbow Gradient:**
```wgsl
let hue = fract(uv.x + uTime * 0.1);
let color = hsv2rgb(vec3(hue, 1.0, 1.0));
```

**Posterize:**
```wgsl
let levels = 4.0;
let posterized = floor(color * levels) / levels;
```

**Duotone:**
```wgsl
let luma = dot(color, vec3(0.299, 0.587, 0.114));
let shadow = vec3(0.1, 0.0, 0.2);   // dark purple
let highlight = vec3(1.0, 0.8, 0.2); // gold
let duotone = mix(shadow, highlight, luma);
```

---

### D.9 Texture/Surface

Surface qualities and textural effects.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Noise** | Random variation | Perlin, Simplex, Value noise |
| **Grain** | Film-like texture | High-freq hash |
| **Scanlines** | Horizontal lines | `mod(fragCoord.y, spacing)` |
| **CRT** | Retro monitor effect | Scanlines + barrel distortion |
| **Halftone** | Dot pattern | Threshold against dot grid |
| **Dither** | Ordered noise pattern | Bayer matrix |
| **Moire** | Interference pattern | Overlapping grids |
| **Static** | TV noise | Per-pixel random |
| **Chromatic Aberration** | RGB color fringing | Offset RGB channels |
| **Pixelate** | Blocky low-res | `floor(uv * res) / res` |

#### WGSL Examples

**Simple Hash Noise:**
```wgsl
fn hash(p: vec2<f32>) -> f32 {
    return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453);
}
```

**Film Grain:**
```wgsl
let grain = hash(uv * 1000.0 + uTime * 100.0) * 0.1;
let color = baseColor + vec3(grain - 0.05);
```

**Scanlines:**
```wgsl
let scanline = sin(fragCoord.y * 3.14159) * 0.5 + 0.5;
let color = baseColor * (0.8 + scanline * 0.2);
```

**CRT Effect:**
```wgsl
// Barrel distortion
let center = uv - 0.5;
let dist = length(center);
let barrel = center * (1.0 + dist * dist * 0.2);
let crtUV = barrel + 0.5;

// Scanlines
let scan = sin(crtUV.y * uResolution.y * 3.14159) * 0.5 + 0.5;

// Vignette
let vignette = 1.0 - dist * 0.5;
```

**Chromatic Aberration:**
```wgsl
let aberration = 0.005;
let r = textureSample(tex, samp, uv + vec2(aberration, 0.0)).r;
let g = textureSample(tex, samp, uv).g;
let b = textureSample(tex, samp, uv - vec2(aberration, 0.0)).b;
let color = vec3(r, g, b);
```

**Pixelate:**
```wgsl
let pixelSize = 64.0;
let pixelated = floor(uv * pixelSize) / pixelSize;
```

---

### D.10 Depth/3D Illusions

Techniques that create the impression of three-dimensional space.

| Term | Description | Core Math |
|------|-------------|-----------|
| **Parallax** | Layered depth motion | Multiple layers, different speeds |
| **Raymarching** | 3D SDF rendering | Distance field stepping |
| **Fog** | Distance-based fade | `exp(-dist * density)` |
| **Shadow** | Darkened regions | Secondary ray or fake |
| **Reflection** | Mirror effect | Flipped coordinates |
| **Refraction** | Bent light | UV displacement by normal |
| **Depth of Field** | Focus blur | Distance-based blur |
| **Ambient Occlusion** | Contact shadows | Local occlusion sampling |
| **Glow/Emission** | Self-illumination | Add brightness |
| **Fresnel** | Edge glow effect | View angle falloff |

#### WGSL Examples

**Parallax Layers:**
```wgsl
let layer1 = texture(uv + vec2(uTime * 0.1, 0.0));  // slow
let layer2 = texture(uv + vec2(uTime * 0.3, 0.0));  // medium
let layer3 = texture(uv + vec2(uTime * 0.5, 0.0));  // fast (foreground)
let color = mix(mix(layer1, layer2, 0.5), layer3, 0.5);
```

**Basic Raymarching:**
```wgsl
fn map(p: vec3<f32>) -> f32 {
    // Sphere SDF
    return length(p) - 1.0;
}

fn raymarch(ro: vec3<f32>, rd: vec3<f32>) -> f32 {
    var t = 0.0;
    for (var i = 0; i < 64; i++) {
        let p = ro + rd * t;
        let d = map(p);
        if (d < 0.001) { break; }
        t += d;
        if (t > 100.0) { break; }
    }
    return t;
}
```

**Fog:**
```wgsl
let fogDensity = 0.1;
let fogColor = vec3(0.5, 0.6, 0.7);
let fog = 1.0 - exp(-distance * fogDensity);
let color = mix(objectColor, fogColor, fog);
```

**Fresnel (Edge Glow):**
```wgsl
let viewDir = normalize(cameraPos - worldPos);
let fresnel = pow(1.0 - max(dot(normal, viewDir), 0.0), 3.0);
let color = baseColor + glowColor * fresnel;
```

---

### D.11 Audio-Reactive Mappings

Best practices for connecting audio analysis to visual parameters.

#### Modulator Characteristics

| Modulator | Frequency Range | Character | Best For |
|-----------|-----------------|-----------|----------|
| `uModA` | 30-500 Hz (Bass) | Punchy, rhythmic | Big movements, main pulse, impacts |
| `uModB` | 500-2000 Hz (Mids) | Melodic, sustained | Color shifts, rotation, secondary motion |
| `uModC` | 2000-20000 Hz (Treble) | Crisp, transient | Fine detail, shimmer, edge effects |

#### Recommended Mappings

**uModA (Bass) → Structure**
```wgsl
// Size/scale
let size = 0.5 + uModA * 0.5;

// Brightness flash
let brightness = baseColor + vec3(uModA * 0.3);

// Zoom punch
let zoom = 1.0 + uModA * 0.2;
let punchedUV = (uv - 0.5) / zoom + 0.5;

// Ring expansion (shockwave)
let ringRadius = uModA * 0.5;
```

**uModB (Mids) → Movement**
```wgsl
// Rotation speed
let angle = uTime * (1.0 + uModB * 2.0);

// Hue shift
let hue = fract(baseHue + uModB * 0.2);

// Pattern complexity
let iterations = 3 + i32(uModB * 5.0);

// Wave amplitude
let wave = sin(uv.x * 10.0) * uModB * 0.2;
```

**uModC (Treble) → Detail**
```wgsl
// Noise intensity
let noise = fbm(uv * 10.0) * uModC;

// Edge sharpness
let edge = smoothstep(0.5 - uModC * 0.1, 0.5, pattern);

// Flicker/sparkle
let sparkle = step(0.95, hash(uv + uTime)) * uModC;

// Fine texture
let grain = (hash(uv * 1000.0 + uTime) - 0.5) * uModC * 0.1;
```

#### Compound Effects

**All Three Combined:**
```wgsl
// Bass controls overall intensity
let intensity = 0.5 + uModA * 0.5;

// Mids control color
let hue = fract(uTime * 0.1 + uModB * 0.3);
let color = hsv2rgb(vec3(hue, 0.8, intensity));

// Treble adds sparkle
let sparkle = step(0.98, hash(uv * 500.0 + uTime)) * uModC;
let finalColor = color + vec3(sparkle);
```

---

### D.12 Combining Patterns

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
fn fs_main(@builtin(position) fragCoord: vec4<f32>) -> @location(0) vec4<f32> {
    let uv = fragCoord.xy / uResolution;
    
    // Layer 1: Background gradient
    let bg = mix(vec3(0.1, 0.0, 0.2), vec3(0.0, 0.1, 0.3), uv.y);
    
    // Layer 2: Tunnel structure (bass reactive)
    let center = uv - vec2(0.5);
    let dist = length(center);
    let angle = atan2(center.y, center.x);
    let tunnel = fract(1.0 / dist - uTime + uModA * 0.5);
    let tunnelColor = vec3(tunnel * 0.5, tunnel * 0.3, tunnel);
    
    // Layer 3: Kaleidoscope overlay (mid reactive)
    let segments = 8.0;
    let kAngle = abs(((angle / 3.14159) * segments) % 2.0 - 1.0);
    let kaleidoPattern = sin(kAngle * 20.0 + dist * 30.0 - uTime * 2.0);
    let kaleidoColor = vec3(0.2, 0.5, 0.8) * kaleidoPattern * uModB;
    
    // Layer 4: Sparkle (treble reactive)
    let sparkle = step(0.97, hash(uv * 500.0 + floor(uTime * 30.0))) * uModC;
    
    // Composite
    var color = bg;
    color = mix(color, tunnelColor, 0.6);           // Tunnel over background
    color = color + kaleidoColor * 0.5;              // Additive kaleidoscope
    color = color + vec3(sparkle);                   // Additive sparkle
    
    // Post-processing
    let vignette = 1.0 - smoothstep(0.3, 0.7, dist);
    color = color * vignette;
    
    // Bass flash
    color = color + vec3(uModA * uModA * 0.2);
    
    return vec4<f32>(color, 1.0);
}
```

---

### D.13 Quick Reference Cheat Sheet

#### Essential Functions

```wgsl
// Distance from center
let dist = length(uv - vec2(0.5));

// Angle from center
let angle = atan2(uv.y - 0.5, uv.x - 0.5);

// Repeat/tile
let tiled = fract(uv * 4.0);

// Smooth threshold
let edge = smoothstep(0.4, 0.5, value);

// Rotation
let rotated = mat2x2(cos(a), -sin(a), sin(a), cos(a)) * (uv - 0.5) + 0.5;

// Mix colors
let blended = mix(colorA, colorB, factor);
```

#### Common Value Ranges

| Parameter | Typical Range | Notes |
|-----------|---------------|-------|
| UV coordinates | 0.0 - 1.0 | Screen space |
| Colors | 0.0 - 1.0 | Per channel |
| Modulators | 0.0 - 1.0 | Audio input |
| Time | 0.0 - ∞ | Seconds |
| Angles | -π to π | Radians |
| Distances | 0.0 - ~0.7 | From center |

#### Quick Pattern Starters

```wgsl
// Circles
sin(length(uv - 0.5) * 20.0 - uTime * 2.0)

// Stripes
sin(uv.x * 30.0 + uTime)

// Checkerboard
step(0.5, fract(uv.x * 8.0)) != step(0.5, fract(uv.y * 8.0))

// Radial segments
step(0.5, fract(atan2(uv.y-0.5, uv.x-0.5) * 4.0 / 6.28))

// Moving dots
length(fract(uv * 8.0) - 0.5)
```

---

### D.14 Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | December 2025 | Initial vocabulary catalog |

---

**END OF INDEX D**

---

**END OF USER GUIDE**

Version: Beta V4_7_0  
Date: December 12, 2025  
License: GPL-3.0
