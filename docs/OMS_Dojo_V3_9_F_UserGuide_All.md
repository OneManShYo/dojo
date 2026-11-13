# OMS SHADER BAKER V3.9.0 - USER GUIDE

**Version:** V3.9.0_L_sub20 (Development Build)  
**Date:** November 21, 2025

---

## ⚠️ DEVELOPMENT BUILD NOTICE

This is V3.9.0_L_sub20, a development build with timing architecture improvements.  
If you need a stable version, use V3.8.0.

---

## QUICK START

1. Open `OMS_ShaderBaker_V3_9_0_L_sub20.html`
2. Settings tab (S) → "Set Application Folder"
3. Settings → GREETING → Type custom greeting (optional)
4. Controls tab (C) → Create shader
5. Save as .oms
6. LIVE tab → Drag to grid
7. Pop Out → Second monitor (optional)
8. Space to play!

---

## WHAT'S NEW IN V3.9.0

### Timing Architecture (Internal)

**You won't see these changes, but you'll feel them:**
- More accurate synchronization
- Better tempo change handling
- Improved Master Grid alignment
- Smoother waveform scrolling

**Why it matters:**
- Tempo changes during performance work better
- Multiple tracks stay in sync
- Visual beat grids align perfectly
- No drift during long sets

### Pop-Out Window Fix

**Shaders now render correctly in pop-out window**
- No more compilation errors
- All 4 tracks display properly
- Secondary display works reliably

---

## INTERFACE

### Tabs

**C (Controls):** Shader creation  
**P (Presets):** Library  
**L (Library):** .oms containers *  
**W (Code):** WGSL editor  
**S (Settings):** Config *  
**A (About):** Info  
**LIVE:** Performance *

* = Requires folder

### Header

**Left:** ONEMANSHYO + version  
**Right:** Your custom greeting

---

## SHADER CREATION

### Controls Tab

Sliders for instant preview

### Code Tab

Edit WGSL directly

**Modulators:**
- uModA: Bass (30-500 Hz)
- uModB: Mids (500-2000 Hz)  
- uModC: Treble (2000-20000 Hz)

---

## LIVE PERFORMANCE

### Grid Layout

5 columns × 8 rows  
Tracks 1-4 + Master/Scenes

### Loading

Drag .oms from Library → Grid

### Launching

**Scene:** Click number or Enter  
**Cell:** Double-click or Enter  
**Quantized:** Bar-aligned

### Mixer

**Per-Track:**
- Volume knob (audio)
- Opacity knob (shader)
- M/S buttons (both domains)
- Level meters

**Master:**
- Volume (2x larger)
- M/S buttons
- Tempo control

---

## KEYBOARD SHORTCUTS

**Transport:**
- `Space`: Play/pause
- `Esc`: Emergency stop

**Track Selection:**
- `1-4`: Tracks
- `5`: Master
- `Tab/Shift+Tab`: Cycle

**Navigation:**
- `WASD`: Grid navigation
- `Enter`: Launch

**Audio:**
- `↑↓`: Volume
- `,`: Mute
- `.`: Solo

**Shader:**
- `←→`: Opacity
- `Shift+,`: Mute
- `Shift+.`: Solo

**Zoom:**
- `[ ]`: Discrete
- Trackpad: Continuous

**Cell:**
- `Delete`: Clear

---

## DUAL-DOMAIN MIXER

**Shader Domain:**
- Opacity knob
- M/S (purple/cyan)
- `←→` keys

**Audio Domain:**
- Volume fader
- M/S (standard)
- `↑↓` keys

**Use Cases:**
- Visual build without audio change
- Audio drop with visual hold
- Solo one shader, play all audio

---

## SETTINGS

### APPLICATION
- Set folder
- Canvas size

### PROJECT STATES
- Save/load .omsp
- default.omsp auto-saves

### GREETING
- Custom header text
- 50 char max

### POP-OUT
- Window size
- Always on Top

---

## FILE MANAGEMENT

### .oms Containers

**Create:**
1. Controls → shader
2. Load audio (optional)
3. "Save as .oms"
4. Library tab shows it

**Use:**
1. Drag to LIVE grid
2. Launch!

### .omsp Projects

**Save:**
1. Settings → PROJECT STATES
2. "Save Project"
3. Name it

**Load:**
1. Settings → PROJECT STATES
2. Select dropdown
3. "Load Project"

---

## TESTING V3.9.0

### What to Try

**Single Track:**
- Load and launch one container
- Watch waveform scroll
- Check bar alignment
- Verify smooth playback

**Tempo Changes:**
- Start at 120 BPM
- Play for several bars
- Change tempo (60-200 range)
- Verify waveform adjusts
- Check bars stay aligned

**Multiple Tracks:**
- Launch 2-4 tracks
- At different times
- Verify synchronization
- Check Master Grid alignment

**Pop-Out Window:**
- Click "Pop Out ↗"
- Verify shaders render
- All 4 tracks working
- No errors in console

### What to Report

**If something doesn't work:**
1. Note exact steps
2. Include tempo settings
3. Browser (Chrome/Safari)
4. Check console (F12)
5. Report via social media or contact form

---

## TIPS & TRICKS

### Performance

**Prep:**
- Create shaders beforehand
- Organize Library
- Build grid layout
- Save .omsp
- Test tempo changes

**During:**
- Keyboard only
- Pop-out to projector
- Trust muscle memory
- Esc = emergency

### Workflow

**Shaders:**
- Start simple
- Test incrementally
- Save often
- Name clearly

**Audio:**
- Isolate frequencies
- Test with real music
- Subtle > crazy

### Troubleshooting

**Won't compile:** Check console (F12)  
**Not reactive:** Check FFT ranges  
**Slow:** Lower resolution  
**Overlay stuck:** Press Esc  
**Bars misaligned:** Report (development build)  
**Tempo change issues:** Report (under testing)

---

## DIFFERENCES FROM V3.8.0

### What Changed

**Internal:**
- Master clock architecture
- Beat-based scrolling
- Tempo history tracking
- Launch beat recording

**External:**
- Pop-out window shader fix
- Better timing accuracy
- Improved synchronization

### What Stayed Same

**Everything else:**
- User interface
- Keyboard shortcuts
- File formats (.oms, .omsp)
- Shader creation
- Dual-domain mixer
- All V3.8.0 features

---

## KNOWN ISSUES

⚠️ Development build - some features under testing:
- Master Grid alignment refinement
- Tempo change validation
- Multi-track sync verification

**Production release coming soon with all issues resolved.**

---

## CHEAT SHEET

```
Right Hand:        Left Hand:
    [↑]                 [W]
[←][↓][→]          [A][S][D]
                    [Space]

Track: [1][2][3][4][5]
Zoom: [ ]
M/S: , .  (Shift+ for shader)
```

---

## FOR MORE INFO

**Shader Creation:** See Claude_ShaderCreation_Guide.md  
**FFmpeg Export:** See UserGuide_FFmpeg_Video_Guide.md  
**Development:** See Developer_Documentation.md  
**Changes:** See Complete_Changelog.md

---

**END OF USER GUIDE**

Version: V3.9.0_L_sub20 (Development Build)  
License: GPL-3.0  
Website: https://itswessmithyo.com

**Note:** This is a development build. Production release will follow  
after testing is complete. V3.8.0 remains stable production version.
