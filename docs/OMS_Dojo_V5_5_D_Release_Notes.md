# ONEMANSHYO App [Beta V5_5_0] - RELEASE NOTES

**Release Date:** January 3, 2026  
**Version:** Beta V5_5_0  
**Type:** Creative Expression & Workflow Efficiency

---

## WHAT'S NEW

### 🔄 Transform Animation System (Major!)

**Beat-synced rotation, scale, and tile!**

V5_5_0 introduces automatic transform animation modes that stay locked to your music. No more static visuals - everything moves with the beat.

| Transform | Modes | Range |
|-----------|-------|-------|
| Rotation | Stop, CW, CCW, PingPong | 0-360° |
| Scale | Stop, Grow, Shrink, PingPong | 0.1-4.0 |
| Tile | Stop, Increase, Decrease, PingPong | 1-8 |

**Mode Keys (in Transform Focus - Shift+T):**
| Key | Mode | Behavior |
|-----|------|----------|
| 0 | Stop | Manual control only |
| 1 | CW/Grow/Increase | Ramps up, resets |
| 2 | CCW/Shrink/Decrease | Ramps down, resets |
| 3 | PingPong | Bounces between min/max |

**Speed Control:**
- Use trackpad scroll while in Transform Focus
- Range: 0.03× to 256× (exponential)
- Tiny scrolls = fine adjustment, big scrolls = big jumps

**Loop Length Dropdown (Reordered):**
```
No Loop → 1/64 → 1/32 → 1/16 → 1/8 → 1/4 → 1/2 → 1 Bar → 2 → 4 → 8 → 16 → 32 Bars
```

**Persistence:** Modes and speeds save to .oms containers!

---

### 🎛️ Per-Mod ADSR+T Envelopes (Major!)

**Independent envelope controls for each frequency band!**

Each modulator (Bass/Mids/Treble) now has its own Q, Attack, Decay, Sustain, Release, and Threshold settings.

| Parameter | Mod A (Bass) | Mod B (Mids) | Mod C (Treble) |
|-----------|--------------|--------------|----------------|
| Q | 0.70 | 0.70 | 0.70 |
| Attack | 5ms | 10ms | 2ms |
| Decay | 20ms | 30ms | 15ms |
| Sustain | 70% | 80% | 60% |
| Release | 50ms | 100ms | 80ms |
| Threshold | 10% | 5% | 5% |

**N-Panel Sync:**
- Panel now syncs to selected Reactivity row
- Navigate rows with ↑/↓ in Reactivity Focus
- N-panel automatically shows that mod's envelope
- Live readouts show current values (ms/%)

**Threshold Expanded:** Now 0-100% (was 0-20%)

**Persistence:** All 18 parameters save to .oms containers!

---

### 📹 PIP Camera System (Major!)

**Picture-in-Picture overlays for streaming and recording!**

Add your webcam to your visuals with 3 PIP types plus a full-screen overlay.

| PIP | Key | Location | Tabs |
|-----|-----|----------|------|
| Overlay | O | Full screen | ALL |
| PIP 1 | P | Slideout above shader | PERFORM |
| PIP 2 | P | Top-right corner | CREATE/ORG/ADMIN |
| PIP 3 | Shift+P | Canvas center | CREATE/ORG/ADMIN |

**Shape Options:**
| Shape | Aspect | Border |
|-------|--------|--------|
| Landscape | 16:9 | Square corners |
| Portrait | 9:16 | Square corners |
| Square | 1:1 | Square corners |
| Circle | 1:1 | Fully rounded |

**Color Options (9):**
None (transparent), Cyan, Magenta, Orange, Yellow, Green, Purple, Pink, White

**Border-Click Color Picker:**
- Click anywhere on the PIP border
- 9 color swatches appear
- Click to select, click outside to dismiss

**Genie Zoom Animation:**
- PIPs scale in/out smoothly
- Each has appropriate transform-origin

