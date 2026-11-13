# OMS SHADER BAKER V4.2.0_A - USER GUIDE

**Version:** V4.2.0_A Development Release  
**Date:** November 27, 2025

---

## QUICK START

1. Open `OMS_ShaderBaker_V4_2_0_A.html`
2. Settings tab (S) → "Set Application Folder"
3. Settings → GREETING → Type custom greeting (optional)
4. Controls tab (C) → Create shader
5. Click S button → Snap to bars
6. Save as .oms → "Add to Library"
7. LIVE tab → Drag to grid
8. Pop Out → Second monitor (optional)
9. **Click scene to select, then press Enter to launch!**
10. Watch time display count down!

---

## WHAT'S NEW IN V4.2.0_A

### ⏱️ Time Display per Track

**See exactly where you are!**

When a track is playing, you'll see below the title:
```
0:45 / -2:35 | 3:20
elapsed / remaining | total
```

### ⌨️ Shift+M/T/W Visibility Toggles

**WASD navigation now works properly!**

- Press **W** → Move UP in grid
- Press **Shift+W** → Toggle waveform visibility
- Press **Shift+M** → Toggle Master Grid
- Press **Shift+T** → Toggle Track Grid

### 🎯 Scene Click = Select Only

**No accidental launches!**

- **Click scene** → Selects (cyan border)
- **Press Enter** → Launches selected scene

### 📊 VU Meters Reset

**Professional behavior:**

- Meters drop to 0 on stop (Esc)
- Meters drop to 0 on pause (Space)

### 🎨 Master Grid Default ON

- Grid visible on fresh load
- Opacity reduced 50% (less distracting)

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

### Launching (Changed!)

**Two-Step Process:**
1. **Click scene button** → Selects (cyan border)
2. **Press Enter** → Launches at next bar

**Or for single track:**
- Double-click cell → Launches that track

### Time Display

**While playing, you'll see:**
```
0:45 / -2:35 | 3:20
```
- **0:45** = How long it's been playing
- **-2:35** = How much time left
- **3:20** = Total duration

---

## VISIBILITY TOGGLES

### Shift+M (Master Grid)

**What:** White bar overlay spanning all tracks

**Default:** ON (changed from V4.1.0!)

**Use:**
- Verify track grid alignment
- See the "ruler" tracks scroll under
- Less intense than before (50% opacity)

### Shift+T (Track Grid)

**What:** Cyan/magenta beat markers on waveform

**Default:** ON

**Use:**
- See beat positions in audio
- Navigate by bar numbers
- Visual timing reference

### Shift+W (Waveform)

**What:** Blue amplitude visualization

**Default:** ON

**Use:**
- See audio dynamics
- Identify drops/breaks
- Visual mixing reference

### Why Shift?

In V4.1.0, pressing W would toggle waveform instead of moving up in the grid. Now W moves up, and Shift+W toggles waveform.

---

## MIXER CONTROLS

### Per-Track Controls

**Audio Domain:**
- Volume knob (vertical fader)
- M button (mute - orange)
- S button (solo - green)
- Level meter (drops to 0 when stopped!)

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

### Navigation

**Grid:**
- `W`: Up (previous scene)
- `A`: Left (to Track 4)
- `S`: Down (next scene)
- `D`: Right (to Track 1)

**Track Selection:**
- `1-4`: Select tracks
- `5`: Select master
- `Tab/Shift+Tab`: Cycle

### Transport

- `Space`: Play/pause
- `Esc`: Emergency stop all
- `Enter`: Launch selected
- `Delete`: Clear cell

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

### Display (Changed!)

- `Shift+M`: Toggle Master Grid *(was M)*
- `Shift+T`: Toggle Track Grid *(was T)*
- `Shift+W`: Toggle Waveform *(was W)*

### Zoom

- `[`: Zoom in (10px steps)
- `]`: Zoom out
- Trackpad: Pinch/spread

---

## SCENE SELECTION

### How It Works Now

**Spreadsheet-Style:**
- Click a cell → Cell selected (cyan border)
- Scene button (row header) → Highlights yellow
- Navigate with WASD → Selection follows
- Press Enter → Launches selected scene

### Playing vs Selected

- **Cyan border** = Selected (cursor position)
- **Yellow background** = Actually playing

---

## VU METERS

### Professional Behavior

**When Audio Plays:**
- Meters animate with audio level
- Green/yellow/red gradient

**When Audio Stops (Esc):**
- All meters drop to 0
- Clean visual state

**When Paused (Space):**
- All meters drop to 0
- Resume → meters animate again

---

## TROUBLESHOOTING

### Common Issues

**W Key Moves Cursor Instead of Toggling:**
- This is correct! Use Shift+W for visibility toggle

**Scene Didn't Launch on Click:**
- This is correct! Click selects, Enter launches

**Meters Stuck at Old Level:**
- Press Esc to reset all
- This should be fixed in V4.2.0_A

**Time Display Not Showing:**
- Only appears when track is playing
- Check overlay is visible

### Performance Issues

**Low FPS:**
- Reduce shader complexity
- Lower resolution in Settings
- Close other applications

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
- Verify time display works

### During Performance

**Live Techniques:**
- Use keyboard only (no mouse)
- Trust quantization
- Watch time display for cues
- Enter for re-cueing
- Esc for emergency
- Pop-out to projector

### New in V4.2.0_A

- Check time remaining for transitions
- Use Shift+M to verify alignment
- VU meters show actual audio state
- Scene selection is more deliberate

---

## KEYBOARD QUICK REFERENCE

```
NAVIGATION                 TRANSPORT
    [W] Up                     [Space] Play/Pause
    [A] Left                   [Enter] Launch
    [S] Down                   [Esc] Stop All
    [D] Right                  [Delete] Clear

AUDIO (Vertical)          SHADER (Horizontal)
    [↑] Volume Up             [→] Opacity Up
    [↓] Volume Down           [←] Opacity Down
    [,] Mute                  [Shift+,] Mute
    [.] Solo                  [Shift+.] Solo

SELECTION                  ZOOM
    [1-4] Tracks              [[] Zoom In
    [5] Master                []] Zoom Out
    [Tab] Cycle               [Trackpad] Pinch

VISIBILITY (Shift Required!)
    [Shift+M] Master Grid
    [Shift+T] Track Grid
    [Shift+W] Waveform
```

---

## V4.1.0 → V4.2.0_A CHANGES

### Behavioral Changes

1. **M/T/W** now require **Shift** key
2. **Scene click** = select only (Enter to launch)
3. **Master Grid** ON by default
4. **VU meters** drop to 0 on stop/pause
5. **Time display** shows under track title

### Your Files Work!

- All .oms containers compatible
- All .omsp projects load
- No migration required

---

## SUPPORT

**Documentation:**
- This guide
- Developer docs
- Complete changelog
- Release notes

**Community:**
- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF USER GUIDE**

Version: V4.2.0_A Development Release
Date: November 27, 2025
License: GPL-3.0

*"Cleaner, more professional LIVE tab experience"*
