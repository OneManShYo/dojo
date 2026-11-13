# OMS SHADER BAKER V3.4.0 - RELEASE NOTES

**Release Date:** November 15, 2025  
**Version:** V3.4.0 Production Release  
**Previous Version:** V3.3.0  
**Type:** UI/UX Refinement (No New Features)

---

## TL;DR

V3.4.0 optimizes the interface for MacBook Pro 14" displays and achieves professional visual consistency throughout. This is a polish release - no new features, but significantly improved user experience.

**Key Improvements:**
- Perfect fit on MacBook Pro 14" at default scaling
- Professional mixer aesthetic (all knobs match exactly)
- Cleaner tab organization (single letters: C, P, L, W, S, LIVE)
- Waveform controls with modular hardware look
- Better visual hierarchy and polish

---

## WHAT'S NEW

### 1. MacBook Pro 14" Optimization

**Problem:** V3.3.0 didn't fit on MacBook Pro 14" screens - the Master column was completely cut off.

**Solution:** Reduced total layout from 1470px to 1320px (-150px).

**Result:**
✅ All 6 mixer columns visible (Scenes + 4 Tracks + Master)  
✅ Perfect fit at 1352px viewport width  
✅ 32px breathing room - professional appearance  
✅ No horizontal scrollbar

**Who Benefits:** Anyone using MacBook Pro 14" (or similar 13-15" laptops) at default scaling.

---

### 2. Visual Consistency - Professional Mixer Aesthetic

**Problem:** Zoom knob looked completely different from volume knobs - "like a junk shop" appearance.

**Solution:** Made zoom knob pixel-perfect match to volume knobs.

**Details:**
- Same cyan arc indicator showing value
- Same white center line pointer
- Same grey border
- Same 300° rotation range (7 o'clock to 5 o'clock)
- ZOOM label above knob for clarity

**Result:**
✅ Professional hardware mixer aesthetic  
✅ Visual consistency across all mixer controls  
✅ Easier to scan and understand at a glance

**Who Benefits:** Everyone - this is just better design.

---

### 3. Waveform Controls - Modular Look

**Problem:** Waveform controls had rounded corners, padding, gaps - looked soft and unprofessional.

**Solution:** Created sharp, modular hardware aesthetic.

**Changes:**
- Sharp corners (no border-radius)
- No padding or gaps
- Buttons fill container perfectly (2 × 35px = 70px)
- Flush against waveform canvas
- Aligned exactly with SCENES column above
- Removed unnecessary Q (quantize) buttons

**Result:**
✅ Professional, modular appearance  
✅ Perfect pixel alignment  
✅ Clean, tight spacing  
✅ Hardware mixer aesthetic

**Who Benefits:** Anyone who values professional-looking tools.

---

### 4. Tab Organization - Simpler Navigation

**Problem:** Long tab names took up space and required reading.

**Solution:** Single-letter tab names with clear meanings in context.

**New Tabs:**
- **C** = Controls (shader controls)
- **P** = Presets (project states .omsp)
- **L** = Library (containers .oms)
- **W** = WGSL (code editor)
- **S** = Settings (application config) **[NEW]**
- **LIVE** = Performance grid (unchanged)

**Result:**
✅ Cleaner tab bar  
✅ More space for content  
✅ Tab meanings clear in context  
✅ Settings tab for app-level config

**Who Benefits:** Everyone - faster navigation, cleaner UI.

---

### 5. Settings Tab - Better Organization

**Problem:** Application folder setup button was in Library tab (wrong place).

**Solution:** New Settings tab (S) for application-level configuration.

**What Moved:**
- "Set Application Folder" button → Settings tab
- Folder info display → Settings tab

**Why It Matters:**
Library tab is for content management (browsing, loading .oms files).  
Settings tab is for app configuration (one-time setup).

**Result:**
✅ Better separation of concerns  
✅ Cleaner Library tab  
✅ Logical organization

**Who Benefits:** New users setting up the app, anyone organizing their workspace.

---

### 6. Shader Output Management

**Problem:** Shader output preview was:
- Visible on all tabs (unnecessary)
- Scrolled with library content (annoying)
- Had redundant label taking space

**Solution:** Three improvements:

**A. Only Visible on LIVE Tab**
- Hidden on C, P, L, W, S tabs
- Only shows during performance (LIVE tab)

**B. Fixed at Bottom**
- Doesn't scroll with library containers
- Always visible while browsing
- Monitor shader while selecting content

