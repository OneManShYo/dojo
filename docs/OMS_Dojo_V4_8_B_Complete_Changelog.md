# ONEMANSHYO BETA V4_8_0 - COMPLETE CHANGELOG

**Version:** Beta V4_8_0  
**Release Date:** December 19, 2025  
**Base Version:** V4_7_0  
**Development Iterations:** 31 sub-iterations (V4_8_00 through V4_8_31)

---

## EXECUTIVE SUMMARY

V4_8_0 is a major modulator system overhaul that introduces visual knob controls, independent shader uniforms, and keyboard-based reactivity mode. The CREATE tab now features 9 rotary knobs for Gain/Freq1/Freq2 across all three modulators, a reusable OmsDropdown component with shader thumbnails, and comprehensive keyboard controls designed for live performance workflows.

**Key Achievement:** Independent modulator uniforms give shader authors granular access to Wave/Gain/FFT components, enabling sophisticated audio-reactive effects not possible with combined values alone.

---

## MAJOR FEATURES

### 1. Modulator Knob Controls (V4_8_09-11)

✅ **9 Visual Knobs**
- 3 Gain knobs (0-1 range, linear)
- 6 Freq knobs (20-24000 Hz, log scale)
- 30×30 pixel canvas-based controls
- Purple color matching PERFORM tab

✅ **Knob Interactions**
- Drag up/down to adjust
- Hover shows value in message bar
- Push behavior: Freq1 can't exceed Freq2
- Freq knobs disabled when no audio loaded

### 2. Independent Shader Uniforms (V4_8_26-27)

✅ **Uniform Struct Expansion** (48 → 80 bytes)
```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,              // Legacy combined
    modB: f32,              
    modC: f32,              
    beat: f32,              // NEW: Tempo-synced
    resolutionX: f32,
    resolutionY: f32,
    shaderModAWave: f32,    // NEW: Waveform (0-1)
    shaderModAGain: f32,    // NEW: Gain (0-1)
    shaderModAFFT: f32,     // NEW: FFT level (0-1)
    shaderModBWave: f32,    
    shaderModBGain: f32,    
    shaderModBFFT: f32,     
    shaderModCWave: f32,    
    shaderModCGain: f32,    
    shaderModCFFT: f32,     
    padding1-3: f32         // 20 floats = 80 bytes
}
```

✅ **Backward Compatibility**
- Legacy modA/B/C still work
- Old containers load with default values for new uniforms

### 3. Reactivity Keyboard Mode (V4_8_12-16)

✅ **Mode-Based Control**
| Key | Action |
|-----|--------|
| Shift+R | Enter/exit reactivity mode |
| Escape | Exit reactivity mode |
| ←/→ | Navigate columns |
| ↑/↓ | Navigate rows |
| Shift+← | Decrease value |
| Shift+→ | Increase value |

✅ **Visual Feedback**
- Cyan selection ring on active control
- Message bar shows current selection
- Value changes display instantly

✅ **Progressive Acceleration** (V4_8_31)
- Hold arrow key = faster value changes
- Gain: 0.01 → ~0.11 step (11× max)
- Freq: 1.1 → 2.1 ratio (log scale)

### 4. OmsDropdown Component (V4_8_18-19)

✅ **Reusable Custom Dropdown**
- Shader thumbnail support (32×18)
- Keyboard navigation (arrows, Enter, Escape)
- Click outside to close
- onOpen callback for lazy refresh
- Separator support for grouping

✅ **API**
```javascript
const dropdown = new OmsDropdown('containerId', {
    showThumbnails: true,
    thumbnailSize: { w: 32, h: 18 },
    onChange: (value, item) => { },
    onOpen: async () => { }
});
```

### 5. WGSL Preset Support (V4_8_17)

✅ **Raw Shader Files**
- /Presets/ folder scans both .oms and .wgsl
- Drop shader files from Claude chats
- Quick load without drag/drop

### 6. ID3 BPM Detection (V4_8_05)

