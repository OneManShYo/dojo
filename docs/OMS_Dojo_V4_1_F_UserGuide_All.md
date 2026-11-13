# OMS SHADER BAKER V4.1.0 - USER GUIDE

**Version:** V4.1.0 Production Release  
**Date:** November 27, 2025

---

## QUICK START

1. Open `OMS_ShaderBaker_V4_1_0.html`
2. Settings tab (S) → "Set Application Folder"
3. Settings → GREETING → Type custom greeting (optional)
4. Controls tab (C) → Create shader
5. Click S button → Snap to bars
6. Save as .oms → "Add to Library"
7. LIVE tab → Drag to grid
8. Pop Out → Second monitor (optional)
9. Press Enter to launch!
10. **NEW:** Press M to verify grid alignment!

---

## WHAT'S NEW IN V4.1.0

### 🎯 Perfect Grid Alignment

**Track grids now align exactly with Master Grid!**
- Press M to show the Master Grid overlay
- Track beat markers line up with white bar lines
- Tempo changes maintain alignment
- Zoom changes maintain alignment
- Professional-grade precision

### 🔧 M/T/W Visibility Toggles

**New buttons in LIVE tab:**
- **M** - Master Grid (white bar overlay) - default OFF
- **T** - Track Grid (cyan/magenta markers) - default ON
- **W** - Waveform (blue amplitude bars) - default ON

**Keyboard shortcuts:** Press M, T, or W keys

### 📏 Recalibrated Zoom

**New zoom levels:**
- 0.5x - Very zoomed out
- 1x - Default view (~20+ bars)
- 2x - Closer view
- 4x - Detail view (phrase markers only)
- 8x - Extreme detail (phrase markers only)

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

## SHADER CREATION WORKFLOW

### Step 1: Create Shader

**Controls Tab:**
- Adjust sliders for instant preview
- Load audio file (optional)
- Set BPM, loop points
- Test audio reactivity

**Code Tab:**
- Edit WGSL directly
- Use Claude AI for help
- Copy/paste shaders

### Step 2: Prepare for Performance

1. **Load final audio file**
2. **Set Audio Start** (where Bar 1 begins)
3. **Click S button** → calculates bars
4. **See Last Bar field** → verify count
5. **Click "Add to Library"** → analyzes FFT
6. **Save as .oms** → includes timeline data

### Step 3: Perform in LIVE Tab

1. **Drag .oms** from Library to grid
2. **Press Enter** → launches at next bar
3. **Shaders react** to audio automatically
4. **Press M** → verify grid alignment
5. **Press Enter again** → re-cue to Bar 1

---

## LIVE PERFORMANCE

### Grid Layout

**5 columns × 8 rows:**
- Tracks 1-4: Performance columns
- Master: Scene controls + master mixer

### Loading Containers

**Drag & Drop:**
- Drag .oms from Library
- Drop on any grid cell
- Cell shows container name
- Thumbnail preview visible

### Launching

**Quantized Launch:**
- **Scene:** Click number (launches all 4 tracks)
- **Cell:** Double-click or Enter (single track)
- Always starts at next bar boundary

### Verifying Alignment

**NEW in V4.1.0:**
1. Launch a track
2. Press **M** to show Master Grid
3. Track grid bars align with white lines
4. Change tempo - stays aligned
5. Change zoom - stays aligned

---

## VISIBILITY TOGGLES

### M Button (Master Grid)

**What:** White bar overlay spanning all tracks

**Default:** OFF

**Use:**
- Verify track grid alignment
- See the "ruler" tracks scroll under
- Confirm perfect sync

### T Button (Track Grid)

**What:** Cyan/magenta beat markers on waveform

**Default:** ON

**Use:**
- See beat positions in audio
- Navigate by bar numbers
- Visual timing reference

### W Button (Waveform)

**What:** Blue amplitude visualization

**Default:** ON

**Use:**
- See audio dynamics
- Identify drops/breaks
- Visual mixing reference

### Combinations

**Alignment check:**
- M: ON, T: ON, W: OFF
- See both grids without waveform

**Clean mixing:**
- M: OFF, T: ON, W: ON
- Normal performance view

**Audio only:**
- M: OFF, T: OFF, W: ON
- Just the waveform

---

## MIXER CONTROLS

### Per-Track Controls

**Audio Domain:**
- Volume knob (vertical fader)
- M button (mute - orange)
- S button (solo - green)
- Level meter

**Shader Domain:**
- Opacity knob (visual intensity)
- M button (mute - purple)
- S button (solo - cyan)
- Manual/Auto modes

### Master Controls

**Large Knobs (2x size):**
- Master Volume
- Master Tempo (60-200 BPM)
- Zoom (waveform display)

**Buttons:**
- M/S for audio
- Pop Out (second monitor)

---

## KEYBOARD SHORTCUTS

### Essential

**Transport:**
- `Space`: Play/pause
- `Esc`: Emergency stop all
- `Enter`: Launch selected

**Track Selection:**
- `1-4`: Select tracks
- `5`: Select master
- `Tab/Shift+Tab`: Cycle

### Mixer Control

**Audio (Vertical):**
- `↑`: Volume up
- `↓`: Volume down
- `,`: Mute toggle
- `.`: Solo toggle

**Shader (Horizontal):**
- `←`: Opacity down
- `→`: Opacity up
- `Shift+,`: Mute toggle
- `Shift+.`: Solo toggle

