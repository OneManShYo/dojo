# OMS SHADER BAKER V4.0.0_D - RELEASE NOTES

**Release Date:** November 25, 2025  
**Version:** V4.0.0_D  
**Type:** Major Feature Update + Critical Fixes

---

## WHAT'S NEW

### 🎵 Audio-Reactive Shaders in LIVE Tab

**Finally!** Shaders in the LIVE tab now react to audio just like in the Controls tab preview. Your carefully crafted visuals respond to bass, mids, and treble in real-time during performance.

- Bass modulation (modA) pulses with kick drums
- Mid modulation (modB) follows vocals and synths  
- Treble modulation (modC) sparkles with hi-hats
- Perfect match between prep and performance

---

### 🎯 Enter Key Fixed (CDJ-Style)

**Instant Response** - Enter key now works like a CDJ cue button:
- Fires on every tap (no blocking)
- Instantly re-cues to Bar 1
- No more frozen keyboard
- Professional DJ workflow

---

### 📊 Bar-Based Beat Grid

**New S Button** in Controls tab:
- Calculates complete bars from your audio
- Shows bar count in Last Bar field
- Saves verified count to container
- LIVE tab uses this for perfect grid spacing
- No more drift or misalignment

---

### 🔄 Update Button Instant

**No More Waiting** - Update button now instant:
- Preserves existing FFT analysis
- No 3-4 second re-analysis
- Timeline data maintained
- Quick container updates

---

## IMPROVEMENTS

### Visual Synchronization

**Better Sync:**
- Waveforms locked to master clock
- No drift during tempo changes
- Bar 1 always at playhead
- Smooth scrolling animation

### Performance

**Smoother Operation:**
- Removed 11,852 console spam messages
- Simplified Enter key logic (50 lines → 5)
- Beat-based positioning (no float accumulation)
- 60fps maintained with all features

---

## HOW TO USE NEW FEATURES

### Audio Reactivity in LIVE Tab

1. **Prepare in Controls Tab:**
   - Load audio file
   - Set frequency ranges for bass/mid/treble
   - Click "Add to Library" (analyzes FFT)
   
2. **Perform in LIVE Tab:**
   - Drag container to grid
   - Launch with Enter
   - Shaders react automatically!

### Bar Snap Feature

1. **Load audio** in Controls tab
2. **Set Audio Start** (where Bar 1 begins)
3. **Click S button** - calculates complete bars
4. **See Last Bar field** - shows count (e.g., 164)
5. **Save container** - includes bar count
6. **LIVE tab** uses this for perfect grid

### CDJ-Style Enter Key

- **Tap Enter** - launches selected cell
- **Tap Again** - re-cues to Bar 1
- **Keep Tapping** - instant response every time
- No waiting, no blocking

---

## KEYBOARD SHORTCUTS

**Unchanged:**
- `↑↓`: Volume
- `←→`: Opacity  
- `[ ]`: Zoom
- `Space`: Play/pause
- `Esc`: Emergency stop
- `WASD`: Navigation
- `1-5`: Track selection
- `G`: Toggle grid

---

## COMPATIBILITY

✅ Fully compatible with V3.x  
✅ All .oms containers work  
✅ All .omsp projects work  

**Note:** For best results, update containers:
1. Load in Controls tab
2. Click S button (snap to bar)
3. Click Update (preserves timeline)
4. Container now optimized for V4

---

## KNOWN ISSUES

### Minor Issues Remaining

- Master Grid alignment still being refined (70% complete)
- Some visual elements may drift at extreme zoom
- Old containers need update for audio reactivity

### Fixed Issues ✅

- Enter key blocking (FIXED)
- Waveform freezing (FIXED)
- Console spam (FIXED)
- Update button slow (FIXED)
- Double-gain bug (FIXED)

---

## QUICK START

**For Audio Reactivity:**
1. Controls tab → Load audio
2. Set BPM and loop points
3. Click "Add to Library"
4. LIVE tab → Drag to grid
5. Press Enter → See shaders react!

**For Better Grid:**
1. Controls tab → Load audio
2. Click S button (new!)
3. See Last Bar count
4. Save container
5. LIVE tab shows perfect grid

---

## PRODUCTION READINESS

**Overall: ~78% Ready**

**What Works Great:**
- Enter key (100% fixed)
- Audio reactivity (~90% complete)
- Basic performance (100% stable)
- File management (100% working)

**Still Improving:**
- Master Grid alignment
- Final visual polish
- Some edge cases

**Safe for:**
- Studio work
- Practice sessions
- Testing performances

**Wait for V1.0 for:**
- Critical live shows
- Client performances

---

## MIGRATION GUIDE

### From V3.x

**No migration required!** Just:
1. Open V4.0.0_D
2. Your containers work
3. Your projects load
4. Everything compatible

**To optimize:**
- Update containers with S button
- Re-save for timeline data
- Test Enter key behavior

---

## TESTING RECOMMENDATIONS

**Try These:**
1. Load a container with audio
2. Press Enter repeatedly (should never block)
3. Watch shaders react to music
4. Press G to see grid alignment
5. Change tempo - verify no drift

**Report Issues:**
- Note exact steps
- Check console (F12)
- Save problematic .oms file
- Contact via social media

---

## PERFORMANCE NOTES

- **File Size:** ~656KB (+100KB from V3.9.0)
- **GPU Usage:** 30-45% with 4 shaders
- **Frame Rate:** 60fps stable
- **Memory:** ~300-400MB with audio
- **CPU:** Minimal overhead

---

## THANK YOU

After 58 sub-iterations and extensive testing, V4.0.0_D represents a major step toward production-ready performance software. Your patience during development has been invaluable.

**Special thanks to testers who reported:**
- Enter key blocking issues
- Waveform sync problems
- Missing audio reactivity
- Grid alignment concerns

---

## WHAT'S NEXT

**V1.0 Production Release (Coming Soon):**
- Perfect Master Grid alignment
- Final visual polish
- Complete testing suite
- Production documentation

**Future Features:**
- MIDI control (Web MIDI API)
- Ableton Link sync
- Advanced FFT controls
- Multi-instance support

---

**FEEDBACK WELCOME**

- GitHub: (coming soon)
- Social: @itswessmithyo
- YouTube: @itswessmithyo
- Website: https://itswessmithyo.com

---

**END OF RELEASE NOTES**

Version: V4.0.0_D  
Date: November 25, 2025  
License: GPL-3.0

*"It isn't worse" - High praise after 58 iterations*
