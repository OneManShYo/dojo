# ONEMANSHYO Dojo - Project Instructions for Claude

**Version:** V2_6  
**Last Updated:** November 2025  
**Purpose:** Complete project context for Claude AI development

---

## 1. About Wes Smith (Creator Context)

**Background:**
- 35 years in music, tech, and software
- Electronic music DJ/producer based in San Diego, CA
- 500+ releases across multiple genres, 7,000+ days in Beatport top charts
- Founded Juice Recordings label
- Music in video games (PlayStation, Xbox - Tabletop Racing World Tour)
- Extensive event production background (Miami club nights, etc.)

**Approach:**
- Professional standards - decades of shipping products and building systems
- Knows what works and what's bullshit
- NOT a beginner - assume expert-level knowledge in music production, software development, and industry landscape
- Doesn't need basic explanations or hand-holding

**ONEMANSHYO Dojo Business Model:**
- This specific project is NOT a business venture
- Open source (GPL license) for the community
- About empowering solo creators, not monetization
- **Never suggest:** pricing, revenue models, commercialization strategies for this tool
- (Note: Wes isn't "done with business" - just not doing it for this project)

**If uncertain about context:** Ask for link: https://www.itswessmithyo.com/

---

## 2. ONEMANSHYO Dojo - Vision & Potential

### What It Is

**ONEMANSHYO Dojo** is an audio-reactive WebGPU shader tool that creates music visualizer videos for live DJ performances.

**Core Concept:**
- Create shader-based visual loops that sync to music theory timing (bars, beats, subdivisions)
- Audio reactivity enhances visuals (bass/mid/treble frequency response)
- Shaders work standalone OR with audio - visual timing independent of audio timing
- Exports silent video files for use in live performance software (Ableton + EboSuite)

**Not Shadertoy:**
- Shadertoy has archaic limitations (GLSL, limited parameters, clunky export, online-only)
- Dojo uses modern WebGPU/WGSL, better parameters, offline capability, easy rendering
- Built to replace Shadertoy for OMS Live needs

### The OneManShYo Philosophy

**Mission:** "Anyone can be a OneManShYo"

**What it means:**
- One person can handle professional DJ sets with live reactive visuals
- No team required (no VJ, no crew, no separate visual person)
- Pro-level results without expensive software subscriptions
- Proving it's possible and sharing the tools openly

**What it's NOT:**
- Not gatekeeping knowledge
- Not selling services
- Not a business venture (this tool specifically)

### OMS Live - The Complete System

**Current State (How It Works Today):**

```
Ableton Live Session:
├── Audio Channels: Club tracks (DJ mixing, beat matching)
├── Video Channels: Shader loops (SILENT, pre-rendered)
└── EboSuite (Max for Live): Video mixing/effects

Midifighter Twister:
├── Triggers audio clips (DJ mixing)
├── Triggers video clips (visual switching)
└── Follow actions evolve content automatically

The Setup:
- Videos are SILENT (no audio track) - muted in Ableton
- All audio comes from audio channels (club tracks)
- Videos tempo-synced (rendered at session BPM)
- Audio-reactive (shader modulators respond to bass/mid/treble)
```

**This is DJing with visuals - NOT VJing:**
- VJing = playing music videos (video+audio together)
- This = DJing with separate visual element (silent shader loops)
- **Never use "VJ" or "VJing" terminology** - it's a DJ set with visual element

**The Vision (May Never Reach, But Working Toward):**
- Same DJ workflow
- Real-time shader generation (no pre-rendered video files)
- Shaders render live, synced via Ableton Link
- Cut out the video file middleman entirely
- "OMS" = the complete real-time system

**Why Pre-Rendered Videos Today:**
- Technical limitations (EboSuite works with video files)
- Real-time WebGPU → Ableton integration not yet solved
- Syphon possible but adds complexity
- Pre-rendering is the practical solution for now

**Path Forward:**
- Web version stays core (free, accessible shader creation)
- Native Mac version could enable real-time via Syphon
- But may never reach that - current workflow works fine
- Goal exists, even if never fully achieved

---

## 3. Technical Specifications

### Project Identity

**Name:** ONEMANSHYO Dojo (Audio-Reactive WebGPU Shader Tool)  
**Current Version:** V2_6 (Production Release)  
**Type:** Single-file HTML application (~199KB)  
**Architecture:** HTML + CSS + JavaScript (no external dependencies)

**Technologies:**
- WebGPU rendering with WGSL fragment shaders
- FFT audio analysis (Bass/Mid/Treble frequency bands)
- Loop system with BPM-based beat positioning
- LocalStorage for user preset persistence
- Video export: WebM (VP8, 720p-2160p, 24/30/60fps)

**Single-File Philosophy:**
- All functionality in one HTML file
- No splits, no external dependencies
- Portable, self-contained application

### User Workflows

**Shader Workflow (Primary - Creating Visual Content):**
1. User creates WGSL fragment shader in Claude (conversational)
2. Paste shader into Code tab in Dojo
3. Click [Compile] → preview updates
4. Tweak visual controls (modulators, gains, Shader Tempo)
5. Render video (silent, no audio track)
6. Load video into EboSuite for DJ set

**Audio Workflow (When Testing with Audio):**
1. Upload audio track → Manual tempo entry
2. Set Loop In/Out using arrow keys (beat-based positioning)
3. Fine-tune with tempo decimal precision (±0.01, ±1.0)
4. Adjust Loop Out with fine increments (±1/32, ±1/8 note)
5. Enable Loop button → audition loop (cyan when active)
6. Render video with selected resolution/FPS
7. Video includes audio from Loop In to Loop Out (or render without audio)

**Preset Workflow (NEW in V2_6):**
1. Click Presets tab
2. Browse Library (5 curated shaders) or User Presets
3. Tab through thumbnails to see visuals
4. Press Enter to load shader
5. Adjust settings in Controls tab
6. Render video
7. Repeat for next shader (fast iteration)

### Architecture: Two Independent Modules

**CRITICAL:** These are separate systems with limited integration points.

**Shader Module (Visual Timing System):**
- **UI Location:** Controls tab, Row 1-5
- **Key Field:** "Shader Tempo" (BPM value controlling visual timing)
- **Components:**
  - Shader Tempo field (Row 1)
  - Loop Length dropdown (Row 1)
  - Render Duration field (Row 1)
  - Modulators A/B/C with subdivision, waveform, gain controls (Rows 2-4)
- **Purpose:** Controls visual animation timing in shader
- **Independent Operation:** Shaders work completely without audio

**Audio Module (Musical Timing System):**
- **UI Location:** Audio controls strip below waveform
- **Key Field:** "Audio Tempo" (BPM value controlling beat positioning)
- **Components:**
  - Audio Tempo field (manual entry)
  - Audio Start/End fields (MM:SS.mmm format)
  - Loop In/Out brackets (bar.beat format)
  - Waveform visualization with beat grid
  - Transport controls (Play/Pause, Loop, Remove Audio)
- **Purpose:** Controls audio loop positioning and beat-based navigation
- **Drives:** Loop In/Out bracket positioning, arrow key beat increments

**Module Integration Points (Only Two):**
1. **Loop Brackets → Render Duration:** When audio loaded, Loop In/Out from Audio Module define time range for video rendering
2. **FFT → Modulators:** Audio waveform analysis (Bass/Mid/Treble) from Audio Module feeds Shader Module modulators (Mod A, Mod B, Mod C)

**Naming Convention for Claude:**
- When discussing architecture: "Shader Module" and "Audio Module"
- When discussing UI: Use actual field names ("Shader Tempo" field or "Audio Tempo" field)
- Never use generic "tempo" or "BPM" alone without module context
- Modules remain independent unless explicitly linked by user action (future feature)

### Critical Code Sections

**Render Function (Shader Module vs Audio Module Modes):**

Shader Module Mode (No Audio):
```javascript
const renderDurationStr = document.getElementById('renderDuration').value;
duration = timeStringToSeconds(renderDurationStr); // MM:SS.mmm format
beatCount = Math.ceil((duration * shaderTempo) / 60);
startTime = Date.now(); // Start from beat 0
```

Audio Module Mode (With Audio):
```javascript
const inBeat = barBeatToBeats(document.getElementById('inBeat').value);
const outBeat = barBeatToBeats(document.getElementById('outBeat').value);
const audioBPM = audioTempo; // Audio Module tempo - NOT currentTempo!
beatCount = outBeat - inBeat;
duration = (beatCount * 60) / audioBPM;
const inBeatTime = inBeat / (audioBPM / 60.0);
startTime = Date.now() - (inBeatTime * 1000); // Offset for Loop In
```

**Loop Playback Detection (Audio Module):**
```javascript
const audioBeatsPerSecond = audioTempo / 60.0; // Use Audio Module tempo!
const currentAudioBeat = (audioElement.currentTime - audioStartTime) * audioBeatsPerSecond;

if (loopEnabled && currentAudioBeat >= (outBeat - 0.15)) { // 0.15 beat threshold
    audioElement.currentTime = audioStartTime + (inBeat / audioBeatsPerSecond);
}
```

**WebGPU Uniform Buffer (CRITICAL - 48 bytes):**
```javascript
// MUST be 48 bytes for proper 16-byte alignment
const uniformData = new Float32Array([
    currentTime,      // 0: time
    currentTempo,     // 1: tempo
    modA,             // 2: modA
    modB,             // 3: modB
    modC,             // 4: modC
    0,                // 5: padding1
    canvas.width,     // 6: resolutionX
    canvas.height,    // 7: resolutionY
    0, 0, 0, 0        // 8-11: padding2-5 (48 bytes total)
]);
```

**WGSL Struct Layout (CRITICAL - separate resolutionX/Y):**
```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    padding1: f32,
    resolutionX: f32,  // NOT vec2!
    resolutionY: f32,
}

// Reconstruct in shader:
let resolution = vec2<f32>(uniforms.resolutionX, uniforms.resolutionY);
```

### Completed Features (V2_6)

**NEW - Preset System:**
- Library section with 5 curated shaders
- User Presets with localStorage persistence
- Visual thumbnail previews (WebGPU rendering)
- Drag-and-drop import (.txt/.wgsl files)
- Keyboard navigation (Tab/Enter)
- One-click deletion (no prompts)
- Orientation-responsive cards

**NEW - Render Progress:**
- Real-time elapsed time counter
- Frame-by-frame progress display
- Format: "Rendering 00:15/00:30 | Frame 450/900"
- Updates every 5 frames (performance optimized)

**NEW - Intelligent Filename Parsing:**
- Parses shader names from code comments
- Format: `// ONEMANSHYO Dojo V2_6 - ShaderName`
- Fallback hierarchy: user → parsed → "filename"
- Separate formats for video vs code exports

**UX Improvements:**
- Consistent 400px left panel width (all tabs)
- No auto-tab-switching when loading presets
- Drag-over visual feedback (cyan border)
- Focus tracking for keyboard workflow

**Existing Features (from V2.5.0):**
- Render Duration field for shader-only mode
- Fixed render visual timing with Loop In/Out
- Loop button cyan indicator when active
- Fixed UTF-8 encoding throughout
- 90% file size reduction (1.4MB → 142KB → 199KB with presets)

---

## 4. Development Guidelines

### Token Management (CRITICAL)

**Minimize token usage in all responses:**
- Keep explanations concise and relevant only
- Structure ALL questions to be answerable with simple yes/no or letter/number reference
- NEVER force user to retype information when they can reference a number/letter
- Watch token count and warn user if chat approaching limit

**Question Formatting Rules (MANDATORY):**
- ALL questions use format: 1., 2., 3. for main questions
- ALL options use format: a), b), c) for sub-options
- Structure so user can answer "1.a" or "yes to 2" or "no"
- NEVER use bullet points for questions or options
- NEVER ask open-ended questions requiring paragraph answers when yes/no works

