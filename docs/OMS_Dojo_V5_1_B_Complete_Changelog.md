# ONEMANSHYO App [Beta V5_1_0] - COMPLETE CHANGELOG

**Version:** Beta V5_1_0  
**Release Date:** December 24, 2025  
**Base Version:** V5_0_0  
**Development Iterations:** 34 sub-iterations (V5_0_18 through V5_0_51)

---

## EXECUTIVE SUMMARY

Beta V5_1_0 is a major release introducing BumpR StickRs (video overlay system) and Cue Points (CDJ-style hot cues). The PERFORM tab keyboard workflow was reorganized to prioritize cue jumping (urgent performance action) over track selection (setup action). The Container Output section was clarified with Save/Update button separation.

**Key Achievements:**
1. **BumpR StickRs** - 10-slot video overlay system with alpha channel support
2. **Cue Points** - 3 hot cues per container with visual flags and .oms storage
3. **PERFORM Keyboard** - 1/2/3 for cue jump, Shift+1-5 for track selection
4. **Container Output** - Clear Save vs Update distinction with hover help

---

## MAJOR FEATURES

### 1. BumpR StickRs (V5_0_18-38)

✅ **10-Slot Video Overlay System**
| Feature | Description |
|---------|-------------|
| Slots | 10 total (keys 1-9, 0) |
| Content | WebM VP9 with alpha channel |
| Folder | `/BumpRStickRs/` auto-created |
| Thumbnails | 160×90 data URLs, dark background fill |

✅ **Keyboard Mode (Shift+B)**
| Key | Action |
|-----|--------|
| Shift+B | Enter/exit BumpR mode |
| 1-9, 0 | Trigger slots 1-10 |
| ↑/↓ | Navigate slots |
| Enter | Trigger selected |
| Escape | Exit mode |

✅ **Per-Slot Settings**
| Setting | Options |
|---------|---------|
| Scale | 25%, 50%, 75%, 100% |
| Timing | Instant, Beat, Bar |
| Animate | 16 effects (see below) |
| Duration | Original, Loop, Musical bars, Timed seconds |

✅ **16 Animation Effects**
| Effect | Duration | Loop |
|--------|----------|------|
| None | - | - |
| Bounce | 0.5s | ∞ |
| Breathe | 4s | ∞ |
| Crazy | 0.8s | ∞ |
| Drift | 4s | ∞ |
| Drop | 0.6s | 1× |
| Float | 2s | ∞ |
| Glitch | 0.2s | ∞ |
| Pop | 0.4s | 1× |
| Pulse | 1s | ∞ |
| Shake | 0.3s | ∞ |
| Spin | 1s | ∞ |
| Spiral | 2s | ∞ alt |
| Strobe | 0.1s | ∞ |
| Wobble | 0.5s | ∞ |
| Zoom | 2s | ∞ |

✅ **Quantized Timing**
- Instant: Fires immediately
- Beat: Waits for next beat boundary
- Bar: Waits for next bar boundary
- Tab-aware: PERFORM uses master clock, CREATE uses audio playback

✅ **Duration Control**
- Original: Play video once
- Loop: Repeat until stopped/replaced
- Musical: 1/4, 1/2, 1, 2, 4, 8 bars
- Timed: 1s, 2s, 3s, 5s, 10s

✅ **CDJ Cue Behavior**
- Keys always fire (no toggle)
- Retrigger replaces current playback
- Matches professional DJ equipment

### 2. Cue Points (V5_0_39-45)

✅ **3 Hot Cues Per Container**
| Cue | Default | Beats | Use |
|-----|---------|-------|-----|
| 1 | 1.1 | 0 | Intro |
| 2 | 17.1 | 64 | First drop |
| 3 | 33.1 | 128 | Second drop |

✅ **CREATE Tab Features**
- Visual orange flags on waveform
- Click flag to jump playhead
- Drag flag to reposition (bar snap)
- Stored in .oms container

✅ **PERFORM Tab Features**
- Orange flags on track waveforms
- 1/2/3 keys jump to cue
- Instant jump (no quantization)

