# OMS DOJO V2_9 - RELEASE NOTES

**Version:** V2_9  
**Release Date:** November 2025  
**Type:** Major Architecture Release

---

## WHAT'S NEW

V2_9 fundamentally changes how ONEMANSHYO Dojo manages files, requiring a LIBRARY folder for all saves. This creates a professional, file-based workflow matching DAW standards like Ableton Live.

**Key Changes:**
1. **LIBRARY Folder Required** - All saves go to LIBRARY folder as .oms files
2. **Auto-Save Workflow** - DAW-style auto-save (no manual export)
3. **Keyboard-First Navigation** - Professional keyboard shortcuts
4. **Container Name = Filename** - Predictable 1:1 file naming

**Removed Features (Temporarily):**
- BroadcastChannel integration → Will return in LIVE tab (V2.10.0)
- Audio pitch control → Will return in LIVE tab (V2.10.0)

---

## THE LIBRARY-FIRST PHILOSOPHY

### What Changed

**V2.8.0 (Old):**
- Containers stored in localStorage (browser memory)
- Audio files optionally referenced from folder
- Manual export to .oms files
- Browser memory limits (~100 containers)

**V2_9 (New):**
- LIBRARY folder REQUIRED for all saves
- Everything stored as .oms files
- Auto-save on Add/Update (like Ableton .als files)
- No browser memory limits (file system only)

### Two Modes of Operation

**Mode 1: LIBRARY Set (Persistent)**
- Set Library folder once (File System Access API)
- All saves go to LIBRARY as .oms files
- Survives browser sessions, restarts, reboots
- Professional workflow (recommended)
- Same files accessible across tabs/windows

**Mode 2: No LIBRARY (Temp Session)**
- Work + render only
- Nothing saves
- Close browser = lose everything
- Use case: quick experiments, testing, demos

---

## QUICK START

### First-Time Setup

1. **Open Application**
   - Open OMS_Dojo_V2_9.html in Chrome/Safari

2. **Set Library Folder (One-Time)**
   - Click Library tab
   - Click "Set Library Folder" button
   - Pick folder (e.g., ~/Music/OMS_Library)
   - Grant permission
   - Browser remembers forever (IndexedDB)

3. **Add Audio Files**
   - Put audio files (.mp3, .wav) in LIBRARY folder
   - Organize as you like (subfolders OK)

4. **Create Your First Container**
   - Controls tab → Load preset or write shader
   - Drag audio file from LIBRARY folder
   - Configure settings
   - Library tab → "Add Current" button
   - Enter name: "MyShader"
   - Result: MyShader.oms created in LIBRARY folder

5. **Load Container**
   - Library tab → Click L button on container
   - Shader + audio + settings load instantly
   - Cyan dot (●) shows active container

### Keyboard Workflow

1. **Navigate Library**
   - Switch to Library tab (first card auto-focuses)
   - Arrow Down/Up → scroll through entries (hold for rapid)
   - Enter → load selected container
   - Tab → focus buttons (L/U/X)
   - Enter → activate button

2. **No Mouse Required**
   - Entire library navigable with keyboard
   - Professional speed
   - Works with 100+ entries

---

## KEY FEATURES

### 1. Auto-Save .oms Files

**DAW-Style Workflow:**
- Click "Add Current" → saves immediately
- Click "U" (Update) → overwrites immediately
- No export dialogs
- No manual file management
- Like Ableton .als files

**File Naming:**
- Container name: "MyShader"
- .oms filename: "MyShader.oms" (1:1 match)
- Audio filename: "track01.mp3" (stored in .oms JSON)

**Example:**
```
/LIBRARY
  ├── MyShader.oms (shader + settings + audio reference)
  └── track01.mp3 (audio file)
```

### 2. Control States as .oms Files

**Before (V2.8.0):**
- Control States in localStorage (browser memory)
- Lost when clearing browser data
- Not portable

**After (V2_9):**
- Control States in LIBRARY folder
- Filename: {Name}_ControlState.oms
- Portable across machines
- Example: "QuickTest_ControlState.oms"

**Usage:**
1. Configure render settings (720p, 30fps, etc.)
2. Click "Save State" button
3. Enter name: "QuickTest"
4. Result: QuickTest_ControlState.oms created
5. Appears in "Control States" dropdown
6. Reusable across any shader

### 3. Keyboard-First Navigation

**Auto-Focus:**
- Switch to Library tab → first card focuses automatically
- Ready for arrow keys immediately

**Navigation:**
- Arrow Up/Down → navigate entries
- Hold arrow → rapid scroll
- Enter → load container
- Tab → focus buttons
- Enter → activate button

**Speed:**
- 100+ entries = smooth
- No mouse required
- Professional workflow

### 4. Active Container Indicator

**Visual Feedback:**
- Cyan dot (●) left of L button
- Shows which container is loaded
- Persistent across navigation
- Updates on load/update
- Clears on delete

