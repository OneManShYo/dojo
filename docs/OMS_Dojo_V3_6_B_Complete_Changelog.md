# OMS SHADER BAKER V3.6.0 - COMPLETE CHANGELOG

**Version:** V3.6.0 Production Release  
**Date:** November 16, 2025  
**Base Version:** V3.5.0  
**Development Iterations:** 31 (V3_6_0_B through V3_6_0_AF)

---

## OVERVIEW

V3.6.0 is a **major feature release** that transforms OMS Shader Baker from a single-shader preview tool into a complete four-track visual performance instrument. This release introduces real-time shader compositing, pop-out window support, and professional mixing capabilities that mirror traditional DJ software.

**Development Approach:** 31 surgical iterations over 1 day, ~1,900 lines modified

---

## NEW FEATURES

### 1. Four-Track Shader Compositing

**Description:** Independent WebGPU rendering pipelines for each of 4 tracks with volume-based opacity mixing.

**Implementation:**
- 4 separate WebGPU devices, pipelines, and contexts
- Z-index layering (Track 1 = top, Track 4 = bottom)
- Volume faders control both audio AND visual opacity
- Intensity = volume × 2 (peaks at 50% fader position)
- 60fps performance with all 4 shaders rendering simultaneously

**User Benefit:** Natural DJ mixing workflow - adjust one fader to mix both audio and visuals

**Iterations:** V3_6_0_B, V3_6_0_I, V3_6_0_J, V3_6_0_K

---

### 2. Pop-Out Window for Second Monitor

**Description:** Separate resizable window for audience/projector display

**Features:**
- Two size presets: 640×360 (default) or 1280×720
- Always-on-top option (Settings tab)
- Dual rendering (inset for DJ reference, pop-out for audience)
- Synchronized at 60fps
- Window remembers size/position across sessions

**User Benefit:** DJ sees control interface, audience sees full-screen visuals on projector

**Iterations:** V3_6_0_D, V3_6_0_F, V3_6_0_M, V3_6_0_N

---

### 3. Waveform Overlay System

**Description:** Visual connection between performance grid and audio waveforms

**Features:**
- Container name + shader thumbnail overlaid on each waveform
- Only visible when track is playing
- Positioned at 75% down waveform (lower third, non-intrusive)
- Marquee scrolling for long container names
- Shader thumbnail 90×51px
- Fixed 150px width name box with auto-scroll

**User Benefit:** Instant visual feedback showing which shader is playing on which track

**Iterations:** V3_6_0_W, V3_6_0_X, V3_6_0_Z, V3_6_0_AA, V3_6_0_AB, V3_6_0_AF

---

### 4. Professional Visual Indicators

**Description:** Crystal-clear cursor position and playback state

**Visual System:**
- **Cyan outline (1px):** Selected cell cursor position
- **Yellow border (2px):** Currently playing cell
- **Both visible:** When selected cell is playing
- CSS stacking prevents conflicts (outline > border > content)

**User Benefit:** Never lose track of cursor position or which clips are playing

**Iterations:** V3_6_0_P, V3_6_0_Q, V3_6_0_R, V3_6_0_S, V3_6_0_T, V3_6_0_U

---

### 5. Settings Persistence

**Description:** Application preferences saved in project files

**Settings:**
- Pop-out window size (640×360 or 1280×720)
- Always-on-top toggle
- Saved in .omsp project files
- Auto-loads when opening projects

**User Benefit:** Set preferences once, they persist across sessions

**Iterations:** V3_6_0_N

---

### 6. DELETE Key Cell Clearing

**Description:** Quick cell cleanup with keyboard

**Features:**
- DELETE or Backspace clears selected cell
- Removes audio, waveform, shader (both windows)
- Cross-platform (Mac Backspace = Windows Delete)
- Complete cleanup in one keypress

**User Benefit:** Rapid workflow changes during performance

**Iterations:** V3_6_0_O

---

### 7. Empty Cell Clip Stop

**Description:** Ableton-style clip stop workflow

