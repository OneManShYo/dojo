# OMS SHADER BAKER V3.7.0 - COMPLETE CHANGELOG

**Version:** V3.7.0 Production Release  
**Date:** November 16, 2025  
**Base Version:** V3.6.0  
**Development Iterations:** 31 (V3_7_0_A through V3_7_0_ZE)

---

## OVERVIEW

V3.7.0 is a **major UI/UX release** that transforms the LIVE tab from functional to professional-grade instrument. This release introduces dual-domain mixer architecture (independent shader and audio controls per track), refined visual hierarchy, and pixel-perfect alignment that creates a hardware-inspired modular interface.

**Development Approach:** 31 surgical iterations over 1 day, ~365 lines modified

**Theme:** Dual-Domain Mixer + Professional UI Polish

---

## NEW FEATURES

### 1. Dual-Domain Mixer Architecture (MAJOR FEATURE)

**Description:** Complete restructure into independent visual and audio control domains per track.

**New Layout Per Track:**
```
┌─────────────────────────────────┐
│         [Track N]               │
├──────────────┬──────────────────┤
│   SHADER     │      AUDIO       │
│              │                  │
│  [vM] [vS]   │   [🎚️ volume]    │
│  [🎚️ opacity] │   [▮▮] [▮▮]      │
│              │   [L]   [R]      │
│              │   [aM] [aS]      │
└──────────────┴──────────────────┘
```

**Shader Domain Features:**

