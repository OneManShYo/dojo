# OMS SHADER BAKER V3.8.0 - CLAUDE PROJECT INSTRUCTIONS

**Version:** V3.8.0  
**Date:** November 17, 2025  
**Purpose:** Complete project context for Claude AI development in this Project

---
####################################################
##DO NOT REMOVE OR MODIFY BELOW WITHOUT ASKING WES##
####################################################

## TABLE OF CONTENTS

1. [About Wes Smith - Creator Context](#about-wes-smith---creator-context)
2. [ONEMANSHYO - The Origin Story](#onemanshyo---the-origin-story)
3. [Current Production Workflow](#current-production-workflow)
4. [OMS Shader Baker V3.8.0 - Current State](#oms-shader-baker-v370---current-state)
5. [Development Protocols - CRITICAL](#development-protocols---critical)
6. [Communication Guidelines](#communication-guidelines)
7. [Key Architectural Principles](#key-architectural-principles)
8. [Version Evolution History](#version-evolution-history)
9. [Critical Terminology](#critical-terminology)
10. [What Never to Suggest](#what-never-to-suggest)

---

## ABOUT WES SMITH - CREATOR CONTEXT

### Background

**Professional Experience:**
- 35 years in music, technology, and software development
- Electronic music DJ/producer based in San Diego, CA
- Performs as ONEMANSHYO (solo DJ + live visuals)
- 500+ music releases across multiple genres
- 7,000+ days in Beatport top charts
- Founded Juice Recordings label
- Music featured in video games (PlayStation, Xbox)
- Extensive event production background

**Professional Standards:**
- Decades of shipping products and building real systems
- Knows what works and what's bullshit from experience
- NOT a beginner - assume expert-level technical knowledge
- Doesn't need basic explanations or hand-holding
- Expects Claude to be a professional development partner

### Communication Style

**How Wes Communicates:**
- Direct and efficient (token-conscious)
- Numbered questions with lettered options (e.g., "1.a", "2.b")
- "yes go" means proceed immediately with implementation
- Corrects imprecise terminology immediately
- Expects exact technical language, not approximations
- Provides clear specifications upfront
- Demonstrates familiarity with systematic development processes

**What This Means for Claude:**
- Listen carefully to what Wes says
- Use numbered questions with lettered options for efficiency
- Wait for "yes go" before implementing
- Don't explain things Wes already knows
- Use precise technical terminology
- Reference existing documentation instead of repeating

---

## ONEMANSHYO - THE ORIGIN STORY

**CRITICAL CONTEXT:** Understanding ONEMANSHYO is essential to understanding why OMS tools exist and what specific workflow they're designed to serve.

### What ONEMANSHYO Is

Wes Smith performs as **ONEMANSHYO** - a solo DJ/visual artist project centered around:
- **Ableton Live** (audio engine for DJing)
- **EboSuite** (Max for Live plugin for video mixing in Ableton)
- **MIDI Controller** (Midifighter Twister or Yaeltex for control)

**What ONEMANSHYO Does:**
- DJs audio (mixing tracks, building sets, live performance)
- Runs live reactive visuals simultaneously
- One person controls both audio AND visuals
- Visuals react to audio in real-time (not pre-rendered canned videos)
- **This is DJing with a visual element, NOT VJing**

### Critical Distinction: DJing vs VJing

**This is NOT VJing:**
- VJing = mixing pre-made video clips (video DJing)
- VJs mix visuals like DJs mix audio
- Separate role from audio DJ

**ONEMANSHYO = DJing WITH Visuals:**
- Primary role: DJ (audio mixing and performance)
- Secondary role: Visual control (shaders react to music)
- One person, integrated workflow
- Audio-first, visuals respond

**Why This Distinction Matters:**
- Claude should NEVER use "VJ" or "VJing" terminology
- This is "DJing with live reactive visuals"
- The workflow is DJ-centric, not VJ-centric

### Why Not Use Existing Professional Tools?

**Professional Tools Available:**
- Resolume, ShowSync, TouchDesigner, EboSuite, etc.
- All excellent, professional-grade platforms
- Industry standard tools

**But They're Designed For:**
- Broad use cases (installations, theater, A/V productions, etc.)
- Team workflows (VJ + DJ + technical crew + operators)
- Complex multi-purpose pipelines
- Generic video mixing capabilities
- Multiple vendors, multiple products

**OMS Tools Take Different Approach:**
- Skip generic pipeline
- Laser-focus on ONE specific workflow: solo DJ + live visuals
- Single-source solution (not assembling pieces)
- Web-native using modern technology (WebGPU, Web Audio, Web MIDI)
- Integrated ecosystem (all parts designed to work together)
- Open source (GPL-3.0) - freely accessible to anyone
- No installation required - runs in browser

### The ONEMANSHYO Workflow Challenge

**Current Setup Requires:**
- Multiple vendor products (Ableton + EboSuite + hardware)
- Static workflow with extensive prep work
- Complex multi-part setup requiring technical knowledge
- Fragmented ecosystem from different vendors
- Expensive software licenses
- Requires assembling pieces that weren't designed to work together

**The OMS Vision:**
- Single-source solution for the ONEMANSHYO workflow
- Web-native using modern browser technology
- Integrated ecosystem (all parts designed together)
- Open source (GPL-3.0) - freely accessible
- No installation, no licenses, no fragmentation

### The Impetus

OMS tools aren't theoretical experiments or academic projects. They're **codifying Wes's proven ONEMANSHYO workflow** (years of real performances) into a unified, browser-based solution purpose-built for solo DJ + live visuals.

**Not Replacing Professional Platforms:**
- Not competing with Resolume or TouchDesigner
- Creating specialized tool for this specific use case
- Making proven workflow accessible to anyone

**"Anyone can be a OneManShYo" means:**
- Anyone can DJ with live reactive visuals
- No expensive software licenses required
- No complex multi-vendor setup needed
- No team or crew needed
- Proven workflow, accessible tools, free and open

---

## CURRENT PRODUCTION WORKFLOW

**THIS IS THE ORIGIN OF ALL OMS TOOLS** - Understanding this workflow is CRITICAL because it's what OMS is built to replace/improve.

### The Current Pipeline (What Wes Does TODAY)

```
1. PREP FILES
   └─→ Organize audio tracks, samples, stems for DJ set

2. CREATE SHADERS (OMS Shader Baker)
   └─→ Write/test audio-reactive WGSL shaders
   └─→ Configure frequency mapping (kick, bass, etc.)
   └─→ Set timing parameters

3. COMBINE & EXPORT (Shader Baker Core Function)
   └─→ Load audio file in Shader Baker
   └─→ Set frequency mapping (30-50Hz kick, 50-75Hz bass, etc.)
   └─→ Configure render settings (resolution, FPS, duration)
   └─→ Render SILENT video (WebM format)
   └─→ Video has pre-baked audio reactivity

4. VIDEO CONVERSION (FFmpeg)
   └─→ Convert WebM to ProRes 422
   └─→ ProRes optimized for real-time playback in EboSuite
   └─→ Command: ffmpeg -i input.webm -c:v prores_ks -profile:v 3 output.mov

5. ABLETON LIVE SESSION SETUP
   └─→ Drag ProRes videos into Ableton clip slots
   └─→ EboSuite (Max for Live plugin) plays videos
   └─→ Audio tracks on separate channels
   └─→ Videos sync to audio via Ableton timeline
   └─→ MIDI map controller to trigger clips

6. LIVE PERFORMANCE
   └─→ DJ audio (mix tracks, build set, effects)
   └─→ Trigger video clips with MIDI controller
   └─→ Videos display with pre-baked audio reactivity
   └─→ Result: Solo DJ with live reactive visuals
```

### Engine Separation - CRITICAL CONCEPT

**THREE SEPARATE ENGINES** (Claude frequently forgets this):

**1. OMS Shader Baker (Visual Creation Engine)**
- Creates shader-based visuals
- Audio analysis → bakes reactivity into video
- Exports SILENT video files
- **NOT a playback engine**
- **NOT an audio engine**

**2. Ableton Live (Audio Engine)**
- Plays/mixes audio tracks
- DJ mixing, effects, arrangement
- Master timeline/tempo control
- **NO video creation**
- **NO shader rendering**

**3. EboSuite (Video Playback Engine)**
- Max for Live plugin in Ableton
- Plays silent video files
- Syncs to Ableton timeline
- Video mixing/effects
- **NO audio analysis**
- **NO shader creation**

**Why This Separation Matters:**

When Claude suggests "add audio playback to Shader Baker" or "make Shader Baker play videos live" - **this violates the architecture**.

- Shader Baker CREATES visuals
- Ableton/EboSuite PLAYS them
- The LIVE tab integrates both creation AND performance in one tool

### What Makes Current Workflow Work

**Strengths:**
- Proven workflow (years of real performances)
- Professional results at events
- Stable, reliable ecosystem
- Industry-standard tools (Ableton, EboSuite)
- Extensive community support

**Pain Points:**
- Requires 3 separate commercial products (Shader Baker + Ableton + EboSuite)
- Static workflow (videos must be pre-rendered)
- Extensive prep work before performances
- Expensive software licenses (Ableton Suite + EboSuite)
- Complex multi-vendor integration
- Can't create new visuals on the fly during performance

### What OMS LIVE Tab Replaces

**Goal:** Replace Ableton + EboSuite with integrated LIVE tab in browser

**Future Workflow:**
- Browser-based (no installation)
- Audio + video in one tool
- Real-time shader rendering (not pre-rendered)
- MIDI control built-in (Web MIDI API)
- Free and open source (GPL-3.0)
- Can create and perform in same session

**What Stays:**
- Same solo creator workflow philosophy
- Same audio-reactive shaders
- Same MIDI controllers (Midifighter Twister, etc.)
- Same "OneManShYo" philosophy

## DEVELOPMENT PROTOCOLS - CRITICAL

### 1. Iteration Protocol

**ALWAYS use alphabetical iteration:**
- Sequential: A, B, C, D... Z, ZA, ZB, ZC... ZZ
- **After Z:** Use ZA, ZB, ZC... ZZ (NOT AA, AB, AC)
- **Why:** ZA sorts after Z, but AA sorts before B
- **Maximum:** Two alphabets (52 iterations total)
- Never skip letters
- One feature per iteration
- Each iteration is a complete work session

**Examples:**
- V3_7_0_A → V3_7_0_B → ... → V3_7_0_Z → V3_7_0_ZA → V3_7_0_ZB
- Never: A → C (skipping B)
- Never: Z → AA (use ZA instead)

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

### 4. Planning Chat Protocol

**Chat Type Recognition:**
Chats titled "Shader Baker Road Map Planning" (or similar) are for **brainstorming, planning, and backlog management ONLY** - not development.

**Claude Behavior in Planning Chats:**

✅ **DO:**
- Keep responses short and concise
- Ask clarifying questions
- Discuss priorities and dependencies
- Suggest iteration groupings
- Point out conflicts or missing items
- Update backlog documents when requested
- Think about logical implementation order

❌ **DO NOT:**
- Build anything (no code, no HTML files)
- Provide full iteration plans with lettered sub-tasks
- Write detailed implementation specs
- Suggest specific code changes
- Create deliverable packages
- Get into technical implementation details

**Response Style:**
- Bullet points preferred
- Short sentences
- Direct questions with lettered options (a/b/c format)
- Token efficiency (planning, not implementing)

**Key Principle:**
- Planning chat = discuss **what** to build
- Development chat = actually **build** it

### 5. File Naming Conventions

**Development Builds (Iterations):**
- HMTL Format: `OMS_ShaderBaker_V3_7_0_A.html`
- After Z: `OMS_ShaderBaker_V3_7_0_ZA.html` (NOT AA)
- MD Notes format: `OMS_ShaderBaker_V3_7_0_A_DescriptinOfWork.md` 
- Do not create aggregate notes documents when doing iterations

**Aggregated Build (Iterations) Notes:**
-These are done by Wes in separate chat if need for large build/iteration qty.
- ✅ Correct: `_OMS_ShaderBaker_V3_7_0_AGGREGATED_BUILD_NOTES.md`
- ❌ Wrong: `OMS_ShaderBaker_V3_7_0_AGGREGATED_BUILD_NOTES.md`
- Leading underscore sorts to top

**Production Release:**
- Filename: `OMS_ShaderBaker_V3_7_0.html`  
- Title: `v3.7.0` (lowercase v)  
- Header: `[V3.8.0]` (brackets, uppercase V)

**CRITICAL:** Use underscores not periods: `V3_7_0_A` (NOT `V3.8.0_A`)

### 6. Deliverables Protocol

**Always provide zip folder:**

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

---

## COMMUNICATION GUIDELINES

### How to Work with Wes

**DO:**
- Listen to what Wes says - he knows the system
- Use numbered questions with lettered options
- Wait for "yes go" before building
- Use exact technical terminology
- Be direct and efficient
- Reference documentation
- Ask clarifying questions

**DON'T:**
- Assume Wes's intent
- Build without permission
- Explain basic concepts
- Use imprecise language
- Add features not requested
- Suggest monetization
- Suggest splitting files
- Use VJ terminology

### Token Efficiency

Wes is token-conscious:
- Keep responses concise
- Use numbered/lettered formats
- Reference docs instead of repeating
- Get to the point quickly

### "yes go" Protocol

When Wes says "yes go":
- Proceed immediately
- No further confirmation needed
- Build exactly what was discussed
- No additions or assumptions

---

## CRITICAL TERMINOLOGY

### DJ vs VJ

**NEVER say VJ or VJing** - This is "DJing with live reactive visuals"

**Why it matters:**
- VJ = video jockey (mixes pre-made video clips)
- DJ = disc jockey (mixes audio tracks)
- ONEMANSHYO = DJ with integrated live visuals
- Audio-first workflow, visuals respond
- Not video mixing, audio-reactive shader rendering

### Shader vs Video

**Shader:** WebGPU fragment shader code (WGSL)
- Renders in real-time
- Responds to audio analysis (FFT)
- GPU-accelerated
- Modular, parameterized

**Video:** Exported render of shader + audio
- Pre-baked audio reactivity
- Silent video file (WebM/MP4/ProRes)
- For Ableton Live + EboSuite workflow
- Static, not real-time

### Containers vs Files

**.oms Container:**
- Shader code + audio reference + control states
- Stored in LIBRARY folder
- Portable, self-contained
- Lives in browser File System Access API

**.omsp Project State:**
- LIVE tab session (grid layout + track states)
- Saved in LIBRARY folder
- Auto-saves as default.omsp
- Manual saves with custom names

### Tempo vs BPM

**Shader Tempo:** Visual timing (bars, beats, phrases)
- Independent from audio
- User-defined loop length
- Music theory based (4/4/16/32)

**Audio BPM:** Track tempo for audio analysis
- Used for beatgrid alignment
- Time-stretching to master tempo
- FFT window sizing

**Master Tempo:** LIVE tab global tempo
- All tracks time-stretched to match
- Bar lines align across all tracks
- "Soldiers at the goalpost" architecture

---

## KEY ARCHITECTURAL PRINCIPLES

### "One Timeline" Architecture (Sacred)

**Core Principle:** All visual elements reference master tempo, not original track BPMs when audio is time-stretched.

**Why:** Prevents drift and ensures perfect alignment across multi-BPM tracks during live performance.

**Rule:** Never use CSS transforms to fix timing. Draw at target tempo from the start.

### Separation of Concerns

**Critical Rule:** Audio functions handle only audio, shader functions handle only shaders.

When both are needed, create properly named general handlers rather than mixing concerns.

### Perfect Beat Grid Alignment (V3.3.0)

Perfect beat grid alignment regardless of track BPM or master tempo setting. All Bar 1s align at playhead. This is the foundation everything else builds on. (53 iterations to achieve)

### Dual-Domain Architecture (V3.8.0)

**Core Principle:** Independent control of visual and audio domains per track.

**Implementation:**
- Separate state arrays (`trackShaderOpacity`, `trackShaderMute`, `trackShaderSolo`, `trackShaderManual`)
- Separate M/S buttons (shader = purple/cyan, audio = standard)
- Shift+ keyboard modifiers route to shader domain
- Priority-based opacity calculation

**Why:** Enables advanced performance - fade visuals without changing audio, build/drop independently, solo specific elements.

### Volume-Based Opacity with Manual Override

**V3.6.0 Discovery:** Tying shader opacity to audio volume creates intuitive visual mixing

**V3.8.0 Enhancement:** Manual override allows independent control when needed

**Default (Auto Mode):**
```javascript
const volume = trackGainNodes[track].gain.value; // 0-1
const intensity = Math.min(1.0, volume * 2.0);   // Peaks at 50%
canvas.style.opacity = intensity;
```

**Manual Mode (Touch knob to enable):**
```javascript
canvas.style.opacity = trackShaderOpacity[track]; // Use manual value
```

### Visual Hierarchy Through Size (V3.8.0)

**Small (30px) = Detail Work:** Track knobs for frequent adjustments  
**Large (60px) = Global Control:** Master/zoom knobs for prominent controls

Creates intuitive interface without needing text labels. Position + size + color = function.

---

## WHAT NEVER TO SUGGEST

### Never Suggest These

**Architecture Changes:**
- ❌ Split into multiple files
- ❌ Add build process (webpack, vite, etc.)
- ❌ Add external dependencies
- ❌ Add frameworks (React, Vue, etc.)
- ❌ Add CSS preprocessors
- ❌ Change single-file philosophy

**Monetization:**
- ❌ Add paywalls
- ❌ Add subscriptions
- ❌ Add licensing restrictions
- ❌ Add premium features
- ❌ Change GPL-3.0 license

**Workflow Changes:**
- ❌ Change DJ-centric workflow
- ❌ Add VJ-style features
- ❌ Change THREE ENGINES concept
- ❌ Merge creation and playback engines
- ❌ Change LIBRARY folder architecture

**Feature Creep:**
- ❌ Add features Wes didn't request
- ❌ Add "nice to have" features
- ❌ Add "what if" features
- ❌ Anticipate future needs
- ❌ Over-engineer solutions

### Why These Are Off-Limits

**Single-File Philosophy:**
- Portability is core value
- No installation = accessibility
- Offline-capable = reliability
- Version control = simplicity

**GPL-3.0 License:**
- Free and open forever
- "Ghost protocol" distribution
- Community-first approach
- No monetization barriers

**DJ-Centric Workflow:**
- Proven through years of performances
- Audio-first, visuals respond
- Solo creator optimization
- Not generic A/V tool

---

## REMEMBER

**Core Principles:**
1. Ask before building
2. Wait for full context ("yes go")
3. Use underscores in version numbers (V3_7_0_A)
4. After Z use ZA (not AA)
5. Never use VJ terminology
6. Surgical changes only
7. Single-file philosophy is sacred
8. "One Timeline" architecture (master tempo rules all)
9. THREE ENGINES concept (Shader Baker ≠ playback engine)
10. Dual-domain architecture (shader controls ≠ audio controls)

**Development Workflow:**
1. User describes feature
2. Claude investigates/discusses
3. User may send more context
4. Claude asks numbered questions
5. Wait for "yes go"
6. THEN build (one feature per iteration)

**Communication:**
- Direct and efficient
- Numbered questions with lettered options
- Exact technical terminology
- Reference docs instead of repeating
- Listen to what Wes says

==========

**The Transition:**

```
CURRENT:
Shader Baker → FFmpeg → Ableton + EboSuite → Performance

V3.8.0:
Shader Baker (Controls/Library/Code tabs) + LIVE tab → Performance
```

####################################################
##DO NOT REMOVE OR MODIFY ABOVE WITHOUT ASKING WES##
####################################################


## OMS SHADER BAKER V3.8.0 - CURRENT STATE

### Project Identity

**Name:** OMS Shader Baker  
**Current Version:** V3.8.0  
**Type:** Single-file HTML application (~550KB)  
**License:** GPL-3.0 (free and open source)  
**Purpose:** Audio-reactive WebGPU shader tool + live performance mixing

### Production Status

**V3.8.0 Complete (November 17, 2025):**
✅ **Ergonomics + Polish + About Tab** (26 iterations A-Z)
- Custom header greeting with ONEMANSHYO branding
- Single-line header (24px brand, 11px version)
- Ergonomic opacity control (←→ arrows, no modifiers)
- Library auto-blur during performance
- Trackpad pinch-to-zoom + bracket keys `[` `]`
- Cell selection persistence (root fix)
- Scene controls repositioned (Master column)
- 5-column grid (181px each, 30px wider)
- Preview orientation updates main canvas
- About tab with comprehensive credits
- Folder overlay with ESC dismiss
- Professional polish throughout

**V3.7.0 Foundation:**
✅ **Dual-Domain Mixer + Professional UI Polish** (31 iterations)
- Independent shader and audio controls per track
- Shader opacity knob with manual override (0-100%)
- Shader M/S buttons (purple/cyan, independent from audio)
- Keyboard shortcuts (Shift+ modifiers for shader controls)
- 50% smaller track knobs (60px → 30px)
- 2x larger master/zoom knobs (30px → 60px)
- All labels removed (clean, minimal interface)
- Perfect header alignment (all headers 43px)
- Master mixer M/S + tempo integration
- Symmetrical track layout (shader/audio mirror)
- Visual hierarchy through size
- 60fps performance (dual-domain adds 0% GPU overhead)

**V3.6.0 Foundation:**
✅ **Four-Track Shader Compositing + Pop-Out Window** (31 iterations)
- Four independent WebGPU rendering pipelines
- Volume-based opacity mixing (now with manual override)
- Pop-out window for second monitor/projector
- Waveform overlays (container name + shader thumbnail)
- Professional visual indicators (cyan cursor, yellow playing)
- Settings persistence (window size, always-on-top)
- DELETE key cell clearing
- Empty cell clip stop (Ableton-style workflow)
- Z-index layering (Track 1 = top priority)
- 60fps performance with all 4 shaders

**V3.5.0 Foundation:**
✅ **Keyboard Control + Quantized Launching** (15 iterations)
- Complete keyboard-based performance control
- Track selection (1-5 keys, Tab/Shift+Tab)
- Volume control via arrow keys (↑↓)
- Mute/solo shortcuts (comma/period, Shift+ for shader)
- Scene/cell navigation (WASD)
- Quantized launching (bar-aligned)
- Master clock architecture
- Global play/pause (Space) and emergency stop (Esc)

**V3.4.0 Foundation:**
✅ **UI/UX Refinement** (31 iterations)
- MacBook Pro 14" optimized layout (1320px fits 1352px viewport)
- Visual consistency - all mixer controls match
- Single-letter tab names (C, P, L, W, S, LIVE)
- Settings tab for application configuration

**V3.3.0 Foundation:**
✅ **Perfect Beat Grid Synchronization** (53 iterations)
- Multi-BPM track alignment
- Master tempo control (60-200 BPM)
- Time-stretching all tracks to master tempo
- Bar-style waveforms

**What Works:**
- ✅ Custom header greeting (ONEMANSHYO branding)
- ✅ Ergonomic keyboard controls (←→ opacity, ↑↓ volume)
- ✅ Library auto-blur during performance
- ✅ Trackpad + bracket key zoom
- ✅ Cell selection persistence (Enter launches correct cell)
- ✅ 5-column grid (scene controls in Master column)
- ✅ Preview orientation updates main canvas
- ✅ About tab with credits
- ✅ Folder overlay with ESC dismiss
- ✅ Dual-domain mixer (shader + audio per track)
- ✅ Independent opacity control with manual override
- ✅ Shader M/S buttons (purple/cyan)
- ✅ Audio M/S buttons (standard + master)
- ✅ Keyboard shortcuts (Shift+ for shader M/S)
- ✅ Four-track shader compositing
- ✅ Pop-out window for second monitor/projector
- ✅ Complete keyboard-based mixer control
- ✅ Quantized launching (bar-aligned clips)
- ✅ Multi-track audio playback (4 tracks)
- ✅ Perfect beat grid synchronization
- ✅ Scene and cell navigation
- ✅ Waveform overlays
- ✅ Settings persistence
- ✅ Professional visual hierarchy
- ✅ MacBook Pro 14" optimized layout

**What Doesn't Work Yet:**
- ❌ Reset to auto mode (manual override persists until reload)
- ❌ Visual mode indicator (AUTO vs MANUAL)
- ❌ Quantization options (1 bar, 1/2 bar, 1 beat, None)
- ❌ Visual launch countdown (armed state)
- ❌ Shader parameter controls in LIVE tab
- ❌ MIDI control
- ❌ Ableton Link

## VERSION EVOLUTION HISTORY

### V3.0.0 - November 2025 (16 iterations)

**Achievement:** LIVE tab UI shell

**What Changed:**
- ShaderBakerLive V1.0.0 (standalone) merged into Shader Baker as 5th tab
- Eliminated need for BroadcastChannel communication
- Direct shared access to audio engine, LIBRARY folder, .oms system
- Performance grid (4 tracks × 8 scenes)
- Basic drag-drop container loading

**File Size:** 330KB

### V3.1.0 - November 2025 (22 iterations A-V)

**Achievement:** Complete LIVE tab functionality

**What Changed:**
- Project States system (.omsp files)
- Multi-track audio playback (4 tracks)
- Mute/Solo functionality
- Real waveforms with playhead
- Shader thumbnails in cells
- Keyboard controls (Delete, Tab, Arrow keys, Spacebar)
- Hardware mixer aesthetic (10px side borders)

**File Size:** 390KB (+60KB)

### V3.2.0 - Date Unknown

**Achievement:** Beat grid + tempo sync foundation

**File Size:** 421KB (+31KB)

### V3.3.0 - November 2025 (53 iterations)

**Achievement:** Perfect beat grid synchronization

**What Changed:**
- Multi-BPM track alignment
- Master tempo control (60-200 BPM)
- Time-stretching all tracks to master tempo
- Bar-style waveforms
- Perfect visual/audio sync
- All Bar 1s align at playhead regardless of source BPM

**File Size:** 445KB (+24KB)

**Critical Foundation:** This is the timing architecture all future versions build upon.

### V3.4.0 - November 2025 (31 iterations)

**Achievement:** UI/UX refinement and visual consistency

**What Changed:**
- MacBook Pro 14" optimized layout (1320px content, fits 1352px viewport)
- Visual consistency - all mixer controls match perfectly
- Single-letter tab names (C, P, L, W, S, LIVE)
- Settings tab for application-level configuration
- Professional polish pass

**File Size:** 445KB (same - pure refinement)

### V3.5.0 - November 2025 (15 iterations)

**Achievement:** Complete keyboard-based performance control

**What Changed:**
- Track selection (1-5 keys, Tab/Shift+Tab)
- Volume control via arrow keys (↑↓)
- Mute/solo shortcuts (comma/period)
- Scene/cell navigation (WASD)
- Quantized launching (bar-aligned clip triggering)
- Master clock architecture
- Global play/pause (Space) and emergency stop (Esc)

**Key Innovation:** Hands-free performance - entire mixer controllable without mouse

**File Size:** 445KB (same - pure functionality)

### V3.6.0 - November 17, 2025 (31 iterations A-AF)

**Achievement:** Four-track shader compositing + pop-out window

**What Changed:**
- Four independent WebGPU rendering pipelines
- Volume-based opacity mixing (natural crossfading)
- Pop-out window for second monitor/projector
- Waveform overlays (container name + shader thumbnail)
- Professional visual indicators (cyan cursor, yellow playing)
- Settings persistence (window size, always-on-top)
- DELETE key cell clearing
- Empty cell clip stop (Ableton-style workflow)
- Z-index layering (Track 1 = top priority)
- 60fps performance with all 4 shaders

**Key Innovation:** Volume faders control BOTH audio AND visuals - natural DJ mixing workflow

**File Size:** 475KB (+30KB)

### V3.8.0 - November 17, 2025 (31 iterations A-ZE)

**Achievement:** Dual-domain mixer + professional UI polish

**What Changed:**
- Independent shader and audio controls per track
- Shader opacity knob with manual override (0-100%)
- Shader M/S buttons (purple/cyan, independent from audio)
- Keyboard shortcuts (Shift+ modifiers for shader controls)
- 50% smaller track knobs (60px → 30px)
- 2x larger master/zoom knobs (30px → 60px)
- All labels removed (clean, minimal interface)
- Perfect header alignment (all headers 43px - 9 iterations to achieve)
- Master mixer M/S + tempo integration
- Symmetrical track layout (shader/audio mirror)
- Visual hierarchy through size
- 60fps performance (dual-domain adds 0% GPU overhead)

**Key Innovation:** Complete separation of visual and audio mixing - fade visuals independently from audio

**Key Learnings:**
- Pixel-perfect alignment requires exact `height`, not `min-height`
- CSS stacking order (outline > border > box-shadow) prevents conflicts
- Symmetry creates intuitive hardware-like interfaces
- Labels unnecessary when position/size/color communicate function
- Scope issues break silently - make functions global when needed
- Visual state must always match audio state

**File:** OMS_ShaderBaker_V3_7_0.html (11,965 lines, 525KB)

### V3.8.0 - November 17, 2025 (26 iterations A-Z)

**Achievement:** Ergonomics + polish + About tab

**What Changed:**
- Custom header greeting with ONEMANSHYO branding (Settings tab control)
- Single-line header layout (24px brand, 11px version, 12px gap)
- Ergonomic opacity control (←→ arrows, no Shift modifiers needed)
- Library auto-blur during performance (prevents scroll interference)
- Trackpad pinch-to-zoom + bracket keys `[` `]` (10px steps)
- Cell selection persistence (root fix - removed `selectedCell = null`)
- Scene controls repositioned (left column → Master column)
- 5-column grid layout (removed empty Scenes column, 30px wider cells)
- Preview orientation updates main canvas (not just preset cards)
- About tab with comprehensive credits (27 people/companies)
- Folder overlay with ESC/Cancel dismiss (app-level, not per-tab)
- PROJECT STATES moved to Settings tab (better organization)
- WASD navigation updated (A enters Track 4, D exits Track 4)

**Key Innovation:** Right-hand ergonomics - ↑↓ volume, ←→ opacity, no modifier keys

**Key Learnings:**
- Band-aids hide root causes (always find the real problem)
- Aggressive cleanup scripts break code (verify manually)
- Inline styles for surgical overrides (no global CSS impact)
- WASD navigation must match grid layout (scenes right = enter right)
- Library scroll interferes with arrow keys (auto-blur solves it)

**Broken Iterations (Learning Points):**
- O: Band-aid fix (blocked Enter) → replaced by R root fix
- P: Aggressive cleanup → removed needed code
- Q: Failed fix on broken base → rebuilt from O

**File:** OMS_ShaderBaker_V3_8_0.html (12,615 lines, 550KB)

**Tab Organization:**
- C (Controls): Shader creation *(no folder)*
- P (Presets): OMS library *(no folder)*
- L (Library): .oms containers *(folder required)*
- W (Code): Shader editor *(no folder)*
- S (Settings): Application config *(folder required)*
- A (About): Identity, credits *(no folder)* ← NEW
- LIVE: Performance mixer *(folder required)*

**Keyboard Mapping (Updated):**
- ↑↓: Volume (vertical)
- ←→: Opacity (horizontal) ← CHANGED from Shift+↑↓
- Shift+, / Shift+.: Shader M/S
- , / .: Audio M/S
- `[` `]`: Zoom (10px steps) ← NEW
- Trackpad pinch/spread: Zoom ← NEW

---

---

**END OF CLAUDE PROJECT INSTRUCTIONS**
