# OMS Shader Baker Live V1.0.0 - User Guide

**Complete guide for ShaderBakerLive**

---

## Quick Start

1. Open `ShaderBakerLive_V1_0_0.html` in Chrome
2. Open one or more Shader Baker V2.8.0+ tabs
3. Assign Shader Baker instances to tracks (dropdown in track headers)
4. Compile shaders in Baker → visuals appear in Live grid
5. Ready for performance (audio features coming in V1.1+)

---

## Interface Overview

```
┌─────────────────────────────────────────────────────┐
│ OMS Shader Baker Live [V1.0.0]                      │
├─────────────────────────────────────────────────────┤
│ Scenes │ Track 1      │ Track 2      │ Track 3 ... │ ← Headers (fixed)
├─────────────────────────────────────────────────────┤
│   1    │ [Clip Cell]  │ [Clip Cell]  │ [Clip Cell] │
│   2    │ [Clip Cell]  │ [Clip Cell]  │ [Clip Cell] │ ← Scrollable
│  ...   │     ...      │     ...      │     ...     │
├─────────────────────────────────────────────────────┤
│        │ ⊙ dB  L|R    │ ⊙ dB  L|R    │ ...         │ ← Mixer (fixed)
│        │ [M] [S]      │ [M] [S]      │ ...         │
├─────────────────────────────────────────────────────┤
│ Track 1 Waveform: ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬          │
│ Track 2 Waveform: ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬          │
│ Track 3 Waveform: ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬          │
│ Track 4 Waveform: ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬          │
├─────────────────────────────────────────────────────┤
│ Status: Ready                                       │
└─────────────────────────────────────────────────────┘
```

### Sections

**Fixed Header:**
- Product name and version
- Always visible at top

**Performance Grid:**
- Left column: Scene triggers (1-8)
- 4 track columns with clip cells
- Track headers fixed, scenes scroll
- 4:3 landscape aspect ratio cells

**Mixer Section:**
- Fixed at bottom (always accessible)
- Per-track: Mute, Solo, Volume knob, L/R meters
- Left column: Empty spacer

**Waveform Section:**
- 4 horizontal waveform strips
- Viewing only (no controls)
- Placeholders in V1.0.0

**Status Bar:**
- Footer with system messages
- Color states: cyan (normal), magenta (error), orange (info)

---

## Track Headers

### Instance Dropdown

**Location:** Top of each track column  
**Function:** Assign Shader Baker instance to track

**How to Use:**
1. Click dropdown in track header
2. Select Shader Baker instance from list
3. Status updates: "Connected - [Shader Name]"
4. Grid cells in that column show shader visual

**Options:**
- "Not Connected" (default, no assignment)
- Available Shader Baker instances (auto-discovered)
- Can assign same instance to multiple tracks

**Status Indicators:**
- Gray text: Not connected
- Green text: Connected
- Shows shader name when connected

---

## Clip Grid

### Clip Cells

**Appearance:**
- 4:3 landscape aspect ratio
- Gray background when empty ("Empty" text)
- Shader visual as background when assigned
- Landscape thumbnails preserve shader proportions

**Interaction (V1.0.0):**
- Visual display only
- No click functionality yet
- No clip launching yet
- Placeholders for future features

**Future (V1.1+):**
- Click to launch clip
- Load audio files per clip
- Playback triggers

---

## Scene Triggers

### Scene Buttons

**Location:** Left column (Scene 1-8)  
**Function:** Launch entire scene row (future feature)

**Current Behavior (V1.0.0):**
- Click highlights scene (cyan border)
- Updates status bar: "Scene X launched"
- Visual feedback only
- No playback yet

**Future (V1.1+):**
- Launches all clips in row
- Starts playback across tracks
- Professional scene-based performance

---

## Mixer Controls

### Volume Knob (Rotary)

**Appearance:**
- Circular canvas-rendered knob
- Cyan arc shows current level
- White indicator line
- dB label above knob

**How to Use:**
1. Click and hold knob
2. Drag mouse up/down
3. Up = louder, Down = quieter
4. Release to set value

**Range:**
- 0-100 (internal value)
- Displayed as dB: -∞ to 0dB
- 0 = -∞ (muted)
- 100 = 0dB (full volume)

**Current Status (V1.0.0):**
- Visual control only
- No audio effect yet
- UI interaction works

### Mute Button [M]

**Function:** Mute track output  
**Appearance:** Orange when active

**How to Use:**
1. Click [M] button
2. Toggles on/off
3. Orange = muted
4. Gray = unmuted

**Current Status (V1.0.0):**
- Visual state only
- No audio effect yet

### Solo Button [S]

**Function:** Solo track (mute all others)  
**Appearance:** Cyan when active

**How to Use:**
1. Click [S] button
2. Toggles on/off
3. Cyan = soloed
4. Gray = normal

**Current Status (V1.0.0):**
- Visual state only
- No audio effect yet

### L/R Meters

**Appearance:**
- Vertical bars (left and right)
- Green → yellow → red gradient
- Side-by-side layout

**Function:** Show audio levels per channel

**Current Status (V1.0.0):**
- Placeholders only
- No animation yet
- Requires audio engine (V1.1+)

---

## Waveform Section

### Waveform Strips

**Appearance:**
- 4 horizontal strips (Track 1-4)
- Black background
- Canvas placeholders

**Function:** Display audio waveform

**Current Status (V1.0.0):**
- Placeholders only
- No waveforms yet
- Requires audio loading (V1.1+)

---

## Status Bar

### Status Messages

**Location:** Bottom footer  
**Function:** System feedback

