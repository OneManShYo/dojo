# OMS SHADER BAKER V4.0.0_D - USER GUIDE

**Version:** V4.0.0_D Final Build  
**Date:** November 25, 2025

---

## QUICK START

1. Open `OMS_ShaderBaker_V4_0_0_D.html`
2. Settings tab (S) → "Set Application Folder"
3. Settings → GREETING → Type custom greeting (optional)
4. Controls tab (C) → Create shader
5. Click S button → Snap to bars
6. Save as .oms → "Add to Library"
7. LIVE tab → Drag to grid
8. Pop Out → Second monitor (optional)
9. Press Enter to launch!

---

## WHAT'S NEW IN V4.0.0

### 🎵 Audio-Reactive LIVE Tab

**Your shaders now react to audio during performance!**
- Timeline-based reactivity (pre-analyzed FFT)
- Perfect match to Controls tab preview
- Bass, mids, treble all working
- No CPU overhead (pre-calculated)

### 🎯 Enter Key Fixed

**Works like a CDJ cue button:**
- Instant response every tap
- Re-cues to Bar 1
- No more blocking
- Professional workflow

### 📊 S Button (Snap to Bar)

**New in Controls tab:**
- Calculates complete bars
- Shows count in Last Bar field
- Saves to container
- LIVE tab uses for grid alignment

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

**NEW V4.0.0 Steps:**

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
4. **Press Enter again** → re-cue to Bar 1

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
- Visual countdown (coming in V1.0)

### Audio Reactivity

**Automatic in V4.0.0:**
- No setup required
- Uses timeline data from container
- Modulators work automatically:
  - modA: Bass response
  - modB: Mid response
  - modC: Treble response

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

**Display:**
- `G`: Toggle beat grid

---

## S BUTTON WORKFLOW

### Purpose

Ensures perfect beat grid alignment by calculating exact bar count.

### How to Use

1. **Load audio file** in Controls tab
2. **Set Audio Start** (green line at Bar 1)
3. **Set Audio End** (or use file end)
4. **Set BPM** accurately
5. **Click S button**
6. **Last Bar field** shows count (e.g., 164)
7. **Yellow line** appears at snap point
8. **Save container** → includes bar count

### Why It Matters

- LIVE tab uses this for grid spacing
- Prevents drift over time
- Ensures bars align with music
- "Ableton warp marker" approach

---

## FILE MANAGEMENT

### .oms Containers

**Creating:**
1. Design shader in Controls/Code
2. Load audio (optional)
3. Click S → snap to bars (NEW!)
4. "Add to Library" → FFT analysis
5. Name your container
6. Auto-saves to LIBRARY folder

**Using:**
1. Library tab → see all containers
2. Drag to LIVE grid cell
3. Launch with Enter
4. Update button → instant (V4.0.0)

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

## AUDIO REACTIVITY DETAILS

### Timeline System (V4.0.0)

**How It Works:**
1. **Analysis Phase** (Controls Tab):
   - "Add to Library" analyzes FFT
   - Creates timeline arrays
   - 30 or 60 samples per second
   - Stores in container

2. **Performance Phase** (LIVE Tab):
   - Reads timeline data
   - Calculates waveform live
   - Combines: `timeline × waveform`
   - Perfect sync with audio

### Modulator Settings

**In Controls Tab:**
- Subdivision: Waveform rate (1/4, Whole, etc.)
- Waveform: Type (sine, cycler, etc.)
- Gain: Amplitude (0.1-10.0)
- Freq1/Freq2: FFT range (Hz)

**In LIVE Tab:**
- All settings from container
- No adjustment needed
- Works automatically

---

## TROUBLESHOOTING

### Common Issues

**No Audio Reactivity:**
- Re-save with "Add to Library"
- Check timeline arrays exist
- Update old containers

**Enter Key Not Working:**
- Check cell has container
- Verify not in text field
- Should work every tap now

**Grid Misalignment:**
- Click S button in Controls
- Re-save container
- Check BPM is correct

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

### During Performance

**Live Techniques:**
- Use keyboard only (no mouse)
- Trust quantization
- Enter for re-cueing
- Esc for emergency
- Pop-out to projector

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

- Audio engine (100%)
- File system (100%)
- Basic mixing (100%)
- Enter key (100%)
- Keyboard control (100%)

### What's Very Good 🟢

- Audio reactivity (~90%)
- Visual sync (~80%)
- Beat grid (~80%)
- Timeline system (~90%)

### Still Improving 🟡

- Master Grid alignment (~70%)
- Edge case handling
- Final polish

### Safe For:

✅ Studio work  
✅ Practice sessions  
✅ Demo performances  
⚠️ Wait for V1.0 for critical shows

---

## V3 → V4 MIGRATION

### Your Files Work!

- All .oms containers compatible
- All .omsp projects load
- No migration required

### To Optimize:

1. Load container in Controls
2. Click S button (snap)
3. Click Update (preserves FFT)
4. Re-save
5. Now optimized for V4!

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
- Note version (V4.0.0_D)

---

**END OF USER GUIDE**

Version: V4.0.0_D Final Build  
Date: November 25, 2025  
License: GPL-3.0

*"After 58 iterations, it works!"*
