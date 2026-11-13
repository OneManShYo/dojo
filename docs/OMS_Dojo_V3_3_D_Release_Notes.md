# OMS SHADER BAKER V3.3.0 - RELEASE NOTES

**Release Date:** November 14, 2025  
**Version:** V3.3.0  
**Status:** Production Ready  
**License:** GPL-3.0

---

## 🎯 WHAT'S NEW

### Perfect Beat Grid Synchronization

V3.3.0 solves the fundamental challenge of visual timing in multi-track DJ performance. After 53 development iterations, OMS Shader Baker now provides **professional-grade beat grid alignment** that rivals Ableton Live, Traktor, and Rekordbox.

**"Soldiers at the goalpost"** - all beat grids align perfectly regardless of track BPM or master tempo setting.

### Key Improvements

✅ **Multi-BPM Track Alignment**  
Load tracks at 124, 126, and 128 BPM - they all sync perfectly to any master tempo (60-200 BPM)

✅ **Live Tempo Changes**  
Adjust master tempo during playback - zero drift, instant recalculation, perfect sync maintained

✅ **Beat-Aligned Waveforms**  
Waveform bars now align with beats (1 bar per beat), making musical structure visible

✅ **Dynamic Zoom Range**  
Zoom from 10-200 pixels per second with intelligent minimum based on longest loaded track

✅ **Phrase Markers**  
Magenta markers every 4 bars (aligned with bar numbers 1, 5, 9, 13...) for easy navigation

✅ **Shader Preview (BETA)**  
Real-time WebGPU shader rendering in LIVE tab sidebar - foundation for full track compositing

---

## 🎵 FEATURES

### LIVE Tab Performance Grid

**Professional DJ Workflow:**
- 4 tracks × 8 scenes (32 performance cells)
- Master tempo control (60-200 BPM)
- Time-stretching all tracks to master tempo
- Synced waveform scrolling (Serato-style)
- Complete mixer controls per track

**Visual Timing:**
- Beat grid (cyan bars every musical bar)
- Phrase markers (magenta every 4 bars)
- Bar numbers aligned with phrases
- Beat-aligned waveform bars
- Zoom control (10-200 px/sec)

**Audio Playback:**
- 4 simultaneous tracks
- Individual volume, mute, solo per track
- Real-time VU meters
- Sample-accurate synchronization

### Complete Shader Creation Suite

**Controls Tab:**
- WebGPU shader editor
- Real-time preview
- Audio-reactive controls
- FFT frequency analysis (32768 samples)
- User-definable frequency mapping

**Audio Tab:**
- Drag-and-drop audio loading
- BPM detection integration
- Start/end time selection
- Loop in/out points
- Pitch adjustment

**Render Tab:**
- FFmpeg video export
- Apple ProRes 422 format
- High-quality render settings
- Professional video output for Ableton/EboSuite

**Library Tab:**
- .oms file management
- Visual preset thumbnails
- Quick preset switching
- Folder-based organization

---

## 🔧 WHAT CHANGED

### From V3.2.0 to V3.3.0

**Architectural Overhaul:**
- Eliminated CSS transforms (scaleX) - now render at master tempo directly
- Single timeline for all tracks (the "One Timeline" principle)
- Tempo-independent visual timing
- Real-time scroll at master tempo (not per-track tempo)

**Beat Grid System:**
- Now renders at master tempo (not original track BPM)
- Common pixels-per-second for all tracks
- Tempo scaling formula for multi-BPM alignment
- Sample reading at original tempo, drawing at master tempo

**Waveform Rendering:**
- Beat-aligned bars (1 per beat)
- Musical structure visible
- Proper amplitude calculation at any tempo
- Perfect snap with beat grid lines

**Tempo Control:**
- Live changes during playback
- Instant recalculation (no lag)
- Automatic scroll animation restart
- Zero drift across all tracks

**Zoom System:**
- Dynamic range (10-200 px/sec)
- Intelligent minimum (entire longest track fits)
- Smooth real-time adjustment
- Updates when tracks load/unload

---

## 📊 TECHNICAL HIGHLIGHTS

### The One Timeline Principle

**Problem:** How do tracks with different BPMs (124/126/128) align visually while playing at different speeds?

**Solution:** All tracks use ONE master timeline. Time-stretched audio plays at master tempo, visuals render at master tempo, scrolling happens at real time = master tempo.

