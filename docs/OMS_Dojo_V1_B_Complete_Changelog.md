# OMS Shader Baker Live V1.0.0 - Complete Changelog

**Release Date:** November 2025  
**Development Period:** November 2025  
**Iterations:** A through P (15 iterations, N skipped)

---

## Overview

ShaderBakerLive V1.0.0 represents the foundational UI and communication layer for the future OMS Session performance mixing tool. This release focuses on establishing the visual interface, BroadcastChannel integration with OMS Shader Baker instances, and instance discovery/management workflows.

**Total Development:** 15 iterations  
**Major Features:** Performance grid, hardware mixer controls, instance discovery, canvas snapshots  
**File Size:** ~37KB (single HTML file)

---

## Iteration Summary

### Iteration A: Session View Grid + Audio Decks Layout
**Focus:** Foundational UI structure

**Added:**
- Header with OMS branding (cyan-magenta gradient)
- Session view grid (4 tracks × 8 scenes = 32 clip slots)
- Scene trigger buttons (left column)
- Track headers with connection status
- 4 audio deck placeholders (2×2 grid layout)
- Status bar with version info
- CSS consistent with Shader Baker V2.7.0

**Technical:**
- CSS Grid for session view (5 columns × 9 rows)
- Flexbox for audio decks
- OMS color scheme established

**File Size:** ~12KB

---

### Iteration B: Header Styling Fix + Correct Iteration Labeling
**Focus:** Visual consistency with Shader Baker

**Changed:**
- Product name: "OMS Shader Baker Live" (title case)
- Version span gets gradient (was plain gray)
- Both h1 and version now have cyan-magenta gradient
- Changed audio "decks" to "tracks"
- Horizontal track layout instead of 2×2 grid

**Fixed:**
- Iteration A had wrong header styling
- Iteration B is the corrected first iteration

**File Size:** ~14KB

---

### Iteration C: Footer Status Messaging
**Focus:** Simplified status communication

**Changed:**
- Removed complex status bar (left/right sections)
- Replaced with simple status span (matches Shader Baker exactly)
- Removed redundant version info from footer
- Status color states: cyan (success), magenta (error), orange (info)

**Simplified:**
- Single status message: "Ready"
- Updates on scene launch
- Clean footer matching Shader Baker

**File Size:** ~13KB (reduced)

---

### Iteration D: Track Controls in Vertical Columns
**Focus:** Ableton session view paradigm

**Restructured:**
- Moved track controls to bottom of each vertical column
- Removed load/play/stop buttons from tracks
- Grid structure: Header → 8 clip slots → Track controls (Row 9)
- Bottom section: Waveform viewing only (no controls)

**Added:**
- Volume slider per track (0-100, shows dB)
- Mute button (M) - orange when active
- Solo button (S) - cyan when active
- Audio meter placeholders

**Removed:**
- Audio file loading UI
- Play/Pause/Stop buttons
- Tempo inputs from track section
- Time displays

**File Size:** ~16KB

---

### Iteration E: Fixed Mixer Section
**Focus:** Always-visible mixer controls

**Restructured:**
- Separated performance grid from mixer controls
- Grid in scrollable container (unlimited scenes)
- Mixer section fixed at bottom (always visible)
- Renamed "Session View" to "Performance View"

**Layout:**
- Grid-scroll-container (scrollable)
- Mixer-section (fixed, 140px height)
- Professional DAW workflow

**File Size:** ~17KB

---

### Iteration F: Fixed Track Headers
**Focus:** Excel-style frozen panes

**Restructured:**
- Split grid into fixed headers + scrollable scenes
- Corner + track headers fixed (always visible)
- Only scene rows scroll
- Like spreadsheet frozen panes

**Sections:**
1. Fixed headers (corner + 4 track headers)
2. Scrollable scenes (8 scene rows)
3. Fixed mixer (4 track controls)

**File Size:** ~17KB

---

### Iteration G: Hardware Mixer Controls
**Focus:** Rotary knobs + vertical meters

**Replaced:**
- Horizontal sliders → Rotary knobs (canvas-based)
- Horizontal meters → Vertical L/R stereo bars

