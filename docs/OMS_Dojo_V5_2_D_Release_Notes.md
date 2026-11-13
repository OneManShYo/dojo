# ONEMANSHYO App [Beta V5_2_0] - RELEASE NOTES

**Release Date:** December 26, 2025  
**Version:** Beta V5_2_0  
**Type:** Audio Controls & DJ FX

---

## WHAT'S NEW

### 🎛️ Audio Focus Mode (Shift+A)

**Complete keyboard control of audio parameters!**

Audio Focus Mode gives you keyboard navigation across all CREATE tab audio controls. Enter the mode, then access sub-panels for Volume, Cue, EQ, and FX.

| Key | Action |
|-----|--------|
| `Shift+A` | Enter/exit Audio Focus |
| `V` | Open Volume panel |
| `C` | Open Cue panel |
| `E` | Open EQ panel |
| `F` | Open FX panel |
| `Escape` | Exit panel → focus → normal |

**Gate + Panel Pattern:** First enter the gate (Shift+A), then open panels (V/C/E/F).

---

### 🎚️ 3-Band EQ System

**Professional DJ-style EQ with kill switches!**

| Band | Frequency | Type |
|------|-----------|------|
| High | 2500 Hz | Shelf |
| Mid | 1000 Hz | Peak (Q=0.7) |
| Low | 250 Hz | Shelf |

**Ableton-Style Value Mapping:**
| Knob Position | dB | Description |
|---------------|-----|-------------|
| Full left | -∞ | Kill (mute) |
| 9 o'clock | -12dB | Cut |
| 12 o'clock | -6dB | Attenuate |
| 3 o'clock | 0dB | Unity (default) |
| Full right | +6dB | Boost |

**Keyboard (E Panel):**
| Key | Action |
|-----|--------|
| `↑ / ↓` | Navigate H/M/L |
| `← / →` | Adjust gain |
| `Enter` | Kill/restore band |
| `Escape` | Exit panel |

---

### 🎧 DJ FX System

**Three essential DJ transition effects!**

| Slot | Name | Effect | Sound |
|------|------|--------|-------|
| W | **Washout** | Highpass sweep | Bass disappears as you turn up |
| D | **Dubby** | Dub delay | Classic dub echo trails |
| B | **Break** | Vinyl stop | Audio slows to a crawl |

**Keyboard (F Panel):**
| Key | Action |
|-----|--------|
| `↑ / ↓` | Navigate W/D/B |
| `← / →` | Adjust wet amount |
| `Enter` | Kill/restore effect |
| `Escape` | Exit panel |

---

### 🎯 5 Cue Points (Expanded)

**More hot cues for longer tracks!**

Expanded from 3 to 5 cue points per container.

| Cue | Default Position | Typical Use |
|-----|------------------|-------------|
| 1 | Bar 1.1 | Intro |
| 2 | Bar 17.1 | First drop |
| 3 | Bar 33.1 | Second drop |
| 4 | Bar 49.1 | Third section |
| 5 | Bar 65.1 | Fourth section |

**CREATE Tab:**
- Orange flags visible on waveform
- **Click** a flag to jump playhead
- **Drag** a flag to reposition (snaps to bar)
- Access via C panel in Audio Focus mode

**PERFORM Tab:**
- Press `1-5` to jump to cues (no Shift needed)
- Instant jump on selected track
- Works while playing or stopped

---

### 🔊 Volume Knob with Kill

**Quick volume control in audio header!**

A 24×24 cyan knob now appears in the CREATE tab audio header for instant volume control.

| Action | Result |
|--------|--------|
| Scroll | Adjust volume |
| Click | Toggle kill (mute) |
| Killed | Knob turns magenta |

Access via V panel in Audio Focus mode for keyboard control.

---

### 🎹 Per-Track Audio Processing (PERFORM)

**Independent EQ and FX per track!**

Each of the 4 PERFORM tracks now has its own audio processing chain:

```
Source → EQ (H→M→L) → FX (Washout→Dubby) → Analyser → Gain → Master
```

**Track Modules** replace the old nudge buttons, showing Cue/EQ/FX controls per track.

| Key | Action |
|-----|--------|
| `Shift+1-4` | Select track (gate) |
| `C` | Open Cue panel |
| `E` | Open EQ panel |
| `F` | Open FX panel |
| `1-5` | Jump to cue (no gate) |

---

### 🧠 Centralized Focus Mode

**Smarter keyboard mode management!**

All focus modes now use a single system that prevents conflicts:

- Only one mode active at a time
- Entering a new mode exits the previous
- Clean selection ring management
- Old code still works (compatibility layer)

