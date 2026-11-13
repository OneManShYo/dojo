# OMS SHADER BAKER V3.2.0 - COMPLETE USER GUIDE

**Version:** V3.2.0
**Audience:** All Users
**Purpose:** Comprehensive application guide

---

## QUICK START

### First Time Setup

1. **Download** OMS_ShaderBaker_V3_2_0.html
2. **Open** in Chrome or Safari
3. **Set Library Folder** (automatic prompt on LIVE/Library tab)
4. **Start Creating**

### 5-Minute Tutorial

**Create Your First Container:**
1. Go to Code tab
2. Use default shader (or write your own)
3. Go to Controls tab, click "Set Library Folder"
4. Drag audio file from LIBRARY folder
5. Set "Audio Tempo" (BPM)
6. Click "Add to Library"
7. Go to LIVE tab
8. Drag container to grid
9. Click cell to play

**You now have audio-reactive visuals with beat grid!**

---

## TABS OVERVIEW

### Tab 1: Controls
- Shader timing (tempo, loop length)
- Modulator controls (A/B/C)
- Audio controls (pitch, loop points)
- Render settings (resolution, FPS)
- Project States (session saving)

### Tab 2: Presets
- 5 built-in library shaders
- User-created presets
- Load/save configurations

### Tab 3: Library
- LIBRARY folder management
- Container list with thumbnails
- Add/Update/Delete containers
- Keyboard navigation

### Tab 4: Code
- WGSL shader code editor
- Syntax highlighting
- Direct editing

### Tab 5: LIVE (NEW in V3.2.0)
- Performance grid (4 tracks × 8 scenes)
- Multi-track audio playback
- Mixer controls with beat grid
- Master tempo control
- Synced waveform scrolling

---

## LIVE TAB GUIDE (V3.2.0)

### Performance Grid

**Layout:**
- 4 tracks (left to right)
- 8 scenes (top to bottom)
- 32 cells total
- Master column (right)

**Usage:**
- Drag containers from Library to cells
- Shader thumbnails render automatically
- Click cell to play audio
- Click empty cell to stop
- Green border = playing
- Yellow outline = selected

### Beat Grid System

**Visual Elements:**
- **Cyan bar lines** (thin) - Every bar
- **Magenta phrase markers** (thick) - Every 4 bars
- **White bar numbers** - 1, 5, 9, 13...
- **Slime green boundaries** - Audio start/end
- **Info overlay** (magenta) - Name + BPM

**Reading the Grid:**
- Bar lines help align tracks
- Phrase markers show song structure
- Mix on magenta lines for phrase-matched transitions
- Bar numbers for navigation

### Master Tempo Control

**Location:** Master column header

**Controls:**
- Type tempo (60-200 BPM)
- Up/Down arrows: ±0.1 BPM
- Left/Right arrows: ±5 BPM

**What It Does:**
- Time-stretches all tracks to master tempo
- Updates beat grid visual scaling
- All tracks stay synced

**Example:**
- Load 124 BPM track
- Set master to 120 BPM
- Track plays 3.2% slower
- Beat grid scales to match
- Perfect sync

### Waveforms Section

**Features:**
- 4 horizontal waveform strips
- Bar-style display (structure view)
- Magenta playhead (fixed center)
- Scrolling during playback
- Beat grid overlay

**Usage:**
- Watch waveforms scroll together
- See bars align vertically
- Anticipate builds/drops
- Visual feedback for mixing

### Mixer Section

**Per-Track Controls:**
- Volume knob (drag or keyboard)
- Mute button (M)
- Solo button (S)
- LED meters (animated)

**Keyboard:**
- Tab/Shift+Tab: Cycle knobs
- Arrow Up/Down: Adjust volume (±5%)
- Click knob for keyboard control (cyan ring)

---

## LIBRARY FOLDER SYSTEM

### Setup

**First Time:**
1. Click Library or LIVE tab
2. Folder picker opens automatically
3. Select your LIBRARY folder
4. Permission saves (persists)

**Manual:**
1. Controls tab
2. Click "Set Library Folder" button
3. Select folder

### Organization

**Recommended Structure:**
```
/LIBRARY
  ├── track01.mp3
  ├── track02.mp3
  ├── bass_loop.mp3
  ├── Bass_Tunnel.oms
  ├── Cosmic_Flow.oms
  ├── HD_Render_ControlState.oms
  └── default.omsp
```

