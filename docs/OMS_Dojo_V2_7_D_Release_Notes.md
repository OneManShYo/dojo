# OMS DOJO V2_7 - RELEASE GUIDE
## "Shader Reactivity Release"

**Version:** V2_7  
**Release Date:** November 2025  
**Type:** Major Feature Release  
**File Size:** 215KB

---

## EXECUTIVE SUMMARY

V2_7 is a transformative release that makes ONEMANSHYO Dojo truly responsive to bass music dynamics. Two major systems were added:

1. **User-Definable Frequency Mapping:** Target specific frequency ranges (kick drums, sub-bass, lead bass) with three independent modulators, backed by 64x more precise FFT analysis.

2. **Control States System:** Save and load complete render configurations with custom names, plus 5 curated library presets for common scenarios.

**Bottom Line:** You can now create shaders that respond precisely to specific musical elements, and save your render setups for instant recall. This is the release that makes bass music visualization actually work.

---

## WHAT'S NEW

### 1. User-Definable Frequency Mapping

**Before V2_7:**
- Hardcoded frequency ranges (bass/mid/treble)
- No control over what frequencies each modulator tracked
- Generic "bass" response didn't distinguish kick from sub-bass

**After V2_7:**
- 6 frequency input fields (Freq1/Freq2 for Mod A/B/C)
- Define exact Hz ranges: 30-50Hz for kicks, 50-75Hz for sub, 100-500Hz for lead bass
- Arrow key navigation (±5Hz fine, ±50Hz coarse)
- 32768 FFT analysis provides ~1.46 Hz/bin precision
- Real-time validation with push behavior

**Why It Matters:**
Bass music (house, techno, dubstep) has distinct frequency layers. Now you can make visuals react to specific elements instead of generic "bass."

### 2. Control States System

**What It Does:**
Save complete render configurations (21 values) with custom names. Switch between setups instantly.

**What Gets Saved:**
- Shader Timing: tempo, loop length, render duration
- All 3 Modulators: subdivision, waveform, gain, freq1, freq2
- Output Settings: orientation, resolution, fps

**Library Presets (5 Included):**
1. **Quick Test:** Portrait 720p 24fps, 30sec (fast iteration)
2. **HD Render:** Landscape 1080p 60fps, 1min (standard export)
3. **4K Final:** Landscape 2160p 60fps, 30sec (maximum quality)
4. **Instagram Story:** Portrait 1080p 30fps, 15sec (vertical social)
5. **TikTok:** Portrait 1080p 60fps, 30sec (vertical social)

**Why It Matters:**
Stop manually re-entering render settings. Select preset → render → done.

### 3. UI/UX Improvements

- Section labels: "Shader Timing" and "Shader Reactivity"
- Info box showing current state configuration
- Inline save input (no browser dialogs)
- Professional layout alignment
- Disabled state for frequency fields (cyan overlay when no audio)

### 4. Bug Fixes

- Frequency validation timing (audio load triggers)
- Control States ID schema migration (6 ID mismatches fixed)
- Zero value support (0 gain = disable modulator)
- Unicode corruption cleanup
- DOM timing issues resolved
- Null safety improvements

---

## QUICK START GUIDE

### New User Workflow

**1. Open Application**
- Download OMS_Dojo_V2_7.html
- Open in Safari Technology Preview or Chrome Canary
- WebGPU must be enabled

**2. Load Preset Shader**
- Click **Presets** tab
- Browse Library or User Presets
- Press Tab to navigate, Enter to load
- Shader appears in Code tab automatically

**3. Select Control State**
- Click **Controls** tab
- Select "Quick Test" from Control States dropdown
- All fields update automatically (120bpm, Portrait 720p 24fps, 30sec)

**4. Render Video**
- Click **Render** button
- Watch progress: "Rendering 00:15/00:30 | Frame 450/900"
- Video downloads when complete

