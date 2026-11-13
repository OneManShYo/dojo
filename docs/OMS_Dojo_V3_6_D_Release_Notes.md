# OMS SHADER BAKER V3.6.0 - RELEASE NOTES

**Release Date:** November 16, 2025  
**Version:** V3.6.0 Production  
**Type:** Major Feature Release

---

## WHAT'S NEW

### 🎨 Four-Track Shader Compositing

**Mix four shaders simultaneously with your audio faders!**

- Independent WebGPU rendering on each of 4 tracks
- Volume faders control both audio AND visual opacity
- Z-index layering (left track = top layer)
- Natural DJ mixing workflow
- 60fps performance with all tracks playing

**How to use:**
1. Load different .oms containers into grid cells
2. Launch clips on multiple tracks
3. Adjust volume faders to crossfade visuals
4. Left-to-right = top-to-bottom layer priority

---

### 📺 Pop-Out Window for Second Monitor

**Send visuals to projector/audience display!**

- Resizable output window (640×360 or 1280×720)
- Always-on-top option
- Synchronized with inset view
- Perfect for live performances

**How to use:**
1. Click "Pop Out ↗" button above shader output
2. Drag window to second monitor/projector
3. Optionally enable "Always on Top" in Settings tab
4. DJ sees interface, audience sees full-screen visuals

---

### 🎵 Waveform Overlays

**See which shader is playing on which track!**

- Container name + shader thumbnail on waveforms
- Only visible when track playing
- Positioned at lower third (non-intrusive)
- Marquee scrolling for long names

**User benefit:**
Instant visual feedback connecting grid cells to audio tracks

---

### 💎 Professional Visual Indicators

**Never lose track of cursor position or playback state!**

- **Cyan outline:** Selected cell (where you ARE)
- **Yellow border:** Playing cell (what's making sound)
- **Both visible:** When selected cell is also playing
- No visual conflicts

---

### ⚙️ Settings Persistence

**Your preferences stick around!**

- Pop-out window size saved
- Always-on-top preference saved
- Stored in .omsp project files
- Auto-loads when opening projects

---

### ⌨️ DELETE Key Cell Clearing

**Quick workflow changes during performance!**

- Press DELETE or Backspace to clear selected cell
- Removes audio, waveform, shader (both windows)
- Cross-platform support (Mac + Windows)
- Instant cleanup

---

### 🎛️ Empty Cell Clip Stop

**Ableton-style workflow!**

- Launch empty cell = stop that track
- Launch empty scene = stop all 4 tracks
- Visual feedback (removes yellow border)
- Industry-standard DJ behavior

---

## IMPROVEMENTS

✅ **Scene Column Default** - Cursor starts in SCENE column for faster scene launching  
✅ **Z-Index Priority** - Left-to-right layering matches mixer layout  
✅ **Clean Default Shader** - Black screen instead of gradient  
✅ **Thinner Visual Indicators** - 50% reduction for professional appearance

---

## BUG FIXES

🐛 **Function Scope Issues** - Fixed container loading failures (3 iterations)  
🐛 **Waveform Alignment** - Critical playhead positioning fix  
🐛 **Visual State** - Playing indicators now match audio state perfectly  
🐛 **Canvas Rendering** - Overlay thumbnails now render correctly

---

## PERFORMANCE

**Tested on M1 MacBook Pro 14":**
- 4 shaders rendering: ~30-45% GPU
- 60fps stable with all tracks playing
- Pop-out adds ~5-10% GPU overhead
- File size: 475KB (+30KB from V3.5.0)

**Recommendations:**
- Use 640×360 pop-out for better performance
- Close other GPU-intensive apps during shows
- Enable hardware acceleration in browser

---

## COMPATIBILITY

✅ **Fully backward compatible** with V3.5.0 project files  
✅ **No migration required** - existing .omsp files work immediately  
✅ **All keyboard controls unchanged** - muscle memory intact

---

## SYSTEM REQUIREMENTS

**Minimum:**
- Chrome 113+ or Edge 113+ (WebGPU support)
- 1920×1080 screen resolution
- Integrated graphics (Intel/AMD)

**Recommended:**
- Safari Technology Preview or Chrome latest
- 1512×982 or higher (MacBook Pro 14" optimal)
- Dedicated graphics (NVIDIA/AMD) or Apple Silicon
- Dual monitor setup for pop-out window

---

## QUICK START

**For New Users:**
1. Open HTML file in Chrome/Edge/Safari TP
2. Settings tab → Set Application Folder
3. Create shader in Controls or Code tab
4. Save as .oms container
5. LIVE tab → arrange in grid
6. Pop Out → send to projector
7. Perform!

**For Existing Users:**
1. Open V3.6.0 HTML file
2. Load existing .omsp projects (work instantly)
3. Click "Pop Out ↗" for second monitor
4. Adjust Settings tab preferences
5. Keep creating!

---

## KEYBOARD SHORTCUTS

**Transport:**
- Space: Global play/pause
- Esc: Emergency stop + return to scenes

**Track Selection:**
- 1-5: Select tracks 1-4 or master
- Tab: Next track
- Shift+Tab: Previous track

**Navigation:**
- W/S: Navigate scenes (in scene column)
- A/D: Enter/exit tracks
- W/S: Navigate cells (in track columns)
- Enter: Launch scene or cell

**Volume:**
- ↑/↓: Adjust selected track volume

**Mute/Solo:**
- , (comma): Toggle mute
- . (period): Toggle solo

**Cell Management:**
- Delete/Backspace: Clear selected cell

---

## KNOWN LIMITATIONS

- Pop-out window requires user interaction to open (browser security)
- Always-on-top may not work on all browsers/OS combinations
- Maximum 4 simultaneous shaders (performance limit)
- Waveform overlays only when track playing

---

## UPGRADE PATH

**From V3.5.0 → V3.6.0:**
No changes required. Open V3.6.0 and load existing projects.

**From V3.3.0 or earlier:**
Projects load but may need re-arrangement for 4-track grid layout.

---

## WHAT'S NEXT

**V3.7.0 (Coming Soon):**
- Shader parameter controls in LIVE tab
- Advanced modulation options
- Enhanced audio reactivity

**V3.8.0 (Planned):**
- MIDI controller integration
- Hardware mixer compatibility

**V4.0.0 (Vision):**
- Complete browser-based performance system
- Replace Ableton Live + EboSuite entirely

---

## SUPPORT

**YouTube:** ONEMANSHYO channel (100k monthly views)  
**Website:** https://onemanshyo.com  
**License:** GPL-3.0 (free forever)

**Ghost Protocol:** Tools provided free with no ongoing support commitment.

---

## CREDITS

**Developer:** Wes Jones (ONEMANSHYO)  
**AI Assistant:** Claude (Anthropic)  
**Philosophy:** "Anyone can be a OneManShYo"

Special thanks to the community for testing and feedback!

---

**Enjoy V3.6.0! Now go create something amazing! 🎨🎵✨**

---

END OF RELEASE NOTES

Version: V3.6.0  
Date: November 16, 2025  
License: GPL-3.0