✅ **Automatic Tempo Extraction**
- Parses ID3v2 TBPM tag from MP3s
- Supports v2.2, v2.3, v2.4 formats
- Falls back to filename parsing

---

## ITERATION HISTORY

### V4_8_00-01: Column Layout

**V4_8_00:** Failed attempt at column rebalance
- Shrunk dropdowns but didn't expand inputs enough

**V4_8_01:** Reactivity column rebalance
- Fixed flex values: Subdiv 1.0, Waveform 1.0, Gain 0.8, Freq1 1.3, Freq2 1.3
- Priority: input fields fully visible, dropdowns can truncate

### V4_8_02-04: Waveform Options

**V4_8_02:** None option for modulators
- Added "None" as default in Subdiv/Waveform dropdowns
- calculateMod() returns 0 when either is "none"

**V4_8_03:** Clickable message bar
- showMessage() accepts onClick callback
- "Set application folder" now clickable

**V4_8_04:** Static waveform option
- Returns 1.0 (gain controls directly)
- None = off (0), Static = no oscillation

### V4_8_05-08: Audio & Save Improvements

**V4_8_05:** ID3v2 TBPM tag parsing
- New parseBPMFromID3(file) function
- Priority: ID3 → filename → default 120

**V4_8_06:** Save updates existing container
- Uses currentLoadedContainerId as source of truth
- null = creating, "xyz" = editing

**V4_8_07:** Filename preservation
- Keeps container name when editing
- Only auto-generates for new containers

**V4_8_08:** Message color fix
- "Set application folder" changed to orange (state)

### V4_8_09-11: Knob Controls

**V4_8_09:** Freq knobs
- 6 canvas knobs (30×30) replacing input fields
- Log scale: 20 Hz → 24,000 Hz
- Color-coded by modulator row

**V4_8_10:** Gain knobs
- 3 canvas knobs replacing input fields
- Linear scale: 0.0 → 5.0 (later 0-1)

