# OMS SHADER BAKER V3.1.0 - RELEASE NOTES

**Version:** V3.1.0  
**Release Date:** November 2025  
**Type:** Major Feature Release

---

## WHAT'S NEW

V3.1.0 transforms the LIVE tab from a visual mockup (V3.0.0) into a **complete, functional performance system**. This is the difference between a prototype and a professional tool.

**Key Additions:**
- **LIVE Tab Now Fully Functional** - Multi-track audio, mixing, visual feedback
- **Project States System** - Session persistence with .omsp files
- **Real-Time Performance** - 4-track mixing with professional controls
- **All V2.9.0 Features** - Fully preserved and working

**Current State:**
- LIVE tab is **production-ready** for solo DJ performances
- Complete audio routing with Web Audio API
- Real waveform visualization with playhead
- Professional hardware mixer aesthetic

---

## THE LIVE TAB - NOW FUNCTIONAL

### Performance Grid

**What Works:**
- 4 tracks × 8 scenes = 32 clip cells
- Drag containers from Library to grid
- Drag containers cell-to-cell
- Shader thumbnails (WebGPU rendered)
- Click cell to play audio
- Click empty cell to stop audio
- Green borders show playing cells
- Delete key removes selected container
- Hover shows container name tooltip

**Visual Features:**
- 16:9 landscape cells
- Shader thumbnails render immediately
- Hardware mixer aesthetic
- Professional grid layout

### Mixer Section

**What Works:**
- Rotary volume knobs (drag to adjust, 0-100%)
- Mute buttons (click M, turns orange when active)
- Solo buttons (click S, turns green when active)
- LED meters (animated with real-time FFT data)
- Master volume control
- dB labels update in real-time