**Rotary Knob:**
- 60×60px canvas rendering
- Arc indicator shows volume (cyan)
- White center line reference
- Click-drag interaction (vertical movement)
- 0.5dB per pixel sensitivity

**Vertical Meters:**
- Side-by-side L/R bars
- 8px width × 50px height each
- Gradient: green → yellow → red
- LED-style hardware aesthetic

**Philosophy:** Hardware mixer paradigm (UAD/URI style) instead of DAW

**File Size:** ~18KB

---

### Iteration H: Knob Left + Full-Height Meters Right
**Focus:** Better space utilization

**Restructured:**
- Horizontal layout: Knob left, meters right
- Meters now flex-fill (use full mixer height ~100px)
- Was 50px, now ~100px (2x visual range)

**Layout:**
```
Row 1: [M] [S] -15 dB
Row 2: ⊙  |  ┃ ┃
       Knob | L R Meters
```

**Benefits:**
- More prominent meters
- Easier to read at a glance
- Better use of vertical space
- Hardware mixer aesthetic

**File Size:** ~18KB

---

### Iteration I: Instance Discovery + Manual Mapping
**Focus:** BroadcastChannel integration

**Added:**
- Created 'oms-live-channel' BroadcastChannel
- Listens for Shader Baker announcements
- Stores discovered instances in Map
- Dropdown selector per track header
- Manual instance-to-track assignment

**BroadcastChannel:**
- Message type: 'announce' (from Shader Baker)
- Instance data: {id, name, lastSeen}
- Heartbeat timeout: 30 seconds
- Automatic stale instance cleanup

**Track Assignment:**
- User selects instance from dropdown
- Multiple tracks can use same instance
- Status updates: "Connected - [Shader Name]"
- Green indicator when connected

**File Size:** ~19KB

---

### Iteration J: Canvas Snapshots Display
**Focus:** Visual preview in grid

**Added:**
- Listener for 'snapshot' message type
- Stores base64 canvas image per instance
- Displays snapshots as background-image in clip slots
- Updates on track assignment or new snapshot

**Snapshot Integration:**
- Static thumbnails (on-compile, not video)
- JPEG format, 80% quality (~50-100KB)
- Applied to all cells in assigned track columns
- Real-time visual feedback

**Message Format:**
```javascript
{
  type: 'snapshot',
  id: 'sb-abc123',
  imageData: 'data:image/jpeg;base64,...',
  timestamp: 1699564800000
}
```

**File Size:** ~20KB

---

### Iteration K: Landscape Grid Cells
**Focus:** Proper aspect ratio

**Changed:**
- Clip slots from square to landscape
- aspect-ratio: 4 / 3 (matches Shader Baker canvas)
- height: 80px (width auto-calculated ~106px)

**Benefits:**
- Circular shaders remain circular
- No distortion
- Professional appearance
- Matches source canvas proportions

**File Size:** ~20KB

---

### Iteration L: Compact Grid Layout
**Focus:** Tighter spacing

**Reduced:**
- Cell height: 80px → 60px (25% reduction)
- Grid padding: 20px → 10px (50% reduction)

**Benefits:**
- More cells visible at once
- Less scrolling needed
- Better use of screen space
- Professional tight grid

**File Size:** ~20KB

---

### Iteration M: Fixed Hardware Layout
**Focus:** Non-responsive, centered design

**Changed:**
- Container width: Fixed 1200px (locked)
- No responsive stretching
- Centered horizontally on screen
- Like physical hardware mixer

**Philosophy:**
- Same size every time
- Predictable dimensions
- Muscle memory works
- Hardware consistency

**File Size:** ~20KB

---

### Iteration N: (Skipped)
*Not in development sequence*

---

### Iteration O: Zero Gaps Layout
**Focus:** Completely flush design

**Eliminated:**
- All grid gaps: 0 everywhere
- All padding (except minimal mixer vertical)
- All internal borders
- Performance view border-bottom

**Height Fix:**
- main-container: calc(100vh - 50px) matches header exactly

**Philosophy:**
- Hardware mixer density
- No wasted space
- Professional industrial aesthetic
- Maximizes usable space

**File Size:** ~37KB

---

### Iteration P: Mixer Divider + Left Column Box
**Focus:** Visual separation and consistency

