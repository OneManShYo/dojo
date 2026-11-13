# OMS SHADER BAKER V4.1.0 - CLAUDE PROJECT INSTRUCTIONS

**Version:** V4.1.0  
**Date:** November 27, 2025  
**Purpose:** Complete project context for Claude AI development

---

## CRITICAL NOTES

After 500+ iterations and reaching V4.1.0, the project has matured significantly. Claude must:
- Think through decisions carefully before implementing
- Not rush ahead with changes
- Understand the problem, look at code, discuss, and confirm before development
- Pay close attention to architecture, consistency, and professional standards
- Remember: Shader Baker is no longer a toy - it's approaching production software

**ALWAYS deliver both HTML and iteration notes files when building.**

---

## TABLE OF CONTENTS

1. [About Wes Smith - Creator Context](#about-wes-smith---creator-context)
2. [Current State - V4.1.0](#current-state---v410)
3. [Development Protocols](#development-protocols)
4. [Communication Guidelines](#communication-guidelines)
5. [Key Architectural Principles](#key-architectural-principles)
6. [Version History Summary](#version-history-summary)
7. [Critical Terminology](#critical-terminology)
8. [What Never to Suggest](#what-never-to-suggest)

---

## ABOUT WES SMITH - CREATOR CONTEXT

### Professional Background

- 35 years in music, technology, and software development
- Electronic music DJ/producer (ONEMANSHYO)
- 500+ music releases, 7,000+ days in Beatport charts
- Founded Juice Recordings label
- Based in San Diego, CA

### Development Style

**Communication:**
- Direct and efficient (token-conscious)
- Numbered questions with lettered options
- "yes go" means proceed immediately
- Expects exact technical language
- Provides clear specifications upfront

**Standards:**
- Decades of shipping real products
- NOT a beginner - assume expert knowledge
- Expects professional development partnership
- Values proven workflows over theory

---

## CURRENT STATE - V4.1.0

### Production Readiness: ~85%

**V4.1.0 Complete (35 sub-iterations across A + B):**
✅ **Track/Master Grid Alignment** - POC-proven architecture  
✅ **Tempo-Based Pixel Formula** - Same formula everywhere  
✅ **CSS Positioning Fix** - position: absolute (not relative)  
✅ **Canvas-Drawn Playhead** - Same coordinate system as grid  
✅ **M/T/W Visibility Toggles** - Master/Track/Waveform layers  
✅ **Recalibrated Zoom** - 0.5x to 8x  
✅ **Dynamic Playhead Position** - containerWidth / 2  

### What Works (100%)

- Audio engine with quantization
- Multi-track playback and mixing
- File system (.oms, .omsp)
- Keyboard navigation
- Enter key responsiveness
- Grid alignment (FIXED!)
- Shader compositing

### What's Very Good (90-95%)

- Timeline reactivity system
- Visual synchronization
- Beat grid alignment
- Shader modulation

### Still Improving (80%)

- Edge case handling
- Final visual polish

---

## DEVELOPMENT PROTOCOLS

### Iteration System

**Alphabetical Progression:**
- A, B, C... Z, then ZA, ZB, ZC... ZZ
- Never skip letters
- One feature per iteration
- Maximum 52 iterations (A-Z + ZA-ZZ)

### Sub-Iteration System

**For complex problems:**
- V4_1_0_C_sub1, sub2, sub3... sub35
- Each sub is cumulative (contains all previous)
- Both HTML and MD files get sub numbers
- Final working version renamed without sub

### Build Protocol

**NEVER build without asking:**
- Research/investigate = OK
- Discuss/analyze = OK
- Building code = ALWAYS ASK FIRST

**Pattern:**
1. User describes issue
2. Claude investigates
3. Claude asks: "Ready to build? yes go?"
4. User: "yes go"
5. THEN Claude builds

### Deliverables

**Always provide BOTH:**
- HTML file with updated version
- Markdown notes describing changes
- Update app header version in HTML

---

## COMMUNICATION GUIDELINES

### Token Efficiency

- Use numbered questions with lettered options
- Reference docs instead of repeating
- Get to the point quickly
- Wait for "yes go" before implementing

### Problem Solving

- Be surgical (don't change unrelated things)
- Find root causes, not band-aids
- One issue at a time
- Document everything

---

## KEY ARCHITECTURAL PRINCIPLES

### Master Clock System

**Single Timeline:**
- `audioContext.currentTime` is ultimate reference
- All timing derives from this
- SessionStartTime set once per session
- Beat-based calculations prevent drift

### Tempo-Based Pixel Formula (V4.1.0 - POC-Proven)

**Core Formula:**
```javascript
const BASE_PIXELS_PER_SECOND = 6.25;
const BEATS_PER_BAR = 4;

function getPixelsPerBeat() {
    const masterBPM = gridState.masterTempo || 120;
    const secondsPerBeat = 60 / masterBPM;
    return BASE_PIXELS_PER_SECOND * secondsPerBeat / (gridState.waveformZoom || 1.0);
}

function getPixelsPerBar() {
    return getPixelsPerBeat() * BEATS_PER_BAR;
}
```

**Golden Rule:** Same formula used EVERYWHERE = perfect alignment.

### CSS Positioning Model (V4.1.0)

**Container:**
```css
.waveform-canvas-container {
    position: relative;
    overflow: hidden;
    height: 100%;
    /* NO display: flex */
}
```

**Canvas:**
```css
.waveform-canvas {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
}
```

**Why:** `position: absolute` + translateX = predictable transforms from container origin (0,0).

### Layer Architecture

| Layer | Behavior | Drawing |
|-------|----------|---------|
| Master Grid | FIXED | Bar 1 at playheadX, extends left/right |
| Track Grid | SCROLLS | Drawn at bar positions, translateX moves |
| Waveform | SCROLLS | Blue amplitude bars, same canvas as TG |
| Playhead | FIXED | Magenta line on MG canvas at playheadX |

### Two-Layer Modulator System

**Layer 1: Timeline Data**
- Pre-analyzed FFT arrays
- Stored in container
- Gains already applied

**Layer 2: Waveform**
- Calculated live each frame
- Based on elapsed beats
- Subdivision controls rate

**Final:** `modulator = timeline × waveform`

### Separation of Domains

- **Shader Domain:** Visual controls (opacity, shader M/S)
- **Audio Domain:** Audio controls (volume, audio M/S)
- **Never mix concerns** in functions

---

## VERSION HISTORY SUMMARY

### V3.0.0 → V3.9.0
- LIVE tab creation and integration
- Perfect beat grid synchronization
- Dual-domain mixer
- Four-track shader compositing
- Keyboard control system
- Pop-out window

### V4.0.0 Major Changes
- **A:** Timeline reactivity system
- **B:** Carousel clock logic
- **C:** Grid alignment experiments (83 subs!)
- **D:** Critical production fixes (58 subs)

### V4.1.0 Major Changes (Current)
- **A:** Grid alignment investigation (17 subs) + POC
- **B:** POC architecture port (18 subs)

### Key Lessons from V4.1.0

1. POC first when debugging complex alignment
2. CSS position: relative vs absolute fundamentally different
3. Same formula everywhere = alignment
4. Canvas-drawn playhead guarantees coordinate match
5. Don't mix positioning models (bar1X matters)

---

## CRITICAL TERMINOLOGY

### DJ vs VJ

**NEVER say VJ or VJing**
- This is "DJing with live reactive visuals"
- Audio-first workflow, visuals respond
- ONEMANSHYO = DJ with integrated visuals

### Tempo vs BPM

- **Shader Tempo:** Visual animation speed
- **Audio BPM:** Track tempo for analysis
- **Master Tempo:** LIVE tab global BPM
- These are independent values

### Master Clock vs Session

- **Master Clock:** Browser's audio timeline (never stops)
- **Session Start:** When first track launches
- **Session Time:** Relative to session start

### Pixels Per Beat/Bar

- **pixelsPerBeat:** BASE × (60 / masterBPM) / zoom
- **pixelsPerBar:** pixelsPerBeat × 4
- **Golden Rule:** Same formula in ALL places

---

## WHAT NEVER TO SUGGEST

### Architecture Changes ❌
- Split into multiple files
- Add build process
- Add frameworks
- Change single-file philosophy

### Monetization ❌
- Add paywalls
- Add subscriptions
- Change GPL-3.0 license

### Workflow Changes ❌
- Change DJ-centric workflow
- Add VJ-style features
- Merge creation and playback engines

### Feature Creep ❌
- Add features not requested
- Over-engineer solutions
- Anticipate future needs

---

## CURRENT FOCUS AREAS

### Path to V1.0

**Remaining Work:**
1. Final edge case testing (1-2 days)
2. Documentation polish (0.5 day)
3. No more hackathon fixes

**Timeline:** 2 days to production V1.0

### Testing Priorities

- Multi-track synchronization
- Tempo change handling
- Extended performance sessions
- Edge case documentation

---

## DEVELOPMENT GUIDELINES

### Code Quality

- Surgical changes only
- Document reasoning
- Maintain existing patterns
- Test thoroughly before delivery

### V4.1.0 Achievements to Preserve

**NEVER break these:**
- Tempo-based pixel formula
- CSS position: absolute on canvas
- Canvas-drawn playhead
- Same formula everywhere principle

---

## FILE NAMING CONVENTIONS

**Development:**
- `OMS_ShaderBaker_V4_1_0_C.html`
- `OMS_ShaderBaker_V4_1_0_C_DescriptiveName.md`

**Sub-iterations:**
- `OMS_ShaderBaker_V4_1_0_C_sub1.html`
- `OMS_ShaderBaker_V4_1_0_C_sub1_DescriptiveName.md`

**Production:**
- `OMS_ShaderBaker_V4_1_0.html` (no letter)

**Use underscores, not periods:** V4_1_0 (NOT V4.1.0)

---

## REMEMBER

1. Ask before building
2. Wait for "yes go"
3. Use exact terminology
4. One feature per iteration
5. Be surgical with changes
6. Document everything
7. Update version in app header
8. Deliver both HTML and notes
9. Test before delivery
10. Respect the architecture
11. **Same formula everywhere = alignment**
12. **CSS position: absolute, not relative**

---

**END OF CLAUDE PROJECT INSTRUCTIONS**

Version: V4.1.0
Date: November 27, 2025
Total Iterations: 500+
Production Readiness: ~85%
License: GPL-3.0
