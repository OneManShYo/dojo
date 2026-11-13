# OMS SHADER BAKER V3.0.0 - RELEASE NOTES

**Version:** V3.0.0  
**Release Date:** November 2025  
**Type:** Major Feature Release

---

## WHAT'S NEW

V3.0.0 adds the **LIVE tab** - a complete performance interface integrated directly into Shader Baker. This lays the foundation for real-time mixing of shader visuals and audio, eliminating the need for separate tools like ShaderBakerLive.

**Key Addition:**
- **NEW 5th Tab:** LIVE performance interface
- **UI Shell Complete:** Grid, mixer, waveforms all present
- **All V2.9.0 Features:** Fully preserved and working

**Current State:**
- LIVE tab is **UI shell only** (visual interface complete)
- **No functionality** yet (no audio playback, no shader rendering)
- Foundation for V3.1.0+ feature integration

---

## THE LIVE TAB

### What It Looks Like

**Performance Grid:**
- 4 tracks × 8 scenes = 32 clip cells
- Landscape aspect ratio (4:3) cells
- Scene trigger buttons (1-8)
- Track headers with status display
- Master column (5th column)

**Mixer Section:**
- 5 mixer columns (Track 1-4 + Master)
- Rotary volume knobs (drag to adjust)
- Mute [M] and Solo [S] buttons
- Vertical L/R stereo meters (placeholders)
- dB volume labels

**Waveform Section:**
- 4 horizontal waveform strips
- Viewing area for audio sync
- Placeholders for now

**Layout:**
- Fixed 1070px width (right panel)
- Optimized for 14" MacBook Pro
- App centered at 1470px total width
- Side borders (5px light grey)

---

## HOW TO USE (V3.0.0)

### Accessing the LIVE Tab

1. Open OMS Shader Baker V3.0.0
2. Click "LIVE" tab (5th tab button)
3. **Left side:** Library containers visible
4. **Right side:** LIVE performance interface

**What Works:**
- Tab switches properly
- Layout displays correctly
- Visual interface complete

**What Doesn't Work:**
- Can't load containers into grid yet
- Can't play audio yet
- Cells are empty placeholders
- Mixer controls don't affect audio

### Current Workflow

**For Now, Use Existing Workflow:**
1. Use Controls/Presets/Library/Code tabs (V2.9.0 features)
2. Create shaders, render videos
3. Export to ProRes for Ableton + EboSuite
4. LIVE tab is preview of future features

**Future Workflow (V3.1.0+):**
1. Create shaders in Code tab
2. Save as .oms containers in Library
3. Switch to LIVE tab
4. Assign containers to grid cells
5. Launch scenes, mix audio + visuals
6. Perform live without Ableton

---

## WHAT WORKS IN V3.0.0

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

### LIVE Tab UI Shell ✅

**Visual Interface:**
- 5th tab displays properly
- Performance grid layout (4 tracks × 8 scenes)
- Fixed track headers (always visible)
- Scrollable scene rows
- Fixed mixer section (140px height)
- Waveform section (220px height)

**Mixer Controls (Visual Only):**
- Rotary volume knobs (drag interaction works)
- Mute/Solo buttons (toggle states work)
- Vertical L/R meters (placeholders)
- Master column present

**Tab Integration:**
- LIVE tab shows Library on left
- Mixer displays on right (replaces canvas)
- Tab switching works properly
- No conflicts with existing features

---

## WHAT DOESN'T WORK YET

### LIVE Tab Functionality ❌

**Coming in V3.1.0+:**
- No container → grid assignment yet
- No audio playback yet
- No shader rendering in cells yet
- Scene launching doesn't trigger anything
- Mixer controls don't affect audio
- Meters don't animate (no audio data)
- Waveforms are placeholders

**Why UI Shell Only:**
- V3.0.0 establishes visual foundation
- V3.1.0 will add audio engine integration
- V3.2.0 will add shader rendering
- Iterative development approach

---

## MIGRATION FROM V2.9.0

### Simple Upgrade

**Nothing Changes:**
- All V2.9.0 features work identically
- Same LIBRARY folder system
- Same .oms file format
- Same keyboard shortcuts
- Same workflows

**What's New:**
- 5th LIVE tab button appears
- Click to see performance interface preview
- Everything else unchanged

**No Breaking Changes:**
- Existing containers load normally
- Audio files work the same
- Render settings preserved
- No re-setup required

### Setup Steps

1. **No Setup Needed:**
   - Download V3.0.0 HTML file
   - Open in browser (Chrome/Safari)
   - LIBRARY folder permission persists
   - All containers available immediately

2. **Explore LIVE Tab:**
   - Click LIVE tab
   - See performance layout
   - Understand future workflow
   - Continue using existing tabs for work

---

## BROWSER REQUIREMENTS

**Minimum:**
- Chrome 88+ (recommended)
- Safari 15.2+
- Edge 88+

**Required APIs:**
- WebGPU (shader rendering)
- Web Audio API (FFT analysis)
- File System Access API (LIBRARY folder)
- CSS Grid, Flexbox, aspect-ratio

**Firefox:**
- Partial support (WebGPU only)
- No File System Access API
- Use Chrome/Safari for full features

---

## KNOWN LIMITATIONS (V3.0.0)

### LIVE Tab

**UI Shell Only:**
- Visual interface complete
- No functionality implemented
- Preview of future features

**Fixed Layout:**
- 1470px total width (1070px + 400px)
- 4 tracks only (not scalable yet)
- 8 scenes only (not expandable yet)
- Non-responsive design

