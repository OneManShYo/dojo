# OMS SHADER BAKER V3.2.0 - CLAUDE PROJECT INSTRUCTIONS

**Version:** V3.2.0
**Date:** November 2025
**Purpose:** Complete project context for Claude AI development

---

## CURRENT STATE: V3.2.0

### Production Status

**LIVE Tab - Production Ready:**
- ✅ Beat grid visualization (bars + phrase markers)
- ✅ Master tempo control (60-200 BPM)
- ✅ Time-stretching all tracks to master tempo
- ✅ Synced waveform scrolling (Serato-style)
- ✅ Complete audio timing data storage
- ✅ Bar-style waveforms (mixing-optimized)
- ✅ Multi-track audio playback (4 tracks)
- ✅ Mixer controls (volume, mute, solo, meters)
- ✅ Project States persistence (.omsp)
- ✅ Keyboard navigation

**What Doesn't Work Yet:**
- ❌ Scene launching (rows)
- ❌ Shader rendering in cells (thumbnails only)
- ❌ Master output compositing
- ❌ Waveform control buttons (visual only)
- ❌ MIDI control
- ❌ Ableton Link

**File:** OMS_ShaderBaker_V3_2_0.html (9,783 lines, 421KB)

---

## V3.2.0 DEVELOPMENT SUMMARY

### 31 Iterations (A-Z, AA-AE)

**Phase 1: Scene Launch & Audio Sync (A-I)**
- Scene launching with synchronized start
- Pre-loading audio (Ableton-style)
- Waveform positioning
- Sync buffer optimizations

**Phase 2: Visual & Controls (J-M)**
- Track waveform control strip
- Modular track row design
- Track control buttons (Q/|</|>/<</>>) 

**Phase 3: Audio/Shader Separation (N-P)**
- Shader* prefix for shader controls
- Audio* prefix for audio controls
- Complete audio timing storage in .oms

**Phase 4: Beat Grid Implementation (Q-Z)**
- Beat grid visualization
- Master tempo control
- Phrase markers (magenta every 4 bars)
- Bar-style waveforms
- Critical Update button bug fix

**Phase 5: Critical Debugging (AA-AE)**
- 5 major bugs fixed in AB session
- ScaleX ratio inverted fix (rubber band)
- Transform origin optimization
- Synced scroll speed (final working version)

**Key Achievement:** Complete DJ-ready visual timing with Serato/Rekordbox-style sync

---

## ROADMAP

### V3.3.0: Scene Launching + Quantization
**Status:** Planned
**Timeline:** Q1 2026

**Features:**
- Scene launching (trigger entire rows)
- Quantized launching (snap to bars/phrases)
- Loop bracket visualization
- Crossfading between cells
- Enhanced transport controls

**Why:** Complete the performance workflow with multi-track launching

### V3.4.0: MIDI + Advanced Control
**Status:** Planned
**Timeline:** Q1 2026

**Features:**
- Web MIDI API integration
- "Wilson CC" - tribute to Chris Wilson (MIDI spec creator)
- Midifighter Twister mapping
- Hardware controller support
- Multi-page support (8+ tracks)
- Ableton Link synchronization

**Why:** Hardware control for professional performance

### V3.5.0: Real-Time Shader Rendering
**Status:** Planned
**Timeline:** Q2 2026

**Features:**
- Shader rendering in grid cells (4 simultaneous)
- Master output compositing
- Visual blend modes (additive, multiply, screen)
- Audio-reactive shaders per track
- Complete visual performance

**Why:** Replace shader thumbnails with animated visuals

### V4.0.0: OMS Session
**Status:** Vision
**Timeline:** Q3 2026

**Goal:** Complete Ableton + EboSuite replacement

**Features:**
- Complete performance mixing environment
- Recording/export from LIVE tab
- Advanced session management
- Professional-level control
- "Anyone can be a OneManShYo"

---

## DEVELOPMENT PROTOCOLS

### Version Naming (CRITICAL)

**Production Release:**
- Filename: `OMS_ShaderBaker_V3_2_0.html`
- Title: `v3.2.0` (lowercase v)
- Header: `[V3.2.0]` (brackets)

**Development Build:**
- Filename: `OMS_ShaderBaker_V3_3_0_A.html`
- Title: `v3.3.0a` (lowercase v and letter)
- Header: `[V3.3.0_A]` (underscore before letter)

**Use underscores, not periods:** V3_3_0_A (not V3.3.0_A)

### Iteration Process

**1. Ask Before Building:**
- Present numbered questions with lettered options
- Wait for "yes go" or specific answer
- Never proceed without confirmation

**2. Iterate with Letters:**
- A, B, C... Z, AA, AB, AC... (sequential)
- One feature per iteration
- Never skip letters

**3. Update Version Display:**
- HTML title tag
- UI header display
- Both must match

**4. Deliver Files:**
- HTML file: `OMS_ShaderBaker_V3_3_0_A.html`
- Notes file: `OMS_ShaderBaker_V3_3_0_A_Notes.txt`