**ADMIN → Video Sources:**
- Select camera device
- Toggle mirror mode
- Configure shape/color for each PIP
- Test camera and copy diagnostic report

**Smart Stream Management:** Camera only stops when ALL outputs close!

---

### 📚 ORGANIZE Tab UX Overhaul

**Apple Music-style editing!**

No more OK/Cancel buttons. Changes save automatically when you move away.

| Action | Result |
|--------|--------|
| Edit field, click away | Auto-saves |
| Press Enter | Saves, moves to next field |
| Press Escape | Saves, closes drawer |
| Press ↑/↓ | Saves, navigates to prev/next container |

**Tag Autocomplete:**
| Feature | Behavior |
|---------|----------|
| Start typing | Matching tags appear |
| Shows | "House (5)" with usage count |
| Navigate | ↑/↓ arrows |
| Select | Enter or click |
| Limit | 8 suggestions max |

**Quick Rename:**
| Input | Action |
|-------|--------|
| Mouse | Long-press (500ms) on name |
| Keyboard | Enter key on selected item |

**Selection Visual States:**
| State | Border | Background |
|-------|--------|------------|
| Default | #333 | 3% white |
| Hover | Cyan | unchanged |
| Selected | Cyan | 10% cyan |
| Loaded | Yellow | 10% yellow |
| Both | Yellow | 15% yellow |

---

### ⌨️ PERFORM Keyboard Workflow

**Rapid track loading with bare number keys!**

| Key | Action |
|-----|--------|
| 1-4 | Select track (no modifier needed!) |
| 0 | Deselect track |
| ↑/↓ | Navigate library (when track selected) |
| SHIFT+Right | Load to next empty cell |
| SHIFT+Left | Return focus to library |
| Enter | Load selected container to cursor |

**Quick-Load Scan Order:**
```
Scene 1: T1→T2→T3→T4
Scene 2: T1→T2→T3→T4
... (horizontal through all 8 scenes)
```

**Visual Feedback:**
| Action | Visual |
|--------|--------|
| Press 1-4 | Yellow highlight on track header |
| Hold ←/→ | Purple ring on opacity knob |
| Hold ↑/↓ | Cyan ring on volume knob |

**Critical Bug Fix:** Containers loaded via SHIFT+Right now play at correct tempo!

---

### 🎚️ FX Overlay System

**Per-track VFX and FX controls!**

Click the orange FX button (next to meter toggle) to show the overlay.

| Column | Top Module | Bottom Module |
|--------|------------|---------------|
| Per Track | VFX (A/B/C) | FX (R/D/W) |

**Knob Size:** All knobs standardized to 20×20px

**VFX Knobs:**
- Purple arc color
- Interactive (drag works)
- Placeholder - backend not yet connected

---

## KEYBOARD QUICK REFERENCE

### Universal

| Key | Action |
|-----|--------|
| O | Toggle camera overlay (all tabs) |

### CREATE Tab

| Key | Action |
|-----|--------|
| Shift+T | Enter Transform Focus |
| 0/1/2/3 | Mode: Stop/CW/CCW/PingPong |
| Trackpad | Adjust speed (in Transform Focus) |
| N | Toggle Mod Tuning panel |
| P | Toggle PIP 2 (corner) |
| Shift+P | Toggle PIP 3 (center) |

### ORGANIZE Tab

| Key | Action |
|-----|--------|
| Enter | Quick rename selected |
| CMD+I | Toggle info drawer |
| ↑/↓ | Navigate containers (auto-saves) |
| Escape | Save and close drawer |

### PERFORM Tab

| Key | Action |
|-----|--------|
| 1-4 | Select track |
| 0 | Deselect track |
| ↑/↓ | Navigate library |
| SHIFT+Right | Quick-load to next empty |
| SHIFT+Left | Return to library focus |
| P | Toggle PIP 1 (slideout) |

---

## WORKFLOW TIPS

### Setting Up Transform Animations