**Result:** Perfect alignment that professional DJs expect.

### Key Algorithms

**Tempo Scaling:**
```
scaledTime = originalTime × (trackBPM ÷ masterTempo)
```

**Playback Rate:**
```
playbackRate = masterTempo ÷ trackBPM
```

**Visual Position:**
```
pixelPosition = scaledTime × pixelsPerSecond
```

**Scroll Speed:**
```
scrollOffset = realTimeElapsed × pixelsPerSecond
```

### Performance

- 60fps shader rendering
- Smooth scrolling with 4 tracks loaded
- Instant tempo changes (zero lag)
- Real-time zoom adjustment
- < 500KB file size (single HTML file)

---

## 🎨 USER INTERFACE

### LIVE Tab Layout

**Left Sidebar:**
- Container Library (preset selection)
- Shader Preview Canvas (16:9, beta)
- Set Library Folder button

**Center Area:**
- 4 tracks × 8 scenes grid
- Cell thumbnails (shader presets)
- Track waveforms below grid
- Beat grid visualization
- Synced scrolling

**Right Sidebar:**
- Master tempo control (rotary)
- Zoom control (rotary)
- Mixer controls per track:
  - Volume knob
  - Mute button
  - Solo button
  - VU meter

**Bottom:**
- Waveform control buttons (Q, |<, |>, <<, >>)
- Track control strip

### Visual Design

- Professional DJ software aesthetic
- Hardware mixer case borders (10px gray)
- Cyan beat grids (every bar)
- Magenta phrase markers (every 4 bars)
- Bar-style waveforms (beat-aligned)
- Responsive rotary knobs
- Clean typography

---

## 🚀 GETTING STARTED

### Quick Start

1. **Open the file:** `OMS_ShaderBaker_V3_3_0.html` in Chrome 113+ or Safari Technology Preview
2. **Set library:** Click LIBRARY tab → "Set Library Folder" → choose folder with .oms presets
3. **Switch to LIVE:** Click LIVE tab
4. **Load tracks:** Drag audio files onto track waveforms
5. **Launch:** Click cells to load presets (audio auto-plays)
6. **Mix:** Use tempo, zoom, volume, mute, solo controls

### System Requirements

**Browser:**
- Chrome 113+ (recommended)
- Safari Technology Preview (WebGPU support)

**Hardware:**
- 16GB RAM recommended (for video rendering)
- Dedicated GPU recommended (for WebGPU)
- Audio interface optional

**Files:**
- Audio: MP3, WAV, FLAC, M4A
- Presets: .oms files (OMS Shader Baker format)
- Projects: .omsp files (OMS Project format)

---

## 📖 DOCUMENTATION

### Included Guides

**Complete User Manual:**
- `OMS_ShaderBaker_V3_3_0_F_UserGuide_All.md`
- All tabs documented
- Workflow tutorials
- Tips and tricks

**FFmpeg Video Guide:**
- `OMS_ShaderBaker_V3_3_0_G_UserGuide_FFmpeg_Video_Guide.md`
- Video export workflow
- ProRes conversion commands
- Ableton/EboSuite integration

**Shader Creation Guide:**
- `OMS_ShaderBaker_V3_3_0_E_Claude_ShaderCreation_Guide.md`
- AI-assisted shader development
- Prompt templates
- Best practices

**Developer Documentation:**
- `OMS_ShaderBaker_V3_3_0_C_Developer_Documentation.md`
- Architecture reference
- Code structure
- API documentation

---

## ⚠️ KNOWN LIMITATIONS

### Not Yet Implemented

**Performance Features:**
- Scene launching (trigger entire rows) - planned V3.4.0
- Quantized triggering (snap to bars) - planned V3.4.0
- Loop bracket visualization - planned V3.4.0
- Crossfading between cells - planned V3.4.0

**Advanced Control:**
- MIDI control integration - planned V3.4.0
- Ableton Link synchronization - planned V3.4.0
- Hardware controller support - planned V3.4.0

**Shader System:**
- Shader preview shows default gradient only (beta)
- No sync with Controls tab shader yet
- No audio reactivity in LIVE shader yet
- No track compositing yet

---

## 🔄 UPGRADE GUIDE

### From V3.2.0 to V3.3.0

**No action required!** Simply open your existing .omsp project files in V3.3.0.