**Example Good Format:**
```
1. Should I add the Song Start field?
   a) Yes, add to audio controls strip
   b) No, different location needed
   c) Skip this feature
```

**Example Bad Format:**
```
"What do you think about adding a Song Start field and where should it go?"
(Forces user to write paragraphs instead of "1.a")
```

### Build Process

1. **Always ask before building** - Never proceed without explicit user confirmation
2. **Wait for user response** - User specifies option (e.g., "do 1.a" or "proceed with 3")
3. **Keep pre-build questions minimal and efficient**

### Incremental Development

- **One task per iteration** (A, B, C, etc.) to avoid breaking
- Test each change independently before moving to next
- Document in iteration notes file for every change
- Version strings updated consistently throughout file

### File Creation (Per Iteration)

**Always provide TWO files:**

1. **Build File:** OMS_Dojo_V2_X_Y_[Letter]_[DescriptiveName].html
   - Example: OMS_Dojo_V2_6_A_PresetsTab.html

2. **Notes File:** OMS_Dojo_V2_X_Y_[Letter]_[DescriptiveName]_Notes.txt
   - Example: OMS_Dojo_V2_6_A_PresetsTab_Notes.txt

**Naming Convention:**
- Use underscores: V2_6_0_A_PresetsTab
- Include descriptive name after letter
- NOT generic: V2_6_0_A.html (missing name)
- NOT periods: V2_6.A (breaks sorting)
- NOT hyphens: V2-6-0-A (breaks sorting)

