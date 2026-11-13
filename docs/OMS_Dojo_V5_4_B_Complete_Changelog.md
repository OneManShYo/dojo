# ONEMANSHYO App [Beta V5_4_0] - COMPLETE CHANGELOG

**Version:** Beta V5_4_0  
**Release Date:** December 31, 2025  
**Base Version:** V5_3_0  
**Development Iterations:** 40 iterations + ~90 sub-iterations (V5_3_01 through V5_3_40)

---

## V5_4_0 EXECUTIVE SUMMARY

Beta V5_4_0 is a major release delivering **professional DJ workflow infrastructure**. The Audio Card system enables separate Master/Cue outputs for headphone cueing (Pioneer DJM-style Pre-Fader Listen). The MAIN column provides dedicated output controls with true stereo metering. DSP and VFX slideout panels add master bus compression and GPU-accelerated post-processing. Grid cells are redesigned with half-height aspect ratio and split shader/waveform layout.

**Key Achievements:**
1. **Audio Card & Cue Bus** - Pioneer-style PFL with separate headphone output
2. **MAIN Column** - Dedicated output bus controls (Master → MAIN rename, 64 occurrences)
3. **DSP Processing** - Master bus compressor/limiter with 5 presets
4. **VFX Pipeline** - WebGPU post-processing with 10 effects
5. **Meter System** - Toggle overlay with true stereo L/R metering
6. **Half-Height Grid Cells** - Split layout showing 2× more scenes
7. **Scene Labels & Colors** - Custom labels with ticker animation
8. **ESC Safety** - Double-tap required for Stop All

---

## V5_4_0 MAJOR FEATURES

### 1. Audio Card & Cue Bus System (V5_3_01-02, 12-16, 19)

✅ **Audio Card Settings Panel**
| Feature | Implementation |
|---------|----------------|
| Device enumeration | `navigator.mediaDevices.enumerateDevices()` |
| Master Output | Dropdown for FOH/speakers |
| Cue Output | Dropdown for headphones |
| Test tones | 440Hz (Master), 880Hz (Cue) |
| Diagnostics | Copy Report to clipboard |

✅ **Cue Bus Architecture (PFL)**
```
Track → EQ ──┬──→ Gain → Analyser → Master Gain → speakers
             │
             └──→ Cue Gain [0/1] ─┐   (PRE-FADER tap)
                                  ↓
             All Cue Gains → Cue Master ──┐
                                          │ (crossfade)
                                          ↓
Master Analyser ──→ Main-to-Cue Gain ─────┴─→ Cue Output → headphones
```

✅ **Pioneer DJM-Style Crossfade**
| Mix Position | Main in Headphones | Cue in Headphones |
|--------------|-------------------|-------------------|
| Left (0%) | 100% | 0% |
| Center (50%) | 50% | 50% |
| Right (100%) | 0% | 100% |

✅ **S→C Button Logic**
| Condition | Solo Button | Color |
|-----------|-------------|-------|
| No cue output | S (Solo) | Cyan |
| Cue output configured | C (Cue) | Orange |

✅ **Browser Requirements**
- Chrome/Edge 110+ for `setSinkId()` support
- Multi-channel audio interface recommended (Behringer UMC204HD/404HD)

### 2. MAIN Column Architecture (V5_3_24-33)

✅ **Master → MAIN Rename**
| System | Naming |
|--------|--------|
| Master Clock (timing) | Stays `master*` (`masterTempo`, `masterBarNow`) |
| Master Grid (overlay) | Stays `master*` (`masterGridVisible`) |
| Master Channel (output) | Renamed to `trackMain*` (64 occurrences) |

✅ **Column Layout (110px)**
```
┌─────────────────────────┐
│ MAIN (header)           │
├─────────────────────────┤
│ main-top-module (flex:1)│
│  ├─ Meters (V + A)      │
│  │   52px per section   │
│  └─ Control boxes (5ea) │
├─────────────────────────┤
│ main-bottom-module      │
│ (248px fixed)           │
│  ├─ Clock (1.1.1)       │
│  ├─ Quantize dropdown   │
│  ├─ Zoom dropdown       │
│  ├─ Tempo + knob        │
│  └─ M/T/W buttons       │
└─────────────────────────┘
```

