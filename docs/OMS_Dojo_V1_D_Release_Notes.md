# OMS Shader Baker Live V1.0.0 - Release Guide

**Release Date:** November 2025  
**Version:** 1.0.0  
**Status:** Foundation Release

---

## Executive Summary

ShaderBakerLive V1.0.0 is the foundational UI shell and communication layer for the future OMS Session performance tool. This release establishes the visual interface, BroadcastChannel integration with Shader Baker instances, and manual instance discovery/assignment workflows.

**What's Included:**
- Complete performance grid UI (4 tracks × 8 scenes)
- Hardware-style mixer controls (rotary knobs, vertical meters)
- Instance discovery via BroadcastChannel
- Canvas snapshot display in grid cells
- Professional fixed-width layout (1200px)

**What's NOT Included:**
- No audio playback engine
- No FFT data broadcasting
- No clip launching functionality
- No MIDI control

---

## Release Contents

### Main Application
**File:** ShaderBakerLive_V1_0_0.html  
**Size:** ~37KB  
**Type:** Single HTML file (no dependencies)  
**Requirements:** Modern browser with BroadcastChannel support

### Documentation
1. A_Deliverables.txt - Package inventory
2. B_Complete_Changelog.md - Development history (A-P iterations)
3. C_Developer_Documentation.md - Technical reference
4. D_Release_Guide.md - This file
5. E_Integration_Guide.md - Shader Baker integration
6. UserGuide_All.md - Complete user guide
7. Z_Project_Instructions.md - Claude AI development context
8. OMS_Ecosystem_Architecture_Summary.md - System overview

### Iteration Notes
15 detailed iteration notes files (A-P, N skipped) documenting each development step.

---

## What Works in V1.0.0

✓ **UI Shell Complete**
- Performance grid with 4 tracks × 8 scenes
- Fixed track headers (always visible)
- Scrollable scene rows
- Fixed mixer section (always accessible)
- Professional hardware aesthetic

✓ **Instance Discovery**
- Auto-discovers Shader Baker instances via BroadcastChannel
- Manual track assignment (dropdown per track)
- Multi-instance support
- Heartbeat-based instance management (30sec timeout)

✓ **Visual Integration**
- Receives canvas snapshots from Shader Baker
- Displays shader visuals in grid cells
- Real-time snapshot updates
- 4:3 landscape aspect ratio (matches Baker canvas)

✓ **Mixer Controls (UI Only)**
- Rotary volume knobs (canvas-based)
- Mute/Solo buttons per track
- Vertical L/R stereo meters (placeholders)
- dB label display

✓ **Status Messaging**
- Footer status bar with color states
- Scene launch feedback
- Connection status indicators

---

## What Doesn't Work Yet

❌ **No Audio Engine**
- No audio file loading
- No playback functionality
- No tempo/beat tracking
- Meters are visual placeholders only

❌ **No Data Broadcasting**
- Doesn't send FFT data to Shader Baker
- Doesn't send tempo/beat data
- One-way communication only (receives, doesn't send)

❌ **No Performance Features**
- Clip launching is UI only (no playback)
- Scene launching is UI only (no clips triggered)
- No audio-reactive functionality
- No clip library/storage

❌ **No Advanced Features**
- No MIDI control integration
- No Ableton Link sync
- No .oms file support
- No recording/export

---

## Browser Requirements

**Minimum:**
- Chrome 88+
- Safari 15+
- Firefox 89+
- Edge 88+

**Required APIs:**
- BroadcastChannel API
- CSS Grid
- CSS aspect-ratio property
- Canvas 2D context
- ES6 JavaScript

**Recommended:**
- Chrome for best performance
- 1920×1080 or higher resolution
- HTTPS or localhost

---

## Integration Requirements

**Works With:**
- OMS Shader Baker V2.8.0-A or later

**Shader Baker Must Have:**
- BroadcastChannel broadcasting enabled
- Instance announcement system
- Canvas snapshot broadcasting
- 'oms-live-channel' communication

**See:** E_Integration_Guide.md for complete setup instructions

---

## Known Limitations

### UI/UX
- Fixed 1200px width (non-responsive)
- 4 tracks only (not scalable yet)
- 8 scenes only (not expandable yet)
- No cell resizing

### Performance
- No clip launching (UI shell only)
- No scene launching (triggers don't play)
- Meters don't animate (no audio data)

### Communication
- One-way only (receives, doesn't send)
- Requires Shader Baker V2.8.0+
- No discovery of non-Baker instances

---

## Development Roadmap

### V1.1.0 (Next Release)
- Audio engine integration
- Audio file loading per clip
- Basic playback (play/pause per track)
- Master section (5th mixer column)

### V1.2.0
- FFT data broadcasting to Shader Baker
- Real-time meter animation
- Clip launching functionality
- Scene launching playback

### V2.0.0
- MIDI control (Midifighter Twister)
- Ableton Link integration
- Multi-page support (8+ decks)
- Advanced compositing

### Future: OMS Session
- Real-time visual compositing
- Advanced blend modes
- Crossfader
- Set recording/export
- Professional performance tool

---

## File Size

**V1.0.0:**
- Main app: 37KB
- Documentation: ~150KB
- Total package: ~187KB

**Minimal overhead, maximum portability**

---

## Installation

**No installation required:**
1. Download ShaderBakerLive_V1_0_0.html
2. Open in browser (Chrome recommended)
3. Open OMS Shader Baker V2.8.0+ in separate tab(s)
4. Instances auto-discover
5. Assign instances to tracks
6. Canvas snapshots appear automatically

---

## Support

**No paid support provided:**
- Open source project
- Community-driven development
- "Ghost protocol" release philosophy
- No user assistance guarantee

**For issues:**
- Check documentation thoroughly
- Review iteration notes
- Test with known-good Shader Baker version (V2.8.0-C)

---

## License

Open source (details TBD)  
Free for personal and commercial use  
Part of the OMS ecosystem  

---

## Credits

**Creator:** Wes Smith (OneManShYo)  
**Development:** November 2025  
**Iterations:** A through P (15 total)  
**Philosophy:** "Anyone can be a OneManShYo"

---

**End of Release Guide**