### File Size Management (CRITICAL)

**Keep HTML file lean - do NOT embed changelog:**
- Changelog: External notes files ONLY
- Historical changelog: Never in HTML comments
- Maximum HTML comment block: Lines 1-35 (version format conventions only)
- Line 36: Close comment with -->

**Why:**
- Embedded changelog causes token limit crashes in Claude
- Notes files per iteration = no need for embedded history
- 500KB+ of text serves no functional purpose in production

**If HTML exceeds 1.8MB, audit for embedded documentation bloat**

### Encoding Issues

- Test Unicode symbols - often corrupt
- Use ASCII-safe characters (dashes, underscores)
- Avoid fancy Unicode box-drawing
- Test dropdowns and separators for UTF-8 corruption

---

## 5. Full Release Deliverables Package

When user requests full deliverables, provide **9 files:**

1. **OMS_Dojo_V2_X_Y.html** - Main application
2. **OMS_Dojo_V2_X_Y_A_Deliverables.txt** - Package inventory
3. **OMS_Dojo_V2_X_Y_B_Complete_Changelog.md** - Version history (references iteration files)
4. **OMS_Dojo_V2_X_Y_C_Developer_Documentation.md** - Technical architecture
5. **OMS_Dojo_V2_X_Y_D_Release_Guide.md** - Executive summary + production workflow
6. **OMS_Dojo_V2_X_Y_E_Claude_ShaderCreation_Guide.md** - Shader creation patterns
7. **OMS_Dojo_V2_X_Y_UserGuide_All.md** - Complete user guide
8. **OMS_Dojo_V2_X_Y_UserGuide_FFmpeg_Video_Guide.md** - FFmpeg guide
9. **OMS_Dojo_V2_X_Y_Z_Project_Instructions.md** - This file

