# OMS SHADER BAKER V3.3.0 - CLAUDE PROJECT INSTRUCTIONS

**Version:** V3.3.0
**Date:** November 2025
**Purpose:** Complete project context for Claude AI development

---

## CURRENT STATE: V3.3.0

### Production Status

**LIVE Tab - Production Ready:**
- ✅ Perfect beat grid synchronization (53 iterations)
- ✅ Multi-BPM track alignment (124/126/128 BPM all sync)
- ✅ Master tempo control (60-200 BPM)
- ✅ Time-stretching all tracks to master tempo
- ✅ Synced waveform scrolling (Serato-style)
- ✅ Complete audio timing data storage
- ✅ Bar-style waveforms (mixing-optimized)
- ✅ Multi-track audio playback (4 tracks)
- ✅ Mixer controls (volume, mute, solo, meters)
- ✅ Project States persistence (.omsp)
- ✅ Keyboard navigation
- ✅ Beat-aligned waveform bars (1 per beat)
- ✅ Dynamic zoom range (10-200 px/sec)
- ✅ Phrase markers (magenta every 4 bars)
- ✅ Real-time shader preview (BETA)

**What Doesn't Work Yet:**
- ❌ Scene launching (rows)
- ❌ Shader rendering in cells (thumbnails only)
- ❌ Master output compositing
- ❌ Waveform control buttons (visual only)
- ❌ MIDI control
- ❌ Ableton Link

**File:** OMS_ShaderBaker_V3_3_0.html (10,219 lines, 445KB)

---

## V3.3.0 DEVELOPMENT SUMMARY

### 53 Iterations Total

**The Core Achievement:**
"Soldiers at the goalpost" - perfect beat grid alignment regardless of track BPM or master tempo setting. Professional DJ timing accuracy achieved through the "One Timeline" architecture.

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

**Example:** V3_3_0_A → V3_3_0_B → V3_3_0_C (never A → C)

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
- V3_3_0_A → V3_3_0_B: No deliverables package
- V3_3_0_Z → V3_3_0 (final): Full deliverables package with updated Z file

---

## ROADMAP

### V3.4.0: Scene Launching + MIDI
**Status:** Planned Q1 2026

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

### V3.5.0: Real-Time Shader Rendering
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

## VERSION NAMING CONVENTIONS (CRITICAL)

### Production Release

**Filename:** `OMS_ShaderBaker_V3_3_0.html`  
**Title:** `v3.3.0` (lowercase v)  
**Header:** `[V3.3.0]` (brackets, uppercase V)

### Development Build

**Filename:** `OMS_ShaderBaker_V3_3_0_A.html`  
**Title:** `v3.3.0a` (lowercase v and letter)  
**Header:** `[V3.3.0_A]` (underscore before letter, brackets)

### CRITICAL RULE

**Use underscores, not periods:** `V3_3_0_A` (NOT `V3.3.0_A`)

**This was changed 50+ iterations ago and is now standard!**

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
- HTML file: `OMS_ShaderBaker_V3_3_0_A.html`
- Notes file: `OMS_ShaderBaker_V3_3_0_A_Notes.txt`

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

// Playback rate
playbackRate = masterTempo / trackBPM;

// Visual scale (NOT USED - render at master tempo directly)
// scaleX = trackBPM / masterTempo; // OLD WAY - removed!

