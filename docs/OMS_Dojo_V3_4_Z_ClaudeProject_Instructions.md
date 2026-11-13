# OMS SHADER BAKER V3.4.0 - CLAUDE PROJECT INSTRUCTIONS

**Version:** V3.4.0
**Date:** November 15, 2025
**Purpose:** Complete project context for Claude AI development

---

## PROJECT OVERVIEW

### What is OMS Shader Baker?

OMS Shader Baker is a **WebGPU-based shader creation and performance tool** for solo DJ/visual artists. It creates audio-reactive shader visualizations for live DJ performances, following the proven ONEMANSHYO workflow:

1. Create shaders (with or without audio)
2. Render as silent videos (WebM/MP4)
3. Convert to ProRes 422 via FFmpeg
4. Import into Ableton Live + EboSuite
5. Perform live with synchronized audio-reactive visuals

**Philosophy:** "Anyone can be a OneManShYo" - empowering solo creators to achieve professional audio-visual performances without teams or expensive commercial software.

**Architecture:** Single HTML file (~445KB, 10,207 lines) with zero external dependencies. Complete portability, works offline, no build process.

**License:** GPL-3.0 (open source, free forever, "ghost protocol")

### Core Workflow

**Shader Creation:**
- WebGPU/WGSL shader rendering (GPU-accelerated)
- Visual controls tab (no coding required)
- Code editor tab (direct WGSL editing)
- Three-modulator audio reactivity (bass, mids, treble)
- 32768-sample FFT analysis
- User-definable frequency mapping

**Performance Workflow:**
- 4-track × 8-scene performance grid (LIVE tab)
- Multi-track audio with perfect beat grid synchronization
- Master tempo control (60-200 BPM)
- Time-stretching all tracks to master tempo
- Professional mixer controls (volume, mute, solo, meters)
- Dynamic zoom (10-200 px/sec)
- Bar-style waveforms (Serato-style)

**File Management:**
- .oms containers (complete shader + audio + controls packages)
- .omsp project states (entire LIVE tab session state)
- Application folder (File System Access API)
- Auto-save workflow
- LIBRARY-first architecture

### User Profile

**Wes Jones (ONEMANSHYO):**
- Electronic music producer/DJ with 35 years experience
- 500+ releases, 7,000+ days in Beatport charts
- Founder of Juice Recordings
- YouTube channel: 100k monthly views
- Livestream DJ performances with reactive visuals
- Solo creator philosophy ("Anyone can be a OneManShYo")

---

## CURRENT STATE: V3.4.0

### Production Status