**Example:**
```
[● MyShader]  [L] [U] [X]  ← Active (loaded)
[OtherShader] [L] [U] [X]  ← Not active
```

### 5. Inline Name Editing

**Professional UX:**
- Click container name → becomes input field
- Text auto-selected
- Enter to save, Escape to cancel
- Click away (blur) to save
- No browser prompt dialogs

### 6. Dirty State Tracking

**Unsaved Changes:**
- Tracks shader code edits
- Tracks control changes
- Magenta warning when loading with unsaved changes
- Non-blocking (warns but allows)

**Example:**
```
Status: "⚠ Unsaved changes - loading anyway"
```

### 7. Audio Loading & Persistence

**Automatic:**
- Load container → audio loads from LIBRARY automatically
- No manual file loading
- Graceful error if audio missing

**Persistent:**
- Update container → preserves audio reference
- Audio filename stored in .oms JSON
- Safe null checks prevent crashes

---

## MIGRATION FROM V2.8.0

### What's Removed (Temporarily)

**BroadcastChannel Integration:**
- Instance announcements → Removed
- Canvas snapshots → Removed
- DojoLive communication → Removed
- **Will return in LIVE tab (V2.10.0)**

**Audio Pitch Control:**
- ±16% pitch adjustment → Removed
- Real-time playback rate control → Removed
- **Will return in LIVE tab (V2.10.0)**

### What's Preserved

**All Core Features:**
- WebGPU shader rendering → ✓
- FFT audio analysis → ✓
- Frequency mapping → ✓
- Control States → ✓ (now in LIBRARY)
- Preset system → ✓
- Audio loop controls → ✓
- Video export → ✓
- Modulator system → ✓

### Migration Steps

**If Coming from V2.8.0:**

1. **Export Your Containers (V2.8.0)**
   - Library tab → Export all containers
   - Downloads .oms files

2. **Open V2_9**
   - Open OMS_Dojo_V2_9.html

3. **Set Library Folder**
   - Library tab → "Set Library Folder"
   - Pick folder (e.g., ~/Music/OMS_Library)

4. **Add Audio Files**
   - Copy audio files to LIBRARY folder

5. **Import Containers**
   - Library tab → "Import" button
   - Select exported .oms files
   - Containers appear in library

6. **Ready to Work**
   - All containers now in LIBRARY
   - Auto-save enabled
   - Keyboard navigation works

---

## FILE FORMAT

### Container .oms Structure

```json
{
  "id": "1731234567890",
  "name": "MyShader",
  "shaderCode": "// Tron Tunnel v2.8.3c\n...",
  "controlStates": {
    "tempo": 120,
    "loopLength": 4,
    "modASubdiv": 1,
    "modAWaveform": "sine",
    "modAGain": 0.1,
    "modAFreq1": 30,
    "modAFreq2": 50,
    // ... 21 total values
  },
  "audioFilename": "track01.mp3",
  "audioBPM": 128.0,
  "hasAudio": true,
  "created": "2025-11-11T20:55:54.123Z",
  "updated": "2025-11-11T21:30:12.456Z"
}
```

### Control State .oms Structure

```json
{
  "id": "1731234567890",
  "name": "QuickTest",
  "type": "controlState",
  "controlStates": {
    // Same 21 values as container
  },
  "created": "2025-11-11T20:55:54.123Z"
}
```

### Filename Convention

**Container:**
- Name: "MyShader"
- File: "MyShader.oms"

**Control State:**
- Name: "QuickTest"
- File: "QuickTest_ControlState.oms"

**1:1 Match = Predictable**

---

## BROWSER REQUIREMENTS

### Full Support

**Chrome 88+**
- All features working
- Recommended

**Safari 15.2+**
- All features working
- File System Access API since 15.2

**Edge 88+**
- All features working
- Chromium-based

### Partial Support

**Firefox**
- WebGPU: ✓ (Nightly)
- File System Access API: ✗ (not supported)
- **Use Chrome/Safari for full functionality**

### Required APIs

- WebGPU (shader rendering)
- Web Audio API (FFT analysis)
- File System Access API (LIBRARY folder)
- IndexedDB (permission storage)

---

## WORKFLOWS

### Daily Creator Workflow

**Create Shader:**
1. Controls tab → Load preset
2. Code tab → Tweak shader
3. Drag audio from LIBRARY
4. Configure frequency mapping
5. Library tab → "Add Current"
6. Container saved as .oms file

**Load Shader:**
1. Library tab → Arrow Down to container
2. Enter key → loads instantly
3. Cyan dot shows active
4. Ready to work

**Update Shader:**
1. Edit shader code
2. Magenta warning: "⚠ Unsaved changes"
3. Library tab → Click U on active container
4. Container updated (auto-save)