1. Enter Transform Focus (Shift+T)
2. Select transform column (←/→)
3. Press 1 for clockwise, 2 for counter, 3 for pingpong
4. Scroll trackpad to adjust speed
5. Set Loop Length dropdown for timing
6. Save to container - modes persist!

### Tuning Per-Mod Envelopes

1. Enter Reactivity Focus (Shift+R)
2. Select row (↑/↓) - Mod A, B, or C
3. Press N to open envelope panel
4. Panel shows that mod's settings
5. Adjust Q for filter sharpness
6. Adjust Attack for transient response
7. Navigate to another row - panel updates!

### Using PIP Camera

1. Press O for full-screen overlay (quick check)
2. In PERFORM, press P for slideout PIP
3. In CREATE, press P for corner PIP
4. Click border to change color
5. Go to ADMIN → Video Sources for shape control
6. Settings persist in project state

### Quick-Loading a Set

1. Go to PERFORM tab
2. Press 1 to select Track 1
3. Navigate library with ↑/↓
4. Press SHIFT+Right to load to next empty
5. Repeat - library auto-advances!
6. Press SHIFT+Left when done

### Apple Music-Style Editing

1. Open ORGANIZE tab
2. Click container to select
3. CMD+I to open drawer (or it auto-opens)
4. Edit any field directly
5. Press ↑ or ↓ to navigate - auto-saves!
6. No OK button needed

---

## BUG FIXES

- ✅ Transform animation now beat-synced (was frame-based)
- ✅ Dropdown no longer blocks Transform Focus entry
- ✅ Arrow keys navigate ORGANIZE properly
- ✅ Built-in sets protected from rename
- ✅ Ghost cells cleared on navigation
- ✅ **Audio timing correct on quick-load** (critical fix!)
- ✅ Knobs standardized to 20×20px
- ✅ PIP animation origins correct per-pip

---

## BREAKING CHANGES

### Per-Mod Envelope Schema
- **Before:** Global `modAttack`, `modDecay`, `modSustain`, `modRelease`, `modThreshold`
- **After:** Per-mod `modAAttack`, `modBAttack`, `modCAttack`, etc. (18 fields)
- **Migration:** Old containers use global defaults

### Transform Modes Schema
New fields in controlStates:
```javascript
transformRotationMode: 'stop'    // stop|clockwise|counterclockwise|pingpong
transformRotationSpeed: 1.0      // 0.03-256
transformScaleMode: 'stop'
transformScaleSpeed: 1.0
transformTileMode: 'stop'
transformTileSpeed: 1.0
```

### Project State (.omsp) Additions
```javascript
videoSourcesSettings: { cameraSourceId, cameraMirror }
pip1Settings: { visible, color, label, shape }
pip2Settings: { color, shape }
pip3Settings: { color, shape }
```

### Browser Requirements
- Camera permissions required for PIP system
- WebGPU required for VFX (inherited from V5_4_0)

---

## COMPATIBILITY

✅ Fully compatible with V5_4_0, V5_3_0, V5_2_0, V5_1_0  
✅ All .oms containers work unchanged  
✅ All .omsp projects work (new fields use defaults)  
✅ Per-mod envelopes fall back to global defaults  
✅ Transform modes default to 'stop'  
✅ Uniform struct unchanged (112 bytes)

---

## INHERITED FROM V5_4_0

Everything from V5_4_0 is still here:
- Audio Card & Cue Bus (Pioneer-style PFL)
- MAIN Column with stereo metering
- DSP Output Processing (compressor/limiter)
- VFX Post-Processing (10 effects)
- Half-Height Grid Cells
- Scene Labels & Colors
- ESC Double-Tap Safety
- ADMIN flat navigation

---

## INHERITED FROM V5_3_0

Everything from V5_3_0 is still here:
- Real Audio Reactivity (bandpass peak detection)
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

Version: Beta V5_5_0  
Date: January 3, 2026  
License: GPL-3.0

*"Express yourself. Move with the beat. Stream like a pro."*