**Audio Features:**
- Volume knobs at 12 o'clock (50%) by default
- 7 o'clock = 0%, 5 o'clock = 100%
- Mute silences track (gain = 0)
- Solo isolates track (all others muted)
- Multiple solos work (all solo'd tracks audible)
- Mute + Solo: Mute wins

### Waveform Section

**What Works:**
- 4 horizontal waveform strips (52px each)
- Real waveforms from audio data (not placeholders)
- Magenta playhead (fixed at center, 50%)
- Waveforms position Bar 1 at playhead on load
- Waveforms scroll during playback (tape machine style)
- Mono audio display (cyan on black)
- Full height section (240px total)

**Visual Design:**
- Classic DAW transport view
- Fixed playhead, scrolling waveforms
- Professional audio editing aesthetic

### Keyboard Controls

**What Works:**
- **Tab/Shift+Tab:** Cycle through volume knobs
- **Arrow Up/Down:** Adjust selected knob (5% per press)
- **Delete:** Remove selected container from cell
- **Spacebar:** Stop all playing tracks (LIVE tab)
- **Click cell:** Select for deletion (yellow outline)
- **Click knob:** Select for keyboard control (cyan ring)

**Tab-Aware:**
- Spacebar in LIVE stops tracks
- Spacebar in Controls plays/pauses preview
- Clean separation of tab functionality

### Project States

**NEW System for Session Persistence:**
- Location: Controls tab (bottom, after Control States)
- Auto-saves to default.omsp instantly on any change
- Manual save format: {Name}_YYYY-MM-DD_01.omsp
- Load project states from dropdown
- Delete project states (except default)
- Info box shows project details
- Restores: grid assignments, volumes, mutes, solos, tempo

**What Gets Saved:**
- All 32 grid cell assignments
- Track volumes (4 tracks)
- Track mute states
- Track solo states
- Master volume
- Master tempo
- Last scene launched

---

## HOW TO USE (V3.1.0)

### Quick Start - LIVE Tab

**1. Set Up LIBRARY**
   - Click "Set Library Folder" (Controls tab)
   - Select your LIBRARY folder
   - Permission persists between sessions

**2. Create Containers**
   - Go to Library tab
   - Click "Add Current" to save shader + audio
   - Containers appear in list

**3. Load Performance Grid**
   - Go to LIVE tab
   - Drag containers from left panel to grid cells
   - Shader thumbnails render automatically

**4. Play Audio**
   - Click cell to play audio
   - Click empty cell to stop
   - Adjust volume knobs (drag or arrow keys)
   - Click M to mute, S to solo

**5. Save Session**
   - Changes auto-save to default.omsp
   - Manual save: Controls tab → Project States → Save
   - Enter name, saves as {Name}_YYYY-MM-DD_01.omsp

### Traditional Workflow Still Works

**For Video Export:**
1. Controls tab → Load preset or write shader
2. Code tab → Edit WGSL code
3. Drag audio from LIBRARY
4. Configure frequency mapping
5. Render video (WebM)
6. Convert to ProRes with FFmpeg
7. Import to Ableton + EboSuite

**Nothing Changed:**
- All V2.9.0 features work identically
- Same keyboard shortcuts
- Same LIBRARY system
- Same render settings

---

## WHAT WORKS IN V3.1.0

### All V2.9.0 Features ✅

**LIBRARY-First Architecture:**
- LIBRARY folder required for saves
- .oms file system (containers + control states)
- Auto-save workflow
- Professional file management

**Shader Creation:**
- WebGPU shader rendering
- FFT audio analysis (32768 samples)
- User-definable frequency mapping
- Modulator system
- Video export (WebM VP8)

**Container Management:**
- Add/update/delete containers
- Keyboard-first navigation
- Active container indicators
- Inline editable names
- Dirty state tracking

**Control System:**
- Control States as .oms files
- 5 library presets + user presets
- Render configuration management

### LIVE Tab Complete System ✅

**Performance Grid:**
- 4 tracks × 8 scenes (32 cells)
- Container drag-drop (Library → Grid, Cell → Cell)
- Shader thumbnails (WebGPU rendered)
- Click to play/stop audio
- Green borders show playing cells
- Delete key removes containers
- Hover tooltips show names

**Audio Playback:**
- Multi-track playback (4 simultaneous tracks)
- Web Audio routing (sources → gains → master → destination)
- Looping playback
- Clean audio stop (click empty cell)
- AudioContext resume (browser security handled)

**Mixer Controls:**
- Rotary volume knobs (drag interaction, 0-100%)
- Mute buttons (functional, orange when active)
- Solo buttons (functional, green when active)
- LED meters (animated with FFT data)
- Master volume control
- Real-time audio level display

**Waveforms:**
- Real waveforms (from audio data)
- 4 track strips (52px each)
- Magenta playhead (fixed at center)
- Bar 1 positioned at playhead on load
- Scrolling during playback (60fps)
- Mono display (cyan on black)

**Project States:**
- Auto-save to default.omsp (instant)
- Manual save with date formatting
- Load/Delete project states
- Dropdown shows available projects
- Complete session restoration

**Keyboard Controls:**
- Tab/Shift+Tab (cycle knobs)
- Arrow Up/Down (adjust volume)
- Delete (remove container)
- Spacebar (stop all tracks)
- Click selection (cells and knobs)

**Hardware Aesthetic:**
- 10px side borders (mixer case feel)
- Professional color scheme
- Rotary knobs (7-5 o'clock range)
- LED meters (green/yellow/red)
- Magenta playhead

---

## WHAT DOESN'T WORK YET

### LIVE Tab (Planned V3.2.0+) ❌

**Visual Rendering:**
- Shader rendering in cells (thumbnails only, not animated)
- Master output compositing (no visual canvas)
- Visual blend modes (additive, multiply, etc.)
- Real-time shader animation

**Performance Features:**
- Scene launching (rows don't trigger together)
- Crossfading between cells
- Multi-page support (8+ tracks)
- Recording/export from LIVE tab

**Advanced Control:**
- MIDI control integration (Web MIDI API)
- Ableton Link synchronization
- Hardware controller mapping
- Advanced transport controls

---

## MIGRATION FROM V3.0.0

### Simple Upgrade

**Nothing Changes:**
- All V3.0.0 features work identically
- Same LIBRARY folder system
- Same .oms file format
- Same keyboard shortcuts
- Same workflows

**What's New:**
- LIVE tab now functional (was UI shell only)
- Project States system in Controls tab
- Multi-track audio playback
- Mixing controls work
- Waveforms display real data
- Keyboard navigation enhanced

**No Breaking Changes:**
- Existing containers load normally
- Audio files work the same
- Render settings preserved
- No re-setup required

### Setup Steps

1. **No Setup Needed:**
   - Download V3.1.0 HTML file
   - Open in browser (Chrome/Safari)
   - LIBRARY folder permission persists
   - All containers available immediately

2. **Use LIVE Tab:**
   - Go to LIVE tab
   - Drag containers to grid
   - Click cells to play
   - Adjust volumes, mute/solo
   - Auto-saves to default.omsp

3. **Traditional Workflow:**
   - Use Controls/Presets/Library/Code tabs
   - Create shaders, render videos
   - Export to ProRes for Ableton
   - Same as before

---

## BROWSER REQUIREMENTS

**Minimum:**
- Chrome 88+ (recommended)
- Safari 15.2+
- Edge 88+

**Required APIs:**
- WebGPU (shader rendering)
- Web Audio API (audio playback + FFT analysis)
- File System Access API (LIBRARY folder)
- CSS Grid, Flexbox, aspect-ratio
- Canvas 2D context

**Firefox:**
- Partial support (WebGPU only)
- No File System Access API
- Use Chrome/Safari for full features

---

## KNOWN LIMITATIONS (V3.1.0)

### LIVE Tab

**No Visual Rendering:**
- Cells show shader thumbnails only
- No animated shader rendering in cells
- No master output canvas
- Planned for V3.2.0

**No Scene Launching:**
- Individual cells trigger
- Rows don't launch together
- Manual per-track triggering only
- Planned for V3.3.0

**Fixed Layout:**
- 4 tracks only (not scalable yet)
- 8 scenes only (not expandable yet)
- 1470px total width
- Non-responsive design

### Integration

**No MIDI Control:**
- Keyboard only (no hardware controllers)
- Web MIDI API not implemented
- Planned for V3.4.0

**No Ableton Link:**
- No tempo sync with DAWs
- Internal tempo only
- Planned for V3.4.0

---

## FILE SIZE

**V3.1.0:** ~390KB  
**V3.0.0:** 330KB  
**Growth:** +60KB for complete LIVE functionality

Still a single HTML file - fast, portable, offline-capable.

---

## PERFORMANCE

**No Performance Impact on Existing Features:**
- Controls/Presets/Library/Code tabs identical
- Same shader rendering speed
- Same video export speed

**LIVE Tab Performance:**
- Audio playback: ~50MB memory per track
- Shader thumbnails: ~2MB for all 32 cells
- Total memory: ~300MB with 4 tracks active
- Acceptable for modern systems

**Smooth Operation:**
- 60fps waveform scrolling
- Real-time meter animation
- No audio glitches or dropouts
- GPU-accelerated rendering

---

## NEW WORKFLOWS

### Solo Performance Setup

**Scenario:** DJ set with live reactive visuals

**Setup:**
1. Create shaders in Code tab (audio-reactive)
2. Save as .oms containers with audio
3. Load containers into LIVE grid
4. Arrange tracks and scenes
5. Save as project state (e.g., "Friday_Set")

**Performance:**
1. Load "Friday_Set.omsp"
2. Click cells to trigger tracks
3. Mix with volume knobs
4. Mute/Solo for arrangement
5. Watch waveforms and meters
6. Professional DJ set with visuals

**No External Tools:**
- No Ableton Live
- No EboSuite
- No FFmpeg conversion
- Just browser + MIDI controller (future)

### Quick Jam Session

**Scenario:** Improvise with shader + audio loops

**Workflow:**
1. Load favorite containers into grid
2. Click Track 1, Scene 1 → bass loop plays
3. Click Track 2, Scene 1 → lead loop plays
4. Click Track 3, Scene 1 → drum loop plays
5. Mix with knobs, mute/solo
6. Auto-saves to default.omsp
7. Come back later, session restored

**Instant Gratification:**
- No setup time
- No prep work
- Just play and mix
- Auto-saves everything

---

## TROUBLESHOOTING

### LIVE Tab Audio Not Playing

**Cause:** AudioContext suspended (browser security)  
**Fix:** Automatic - V3.1.0 calls audioContext.resume()

**Cause:** Audio file not found  
**Fix:** Check LIBRARY folder has audio files

**Cause:** Container has no audio  
**Fix:** Add audio to container in Library tab

### Volume Knobs Not Moving

**Cause:** Canvas 2D context issue  
**Fix:** Refresh browser, check console for errors

**Cause:** Not dragging on knob  
**Fix:** Click and drag center of knob

### Waveforms Not Showing

**Cause:** Container has no audio loaded  
**Fix:** Verify audio file in LIBRARY folder

**Cause:** Canvas height issue  
**Fix:** V3.1.0 fixes this (was V3.1.0_N bug)

### LIBRARY Folder Not Persisting

**Cause:** Browser permission not requested  
**Fix:** V3.1.0_J fixes this (auto-requests permission)

**Cause:** Browser privacy settings  
**Fix:** Allow File System Access for this site

---

## GETTING HELP

**Documentation:**
- UserGuide_All.md (comprehensive guide with LIVE tab)
- Developer_Documentation.md (technical reference)
- Complete_Changelog.md (all 22 iterations)

**Support:**
- No paid support (open source project)
- "Ghost protocol" release philosophy
- Community-driven development

**Website:**
https://www.itswessmithyo.com/

---

## SUMMARY

V3.1.0 makes the LIVE tab **production-ready**. This isn't a prototype or a demo - it's a complete, professional performance system for solo DJ creators.

**Ready to Use:**
- Multi-track audio mixing ✅
- Real-time visual feedback ✅
- Session persistence ✅
- Professional controls ✅
- Keyboard navigation ✅

**Coming Soon (V3.2.0):**
- Real-time shader rendering
- Master output compositing
- Visual blend modes
- Scene launching

**The Vision:**
"Anyone can be a OneManShYo" - solo DJ with live reactive visuals, no team required, all in the browser.

---

**Welcome to V3.1.0! The future of solo performance is here!**