**5. Test Result**
- Load video in your DJ software (Ableton + EboSuite)
- Video is silent (no audio track)
- Plays at 120 BPM, syncs with music

### Bass Music Workflow (NEW)

**Goal:** Create shader that responds to kick drums, sub-bass, and lead bass separately.

**1. Load Bass Track**
- Click **Upload Audio** button
- Select your track (house/techno/dubstep)
- Enter tempo manually (e.g., 128 BPM)
- Waveform displays with beat grid

**2. Define Frequency Ranges**
- **Mod A Freq1/Freq2:** 30-50 Hz (kick drums)
- **Mod B Freq1/Freq2:** 50-75 Hz (sub-bass)
- **Mod C Freq1/Freq2:** 100-500 Hz (lead bass)
- Use arrow keys to fine-tune (±5Hz, ±50Hz with Shift)

**3. Adjust Gains**
- **Mod A Gain:** 1.0 (full kick response)
- **Mod B Gain:** 0.5 (subtle sub movement)
- **Mod C Gain:** 0.8 (strong lead bass reaction)
- Gain = 0 disables modulator entirely

**4. Set Loop Range**
- Use arrow keys to position Loop In/Out brackets
- Target drop section with strong bass elements
- Audition loop with Play button (Loop button cyan when active)

**5. Save Control State**
- Click **Save** button in Control States section
- Enter name: "Techno Bass Reactivity"
- State saves to browser localStorage
- Appears in dropdown for future use

**6. Render**
- Click **Render** button
- Video duration = Loop In to Loop Out
- Output includes audio from loop range
- Download and test in Ableton

### Instagram/TikTok Export

**1. Select Social Preset**
- **Instagram Story:** Portrait 1080p 30fps, 15sec
- **TikTok:** Portrait 1080p 60fps, 30sec
- Optimized for vertical video platforms

**2. Load Shader**
- Use vibrant, high-contrast shader
- Portrait orientation works best for vertical

**3. Render**
- Format already configured by preset
- No manual settings needed
- Perfect 9:16 aspect ratio

**4. Upload**
- Video ready for Instagram Stories or TikTok
- No additional processing required

---

## PRODUCTION WORKFLOW

### Complete DJ Set Preparation

**Phase 1: Shader Development**
1. Create shaders in Claude (conversational WGSL generation)
2. Test in Dojo Code tab
3. Tweak visual controls (modulators, tempo)
4. Save to User Presets

**Phase 2: Audio-Reactive Configuration**
1. Upload track to Dojo
2. Set Loop In/Out to drop section
3. Define frequency ranges for bass response
4. Test with Loop playback
5. Save as Control State (e.g., "Track Name - Drop")

**Phase 3: Video Rendering**
1. Select Control State for track
2. Render video (HD Render or 4K Final preset)
3. Convert to ProRes 422 with FFmpeg (see FFmpeg guide)
4. Import to Ableton session

**Phase 4: Live Performance**
1. Load videos in EboSuite (silent, no audio track)
2. Audio tracks in separate channels (DJ mixing)
3. Trigger videos with Midifighter Twister
4. Mix visuals with audio seamlessly

**Repeat for each track in DJ set.**

---

## FEATURE DEEP DIVE

### Frequency Mapping System

**How It Works:**

1. **FFT Analysis:** Audio waveform analyzed at 32768 samples
2. **Frequency Bins:** 16384 bins, each ~1.46 Hz wide
3. **User Range:** You define Freq1-Freq2 (e.g., 30-50 Hz)
4. **Bin Selection:** System calculates which bins to average
5. **Amplitude:** Average amplitude across bins = 0-1
6. **Smoothing:** Exponential smoothing (0.7) reduces jitter
7. **Gain:** Multiply by user gain (0-10x)
8. **Shader:** Final value sent to shader as modA/modB/modC

**Practical Example:**