**Added:**
- border-top: 2px solid #444 to mixer-section
- Styled mixer-spacer to match track boxes
- Background: #2a2a2a
- Border: 1px solid #444
- Border-radius: 3px

**Benefits:**
- Clear visual separation between grid and mixer
- Left column visually consistent
- Professional appearance

**File Size:** ~37KB

---

## Feature Timeline

### UI Foundation (A-F)
- Performance grid established
- Mixer controls positioned
- Fixed headers + scrollable scenes
- Professional layout complete

### Hardware Aesthetics (G-H)
- Rotary knobs replace sliders
- Vertical L/R stereo meters
- Hardware mixer paradigm

### Communication Layer (I-J)
- BroadcastChannel integration
- Instance discovery
- Canvas snapshot display
- Visual feedback complete

### Layout Refinement (K-P)
- Landscape aspect ratio (4:3)
- Compact spacing
- Fixed 1200px width
- Zero gaps flush design
- Visual polish

---

## Technical Achievements

**Architecture:**
- Single HTML file (~37KB)
- Zero external dependencies
- BroadcastChannel communication
- Instance discovery and mapping
- Canvas snapshot integration

**UI/UX:**
- Fixed 1200px hardware layout
- Scrollable grid with frozen headers
- Always-visible mixer section
- Rotary controls + vertical meters
- 4:3 landscape thumbnails
- Completely flush zero-gap design

**Communication:**
- Receives instance announcements
- Receives canvas snapshots
- Manual track assignment
- Multi-instance support
- Heartbeat-based instance management

---

## What Works in V1.0.0

✓ Performance grid UI (4 tracks × 8 scenes)  
✓ Scene trigger buttons (UI only)  
✓ Fixed track headers (always visible)  
✓ Scrollable scene rows  
✓ Fixed mixer section (always accessible)  
✓ Rotary volume knobs (UI only)  
✓ Mute/Solo buttons (UI only)  
✓ Vertical L/R meters (placeholders)  
✓ BroadcastChannel instance discovery  
✓ Manual instance-to-track assignment  
✓ Canvas snapshot display in grid  
✓ Status messaging system  
✓ Fixed 1200px hardware layout  
✓ Professional aesthetics  

---

## What Doesn't Work Yet

❌ No audio playback engine  
❌ No FFT data broadcasting/receiving  
❌ No clip launching functionality  
❌ No scene launching playback  
❌ Meters are placeholders (no animation)  
❌ No MIDI control integration  
❌ No Ableton Link sync  
❌ No .oms file support  
❌ Waveforms are placeholders  

---

## Detailed Iteration Notes

Complete technical notes for each iteration are available in separate files:
- ShaderBakerLive_V1_0_0_A_SessionView_Notes.txt
- ShaderBakerLive_V1_0_0_B_SessionView_Notes.txt
- ShaderBakerLive_V1_0_0_C_StatusFooter_Notes.txt
- ShaderBakerLive_V1_0_0_D_VerticalControls_Notes.txt
- ShaderBakerLive_V1_0_0_E_FixedMixer_Notes.txt
- ShaderBakerLive_V1_0_0_F_FixedHeaders_Notes.txt
- ShaderBakerLive_V1_0_0_G_HardwareMixerControls_Notes.txt
- ShaderBakerLive_V1_0_0_H_KnobLeftMetersRight_Notes.txt
- ShaderBakerLive_V1_0_0_I_InstanceDiscovery_Notes.txt
- ShaderBakerLive_V1_0_0_J_CanvasSnapshots_Notes.txt
- ShaderBakerLive_V1_0_0_K_LandscapeGrid_Notes.txt
- ShaderBakerLive_V1_0_0_L_CompactGrid_Notes.txt
- ShaderBakerLive_V1_0_0_M_FixedLayout_Notes.txt
- ShaderBakerLive_V1_0_0_O_NoGaps_Notes.txt
- ShaderBakerLive_V1_0_0_P_MixerDivider_Notes.txt

---

## Next Version (V1.1.0+)

**Planned Features:**
- Audio engine integration
- FFT data broadcasting to Shader Baker
- Real-time meter animation
- Clip launching functionality
- Scene launching playback
- Master section (5th column)
- Audio file loading per clip
- Tempo synchronization

---

**End of Changelog**