**Behavior:**
- Launching empty cell stops that track
- Launching empty scene stops all 4 tracks
- Visual feedback (removes yellow playing border)
- Mouse and keyboard both supported

**User Benefit:** Industry-standard DJ workflow

**Iterations:** V3_6_0_M, V3_6_0_V

---

## IMPROVEMENTS

### Navigation & Control

**Scene Column Default (V3_6_0_L):**
- Cursor starts in SCENE column (not Track 1)
- W/S navigate scenes, Enter launches scene
- D enters tracks, A/ESC returns to scenes
- Matches professional DJ software paradigm

**Z-Index Priority (V3_6_0_K):**
- Track 1 (leftmost) renders on top
- Track 4 (rightmost) renders on bottom
- Left-to-right priority matches mixer layout

### Visual Refinements

**Clean Default Shader (V3_6_0_C):**
- Black screen instead of purple gradient
- Professional appearance before launching clips

**Thinner Indicators (V3_6_0_U):**
- Reduced 50% per user feedback
- Cyan: 2px → 1px
- Yellow: 3px → 2px
- More subtle, professional appearance

### Bug Fixes

**Scope Issues (V3_6_0_E, V3_6_0_G, V3_6_0_H):**
- Moved functions to global scope for accessibility
- Removed duplicate function definitions
- Fixed container loading failures

**Waveform Playhead Alignment (V3_6_0_AE):**
- Fixed offsetLeft calculation for position:relative containers
- Waveforms now align perfectly at Bar 1.1.1
- Critical fix for accurate visual sync

**Visual State Accuracy (V3_6_0_V):**
- Playing class removed when tracks stop
- Yellow borders disappear correctly
- Visual state matches audio state perfectly

**Canvas Dimensions (V3_6_0_X):**
- Added width/height attributes to overlay canvases
- Fixed rendering issues
- Internal buffer size now matches display size

---

## TECHNICAL CHANGES

### Architecture

**Multi-Shader Arrays:**
```javascript
// Before (V3.5.0): Single shader
let liveShaderDevice = null;
let liveShaderPipeline = null;

// After (V3.6.0): 4 independent shaders
let liveShaderDevices = [null, null, null, null];
let liveShaderPipelines = [null, null, null, null];
```

**Dual Rendering:**
- Inset canvases (4) for DJ reference
- Pop-out canvases (4) for audience display
- Single render loop updates all 8 canvases
- ~5-10% GPU overhead for pop-out

### Performance

**GPU Usage:**
- 4 shaders rendering: ~30-45% (M1 MacBook Pro)
- 60fps stable with all tracks playing
- Pop-out adds ~5-10%
- Recommended: Close other GPU apps during performance

**File Size:**
- V3.5.0: 445KB
- V3.6.0: 475KB (+30KB, +6.7%)
- ~1,900 lines added/modified across 31 iterations

### Settings Schema

**New appSettings Object:**
```javascript
{
    popOutWindowSize: '640x360' | '1280x720',
    popOutAlwaysOnTop: true | false
}
```

Saved in .omsp files under `settings` key

---

## COMPLETE ITERATION LIST

### Phase 1: Multi-Shader Compositing (B-C)
- **V3_6_0_B:** Four independent WebGPU systems (~290 lines)
- **V3_6_0_C:** Black default shader (~2 lines)

### Phase 2: Pop-Out Window (D-H)
- **V3_6_0_D:** Initial pop-out implementation (~334 lines)
- **V3_6_0_E:** Scope fix (recalculateZoomRange) (~100 lines)
- **V3_6_0_F:** Clean pop-out from working base (~327 lines)
- **V3_6_0_G:** Scope fix 2 (drawZoomKnob) (~101 lines)
- **V3_6_0_H:** Remove duplicate functions (~-101 lines)

### Phase 3: Volume & Navigation (I-M)
- **V3_6_0_I:** Binary playing/stopped opacity (~10 lines)
- **V3_6_0_J:** Volume intensity curve (×2) (~15 lines)
- **V3_6_0_K:** Z-index order fix (~5 lines)
- **V3_6_0_L:** Scene column navigation default (~30 lines)
- **V3_6_0_M:** Empty cell stop + pop-out improvements (~40 lines)

