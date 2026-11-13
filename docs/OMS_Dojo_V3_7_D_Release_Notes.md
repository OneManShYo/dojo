# OMS SHADER BAKER V3.7.0 - RELEASE NOTES

**Release Date:** November 16, 2025  
**Version:** V3.7.0 Production  
**Type:** Major UI/UX Release

---

## WHAT'S NEW

### 🎛️ Dual-Domain Mixer (GAME CHANGER!)

**Mix visuals and audio independently on each track!**

Each of the 4 tracks now has TWO separate control sections:

**SHADER DOMAIN** (Purple controls):
- Opacity knob (0-100% manual override)
- Mute button (purple when active) 
- Solo button (cyan when active)
- Controls visuals ONLY

**AUDIO DOMAIN** (Standard controls):
- Volume fader (audio output level)
- Level meters (L/R VU meters)
- Mute button (standard when active)
- Solo button (standard when active)
- Controls audio ONLY

**How to use:**
1. Load different shaders on tracks 1-4
2. Launch clips
3. Fade audio with volume faders
4. Fade visuals independently with opacity knobs
5. Create visual-only builds without changing audio mix

**New keyboard shortcuts:**
- `Shift+,` - Toggle shader mute
- `Shift+.` - Toggle shader solo
- `Shift+↑` - Increase shader opacity (+2%)
- `Shift+↓` - Decrease shader opacity (-2%)

**Default behavior:**
- Shader opacity follows audio volume (V3.6.0 style)
- Touch opacity knob = switch to manual override
- Manual mode persists until project reload

---

### 💎 Professional UI Polish

**50% smaller track knobs, 2x larger master controls!**

**Visual hierarchy through size:**
- Track knobs: 30px × 30px (detail work)
- Master/zoom knobs: 60px × 60px (global control)
- Clear visual language without needing labels

**All labels removed:**
- Interface itself is the documentation
- Position + color + muscle memory = clear function
- Cleaner, more professional appearance
- Hardware-inspired modular aesthetic

**Before V3.7.0:**
```
[Track 1]
SHADER          AUDIO
Volume: 75%     -∞ to 100%
[L] [R] meters
MANUAL override
```

**After V3.7.0:**
```
[Track 1]
────┬────
 🎚️  │ 🎚️
[M][S]│[▮▮][▮▮]
      │[M][S]
```

Clean. Minimal. Professional.

---

### 🎚️ Master Mixer Enhancements

**Master channel now has parity with tracks!**

New features:
- M/S buttons on master (mute/solo global output)
- Tempo control integrated with master section
- Symmetrical layout (matches track structure)
- 3x taller meters (120px vs 40px for tracks)

Master section positioning:
```
┌────────────┐
│  [Volume]  │ ← 60px knob
│            │
│  [Meters]  │ ← 120px tall
│   [M] [S]  │ ← M/S buttons
│            │
│ [− 120 +]  │ ← Tempo control
└────────────┘
```

---

### 📐 Perfect Header Alignment

**9 iterations to nail pixel-perfect alignment!**

All grid headers now exactly 43px tall:
- "Scenes" header
- "Track 1/2/3/4" headers
- No gaps, no misalignment
- Professional polish

Visual consistency everywhere:
- Module borders: 2px (primary)
- Internal guidelines: 0.5px (secondary, 50% reduction)
- Rounded corners: 3px on all modules
- Hidden scrollbar in scenes column

---

### 🏗️ Modular Hardware Aesthetic

**Three distinct modules with complete framing:**

```
┌────────────────────┐
│  GRID SECTION      │ ← Scenes + Clips
└────────────────────┘
      (10px gap)
┌────────────────────┐
│  MIXER SECTION     │ ← Knobs + Faders
└────────────────────┘
      (10px gap)
┌────────────────────┐
│  WAVEFORM SECTION  │ ← Timeline
└────────────────────┘
```

Each module:
- Complete 2px border
- 3px rounded corners
- 10px spacing between modules
- Feels like hardware rack unit

---

### 🔄 Symmetrical Track Layout

**Shader and audio controls mirror each other!**

Hardware-inspired parallel control:
- Left column: Shader (M/S, opacity knob)
- Right column: Audio (volume knob, meters, M/S)
- Vertical symmetry creates tactile feel
- Immediately intuitive

**Design quote:** "Symmetry is the language of hardware"

---

## IMPROVEMENTS

✅ **Reduced Secondary Guidelines** - 50% thinner (1px → 0.5px)  
✅ **Rounded Module Corners** - 3px radius for softer appearance  
✅ **Compact Meters** - 40px (tracks) vs 120px (master)  
✅ **Hidden Scrollbar** - Scenes column still scrolls, just invisible  
✅ **Grey Header Bar** - Matches track header styling  
✅ **Marquee Scroll Fix** - Smooth looping for long container names

---

## BUG FIXES

🛠️ **Marquee Jerk** - Fixed visible "jerk" when looping long names  
🛠️ **Double Gap** - Eliminated excessive whitespace in mixer section  
🛠️ **Header Alignment** - 9 iterations to achieve perfect 43px headers

---

## PERFORMANCE

**Tested on M1 MacBook Pro 14":**
- 4 shaders rendering: ~30-45% GPU (same as V3.6.0)
- 60fps stable with all tracks playing
- Dual-domain mixer adds 0% GPU overhead (UI only)
- File size: 525KB (+50KB from V3.6.0)

**No performance impact** - all improvements are UI/UX only!

---

## COMPATIBILITY

✅ **Fully backward compatible** with V3.6.0 project files  
✅ **No migration required** - existing .omsp files work immediately  
✅ **All V3.6.0 features preserved** - four-track compositing, pop-out window, etc.  
✅ **All keyboard controls unchanged** - Shift+ modifiers added, existing shortcuts still work

