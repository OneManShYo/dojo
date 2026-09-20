# OMS DOJO V2_9 - COMPLETE USER GUIDE

**Version:** V2_9  
**Last Updated:** November 2025  
**Purpose:** Complete user documentation

---

## TABLE OF CONTENTS

1. [Getting Started](#getting-started)
2. [User Interface Overview](#user-interface-overview)
3. [LIBRARY Folder Setup](#library-folder-setup-required)
4. [Controls Tab](#controls-tab)
5. [Presets Tab](#presets-tab)
6. [Library Tab](#library-tab)
7. [Code Tab](#code-tab)
8. [Keyboard Navigation](#keyboard-navigation)
9. [Rendering Videos](#rendering-videos)
10. [Workflows](#workflows)
11. [Troubleshooting](#troubleshooting)

---

## GETTING STARTED

### System Requirements

**Browser:** Chrome 88+, Safari 15.2+, or Edge 88+ (WebGPU + File System Access API required)  
**OS:** macOS, Windows, or Linux  
**Storage:** LIBRARY folder on your filesystem (unlimited size)

### Opening the Application

1. Download `OMS_Dojo_V2_9.html`
2. Double-click to open in default browser, or
3. Right-click → Open With → Chrome/Safari

**Note:** Firefox not supported (no File System Access API)

### First Launch

1. Application opens to Controls tab with default shader
2. Click **Compile** to see shader preview
3. **To save anything:** Must set up LIBRARY folder first

---

## USER INTERFACE OVERVIEW

### Layout

```
┌─────────────┬───────────────────────────────────┐
│  CONTROLS   │                                   │
│  PRESETS    │        CANVAS                     │
│  LIBRARY    │       (Shader Preview)            │
│  CODE       │                                   │
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
- **Presets** - Library shaders (5 built-in)
- **Library** - Container management (shader + audio + settings)
- **Code** - WGSL shader editor

### Main Buttons

- **Compile** - Compile shader and start preview
- **Stop** - Stop shader preview
- **Render** - Export video (WebM format)

---

## LIBRARY FOLDER SETUP (REQUIRED)

### Why LIBRARY Folder Required?

**V2_9 requires LIBRARY folder for all saves.** This creates a professional, file-based workflow like DAW software (Ableton Live, Logic Pro).

**Two Modes:**
1. **LIBRARY Set** - All saves to folder (persistent, professional)
2. **No LIBRARY** - Work + render only, nothing saves (temp session)

### Setting Up LIBRARY Folder

**One-Time Setup:**

1. Click **Library** tab
2. Click **"Set Library Folder"** button (bottom of tab)
3. Browser shows folder picker
4. Navigate to desired location (e.g., `~/Music/OMS_Library`)
5. Click **Select** (or **Open** depending on browser)
6. Grant permission

**Permission stored in browser forever** (IndexedDB). You won't be asked again unless you clear browser data.

### Adding Audio Files

After setting LIBRARY folder:

1. Navigate to folder in Finder/Explorer
2. Copy audio files (.mp3, .wav, .flac) into folder
3. Organize as you like (subfolders OK)
4. Files immediately available in application

**Recommended Structure:**
```
/OMS_Library
  ├── Audio/
  │   ├── track01.mp3
  │   ├── track02.mp3
  │   └── bass_loop.mp3
  └── (containers auto-save here as .oms files)
```

### What Gets Saved in LIBRARY

**Containers (.oms files):**
- Filename: `{ContainerName}.oms`
- Contains: shader code, settings, audio reference
- Example: `MyShader.oms`

**Control States (.oms files):**
- Filename: `{StateName}_ControlState.oms`
- Contains: render settings only
- Example: `QuickTest_ControlState.oms`

**Audio Files:**
- Your original files (.mp3, .wav, etc.)
- Referenced by containers

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
- Example: `00:30.000` = 30 seconds

### Shader Reactivity Section

**Modulator A, B, C** - Three independent audio-reactive modulators

**For Each Modulator:**

**Subdivision:**
- How often modulator updates
- Options: Whole, Half, Quarter, 8th, 16th, 32nd, 64th
- Example: Quarter = updates every 1/4 beat

**Waveform:**
- Shape of modulation
- Options: Sine, Sawtooth, Triangle, Square
- Affects visual rhythm

**Gain:**
- Amount of audio reactivity
- Range: 0.0 (none) to 50.0 (extreme)
- Default: 0.1 (subtle)

**Frequency Range (Hz):**
- Which audio frequencies trigger modulator
- Two fields: Min and Max
- Examples:
  - 30-50Hz: Kick drum
  - 50-75Hz: Sub-bass
  - 100-500Hz: Lead bass
  - 500-2000Hz: Mids
  - 2000-8000Hz: Highs

### Output Section

**Orientation:**
- Landscape (1920×1080)
- Portrait (1080×1920)
- Square (1080×1080)

**Resolution:**
- 720p (1280×720 or 720×1280)
- 1080p (1920×1080 or 1080×1920)
- 1440p (2560×1440 or 1440×2560)
- 2160p (3840×2160 or 2160×3840) - 4K

**Frame Rate:**
- 24 FPS (cinematic)
- 30 FPS (standard)
- 60 FPS (smooth)

### Control States

**What Are Control States?**
- Saved render configurations
- Reusable across any shader
- Quick switching between settings

**Library States (5 Built-In):**
1. Default
2. Quick Test (720p/30fps)
3. HD (1080p/60fps)
4. 4K (2160p/60fps)
5. Instagram (1080×1350/30fps)
6. TikTok (1080×1920/30fps)

**User States:**
- Your saved configurations
- Saved as `{Name}_ControlState.oms` in LIBRARY
- Appear below "--- User States ---" in dropdown

**Saving Control State:**
1. Configure all settings
2. Click **"Save State"** button
3. Enter name (e.g., "QuickTest")
4. Saves to LIBRARY folder
5. Appears in dropdown

**Loading Control State:**
1. Click dropdown
2. Select state
3. All settings apply immediately

---

## PRESETS TAB

### Library Shaders (5 Built-In)

**1. Tron Tunnel** - Classic tunnel effect with color cycling  
**2. Son of Tron Grid** - Animated grid with perspective  
**3. Lava Lamp** - Metaball-style organic blobs  
**4. Flower Power** - Kaleidoscopic mandala patterns  
**5. Retro Fish Farm** - Nostalgic ray-traced scene

**Loading Preset:**
- Click preset button
- Shader loads into editor
- Click **Compile** to see preview

---

## LIBRARY TAB

### Overview

Library tab manages containers - complete packages of shader + audio + settings.

**Buttons at Top:**
- **Add Current** - Save current work as container
- **Import** - Import .oms file from disk

**Container List:**
- Shows all saved containers
- Keyboard navigation (arrow keys)
- Active container marked with cyan dot (●)

**Container Buttons:**
- **L** - Load container
- **U** - Update container with current editor state
- **X** - Delete container (with confirmation)

**Bottom:**
- **Set Library Folder** - One-time setup (if not done)
- **Change Library** - Pick different folder
- Info box shows current LIBRARY folder path

### Adding Container

**Process:**
1. Create/edit shader in Code tab
2. Load audio file (optional)
3. Configure settings in Controls tab
4. Click **Library** tab
5. Click **"Add Current"** button
6. Enter container name
7. Container saves as `{Name}.oms` in LIBRARY

**Auto-Suggested Name:**
- With audio: `AudioName_ShaderName`
- Without audio: `ShaderName`
- Example: `track01_TronTunnel`

### Loading Container

**Mouse:**
- Click **L** button on container

**Keyboard:**
- Arrow Down/Up to navigate
- Enter to load

**What Loads:**
- Shader code → Code tab
- Settings → Controls tab
- Audio file → from LIBRARY folder
- Canvas compiles automatically
- Cyan dot (●) shows active container

### Updating Container

**After editing loaded container:**
1. Make changes (shader code, settings, audio)
2. Status shows: "⚠ Unsaved changes"
3. Click **U** button on same container
4. Container updates (auto-save)
5. Warning clears

### Renaming Container

**Process:**
1. Click container name
2. Input field appears
3. Edit name
4. Press Enter (or click away)
5. .oms file renames automatically

### Deleting Container

**Process:**
1. Click **X** button
2. Confirm deletion
3. .oms file deleted from LIBRARY
4. Container removed from list

---

## CODE TAB

### WGSL Shader Editor

**Textarea:**
- Full WebGPU Shading Language (WGSL) editor
- Syntax: similar to Rust/C
- Write fragment shaders (pixel shaders)

**Basic Structure:**
```wgsl
// ONEMANSHYO Dojo V2_9 - MyShader

@fragment
fn main(@location(0) fragCoord: vec2f) -> @location(0) vec4f {
    // Your shader code here
    return vec4f(1.0, 0.0, 1.0, 1.0); // Magenta
}
```

**Shader Name:**
- First line comment: `// ONEMANSHYO Dojo V2_9 - MyShader`
- Used for auto-suggested container names

**Compile:**
- Click **Compile** button (or Cmd/Ctrl + Enter)
- Errors show in status bar
- Success: shader runs in canvas

### Uniforms Available

**Timing:**
- `u.time` - Elapsed time (seconds)
- `u.beat` - Current beat number
- `u.bar` - Current bar number

**Modulators (Audio-Reactive):**
- `u.modA` - Modulator A value (0.0-1.0)
- `u.modB` - Modulator B value (0.0-1.0)
- `u.modC` - Modulator C value (0.0-1.0)

**Canvas:**
- `u.resolution` - Canvas size (vec2f)

**Example Usage:**
```wgsl
let pulse = u.modA * 2.0; // Kick drum pulse
let color = vec3f(u.modB, u.modC, 1.0); // Bass-reactive color
```

---

## KEYBOARD NAVIGATION

### Library Tab Navigation

**Auto-Focus:**
- Switch to Library tab → first container auto-focuses
- Ready for arrow keys immediately

**Arrow Keys:**
- Arrow Down → next container
- Arrow Up → previous container
- Hold arrow → rapid scroll

**Enter Key:**
- On container → Load container
- On button (after Tab) → Activate button

**Tab Key:**
- Tab → cycle through buttons (L/U/X)
- Shift+Tab → reverse

**Example Workflow:**
1. Click Library tab
2. Arrow Down 3 times
3. Enter → load container
4. Edit shader
5. Tab to U button
6. Enter → update container
7. Arrow Down → next container

**No mouse required!**

---

## RENDERING VIDEOS

### Render Process

1. Configure settings in Controls tab
2. Click **Render** button
3. Recording starts (canvas captures frames)
4. Progress shows in status bar
5. When complete, browser downloads WebM file

### Render Settings

**Resolution affects:**
- File size
- Render time
- Quality

**Recommendations:**
- **Testing:** 720p/30fps (fast)
- **Final:** 1080p/30fps (balanced)
- **High Quality:** 1080p/60fps or 4K

### File Naming

**Format:** `OMS_Dojo_YYYYMMDD_HHMMSS.webm`  
**Example:** `OMS_Dojo_20251112_143022.webm`

### Converting to ProRes

**For Ableton Live + EboSuite:**

WebM works but ProRes 422 performs better. Use FFmpeg to convert:

```bash
ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov
```

See **FFmpeg Video Guide** for detailed instructions.

---

## WORKFLOWS

### Basic Shader Creation

1. Click **Code** tab
2. Write or paste shader code
3. Click **Compile** (Cmd/Ctrl + Enter)
4. Preview appears in canvas
5. Adjust settings in **Controls** tab
6. Click **Render** to export video

### Container Workflow (Recommended)

**Create:**
1. Load preset from **Presets** tab
2. Tweak shader in **Code** tab
3. Load audio file
4. Configure reactivity in **Controls** tab
5. **Library** tab → **"Add Current"**
6. Enter name → saves as .oms file

**Load:**
1. **Library** tab → Arrow Down to container
2. Enter → loads instantly
3. Everything restored (shader + audio + settings)

**Update:**
1. Edit loaded container
2. Click **U** button
3. Auto-saves changes

### Audio-Reactive Workflow

1. Set up LIBRARY folder
2. Add audio files to folder
3. Create shader in **Code** tab
4. Drag audio file from LIBRARY
5. **Controls** tab → set frequency ranges:
   - Mod A: 30-50Hz (kick)
   - Mod B: 50-75Hz (sub-bass)
   - Mod C: 100-500Hz (lead bass)
6. Adjust gains (start with 0.1)
7. Click **Compile** → see reactivity
8. Fine-tune until perfect
9. **Library** tab → **"Add Current"**
10. Render video

### Performance Prep Workflow

**For Live DJ Sets:**

1. Create 10-20 shader variations
2. Save as containers in LIBRARY
3. Render each as video (WebM)
4. Convert to ProRes (FFmpeg)
5. Import to Ableton Live + EboSuite
6. Trigger with MIDI controller during set

---

## TROUBLESHOOTING

### "Set Library folder first to save"

**Problem:** Trying to save without LIBRARY folder set

**Solution:**
1. Library tab → "Set Library Folder"
2. Pick folder
3. Grant permission
4. Try save again

### "Audio not found: filename.mp3"

**Problem:** Container references audio file that doesn't exist

**Solution:**
1. Add audio file to LIBRARY folder
2. Or load container without audio (shader still works)
3. Like Ableton "Find Missing File"

### Container Not Loading

**Check:**
- LIBRARY folder permission granted?
- .oms file exists in folder?
- Filename matches container name?

**Solution:**
- Re-grant LIBRARY permission if needed
- Check LIBRARY folder for .oms file
- Permission may expire after long inactivity

### Keyboard Navigation Not Working

**Check:**
- Library tab active?
- Container focused (cyan border)?

**Solution:**
- Click Library tab
- First container should auto-focus
- Press Arrow Down to start

### Shader Won't Compile

**Common Errors:**

**"Entry point 'main' not found"**
- Missing `@fragment fn main()` function
- Check function signature

**"Unknown identifier"**
- Typo in variable name
- Check uniform names (u.time, u.modA, etc.)

**"Type mismatch"**
- Wrong type for operation
- Example: `vec3f + f32` (should be `vec3f + vec3f`)

### Firefox Not Working

**Problem:** File System Access API not supported

**Solution:** Use Chrome or Safari

**Workaround:** None - LIBRARY folder required in V2_9

### Render Hangs

**Problem:** Render stuck at "Recording..."

**Possible Causes:**
- Browser tab in background
- Computer went to sleep
- Insufficient memory

**Solution:**
- Keep browser tab active
- Close other applications
- Reduce resolution/FPS
- Try again

### Audio Not Loading

**Check:**
- LIBRARY folder set?
- Audio file in LIBRARY folder?
- Correct filename in container?

**Solution:**
- Verify LIBRARY folder path
- Add audio file if missing
- Check console for error messages

---

## KEYBOARD SHORTCUTS

### Global

- **Cmd/Ctrl + Enter** - Compile shader
- **Arrow Keys** - Adjust numeric fields (in Controls)
- **Tab** - Navigate between controls

### Library Tab

- **Arrow Down** - Next container
- **Arrow Up** - Previous container
- **Enter** - Load container
- **Tab** - Focus buttons (L/U/X)
- **Escape** - Cancel inline editing

### Audio Controls

- **Spacebar** - Play/Pause (when audio loaded)

---

## FILE FORMATS

### .oms Container Format

**Structure:**
```json
{
  "name": "MyShader",
  "shaderCode": "...",
  "controlStates": { ... },
  "audioFilename": "track01.mp3",
  "audioBPM": 128.0,
  "hasAudio": true,
  "created": "2025-11-11T20:55:54.123Z",
  "updated": "2025-11-11T21:30:12.456Z"
}
```

**Filename:** `{ContainerName}.oms`  
**Location:** LIBRARY folder  
**Portable:** Yes (copy to share)

### Control State Format

**Structure:**
```json
{
  "name": "QuickTest",
  "type": "controlState",
  "controlStates": { ... },
  "created": "2025-11-11T20:55:54.123Z"
}
```

**Filename:** `{StateName}_ControlState.oms`  
**Location:** LIBRARY folder  
**Reusable:** Across any shader

---

## TIPS & BEST PRACTICES

### Organization

**Naming:**
- Use descriptive container names
- Include version: "TronTunnel_v1", "TronTunnel_v2"
- Include audio reference: "track01_TronTunnel"

**LIBRARY Structure:**
```
/OMS_Library
  ├── Audio/
  │   ├── Kicks/
  │   ├── Bass/
  │   └── Leads/
  └── (containers auto-save at root)
```

### Performance

**Testing:**
- Use "Quick Test" Control State (720p/30fps)
- Fast iteration

**Final:**
- Use "HD" Control State (1080p/60fps)
- Render at target resolution

### Backup

**Regular:**
- Copy entire LIBRARY folder
- External drive or cloud storage
- Contains all .oms files + audio

**Before Major Changes:**
- Export containers as backup
- Save with version numbers

### Collaboration

**Sharing:**
1. Copy .oms file from LIBRARY
2. Copy audio file (if used)
3. Upload to Discord/GitHub
4. Others import and use

**Receiving:**
1. Download .oms + audio files
2. Copy to LIBRARY folder
3. Containers appear automatically

---

## GETTING HELP

**Documentation:**
- Release_Notes.md - What's new
- Developer_Documentation.md - Technical details
- Complete_Changelog.md - Version history
- FFmpeg_Video_Guide.md - Video conversion

**Website:**
https://www.itswessmithyo.com/

**License:** GPL-3.0 (free and open source)

---

## SUMMARY

**V2_9 Key Features:**
- LIBRARY folder required for saves
- Auto-save .oms files (DAW workflow)
- Keyboard-first navigation
- Container management (shader + audio + settings)
- Control States as .oms files
- Professional file-based workflow

**Recommended Workflow:**
1. Set up LIBRARY folder (one-time)
2. Add audio files to folder
3. Create shaders, save as containers
4. Keyboard navigate library
5. Render videos for performance

---

**Happy shader creating!**