### Phase 4: Settings & Cleanup (N-O)
- **V3_6_0_N:** Settings save/load system (~60 lines)
- **V3_6_0_O:** DELETE key cell clearing (~25 lines)

### Phase 5: Visual Indicators (P-U)
- **V3_6_0_P:** Clean indicators (cyan cursor, green playing) (~50 lines)
- **V3_6_0_Q:** Outline cursor fix (~30 lines)
- **V3_6_0_R:** Yellow playing dominant (~40 lines)
- **V3_6_0_S:** Property separation (outline vs box-shadow) (~35 lines)
- **V3_6_0_T:** Perfect CSS stacking order (~25 lines)
- **V3_6_0_U:** Thinner indicators (50% reduction) (~15 lines)

### Phase 6: Bug Fixes (V)
- **V3_6_0_V:** Playing class removal fix (~10 lines)

### Phase 7: Waveform Overlays (W-Z, AA-AB, AE-AF)
- **V3_6_0_W:** Initial overlay implementation (~150 lines)
- **V3_6_0_X:** Canvas dimension fix (~8 lines)
- **V3_6_0_Z:** Overlay position refinement (~20 lines)
- **V3_6_0_AA:** Name styling (transparent, wide) (~30 lines)
- **V3_6_0_AB:** Size & position (~25 lines)
- **V3_6_0_AE:** Critical playhead alignment fix (~20 lines)
- **V3_6_0_AF:** Shader-first + marquee scrolling (~60 lines)

**Total:** 31 iterations, ~1,900 lines modified

---

## BREAKING CHANGES

None. V3.6.0 is fully backward compatible with V3.5.0 project files.

**Migration Notes:**
- Existing .omsp files load without modification
- Settings object added but defaults used if missing
- Old single-shader preview replaced with 4-track compositing
- All keyboard controls remain unchanged

---

## KNOWN ISSUES

None reported at time of release.

**Performance Notes:**
- Lower pop-out resolution (640×360) if experiencing frame drops
- Close other GPU-intensive applications during performance
- Disable browser extensions that modify web pages
- Use hardware acceleration in browser settings

---

## DEPRECATIONS

None.

---

## UPGRADE NOTES

**From V3.5.0:**
1. Open V3.6.0 HTML file
2. Load existing .omsp project files (fully compatible)
3. Containers with shaders will render on all 4 tracks
4. Click "Pop Out ↗" for second monitor/projector display
5. Set pop-out window size in Settings tab
6. Enable "Always on Top" if desired

**No data migration required** - all existing files work immediately.

---

## COMPARISON WITH PREVIOUS VERSIONS

### V3.6.0 vs V3.5.0

| Feature | V3.5.0 | V3.6.0 |
|---------|--------|--------|
| Shader rendering | Single shader preview | 4-track compositing |
| Second monitor | Not supported | Pop-out window |
| Visual mixing | Not available | Volume-based opacity |
| Waveform overlays | Not available | Container + thumbnail |
| Settings persistence | Not available | Window size + always-on-top |
| DELETE key | Not supported | Clear cells |
| Visual indicators | Box-shadow only | Outline + border (no conflicts) |
| File size | 445KB | 475KB (+6.7%) |

### V3.6.0 vs V3.3.0 (Foundation)

V3.6.0 builds on the "soldiers at the goalpost" beat synchronization foundation established in V3.3.0 (53 iterations). All timing accuracy maintained while adding visual performance capabilities.

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

**V3.7.0 (Planned):**
- Advanced shader effects and modulation
- Shader parameter controls in LIVE tab
- Enhanced audio reactivity options
- Performance optimizations

**V3.8.0 (Planned):**
- MIDI controller integration
- Hardware mixer compatibility
- Custom control mappings

**V4.0.0 (Vision):**
- Complete Ableton Live + EboSuite replacement
- Native multi-track audio recording
- Real-time audio effects
- Full browser-based performance system

---

**END OF CHANGELOG**

Version: V3.6.0  
Date: November 16, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com