### Code Practices

**Single-File Philosophy:**
- All functionality in ONE HTML file
- No splits, no external dependencies
- Portable, self-contained

**Surgical Changes:**
- Modify specific functions only
- Don't rewrite entire files
- Preserve existing functionality
- Test incrementally

**Token Management:**
- Structure questions as numbered with letters
- Keep responses concise
- Watch token limits
- Warn if approaching limit

---

## TECHNICAL ARCHITECTURE

### Beat Grid System

**Components:**
- trackContainerData[] - Audio timing per track
- drawBeatGridOnWaveform() - Canvas rendering
- Master tempo sync - Time-stretching engine
- CSS transforms - GPU-accelerated scaling
- Synced scrolling - RequestAnimationFrame loop

**Key Algorithms:**
```javascript
// Audio playback rate
playbackRate = masterTempo / audioBPM

// Visual scaleX (INVERTED!)
scaleX = audioBPM / masterTempo

// Transform origin
transformOrigin = bar1PositionPx

// Synced scroll
scrollOffset = elapsed * pixelsPerSecond * scaleX
```

### Audio Data Storage

**Extended .oms Format:**
```json
{
  "shaderCode": "...",
  "controlStates": { "shader*": ... },
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

### From V3.2.0_AB Debugging

**The 5 Bugs:**
1. Double scaling (grid + CSS)
2. Wrong transform origin
3. Array lookup vs parameter
4. getBoundingClientRect transformed position
5. Scroll animation losing scaleX

**Key Insights:**
- Audio was always right (proves timing data correct)
- Visual must match audio (same ratio, same approach)
- Transform origin is critical (wrong pivot breaks everything)
- Ableton "Start from here" analogy helpful
- Each track independent (scale from THEIR Bar 1)

**The Lesson:**
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

### Claude's Response Pattern

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

**Business/Monetization:**
- ❌ Pricing models
- ❌ Revenue strategies
- ❌ Target markets
- ❌ Subscription plans

**Why:** Open source (GPL-3.0), free forever, "ghost protocol"

**Platform/Distribution:**
- ❌ Max for Live version
- ❌ VST plugin
- ❌ App store distribution
- ❌ Native apps

**Why:** Web-first philosophy, accessible to all

**Architecture Changes:**
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
- Beat grid visualization
- Master tempo control (60-200 BPM)
- Synced waveform scrolling
- Mixer controls (volume, mute, solo, meters)
- Bar-style waveforms
- Project States (.omsp)
- Keyboard navigation

**All Tabs:**
- WebGPU shader rendering
- FFT audio analysis (32768 samples)
- User-definable frequency mapping
- Control States system
- LIBRARY-first architecture
- .oms file system
- Auto-save workflow

### Planned (Future Versions)

**V3.3.0:**
- Scene launching
- Quantized triggers
- Loop brackets
- Crossfading

**V3.4.0:**
- MIDI control
- Ableton Link
- Hardware controllers

**V3.5.0:**
- Real-time shader rendering
- Master output compositing
- Visual blend modes

---

## DELIVERABLES PACKAGE

**CRITICAL - ALWAYS DO THIS:**
1. Create properly named folder: `OMS_ShaderBaker_V{version}_Deliverables/`
2. Put all 9 files inside the folder
3. Zip the folder: `OMS_ShaderBaker_V{version}_Deliverables.zip`
4. Provide ONE download link to the zip file

**DO NOT just dump files in outputs folder - Wes will quit using Claude if you do this wrong!**

**9 Files per Release:**
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

**File Size Evolution:**
- V3.0.0: 330KB (LIVE UI shell)
- V3.1.0: 390KB (+LIVE functionality)
- V3.2.0: 421KB (+beat grid + tempo sync)

---

## SUCCESS CRITERIA

**V3.2.0 Complete When:**
- ✅ Beat grid renders on waveforms
- ✅ Master tempo controls all tracks
- ✅ Waveforms scroll together
- ✅ Audio timing data saves/loads
- ✅ All V3.1.0 features preserved
- ✅ Professional visual timing reference

**V3.3.0 Complete When:**
- ⏳ Scene launching works (trigger rows)
- ⏳ Quantized launching (snap to bars)
- ⏳ Loop brackets visualized
- ⏳ Crossfading between cells

---

## CRITICAL REMINDERS

**Development:**
- Start from working base
- Make surgical changes only
- One feature per iteration
- Test incrementally
- Use underscores in version numbers

**Communication:**
- Listen to what Wes says
- Use numbered questions
- Wait for "yes go"
- Use exact technical terminology
- Reference documentation

**Technical:**
- Single-file philosophy
- GPU-accelerated when possible
- Clean separation (shader vs audio)
- Backward compatible
- Professional standards

---

**END OF PROJECT INSTRUCTIONS**

V3.2.0: Professional visual timing reference - bars, beats, sync, ready to mix.
V3.3.0: Next stop - scene launching and quantization!