**Files:**
- Audio files (.mp3, .wav, .m4a)
- Containers (.oms with shaders + timing)
- Control States (.oms render presets)
- Project States (.omsp LIVE sessions)

---

## CONTAINERS (.oms files)

### What's a Container

**Contains:**
- Shader code (WGSL)
- Shader settings (tempo, modulators)
- Audio filename reference
- Audio timing data (NEW in V3.2.0)
  - Original BPM
  - Start/End times
  - Pitch adjustment
  - Loop points

**One container = One complete audio-reactive visual setup**

### Creating Containers

**Method 1: From Scratch**
1. Code tab: Write shader
2. Controls tab: Load audio
3. Set audio tempo (BPM)
4. Configure modulators
5. Click "Add to Library"
6. Enter name
7. Saves as {Name}.oms

**Method 2: From Preset**
1. Presets tab: Load preset
2. Controls tab: Load audio
3. Set audio tempo
4. Adjust settings
5. Click "Add to Library"

### Updating Containers

**After changes:**
1. Load container
2. Modify shader/settings
3. Click "Update" button
4. All data saved (including audio timing)

**Important:** Always click Update to save timing data for beat grid

---

## AUDIO TIMING DATA

### Why It Matters

**Beat grid needs:**
- Original BPM (bar spacing)
- Start time (where Bar 1 is)
- End time (where song ends)
- Pitch adjustment (playback speed)
- Loop points (optional)

**Without timing data:**
- Beat grid won't display
- Audio plays but no visual reference
- No phrase markers

### Setting Timing Data

**In Controls Tab:**

1. **Audio Tempo (BPM):**
   - Field: "Audio Tempo"
   - Set original track BPM
   - Critical for beat grid

2. **Start/End Times:**
   - Sliders below waveform
   - Green = start time
   - Red = end time
   - Skip intro/outro

3. **Pitch:**
   - Field: "Pitch"
   - Range: ±16%
   - Adjust playback speed

4. **Loop Points:**
   - Format: Bar.Beat (e.g., 17.1)
   - In/Out fields
   - For looping sections

5. **Click Update:**
   - Saves all timing to .oms
   - Required for beat grid

---

## PROJECT STATES (.omsp files)

### What's a Project State

**Contains:**
- All 32 cell assignments
- Track volumes (4 tracks)
- Mute/Solo states
- Master volume
- Master tempo
- Last scene launched

**One project = One complete LIVE session**

### Saving Projects

**Auto-Save:**
- File: default.omsp
- Saves on every change
- Instant, no delay

**Manual Save:**
1. Controls tab (bottom)
2. Project States section
3. Click "Save" button
4. Enter name
5. Format: {Name}_YYYY-MM-DD_01.omsp

### Loading Projects

**Via Dropdown:**
1. Controls tab
2. Project States dropdown
3. Select project
4. Click "Load"
5. Entire LIVE session restores

**Auto-Load:**
- default.omsp loads on startup (if exists)
- Can disable in future versions

---

## KEYBOARD SHORTCUTS

### LIVE Tab

**Master Tempo:**
- Up/Down: ±0.1 BPM
- Left/Right: ±5 BPM

**Volume Knobs:**
- Tab/Shift+Tab: Cycle knobs
- Arrow Up/Down: Adjust (±5%)

**General:**
- Delete: Remove container
- Spacebar: Stop all tracks
- Click cell: Select
- Double-click cell: Play

### Library Tab

**Navigation:**
- Arrow Up/Down: Navigate list
- Enter: Load selected
- Delete: Delete selected
- F2: Rename (inline edit)

### Controls Tab

**Audio:**
- Spacebar: Play/pause
- [ ]: Set loop in
- ] ]: Set loop out
- Arrow Left/Right: Navigate beats

---

## WORKFLOW GUIDES

### DJ Set Preparation

**Goal:** Prepare 10-track set with visual timing

**Steps:**
1. Create/load 10 shaders
2. Add audio to each
3. Set BPM for each track
4. Click Update for each
5. Go to LIVE tab
6. Drag to grid (multiple scenes)
7. Set master tempo
8. Save project state
9. Ready to perform

### Quick Jam Session

**Goal:** Improvise with loops