```
Kick drum hits at 40 Hz
Mod A set to 30-50 Hz
FFT detects amplitude spike in bins 20-34
Average amplitude = 0.8
After smoothing = 0.75
Gain = 1.0
Result: modA = 0.75 in shader
Visual responds strongly to kick
```

**Validation Rules:**

- Freq1 must be ≤ Freq2
- Both in range 0-24000 Hz (Nyquist limit)
- **Push Behavior:**
  - Freq1 increase pushes Freq2 up if needed
  - Freq2 decrease pushes Freq1 down if needed
  - Prevents invalid ranges automatically

**Disabled State:**

- Frequency fields disabled when no audio loaded
- Cyan overlay indicates "Audio Required"
- Values visible but not editable
- Professional UX for non-functional state

### Control States System

**Storage:**

- Saved to browser LocalStorage
- Namespace: `controlStates`
- Survives browser restarts
- Per-domain (claude.ai vs local file)

**State Types:**

1. **Library States:** Hardcoded, can't be deleted
2. **User States:** Custom, can be deleted
3. **Custom State:** Always available, shows current values

**Save Workflow:**

1. Adjust all controls to desired values
2. Click **Save** button
3. Inline input appears (no browser dialog)
4. Enter name (e.g., "My Drop Config")
5. Press Enter to save, Escape to cancel
6. State appears in dropdown immediately

**Load Workflow:**

1. Select state from dropdown
2. All 21 fields update instantly
3. Info box displays configuration
4. Ready to render

**Auto-Switch to Custom:**

- Any field edit switches dropdown to "Custom"
- Indicates current values ≠ any saved state
- Prevents accidental overwrites

**Info Box Display:**

Shows complete configuration:
```
Quick Test
Tempo: 120 BPM | Loop: 16 bars | Duration: 00:30.000
Mod A: 1/4 sine × 1 | 30-50 Hz
Mod B: 1/4 sine × 1 | 50-75 Hz
Mod C: 1/4 sine × 1 | 100-500 Hz
Output: portrait 720p @ 24fps
```

### Zero Value Feature

**Use Case:** Selective reactivity

**Examples:**
- Set Mod A gain = 0 → disable bass response
- Set Mod B/C gain = 1.0 → keep mid/treble active
- Create "visual-only" presets (all gains = 0)
- Fine-tune which elements react to audio

**Implementation:**
- parseFloat() handles 0 correctly
- No "|| 0" fallbacks (treats 0 as falsy)
- Zero values save and restore properly

---

## KEYBOARD SHORTCUTS

### Global
- **Ctrl+S:** Save current shader to User Presets
- **Ctrl+R:** Start/stop render

### Frequency Fields (When Focused)
- **↑/↓ Arrow:** ±5 Hz (fine adjustment)
- **Shift+↑/↓:** ±50 Hz (coarse adjustment)

### Presets Tab
- **Tab:** Navigate thumbnails
- **Enter:** Load selected shader

### Control States Save
- **Enter:** Confirm save
- **Escape:** Cancel save

---

## TECHNICAL SPECIFICATIONS

### FFT Analysis
- **Size:** 32768 samples
- **Bins:** 16384
- **Sample Rate:** 48000 Hz (detected)
- **Nyquist:** 24000 Hz
- **Bin Width:** ~1.46 Hz/bin
- **Smoothing:** 0.7 exponential

### File Size
- **V2.6.0:** 199KB
- **V2_7:** 215KB (+16KB)
- **Single HTML file:** No external dependencies

### Browser Requirements
- Safari Technology Preview 15+ (recommended)
- Chrome Canary with WebGPU flag enabled
- Edge Canary with WebGPU flag enabled

### Export Formats
- **Container:** WebM
- **Codec:** VP8
- **Resolutions:** 720p, 1080p, 1440p, 2160p
- **Frame Rates:** 24, 30, 60 FPS
- **Orientations:** Portrait, Landscape, Square

---

## COMMON WORKFLOWS

### 1. Fast Iteration (Testing Multiple Shaders)

**Goal:** Quickly preview many shaders to find the best one.