**ZIP Name:** OMS_Dojo_V2_X_Y_Deliverables.zip

**Note:** All iteration notes files (A through X) exist separately and are referenced in changelog.

---

## 6. Communication Guidelines

### DO Use These Terms:
- ✅ "DJing with live reactive visuals"
- ✅ "DJ set with visual element"
- ✅ "Club DJing" or "DJ performance"
- ✅ "Shader loops for DJ sets"

### NEVER Use These Terms:
- ❌ "VJing" or "VJ" (this is DJing, not VJing)
- ❌ "A/V performance" or "audiovisual performance" (too art-world)
- ❌ "Music videos" (videos are silent)

### NEVER Suggest:
- ❌ Business models, pricing, revenue, monetization (for this project)
- ❌ "This could be a business" or "you could sell this"
- ❌ Target markets, customer acquisition, growth strategies
- ❌ Ableton integration beyond Link sync (external workflow, not in-app)
- ❌ Server-side processing (single-file philosophy)
- ❌ TouchDesigner replacement (different use case)
- ❌ Max for Live version (locks out non-Ableton users, kills accessibility)

### Core Philosophy

**"Go wild, design whatever you want in the shader"**
- Encourage creativity
- Provide technical support
- Let users drive creative vision

**"Anyone can be a OneManShYo"**
- Empowering solo creators
- Not building businesses
- Open source for the community

---

## 7. Common Pitfalls to Avoid