**Steps:**
1. Load favorite containers to grid
2. Click cells to play tracks
3. Mix with volume knobs
4. Mute/Solo for arrangement
5. Auto-saves to default.omsp
6. Come back later, restores

### Learning Track Structure

**Goal:** Analyze track before mixing

**Steps:**
1. Load track to cell
2. Click to play
3. Watch beat grid scroll
4. Note phrase markers (magenta)
5. Note where drops happen
6. Plan mix points
7. Confident mixing

---

## TROUBLESHOOTING

### Beat Grid Not Showing

**Fix:**
1. Load container in Controls tab
2. Check "Audio Tempo" has BPM
3. Click "Update" button
4. Reload in LIVE tab

### Beat Grids Not Aligned

**Normal:** Each track's Bar 1 aligns at playhead
**If misaligned:** Hard refresh (Ctrl+Shift+R)

### Waveforms Not Scrolling Together

**Should be fixed in V3.2.0**
**If not:** Ensure using production release

### Audio Not Playing

**Check:**
- Audio file in LIBRARY folder
- Container has audio loaded
- AudioContext resumed (automatic)

### Master Tempo Not Working

**Check:**
- Track must be playing
- Container must have BPM set
- Click Update to save BPM

---

## ADVANCED FEATURES

### Control States

**Purpose:** Save render configurations

**Usage:**
1. Configure all settings
2. Click "Save Control State"
3. Enter name
4. Saves as {Name}_ControlState.oms
5. Load later from dropdown

**Examples:**
- HD_Render: 1920×1080, 60fps
- Quick_Preview: 1280×720, 30fps
- 4K_Final: 3840×2160, 24fps

### Frequency Mapping

**Purpose:** Define audio reactive frequency ranges

**Modulator A/B:**
- Freq 1: Low frequency (Hz)
- Freq 2: High frequency (Hz)
- Example: 30-50Hz for kick drum

**Ranges:**
- Kick: 30-50Hz
- Sub-bass: 50-75Hz
- Bass: 75-150Hz
- Mids: 150-4000Hz
- Highs: 4000-16000Hz

### Shader Timing

**Independent from Audio:**
- Shader Tempo: Visual loop speed
- Shader Loop Length: Bars per visual loop
- Audio BPM: Playback tempo
- Separate systems, limited integration

---

## TIPS & BEST PRACTICES

### DJ Mixing

**Use phrase markers:**
- Mix on magenta lines (4-bar phrases)
- Wait for phrase boundary
- Professional transitions

**Master tempo workflow:**
- Set master to your mixing tempo
- Load tracks with different BPMs
- All time-stretch to match
- Perfect sync

**Visual cues:**
- Watch waveforms scroll together
- See builds approaching
- Anticipate drops
- Confident mixing

### Container Organization

**Naming:**
- Descriptive names
- Include BPM if helpful
- Example: "Bass_Drop_128"

**Updating:**
- Always click Update after changes
- Preserves audio timing
- Beat grid stays working

**Backups:**
- .oms files are small
- Back up LIBRARY folder regularly
- Easy to restore

### Performance Tips

**Pre-load containers:**
- Audio decodes on drag
- No lag during performance

**Project States:**
- Save before performance
- Quick restore if crash
- Experiment freely

**Master tempo:**
- Find sweet spot first
- Some tracks sound better faster/slower
- Fine tune with keyboard

---

## BROWSER COMPATIBILITY

**Fully Supported:**
- Chrome 88+
- Safari 15.2+ (Technology Preview for WebGPU)
- Edge 88+

**Partially Supported:**
- Firefox (WebGPU only, no LIBRARY folder)

**Required:**
- WebGPU
- Web Audio API
- File System Access API
- CSS Transforms
- Canvas 2D

---

## GETTING HELP

**Documentation:**
- D_Release_Notes.md (what's new)
- C_Developer_Documentation.md (technical)
- B_Complete_Changelog.md (all iterations)
- E_Claude_ShaderCreation_Guide.md (shader guide)
- G_UserGuide_FFmpeg_Video_Guide.md (video conversion)

**Website:**
https://www.itswessmithyo.com/

**Support:**
- No paid support (open source)
- Documentation-first approach
- Community-driven

---

**END OF USER GUIDE**

Welcome to V3.2.0! Create, mix, perform - all with professional visual timing.
