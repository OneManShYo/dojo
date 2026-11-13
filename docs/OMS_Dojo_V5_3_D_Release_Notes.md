# ONEMANSHYO App [Beta V5_3_0] - RELEASE NOTES

**Release Date:** December 27, 2025  
**Version:** Beta V5_3_0  
**Type:** Real Audio Reactivity & Library Organization

---

## WHAT'S NEW

### 🎚️ Real Audio Reactivity (Major!)

**Your shaders now ACTUALLY respond to audio!**

Previous versions used FFT bin averaging that produced constant ~0.02 values regardless of audio content. V5_3_0 replaces this with bandpass-filtered peak detection that captures real dynamics.

| Before | After |
|--------|-------|
| Kick hit → 0.02 | Kick hit → 0.72 |
| Silence → 0.02 | Silence → 0.01 |
| Hi-hat → 0.02 | Hi-hat → 0.15 |
| Constant sausage | Real dynamics! |

**What This Means:**
- Shaders react to actual transients (kick hits, snares)
- Different frequency ranges produce different responses
- Freq1/Freq2 knobs now matter
- Audio reactivity is no longer fake

---

### 🎛️ Mod Tuning Panel (N Key)

**Professional envelope shaping for your modulators!**

Press `N` while in Reactivity Focus (Shift+R) to open the Mod Tuning panel. This Blender-style slide-out gives you precise control over how modulators respond to audio.

**Per-Modulator Q Knobs:**
| Control | Range | Effect |
|---------|-------|--------|
| Mod A Q | 0.5-10 | Filter sharpness for bass |
| Mod B Q | 0.5-10 | Filter sharpness for mids |
| Mod C Q | 0.5-10 | Filter sharpness for highs |

**Global ADSR Envelope:**
| Parameter | Range | Default | What It Does |
|-----------|-------|---------|--------------|
| Attack | 0-100ms | 5ms | How fast mod rises to peak |
| Decay | 0-200ms | 20ms | How fast mod falls to sustain |
| Sustain | 0-100% | 70% | Level held while signal present |
| Release | 0-500ms | 50ms | How fast mod returns to zero |
| Threshold | 0-20% | 1% | Noise gate (signals below ignored) |

**Keyboard:**
| Key | Action |
|-----|--------|
| `Shift+R` | Enter Reactivity Focus |
| `N` | Toggle Mod Tuning panel |
| Drag knobs | Adjust values |
| `Escape` | Close panel |

---

### 📁 Unified Sets System

**Manual and Smart sets in one place!**

Sets are now unified under a single system. Both manual sets (drag containers) and smart sets (saved searches) live in the `/Sets/` folder and appear in the same sidebar list.

**Creating Sets:**
| Type | How to Create |
|------|---------------|
| Manual | Click "+ Set", drag containers in |
| Smart | Search something, click "Save" |

**Visual Indicator:**
- **Cyan dot** = Smart set (saved search)
- **No dot** = Manual set

**Panel Navigation:**
| Action | Result |
|--------|--------|
| Click "Sets" header | Focus sets panel |
| Click "Containers" header | Focus containers panel |
| `↑ / ↓` | Navigate within panel |
| Double-click name | Rename inline |

---

### 🏷️ Container Tags

**Organize your library with tags!**

Every container can now have multiple tags for easy organization and searching.

**Adding Tags:**
1. Open Info Drawer (click container name or press `I`)
2. Find Tags section
3. Type tag name + Enter (or comma)
4. Tags appear as chips

**Features:**
- Type + Enter creates chip
- × button removes chip
- Backspace on empty removes last chip
- Duplicates prevented (case-insensitive)
- Tags included in search matching

**Admin Panel (ADMIN → Containers → Tags):**
- View all tags with container counts
- Click to rename (affects all containers)
- Delete tag from entire library

---

### 📊 Proper Peak Metering (PERFORM)

**Real dBFS meters!**

PERFORM tab meters now show actual peak levels instead of meaningless FFT averages.

| Zone | dB Range | Color |
|------|----------|-------|
| Safe | -48 to -18 | Green |
| Warm | -18 to -6 | Yellow |
| Hot | -6 to 0 | Red |

The meters respond to actual audio dynamics - you'll see them jump on transients and settle during quiet passages.

---

### ⏱️ Quantize Control (PERFORM)

**Choose your launch timing!**

The old RMS input is now a Quantize dropdown with three options:

| Option | Behavior |
|--------|----------|
| Off | Immediate launch |
| 1 Beat | Snaps to next beat |
| 1 Bar | Snaps to next bar (default) |

**Forward-Only:** Quantize now always snaps forward, never backward. No more timing surprises!

---

### 📍 Draggable Bar 1 Marker (CREATE)

**Set your beatgrid by ear!**

The Bar 1 marker on the waveform is now draggable. Click and drag the white "1" marker to set exactly where the downbeat falls.