**V4_8_11:** Knob visual unification
- All knobs purple (#9d4edd)
- Fixed 36px width columns

### V4_8_12-16: Keyboard Controls

**V4_8_12:** Modulator keyboard controls
- Shift+1-5 select column
- Shift+Up/Down cycle rows
- Shift+Left/Right adjust values

**V4_8_13:** Shift+Arrow conflict fix
- Added `&& !e.shiftKey` to playhead handler

**V4_8_14:** Trackpad scrub & simplified mod keys
- Horizontal swipe = audio scrub
- CREATE tab: mod keys work without Shift

**V4_8_15:** Shift+R reactivity mode
- Toggle mode with Shift+R
- Arrows navigate, trackpad adjusts

**V4_8_16:** Shift+Arrow value adjust
- Replaced trackpad with Shift+Left/Right
- All-keyboard control

### V4_8_17-19: Dropdown & Presets

**V4_8_17:** WGSL files in Presets
- Scans /Presets/ for both .oms and .wgsl

**V4_8_18:** OmsDropdown component
- Custom dropdown with thumbnails
- Replaced native select for shader name

**V4_8_19:** OmsDropdown refresh fix
- onOpen callback properly calls refresh
- /Presets/ folder auto-created

### V4_8_20-24: Grid Navigation

**V4_8_20:** Shift+trackpad snap to beat
- Quantized beat jumping

**V4_8_21:** Grid snap beats
- gridSnapBeats = 16 (4 bars)
- Unified visual and snap spacing

**V4_8_22:** Shift+<> grid jump keys
- Shift+< previous, Shift+> next grid line

**V4_8_23-24:** Backward grid jump fixes
- Fixed stuck on same line
- Simplified: backward = floor - 1

### V4_8_25-27: Shader Uniforms

**V4_8_25:** Disable tempo when audio loaded
- Shader tempo grayed out
- Audio tempo = THE tempo

**V4_8_26:** Add uniforms.beat
- Tempo-synced beat position
- Follows audio when playing

**V4_8_27:** Independent modulator uniforms
- 9 new uniforms (Wave/Gain/FFT × 3)
- Struct expanded to 80 bytes

### V4_8_28-31: Knob Refinements

**V4_8_28:** Freq knobs disabled when no audio
- Visual dimming, interaction blocked

**V4_8_29:** Gain range 0-1
- Changed from 0-5
- Old containers normalized

**V4_8_30:** Gain keyboard fix
- Step 0.01 (100 levels)
- Range matched to 0-1

**V4_8_31:** Gain default & progressive keyboard
- Default 0.10 (not max)
- Hold key = accelerating changes
- Fixed zero wrap bug

---

## NEW FUNCTIONS

### parseBPMFromID3(file)
**Purpose:** Extract BPM from ID3v2 TBPM tag  
**Returns:** Integer BPM or null

### initGainKnobs()
**Purpose:** Initialize 3 gain knob canvas controls

### initFreqKnobs()
**Purpose:** Initialize 6 freq knob canvas controls

### setFreqKnobsDisabled(disabled)
**Purpose:** Enable/disable freq knobs based on audio state

### OmsDropdown class
**Purpose:** Reusable custom dropdown with thumbnails

### adjustModValue(delta)
**Purpose:** Keyboard value adjustment for modulators

### updateModSelection()
**Purpose:** Apply/remove visual selection ring

### exitReactivityMode()
**Purpose:** Exit keyboard reactivity mode

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| showMessage() | Added onClick callback parameter |
| calculateMod() | Handles 'none' and 'static' waveforms |
| loadAudioFile() | ID3 parsing, tempo field disable |
| removeAudio() | Re-enable tempo field |
| Uniform buffer writes | Expanded to 80 bytes, 9 new values |

---

## CSS CHANGES

### Knob Styles
```css
.mod-knob-container {
    flex: 0 0 36px;
    display: flex;
    justify-content: center;
}
```

### Selection Ring
```css
.mod-selected {
    outline: 2px solid #00d4ff;
    outline-offset: 2px;
}
```

### OmsDropdown
```css
.oms-dropdown-list {
    position: absolute;
    max-height: 300px;
    overflow-y: auto;
    background: #1a1a1a;
    border: 1px solid #444;
}
```

---

## BUG FIXES

| Issue | Cause | Fix |
|-------|-------|-----|
| Freq column truncation | Flex values too small | Rebalanced to 1.3 |
| Shift+Arrow playhead conflict | Handler not checking shiftKey | Added `&& !e.shiftKey` |
| Backward grid jump stuck | Wrong floor/ceil logic | Simplified to floor-1 |
| .wgsl files not in dropdown | onOpen not calling refresh | Fixed callback chain |
| Gain keyboard range | Using old 0-5 range | Updated to 0-1, step 0.01 |
| Zero wrap to max | `\|\| 0.1` on falsy 0 | Changed to isNaN() check |
| Filename overwrite | Auto-gen when editing | Check currentLoadedContainerId |

---

## STATISTICS

- **Total Sub-iterations:** 31
- **Lines Changed:** ~1200+
- **Functions Added:** 8
- **New Uniforms:** 10 (beat + 9 modulator components)
- **New UI Controls:** 9 knobs + 1 custom dropdown
- **Keyboard Shortcuts Added:** 8 (reactivity mode)

---

## PRODUCTION READINESS

**Overall: ~97%**

**100% Complete:**
- Modulator knob controls
- Independent shader uniforms
- Reactivity keyboard mode
- OmsDropdown component
- WGSL preset support
- ID3 BPM detection

**Inherited from V4_7_0 (100%):**
- Message bar system
- PERFORM tab controls
- Color-coded feedback
- User-created sets

**Remaining:**
- MIDI integration
- PERFORM EQ controls
- Knobs visual container (deferred)

---

**END OF COMPLETE CHANGELOG**

Version: Beta V4_8_0
Date: December 19, 2025
Total Development: 31 sub-iterations
License: GPL-3.0