// Scroll speed
const scrollOffset = realTimeElapsed * pixelsPerSecond; // NO playbackRate!
```

**Critical Insight:** Don't use CSS transforms (scaleX) to fix timing. Render everything at master tempo from the start.

### Beat Grid System

**Components:**
- trackContainerData[] - Audio timing per track
- drawBeatGridOnWaveform() - Canvas rendering at master tempo
- Master tempo sync - Time-stretching engine
- Synced scrolling - RequestAnimationFrame loop

**Visual Elements:**
- Cyan bars: Every musical bar (4 beats)
- Magenta markers: Every 4 bars (phrase markers)
- Bar numbers: 1, 5, 9, 13... (aligned with magenta)
- Waveform bars: 1 per beat (beat-aligned)

### Audio Data Storage

**Extended .oms Format:**
```json
{
  "shaderCode": "...",
  "controlStates": {
    "shader*": "...",
    "audio*": "..."
  },
  "audioBPM": 128.0,
  "audioPitch": 2.5,
  "audioStartTime": 0.0,
  "audioEndTime": 180.5,
  "audioLoopIn": 64.0,
  "audioLoopOut": 128.0
}
```

**Clean Separation:**
- Shader controls: shader* prefix
- Audio controls: audio* prefix
- Render settings: no prefix

---

## CRITICAL LESSONS

### From V3.3.0 Development

**The 5 Core Insights:**

1. **Don't Fix with Transforms**
   - BAD: Draw at original tempo, use scaleX to stretch
   - GOOD: Draw at target tempo from the start

2. **One Timeline to Rule Them All**
   - BAD: Per-track timelines with different scales
   - GOOD: Single master timeline, all tracks reference it

3. **Time Domains Matter**
   - BAD: Mix original and scaled time values
   - GOOD: Convert explicitly between domains
   - Sample reading: original time
   - Visual drawing: scaled time

4. **Animation Lifecycle**
   - BAD: Let old animations persist when parameters change
   - GOOD: Restart animations when state changes

5. **Start Simple, Iterate Complex**
   - BAD: Try to build complete system immediately
   - GOOD: Walking skeleton first, then enhance

**The Quote:**
"you do this every time mate..." - Make surgical changes, don't rewrite entire UI

---

## COMMUNICATION

### Wes's Patterns

**Style:**
- Direct and efficient
- Numbered questions with lettered options
- "yes go" means proceed
- Corrects imprecise terminology immediately
- Expects exact technical language

**Decision Making:**
- Answers efficiently ("1.a, 2.b, yes go")
- Makes technical decisions quickly
- Knows what he wants
- Often multi-messages context

**Workflow:**
- Describes issue
- May send 2-3 messages elaborating
- Claude waits for full context
- Claude asks confirmation
- "yes go" triggers implementation

### Claude's Response Pattern

**Investigation Phase:**
- Research the issue
- Analyze code
- Discuss options
- NO building yet

**Ask Phase:**
- Numbered questions with lettered options
- One concept per question
- Clear and distinct options
- Wait for answer

**Implementation Phase:**
- One feature per iteration
- Complete context in notes
- Reference documentation
- Use exact technical language

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

**V3.4.0:**
- Scene launching
- Quantized triggers
- MIDI control
- Ableton Link

**V3.5.0:**
- Real-time shader rendering per track
- Master output compositing
- Visual blend modes

---

## DELIVERABLES PACKAGE FORMAT

### CRITICAL - FOLDER STRUCTURE

**Create folder first:**
```
OMS_ShaderBaker_V3_3_0_Deliverables/
  ├── (all 9 files inside)
  └── ...
```

**Then zip:**
```
OMS_ShaderBaker_V3_3_0_Deliverables.zip
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

### V3.3.0 Complete ✅

- ✅ Beat grid renders on waveforms
- ✅ Master tempo controls all tracks
- ✅ Waveforms scroll together perfectly
- ✅ Audio timing data saves/loads
- ✅ Multi-BPM tracks align at any tempo
- ✅ Live tempo changes with zero drift
- ✅ Professional visual timing reference
- ✅ "Soldiers at the goalpost" achieved

### V3.4.0 Complete When

- ⏳ Scene launching works (trigger rows)
- ⏳ Quantized launching (snap to bars)
- ⏳ MIDI control functional
- ⏳ Shader preview synced with Controls tab

---

## CRITICAL REMINDERS

### Development

**Always:**
- Start from working base
- Make surgical changes only
- One feature per iteration
- Test incrementally
- Use underscores in version numbers (V3_3_0_A)
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

**Production:** V3_3_0 (underscores!)  
**Development:** V3_3_0_A (underscore before letter!)

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

**Current Status:** V3.3.0 Production Ready  
**Next Version:** V3.4.0 (Scene Launching + MIDI)  
**Vision:** V4.0.0 (Complete Ableton + EboSuite replacement)

**Quote:** "Soldiers at the goalpost" - all bars aligned perfectly ✓

**Remember:**
1. Ask before building
2. Wait for full context
3. Use underscores (V3_3_0_A)
4. Surgical changes only
5. Deliverables in folder → zip → ONE link