---

## SYSTEM REQUIREMENTS

**Minimum:**
- Chrome 113+ or Edge 113+ (WebGPU support)
- 1920×1080 screen resolution
- Integrated graphics (Intel/AMD)

**Recommended:**
- Safari Technology Preview or Chrome latest
- 1512×982 or higher (MacBook Pro 14" optimal)
- Dedicated graphics (NVIDIA/AMD) or Apple Silicon
- Dual monitor setup for pop-out window

---

## QUICK START

**For New Users:**
1. Open HTML file in Chrome/Edge/Safari TP
2. Settings tab → Set Application Folder
3. Create shader in Controls or Code tab
4. Save as .oms container
5. LIVE tab → arrange in grid
6. Pop Out → send to projector
7. Use Shift+ shortcuts for shader control
8. Perform!

**For Existing Users:**
1. Open V3.7.0 HTML file
2. Load existing .omsp projects (work instantly)
3. Touch shader opacity knob to enable manual override
4. Use Shift+ shortcuts for independent visual control
5. Enjoy the cleaner interface!

---

## KEYBOARD SHORTCUTS

### NEW IN V3.7.0

**Shader Controls (Shift+ Modifiers):**
- Shift+, (comma): Toggle shader mute
- Shift+. (period): Toggle shader solo
- Shift+↑: Increase shader opacity (+2%)
- Shift+↓: Decrease shader opacity (-2%)

### EXISTING SHORTCUTS (Still Work!)

**Transport:**
- Space: Global play/pause
- Esc: Emergency stop + return to scenes

**Track Selection:**
- 1-4: Select tracks 1-4
- 5: Select master track
- Tab: Next track
- Shift+Tab: Previous track

**Navigation:**
- W/S: Navigate scenes/cells (vertical)
- A/D: Navigate tracks/cells (horizontal)
- Enter: Launch scene or cell

**Audio Controls:**
- ↑/↓: Adjust audio volume on selected track
- , (comma): Toggle audio mute
- . (period): Toggle audio solo

**Cell Management:**
- Delete/Backspace: Clear selected cell

---

## DUAL-DOMAIN WORKFLOW EXAMPLES

### Example 1: Visual Build Without Audio Change

**Scenario:** Build visuals during breakdown without touching audio mix

**Steps:**
1. Audio stays at current levels (don't touch volume faders)
2. Use shader opacity knobs to fade in visuals
3. `Shift+↑` on tracks 1, 2, 3 to gradually add layers
4. Audio mix unchanged, visuals build smoothly

### Example 2: Audio Drop With Visual Hold

**Scenario:** Drop audio out but keep visuals present

**Steps:**
1. Use shader manual override on all tracks (touch opacity knobs)
2. Set shader opacity to 100% on all tracks
3. Drop audio volume faders to 0%
4. Visuals stay visible, audio silent

### Example 3: Visual Solo for Specific Element

**Scenario:** Show only one shader while playing all audio

**Steps:**
1. `Shift+.` on track 1 to solo shader
2. All other shaders invisible (tracks 2-4)
3. All audio still playing (normal mix)
4. Perfect for highlighting specific visual element

---

## KNOWN LIMITATIONS

**No Reset to Auto:**
- Manual shader opacity can't be reset without project reload
- Planned for V3.8.0: Double-click knob to reset

**No Visual Feedback:**
- No indication of auto vs manual opacity mode
- Planned for V3.8.0: Mode indicator badge (AUTO/MANUAL)

**Master Channel:**
- No shader controls on master (audio-only)
- Future consideration for master shader compositor

---

## UPGRADE PATH

**From V3.6.0 → V3.7.0:**
1. Open V3.7.0 HTML file
2. Load existing .omsp projects (fully compatible)
3. Shader opacity starts in auto mode (volume-based)
4. Touch opacity knob to enable manual override
5. Use Shift+ shortcuts for shader control

No other changes required!

**From V3.5.0 or earlier:**
Projects load but may benefit from re-arrangement for dual-domain layout.

---

## WHAT'S NEXT

**V3.8.0 (Coming Soon):**
- Tooltips on mouseover (show current values + shortcuts)
- Visual mode indicator (AUTO vs MANUAL badge)
- Double-click reset to auto mode
- Joined volume controls (link multiple tracks)

**V3.9.0 (Planned):**
- Per-track shader parameter controls
- Advanced modulation routing
- Enhanced audio reactivity options

**V4.0.0 (Vision):**
- Complete browser-based performance system
- Web MIDI API integration ("Wilson CC" module)
- Replace Ableton Live + EboSuite entirely
- Native multi-track audio recording
- Real-time audio effects

---

## SUPPORT

**YouTube:** ONEMANSHYO channel (100k monthly views)  
**Website:** https://onemanshyo.com  
**License:** GPL-3.0 (free forever)

**Ghost Protocol:** Tools provided free with no ongoing support commitment.

---

## CREDITS

**Developer:** Wes Jones (ONEMANSHYO)  
**AI Assistant:** Claude (Anthropic)  
**Philosophy:** "Anyone can be a OneManShYo"

Special thanks to the community for testing and feedback!

---

**Quotes from V3.7.0 Development:**

> "Two MIDI Fighters or one Yaeltex - same mental model!" 🎛️  
> "The interface itself is the documentation." 💎  
> "Symmetry is the language of hardware." 🔄  
> "Compact is professional." 📐

---

**Enjoy V3.7.0! Now go create something amazing with dual-domain mixing! 🎨🎵✨**

---

END OF RELEASE NOTES

Version: V3.7.0  
Date: November 16, 2025  
License: GPL-3.0