**V3.4.0 Complete (November 15, 2025):**
✅ **UI/UX Refinement Release** (31 iterations, no new features)
- MacBook Pro 14" optimized layout (1320px fits 1352px viewport)
- Visual consistency - all mixer controls match perfectly
- Single-letter tab names (C, P, L, W, S, LIVE)
- Settings tab for application-level configuration
- Waveform controls with modular aesthetic (sharp, tight)
- Zoom knob matches volume knobs exactly (cyan arc, white line, border)
- Shader output fixed at bottom (doesn't scroll)
- Hidden scrollbars for cleaner UI
- Refactored naming: audioLibrary → applicationFolder (42 variables, 4 functions, 3 IDs)
- Professional hardware mixer aesthetic throughout

**V3.3.0 Foundation (Still Functional - CRITICAL):**
✅ **Perfect Beat Grid Synchronization** ("Soldiers at the goalpost" - 53 iterations)
- Multi-BPM track alignment (124/126/128 BPM all sync)
- Master tempo control (60-200 BPM)
- Time-stretching all tracks to master tempo
- Synced waveform scrolling (Serato-style)
- Complete audio timing data storage
- Bar-style waveforms (mixing-optimized)
- Multi-track audio playback (4 tracks)
- Mixer controls (volume, mute, solo, meters)
- Project States persistence (.omsp)
- Keyboard navigation
- Beat-aligned waveform bars (1 per beat)
- Dynamic zoom range (10-200 px/sec)
- Phrase markers (magenta every 4 bars)
- Real-time shader preview (BETA)

**What Doesn't Work Yet:**
- ❌ Scene launching (rows)
- ❌ Shader rendering in cells (thumbnails only)
- ❌ Master output compositing
- ❌ Waveform control buttons (visual only)
- ❌ MIDI control
- ❌ Ableton Link

**File:** OMS_ShaderBaker_V3_4_0.html (10,207 lines, 445KB)

---

## V3.4.0 DEVELOPMENT SUMMARY

### Overview

**Type:** UI/UX Refinement Release (No New Features)
**Iterations:** 31 (V3_4_0_A through V3_4_0_AE)
**Development Time:** 1 day (November 15, 2025)
**Lines Modified:** ~440 lines across 31 iterations
**Approach:** Surgical changes, one improvement per iteration

### Major Achievements

**1. MacBook Pro 14" Optimization**
- Reduced total layout: 1470px → 1320px (-150px)
- All 6 mixer columns visible at 1352px viewport width
- Perfect fit with 32px breathing room
- Calculated uniform column widths (153px each)
- Asymmetric padding for visual balance (1px left, 4px right)

**2. Visual Consistency**
- Zoom knob matches volume knobs exactly (same drawing code)
- Waveform controls modular look (sharp corners, 70px, no gaps)
- Professional hardware mixer aesthetic throughout
- All rotary knobs identical styling
- Clean, tight spacing everywhere

**3. Tab Organization**
- Single-letter tab names: C (Controls), P (Presets), L (Library), W (WGSL Code), S (Settings), LIVE
- Settings tab added for application-level controls
- Library folder button moved to Settings (better separation of concerns)
- Clean headers: "APPLICATION", "Containers"
- Shader output only visible on LIVE tab

**4. Shader Output Management**
- Fixed at bottom of left panel (doesn't scroll with containers)
- Only visible on LIVE tab (hidden on C, P, L, W, S)
- Proper flexbox positioning (flex: 0 0 auto)
- Recovered vertical space (removed redundant "LIVE SHADER OUTPUT" label)
- Monitor shader while browsing library

**5. Code Quality**
- Naming refactor: audioLibrary → applicationFolder
  - 42 variable renames
  - 4 function renames
  - 3 element ID updates
  - Fixed 2 broken function calls
- CSS borders instead of pseudo-elements (cleaner box model)
- Hidden scrollbars (cross-browser support)
- Better separation of concerns

### Key Learnings from V3.4.0

**1. Structural Consistency Over Visual Similarity**
- If elements should align, they MUST use same layout structure
- Visual matching requires identical HTML structure AND CSS properties
- Don't fix with transforms or inline styles - use proper CSS

**2. Flexbox Requires Container Matching**
- Container width affects flex child behavior
- Equal widths ensure consistent flex behavior
- min-height: 0 critical for nested scrolling in flex containers

**3. Modular Design = No Compromises**
- Sharp corners, no padding, no gaps - fills space perfectly
- Hardware mixer aesthetic requires precision, not approximation
- Professional polish comes from eliminating ALL extra space

**4. User Feedback Drives Iteration**
- Direct, immediate feedback prevents wasted effort (e.g. "wait, WTF mate...this is worse...lol")
- 31 iterations refined one concept to perfection
- Sometimes the answer is simpler than you think

---

## V3.3.0 FOUNDATION (STILL CRITICAL)

### The "Soldiers at the Goalpost" Achievement

**The Core Innovation:**
Perfect beat grid alignment regardless of track BPM or master tempo setting. Professional DJ timing accuracy achieved through the "One Timeline" architecture.

**53 Iterations Total (V3_3_0_A through V3_3_0_BETA_A):**

**Phase 1: Early Attempts (A-AF, 32 iterations)**
- Explored scaleX transforms (failed)
- Attempted per-track timelines (failed)
- Discovered fundamental architecture needed

**Phase 2: The Breakthrough (K-M, 3 iterations)**
- V3.3.0_K: One Timeline architecture (revolutionary)
- V3.3.0_L: Canvas sizing fix
- V3.3.0_M: Scroll at real time (PERFECT SYNC ACHIEVED!)

**Phase 3: Refinements (N-Z, 13 iterations)**
- Magenta phrase marker alignment
- Zoom control (10-200 px/sec)
- Beat-aligned waveform bars
- Tempo change handling
- Sample reading vs drawing separation

**Phase 4: Shader Preview (BETA_A)**
- Real-time WebGPU rendering in LIVE tab
- Foundation for track compositing

**Key Quote:** "mate we have solved it" - Wes Jones, after V3.3.0_M

---

## DEVELOPMENT PROTOCOLS (CRITICAL)

### 1. Iteration Protocol

**ALWAYS use alphabetical iteration:**
- Sequential: A, B, C, D... Z, AA, AB, AC... AZ, BA, BB...
- Never skip letters
- One feature per iteration
- Each iteration is a complete work session

**Example:** V3_5_0_A → V3_5_0_B → V3_5_0_C (never A → C)

### 2. Build Protocol (CRITICAL)

**NEVER build without asking first:**
- Research/investigate = fine (no ask needed)
- Discuss/analyze = fine (no ask needed)
- Building code = ALWAYS ask first (mandatory)

**Why:** User is usually multi-messaging about the topic. Wait for full context before implementing.

**Pattern:**
1. User describes issue/feature
2. Claude investigates/discusses
3. User may send multiple messages
4. Claude asks: "Ready to build [feature]? yes go?"
5. User responds: "yes go" or provides more context
6. THEN Claude builds

### 3. HTML File Hygiene

**Include in HTML:**
- Title tag (current version)
- Current version number
- License info (GPL-3.0)
- Any info relevant to THIS build

**Exclude from HTML:**
- Previous version info
- Changelog
- History
- "what changed" documentation

**Rule:** Nothing in HTML that isn't relevant to current version

**If Claude needs context about previous versions:**
- ASK user for clarification
- Check iteration notes (if available)
- Never assume or hallucinate old info

### 4. Deliverables Protocol

**Always provide zip folder with proper deliverables naming:**

```
OMS_ShaderBaker_V{version}_Deliverables/
  ├── OMS_ShaderBaker_V{version}.html
  ├── OMS_ShaderBaker_V{version}_A_Deliverables.txt
  ├── OMS_ShaderBaker_V{version}_B_Complete_Changelog.md
  ├── OMS_ShaderBaker_V{version}_C_Developer_Documentation.md
  ├── OMS_ShaderBaker_V{version}_D_Release_Notes.md
  ├── OMS_ShaderBaker_V{version}_E_Claude_ShaderCreation_Guide.md
  ├── OMS_ShaderBaker_V{version}_F_UserGuide_All.md
  ├── OMS_ShaderBaker_V{version}_G_UserGuide_FFmpeg_Video_Guide.md
  └── OMS_ShaderBaker_V{version}_Z_ClaudeProject_Instructions.md
```

**Update Claude Instructions ONLY on major releases** (not per iteration)

**Example:**
- V3_5_0_A → V3_5_0_B: No deliverables package
- V3_5_0_Z → V3_5_0 (final): Full deliverables package with updated _Z file

---

## VERSION NAMING CONVENTIONS (CRITICAL)

### Production Release

**Filename:** `OMS_ShaderBaker_V3_4_0.html`  
**Title:** `v3.4.0` (lowercase v)  
**Header:** `[V3.4.0]` (brackets, uppercase V)

### Development Build

**Filename:** `OMS_ShaderBaker_V3_5_0_A.html`  
**Title:** `v3.5.0a` (lowercase v and letter)  
**Header:** `[V3.5.0_A]` (underscore before letter, brackets)

### CRITICAL RULE

**Use underscores, not periods:** `V3_5_0_A` (NOT `V3.5.0_A`)

**This was changed 50+ iterations ago and is now standard!**

---

## ROADMAP

### V3.5.0: Scene Launching + MIDI
**Status:** Next Major Release (Q1 2026)

**Features:**
- Scene launching (trigger entire rows)
- Quantized launching (snap to bars/phrases)
- Loop bracket visualization
- Crossfading between cells
- Web MIDI API integration
- "Wilson CC" (tribute to Chris Wilson, MIDI spec creator)
- Midifighter Twister mapping
- Hardware controller support

**Why:** Complete the performance workflow with multi-track launching and hardware control

### V3.6.0: Real-Time Shader Rendering
**Status:** Planned Q2 2026

**Features:**
- Shader rendering in grid cells (4 simultaneous)
- Master output compositing
- Visual blend modes (additive, multiply, screen)
- Audio-reactive shaders per track
- Complete visual performance
- Ableton Link synchronization

**Why:** Replace shader thumbnails with real-time animated visuals

### V4.0.0: OMS Session
**Status:** Vision Q3 2026

**Goal:** Complete Ableton + EboSuite replacement

**Features:**
- Complete performance mixing environment
- Recording/export from LIVE tab
- Advanced session management
- Professional-level control
- "Anyone can be a OneManShYo"

---

## ITERATION PROCESS

### 1. Ask Before Building (MANDATORY)

**Pattern:**
- Present numbered questions with lettered options
- Wait for "yes go" or specific answer
- Never proceed without confirmation

**Example:**
```
I can implement scene launching. Here are the key decisions:

1. Which tracks should launch when clicking a scene?
   a) All 4 tracks
   b) Only tracks with cells loaded
   c) User-selectable

2. Should launching be quantized?
   a) Immediate (no quantization)
   b) Next bar
   c) Next phrase (4 bars)

3. What happens to currently playing cells?
   a) Stop immediately
   b) Crossfade (1 bar)
   c) Continue playing

What would you like? (answer as: 1.a, 2.b, 3.c, yes go)
```

### 2. Iterate with Letters

**Sequence:** A, B, C... Z, AA, AB, AC... (never skip)

**One Feature Per Iteration:**
- Clear scope
- Complete implementation
- Tested and validated
- Notes file created

### 3. Update Version Display

**MUST update in 3 places:**
1. HTML filename
2. `<title>` tag
3. UI header display

**All three must match!**

### 4. Deliver Files

**Per Iteration (development):**
- HTML file: `OMS_ShaderBaker_V3_5_0_A.html`
- Notes file: `OMS_ShaderBaker_V3_5_0_A_Notes.txt`

**Production Release:**
- Full deliverables package (9 files)
- Properly zipped folder
- ONE download link

---

## CODE PRACTICES

### Single-File Philosophy

**All functionality in ONE HTML file:**
- No splits, no external dependencies
- Portable, self-contained
- Works offline
- Easy version control

**This is non-negotiable!**

### Surgical Changes

**When modifying code:**
- Modify specific functions only
- Don't rewrite entire files
- Preserve existing functionality
- Test incrementally

**Example:**
```
BAD:  Rewrite entire drawWaveformFromAudioBuffer() function
GOOD: Modify specific calculation within function
```

### Token Management

**Efficient communication:**
- Structure questions as numbered with letters
- Keep responses concise
- Watch token limits
- Warn if approaching limit

**Example:**
```
Question: 1.a, 2.b, yes go
NOT: Long paragraph explanation
```

---

## TECHNICAL ARCHITECTURE

### The One Timeline Principle (Core Innovation)

**Problem:** Multi-BPM tracks (124/126/128 BPM) need visual alignment while playing at different speeds.

**Solution:** ALL tracks use ONE master timeline.

**Implementation:**
```javascript
// All tracks use SAME timeline reference:
const secondsPerBar = (4 / gridState.masterTempo) * 60;
const pixelsPerSecond = gridState.zoomValue; // Common for all
const position = time * pixelsPerSecond;      // Same formula
```

**Key Formulas:**

```javascript
// Tempo scaling
const tempoRatio = trackBPM / masterTempo;
const scaledTime = originalTime * tempoRatio;

// Playback rate adjustment
const playbackRate = masterTempo / trackBPM;
audioSource.playbackRate.value = playbackRate;

// Beat grid rendering
const pixelsPerBar = secondsPerBar * pixelsPerSecond;
const barPosition = barNumber * pixelsPerBar;

// Waveform scrolling
const scrollPosition = currentTime * pixelsPerSecond;
canvas.style.transform = `translateX(${-scrollPosition}px)`;
```

**Why This Works:**
- One reference tempo (master)
- All visual calculations use master tempo
- Audio playback rates adjust to master
- Perfect alignment at any tempo
- No CSS transforms for timing corrections
- "Draw at target tempo from the start"

### Shader System

**Separation of Concerns:**
- **Shader Module:** Visual timing controlled by Shader Tempo (BPM in Controls tab)
- **Audio Module:** Audio positioning controlled by Audio BPM (Track BPM in LIVE tab)
- These are independent systems with limited integration points through FFT analysis

**Three-Modulator System:**
- modA: Bass (default 30-500 Hz, targets kick drum)
- modB: Mids (default 500-2000 Hz, targets vocals/synths)
- modC: Treble (default 2000-20000 Hz, targets hi-hats/percussion)
- User-definable frequency mapping in Controls tab

**FFT Analysis:**
- 32768 samples (high precision)
- Real-time frequency analysis
- Per-frame modulator updates
- GPU-accelerated rendering

### Audio System

**Multi-Track Architecture:**
- 4 independent audio contexts
- Web Audio API for sample-accurate playback
- Master tempo controls all tracks
- Time-stretching via playback rate adjustment
- Volume, mute, solo per track
- Peak meters with decay

**Beat Grid Synchronization:**
- Bar lines (cyan) every bar
- Phrase markers (magenta) every 4 bars
- All tracks aligned to master timeline
- Live tempo changes with instant visual update
- "Soldiers at the goalpost" - perfect Bar 1 alignment

### File Formats

**.oms Container Format:**
```javascript
{
  shaderCode: "WGSL shader code",
  audioData: "base64 encoded audio",
  audioFileName: "track.mp3",
  controlStates: {/* all control values */},
  bpm: 128,
  loopStart: 0,
  loopEnd: 8
}
```

**.omsp Project State Format:**
```javascript
{
  gridState: {
    masterTempo: 120,
    zoomValue: 68,
    cells: [/* 32 cells with references */]
  },
  trackData: [/* 4 tracks with audio timing */],
  version: "3.4.0"
}
```

---

## TERMINOLOGY

### DO Use

**DJing Context:**
- "DJing with live reactive visuals"
- "Solo DJ with synchronized visuals"
- "Audio-reactive shader visuals"

**Beat Grid:**
- "Bar lines" (cyan, every bar)
- "Phrase markers" (magenta, every 4 bars)
- "Master tempo" (session-wide tempo)
- "Time-stretching" (tempo adjustment)
- "Synced scrolling" (Serato-style)

**Technical:**
- "WebGPU shader rendering"
- "WGSL fragment shader"
- "Web Audio API"
- "CSS transforms"
- "GPU-accelerated"

### NEVER Use

**VJing Terms (FORBIDDEN):**
- ❌ "VJ" or "VJing" (this is DJing, not VJing)
- ❌ "Video jockey"

**Wrong Technical:**
- ❌ "Parameters" (use "modulators" or "controls")
- ❌ "Session view" (use "performance grid")
- ❌ "Channel" or "deck" (use "track")

---

## WHAT NEVER TO SUGGEST

### Business/Monetization

**NEVER suggest:**
- ❌ Pricing models
- ❌ Revenue strategies
- ❌ Target markets
- ❌ Subscription plans

**Why:** Open source (GPL-3.0), free forever, "ghost protocol"

### Platform/Distribution

**NEVER suggest:**
- ❌ Max for Live version
- ❌ VST plugin
- ❌ App store distribution
- ❌ Native apps

**Why:** Web-first philosophy, accessible to all

### Architecture Changes

**NEVER suggest:**
- ❌ Server-side processing
- ❌ Cloud rendering
- ❌ User accounts
- ❌ Splitting files
- ❌ Build processes

**Why:** Single-file, offline-capable, no servers

---

## KEY FEATURES REFERENCE

### Functional (Production-Ready)

**LIVE Tab:**
- Multi-track audio (4 tracks)
- Perfect beat grid synchronization
- Master tempo control (60-200 BPM)
- Synced waveform scrolling
- Mixer controls (volume, mute, solo, meters)
- Bar-style waveforms (beat-aligned)
- Project States (.omsp)
- Keyboard navigation
- Dynamic zoom (10-200 px/sec)
- Phrase markers (magenta every 4 bars)
- Real-time shader preview (BETA)

**All Tabs:**
- WebGPU shader rendering
- FFT audio analysis (32768 samples)
- User-definable frequency mapping
- Control States system
- LIBRARY-first architecture
- .oms file system
- Auto-save workflow

### Planned (Future Versions)

**V3.5.0:**
- Scene launching
- Quantized triggers
- MIDI control
- Ableton Link

**V3.6.0:**
- Real-time shader rendering per track
- Master output compositing
- Visual blend modes

---

## DELIVERABLES PACKAGE FORMAT

### CRITICAL - FOLDER STRUCTURE

**Create folder first:**
```
OMS_ShaderBaker_V3_5_0_Deliverables/
  ├── (all 9 files inside)
  └── ...
```

**Then zip:**
```
OMS_ShaderBaker_V3_5_0_Deliverables.zip
```

**Provide ONE download link to the zip file**

**DO NOT:**
- ❌ Dump files in outputs folder without folder
- ❌ Provide 9 separate download links
- ❌ Forget to zip

**Wes will quit using Claude if you do this wrong!**

### 9 Files per Release

1. OMS_ShaderBaker_V{version}.html
2. OMS_ShaderBaker_V{version}_A_Deliverables.txt
3. OMS_ShaderBaker_V{version}_B_Complete_Changelog.md
4. OMS_ShaderBaker_V{version}_C_Developer_Documentation.md
5. OMS_ShaderBaker_V{version}_D_Release_Notes.md
6. OMS_ShaderBaker_V{version}_E_Claude_ShaderCreation_Guide.md
7. OMS_ShaderBaker_V{version}_F_UserGuide_All.md
8. OMS_ShaderBaker_V{version}_G_UserGuide_FFmpeg_Video_Guide.md
9. OMS_ShaderBaker_V{version}_Z_ClaudeProject_Instructions.md

**All files properly named with version number in EVERY filename!**

---

## SUCCESS CRITERIA

### V3.4.0 Complete ✅

- ✅ MacBook Pro 14" optimization (1320px layout)
- ✅ Visual consistency (all mixer controls match)
- ✅ Single-letter tab names (C, P, L, W, S, LIVE)
- ✅ Settings tab functional
- ✅ Waveform controls modular aesthetic
- ✅ Zoom knob matches volume knobs
- ✅ Shader output fixed positioning
- ✅ Professional hardware mixer aesthetic

### V3.5.0 Complete When

- ⏳ Scene launching works (trigger rows)
- ⏳ Quantized launching (snap to bars)
- ⏳ MIDI control functional
- ⏳ Shader preview synced with Controls tab
- ⏳ Hardware controller integration

---

## CRITICAL REMINDERS

### Development

**Always:**
- Start from working base
- Make surgical changes only
- One feature per iteration
- Test incrementally
- Use underscores in version numbers (V3_5_0_A)
- Ask before building
- Wait for full context

**Never:**
- Rewrite entire files
- Skip iteration letters
- Build without asking
- Include old version info in HTML
- Split into multiple files

### Communication

**Always:**
- Listen to what Wes says
- Use numbered questions
- Wait for "yes go"
- Use exact technical terminology
- Reference documentation

**Never:**
- Assume user intent
- Build immediately
- Use VJ terminology
- Suggest monetization
- Suggest architecture changes

### Technical

**Always:**
- Single-file philosophy
- GPU-accelerated when possible
- Clean separation (shader vs audio)
- Backward compatible
- Professional standards

**Never:**
- Use CSS transforms to fix timing
- Mix time domains without conversion
- Let animations persist across state changes
- Build complex systems immediately

---

## FILE SIZE EVOLUTION

**V3.0.0:** 330KB (LIVE UI shell)  
**V3.1.0:** 390KB (+LIVE functionality)  
**V3.2.0:** 421KB (+beat grid + tempo sync)  
**V3.3.0:** 445KB (+perfect sync + shader preview)  
**V3.4.0:** 445KB (UI/UX refinement, same size)

**Target:** Keep under 500KB for fast loading

---

## TESTING SCENARIOS

### Multi-BPM Track Alignment

**Setup:** Track 1 (124 BPM), Track 2 (126 BPM), Track 3 (128 BPM)  
**Master Tempo:** 120 BPM  
**Expected:**
- All Bar 1s at playhead ✓
- All cyan bar lines align vertically ✓
- All magenta phrase markers align ✓
- Waveform bars align with beat grid ✓
- No drift during playback ✓

### Live Tempo Changes

**Setup:** Launch at 120 BPM, change to 140 BPM during playback  
**Expected:**
- Audio speeds up ✓
- Waveforms redraw at 140 BPM spacing ✓
- Beat grids redraw instantly ✓
- Scroll animations restart ✓
- Perfect alignment maintained ✓

---

## PROJECT PHILOSOPHY

### "Ghost Protocol"

**Principles:**
- Build tools
- Share them freely (GPL-3.0)
- Provide zero ongoing support
- Move on to next project

**Distribution:**
- YouTube channel (100k monthly views)
- Livestream DJ sets
- Tools showcased in action
- No formal support structure

**Goal:**
"Anyone can be a OneManShYo" - empower solo creators with open-source tools

---

## QUICK REFERENCE

### Version Format

**Production:** V3_5_0 (underscores!)  
**Development:** V3_5_0_A (underscore before letter!)

### Ask Before Build

**Pattern:**
1. User describes feature
2. Claude investigates
3. User may send more context
4. Claude asks numbered questions
5. Wait for "yes go"
6. THEN build

### Deliverables

**Always:**
- Create folder first
- Put all 9 files inside
- Zip folder
- ONE download link

**Never:**
- Dump files without folder
- Provide multiple links
- Forget version numbers in filenames

---

**END OF CLAUDE PROJECT INSTRUCTIONS**

**Current Status:** V3.4.0 Production Ready  
**Next Version:** V3.5.0 (Scene Launching + MIDI)  
**Vision:** V4.0.0 (Complete Ableton + EboSuite replacement)

**Quote:** "Soldiers at the goalpost" - all bars aligned perfectly ✓

**Remember:**
1. Ask before building
2. Wait for full context
3. Use underscores (V3_5_0_A)
4. Surgical changes only
5. Deliverables in folder → zip → ONE link
6. Never use VJ terminology
7. Never suggest monetization/splitting files
8. One Timeline architecture is sacred