**C. No Label**
- Removed "LIVE SHADER OUTPUT" text
- Recovered 23px vertical space
- More room for canvas on small screens

**Result:**
✅ Cleaner non-LIVE tabs  
✅ Monitor shader while browsing  
✅ Better screen space usage on MacBook Pro 14"

**Who Benefits:** Anyone using LIVE tab for performance, anyone on smaller screens.

---

### 7. Code Quality Improvements

**Naming Refactor: audioLibrary → applicationFolder**

**Why:** Folder stores ALL app data (.oms containers, .omsp project states, audio files), not just audio.

**Changes:**
- 42 variable renames
- 4 function renames
- 3 element ID updates
- All UI text updated
- Fixed 2 broken function calls

**Result:**
✅ Clearer, more accurate naming  
✅ Better code maintainability  
✅ Fixed legacy bugs

**Who Benefits:** Future development, code contributors, power users reading code.

**Other Quality Improvements:**
- Grey borders use proper CSS (not pseudo-elements)
- Hidden scrollbars for cleaner appearance
- Better flexbox architecture for shader output positioning

---

## WHAT DIDN'T CHANGE

### All Existing Features Work Identically

✅ Shader creation and editing  
✅ Audio-reactive modulators (modA, modB, modC)  
✅ FFT analysis (32768 samples)  
✅ .oms container save/load  
✅ .omsp project state save/load  
✅ Multi-track audio (4 tracks in LIVE tab)  
✅ Perfect beat grid synchronization  
✅ Master tempo control (60-200 BPM)  
✅ Time-stretching (tempo scaling)  
✅ Waveform visualization  
✅ Mixer controls (volume, mute, solo)  
✅ Zoom control (10-200 px/sec)

### Files Remain Compatible

✅ Your existing .oms container files work unchanged  
✅ Your existing .omsp project state files work unchanged  
✅ Your existing shaders continue to render  
✅ Your existing audio files work  
✅ Application folder location persists

**No Migration Required:** Just replace V3.3.0 HTML with V3.4.0 HTML. Everything else stays the same.

---

## WHAT'S STILL COMING (Future Versions)

**V3.4.0 is a POLISH release.** No new functional features were added.

### Planned for V3.5.0

🔜 **Scene Launching**
- Click scene button to trigger entire row
- Quantized launching (snap to bars/phrases)
- Loop bracket visualization
- Crossfading between cells

🔜 **MIDI Control**
- Web MIDI API integration
- Hardware controller support
- Midifighter Twister mapping
- CC assignments for knobs

🔜 **Ableton Link**
- Synchronize with Ableton Live
- Network tempo sync with other devices
- Professional DJ workflow integration

### Vision for V4.0.0

🎯 **OMS Session**
- Complete Ableton + EboSuite replacement
- Recording/export from LIVE tab
- Advanced session management
- Professional-level performance environment

**Philosophy:** "Anyone can be a OneManShYo"

---

## UPGRADE INSTRUCTIONS

### How to Upgrade from V3.3.0

**Super Simple:**
1. Download OMS_ShaderBaker_V3_4_0.html
2. Replace your V3.3.0 HTML file
3. Done!

**Everything Else Persists:**
- Your application folder location (browser storage)
- Your .oms container files (compatible)
- Your .omsp project state files (compatible)
- Your shader code (unchanged)
- Your audio files (compatible)

**No Configuration Required**

---

## SYSTEM REQUIREMENTS

### Browser Support

**Required:** WebGPU-enabled browser

**Recommended:**
- Chrome 113+ (best WebGPU support)
- Edge 113+

**Experimental:**
- Safari Technology Preview
- Firefox with WebGPU flag enabled

### Operating System

**macOS:** 10.15+ (Catalina or later)  
**Windows:** 10/11 with DirectX 12 support  
**Linux:** Vulkan support required

### Screen Resolution

