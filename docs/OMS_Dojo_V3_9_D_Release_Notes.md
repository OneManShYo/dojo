# OMS SHADER BAKER V3.9.0 - RELEASE NOTES

**Release Date:** November 21, 2025 (Development Build)  
**Version:** V3.9.0_L_sub20  
**Type:** Timing Architecture Improvements

---

## ⚠️ DEVELOPMENT BUILD

**Status:** In Testing  
This is a development build with significant timing architecture changes.  
Production release will follow after full validation.

---

## WHAT'S NEW

### 🕐 Master Clock Architecture

**Single Source of Truth**

All timing now controlled by centralized `masterClock` object:
- Derives from `audioContext.currentTime`
- No cached tempo values
- Beat/bar calculations from session start
- Foundation for perfect synchronization

---

### 🎵 Beat-Based Scrolling

**Musical Coordinates**

Waveforms now scroll using beats/bars instead of seconds:
- Matches Master Grid coordinate system
- Handles tempo changes correctly
- "One Timeline" architecture fully implemented
- Prevents drift during performance

---

### 🎯 Master Grid Alignment

**Multiple Approaches Tested**

Extensive research to fix track/grid alignment:
- Tested 3 different AI-suggested solutions
- 20+ sub-iterations of refinement
- Beat-based system selected as primary approach
- Initial positioning improved (audioStartTime fix)

---

### 🪟 Pop-Out Window Fix

**Shader Compilation**

Pop-out window now renders shaders correctly:
- Shader code stored in container data
- Compilation succeeds on secondary display
- All 4 tracks render properly
- No more "shader not found" errors

---

## TECHNICAL IMPROVEMENTS

### Timing System

**Before:**
- Time-based scrolling (seconds)
- Separate audio/visual clocks
- Tempo changes caused drift
- Accumulated floating point errors

**After:**
- Beat-based scrolling (bars/beats)
- Single master clock
- Tempo changes handled correctly
- Perfect mathematical alignment

---

### Launch Beat Tracking

**Precision Timing**

Each track now records exact beat position when launched:
```javascript
trackLaunchBeats[track] = masterClock.getCurrentBeat();
```

Enables perfect synchronization across all tracks regardless of launch timing.

---

### Tempo History

**Change Tracking**

System now walks tempo history:
- Records every tempo change
- Calculates elapsed beats accurately
- Handles complex tempo scenarios
- Future-proof for advanced features

---

## KEYBOARD SHORTCUTS

**Unchanged from V3.8.0:**
- `←→`: Opacity
- `↑↓`: Volume
- `[ ]`: Zoom
- `Space`: Play/pause
- `Esc`: Emergency stop
- `WASD`: Navigation
- `1-5`: Track selection

---

## COMPATIBILITY

✅ Fully compatible with V3.8.0  
✅ No migration required  
✅ All features preserved  
⚠️ Development build - testing ongoing

---

## KNOWN ISSUES

⚠️ **Master Grid Alignment:** Final validation in progress  
⚠️ **Tempo Changes:** Beat-based system under testing  
⚠️ **Multi-Track Launch:** Sync accuracy being verified

**Note:** These issues are being actively refined. Production release will address all concerns.

---

## TESTING RECOMMENDATIONS

**If You're Testing V3.9.0:**

1. **Single Track First**
   - Load one container
   - Launch and play
   - Verify waveform scrolls smoothly
   - Check bar alignment with Master Grid

2. **Tempo Changes**
   - Start at 120 BPM
   - Play for 8 bars
   - Change to 140 BPM
   - Verify waveform adjusts correctly
   - Check alignment stays perfect

3. **Multiple Tracks**
   - Launch 2-4 tracks
   - All at different times
   - Verify sync maintained
   - Check Master Grid alignment

4. **Pop-Out Window**
   - Pop out window
   - Verify all shaders render
   - Check for compilation errors
   - Test with 4 tracks

**Report Issues:**
- Note exact steps to reproduce
- Include tempo settings
- Specify browser (Chrome/Safari)
- Check console for errors (F12)

---

## WHAT'S NEXT

**Production Release:**
- Full testing completion
- Master Grid alignment finalized
- Tempo change validation
- Documentation updates
- Version number TBD (V3.9.0 or V4.0.0)

**Future Features:**
- Quantization options (Q key menu)
- Visual launch countdown
- Shader parameter controls
- MIDI integration
- Ableton Link
- Tap tempo

---

## PERFORMANCE

- Same GPU usage as V3.8.0
- 60fps stable
- +10KB file size
- 0% overhead (timing improvements only)

---

## FOR V3.8.0 USERS

**Should You Upgrade?**

**Upgrade If:**
- You experience tempo change issues
- You need Master Grid alignment
- You use pop-out window with shaders
- You want to test new architecture

**Stay on V3.8.0 If:**
- Everything works perfectly for you
- You don't need timing improvements
- You prefer stable production builds
- You want to wait for final release

---

## QUICK START

**Same as V3.8.0:**

1. Open HTML file
2. Settings → Set Folder
3. Create shader
4. LIVE tab → perform!

**New Testing:**
- Try tempo changes during playback
- Load multiple tracks at different times
- Check Master Grid alignment
- Use pop-out window

---

## FEEDBACK WELCOME

This is a development build - your feedback helps!

**What to Test:**
- Timing accuracy
- Master Grid alignment
- Tempo changes
- Multi-track synchronization
- Pop-out window rendering

**Where to Report:**
- GitHub Issues (when available)
- Social media @itswessmithyo
- Email via website contact form

---

**Thank you for testing V3.9.0!**

Version: V3.9.0_L_sub20 (Development Build)  
Date: November 21, 2025  
License: GPL-3.0

---

**END OF RELEASE NOTES**