1. Select "Quick Test" Control State (720p 24fps 30sec)
2. Click Presets tab
3. Tab through shader thumbnails
4. Press Enter to load shader
5. Click Render
6. Download and review
7. Repeat for next shader

**Time:** ~2-3 minutes per shader (30sec render + review)

### 2. Final Export (High Quality)

**Goal:** Create production-ready video for DJ set.

1. Select "4K Final" Control State (2160p 60fps 30sec)
2. Load your best shader
3. Upload audio track
4. Set Loop In/Out to key section
5. Fine-tune frequency ranges
6. Click Render (takes ~5-10 minutes)
7. Convert to ProRes 422 with FFmpeg
8. Import to Ableton

**Time:** ~15-20 minutes total

### 3. Social Media Content

**Goal:** Create vertical video for Instagram/TikTok.

1. Select "Instagram Story" or "TikTok" preset
2. Load vibrant shader
3. Render (no audio needed)
4. Upload directly to platform

**Time:** ~1-2 minutes

### 4. Bass Music Showcase

**Goal:** Demonstrate bass-reactive visuals.

1. Load heavy bass track (dubstep/techno)
2. Set Mod A: 30-50 Hz (kicks)
3. Set Mod B: 50-75 Hz (sub-bass)
4. Set Mod C: 100-500 Hz (lead bass)
5. Loop drop section
6. Render with "HD Render" preset
7. Share on YouTube/Instagram

**Time:** ~10-15 minutes

---

## TROUBLESHOOTING

### Frequency Fields Not Updating Info Box

**Problem:** Change freq values, info box doesn't update.

**Solution:** This is expected when no audio loaded. Frequency fields are disabled until audio present. Upload audio track, fields become active, info box updates.

### Control State Not Loading

**Problem:** Select state from dropdown, fields don't change.

**Cause:** Browser cache issue or localStorage corruption.

**Solution:**
1. Open browser console (F12)
2. Type: `localStorage.clear()`
3. Refresh page
4. Library states reload automatically
5. User states lost (recreate as needed)

### Zero Gain Not Working

**Problem:** Set gain = 0, modulator still active.

**Cause:** Old browser cache (pre-V2_7).

**Solution:**
1. Hard refresh (Ctrl+Shift+R)
2. Clear browser cache
3. Reload page
4. Zero values now work correctly

### Render Progress Not Showing

**Problem:** Render starts, no elapsed time or frame counter.

**Cause:** JavaScript error (check console).

**Solution:**
1. Open browser console (F12)
2. Look for errors
3. Refresh page
4. Try again
5. If persists, report issue

### FFT Performance Issues

**Problem:** Frame drops during audio playback.

**Cause:** 32768 FFT too heavy for older hardware.

**Solution:** Code modification required (reduce FFT to 8192). See Developer Documentation for instructions.

---

## MIGRATION FROM V2.6.0

### Breaking Changes

**Field ID Changes:**
- Old: bassSubdiv, bassWave, bassGain
- New: modASubdiv, modAWaveform, modAGain
- Same for mid → modB, treble → modC

**Impact:** User presets from V2.6.0 won't work in V2_7.

**Workaround:** Recreate presets in V2_7. (Preset code is compatible, just control states need recreation.)

### New Features Available

- Define custom frequency ranges
- Save render configurations
- Use library presets
- Zero gain for selective reactivity

### Unchanged Features

- Shader Presets system (separate from Control States)
- Video export workflow
- FFmpeg conversion process
- Ableton integration workflow

---

## BEST PRACTICES

### Frequency Range Selection

**Kick Drums:** 30-50 Hz  
**Sub-Bass:** 50-75 Hz  
**Bass:** 80-250 Hz  
**Low-Mid:** 250-500 Hz  
**Mid:** 500-2000 Hz  
**High-Mid:** 2000-6000 Hz  
**Treble:** 6000-20000 Hz