✅ **Stereo Metering**
| Implementation | Detail |
|----------------|--------|
| Splitter | `ChannelSplitterNode` (2 channels) |
| Left analyser | `trackMainAnalyserL` |
| Right analyser | `trackMainAnalyserR` |
| Display | True L/R meter bars in MAIN column |

### 3. DSP Output Processing (V5_3_34)

✅ **Slideout Panel**
| Aspect | Value |
|--------|-------|
| Location | Emerges from behind MAIN column |
| Width | 200px |
| Toggle | DSP button in MAIN column |
| Z-index | 5 (MAIN = 10, emerges from behind) |

✅ **Controls**
| Control | Range | Default |
|---------|-------|---------|
| Preset | Off/Gentle/Medium/Aggressive/Limiter | Medium |
| Threshold | -48 to 0 dB | -18 dB |
| Ratio | 1:1 to 20:1 | 4:1 |
| Output | -12 to +12 dB | 0 dB |
| GR Meter | Real-time gain reduction | - |

✅ **Presets**
| Preset | Threshold | Ratio | Output |
|--------|-----------|-------|--------|
| Off | 0 dB | 1:1 | 0 dB |
| Gentle | -24 dB | 2:1 | +2 dB |
| Medium | -18 dB | 4:1 | +4 dB |
| Aggressive | -12 dB | 8:1 | +6 dB |
| Limiter | -6 dB | 20:1 | +3 dB |

✅ **Audio Chain**
```
Track outputs → Master Gain → DSP Compressor → DSP Output Gain → destination
```

### 4. VFX Post-Processing Pipeline (V5_3_35-36)

✅ **WebGPU Pipeline**
```
Track 1-4 Shaders → CSS stacking → Composite Canvas → WebGPU Post-FX → VFX Output
```

✅ **Available Effects (10)**
| Effect | Description |
|--------|-------------|
| Off | No processing |
| RGB Split | Chromatic aberration |
| Glitch | Block displacement + corruption |
| Pixelate | Lower resolution sampling |
| Scanlines | CRT effect + RGB offset |
| Feedback | Blend with previous frame |
| Mirror | Horizontal reflection |
| Kaleidoscope | Radial segment reflections |
| VHS | Tracking errors, color bleed |
| Shake | Camera shake effect |