### Navigation

**Grid:**
- `W`: Up (previous scene)
- `A`: Left (to Track 4)
- `S`: Down (next scene)
- `D`: Right (to Track 1)
- `Delete`: Clear cell

**Zoom:**
- `[`: Zoom in (10px steps)
- `]`: Zoom out
- Trackpad: Pinch/spread

### Display (NEW!)

- `M`: Toggle Master Grid
- `T`: Toggle Track Grid
- `W`: Toggle Waveform
- `G`: Toggle grid (legacy, same as M)

---

## ZOOM SYSTEM

### Zoom Levels

| Level | View | Best For |
|-------|------|----------|
| 0.5x | Very zoomed out | Overview |
| 1x | Default (~20+ bars) | Normal mixing |
| 2x | Closer | Detail work |
| 4x | Detail | Beat alignment |
| 8x | Extreme | Phrase markers only |

### At High Zoom (4x, 8x)

- Bar numbers hidden (too cluttered)
- Only magenta phrase markers shown
- Better for seeing song structure

---

## FILE MANAGEMENT

### .oms Containers

**Creating:**
1. Design shader in Controls/Code
2. Load audio (optional)
3. Click S → snap to bars
4. "Add to Library" → FFT analysis
5. Name your container
6. Auto-saves to LIBRARY folder

**Using:**
1. Library tab → see all containers
2. Drag to LIVE grid cell
3. Launch with Enter
4. Update button → instant

### .omsp Projects

**Saving:**
1. Settings → PROJECT STATES
2. "Save Project"
3. Name your project
4. Saves grid layout + settings

**Loading:**
1. Settings → PROJECT STATES
2. Select from dropdown
3. "Load Project"
4. Entire session restored

**Auto-Save:**
- default.omsp saves automatically
- Every change preserved
- Survives browser refresh

---

## TROUBLESHOOTING

### Common Issues

**Grid Not Aligned:**
- This should be fixed in V4.1.0!
- Press M to verify alignment
- If misaligned, report as bug

**No Audio Reactivity:**
- Re-save with "Add to Library"
- Check timeline arrays exist
- Update old containers

**Enter Key Not Working:**
- Check cell has container
- Verify not in text field
- Should work every tap

**Shader Not Compiling:**
- Check Code tab for errors
- Press F12 for console
- Verify WGSL syntax

### Performance Issues

**Low FPS:**
- Reduce shader complexity
- Lower resolution in Settings
- Close other applications
- Check GPU usage

**Audio Glitches:**
- Check master tempo
- Verify BPM settings
- Reduce track count

---

## TIPS & TRICKS

### Preparation

**Before Performance:**
- Create all shaders
- Click S on all containers
- Update with timeline data
- Build grid layout
- Save project (.omsp)
- Test Enter key response
- **NEW:** Press M to verify alignment

### During Performance

**Live Techniques:**
- Use keyboard only (no mouse)
- Trust quantization
- Enter for re-cueing
- Esc for emergency
- Pop-out to projector
- **NEW:** M to show grid if needed

### Optimization

**For Best Results:**
- 60 FPS target
- 1080p resolution
- 4 tracks maximum
- Simple shaders for base
- Complex for highlights

---

## PRODUCTION READINESS

### What's Rock Solid ✅

- Grid alignment (100% fixed in V4.1.0)
- Audio engine (100%)
- File system (100%)
- Basic mixing (100%)
- Enter key (100%)
- Keyboard control (100%)

### What's Very Good 🟢

- Audio reactivity (~90%)
- Visual sync (~95%)
- Beat grid (~100%)
- Timeline system (~90%)

### Still Improving 🟡

- Edge case handling (~80%)
- Final polish

### Safe For:

✅ Studio work  
✅ Practice sessions  
✅ Demo performances  
⚠️ Wait for V1.0 for critical shows

---

## V4.0.x → V4.1.0 MIGRATION

### Your Files Work!

- All .oms containers compatible
- All .omsp projects load
- No migration required

### What's Better:

- Grid alignment fixed
- New M/T/W toggles
- Recalibrated zoom
- Canvas-drawn playhead

---

## KEYBOARD QUICK REFERENCE

```
AUDIO (Vertical)          SHADER (Horizontal)
    [↑] Volume Up             [→] Opacity Up
    [↓] Volume Down           [←] Opacity Down
    [,] Mute                  [Shift+,] Mute
    [.] Solo                  [Shift+.] Solo

NAVIGATION                 TRANSPORT
    [W] Up                     [Space] Play/Pause
    [A] Left                   [Enter] Launch
    [S] Down                   [Esc] Stop All
    [D] Right                  [Delete] Clear

SELECTION                  ZOOM
    [1-4] Tracks              [[] Zoom In
    [5] Master                []] Zoom Out
    [Tab] Cycle               [Trackpad] Pinch

VISIBILITY (NEW!)
    [M] Master Grid
    [T] Track Grid
    [W] Waveform
```

---

## SUPPORT

**Documentation:**
- This guide
- Developer docs
- Claude shader guide
- FFmpeg export guide

**Community:**
- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

**Feedback:**
- Report bugs with steps
- Include .oms files if possible
- Check console (F12)
- Note version (V4.1.0)

---

**END OF USER GUIDE**

Version: V4.1.0 Production Release
Date: November 27, 2025
License: GPL-3.0

*"Perfect grid alignment, finally!"*