**Tip:** Use narrow ranges (20-30 Hz) for precise targeting. Use wide ranges (100-1000 Hz) for general frequency response.

### Gain Settings

**Subtle Movement:** 0.3-0.5  
**Normal Response:** 0.8-1.2  
**Strong Reaction:** 1.5-2.5  
**Extreme Effect:** 3.0-10.0  
**Disabled:** 0.0

**Tip:** Start with gain = 1.0, adjust based on visual intensity.

### Control State Organization

**Naming Convention:**
- Test renders: "Quick Test", "Preview"
- Final exports: "4K Final", "HD Export"
- Track-specific: "Track Name - Drop", "Track Name - Breakdown"
- Social media: "IG Story", "TikTok 60fps"

**Tip:** Create states for common scenarios first (test/preview/final), then track-specific states as needed.

### Render Strategy

**Development Phase:**
- Use "Quick Test" preset (720p 24fps 30sec)
- Iterate rapidly through shaders
- Don't waste time on 4K renders yet

**Final Export Phase:**
- Use "HD Render" or "4K Final" preset
- Only render shaders you've already tested
- Convert to ProRes 422 for Ableton

---

## PERFORMANCE TIPS

### Faster Renders

1. **Lower Resolution:** 720p renders 4x faster than 1440p
2. **Lower FPS:** 24fps renders 2.5x faster than 60fps
3. **Shorter Duration:** 15sec renders 4x faster than 60sec
4. **Simpler Shaders:** Fewer calculations = faster render

### Better Quality

1. **Higher Resolution:** 2160p for maximum detail
2. **Higher FPS:** 60fps for smooth motion
3. **Longer Duration:** 60-120sec for full track sections
4. **Complex Shaders:** More visual interest

### Balance

**Recommended for most users:**
- Resolution: 1080p (quality/speed balance)
- FPS: 30fps (smooth enough, renders fast)
- Duration: 30-60sec (useful length, manageable render time)

**Render Time Examples (M1 Mac):**
- 720p 24fps 30sec: ~45 seconds
- 1080p 30fps 60sec: ~2 minutes
- 2160p 60fps 30sec: ~8 minutes

---

## NEXT STEPS

### Immediate Actions

1. **Download V2_7 HTML file**
2. **Open in Safari Technology Preview**
3. **Load a Library shader from Presets tab**
4. **Select "Quick Test" Control State**
5. **Click Render and download your first video**

### Learning Path

1. **Week 1:** Experiment with Library shaders and presets
2. **Week 2:** Create custom shaders with Claude
3. **Week 3:** Upload audio tracks, explore frequency mapping
4. **Week 4:** Build Control States library for your DJ set

### Community

- Share shaders and presets with other users
- Post bass-reactive videos on social media
- Tag #OMSDojo #OneManShYo
- Contribute to open source (GPL-3.0 license)

---

## SUPPORT & RESOURCES

**Website:** https://www.itswessmithyo.com/  
**License:** GPL-3.0 (free and open source)  
**Documentation:** Complete user guide included in deliverables  
**FFmpeg Guide:** Included for ProRes 422 conversion

**For Issues:**
- Check browser console for errors
- Verify WebGPU enabled
- Try hard refresh (Ctrl+Shift+R)
- Check deliverables for technical docs

---

## VERSION HISTORY

**V2_7 - November 2025**  
33 iterations focused on frequency mapping and Control States system

**V2.6.0 - November 2025**  
20 iterations focused on preset system with visual thumbnails

**V2.5.0 - November 2025**  
24 iterations focused on render accuracy and 90% file size reduction

---

## CREDITS

**Created by:** Wes Smith (OneManShYo)  
**Development:** Claude AI (Anthropic)  
**Technology:** WebGPU, WGSL, Web Audio API  
**Purpose:** Empowering solo DJ/VJ creators with professional tools

**Mission:** "Anyone can be a OneManShYo"

---

**END OF RELEASE GUIDE**