**Active Modes:**
- Shift+A → Audio Focus
- Shift+T → Transform Focus
- Shift+R → Reactivity Focus
- Shift+M → Music Timing Focus
- Shift+B → BumpR Mode

---

### 🔒 Tab Isolation

**Keyboard shortcuts stay in their lane!**

CREATE tab shortcuts only work on CREATE tab. PERFORM tab shortcuts only work on PERFORM tab. No more cross-tab interference.

---

## KEYBOARD QUICK REFERENCE

### Audio Focus Mode (Shift+A) - CREATE Tab

| Key | Action |
|-----|--------|
| `Shift+A` | Toggle mode |
| `V` | Volume panel |
| `C` | Cue panel |
| `E` | EQ panel |
| `F` | FX panel |
| `↑ / ↓` | Navigate rows |
| `← / →` | Adjust values |
| `Enter` | Kill/jump |
| `Escape` | Exit (nested) |

### PERFORM Tab

| Key | Action |
|-----|--------|
| `1-5` | Jump to Cue 1-5 |
| `Shift+1-4` | Select Track |
| `Shift+5` | Select Master |
| `C / E / F` | Open panel (after track select) |
| `W/A/S/D` | Navigate grid |
| `Enter` | Launch cell |
| `Space` | Stop track |

### Other Modes (CREATE Tab)

| Key | Action |
|-----|--------|
| `Shift+T` | Transform mode |
| `Shift+R` | Reactivity mode |
| `Shift+M` | Music Timing mode |
| `Shift+B` | BumpR mode |
| `G` | Toggle beatgrid |
| `Space` | Play/pause audio |

---

## WORKFLOW TIPS

### Using EQ During Performance

1. Select track (`Shift+1-4`)
2. Press `E` to open EQ panel
3. Use `↑/↓` to select H/M/L band
4. Use `←/→` to cut or boost
5. Press `Enter` to kill a band completely

### Using DJ FX

1. Select track (`Shift+1-4`)
2. Press `F` to open FX panel
3. Use `↑/↓` to select Washout/Dubby/Break
4. Use `←/→` to increase effect
5. **Washout:** Great for transitions (sweeps out bass)
6. **Dubby:** Add echo trails for dub vibes
7. **Break:** Vinyl stop for dramatic endings

### Quick Cue Jumping

In PERFORM tab, just press `1-5` to jump to cues on the selected track. No need to open a panel - it's instant!

### Audio Header at a Glance

The CREATE tab audio header now shows:
- Volume knob (left, cyan/magenta)
- Container name (scrolling if long)
- Current time display
- Swap and Remove buttons (right)

---

## BUG FIXES

- ✅ Multi-knob drag fixed (only dragged knob moves)
- ✅ Dropdowns blocked during focus modes
- ✅ Dual selection rings eliminated
- ✅ Tab isolation prevents cross-tab shortcuts
- ✅ Compatibility layer preserves old variable names
- ✅ Cue 4/5 hotkeys now work
- ✅ Master clock checks fixed for cue jumping
- ✅ Arrow key handling in all focus modes

---

## BREAKING CHANGES

### Cue Points Expanded
- **Before:** 3 cue points
- **After:** 5 cue points
- Legacy containers auto-padded with defaults

### FX Renamed
- Reverb → **Washout** (highpass sweep)
- Delay → **Dubby** (dub echo)
- Wash → **Break** (vinyl stop)

### Keyboard Pattern Changed
- **Before:** Direct panel toggle (Shift+E/F/C)
- **After:** Gate pattern (Shift+A then E/F/C)

### PERFORM Track Layout
- Nudge buttons removed
- Track modules added (Cue/EQ/FX)

---

## COMPATIBILITY

✅ Fully compatible with V5_1_0 and V5_0_0  
✅ All .oms containers work (3-cue containers padded to 5)  
✅ All .omsp projects work (EQ/FX initialized to defaults)  
✅ BumpR StickRs work as before  
✅ Transform controls work as before  
✅ Reactivity controls work as before

---

## INHERITED FROM V5_1_0

Everything from V5_1_0 is still here:
- BumpR StickRs (10-slot video overlays)
- 16 animation effects
- Quantized timing (Beat/Bar sync)
- Per-slot settings
- Shift+B keyboard mode

---

## INHERITED FROM V5_0_0

Everything from V5_0_0 is still here:
- Transform controls (Rotation, Scale, Tile, XY, Pivot)
- Shift+T keyboard mode
- Shift+R reactivity mode
- Shift+scroll trackpad control
- OmsDropdown component
- 112-byte uniform struct
- All 28 shader uniforms

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**

Version: Beta V5_2_0  
Date: December 26, 2025  
License: GPL-3.0

*"Mix it. EQ it. FX it. Cue it."*
