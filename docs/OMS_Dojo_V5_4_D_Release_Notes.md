# ONEMANSHYO App [Beta V5_4_0] - RELEASE NOTES

**Release Date:** December 31, 2025  
**Version:** Beta V5_4_0  
**Type:** Professional DJ Workflow & Output Infrastructure

---

## WHAT'S NEW

### 🎧 Audio Card & Cue Bus (Major!)

**Pioneer DJM-style headphone cueing!**

V5_4_0 introduces professional Pre-Fader Listen (PFL) routing. Configure separate Master and Cue outputs to preview tracks in your headphones before they hit the speakers.

| Output | Purpose |
|--------|---------|
| Master | FOH / Speakers (audience hears) |
| Cue | Headphones (only you hear) |

**Setup:**
1. Go to ADMIN → Audio Card
2. Select your audio interface for Master Output
3. Select your headphones for Cue Output
4. Use Test buttons to verify routing

**Mix Knob (Pioneer-Style):**
| Position | Main in Headphones | Cue in Headphones |
|----------|-------------------|-------------------|
| Left | 100% | 0% |
| Center | 50% | 50% |
| Right | 0% | 100% |

**S→C Button:**
- When Cue output is configured, Solo buttons become Cue buttons (orange)
- Press C to hear that track in your headphones (Pre-Fader!)
- Track at 0% volume? Still audible in cue!

**Requirements:** Chrome/Edge 110+ and multi-channel audio interface (Behringer UMC204HD/404HD recommended)

---

### 🎚️ MAIN Column

**Dedicated output bus controls!**

The right side of PERFORM tab now has a proper MAIN column with stereo metering and output controls.

| Section | Content |
|---------|---------|
| Visual (V) | Purple meter, opacity readout, M button, opacity knob |
| Audio (A) | Stereo L/R meters, peak dB readout, M button, volume knob |
| Bottom | Clock, Quantize, Zoom, Tempo, M/T/W buttons |

**Stereo Metering:**
- True L/R channel separation
- Real-time peak detection
- dB readout shows max of L/R

**Note:** "Master" now means timing/grid systems. "MAIN" means output bus.

---

### 🔊 DSP Output Processing

**Master bus compression!**

Click the DSP button in the MAIN column to open the compressor/limiter slideout.

**Presets:**
| Preset | Threshold | Ratio | Output |
|--------|-----------|-------|--------|
| Off | 0 dB | 1:1 | 0 dB |
| Gentle | -24 dB | 2:1 | +2 dB |
| Medium | -18 dB | 4:1 | +4 dB |
| Aggressive | -12 dB | 8:1 | +6 dB |
| Limiter | -6 dB | 20:1 | +3 dB |

**Controls:**
- Threshold: -48 to 0 dB
- Ratio: 1:1 to 20:1
- Output: -12 to +12 dB makeup gain
- GR Meter: Shows real-time gain reduction

**Tip:** Start with "Medium" preset for glue, switch to "Limiter" for safety.

---

### ✨ VFX Post-Processing

**GPU-powered visual effects!**

Click the VFX button in the MAIN column to open the effects slideout.

**Available Effects (10):**
| Effect | Description |
|--------|-------------|
| Off | No processing |
| RGB Split | Chromatic aberration |
| Glitch | Block displacement |
| Pixelate | Resolution reduction |
| Scanlines | CRT effect |
| Feedback | Frame blending |
| Mirror | Horizontal reflection |
| Kaleidoscope | Radial segments |
| VHS | Tracking errors |
| Shake | Camera shake |

**Controls:**
- Effect dropdown (OmsDropdown style)
- Amount knob: 0-100%
- Double-click knob to reset to 0%
- Selecting "Off" auto-resets amount (live-safe!)

**Note:** Requires WebGPU support (Chrome/Edge)

---

### 📊 Meter System

**Toggle overlay with V/A modules!**

Click the split-cube button (purple/cyan gradient) above the grid to show the meter overlay.

| Per-Track Module | Content |
|------------------|---------|
| Visual (left) | Purple bar, opacity % |
| Audio (right) | Stereo L/R bars, peak dB |

**Color Coding:**
| Zone | dB Range | Color |
|------|----------|-------|
| Safe | -48 to -18 | Green |
| Warm | -18 to -6 | Yellow |
| Hot | -6 to 0 | Red |

**MAIN meters are always visible** (no toggle needed).

---

### 🎬 Half-Height Grid Cells

**See 2× more scenes!**

Grid cells are now half-height with a split layout showing both shader and audio content.

| Left Half | Right Half |
|-----------|------------|
| Shader thumbnail (80×45) | Waveform bars |
| 30% purple if no shader | 30% cyan if no audio |

**State Indicators:**
| State | Visual |
|-------|--------|
| Playing | Yellow tint + outline |
| Selected (cursor) | Cyan outline (always on top) |
| Empty | "Empty" text centered |

**Ghost Cell Fix:** Empty cells in a scene no longer get yellow .playing state!

---

### 🏷️ Scene Labels & Colors

**Customize your scenes!**

Each scene trigger can have a custom label (up to 128 characters) and one of 8 color presets.

**Editing in PERFORM:**
| Action | Result |
|--------|--------|
| Click + hold 500ms | Inline label edit |
| Right-click | Color picker popup |

**Editing in ADMIN:**
- Go to ADMIN → Application Labels
- Edit labels for all 8 scenes
- Select colors from dropdown

