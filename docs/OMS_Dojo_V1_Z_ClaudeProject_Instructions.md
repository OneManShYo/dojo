# OMS Ecosystem - Comprehensive Project Instructions

**Version:** Shader Baker V2.7.0 + ShaderBakerLive V1.0.0  
**Last Updated:** November 2025  
**Purpose:** Complete ecosystem context for Claude AI development

---

## TABLE OF CONTENTS

1. [About Wes Smith (Creator Context)](#1-about-wes-smith)
2. [OMS Ecosystem Overview](#2-oms-ecosystem-overview)
3. [OMS Shader Baker (Main Tool)](#3-oms-shader-baker)
4. [ShaderBakerLive (Performance Mixer)](#4-shaderbakerlive)
5. [Integration & Communication](#5-integration--communication)
6. [Development Guidelines (Both Tools)](#6-development-guidelines)
7. [Communication Protocols](#7-communication-protocols)
8. [Roadmap & Future Vision](#8-roadmap--future-vision)

---

## 1. About Wes Smith (Creator Context)

### Background
- 35 years in music, tech, and software
- Electronic music DJ/producer based in San Diego, CA
- 500+ releases across multiple genres
- 7,000+ days in Beatport top charts
- Founded Juice Recordings label
- Music in video games (PlayStation, Xbox)
- Extensive event production background

### Professional Standards
- Decades of shipping products and building systems
- Knows what works and what's bullshit
- NOT a beginner - assume expert-level knowledge
- Doesn't need basic explanations or hand-holding

### Communication Style
- Direct, efficient, token-conscious
- Numbered questions with lettered options (e.g., "1.a", "2.b")
- "yes go" means proceed immediately
- Corrects imprecise terminology immediately
- Expects exact technical language

---

### ONEMANSHYO - The Artist Project That Started It All

**CRITICAL CONTEXT:** Understanding ONEMANSHYO is essential to understanding why OMS tools exist and what specific workflow they're designed to serve.

**Wes Smith performs as ONEMANSHYO** - a solo DJ/visual artist project centered around:
- **Ableton Live** (audio engine)
- **EboSuite** (video mixing in Ableton)
- **Midifighter Twister or Yaeltex controller** (MIDI control)

**What ONEMANSHYO Does:**
- DJs audio (mixing tracks, building sets)
- Runs live reactive visuals simultaneously
- One person, one setup, both audio AND visuals
- Visuals react to audio in real-time (not canned videos)

**This is NOT VJing:**
- **VJing** = mixing pre-made video clips (video DJing)
- **ONEMANSHYO** = DJing with live reactive visuals (audio-first, visuals respond)
- The distinction is critical - this is DJing WITH a visual element, not video DJing

**Why Not Existing Professional Tools?**

**Resolume, ShowSync, TouchDesigner, EboSuite, etc. are excellent professional tools** - but they're generic platforms designed for:
- Broad use cases (installations, live shows, A/V productions, theater, etc.)
- Team workflows (VJ + DJ + technical crew + operators)
- Complex multi-purpose pipelines
- Generic video mixing capabilities

**OMS tools skip the generic pipeline and laser-focus on ONE specific workflow:**
- Solo creator (no team)
- DJing with live reactive visuals (not VJing)
- No separate VJ, no crew, no multi-person setup
- Streamlined, integrated ecosystem
- Purpose-built for this exact use case

**The Current Workflow Challenge:**
- Requires multiple vendor products (Ableton, EboSuite, hardware controllers)
- Static workflow with extensive prep work
- Complex multi-part setup
- Fragmented ecosystem
- Requires assembling pieces from different vendors

**The OMS Vision:**
- **Single-source solution** for the ONEMANSHYO workflow
- **Web-native** using modern technology (WebGPU, Web Audio, Web MIDI)
- **Integrated ecosystem** (all parts designed to work together)
- **Open source (GPL-3.0)** - freely accessible to anyone
- **No installation required** - runs in browser

**The Impetus:**
OMS tools aren't theoretical experiments - they're codifying Wes's proven ONEMANSHYO workflow (years of real performances) into a unified, browser-based solution purpose-built for solo DJ + live visuals. Not replacing professional platforms like Resolume or TouchDesigner - creating a specialized tool for this specific use case that anyone can access.

**"Anyone can be a OneManShYo"** means:
- Anyone can DJ with live reactive visuals
- No expensive software required
- No complex multi-vendor setup
- No team needed
- Proven workflow, accessible tools

---

### OMS Philosophy
**"OneManShYo" (OMS)** - The name says it all:
- Solo creators producing professional results
- One person handling DJ sets with live reactive visuals
- No team required (no VJ, no crew, no separate visual person)
- Proving it's possible and sharing tools openly
- Everyone can adopt this workflow

### Business Approach
- **OMS Shader Baker:** Open source (GPL), NOT a business
- **ShaderBakerLive:** Open source, community tool
- NOT monetizing these specific tools
- "Ghost protocol" - build, release, zero support
- About empowering creators, not revenue

### Critical Notes
**NEVER suggest for these projects:**
- Pricing, revenue models, commercialization
- "This could be a business"
- Target markets, customer acquisition
- Growth strategies, monetization

**If uncertain about context:** Ask for link: https://www.itswessmithyo.com/

---

## 2. OMS Ecosystem Overview

### The Two-Tool System

```
OMS SHADER BAKER                    SHADERBAKERLIVE
(Visual Creation)                   (Performance Mixer)
     │                                    │
     │  BroadcastChannel                 │
     │  'oms-live-channel'                │
     │                                    │
     ├─→ announces presence ──────────→ discovers
     ├─→ broadcasts snapshots ─────────→ displays
     │                                    │
     │  (Future: FFT/tempo) ←───────────┤ sends audio
     │                                    │
```

### Current State (V1.0.0/V2.8.0)

**Shader Baker (V2.8.0):**
- Creates shader-based visual loops
- Audio-reactive (user-definable frequency mapping)
- Exports silent video for Ableton Live + EboSuite
- Broadcasts presence and canvas snapshots
- Standalone shader development tool

**ShaderBakerLive (V1.0.0):**
- Performance grid UI (4 tracks × 8 scenes)
- Discovers Shader Baker instances
- Displays canvas snapshots in grid
- Hardware mixer controls (UI only)
- Foundation for future OMS Session

**Communication:**
- One-way: Shader Baker → ShaderBakerLive
- Instance announcements + canvas snapshots
- Manual track assignment

### Future Vision: OMS Session

**Complete Performance System:**
- Real-time audio engine with tempo/FFT
- Two-way BroadcastChannel communication
- Shader Baker receives audio data, updates visuals
- Multi-deck performance mixing (4+ channels)
- MIDI control (Midifighter Twister)
- Real-time visual compositing
- Professional VJ-style performance tool

**End Goal:**
- Solo creator DJing with synchronized live visuals
- No external VJ needed
- No expensive software subscriptions
- Browser-based, globally accessible
- **"Anyone can be a OneManShYo"** - the proven ONEMANSHYO workflow, accessible to all

---

## 3. OMS Shader Baker

### Project Identity

**Name:** OMS Shader Baker  
**Current Version:** V2.7.0 (Production), V2.8.0 (BroadcastChannel)  
**Type:** Single-file HTML application (~215KB)  
**Purpose:** Audio-reactive WebGPU shader tool for DJ performance visuals

### Core Concept

**OMS Shader Baker is a COMPLETE professional production tool, NOT just a content container.**

Creates shader-based visual loops that sync to music theory timing:
- Shaders work standalone OR with audio
- Visual timing independent of audio timing
- Exports silent video files for Ableton Live + EboSuite
- User-definable frequency mapping (30-50Hz kick, 50-75Hz sub, etc.)

**What Shader Baker Actually Is:**

**1. Shader Creation & Development Tool:**
- WGSL fragment shader editor
- Real-time compilation and preview
- WebGPU rendering pipeline
- Live shader development environment

**2. Audio-Reactive Visualization System:**
- 32768 FFT analysis (~1.46Hz precision)
- User-definable frequency mapping (6 fields: Freq1/Freq2 for Mod A/B/C)
- Target specific frequencies (30-50Hz kick, 50-75Hz sub, 100-500Hz bass)
- Arrow key navigation (±5Hz fine, ±50Hz coarse)
- Three independent modulators (A/B/C) with subdivision, waveform, and gain controls

**3. Music Theory Timing Engine:**
- Shader Module (visual timing - Shader Tempo BPM)
- Audio Module (musical timing - Audio Tempo BPM)
- Loop In/Out brackets (bar.beat format)
- Beat-synchronized waveform display
- Musical reference frame (proper bar/beat structure)

**4. Professional Video Export System:**
- WebM VP8 rendering (720p-2160p)
- Frame rate control (24/30/60fps)
- Silent video output for Ableton Live + EboSuite
- Render progress tracking
- Duration control

**5. Preset Management System:**
- 5 curated library shaders
- User preset import/export (.txt files)
- Visual thumbnail previews
- Drag-and-drop workflow
- Keyboard navigation

**6. Control States System:**
- Save complete render configurations (21 values)
- 5 library presets (Quick Test, HD, 4K, Instagram, TikTok)
- Unlimited user states
- Real-time state info display
- Zero value support for selective reactivity

**7. BroadcastChannel Broadcasting (V2.8.0):**
- Instance announcements (unique ID, shader name)
- Canvas snapshots (JPEG base64, ~50-100KB)
- 5-second heartbeat keep-alive
- Foundation for Live integration

**Shader Baker is a complete standalone tool that works perfectly alone.**  
ShaderBakerLive doesn't make it useful - it adds performance mixing capabilities to an already professional tool.

### Two Independent Modules

**CRITICAL:** Separate systems with limited integration:

**1. Shader Module (Visual Timing):**
- Controls visual animation timing
- Shader Tempo field (BPM for visual timing)
- Loop Length, Render Duration
- Modulators A/B/C (subdivision, waveform, gain)
- Works completely without audio

**2. Audio Module (Musical Timing):**
- Controls audio loop positioning
- Audio Tempo field (BPM for beat positioning)
- Loop In/Out brackets (bar.beat format)
- Waveform visualization with beat grid
- Drives beat-based navigation

**Integration Points (Only Two):**
1. Loop brackets → Render Duration (when audio loaded)
2. FFT → Modulators (audio analysis enhances shader modulators)

### V2.7.0 Major Features

**User-Definable Frequency Mapping:**
- 6 frequency input fields (Freq1/Freq2 for Mod A/B/C)
- 32768 FFT analysis (~1.46 Hz/bin precision)
- Target specific frequencies (30-50Hz kick, 50-75Hz sub, 100-500Hz bass)
- Arrow key navigation (±5Hz fine, ±50Hz coarse)

**Control States System:**
- Save complete render configurations (21 values)
- 5 library presets + unlimited user presets
- Info box with real-time state display
- Zero value support for selective reactivity

**Preset System:**
- 5 library shaders (curated)
- User presets with visual thumbnails
- Drag-and-drop import
- Keyboard navigation

### V2.8.0 BroadcastChannel Features

**Instance Announcement:**
- Unique ID: 'sb-XXXXXXXXX'
- Broadcasts presence on page load
- Announces when shader compiles
- 5-second heartbeat keep-alive
- Shader name from first comment line

**Canvas Snapshots:**
- JPEG snapshots on compile
- Base64 encoded for BroadcastChannel
- Auto-snapshot on page load (500ms delay)
- ~50-100KB per snapshot

### Technical Stack

- **Rendering:** WebGPU/WGSL fragment shaders
- **Audio Analysis:** Web Audio API, 32768 FFT samples
- **Video Export:** WebM VP8 (720p-2160p, 24/30/60fps)
- **Storage:** LocalStorage (presets, states)
- **Communication:** BroadcastChannel ('oms-live-channel')

### File Management

**Single-File Philosophy:**
- One HTML file (~215KB)
- All functionality self-contained
- No external dependencies
- Portable, offline-capable

**Deliverables Package (9 files):**
1. Main HTML application
2. Deliverables inventory
3. Complete changelog
4. Developer documentation
5. Release guide
6. Claude shader creation guide
7. User guide (all)
8. FFmpeg video guide
9. Project instructions

### Critical Terminology

**DO Use:**
- "DJing with live reactive visuals"
- "DJ set with visual element"
- "Shader loops for DJ sets"

**NEVER Use:**
- "VJing" or "VJ" (this is DJing, not VJing)
- "A/V performance" (too art-world)
- "Music videos" (videos are silent)

---

## 4. ShaderBakerLive

### Project Identity

**Name:** OMS Shader Baker Live (ShaderBakerLive)  
**Current Version:** V1.0.0  
**Future Name:** OMS Session  
**Type:** Single-file HTML application (~37KB)  
**Purpose:** Performance mixer interface for shader-based visual sets

### Current State (V1.0.0)

**What Works:**
- Performance grid UI (4 tracks × 8 scenes)
- Instance discovery via BroadcastChannel
- Manual instance-to-track assignment
- Canvas snapshot display in grid
- Hardware mixer controls (visual only)
- Fixed 1200px layout (non-responsive)

**What Doesn't Work:**
- No audio engine
- No FFT broadcasting
- No clip launching
- No scene launching
- Meters are placeholders
- No MIDI control

### Architecture

**Layout Structure:**
```
Fixed Header (50px)
├─ Performance View (flex: 1)
│  ├─ Grid Headers (fixed) - Track headers + dropdowns
│  ├─ Scenes Scroll (scrollable) - 8 scenes × 4 tracks
├─ Mixer Section (140px, fixed) - Always visible
│  └─ 4 Track controls (rotary knobs, M/S, meters)
├─ Waveform Section (200px) - Placeholders
└─ Status Bar (30px) - System messages
```

**Design Philosophy:**
- Fixed 1200px width (hardware paradigm)
- Frozen panes (headers/mixer fixed, scenes scroll)
- Rotary knobs (not sliders)
- Vertical L/R meters (not horizontal)
- Zero gaps, flush design
- Hardware mixer aesthetic

### Development History (V1.0.0)

**15 Iterations (A-P, N skipped):**
- A-D: UI foundation (grid, tracks, controls)
- E-F: Fixed sections (mixer, headers)
- G-H: Hardware controls (rotary, vertical meters)
- I-J: BroadcastChannel (discovery, snapshots)
- K-L: Grid refinement (landscape, compact)
- M-P: Layout polish (fixed width, zero gaps, divider)

### Roadmap

**V1.1.0 (Next):**
- Audio engine (Web Audio API)
- Audio file loading per clip
- Basic playback (play/pause per track)
- Real-time meter animation
- Master section (5th mixer column)

**V1.2.0:**
- FFT broadcasting to Shader Baker
- Scene launching functionality
- Clip launching across tracks
- Tempo synchronization

**V2.0.0+:**
- MIDI control integration
- Ableton Link sync
- Multi-page support
- Advanced compositing
- Crossfader

### Terminology

**DO Use:**
- "Performance view" (not "session view")
- "Track" (not "channel" or "deck")
- "Scene" (not "clip launcher row")
- "Clip cell" (not "clip slot")

**NEVER Use:**
- "Session view" (Ableton proprietary)
- "VJ/VJing" (different paradigm)
- "A/V performance" (too art-world)

---

## 5. Integration & Communication

### BroadcastChannel Protocol

**Channel Name:** `'oms-live-channel'`

**Current Messages (V1.0.0/V2.8.0):**

**1. Announce (Shader Baker → Live):**
```javascript
{
  type: 'announce',
  id: 'sb-abc123',
  name: 'Bass Tunnel v3',
  timestamp: Date.now()
}
```
- Frequency: Page load + every 5 seconds (heartbeat)
- Purpose: Instance discovery

**2. Snapshot (Shader Baker → Live):**
```javascript
{
  type: 'snapshot',
  id: 'sb-abc123',
  imageData: 'data:image/jpeg;base64,...',
  timestamp: Date.now()
}
```
- Frequency: On compile + page load
- Purpose: Visual preview in grid

**Future Messages (V1.2.0+):**

**3. Audio Update (Live → Shader Baker):**
```javascript
{
  type: 'audio_update',
  tempo: 128,
  beat: 1.5,
  bassFFT: 0.75,
  midFFT: 0.45,
  trebleFFT: 0.60,
  timestamp: Date.now()
}
```
- Frequency: 60fps during playback
- Purpose: Audio-reactive shaders

**4. Scene Launch (Live → Shader Baker):**
```javascript
{
  type: 'scene_launch',
  scene: 1,
  timestamp: Date.now()
}
```
- Frequency: On scene trigger
- Purpose: Synchronized shader switching

### Integration Workflow

**Current (V1.0.0):**
1. Open ShaderBakerLive
2. Open Shader Baker tab(s)
3. Live auto-discovers instances
4. User assigns instances to tracks (dropdown)
5. Compile shaders → snapshots appear in grid
6. Visual preview/selection ready

**Future (V1.2.0+):**
1. Load audio files in Live clips
2. Launch scene → audio plays
3. Live analyzes audio (FFT, tempo, beat)
4. Live broadcasts audio data
5. Shader Baker receives data
6. Shader Baker updates modulators
7. Real-time audio-reactive visuals

### Compatibility

**Shader Baker Requirements:**
- V2.8.0-A or later (BroadcastChannel support)
- Broadcasting enabled
- Instance announcement system
- Canvas snapshot system

**Browser Requirements:**
- Chrome 88+ (recommended)
- Safari 15+
- Firefox 89+
- BroadcastChannel API support
- WebGPU support (for Shader Baker)

---

## 6. Development Guidelines

### Iteration Process (Both Tools)

**Standard Pattern:**
1. Read relevant SKILL.md files (if applicable)
2. Ask before building (numbered questions with lettered options)
3. One iteration per build (A, B, C, etc.)
4. Provide two files per iteration:
   - [Tool]_V#_#_#_[Letter]_[DescriptiveName].html
   - [Tool]_V#_#_#_[Letter]_[DescriptiveName]_Notes.txt

**Naming Convention:**
- Use underscores: V2_7_0_A_FeatureName
- NOT periods: V2.7.0.A (breaks sorting)
- NOT hyphens: V2-7-0-A (breaks sorting)
- Descriptive names: V2_7_0_A_FrequencyFields (not V2_7_0_A)

### Token Management

**Keep responses concise:**
- Structure ALL questions as numbered with lettered options
- Allow user to answer "1.a" or "yes to 2"
- Don't force paragraph answers when yes/no works
- Watch token count, warn if approaching limit

**Example Good Format:**
```
1. Should I add audio engine in iteration Q?
   a) Yes, add Web Audio API setup
   b) No, different feature first
   c) Skip for now
```

**Example Bad Format:**
```
"What do you think about adding audio? How should it work?"
(Forces paragraph response, wastes tokens)
```

### File Size Management

**Keep HTML files lean:**
- No embedded changelog (separate notes files)
- Minimal comments in HTML (lines 1-35 max)
- No historical notes in code
- Clean, production-ready code only

**Why:**
- Embedded changelog causes token crashes in Claude
- Notes files track history separately
- Production file stays portable
- 500KB+ of embedded text serves no purpose

### Single-File Philosophy (Both Tools)

**Critical:**
- All functionality in one HTML file
- No splits, no external dependencies
- Portable, self-contained
- Works offline (except BroadcastChannel)

**Benefits:**
- Easy distribution (share one file)
- No installation required
- No build process
- Open in browser, done

### Communication Guidelines

**NEVER Suggest:**
- Business models, pricing, revenue (for these tools)
- "This could be a business"
- Target markets, customer acquisition
- Max for Live versions (locks out non-Ableton)
- Server-side processing (client-side philosophy)
- TouchDesigner replacement (different use case)
- Resolume replacement (different use case)

**DO:**
- Encourage creativity
- Provide technical support
- Let users drive vision
- Follow "OneManShYo" philosophy

---

## 7. Communication Protocols

### Development Communication

**Wes's Patterns:**
- Provides clear specifications upfront
- Expects immediate implementation after "yes go"
- Demonstrates familiarity with systematic development
- Corrects imprecise terminology immediately
- Uses direct feedback like "solid" or "yes go"

**Claude's Response:**
- Ask clarifying questions using numbered format
- One feature per iteration (surgical changes)
- Provide complete context in iteration notes
- Reference existing documentation
- Use exact technical language

### Technical Language

**Shader Baker:**
- "Shader Module" vs "Audio Module" (separate systems)
- "Shader Tempo" vs "Audio Tempo" (different fields)
- "Modulators" (not "parameters")
- "Control States" (not "presets" - presets are shaders)
- "Loop In/Out" (not "start/end")

**ShaderBakerLive:**
- "Performance view" (not "session view")
- "Track" (not "channel" or "deck")
- "Scene" (not "scene launcher")
- "Clip cell" (not "clip slot")
- "Mixer section" (not "mixer strip")

---

## 8. Roadmap & Future Vision

### Shader Baker Evolution

**V2.8.0 (Current):**
- ✓ BroadcastChannel broadcasting
- ✓ Instance announcements
- ✓ Canvas snapshots

**V2.9.0+:**
- Receive FFT/tempo from ShaderBakerLive
- Update modulators from external audio
- Scene launch commands
- Multi-shader library per instance
- Remote preset loading

**V3.0.0:**
- Remove all audio features (full separation)
- Pure visual player
- Receives all data from OMS Session
- Continuous play mode

### ShaderBakerLive → OMS Session

**V1.1.0:**
- Audio engine (Web Audio API)
- Clip loading/launching
- Master section
- Real-time meters

**V1.2.0:**
- FFT broadcasting
- Scene launching
- Clip launching across tracks
- Tempo synchronization

**V2.0.0:**
- MIDI control (Midifighter Twister)
- Ableton Link integration
- Multi-page support (8+ decks)
- Advanced compositing

**OMS Session (Final Form):**
- Professional performance tool
- Real-time visual mixing
- 4-deck (expandable) system
- Complete hardware control
- Recording/export capabilities
- Professional VJ-level features
- **"Anyone can be a OneManShYo"** - the proven workflow, accessible to all

### The Complete Ecosystem

```
CREATOR'S WORKFLOW:

1. OMS Shader Baker
   - Create shader visuals
   - Export silent video OR
   - Run live in browser tab

2. OMS Session (ShaderBakerLive evolved)
   - Load audio files
   - Mix audio across decks
   - Real-time FFT analysis
   - Broadcast to Shader Baker instances
   - Visual compositing
   - MIDI control

3. PERFORMANCE:
   - Solo creator
   - Professional DJ set with visuals
   - No team needed
   - No expensive software
   - "OneManShYo" realized
```

### The Vision

**Current Reality:**
- DJs need separate VJ for visuals
- Or complex multi-vendor setups (Ableton + EboSuite + hardware)
- Expensive software subscriptions
- Complex technical setups
- Team required for professional shows OR extensive prep work

**OMS Future:**
- One person, one laptop
- Browser-based tools (free, open source)
- Professional-level results
- Accessible to anyone
- **Codifies the proven ONEMANSHYO workflow into accessible tools**
- **"Anyone can be a OneManShYo"**

---

## 9. When Starting New Development

### For Shader Baker V2.9.0+

**Before making changes:**
1. Add this comprehensive instructions file to Claude project
2. Review Shader Baker V2.7.0/V2.8.0 deliverables
3. Read recent iteration notes (last 5-10)
4. Understand current architecture state
5. Ask user about goals for new version
6. Create iteration plan with numbered questions
7. Get "yes go" confirmation
8. Proceed incrementally

**Key Files:**
- OMS_ShaderBaker_V2_7_0_Z_Project_Instructions.md (specific context)
- OMS_Comprehensive_Project_Instructions.md (this file - full context)
- Recent iteration notes (A-AG for V2.7.0, A-C for V2.8.0)

### For ShaderBakerLive V1.1.0+

**Before making changes:**
1. Add this comprehensive instructions file to Claude project
2. Review ShaderBakerLive V1.0.0 deliverables
3. Read iteration notes (A-P)
4. Understand UI shell foundation
5. Discuss audio engine architecture
6. Plan BroadcastChannel sending (not just receiving)
7. Create iteration plan with numbered questions
8. Get "yes go" confirmation
9. Proceed incrementally

**Key Files:**
- ShaderBakerLive_V1_0_0_Z_Project_Instructions.md (specific context)
- OMS_Comprehensive_Project_Instructions.md (this file - full context)
- Iteration notes A-P

---

## 10. Critical Reminders

### For Both Tools

**Technical:**
- Single-file architecture (non-negotiable)
- No external dependencies
- Incremental development (one feature per iteration)
- Proper version control and documentation
- Token-efficient communication

**Communication:**
- No business suggestions for these tools
- No "VJ/VJing" terminology
- No Ableton proprietary terms
- Expert-level knowledge assumed
- Direct, efficient responses

**Philosophy:**
- **"OneManShYo"** - empowering solo creators to adopt the proven ONEMANSHYO workflow
- Open source, community-driven
- "Ghost protocol" - build and release
- Professional standards always
- Free and accessible

### Integration Notes

**Current State:**
- One-way: Shader Baker → ShaderBakerLive
- Announcements + snapshots only
- Manual track assignment

**Future State:**
- Two-way: Both directions
- Audio data: Live → Shader Baker
- Scene commands: Live → Shader Baker
- Real-time synchronized performance

---

## 11. Success Metrics

### Shader Baker V2.7.0
✓ User-definable frequency mapping  
✓ 32768 FFT precision  
✓ Control States system  
✓ Professional bass music responsiveness  
✓ 33 iterations without system crashes  

### Shader Baker V2.8.0
✓ BroadcastChannel integration  
✓ Instance announcement system  
✓ Canvas snapshot broadcasting  
✓ Auto-snapshot on page load  
✓ Foundation for Live integration  

### ShaderBakerLive V1.0.0
✓ Professional UI aesthetics  
✓ Hardware mixer paradigm  
✓ Instance discovery working  
✓ Canvas snapshot display  
✓ Fixed/scrollable layout  
✓ Zero gaps, flush design  
✓ 15 iterations A-P complete  

### Future Goals
- V1.1.0: Audio engine functional
- V1.2.0: Two-way BroadcastChannel
- V2.0.0: Professional performance tool
- OMS Session: Complete ecosystem realized
- **Ultimate Goal: "Anyone can be a OneManShYo"**

---

**Document Version:** Shader Baker V2.7.0/V2.8.0 + ShaderBakerLive V1.0.0  
**Last Updated:** November 2025  
**Next Update:** When major version development begins

---

**END OF COMPREHENSIVE PROJECT INSTRUCTIONS**
