# ONEMANSHYO Dojo App V2 — Claude Project Instructions

**Version:** V2  
**Build Date:** November 4, 2025  
**License:** GPL-3.0  
**Purpose:** Development context for AI-assisted work on this codebase

---

## TABLE OF CONTENTS

1. [About Wes Smith](#1-about-wes-smith)
2. [ONEMANSHYO](#2-onemanshyo)
3. [Project Identity](#3-project-identity)
4. [Architecture](#4-architecture)
5. [Critical Terminology](#5-critical-terminology)
6. [Development Guidelines](#6-development-guidelines)
7. [Communication Protocols](#7-communication-protocols)
8. [What Never to Suggest](#8-what-never-to-suggest)

---

## 1. About Wes Smith

### Background

- 35 years in music, technology, and software development
- Electronic music DJ/producer
- Performs as ONEMANSHYO — solo DJ with live reactive visuals
- 500+ music releases; founded Juice Recordings
- Music featured in video games (PlayStation, Xbox)

### Professional Standards

Decades of shipping products and building real systems. Not a beginner — assume
expert-level technical knowledge. No basic explanations, no hand-holding. Expects a
professional development partner.

### Communication Style

- Direct and efficient, token-conscious
- Numbered questions with lettered options
- "yes go" means proceed immediately
- Corrects imprecise terminology immediately
- Expects exact technical language, not approximations

---

## 2. ONEMANSHYO

Wes performs as ONEMANSHYO — a solo DJ/visual artist project:

- DJs audio (mixing tracks, building sets, live performance)
- Runs live reactive visuals simultaneously
- One person controls both audio and visuals

### Critical Distinction: DJing, Not VJing

**Never use "VJ" or "VJing."** This is DJing with live reactive visuals.

- VJing = mixing pre-made video clips, a separate role from the audio DJ
- ONEMANSHYO = audio-first, one person, visuals respond to the music

The workflow is DJ-centric. Tools built for it should reflect that.

### Philosophy

- GPL-3.0, free, open source
- Single-file HTML, no dependencies, no build step
- "Anyone can be a OneManShYo" — no licenses, no complex setup, no crew

---

## 3. Project Identity

**Dojo** is a browser-native tool for authoring audio-visual shader content. It
compiles WGSL fragment shaders, drives them with tempo-synced modulation, renders
via WebGPU, and exports to WebM.

**V2 scope:** shader authoring with tempo-synced modulation. No audio input, no
persistence, no library. Those come later in the V2 series.

### Target Platform

**Chrome on macOS. That's it.**

No cross-browser testing. No mobile. No Firefox fallback. WebGPU is assumed.

---

## 4. Architecture

### Single File

One HTML file — markup, CSS, and JavaScript inline. ~970 lines, ~35KB.

This is not a limitation to work around. It's the design. Portable, archivable,
offline-capable, and the source is readable via View Source.

### Core Systems

| System | Role |
|--------|------|
| WebGPU Pipeline | Device, shader compilation, render pass |
| Modulation Engine | `calculateMod()` — beat-derived Mod A/B/C |
| Timing | Elapsed seconds → beats via tempo |
| Render Loop | Per-frame uniform write and draw |
| Export | Canvas capture → WebM |
| Status | `showStatus()` — the only feedback surface |

### The Shader Contract

Prepended to every user shader automatically:

```wgsl
struct Uniforms {
    time: f32,
    tempo: f32,
    modA: f32,
    modB: f32,
    modC: f32,
    resolution: vec2<f32>,
}
```

**This contract is load-bearing.** Changing it breaks every shader ever written for
the app. Adding a uniform requires three coordinated edits — the struct, the
`Float32Array` in `render()`, and the buffer size — plus attention to `vec2`
alignment padding.

### Timing Model

All motion derives from beat position, never from raw elapsed time:

```javascript
const beat = ((Date.now() - startTime) / 1000.0) * (currentTempo / 60.0);
```

Shaders that animate on `time` directly will drift against music. Beat-based
calculation is the whole point of the tempo system.

---

## 5. Critical Terminology

**Shader** — WGSL fragment shader code, rendered in real time  
**Video** — an exported render; pre-baked, no longer interactive

**Mod A / Mod B / Mod C** — the three modulation sources. Not "modulators,"
not "LFOs." Mod A, B, C.

**Subdivision** — how often a mod cycles, expressed musically (Whole, Half, 1/4,
1/8, 1/16). Stored as a beat multiplier, not a frequency.

**Waveform** — the shape of a mod's cycle. Ten generators.

**Tempo** — BPM. The rate everything derives from.

**Beat** — continuous count from compile time. Not a discrete event.

---

## 6. Development Guidelines

### Before Building Anything

Ask first. Research and discussion need no permission; writing code does.

1. User describes the issue or feature
2. Claude investigates and discusses
3. Claude asks numbered questions with lettered options
4. User says "yes go"
5. Then build

### Iteration Discipline

- One feature per iteration
- New version file per build — never edit in place except for bug fixes
- Test before proceeding
- Surgical changes; don't touch what wasn't asked about

### Scope Checks

| Check | Action |
|-------|--------|
| Touching the uniform contract? | Stop, discuss — it breaks every existing shader |
| Touching `calculateMod()`? | Trace which mods and shaders depend on the behavior |
| Touching the timing model? | Stop — everything derives from it |
| Changes in 10+ locations? | Stop, verify scope first |
| CSS/labels/colors only? | Safe to proceed |

### Documentation Integrity

Never gut existing docs when versioning. Surgical updates only — show prior versus
new content, get approval per deliverable.

---

## 7. Communication Protocols

**Do:**
- Listen to what's actually being asked
- Use numbered questions with lettered options
- Wait for "yes go"
- Use exact technical terminology
- Reference existing documentation instead of restating it

**Don't:**
- Assume intent
- Build without permission
- Explain basics
- Add unrequested features
- Use VJ terminology

---

## 8. What Never to Suggest

**Architecture:**
- Splitting into multiple files
- Adding a build process (webpack, vite, etc.)
- Adding external dependencies
- Adding a framework (React, Vue, etc.)

**Monetization:**
- Paywalls, subscriptions, licensing restrictions

**Workflow:**
- Changing the DJ-centric workflow
- Adding VJ-style features

**Scope:**
- Features that weren't requested

### Why

Single-file means portability, no installation, offline capability. GPL-3.0 means
free and open, permanently. DJ-centric means audio-first, solo-creator workflow.
These aren't preferences to be optimized away.

---

**End of Project Instructions**