✅ **UX Features**
| Feature | Behavior |
|---------|----------|
| Hover ring | Purple (#a855f7) on amount knob |
| Double-click | Reset amount to 0% |
| "Off" selection | Auto-reset amount to 0% (live-safe) |
| Dropdown | OmsDropdown component (not native select) |

### 5. Meter System Overhaul (V5_3_20-31)

✅ **Toggle Overlay**
| Aspect | Value |
|--------|-------|
| Trigger | Split-cube button (purple/cyan gradient) |
| Location | mixerScenes area |
| Size | 795px × 120px |
| Columns | 5 (Tracks 1-4 + Scenes placeholder) |

✅ **Per-Track Module Structure**
| Side | Content |
|------|---------|
| Visual (left) | Purple meter bar, opacity readout |
| Audio (right) | Stereo L/R bars, peak readout, volume readout |

✅ **Color Thresholds (Audio)**
| Zone | dB Range | Color |
|------|----------|-------|
| Safe | -48 to -18 dB | Green |
| Warm | -18 to -6 dB | Yellow |
| Hot | -6 to 0 dB | Red |

✅ **MAIN Meters (Always Visible)**
| Section | Content |
|---------|---------|
| Visual | Single purple meter + opacity readout |
| Audio | Stereo L/R meters + peak readout (dB) |

### 6. Grid Cell Visual Overhaul (V5_3_40)

✅ **Half-Height Cells**
| Before | After |
|--------|-------|
| `aspect-ratio: 16/9` | `aspect-ratio: 32/9` |
| Standard height | Half height (2× more scenes visible) |

✅ **Split Layout Structure**
```html
<div class="clip-slot assigned">
    <div class="cell-split-container">
        <div class="cell-visual-half">
            <canvas><!-- 80×45 shader thumbnail --></canvas>
        </div>
        <div class="cell-audio-half">
            <canvas><!-- waveform bars --></canvas>
        </div>
    </div>
    <div class="cell-name-overlay">Container Name</div>
</div>
```

✅ **Cell States**
| State | Left Half | Right Half |
|-------|-----------|------------|
| Empty | "Empty" centered | |
| No visual, no audio | 30% purple | 30% cyan |
| Has visual, no audio | Shader thumbnail | 30% cyan |
| No visual, has audio | 30% purple | Waveform bars |
| Both | Shader thumbnail | Waveform bars |

✅ **Layered State Indicators**
| Layer | Pseudo-Element | Z-Index | State |
|-------|----------------|---------|-------|
| Content | .cell-split-container | default | Shader/waveform |
| Playing | ::before | 5 | Yellow tint + outline |
| Cursor | ::after | 10 | Cyan outline (always on top) |

### 7. Scene Labels & Colors (V5_3_37, 39)

✅ **Customization**
| Feature | Limit |
|---------|-------|
| Label length | 128 characters |
| Colors | 8 presets |
| Animation | 6-second ticker scroll |

✅ **Color Presets**
| Name | Hex |
|------|-----|
| Default | null (grey) |
| Cyan | #00d4ff |
| Magenta | #ff006e |
| Orange | #ffa500 |
| Yellow | #ffff00 |
| Green | #00ff88 |
| Purple | #a855f7 |
| Red | #ff4444 |

✅ **PERFORM Tab Interaction**
| Action | Result |
|--------|--------|
| Click+hold 500ms | Inline label edit |
| Right-click | Color picker popup |

### 8. ADMIN Tab Restructure (V5_3_38)

✅ **Flat Navigation (13 items)**
| Item | Description |
|------|-------------|
| About | App info |
| Application | General settings |
| Application Labels | Scene/track labels |
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

✅ **Keyboard Navigation**
| Key | Action |
|-----|--------|
| ↑ | Previous section (wraps) |
| ↓ | Next section (wraps) |

### 9. Safety & UX (V5_3_37, 04-10, 18)

✅ **ESC Double-Tap Safety**
| Action | Result |
|--------|--------|
| Single ESC | Magenta warning "Hit ESC 2× quickly to Stop All" |
| Double ESC (within 500ms) | Executes Stop All |

✅ **Stop All Button**
| Aspect | Value |
|--------|-------|
| Size | 16×16px mini button |
| Symbol | ■ (black square) |
| Flash | Red on activation |

---

## V5_4_0 ITERATION DETAILS

### V5_3_01-03: Audio Card Foundation
- Audio Card ADMIN section with device dropdowns
- Device enumeration via Web Audio API
- Test tone buttons (440Hz/880Hz)
- Copy Report diagnostic button

### V5_3_04-10: Mixer Compaction
- 20×20 control sizes (down from 24×24)
- Removed clip duration display (meaningless without waveforms)
- Removed BPM display (redundant with master tempo)
- Gap-as-divider CSS pattern

### V5_3_12-19: Cue Bus Implementation
- Pre-fader tap for PFL routing
- Pioneer-style crossfade (not Ableton additive)
- S→C button logic (Solo becomes Cue when configured)
- Cue mix knob in ADMIN

### V5_3_20-23: Meter Overlay
- 120px slideout panel above mixer
- Per-track V/A modules
- Split-cube toggle button
- True stereo L/R metering

### V5_3_24-33: MAIN Column Architecture
- 110px dedicated column
- Master→MAIN rename (64 occurrences)
- Stereo metering via ChannelSplitterNode
- Two-module layout (top flex, bottom 248px fixed)

### V5_3_34-36: DSP & VFX
- DSP slideout with compressor/limiter
- 5 presets (Off/Gentle/Medium/Aggressive/Limiter)
- VFX slideout with 10 GPU effects
- Live-safe "Off" auto-reset

### V5_3_37-40: Labels & Grid
- Scene labels (128 char, ticker animation)
- 8 color presets
- Half-height grid cells (32/9 aspect ratio)
- Split shader/waveform layout
- Layered state indicators (::before/::after)
- ESC double-tap safety

---

## V5_4_0 NEW FUNCTIONS

### Audio Card
```javascript
initAudioCard()
enumerateAudioDevices()
setAudioCardMain(deviceId)
setAudioCardCue(deviceId)
applyMasterOutputDevice()
applyCueOutputDevice()
isCueModeEnabled()
updateCueMixGains()
updateTrackCueRouting(track, isCued)
testAudioCardMaster()
testAudioCardCue()
copyAudioCardReport()
```

### Meters
```javascript
toggleMeters()
updateOpacityMeter(track, opacity)
animateMeters()
drawMeterBar(ctx, value, width, height, color)
```

### DSP
```javascript
toggleDSPPanel()
initDSPProcessing()
setDSPPreset(name)
updateDSPThreshold(value)
updateDSPRatio(value)
updateDSPOutput(value)
startDSPReductionMeter()
```

### VFX
```javascript
toggleVFXPanel()
initVFXPipeline()
buildVFXPipeline(effect)
compositeTracksToTexture()
renderVFXPass()
setVFXEffect(effect)
setVFXAmount(value)
applyVFXEffect()
```

### Scene Labels
```javascript
showSceneColorPicker(index, x, y)
startSceneLabelEdit(index, triggerElement)
updateSceneTriggerDisplay(index)
refreshAllSceneTriggers()
resetSceneLabels()
resetSceneColors()
```

### MAIN Column
```javascript
updateTrackMainShaderOpacity()
initTrackMainStereoMeters()
```

---

## V5_4_0 BUG FIXES

| Build | Issue | Fix |
|-------|-------|-----|
| V5_3_04 | Removing HTML without JS cleanup | Grep for references before removing |
| V5_3_17 | Grid visibility not working on load | Move declarations before usage |
| V5_3_17 | Track grid required audioSnapLastBar | Calculate from duration as fallback |
| V5_3_19 | Project state not loading from other tabs | Call autoLoadDefaultProject after initLiveTab |
| V5_3_19 | Audio devices no auto-scan | Scan when no devices cached |
| V5_3_31 | Master shader opacity not applied | Apply in both render loops |
| V5_3_38d | Message bar invisible since it35 | Restore 5 missing </div> tags |
| V5_3_40 | Ghost cells getting .playing | Move cleanup outside trackAudioSources check |
| V5_3_40 | Yellow covering cyan selection | Layered pseudo-elements (::before/::after) |
| V5_3_40 | Library yellow highlight useless | Removed isActive/activeStyle logic |

---

## V5_4_0 BREAKING CHANGES

### Master → MAIN Rename
- **Before:** `masterShaderOpacity`, `masterVolume`, `masterGainNode`
- **After:** `trackMainShaderOpacity`, `trackMainVolume`, `trackMainGainNode`
- **Impact:** 64 occurrences renamed
- **Migration:** Internal only, no user impact

### Project State Schema Additions
```javascript
{
    audioCardSettings: { masterOutputId, cueOutputId, cueMix },
    trackCues: [false, false, false, false],
    sceneLabels: ['1'-'8'],
    sceneColors: [null × 8],
    audioPrefs: { masterTempo: 120 },
    dspSettings: { enabled, preset, threshold, ratio, output },
    vfxSettings: { effect, amount }
}
```

### Browser Requirements
- Chrome/Edge 110+ required for Audio Card (setSinkId)
- WebGPU required for VFX pipeline

---

## V5_3_0 EXECUTIVE SUMMARY

Beta V5_3_0 is a major release delivering **real audio reactivity** - the foundational promise of the application. Previous FFT bin averaging produced constant values (~0.02); shaders appeared reactive only due to tempo-synced oscillators. This release replaces that with bandpass-filtered peak detection that captures true audio dynamics (kick hits now register 0.6-0.8). The Mod Tuning panel provides professional ADSR envelope shaping, and the unified sets system simplifies library organization.

**Key Achievements:**
1. **Real Audio Reactivity** - Bandpass peak detection replaces fake FFT averaging (CORTEX change)
2. **Mod Tuning Panel** - Blender-style N-panel with per-mod Q and global ADSR envelope
3. **Unified Sets System** - Manual and smart sets in single cache with type field
4. **Container Tags** - Chip-based tag UI with admin panel for bulk management
5. **Proper Peak Metering** - True dBFS metering (48dB range) in PERFORM tab
6. **Quantize Control** - Forward-only quantization (Off/1 Beat/1 Bar)
7. **Draggable Bar 1** - Set beatgrid anchor by ear on waveform

---

## MAJOR FEATURES

### 1. Real Audio Reactivity (V5_2_31-41) - CORTEX

✅ **The Problem (V5_2_0 and earlier)**
| Behavior | Result |
|----------|--------|
| FFT bin averaging | Constant ~0.02 values |
| No dynamics | Shaders appeared reactive only from oscillators |
| Audio irrelevant | Could disconnect audio, same visual output |

✅ **The Solution: Bandpass Peak Detection**
| Feature | Implementation |
|---------|----------------|
| Signal tap | Pre-EQ source node |
| Filter chain | Highpass → Lowpass per modulator |
| Detection | `getFloatTimeDomainData()` for true peaks |
| Updates | Real-time when adjusting Freq1/Freq2 knobs |

✅ **Signal Chain Architecture**
```
Audio Source (pre-EQ tap)
    ├──→ Existing chain → speakers (unchanged)
    │
    ├──→ modAHighpass → modALowpass → modAAnalyser → Peak
    ├──→ modBHighpass → modBLowpass → modBAnalyser → Peak
    └──→ modCHighpass → modCLowpass → modCAnalyser → Peak
```

✅ **Results Comparison**
| Scenario | FFT Averaging (Before) | Bandpass Peak (After) |
|----------|------------------------|----------------------|
| Kick hit | 0.02 | 0.72 |
| Silence | 0.02 | 0.01 |
| Hi-hat | 0.02 | 0.15 |
| Full mix | 0.02 | Dynamic range |

✅ **Offline Baking**
- Timeline analysis uses same bandpass approach
- `.oms` files contain real dynamic data
- PERFORM playback matches CREATE preview exactly

### 2. Mod Tuning Panel (V5_2_43-51)

✅ **Access Pattern**
| Step | Action |
|------|--------|
| 1 | Enter Reactivity Focus (Shift+R) |
| 2 | Press N to open panel |
| 3 | Panel slides from left edge |
| 4 | Escape or N to close |

✅ **Per-Modulator Q Knobs**
| Control | Range | Default | Effect |
|---------|-------|---------|--------|
| Mod A Q | 0.5-10.0 | 0.7 | Filter sharpness |
| Mod B Q | 0.5-10.0 | 0.7 | Filter sharpness |
| Mod C Q | 0.5-10.0 | 0.7 | Filter sharpness |

✅ **Global ADSR Envelope**
| Parameter | Min | Max | Default | Unit |
|-----------|-----|-----|---------|------|
| Attack | 0 | 100 | 5 | ms |
| Decay | 0 | 200 | 20 | ms |
| Sustain | 0 | 1.0 | 0.7 | ratio |
| Release | 0 | 500 | 50 | ms |
| Threshold | 0 | 0.20 | 0.01 | - |

✅ **Envelope Behavior**
| State | Phase | Action |
|-------|-------|--------|
| Signal ≥ threshold | Gate ON | Attack toward peak, decay toward sustain |
| Signal < threshold | Gate OFF | Release toward zero |

✅ **.oms Storage**
```json
{
  "modTuning": {
    "modAQ": 0.7, "modBQ": 0.7, "modCQ": 0.7,
    "modAttack": 5, "modDecay": 20, "modSustain": 0.7,
    "modRelease": 50, "modThreshold": 0.01
  }
}
```

### 3. Unified Sets System (V5_2_52-57)

✅ **Architecture Change**
| Before | After |
|--------|-------|
| `userSetsCache` (manual sets) | `userSetsCache` (ALL sets) |
| `smartSetsCache` (smart sets) | Removed |
| Smart sets in project state | All sets in `/Sets/` folder |

✅ **Set Object Structure**
```javascript
// Manual set - drag containers to add
{
  id: "set_1703712000000",
  name: "My Favorites",
  type: "manual",
  containers: ["container1", "container2"],
  created: "2025-12-27T..."
}

// Smart set - saved search query
{
  id: "smart_1703712000000",
  name: "Bass 128+",
  type: "smart",
  query: "bass",
  created: "2025-12-27T..."
}
```

✅ **Visual Indicators**
| Indicator | Meaning |
|-----------|---------|
| Cyan dot | Smart set (saved search) |
| No dot | Manual set |

✅ **Panel Navigation**
| Action | Result |
|--------|--------|
| Click "Sets" header | Focus sets panel |
| Click "Containers" header | Focus containers panel |
| ↑/↓ | Navigate within focused panel |
| Double-click name | Inline rename |
| Escape | Exit focus mode |

### 4. Container Tags System (V5_2_53-56)

✅ **Schema Addition**
```json
{
  "tags": ["bass", "heavy", "128bpm"]
}
```

✅ **Chip UI Features**
| Interaction | Result |
|-------------|--------|
| Type + Enter/comma | Creates chip |
| × button | Removes chip |
| Backspace on empty | Removes last chip |
| Duplicate entry | Prevented (case-insensitive) |

✅ **Search Integration**
- Tags included in search matching
- Query matches name, shader code, or tags
- Smart sets can filter by tags

✅ **Admin Panel (ADMIN → Containers → Tags)**
| Feature | Description |
|---------|-------------|
| Tag list | All unique tags with counts |
| Inline rename | Click name to edit |
| Propagation | Rename affects all containers |
| Delete | Remove tag from all containers |

### 5. Proper Peak Metering (V5_2_26-28)

✅ **Implementation Change**
| Before | After |
|--------|-------|
| `getByteFrequencyData()` | `getFloatTimeDomainData()` |
| FFT spectral bins | Waveform samples |
| Meaningless average | True peak amplitude |

✅ **dB Conversion**
```javascript
const dB = 20 * Math.log10(peak);
const percent = ((dB - (-48)) / 48) * 100;
```

✅ **Color Thresholds**
| Zone | dB Range | Meter % |
|------|----------|---------|
| Green | -48 to -18 dB | 0-62.5% |
| Yellow | -18 to -6 dB | 62.5-87.5% |
| Red | -6 to 0 dB | 87.5-100% |

### 6. Quantize Control (V5_2_08-12)

✅ **Replaced RMS Field**
| Before | After |
|--------|-------|
| RMS input (unused) | Quantize dropdown |

✅ **Options**
| Label | Value | Behavior |
|-------|-------|----------|
| Off | 0 | Immediate launch |
| 1 Beat | 1 | Snap to next beat |
| 1 Bar | 4 | Snap to next bar (default) |

✅ **Forward-Only Fix**
```javascript
// Before (could snap backward)
const nextBar = Math.round(beat / 4) * 4;

// After (always forward)
const nextBar = Math.ceil(beat / 4) * 4;
```

✅ **Tempo History**
- Tracks last 8 tempo samples
- Handles tempo changes mid-quantize

### 7. Draggable Bar 1 Marker (V5_2_13-16)

✅ **Visual**
| Element | Description |
|---------|-------------|
| White line | Vertical marker at Bar 1 position |
| "1" label | Number indicator above line |
| Grey waveform | Before Bar 1 |
| Blue waveform | After Bar 1 |

✅ **Interaction**
| Action | Result |
|--------|--------|
| Click + drag | Move Bar 1 position |
| Release | Updates audioStartTime |
| Snap | 100ms grid alignment |

---

## ITERATION HISTORY

### V5_2_01-07: PERFORM Tab Foundation
| Build | Focus |
|-------|-------|
| V5_2_01 | Grid initialization fix |
| V5_2_02 | Mixer state sync |
| V5_2_03-04 | Cue/scene refinements |
| V5_2_05 | Scene master orange styling |
| V5_2_06-07 | Master control fixes |

### V5_2_08-12: Quantize System
| Build | Focus |
|-------|-------|
| V5_2_08-10 | Quantize dropdown |
| V5_2_11-12 | Tempo history tracking |

### V5_2_13-22: CREATE Tab Audio/Waveform
| Build | Focus |
|-------|-------|
| V5_2_13-15 | Draggable Bar 1 marker |
| V5_2_16 | Grey/blue waveform |
| V5_2_17-18 | Cue click plays |
| V5_2_19 | Volume dB scale |
| V5_2_20 | FX simplification |
| V5_2_21-22 | Negative bar fix |

### V5_2_23-30: ORGANIZE Tab & Metering
| Build | Focus |
|-------|-------|
| V5_2_23-25 | Info drawer structure |
| V5_2_26-28 | Proper peak metering |
| V5_2_29 | Bracket keys for tempo |
| V5_2_30 | Grid auto-scroll |

### V5_2_31-42: Real Audio Reactivity (MAJOR)
| Build | Focus |
|-------|-------|
| V5_2_31 | Problem identification |
| V5_2_32-33 | Bandpass architecture |
| V5_2_34-35 | Filter implementation |
| V5_2_36-37 | Peak detection |
| V5_2_38-39 | Live filter updates |
| V5_2_40-41 | Offline baking |
| V5_2_42 | Frequency signal LEDs |

### V5_2_43-51: Mod Tuning Panel
| Build | Focus |
|-------|-------|
| V5_2_43 | Panel architecture |
| V5_2_44-45 | Per-mod Q knobs |
| V5_2_46-47 | ADSR envelope |
| V5_2_48 | Threshold gate |
| V5_2_49 | Envelope processing |
| V5_2_50 | Live meters |
| V5_2_51 | .oms storage |

### V5_2_52-57: Sets & Tags
| Build | Focus |
|-------|-------|
| V5_2_52 | Smart sets initial |
| V5_2_53 | Tags schema |
| V5_2_54 | Chip UI |
| V5_2_55 | File-based sets |
| V5_2_56-57 | Unified architecture |

---

## NEW STATE VARIABLES

### Audio Reactivity (~15 variables)
```javascript
// Bandpass filter nodes (per modulator)
let modAHighpass, modALowpass, modAAnalyser, modADataArray;
let modBHighpass, modBLowpass, modBAnalyser, modBDataArray;
let modCHighpass, modCLowpass, modCAnalyser, modCDataArray;

// Envelope follower state
let envFollowerA, envFollowerB, envFollowerC;
let envPeakA, envPeakB, envPeakC;
let lastEnvTime;
```

### Mod Tuning (~10 variables)
```javascript
let modAQ = 0.7, modBQ = 0.7, modCQ = 0.7;
let modAttack = 5, modDecay = 20, modSustain = 0.7;
let modRelease = 50, modThreshold = 0.01;
let modTuningPanelActive = false;
```

### Sets & Tags (~5 variables)
```javascript
let setsFolderHandle = null;
let libraryFocusPanel = null;  // 'sets' | 'containers' | null
// userSetsCache now unified (smartSetsCache removed)
```

### PERFORM Tab (~5 variables)
```javascript
let quantizeValue = 4;  // 0, 1, or 4
let tempoHistory = [];  // Last 8 tempo samples
```

---

## NEW FUNCTIONS (~30 total)

### Audio Reactivity
```javascript
updateBandpassFilters()
getPeakFromAnalyser(analyser, dataArray)
applyEnvelopeFollower(rawA, rawB, rawC)
processEnvelope(input, current, peak, dt)
updateFilterQ(mod, value)
```

### Mod Tuning Panel
```javascript
toggleModTuningPanel()
openModTuningPanel()
closeModTuningPanel()
updateModMeters(modA, modB, modC)
drawModKnob(canvas, value, min, max, color)
initModTuningKnobs()
getModTuningState()
applyModTuningState(state)
```

### Tags
```javascript
addTagChip(text)
removeTagChip(btn)
getTagsFromChips()
populateTagChips(tags)
startRenameTag(oldTag)
finishRenameTag(oldTag, newTag)
deleteTag(tag)
```

### Sets
```javascript
saveCurrentSearch()
deleteSet(setId)
containerMatchesSearch(container, query)
migrateSmartSets()
```

### Volume/Metering
```javascript
volumePositionToGain(position)
volumePositionToDbString(position)
getPeakLevel(analyser)
peakToMeterPercent(peak)
peakToDb(peak)
```

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| `analyzeAudio()` | Bandpass peak detection |
| `analyzeAudioTimeline()` | Same bandpass approach |
| `buildContainerObject()` | Added tags, modTuning |
| `loadContainer()` | Restore tags, modTuning |
| `updateFreqKnobs()` | Calls updateBandpassFilters() |
| `refreshLibraryList()` | Unified sets loading |
| `initLiveTab()` | Grid init + mixer sync |
| `beatsToBarBeat()` | Clamp to minimum bar 1 |

---

## CSS ADDITIONS

### Mod Tuning Panel
```css
.mod-tuning-panel {
    position: absolute;
    left: 0;
    width: 180px;
    transform: translateX(-100%);
    transition: transform 0.2s;
}
.mod-tuning-panel.active { transform: translateX(0); }
.mod-tuning-row { ... }
.mod-q-knob { width: 24px; height: 24px; }
.mod-env-control { ... }
```

### Tags
```css
.tags-input-container { ... }
.tags-chips { display: flex; flex-wrap: wrap; gap: 4px; }
.tag-chip {
    background: rgba(0, 212, 255, 0.2);
    border: 1px solid #00d4ff;
    border-radius: 12px;
    padding: 2px 8px;
}
.tag-remove { ... }
```

### Smart Set Indicator
```css
.smart-set-dot {
    width: 6px;
    height: 6px;
    background: #00d4ff;
    border-radius: 50%;
}
```

### Draggable Bar 1
```css
.bar1-marker { cursor: ew-resize; }
.bar1-label { ... }
```

---

## DOM ELEMENTS ADDED

### Mod Tuning Panel
- `.mod-tuning-panel` slide-out
- Per-mod rows with meters, values, Q knobs
- ADSR envelope row
- Threshold control

### Tags (Info Drawer)
- `.tags-input-container`
- `.tags-chips` container
- `.tag-chip` elements
- `.tags-input` field

### Admin Tags Panel
- Tag list with counts
- Inline rename inputs
- Delete buttons

### Quantize (PERFORM)
- Dropdown replacing RMS input
- Options: Off, 1 Beat, 1 Bar

---

## BUG FIXES (12 total)

| Build | Issue | Fix |
|-------|-------|-----|
| V5_2_01 | Grid not showing on PERFORM first visit | Added initLiveTab() in folder setup |
| V5_2_02 | Mixer state not syncing | Added sync block at end of initLiveTab() |
| V5_2_03 | Cue highlights bleeding across tracks | Clear ALL tracks before highlighting |
| V5_2_04 | Rename race condition | Added dataset.saving guard |
| V5_2_08-10 | Quantize snapping backward | Math.round → Math.ceil |
| V5_2_14 | Shader swap clearing container ID | Check audio loaded before clearing |
| V5_2_21 | Negative bar display | Clamped bar to minimum 1 |
| V5_2_28 | Quantize visual sync | Clamp elapsedBeats to 0 |
| V5_2_33 | Scene double-click not working | Added dblclick handler |
| V5_2_06-07 | Master controls unresponsive | Fixed pointer-events: none |

---

## BREAKING CHANGES

### Audio Reactivity (CORTEX)
- **Before:** FFT bin averaging (fake dynamics)
- **After:** Bandpass peak detection (real dynamics)
- **Impact:** Timeline arrays now contain actual dynamic values
- **Migration:** Existing containers work but may need re-analysis for optimal results

### Sets System
- **Before:** Separate caches for manual/smart sets
- **After:** Unified `userSetsCache` with type field
- **Impact:** Smart sets in old project states migrated to files
- **Migration:** Automatic on first load

### Variable Renames
| Old | New |
|-----|-----|
| `performActivePanel` | `liveActivePanel` |
| `performCueSelectedRow` | `liveCueSelectedRow` |
| `performEQSelectedBand` | `liveEQSelectedBand` |
| `performFXSelectedSlot` | `liveFXSelectedSlot` |
| `npanel*` | `mod*` |
| `reactivityNPanelActive` | `modTuningPanelActive` |

### New Schema Fields
- `tags: []` - Array of strings
- `modTuning: {}` - Envelope settings object

---

## FOLDER STRUCTURE

```
[App Folder]/
├── Library/
│   └── *.oms
├── Sets/              ← All sets (manual + smart)
│   └── *.oms-set
├── Presets/
│   └── *.oms
├── BumpRStickRs/
│   └── *.webm
├── Exports/
│   └── *.oms.zip
└── onemanshyo.omsp
```

---

## STATISTICS

| Metric | Value |
|--------|-------|
| Total Iterations | 57 |
| Lines Added | ~4,000 |
| New Functions | ~30 |
| New State Variables | ~35 |
| Bug Fixes | 12 |
| Schema Additions | 2 (tags, modTuning) |

---

## COMPATIBILITY

✅ Backward compatible with V5_2_0, V5_1_0, V5_0_0
✅ Existing shaders work unchanged
✅ Existing .oms containers load (new fields use defaults)
✅ Legacy smartSets in project state auto-migrated
✅ Uniform struct unchanged (112 bytes)

---

## KNOWN ISSUES

1. **Orientation dropdown sync** - Visual dropdown may not match loaded orientation value (inherited from V5_1_0)

---

## PRODUCTION READINESS: ~99%

**100% Complete:**
- Real Audio Reactivity
- Mod Tuning Panel (ADSR)
- Unified Sets System
- Container Tags
- Proper Peak Metering
- Quantize Control
- Draggable Bar 1

**Inherited from V5_2_0 (100%):**
- Audio Focus Mode
- EQ3 System
- DJ FX System
- 5 Cue Points
- Per-Track Audio Processing
- Centralized Focus Mode

**Remaining:**
- MIDI integration
- Orientation dropdown sync

---

**END OF COMPLETE CHANGELOG**

Version: Beta V5_3_0
Date: December 27, 2025
Total Development: 57 sub-iterations
License: GPL-3.0
