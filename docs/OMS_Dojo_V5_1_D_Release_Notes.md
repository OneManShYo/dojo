# ONEMANSHYO App [Beta V5_1_0] - RELEASE NOTES

**Release Date:** December 24, 2025  
**Version:** Beta V5_1_0  
**Type:** BumpR StickRs & Cue Points

---

## WHAT'S NEW

### 🎬 BumpR StickRs - Video Overlays

**Fire branded video clips during your set!**

BumpR StickRs is a 10-slot video overlay system for personal branding during live performance. Drop WebM files with transparency and trigger them with keyboard shortcuts.

| Feature | Description |
|---------|-------------|
| 10 Slots | Store 10 different video clips |
| Alpha Support | WebM VP9 with transparency |
| CDJ-Style | Retrigger replaces current playback |
| Per-Slot Settings | Each slot has its own Scale, Timing, Animate, Duration |

**Location:** ADMIN → Settings → BumpR StickRs

---

### ⌨️ BumpR Keyboard Mode (Shift+B)

**Trigger videos like CDJ hot cues!**

| Key | Action |
|-----|--------|
| `Shift+B` | Enter/exit BumpR mode |
| `1-9, 0` | Trigger slots 1-10 directly |
| `↑ / ↓` | Navigate between slots |
| `Enter` | Trigger selected slot |
| `Escape` | Exit mode |

A slideout panel shows thumbnails of all 10 slots. Empty slots are dimmed.

---

### 🎛️ Per-Slot Settings

**Customize each slot independently!**

| Setting | Options |
|---------|---------|
| **Scale** | 25%, 50%, 75%, 100% |
| **Timing** | Instant, Beat-sync, Bar-sync |
| **Animate** | 16 animation effects |
| **Duration** | Original, Loop, Musical bars, Timed seconds |

**Timing is tab-aware:**
- PERFORM tab: Syncs to master clock
- CREATE tab: Syncs to audio playback

---

### ✨ 16 Animation Effects

**Add motion to your video overlays!**

| Effect | Style |
|--------|-------|
| Bounce | Pulsing scale |
| Breathe | Slow gentle pulse |
| Crazy | Everything at once! |
| Drift | Slow diagonal float |
| Drop | Fall and bounce (once) |
| Float | Gentle bob up/down |
| Glitch | Rapid position jitter |
| Pop | Scale entrance (once) |
| Pulse | Fading in/out |
| Shake | Quick wiggle |
| Spin | Continuous rotation |
| Spiral | Spin while scaling |
| Strobe | Rapid flash |
| Wobble | Tilting left/right |
| Zoom | Scaling in and out |

---

### 🎯 Cue Points

**CDJ-style hot cues for instant navigation!**

3 cue points per container, stored in beats. Visual orange flags on the waveform.

| Cue | Default Position | Typical Use |
|-----|------------------|-------------|
| 1 | Bar 1.1 | Intro |
| 2 | Bar 17.1 | First drop |
| 3 | Bar 33.1 | Second drop |

**CREATE Tab:**
- Orange flags visible on waveform
- **Click** a flag to jump playhead
- **Drag** a flag to reposition (snaps to bar)
- Cues save with your .oms container

**PERFORM Tab:**
- Press `1`, `2`, or `3` to jump to cues
- Instant jump (no quantization)
- Works on selected track

---

### 🎹 PERFORM Keyboard Changes

**Cue jumping is now the priority!**

Since cuing is urgent during performance, plain number keys now jump to cues:

| Before | After |
|--------|-------|
| `1` = Select Track 1 | `1` = Jump to Cue 1 |
| `2` = Select Track 2 | `2` = Jump to Cue 2 |
| `3` = Select Track 3 | `3` = Jump to Cue 3 |
| `4` = Select Track 4 | `Shift+1` = Select Track 1 |
| `5` = Select Master | `Shift+2` = Select Track 2 |
| - | `Shift+3` = Select Track 3 |
| - | `Shift+4` = Select Track 4 |
| - | `Shift+5` = Select Master |

Track selection is a setup task (less urgent), cuing is a performance task (more urgent).

---

### 📦 Container Output Section

**Clearer organization of output options!**

The OUTPUT section is now called "CONTAINER OUTPUT" to clarify that it creates containers (shader + audio + settings), not just video files.

**Button Changes:**
- `[Save]` and `[Update]` moved to CONTAINER OUTPUT header
- `[Save]` always creates a NEW container
- `[Update]` saves changes to the loaded container
- `[Update]` only visible when a container is loaded

**Hover for help:** All controls show info messages on hover.

---

## KEYBOARD QUICK REFERENCE

### BumpR Mode (Shift+B)

| Key | Action |
|-----|--------|
| `Shift+B` | Toggle mode |
| `1-9, 0` | Trigger slot |
| `↑ / ↓` | Navigate |
| `Enter` | Trigger selected |
| `Escape` | Exit |

### PERFORM Tab

| Key | Action |
|-----|--------|
| `1 / 2 / 3` | Jump to Cue 1/2/3 |
| `Shift+1-5` | Select Track/Master |
| `W/A/S/D` | Navigate grid |
| `Enter` | Launch cell |
| `Space` | Stop track |

### CREATE Tab (unchanged)

| Key | Action |
|-----|--------|
| `Shift+T` | Transform mode |
| `Shift+R` | Reactivity mode |
| `G` | Toggle beatgrid |
| `Space` | Play/pause audio |

---

## WORKFLOW TIPS

### Setting Up BumpR StickRs

1. Go to ADMIN → Settings → BumpR StickRs
2. Drag WebM files onto empty slots (or anywhere on page)
3. Configure Scale, Timing, Animate, Duration per slot
4. Settings auto-save to your project

### Using BumpR During Performance

1. Press `Shift+B` to arm BumpR mode
2. Press `1-9` or `0` to fire clips
3. Retrigger same slot to restart
4. Trigger different slot to switch
5. Press `Escape` or `Shift+B` to exit

### Creating WebM Files with Alpha

Use FFmpeg to convert videos with transparency:
```bash
ffmpeg -i input.mov -c:v libvpx-vp9 -pix_fmt yuva420p output.webm
```

### Setting Cue Points

1. Load audio in CREATE tab
2. Drag orange cue flags to your preferred positions
3. Flags snap to bar boundaries
4. Save container - cues are stored!

### Using Cues in PERFORM

1. Load container to grid cell
2. Select track (`Shift+1-4`)
3. Start playback (Enter)
4. Press `1`, `2`, or `3` to jump to cues

---

## BUG FIXES

- ✅ Delete key now works in ORGANIZE input fields
- ✅ Save button properly sets container ID for updates
- ✅ Cue points preserved when updating containers
- ✅ BroadcastChannel console spam eliminated
- ✅ Waveform background stays dark during video playback
- ✅ BumpR and Reactivity modes can't be active simultaneously

---

## COMPATIBILITY

✅ Fully compatible with V5_0_0  
✅ All .oms containers work (cues default to 1.1, 17.1, 33.1)  
✅ All .omsp projects work (BumpR settings initialized)  
✅ Transform controls work as before  
✅ Reactivity controls work as before

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

Version: Beta V5_1_0  
Date: December 24, 2025  
License: GPL-3.0

*"Fire your brand!"*