✅ **.oms Storage**
```json
{
    "audioCuePoints": [0, 64, 128]
}
```
Legacy containers default to `[0, 64, 128]`.

### 3. PERFORM Keyboard Reorganization (V5_0_43)

**Rationale:** Cue jumping is urgent during performance (needs fast access), track selection is a setup task (less urgent).

✅ **Track Selection Changed**
| Before | After |
|--------|-------|
| 1 = Track 1 | Shift+1 = Track 1 |
| 2 = Track 2 | Shift+2 = Track 2 |
| 3 = Track 3 | Shift+3 = Track 3 |
| 4 = Track 4 | Shift+4 = Track 4 |
| 5 = Master | Shift+5 = Master |

✅ **Cue Jump Added**
| Key | Action |
|-----|--------|
| 1 | Jump to Cue 1 |
| 2 | Jump to Cue 2 |
| 3 | Jump to Cue 3 |

### 4. Container Output Refinements (V5_0_46-49)

✅ **Section Rename**
- "OUTPUT" → "CONTAINER OUTPUT"
- Clarifies dual purpose: video file or library save

✅ **Save vs Update Separation**
| Button | Action | Visibility |
|--------|--------|------------|
| Save | Creates NEW container | Always |
| Update | Updates loaded container | Only when loaded |

✅ **Button Location**
- Moved from SHADER CONTROLS to CONTAINER OUTPUT header
- Groups all output actions together

✅ **CORTEX Hover Messages**
| Element | Message |
|---------|---------|
| Header | "Container = Shader + Audio + Settings" |
| Save | "Save as NEW container to Library" |
| Update | "Update loaded container (keeps name)" |
| Resolution | "Video resolution for render" |
| FPS | "Frame rate for render" |
| Filename | "Name for container and video file" |
| R button | "Render video file" |

✅ **Update Behavior**
- Stays on CREATE tab (no navigation)
- Shows "Updated: [name]" message

---

## ITERATION HISTORY

### V5_0_18-20: BumpR StickRs Foundation

**V5_0_18:** UI foundation in ADMIN → Settings → BumpR StickRs
- Single slot placeholder
- How-to-use info box

**V5_0_19:** File handling
- Drag-drop WebM detection
- /BumpRStickRs/ folder creation
- Thumbnail generation (160×90)
- Project state persistence

**V5_0_20:** Playback
- F1 keyboard trigger (later changed)
- Video element overlay
- Auto-hide on completion

### V5_0_21-28: BumpR Settings & Animations

**V5_0_21:** Keyboard change to Cmd+1-5

**V5_0_22:** Timing options (Immediate/Beat/Bar)

**V5_0_23:** Clean layout (removed drag zone)

**V5_0_24:** Info box reorganization

**V5_0_25:** Timing dropdown (replaced radios)

**V5_0_26:** Scale dropdown (25/50/75/100%)

**V5_0_27:** Animate dropdown (4 effects)

**V5_0_28:** Full animation set (16 effects)

### V5_0_29-33: Expansion & Mode

**V5_0_29:** All 5 slots UI

**V5_0_30:** Per-slot settings + console spam fix

**V5_0_31:** Shift+B mode + 10 slots + slideout

**V5_0_32:** Slideout refinement (canvas-contained)

**V5_0_33:** Keyboard navigation (↑/↓) + isolation

### V5_0_34-38: Polish & Duration

**V5_0_34:** Reactivity dropdown messages

**V5_0_35:** Navigation to ↑/↓

**V5_0_36:** Mode mutual exclusivity + waveform fix

**V5_0_37:** Duration dropdown + loop mode

**V5_0_38:** Unified audio timing sync

### V5_0_39-45: Cue Points

**V5_0_39:** Visual cue flags

**V5_0_40:** Draggable with bar snap

**V5_0_41:** Click to jump playhead

**V5_0_42:** .oms storage

**V5_0_43:** PERFORM integration + keyboard remap

**V5_0_44:** Improved cue UX (not playing)

**V5_0_45:** Save fix + updateContainer fix

### V5_0_46-49: Container Output

**V5_0_46:** Section rename + Save/Update separation