**Technical:**
1. Don't confuse Shader Module with Audio Module - separate systems
2. Don't embed changelog in HTML comments - causes token crashes
3. Don't use Unicode symbols - UTF-8 encoding issues
4. Don't make multiple changes per iteration - breaks stability
5. Don't skip iteration notes files - essential documentation
6. Don't use bullet points in questions - use numbered format
7. Don't use vec2 for resolution in uniforms - use separate resolutionX/Y
8. Don't create uniform buffer < 48 bytes - causes WebGPU validation errors

**Communication:**
9. Don't suggest business models for this project
10. Don't use "VJ/VJing" terminology
11. Don't forget Wes has 35 years experience - not a beginner
12. Don't waste tokens repeating yourself - numbered questions, get confirmation, move forward
13. Don't use "A/V performance" terminology
14. Don't explain basics unless asked

---

## 8. When Starting New Version Development

**Before making ANY changes:**

1. Add this Z_Project_Instructions.md to new Claude project
2. Review current version's deliverables package
3. Read through recent iteration notes (last 5-10)
4. Understand current architecture state
5. Ask user about goals for new version
6. Create iteration plan with user approval
7. Proceed incrementally (one iteration at a time)

---

## 9. Technical Reference Quick Access

**WebGPU Pipeline:**
- Fragment shader only (no vertex shader control)
- Uniforms: time, tempo, modA, modB, modC, resolutionX, resolutionY, padding
- Real-time rendering at 60fps
- Canvas captures to MediaRecorder for video export

**Audio Analysis:**
- Web Audio API with AnalyserNode
- FFT Size: 2048 samples
- Frequency Bands: Bass (20-250Hz), Mid (250-4kHz), Treble (4-20kHz)
- Smoothing: 0.8
- Update rate: Per animation frame

**Time Formats:**
- Render Duration: MM:SS.mmm (left-to-right parsing)
- Audio Start/End: MM:SS.mmm (millisecond precision)
- Loop In/Out: bar.beat (1-indexed, 4/4 time)
- Conversion functions: timeStringToSeconds(), barBeatToBeats()

**Export Settings:**
- Container: WebM
- Codec: VP8
- Resolution: 720p, 1080p, 1440p, 2160p
- Frame Rate: 24, 30, 60 FPS
- Orientation: Portrait, Landscape, Square

---

## 10. Version History Context

**V2_6 (Current) - November 2025**
20 iterations (A-U, skipping K) focused on presets system, render progress, and intelligent filename handling.
Major achievements: 
- Complete preset system (Library + User Presets)
- Visual thumbnails with WebGPU rendering
- Real-time render progress tracking
- Drag-and-drop import
- Keyboard navigation workflow

**V2.5.0 - November 2025**
24 iterations (A-X) focused on render accuracy, file optimization, UX improvements.
Major achievement: 90% file size reduction (1.4MB → 142KB)

**V2.4.0 - November 2025**
12 iterations (A-L) implementing manual beatgrid system with visual feedback.
Introduced Song Start/End fields and grid overlay.

**V2.3.0 - November 2025**
10 iterations (A-J) establishing Shader/Audio Module separation.
Removed automatic BPM detection, added manual tempo entry workflow.

---

## 11. Success Metrics (V2_6)

- ✅ Preset system with 5 library shaders
- ✅ User preset import and management
- ✅ LocalStorage persistence
- ✅ Visual thumbnail previews
- ✅ Drag-and-drop import
- ✅ Keyboard navigation (Tab/Enter)
- ✅ Real-time render progress (elapsed time + frame counter)
- ✅ Intelligent filename parsing from code comments
- ✅ One-click deletion (no prompts)
- ✅ WebGPU uniform buffer alignment fixed (48 bytes)
- ✅ WGSL struct layout corrected (separate resolutionX/Y)
- ✅ Consistent left panel width (400px)
- ✅ No auto-tab-switching
- ✅ 20 incremental iterations without system crashes
- ✅ Comprehensive 9-file deliverables package

---

**Document Version:** V2_6  
**Last Updated:** November 2025  
**Next Update:** When V2.7.0 development begins

---

**END OF PROJECT INSTRUCTIONS**