**Color Presets:**
Cyan, Magenta, Orange, Yellow, Green, Purple, Red, Default (grey)

**Ticker Animation:** Long labels automatically scroll horizontally (6-second loop)

---

### ⚙️ ADMIN Tab Restructure

**Flat alphabetical navigation!**

The ADMIN tab now has 13 flat sections (no nesting):

| Section | Content |
|---------|---------|
| About | App info |
| Application | General settings |
| Application Labels | Scene/track names |
| Audio Card | Device routing |
| Audio Cues | Cue point defaults |
| Audio EQ | EQ defaults |
| Audio FX | FX defaults |
| Audio Loop | Loop defaults |
| Audio Master Tempo | PERFORM default tempo |
| Container Tags | Tag management |
| Shader Tempo | CREATE default tempo |
| Utilities | Tools |
| Visual BumpR StickRs | Video overlays |

**Keyboard:**
| Key | Action |
|-----|--------|
| ↑ | Previous section (wraps) |
| ↓ | Next section (wraps) |

---

### 🛑 Safety Features

**ESC Double-Tap Protection!**

Accidentally hitting ESC won't kill your set anymore.

| Press | Result |
|-------|--------|
| Single ESC | Magenta warning message |
| Double ESC (within 500ms) | Executes Stop All |

**Stop All Button:**
- 16×16 mini button with ■ symbol
- Located in mixerScenes area
- Red flash on activation

---

## KEYBOARD QUICK REFERENCE

### PERFORM Tab

| Key | Action |
|-----|--------|
| ESC ESC | Stop All (double-tap required) |
| ↑ / ↓ | ADMIN section navigation |

### ADMIN Tab

| Key | Action |
|-----|--------|
| ↑ | Previous section |
| ↓ | Next section |

---

## WORKFLOW TIPS

### Setting Up Headphone Cueing

1. Connect multi-channel audio interface
2. Go to ADMIN → Audio Card
3. Select your interface for Master Output
4. Select headphones for Cue Output
5. Click Test buttons to verify
6. Solo buttons now become orange C (Cue) buttons
7. Press C on any track to preview in headphones!

### Using DSP Compression

1. Click DSP button in MAIN column
2. Select preset (start with "Medium")
3. Watch GR meter during loud passages
4. Adjust threshold if too much/little compression
5. Use output gain for makeup

### Applying VFX

1. Click VFX button in MAIN column
2. Select effect from dropdown
3. Adjust amount knob
4. Watch preview update in real-time
5. Double-click amount to reset quickly
6. Selecting "Off" auto-resets (performance-safe)

### Organizing with Scene Labels

1. Right-click scene trigger for color
2. Click+hold for label edit
3. Use short names that fit
4. Long names automatically ticker-scroll
5. Labels persist in project state

---

## BUG FIXES

- ✅ Grid visibility works on first load
- ✅ Track grid calculates correctly without audioSnapLastBar
- ✅ Project state loads from any tab
- ✅ Audio devices auto-scan when not cached
- ✅ Master shader opacity applied in both render loops
- ✅ Message bar visible (restored 5 missing </div> tags)
- ✅ Ghost cells don't get .playing state
- ✅ Yellow playing state doesn't cover cyan selection
- ✅ Library yellow highlight removed (useless)
- ✅ HTML removed with proper JS cleanup (no orphan references)

---

## BREAKING CHANGES

### Master → MAIN Rename
- **Before:** `masterShaderOpacity`, `masterVolume`, `masterGainNode`
- **After:** `trackMainShaderOpacity`, `trackMainVolume`, `trackMainGainNode`
- 64 occurrences renamed internally
- No user-facing impact

### New Project State Fields
```javascript
audioCardSettings: { masterOutputId, cueOutputId, cueMix }
trackCues: [false, false, false, false]
sceneLabels: ['1'-'8']
sceneColors: [null × 8]
audioPrefs: { masterTempo: 120 }
dspSettings: { enabled, preset, threshold, ratio, output }
vfxSettings: { effect, amount }
```

### Browser Requirements
- Chrome/Edge 110+ for Audio Card (setSinkId)
- WebGPU required for VFX pipeline

---

## COMPATIBILITY

✅ Fully compatible with V5_3_0, V5_2_0, V5_1_0, V5_0_0  
✅ All .oms containers work unchanged  
✅ All .omsp projects work (new fields use defaults)  
✅ Real audio reactivity preserved  
✅ Mod Tuning panel preserved  
✅ Sets and Tags systems preserved

---

## INHERITED FROM V5_3_0

Everything from V5_3_0 is still here:
- Real Audio Reactivity (bandpass peak detection)
- Mod Tuning Panel (N key, ADSR envelope)
- Unified Sets System (manual + smart)
- Container Tags
- Proper Peak Metering
- Quantize Control (Off/Beat/Bar)
- Draggable Bar 1 Marker

---

## INHERITED FROM V5_2_0

Everything from V5_2_0 is still here:
- Audio Focus Mode (Shift+A)
- 3-Band EQ System (H/M/L)
- DJ FX System (Washout/Dubby/Break)
- 5 Cue Points per container
- Per-Track Audio Processing
- Centralized Focus Mode

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

Version: Beta V5_4_0  
Date: December 31, 2025  
License: GPL-3.0

*"Professional output. Professional workflow. Professional results."*