**Shader Opacity Knob:**
- Range: 0-100% (manual override)
- Color: Purple (#9d4edd)
- Behavior: Touching enables "MANUAL" override mode
- Overrides V3.6.0 volume-based opacity when set manually

**Shader Mute/Solo Buttons:**
- Independent from audio M/S buttons
- Mute active: Purple background
- Solo active: Cyan background
- Visual-only control (audio unaffected)

**Audio Domain Features:**
- Volume fader (audio output level)
- Level meters (L/R VU meters, 40px height)
- Audio mute/solo buttons (standard appearance)
- Independent from shader controls

**Keyboard Controls (NEW):**
- `Shift+,` - Toggle shader mute on selected track
- `Shift+.` - Toggle shader solo on selected track
- `Shift+↑` - Increase shader opacity (+2%)
- `Shift+↓` - Decrease shader opacity (-2%)

**Opacity Calculation Logic:**
New `calculateTrackShaderOpacity(track)` considers priority order:
1. Shader Mute → opacity 0
2. Shader Solo → opacity 0 if not soloed
3. Manual Override → use manual value
4. Volume-Based (Auto) → volume × 2.0 (V3.6.0 behavior)
5. Not Playing → opacity 0

**State Management:**
```javascript
trackShaderOpacity = [1.0, 1.0, 1.0, 1.0]  // Manual override values
trackShaderMute = [false, false, false, false]
trackShaderSolo = [false, false, false, false]
trackShaderManual = [false, false, false, false]  // Override enabled?
```

**MIDI-Ready Design:**
- Option A: Dual MIDI Fighters (one for shader, one for audio)
- Option B: Single MF + Shift button for domain switching
- Option C: Custom Yaeltex (8 encoders, dual M/S rows)

**User Benefit:** Complete separation of visual and audio mixing - fade visuals independently from audio for advanced performance techniques

**Iterations:** V3_7_0_J

---

### 2. Professional UI Polish (50% Smaller Knobs)

**Description:** Major CSS cleanup removing all labels and reducing track knob sizes by 50%

**Changes:**

**1. Removed "Not Connected" Dropdowns:**
- Track header dropdowns eliminated (legacy multi-app strategy)
- Clean track titles only

**2. Removed All Mixer Labels:**
- Track labels ("Track 1/2/3/4") - already labeled at grid top
- Domain labels ("SHADER"/"AUDIO") - position communicates function
- Volume labels ("-∞", "100%") - visual position clear
- Meter labels ("L"/"R") - standard stereo meters
- Override indicator ("MANUAL") - behavior self-evident

**3. Reduced Rotary Knob Size by 50%:**
- Original: 60px × 60px
- New: 30px × 30px
- Affected: All track knobs (volume, shader opacity)
- Drawing coordinates scaled proportionally
- Master/zoom knobs remain 60px (2x larger for prominence)

**4. Visual Hierarchy Through Size:**
- Small (30px) = detail work (track controls)
- Large (60px) = global control (master/zoom)
- Clear visual language without needing text labels

**Design Philosophy:** "The interface itself is the documentation" - position, color, and muscle memory communicate function better than redundant text

**User Benefit:** Cleaner, more professional appearance with stronger visual hierarchy

**Iterations:** V3_7_0_K

---

### 3. Master Mixer Enhancements

**Description:** Added M/S buttons and tempo integration to master channel

**Features:**

**Master Mute/Solo Buttons:**
- Positioned below master volume knob
- Match track M/S layout (symmetry)
- Control global audio output

**Tempo Display Integration:**
- Positioned below M/S buttons
- Shows current master tempo (60-200 BPM)
- ± buttons for adjustment
- Symmetrical vertical layout

**Compact Master Meters:**
- 120px height (vs 40px for tracks)
- 3x taller for prominence
- Clear visual hierarchy

**User Benefit:** Master channel now has parity with track channels in symmetrical modular layout

**Iterations:** V3_7_0_O, V3_7_0_P, V3_7_0_Q, V3_7_0_R, V3_7_0_S

---

### 4. Perfect Header Alignment (43px)

**Description:** 9 iterations to achieve pixel-perfect header alignment

**Problem:** Scenes header and Track headers had different heights, creating visual misalignment

**Solution Evolution:**
- V3_7_0_V: Attempted min-height matching
- V3_7_0_X: Reduced SCENES padding
- V3_7_0_Y: Added min-height + box-sizing
- V3_7_0_Z: Matched capitalization ("SCENES" → "Scenes")
- V3_7_0_ZB: Reduced grid-corner padding
- V3_7_0_ZC: Forced exact height (43px, not min-height)
- V3_7_0_ZD: Grey header bar + hidden scrollbar
- V3_7_0_ZE: Grid-headers container height match (FINAL FIX)

**Final Formula:**
```css
.grid-corner,
.track-header {
    height: 43px;           /* Force exact, not min */
    box-sizing: border-box; /* Include borders in height */
}

.grid-headers {
    height: 43px;           /* Container matches content */
}
```

**User Benefit:** Professional pixel-perfect alignment across all grid headers

**Iterations:** V3_7_0_V, V3_7_0_W, V3_7_0_X, V3_7_0_Y, V3_7_0_Z, V3_7_0_ZB, V3_7_0_ZC, V3_7_0_ZD, V3_7_0_ZE

---

### 5. Complete Modular Framing

**Description:** Added complete borders to all three main LIVE sections

**Sections:**
1. Grid Section (scenes/clips)
2. Mixer Section (knobs/faders)
3. Waveform Section (timeline)

**Visual Result:**
```
┌────────────────────┐
│  GRID SECTION      │ ← 2px border
└────────────────────┘
      (10px gap)
┌────────────────────┐
│  MIXER SECTION     │ ← 2px border
└────────────────────┘
      (10px gap)
┌────────────────────┐
│  WAVEFORM SECTION  │ ← 2px border
└────────────────────┘
```

**CSS:**
```css
.grid-section,
.mixer-section,
.waveforms-section {
    border: 2px solid #444;
    border-radius: 3px;
}

.performance-view {
    gap: 10px;  /* Uniform spacing */
}
```

**Visual Hierarchy:**
- Primary borders (2px): Module frames
- Secondary guidelines (0.5px): Internal dividers (reduced 50% from 1px)
- Rounded corners (3px): Softer, refined appearance

**User Benefit:** Professional modular hardware aesthetic, clear section separation

**Iterations:** V3_7_0_B, V3_7_0_C, V3_7_0_D, V3_7_0_E, V3_7_0_G, V3_7_0_I

---

### 6. Symmetrical Track Layout

**Description:** Shader and audio controls mirror each other vertically

**Layout Philosophy:**
- Left column: Shader controls (M/S buttons, opacity knob)
- Right column: Audio controls (volume knob, meters, M/S buttons)
- Mirror symmetry creates hardware-like tactile feel
- Parallel control domains immediately intuitive

**Design Quote:** "Symmetry is the language of hardware"

**User Benefit:** Hardware-inspired interface that feels like physical mixer

**Iterations:** V3_7_0_L, V3_7_0_M, V3_7_0_N

---

## IMPROVEMENTS

### Visual Refinements

**Reduced Secondary Guidelines (V3_7_0_G):**
- Internal borders: 1px → 0.5px (50% reduction)
- Cleaner, more refined console appearance
- Primary borders (2px) remain unchanged

**Rounded Module Corners (V3_7_0_I):**
- Added `border-radius: 3px` to all main modules
- Softer, more refined appearance
- Matches other modular elements

**Compact Meters (V3_7_0_M):**
- Track meters: 40px height
- Master meters: 120px height (3x taller)
- Clear visual hierarchy

**Hidden Scrollbar (V3_7_0_ZD):**
- Scenes column scrollbar invisible (still functional)
- Cross-browser support (Firefox, Chrome, Safari, Edge)
- Cleaner visual appearance

**Grey Header Bar (V3_7_0_ZD):**
- Grid-headers background: #0c0c0c → #2a2a2a
- Matches track header styling
- Unified visual language

### Bug Fixes

**Marquee Scroll Loop (V3_7_0_A):**
- Fixed visible "jerk" when looping
- Removed CSS padding-right
- Added visual separator: `name + '   ·   ' + name`
- Result: Smooth continuous marquee

**Mixer Padding Elimination (V3_7_0_E):**
- Removed double gap feeling
- Eliminated 10px internal padding
- More space for grid, compressed mixer

---

## TECHNICAL CHANGES

### Architecture

**New State Arrays:**
```javascript
// V3.7.0 additions
let trackShaderOpacity = [1.0, 1.0, 1.0, 1.0];
let trackShaderMute = [false, false, false, false];
let trackShaderSolo = [false, false, false, false];
let trackShaderManual = [false, false, false, false];
```

**New Functions:**
```javascript
calculateTrackShaderOpacity(track)  // Priority logic for opacity
toggleTrackShaderMute(track)        // Shader mute independent from audio
toggleTrackShaderSolo(track)        // Shader solo independent from audio
```

**Keyboard Handler Extensions:**
```javascript
// Shift+ modifiers for shader controls
case 'Comma':
    if (event.shiftKey) toggleTrackShaderMute(selectedTrack);
    else toggleTrackMute(selectedTrack);
    break;

case 'Period':
    if (event.shiftKey) toggleTrackShaderSolo(selectedTrack);
    else toggleTrackSolo(selectedTrack);
    break;

case 'ArrowUp':
    if (event.shiftKey) adjustShaderOpacity(selectedTrack, 0.02);
    else increaseVolume(selectedTrack);
    break;

case 'ArrowDown':
    if (event.shiftKey) adjustShaderOpacity(selectedTrack, -0.02);
    else decreaseVolume(selectedTrack);
    break;
```

### Performance

**GPU Usage (Same as V3.6.0):**
- 4 shaders rendering: ~30-45% (M1 MacBook Pro)
- 60fps stable with all tracks playing
- Dual-domain mixer adds minimal overhead (UI only, no shader impact)

**File Size:**
- V3.6.0: 475KB
- V3.7.0: 525KB (+50KB, +10.5%)
- ~365 lines added/modified across 31 iterations

### Settings Schema

**New Project State Fields:**
```javascript
{
    // ... existing fields
    trackShaderOpacity: [1.0, 1.0, 1.0, 1.0],
    trackShaderMute: [false, false, false, false],
    trackShaderSolo: [false, false, false, false],
    trackShaderManual: [false, false, false, false]
}
```

Saved in .omsp files under root level (backwards compatible)

---

## COMPLETE ITERATION LIST

### Phase 1: Visual Refinements (A-I)
- **V3_7_0_A:** Fix marquee scroll loop (~5 lines)
- **V3_7_0_B:** Add 10px border between mixer and waveforms (~3 lines)
- **V3_7_0_C:** Complete modular borders (~10 lines)
- **V3_7_0_D:** Complete modular framing (three sections) (~15 lines)
- **V3_7_0_E:** Remove mixer padding (~2 lines)
- **V3_7_0_G:** Reduce secondary guidelines by 50% (~20 lines)
- **V3_7_0_H:** Move mixer content up (~2 lines)
- **V3_7_0_I:** Rounded module corners (~3 lines)

### Phase 2: Dual-Domain Mixer (J)
- **V3_7_0_J:** DUAL-DOMAIN MIXER LAYOUT (+405 lines) **MAJOR FEATURE**

### Phase 3: CSS Cleanup (K-N)
- **V3_7_0_K:** CSS CLEANUP - labels removed, 50% knobs (-103 lines) **MAJOR**
- **V3_7_0_L:** Symmetrical track layout refinement (~20 lines)
- **V3_7_0_M:** Compact meter heights (~10 lines)
- **V3_7_0_N:** Final symmetry adjustments (~15 lines)

### Phase 4: Master Mixer (O-S)
- **V3_7_0_O:** Add M/S buttons to master (~60 lines)
- **V3_7_0_P:** Position tempo below M/S (~15 lines)
- **V3_7_0_Q:** Master layout refinement (~10 lines)
- **V3_7_0_R:** Master meter height (120px) (~5 lines)
- **V3_7_0_S:** Final master symmetry (~10 lines)

### Phase 5: Zoom Section (T-U)
- **V3_7_0_T:** Zoom section visual parity (~20 lines)
- **V3_7_0_U:** Zoom fine-tuning (~10 lines)

### Phase 6: Header Alignment (V-ZE, 9 iterations)
- **V3_7_0_V:** Attempt min-height match (~10 lines)
- **V3_7_0_W:** Scenes header padding (~5 lines)
- **V3_7_0_X:** SCENES header compact (~5 lines)
- **V3_7_0_Y:** All headers same height (~10 lines)
- **V3_7_0_Z:** SCENES header styling match (~3 lines)
- **V3_7_0_ZB:** SCENES header height fix attempt 2 (~5 lines)
- **V3_7_0_ZC:** Force exact header heights (~10 lines)
- **V3_7_0_ZD:** Grey bar + hide scrollbar (~15 lines)
- **V3_7_0_ZE:** Grid headers height match - FINAL FIX (~5 lines)

**Total:** 31 iterations, ~365 lines modified

---

## BREAKING CHANGES

None. V3.7.0 is fully backward compatible with V3.6.0 project files.

**Migration Notes:**
- Existing .omsp files load without modification
- New shader state fields added but defaults used if missing
- All V3.6.0 features preserved and functional
- All keyboard controls remain unchanged (Shift+ modifiers added)

---

## KNOWN ISSUES

**No Reset to Auto:**
- Manual shader opacity override can't be reset without reloading project
- Planned for V3.8.0: Double-click knob to reset

**No Visual Feedback:**
- No indication of auto vs manual opacity mode
- Planned for V3.8.0: Mode indicator (AUTO/MANUAL badge)

**Master Channel:**
- No shader controls on master (audio-only)
- Future consideration for master shader compositor

---

## DEPRECATIONS

None.

---

## UPGRADE NOTES

**From V3.6.0:**
1. Open V3.7.0 HTML file
2. Load existing .omsp project files (fully compatible)
3. New shader controls available via Shift+ keyboard shortcuts
4. Touch shader opacity knob to enable manual override
5. All V3.6.0 workflows still work identically

**No data migration required** - all existing files work immediately.

---

## COMPARISON WITH PREVIOUS VERSIONS

### V3.7.0 vs V3.6.0

| Feature | V3.6.0 | V3.7.0 |
|---------|--------|--------|
| Shader opacity control | Volume-based only | Manual override + auto |
| Shader M/S buttons | Not available | Independent per track |
| Audio M/S buttons | Not available | Per track + master |
| Track knob size | 60px | 30px (50% smaller) |
| Master knob size | 30px | 60px (2x larger) |
| Interface labels | Some present | All removed |
| Keyboard shortcuts | Audio only | Shader (Shift+) + Audio |
| Visual hierarchy | Flat | Size-based (30px/60px) |
| Master mixer | Basic | M/S + tempo integration |
| Header alignment | Good | Pixel-perfect (43px) |
| Module borders | Basic | Complete + rounded |
| File size | 475KB | 525KB (+10.5%) |

### V3.7.0 vs V3.3.0 (Foundation)

V3.7.0 builds on the "soldiers at the goalpost" beat synchronization foundation established in V3.3.0 (53 iterations). All timing accuracy maintained while adding professional dual-domain mixing capabilities.

---

## CONTRIBUTORS

**Developer:** Wes Jones (ONEMANSHYO)  
**AI Assistant:** Claude (Anthropic)  
**Philosophy:** "Anyone can be a OneManShYo"

---

## LICENSE

Copyright (C) 2024-2025 Wes Jones / ONEMANSHYO  
Licensed under GNU General Public License v3.0

---

## NEXT RELEASE

**V3.8.0 (Planned):**
- Tooltips on mouseover (show current values)
- Visual mode indicator (AUTO vs MANUAL)
- Double-click reset to auto mode
- Joined volume controls (link tracks)

**V3.9.0 (Planned):**
- Per-track shader parameter controls
- Advanced modulation options
- Enhanced audio reactivity

**V4.0.0 (Vision):**
- Complete Ableton Live + EboSuite replacement
- Web MIDI API integration ("Wilson CC" module)
- Native multi-track audio recording
- Real-time audio effects
- Full browser-based performance system

---

**END OF CHANGELOG**

Version: V3.7.0  
Date: November 16, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com