**Recommended:** 1512×982 or higher (MacBook Pro 14" optimal)  
**Minimum:** 1920×1080 for comfortable use

### Hardware

**CPU:** Modern multi-core processor (Intel i5/Ryzen 5 or better)  
**GPU:** Integrated graphics OK, dedicated GPU better  
**RAM:** 8GB minimum, 16GB recommended  
**Storage:** Minimal (single 445KB HTML file)

---

## KNOWN ISSUES

**None reported.**

All 31 development iterations were tested incrementally.  
Final V3.4.0 production build tested on MacBook Pro 14" at default scaling.

---

## SUPPORT & COMMUNITY

### Where to Get Help

**YouTube:** ONEMANSHYO channel  
- Tutorial videos
- Performance examples
- Shader creation guides
- 100k monthly views

**Website:** https://onemanshyo.com

**License:** GPL-3.0 (open source, free forever)

### "Ghost Protocol" Philosophy

**What It Means:**
- Tools provided free and open source
- No ongoing support commitment
- Use at your own risk
- Community-driven improvement

**Why It Works:**
- Empowers solo creators
- No dependency on company support
- Open source = immortal software
- "Anyone can be a OneManShYo"

---

## CREDITS

**Developer:** Wes Jones (ONEMANSHYO)  
**AI Assistant:** Claude (Anthropic) - V3.4.0 development series  
**Philosophy:** Solo creator empowerment  
**License:** GPL-3.0

**Special Thanks:**
- Electronic music community
- Open source contributors
- Beta testers on MacBook Pro 14"
- Everyone who performs with these tools

Your creativity inspires continued development.

---

## CHANGELOG SUMMARY

**31 Iterations (V3_4_0_A through V3_4_0_AE)**

### Layout & Display
- MacBook Pro 14" optimization (1470px → 1320px)
- Equal column widths (153px uniform)
- Asymmetric padding for visual balance
- Grey borders with proper CSS

### Visual Consistency
- Zoom knob matches volume knobs exactly
- Waveform controls modular look
- Professional mixer aesthetic
- Hidden scrollbars

### Tab Organization
- Single-letter tab names (C, P, L, W, S, LIVE)
- Settings tab added
- Cleaner headers
- Better separation of concerns

### Shader Output
- Only visible on LIVE tab
- Fixed at bottom (doesn't scroll)
- No redundant label
- Better screen space usage

### Code Quality
- Naming refactor: audioLibrary → applicationFolder
- 42 variable renames, 4 function renames
- Fixed 2 broken function calls
- Cleaner CSS architecture

**Total Changes:** ~440 lines modified across 31 iterations  
**Development Time:** 1 day (November 15, 2025)

For complete technical details, see:  
📄 OMS_ShaderBaker_V3_4_0_B_Complete_Changelog.md

---

## QUICK START

### First Time Setup

1. **Open the App**
   - Double-click OMS_ShaderBaker_V3_4_0.html
   - Browser opens (Chrome/Edge recommended)

2. **Set Application Folder**
   - Click Settings tab (S)
   - Click "Set Application Folder"
   - Choose a folder for your .oms and .omsp files
   - Permission persists between sessions

3. **Create Your First Shader**
   - Click Controls tab (C) for visual controls
   - OR click Code tab (W) for WGSL editing
   - Modify parameters and see live preview
   - Click "Save Container" to save as .oms file

4. **Arrange for Performance**
   - Click LIVE tab
   - Drag .oms containers from Library to grid cells
   - Set master tempo
   - Adjust volume, mute, solo per track
   - Launch cells during performance

5. **Export Video**
   - Set render resolution in Controls tab
   - Click "Render Video"
   - Use FFmpeg to convert to ProRes 422
   - Import into Ableton Live + EboSuite

**That's It!**

For detailed guides, see:  
📖 OMS_ShaderBaker_V3_4_0_F_UserGuide_All.md  
📖 OMS_ShaderBaker_V3_4_0_G_UserGuide_FFmpeg_Video_Guide.md

---

## FEEDBACK

V3.4.0 is a polish release preparing the foundation for future features.

**What do you think?**
- Does it feel more professional?
- Is the MacBook Pro 14" optimization helpful?
- Any bugs or issues?
- What features do you want next?

Share your feedback:
- YouTube comments on ONEMANSHYO videos
- GitHub issues (if repository exists)
- Community forums

---

## FINAL THOUGHTS

V3.4.0 took 31 iterations to get right. That's 31 small, surgical improvements building on each other. This is how great software is made - not one big rewrite, but many small refinements.

The result: A professional, polished tool that fits your screen and looks good doing it.

**Thanks for using OMS Shader Baker.**

Now go create something amazing! 🎨🎵✨

---

**END OF RELEASE NOTES**

Version: V3.4.0 Production Release  
Date: November 15, 2025  
License: GPL-3.0  
Website: https://onemanshyo.com