### Integration

**No Audio Playback:**
- Can't load audio into grid
- Mixer controls are visual only
- Meters don't animate
- No multi-track mixing yet

**No Shader Rendering:**
- Can't render shaders in cells
- Grid cells are empty placeholders
- No visual preview in grid yet

**No Scene Launching:**
- Scene buttons give visual feedback
- Don't trigger any playback
- No clip launching functionality

---

## FILE SIZE

**V3.0.0:** ~330KB
**V2.9.0:** 272KB
**Growth:** +58KB for LIVE tab UI

Still a single HTML file - fast, portable, offline-capable.

---

## PERFORMANCE

**No Performance Impact:**
- LIVE tab only initializes when first viewed
- No background processes
- Same performance as V2.9.0 for existing features
- UI-only additions

**Future (V3.1.0+):**
- Audio playback: +100MB memory per track
- Shader rendering: +50MB per track
- Total estimated: ~500MB with 4 tracks active

---

## QUICK START

### For Existing Users

1. **Download V3.0.0**
2. **Open in browser**
3. **Use normally** (Controls/Presets/Library/Code)
4. **Check out LIVE tab** (preview of future)
5. **Continue current workflow** until V3.1.0

### For New Users

1. **Download V3.0.0**
2. **Open in Chrome/Safari**
3. **Set Library Folder** (one-time setup)
4. **Create shaders** in Controls/Code tabs
5. **Render videos** for DJ performances
6. **LIVE tab** coming in future updates

---

## DEVELOPMENT ROADMAP

### V3.1.0 (Next Release)
**Timeline:** December 2025

**Features:**
- Container → grid cell assignment
- Audio file loading per cell
- Basic playback (play/pause per track)
- Real-time meter animation
- Scene launching triggers playback

### V3.2.0
**Timeline:** Q1 2026

**Features:**
- Shader rendering in grid cells (4 simultaneous)
- Visual compositing (blend modes)
- FFT data per track
- Audio-reactive shaders in real-time

### V3.3.0
**Timeline:** Q1 2026

**Features:**
- MIDI control (Midifighter Twister)
- Ableton Link synchronization
- Advanced crossfader
- Multi-page support (8+ tracks)

### V4.0.0: OMS Session
**Timeline:** Q2 2026

**Features:**
- Complete Ableton + EboSuite replacement
- Professional performance mixing
- Recording/export capabilities
- Advanced compositing modes
- "Anyone can be a OneManShYo"

---

## WORKFLOWS

### Current Workflow (V3.0.0)

**Shader Creation:**
1. Controls tab → Load preset or write shader
2. Code tab → Edit WGSL code
3. Drag audio from LIBRARY
4. Configure frequency mapping
5. Render video (WebM)
6. Convert to ProRes with FFmpeg
7. Import to Ableton + EboSuite

**Container Management:**
1. Library tab → Create/manage .oms files
2. Auto-save workflow
3. Keyboard navigation
4. Organize in LIBRARY folder

**LIVE Tab:**
1. Click LIVE tab → See future interface
2. Library on left (browse containers)
3. Performance grid on right (placeholder)
4. Wait for V3.1.0 functionality

### Future Workflow (V3.1.0+)

**Performance Setup:**
1. Create shaders in Code tab
2. Save as .oms containers
3. Switch to LIVE tab
4. Assign containers to grid cells
5. Audio loads automatically from containers

**Live Performance:**
1. Launch scenes (trigger 4 tracks)
2. Mix audio with volume knobs
3. Mute/Solo individual tracks
4. Shaders render live (audio-reactive)
5. Visual compositing to master output

**No External Tools Needed:**
- No Ableton Live
- No EboSuite
- No FFmpeg conversion
- Just browser + MIDI controller

---

## TROUBLESHOOTING

### LIVE Tab Not Showing

**Cause:** Browser compatibility  
**Fix:** Use Chrome 88+ or Safari 15.2+

**Cause:** JavaScript error  
**Fix:** Check browser console for errors

### Mixer Controls Not Working

**Expected:** V3.0.0 controls are visual only  
**Status:** Working as designed  
**Fix:** Wait for V3.1.0 audio integration

### Grid Cells Empty

**Expected:** V3.0.0 has no container loading  
**Status:** Working as designed  
**Fix:** Wait for V3.1.0 functionality

### Tab Switching Issues

**Cause:** Initialization error  
**Fix:** Refresh browser, check console

---

## GETTING HELP

**Documentation:**
- UserGuide_All.md (comprehensive guide)
- Developer_Documentation.md (technical reference)
- Complete_Changelog.md (iteration history)

**Support:**
- No paid support (open source project)
- "Ghost protocol" release philosophy
- Community-driven development

**Website:**
https://www.itswessmithyo.com/

---

## SUMMARY

V3.0.0 adds the LIVE tab - a complete performance interface UI shell integrated into Shader Baker. This is the foundation for future real-time mixing features that will replace the need for Ableton + EboSuite.

**For Now:**
- Use existing V2.9.0 features normally
- LIVE tab is preview of future
- Continue current DJ workflow

**Coming Soon (V3.1.0):**
- Load containers into grid
- Multi-track audio playback
- Real-time shader rendering
- Complete performance tool

**The Vision:**
"Anyone can be a OneManShYo" - solo DJ with live reactive visuals, no team required, all in the browser.

---

**Welcome to V3.0.0! The future of solo performance awaits!**