**Keyboard Workflow:**
- No mouse required
- Arrow keys navigate
- Enter loads
- Tab to buttons
- Professional speed

### Performance Prep Workflow

**Prep Phase:**
1. Create 10 shader variations
2. Save as containers (.oms files)
3. Export containers (backup)
4. Render videos (WebM)
5. Convert to ProRes (FFmpeg)

**Performance Phase:**
1. Import to Ableton Live + EboSuite
2. Trigger with MIDI controller
3. Mix with DJ audio tracks

### Community Sharing Workflow

**Share:**
1. Create amazing shader
2. Library tab → right-click container
3. Copy .oms file from LIBRARY folder
4. Upload to Discord/GitHub

**Receive:**
1. Download .oms file
2. Copy to LIBRARY folder
3. Library refreshes automatically
4. Load and enjoy

---

## TROUBLESHOOTING

### "Set Library folder first to save"

**Cause:** No LIBRARY folder configured

**Solution:**
1. Library tab → "Set Library Folder"
2. Pick folder
3. Grant permission

### "Audio not found: filename.mp3"

**Cause:** Audio file missing from LIBRARY folder

**Solution:**
1. Add audio file to LIBRARY folder
2. Or click L again (loads shader without audio)

**Like Ableton:** "Find Missing File" workflow

### Container Not Loading

**Check:**
- LIBRARY folder set? (permission may expire)
- .oms file exists in LIBRARY?
- Filename matches container name?

**Solution:**
- Re-grant LIBRARY permission if needed
- Check LIBRARY folder for .oms file

### Keyboard Navigation Not Working

**Check:**
- Library tab active?
- Card focused (cyan border)?

**Solution:**
- Click Library tab
- First card should auto-focus
- Press Arrow Down to start navigating

### Firefox - File System Access API Error

**Cause:** Firefox doesn't support File System Access API

**Solution:** Use Chrome or Safari

**Workaround:** None - LIBRARY folder required

---

## PERFORMANCE TIPS

### File Organization

**Recommended Structure:**
```
/LIBRARY
  ├── Containers/
  │   ├── Bass_Tunnel.oms
  │   ├── Lava_Lamp.oms
  │   └── Tron_Grid.oms
  ├── ControlStates/
  │   ├── QuickTest_ControlState.oms
  │   └── HD_Settings_ControlState.oms
  └── Audio/
      ├── track01.mp3
      └── track02.mp3
```

**Note:** App scans entire LIBRARY folder (including subfolders)

### Backup Strategy

**Regular Backups:**
- Copy entire LIBRARY folder
- External drive or cloud storage
- Contains all .oms files + audio

**Version Control:**
- Name containers with versions: "MyShader_v1", "MyShader_v2"
- Export important .oms files separately
- Archive old versions

### Performance

**Large Libraries (100+ containers):**
- Keyboard navigation handles smoothly
- Smooth scrolling with arrow keys
- No lag with proper hardware

**Rendering:**
- Test with "Quick Test" Control State first
- Render at target resolution
- Close other tabs during render

---

## WHAT'S NEXT (V2.10.0+)

### LIVE Tab Integration

**Coming in V2.10.0:**
- 5th LIVE tab added to Dojo
- Performance grid (4 tracks × 8 scenes)
- Mixer section (4 tracks + master)
- Real-time shader rendering (4 simultaneous)
- Real-time audio mixing
- Visual compositing
- Load containers from LIBRARY into grid

**Restored Features:**
- BroadcastChannel (for instance discovery)
- Pitch control (for tempo sync)
- FFT analysis per track
- Scene launching

### Future Enhancements

**V2.11.0+:**
- Container thumbnail previews
- Sort/filter containers
- Batch operations
- Subfolder organization
- MIDI control integration
- Ableton Link sync

---

## GETTING HELP

**Documentation:**
- UserGuide_All.md - Comprehensive guide
- Developer_Documentation.md - Technical reference
- Complete_Changelog.md - Version history
- FFmpeg_Video_Guide.md - Video conversion

**Website:**
https://www.itswessmithyo.com/

**Requirements:**
- Chrome 88+ or Safari 15.2+
- WebGPU support
- File System Access API

---

## SUMMARY

V2_9 is a major architecture update requiring LIBRARY folder for all saves. This creates a professional, file-based workflow matching DAW standards.

**Key Benefits:**
- No browser memory limits
- Portable .oms files
- Auto-save workflow
- Keyboard-first navigation
- Professional file management

**Trade-offs:**
- LIBRARY folder required (no temp saves)
- Chrome/Safari only (no Firefox)
- BroadcastChannel removed (returns in V2.10.0)
- Pitch control removed (returns in V2.10.0)

**Recommendation:**
Upgrade if you need professional file management and have Chrome/Safari. Wait for V2.10.0 if you need BroadcastChannel or pitch control.

---

**Welcome to V2_9! Professional shader management awaits!**