| Visual | Meaning |
|--------|---------|
| Grey waveform | Before Bar 1 |
| Blue waveform | After Bar 1 |

**Tip:** Play the track, find the first downbeat, pause, drag Bar 1 to that spot.

---

### 🔊 Volume dB Scale

**Industry-standard volume mapping!**

Volume now follows Ableton-style dB mapping across both CREATE and PERFORM tabs.

| Knob Position | Gain | dB |
|---------------|------|-----|
| 0% (min) | 0.0× | -∞ |
| 50% (noon) | 1.0× | 0 dB (unity) |
| 100% (max) | 2.0× | +6 dB |

---

## KEYBOARD QUICK REFERENCE

### Mod Tuning (CREATE Tab)

| Key | Action |
|-----|--------|
| `Shift+R` | Enter Reactivity Focus |
| `N` | Toggle Mod Tuning panel |
| `Escape` | Close panel |

### Sets Panel (ORGANIZE Tab)

| Key | Action |
|-----|--------|
| Click header | Focus panel |
| `↑ / ↓` | Navigate sets/containers |
| `Enter` | Select/load |
| `Escape` | Exit focus |

### Quantize (PERFORM Tab)

| Key | Action |
|-----|--------|
| `[` | Tempo -0.5 BPM |
| `]` | Tempo +0.5 BPM |

---

## WORKFLOW TIPS

### Dialing In Audio Reactivity

1. Load audio with clear kick drums (bass music works great)
2. Enter Reactivity Focus (`Shift+R`)
3. Open Mod Tuning panel (`N`)
4. Play audio, watch the meters
5. Adjust Q for sharper frequency isolation
6. Adjust Attack/Decay for snappier or smoother response
7. Set Threshold to reject noise floor

### Using Smart Sets

1. Type a search term (e.g., "bass")
2. See filtered results
3. Click "Save" to create smart set
4. Smart set automatically updates when you add matching containers

### Organizing with Tags

1. Create tags for categories: "bass", "melodic", "128", "drops"
2. Tag your containers as you build them
3. Search by tag name to find related content
4. Use Admin panel to rename/delete tags library-wide

### Setting Beatgrid by Ear

1. Load audio in CREATE tab
2. Press Space to play
3. Listen for the first clear downbeat
4. Press Space to pause
5. Drag the white "1" marker to align with the downbeat
6. Resume playback to verify alignment

---

## BUG FIXES

- ✅ Grid now shows on first PERFORM tab visit
- ✅ Mixer state syncs correctly on project load
- ✅ Cue highlights no longer bleed across tracks
- ✅ Rename race condition fixed
- ✅ Quantize always snaps forward (never backward)
- ✅ Shader swap preserves container ID when audio loaded
- ✅ Negative bar numbers no longer display
- ✅ Scene double-click works correctly
- ✅ Master controls respond to clicks

---

## BREAKING CHANGES

### Audio Reactivity (Timeline Arrays)
- **Before:** Fake constant values (~0.02)
- **After:** Real dynamic values (0.01-0.80+)
- Existing containers work but may benefit from re-analysis

### Sets System
- **Before:** Separate manual/smart caches
- **After:** Unified cache with type field
- Legacy smart sets auto-migrated to files

### Variable Names
- `performActivePanel` → `liveActivePanel`
- `npanel*` → `mod*` (Mod Tuning panel)

### New Schema Fields
- `tags: []` - Container tag array
- `modTuning: {}` - Envelope settings object

---

## COMPATIBILITY

✅ Fully compatible with V5_2_0, V5_1_0, V5_0_0  
✅ All .oms containers work (new fields use defaults)  
✅ All .omsp projects work (legacy smart sets migrated)  
✅ EQ3 and DJ FX work as before  
✅ Audio Focus Mode works as before  
✅ Per-track processing works as before

---

## INHERITED FROM V5_2_0

Everything from V5_2_0 is still here:
- Audio Focus Mode (Shift+A)
- 3-Band EQ System (H/M/L)
- DJ FX System (Washout/Dubby/Break)
- 5 Cue Points per container
- Per-Track Audio Processing
- Centralized Focus Mode
- Tab Isolation

---

## INHERITED FROM V5_1_0

Everything from V5_1_0 is still here:
- BumpR StickRs (10-slot video overlays)
- 16 animation effects
- Quantized timing options
- Shift+B keyboard mode

---

## INHERITED FROM V5_0_0

Everything from V5_0_0 is still here:
- Transform controls (Rotation, Scale, Tile, XY, Pivot)
- Shift+T keyboard mode
- Shift+R reactivity mode
- 112-byte uniform struct
- All 28 shader uniforms

---

## FEEDBACK

- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**

Version: Beta V5_3_0  
Date: December 27, 2025  
License: GPL-3.0

*"Real reactivity. Real dynamics. Real results."*