**What You'll Notice:**
- Beat grids now align perfectly across all tracks
- Tempo changes don't cause drift anymore
- Waveforms show musical structure (beat-aligned bars)
- Zoom has wider range (10-200 px/sec)
- New shader preview canvas in LIVE tab

**What Works Exactly the Same:**
- All tabs function identically
- .oms files fully compatible
- .omsp projects fully compatible
- FFmpeg export workflow unchanged
- Keyboard shortcuts unchanged

**File Compatibility:**
- ✅ .oms files: 100% compatible
- ✅ .omsp projects: 100% compatible
- ✅ Audio files: Same support
- ✅ Rendered videos: Same format

---

## 🎓 TUTORIALS

### Basic Mixing Workflow

1. **Load Presets:** Click cells in grid to load shader presets
2. **Load Audio:** Drag audio files onto waveforms
3. **Set Tempo:** Adjust master tempo knob (tracks auto-stretch)
4. **Mix:** Use volume, mute, solo to blend tracks
5. **Navigate:** Use zoom to see overview or detail
6. **Perform:** Beat grids keep everything aligned

### Creating Custom Shaders

1. **CONTROLS Tab:** Write/edit WGSL shader code
2. **AUDIO Tab:** Load reference audio
3. **Test:** Preview with audio reactivity
4. **LIBRARY Tab:** Save as .oms preset
5. **LIVE Tab:** Use in performance

### Video Export for Ableton

1. **RENDER Tab:** Set start/end times, quality
2. **Launch Export:** Creates .mov file
3. **FFmpeg:** Convert to ProRes 422
4. **Ableton:** Load into EboSuite video track
5. **Perform:** DJ with synchronized visuals

---

## 🆘 SUPPORT

### "Ghost Protocol"

OMS Shader Baker follows the "ghost protocol":
- Free and open source (GPL-3.0)
- No official support provided
- Community-driven learning
- YouTube channel for demos

### Resources

**YouTube Channel:** @ONEMANSHYO
- 100k monthly views
- Live DJ sets with OMS tools
- Workflow demonstrations
- New feature showcases

**Documentation:**
- All guides included in deliverables package
- Developer documentation for code reference
- Comprehensive user manuals

**Community:**
- Watch YouTube livestreams
- Learn from real performances
- See tools in action

---

## 🎯 ROADMAP

### V3.4.0 (Planned Q1 2026)

**Scene Launching:**
- Trigger entire rows (8 cells at once)
- Quantized launching (snap to bars/phrases)
- Loop bracket visualization
- Crossfading between cells

**Advanced Control:**
- Web MIDI API integration
- "Wilson CC" (tribute to Chris Wilson, MIDI spec creator)
- Midifighter Twister mapping
- Hardware controller support

**Shader System:**
- Complete shader preview integration
- Audio reactivity in LIVE shader
- Track compositing (blend 4 shaders)
- Visual blend modes

### V3.5.0 (Vision Q2 2026)

**Real-Time Rendering:**
- Shader rendering per track (4 simultaneous)
- Master output compositing
- Advanced visual effects
- Complete Ableton + EboSuite replacement

### V4.0.0 (Vision Q3 2026)

**OMS Session:**
- Complete performance mixing environment
- Recording/export from LIVE tab
- Professional-level control
- "Anyone can be a OneManShYo"

---

## 🙏 CREDITS

**Development:** Wes Jones / ONEMANSHYO  
**AI Development Assistant:** Claude (Anthropic)  
**Key Insight:** Christoph Guttandin - "One Timeline" architecture  
**Testing:** Solo DJ workflow validation (35 years experience)  
**Philosophy:** "Ghost protocol" - build, share, move on

---

## 📜 LICENSE

**GPL-3.0 - Free and Open Source**

Copyright (C) 2024-2025 Wes Jones / ONEMANSHYO

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

---

## 🎉 CELEBRATION

**53 iterations** solving one of the hardest problems in music software:  
**Perfect visual synchronization across multi-BPM tracks with live tempo changes.**

**Result:** Professional DJ timing accuracy that "just works."

**Quote:** *"mate we have solved it"* - Wes Jones, after V3.3.0_M

**Achievement Unlocked:** "Soldiers at the goalpost" - all bars aligned perfectly ✓

---

**Download:** https://onemanshyo.com  
**Watch:** YouTube @ONEMANSHYO  
**Perform:** Solo DJ + Live Reactive Visuals

**END OF RELEASE NOTES**