**V5_0_47:** Button move to header

**V5_0_48:** CORTEX hover messages

**V5_0_49:** Update stays on CREATE

### V5_0_50-51: Bug Fixes

**V5_0_50:** Orientation dropdown sync attempt

**V5_0_51:** Orientation dropdown sync attempt 2

---

## NEW GLOBAL STATE

### BumpR StickRs

```javascript
let bumprStickrsFolderHandle = null;    // Folder handle
let bumprStickrsSlots = [null × 10];    // {filename, thumbnail}
let bumprTimingModes = ['instant' × 10];
let bumprScaleModes = [100 × 10];
let bumprAnimateModes = ['none' × 10];
let bumprDurationModes = ['original' × 10];
let bumprModeActive = false;
let bumprSelectedSlot = 0;
let bumprPlaying = false;
let bumprQueuedTimeout = null;
let bumprActiveVideoEl = null;
```

### Cue Points

```javascript
let audioCuePoints = [0, 64, 128];
let draggingCueIndex = -1;
let dragStartX = 0;
let cueDragged = false;
const CUE_HIT_TOLERANCE = 20;
```

---

## NEW FUNCTIONS

### BumpR File Handling
```javascript
handleBumprSlotDrop(slotIndex, event)
generateBumprThumbnail(file)
updateBumprSlotUI(slotIndex)
removeBumprSlot(slotIndex)
```

### BumpR Playback
```javascript
triggerBumprStickr(slotIndex)
playBumprVideo(slotIndex)
stopBumprVideo()
getBumprDelay(slotIndex)
```

### BumpR Mode
```javascript
enterBumprMode()
exitBumprMode()
updateBumprSlideout()
showBumprSlotInfo(slotIndex)
clearBumprSlotInfo()
```

### BumpR Settings
```javascript
toggleBumprDropdown(dropdownEl)
setBumprTiming(slotIndex, mode)
setBumprScale(slotIndex, scale)
setBumprAnimate(slotIndex, animate)
setBumprDuration(slotIndex, duration)
```

### Cue Points
```javascript
getCueAtPosition(x)      // Hit test
updateCuePosition(e)     // Drag handler
```

---

## MODIFIED FUNCTIONS

| Function | Change |
|----------|--------|
| `drawProgress()` | Added cue flag drawing |
| `drawWaveformFromAudioBuffer()` | Added PERFORM cue flags |
| `buildContainerObject()` | Added audioCuePoints |
| `loadContainer()` | Restore audioCuePoints |
| `updateContainer()` | Include audioCuePoints |
| `removeAudio()` | Reset cues to defaults |
| `getCurrentProjectState()` | Added bumpr arrays |
| `applyProjectState()` | Restore bumpr arrays |
| `setupApplicationFolder()` | Create /BumpRStickRs/ |
| `switchTab()` | Exit BumpR mode on switch |
| `switchAdminSection()` | BumpR StickRs panel |

---

## CSS ADDITIONS

### Animation Keyframes (15 total)
```css
@keyframes bumpr-bounce { 0%, 100% { transform: scale(var(--bumpr-scale, 1)); } 50% { transform: scale(calc(var(--bumpr-scale, 1) * 1.15)); } }
@keyframes bumpr-breathe { ... }
@keyframes bumpr-crazy { ... }
@keyframes bumpr-drift { ... }
@keyframes bumpr-drop { ... }
@keyframes bumpr-float { ... }
@keyframes bumpr-glitch { ... }
@keyframes bumpr-pop { ... }
@keyframes bumpr-pulse { ... }
@keyframes bumpr-shake { ... }
@keyframes bumpr-spin { ... }
@keyframes bumpr-spiral { ... }
@keyframes bumpr-strobe { ... }
@keyframes bumpr-wobble { ... }
@keyframes bumpr-zoom { ... }
```