**Message Types:**
- "Ready" - Default state (cyan)
- "Scene X launched" - Scene trigger feedback (cyan)
- "Instance discovered" - New Shader Baker found (cyan)
- "Connection failed" - Error state (magenta)
- "Loading..." - Info state (orange)

**Color States:**
- Cyan border/text: Normal/success
- Magenta border/text: Error
- Orange border/text: Info/warning

---

## Keyboard Shortcuts

**None in V1.0.0**  
(Future versions will add keyboard control)

---

## Mouse Controls

### Rotary Knobs
- Click-drag up/down to adjust volume
- Vertical mouse movement
- 0.5dB sensitivity per pixel

### Buttons
- Click to toggle (Mute/Solo)
- Hover for highlight effect

### Scene Triggers
- Click to launch scene (visual feedback)

### Dropdowns
- Click to open instance list
- Click option to assign

---

## Workflow Examples

### Example 1: 4-Shader Visual Grid

**Goal:** Display 4 different shaders in grid

**Steps:**
1. Open ShaderBakerLive
2. Open 4 Shader Baker tabs
3. Load different shader in each:
   - Tab 1: "Bass Tunnel"
   - Tab 2: "Plasma Shader"
   - Tab 3: "Geometric Pattern"
   - Tab 4: "Particle System"
4. Compile each shader in Baker
5. In Live, assign instances to tracks:
   - Track 1 → Bass Tunnel
   - Track 2 → Plasma Shader
   - Track 3 → Geometric Pattern
   - Track 4 → Particle System
6. Result: Grid shows 4 different visuals

**Use Case:**
- Visual preview/selection
- Multi-shader development
- Performance preparation

### Example 2: A/B Comparison

**Goal:** Compare two versions of same shader

**Steps:**
1. Open ShaderBakerLive
2. Open 2 Shader Baker tabs
3. Load version A in Tab 1
4. Load version B in Tab 2
5. Assign in Live:
   - Track 1 → Tab 1 (Version A)
   - Track 2 → Tab 2 (Version B)
6. Compile both
7. Result: Side-by-side visual comparison

**Use Case:**
- Shader development
- Parameter tuning
- Visual testing

### Example 3: Single Shader on Multiple Tracks

**Goal:** Redundancy or multi-output

**Steps:**
1. Open ShaderBakerLive
2. Open 1 Shader Baker tab
3. Load shader, compile
4. Assign in Live:
   - Track 1 → Baker instance
   - Track 2 → Same instance
   - Track 3 → Same instance
5. Result: 3 tracks show identical shader

**Use Case:**
- Redundancy (backup)
- Different mixer settings per track
- Future: Different blend modes per track

---

## Tips & Best Practices

### Instance Management
- Close unused Shader Baker tabs (reduces dropdown clutter)
- Name shaders clearly (first comment line = instance name)
- Use descriptive names: "// Bass Tunnel v3" not just "// Shader"

### Performance Workflow
1. Open Live first
2. Open all Baker tabs needed
3. Assign instances before scene work
4. Verify snapshots appear
5. Check connection status (green)

### Shader Development
- Compile frequently to update Live snapshots
- Keep Baker canvas visible (not minimized)
- Test shaders in Baker before using in Live

### Layout
- Fixed 1200px width - works best at 1920×1080+
- Scroll through scenes while mixer stays accessible
- Track headers always visible while scrolling

---

## Troubleshooting

### Instances Not Appearing in Dropdowns

**Cause:** Shader Baker not broadcasting  
**Fix:** Use Shader Baker V2.8.0-A or later

**Cause:** Different browser or origin  
**Fix:** Open both in same browser, same origin (localhost)

### Snapshots Not Showing

**Cause:** Shader not compiled  
**Fix:** Click Compile button in Shader Baker

**Cause:** Instance not assigned  
**Fix:** Select instance from track dropdown

**Cause:** Baker canvas not rendering  
**Fix:** Check Baker console for WebGPU errors

### Dropdowns Empty

**Cause:** No Shader Baker tabs open  
**Fix:** Open Shader Baker V2.8.0+ in new tab

**Cause:** Broadcasting not working  
**Fix:** Check browser console for BroadcastChannel errors

### Stale Instances

**Symptom:** Instance in dropdown but tab closed  
**Fix:** Wait 30 seconds, Live auto-removes stale instances

---

## Limitations (V1.0.0)

### No Audio Playback
- Mixer controls are visual only
- No audio file loading
- No playback functionality
- Meters don't animate

### No Clip Launching
- Clip cells are visual only
- No click functionality
- No audio triggers

### No Scene Launching
- Scene buttons give visual feedback
- Don't trigger playback
- No clip launching on scene launch

### Fixed Layout
- 1200px width (non-responsive)
- 4 tracks only
- 8 scenes only
- No customization

---

## Coming in V1.1.0+

### Audio Engine
- Audio file loading per clip
- Play/pause per track
- Real-time meter animation
- Volume/mute/solo functionality

### Clip Launching
- Click clip cell to launch
- Audio playback starts
- Visual feedback
- Stop button

### Scene Launching
- Click scene trigger to launch row
- All clips in scene play
- Professional performance workflow

### Master Section
- 5th mixer column
- Master volume
- Master meters
- Global controls

---

## System Requirements

**Browser:**
- Chrome 88+ (recommended)
- Safari 15+
- Firefox 89+
- Edge 88+

**Display:**
- 1920×1080 or higher recommended
- 1200px minimum width

**Integration:**
- OMS Shader Baker V2.8.0+ required
- BroadcastChannel API support
- Same origin (localhost or HTTPS)

---

## Support

**No paid support provided**
- Open source project
- "Ghost protocol" philosophy
- Community-driven

**For Help:**
- Read documentation thoroughly
- Check troubleshooting section
- Review iteration notes for details

---

**End of User Guide**