### BumpR Dropdown System
```css
.bumpr-dropdown { ... }
.bumpr-dropdown.scale { width: 50px; }
.bumpr-dropdown.timing { width: 55px; }
.bumpr-dropdown.animate { width: 65px; }
.bumpr-dropdown.duration { width: 60px; }
.bumpr-dropdown-label { ... }
.bumpr-dropdown-arrow { ... }
.bumpr-dropdown-options { ... }
.bumpr-dropdown-item { ... }
.bumpr-dropdown-sep { ... }
```

### Slideout Panel
```css
.bumpr-slideout {
    position: absolute;
    top: 0; bottom: 0; right: -85px;
    width: 80px;
    background: rgba(20, 20, 20, 0.95);
    border-left: 1px solid #333;
    z-index: 50;
    transition: right 0.2s ease;
}
.bumpr-slideout.active { right: 0; }
.bumpr-slot-row { ... }
.bumpr-slot-row.selected .bumpr-slot-thumb { border-color: #00d4ff; }
.bumpr-slot-row.empty { opacity: 0.3; }
```

---

## DOM ELEMENTS ADDED

### Video Overlays
- `<video id="bumprOverlayVideo">` in preview-container (z-index: 100)
- `<video id="bumprOverlayVideoLive">` in liveShaderOutputContainer (z-index: 100)

### Slideout Panel
- `.bumpr-slideout` inside preview-container
- 10 slot rows with thumbnails

### ADMIN Panel
- "BumpR StickRs" navigation item
- `settings-bumprstickrs` section with 10 slot UIs

---

## PROJECT STATE ADDITIONS

### onemanshyo.omsp
```javascript
bumprStickrsSlots: [...],
bumprTimingModes: [...],
bumprScaleModes: [...],
bumprAnimateModes: [...],
bumprDurationModes: [...]
```

### .oms Containers
```javascript
audioCuePoints: [0, 64, 128]
```

---

## FOLDER STRUCTURE CHANGE

```
[Application Folder]/
├── Library/
├── Presets/
├── BumpRStickRs/     ← NEW
│   └── *.webm files
└── onemanshyo.omsp
```

---

## BUG FIXES

| Issue | Version | Fix |
|-------|---------|-----|
| Delete key in ORGANIZE input | V5_0_46 | Check input field first |
| currentLoadedContainerId not set | V5_0_45 | Set after initial save |
| updateContainer missing audioCuePoints | V5_0_45 | Added field |
| Shift+1 key detection | V5_0_43 | e.key → e.code |
| Console spam (BroadcastChannel) | V5_0_30 | Only log first |
| Waveform white background | V5_0_36 | Explicit dark fill |
| Mode mutual exclusivity | V5_0_36 | Exit other on enter |
| Thumbnail alpha channel | V5_0_33 | Dark background fill |
| Orientation dropdown sync | V5_0_50-51 | Multiple attempts (not resolved) |

---

## ADMIN SETTINGS ORDER

Alphabetized: Application → BumpR StickRs → Create → Organize → Perform

---

## STATISTICS

- **Total Sub-iterations:** 34
- **Lines Added:** ~1,500
- **New Functions:** ~20
- **New CSS Keyframes:** 15
- **New State Variables:** ~15
- **BumpR Slots:** 10
- **Animation Effects:** 16
- **Cue Points Per Container:** 3

---

## COMPATIBILITY

✅ Backward compatible with V5_0_0, V4_8_0, V4_7_0
✅ Existing shaders work unchanged
✅ Existing .oms containers load (cues default)
✅ Existing .omsp projects load (BumpR arrays padded)
✅ Uniform struct unchanged (112 bytes)

---

## KNOWN ISSUES

1. **Orientation dropdown sync** - Visual dropdown may not match loaded orientation value

---

## PRODUCTION READINESS: ~99%

**100% Complete:**
- BumpR StickRs (all features)
- Cue Points (CREATE + PERFORM)
- Container Output refinements
- PERFORM keyboard workflow

**Inherited (100%):**
- Transform controls
- Reactivity controls
- Design System
- OmsDropdown

**Remaining:**
- MIDI integration
- Orientation dropdown sync

---

**END OF COMPLETE CHANGELOG**

Version: Beta V5_1_0
Date: December 24, 2025
Total Development: 34 sub-iterations
License: GPL-3.0
