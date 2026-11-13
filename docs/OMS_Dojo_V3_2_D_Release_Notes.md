# OMS SHADER BAKER V3.2.0 - RELEASE NOTES

**Version:** V3.2.0  
**Release Date:** November 2025  
**Type:** Major Feature Release

---

## WHAT'S NEW

V3.2.0 transforms the LIVE tab into a professional DJ performance tool with complete visual timing reference. The beat grid system provides the same visual feedback you get in Serato, Rekordbox, or Traktor - bars, phrase markers, and perfect sync across all tracks.

**Major Additions:**
- **Beat Grid Visualization** - Bars, phrase markers, bar numbers
- **Master Tempo Control** - Time-stretch all tracks to one tempo
- **Synced Scrolling** - All waveforms move together (Serato-style)
- **Complete Audio Data** - Timing preserved in .oms files
- **Bar-Style Waveforms** - Optimized for mixing workflow

**Current State:**
- LIVE tab is **production-ready** for DJ performances with visual timing
- Complete beat grid with phrase structure
- Professional tempo synchronization
- All V3.1.0 features fully preserved

---

## THE BEAT GRID SYSTEM

### Visual Timing Reference

**What You See:**
- **Bar Lines** (cyan, thin) - Every bar boundary
- **Phrase Markers** (magenta, thick) - Every 4 bars (drops, breaks, builds)
- **Bar Numbers** (white) - 1, 5, 9, 13, 17... (navigation markers)
- **Audio Boundaries** (slime green) - Start and end of musical content
- **Info Overlay** (magenta) - Container name + original BPM

**Example:**
```
Bar 1  |  Bar 2  |  Bar 3  |  Bar 4  ← magenta phrase marker
Bar 5  |  Bar 6  |  Bar 7  |  Bar 8  ← magenta phrase marker
Bar 9  |  Bar 10 |  Bar 11 |  Bar 12 ← magenta phrase marker
```

### Master Tempo Synchronization

**How It Works:**
- Set master tempo (default 120 BPM)
- All tracks time-stretch to match
- Beat grids align vertically across tracks
- Waveforms scroll together at same speed
- Perfect sync like Serato/Rekordbox sync mode

**Example:**
- Track 1: 124 BPM → plays at 120 BPM (3.2% slower)
- Track 2: 128 BPM → plays at 120 BPM (6.25% slower)
- Track 3: 115 BPM → plays at 120 BPM (4.3% faster)
- Result: All bars line up, all waveforms move together

### Bar-Style Waveforms

**Optimized for Mixing:**
- Cleaner bars showing song structure
- RMS-based amplitude (not detailed transients)
- Easy to see drops, breaks, builds at a glance
- Better for quick mix decisions

**Why Not Detailed Waveforms:**
- DJs need structure, not transient detail
- Bar-style matches Serato/Rekordbox/Traktor
- Cleaner visual for performance workflow

---

## HOW TO USE (V3.2.0)

### Quick Start - Beat Grid

**1. Prepare Containers**
   - Open container in Controls tab
   - Ensure audio loaded with BPM set
   - Click "Update" to save timing data
   - Repeat for all containers you'll use

**2. Load LIVE Tab**
   - Go to LIVE tab
   - Drag containers from left to grid cells
   - Beat grids render automatically

**3. Set Master Tempo**
   - Master column header (LIVE tab)
   - Default: 120 BPM
   - Keyboard: Up/Down ±0.1, Left/Right ±5
   - All tracks time-stretch to this tempo

**4. Play and Mix**
   - Click cells to play audio
   - Watch waveforms scroll together
   - Beat grids stay aligned
   - Mix on phrase markers (magenta lines)

### Master Tempo Control

**Location:** Master column header (LIVE tab)

**Controls:**
- Click field, type new tempo (60-200 BPM)
- Up/Down arrows: ±0.1 BPM (fine tune)
- Left/Right arrows: ±5 BPM (coarse adjust)
- No spinner arrows (clean field)

**What It Does:**
- Time-stretches all playing tracks
- Updates beat grid visual scaling
- Maintains perfect sync
- Auto-saves to project

**Use Cases:**
- Match tempo to your mixing style
- Speed up/slow down entire set
- Test mixes at different tempos
- Find sweet spot for track combinations

### Reading the Beat Grid

**Bar Lines (Cyan):**
- Every bar boundary
- Use for precise alignment
- See exactly where beats land

**Phrase Markers (Magenta):**
- Every 4 bars
- Mark drops, breaks, builds
- Mix on these lines for phrase-matched transitions
- "Wait for the magenta" = wait for phrase boundary

**Bar Numbers (White):**
- 1, 5, 9, 13, 17, 21...
- Quick navigation reference
- "I'm at bar 17" = easy communication

**Audio Boundaries (Slime Green):**
- Start: Where musical content begins
- End: Where musical content ends
- Ignore intro/outro
- Focus on the "meat" of the track

---

## WHAT WORKS IN V3.2.0

### All V3.1.0 Features ✅

**LIVE Tab:**
- Multi-track audio playback (4 tracks)
- Mixer controls (volume, mute, solo, meters)
- Real waveforms with scrolling
- Project States (.omsp files)
- Keyboard navigation
- Shader thumbnails in cells

### NEW: Beat Grid System ✅

**Visual Elements:**
- Bar lines every bar
- Phrase markers every 4 bars
- Bar numbers for navigation
- Audio boundaries (start/end)
- Info overlay (name + BPM)
- All align vertically across tracks

**Master Tempo:**
- Control in Master header
- Range: 60-200 BPM
- Keyboard shortcuts
- Time-stretches all tracks
- Visual scaling matches audio
- GPU-accelerated transforms

**Synced Scrolling:**
- All waveforms move at same speed
- Beat grids stay aligned
- Cue points move together
- Like Serato/Rekordbox sync mode
- See builds/drops approaching together

### NEW: Audio Data Storage ✅

**Complete Timing:**
- audioPitch (playback speed)
- audioStartTime (where music begins)
- audioEndTime (where music ends)
- audioLoopIn/Out (loop points)
- Saved to .oms files
- Update button preserves all data

### NEW: UX Improvements ✅

**Auto LIBRARY Picker:**
- First-time folder selection automatic
- No confusing empty states
- Smooth experience

**Tempo Keyboard:**
- Fine tune: Up/Down ±0.1 BPM
- Coarse adjust: Left/Right ±5 BPM
- Clean field (no spinner arrows)

---

## WHAT DOESN'T WORK YET

### LIVE Tab (Planned V3.3.0+) ❌

**Performance:**
- Scene launching (rows don't trigger together)
- Waveform buttons (Q, |<, |>, <<, >> are visual only)
- Crossfading between cells
- Quantized launching (snap to bars)

**Visual:**
- Shader rendering in cells (thumbnails only)
- Master output compositing
- Real-time animated shaders

**Control:**
- MIDI integration
- Ableton Link sync
- Hardware controllers

---

## MIGRATION FROM V3.1.0

### Seamless Upgrade

**Nothing Changes:**
- All V3.1.0 features identical
- Same LIBRARY folder
- Same keyboard shortcuts
- Same workflows

**What's New:**
- Beat grid in LIVE tab
- Master tempo control
- Synced scrolling
- Audio timing storage

**No Breaking Changes:**
- Old containers work fine
- Missing timing data uses defaults
- No re-setup required

### Enable Beat Grid for Old Containers

**Quick Fix:**
1. Load container in Controls tab
2. Check Audio Tempo field has BPM
3. Click "Update" button
4. Load to LIVE tab
5. Beat grid appears

**What Gets Saved:**
- Original BPM (for beat grid spacing)
- Start/End times (for Bar 1 position)
- Pitch adjustment (playback speed)
- Loop points (if set)

**One-Time:** Update each container once, then beat grid always works

---

## BROWSER REQUIREMENTS

**Minimum:**
- Chrome 88+ (recommended)
- Safari 15.2+ (Safari Technology Preview for WebGPU)
- Edge 88+

**Required:**
- WebGPU (shader rendering + thumbnails)
- Web Audio API (playback + time-stretching + FFT)
- File System Access API (LIBRARY folder)
- CSS Transforms (beat grid scaling)
- Canvas 2D (beat grid drawing)

**Firefox:** Partial support (no File System Access API)

---

## KNOWN LIMITATIONS

### Beat Grid Requirements

**Must Have:**
- Audio file in LIBRARY folder
- audioBPM field set
- audioStartTime/audioEndTime saved
- Container saved/updated in V3.2.0

**Without These:**
- Beat grid won't appear
- Audio plays but no visual timing
- Fix: Load + Update in Controls tab

### Visual vs Audio

**Beat Grid Shows:**
- Original BPM structure (how track was made)
- Example: 128 BPM track shows bars at 128 BPM spacing

**Audio Plays:**
- At master tempo (time-stretched)
- Example: 128 BPM track plays at 120 BPM master

**Visual Scaling:**
- CSS transforms adjust spacing
- Beat grid visually matches playback tempo
- Both sync perfectly

---

## PERFORMANCE

**No Impact on Existing:**
- Same shader rendering speed
- Same video export speed
- Same LIBRARY performance

**LIVE Tab:**
- Beat grid: GPU-accelerated (CSS transforms)
- Waveform scaling: GPU-accelerated
- Scroll: 60fps smooth
- No canvas redrawing for tempo changes
- Instant visual updates

**Memory:**
- Beat grid: negligible (~500KB for overlays)
- Audio: ~50MB per track
- Shader thumbnails: ~2MB for 32 cells
- Total: ~300MB with 4 tracks

---

## TROUBLESHOOTING

### Beat Grid Not Showing

**Cause:** Container missing timing data  
**Fix:** Controls tab → Load container → Update button

**Cause:** Old .oms file  
**Fix:** Load + Update to add timing fields

**Cause:** Audio BPM not set  
**Fix:** Set "Audio Tempo" field, click Update

### Beat Grids Not Aligned

**Cause:** Should be fixed in V3.2.0  
**Fix:** Hard refresh browser (Ctrl+Shift+R)

**Cause:** Different audioStartTime values  
**Fix:** Normal - each track's Bar 1 aligns at playhead correctly

### Waveforms Not Scrolling Together

**Cause:** Should be fixed in V3.2.0  
**Fix:** Ensure using production release (not old dev build)

### Master Tempo Not Working

**Cause:** Track not playing  
**Fix:** Master tempo only affects playing tracks

**Cause:** Container missing audioBPM  
**Fix:** Set tempo, Update container

---

## NEW WORKFLOWS

### DJ Set Preparation

**Scenario:** Prepare 10-track set with visual timing

**Workflow:**
1. Create all shaders in Code tab
2. Save as containers with audio
3. Set BPM for each track
4. Click Update for each container
5. Go to LIVE tab
6. Load containers into grid (10+ tracks across scenes)
7. Set master tempo (e.g., 120 BPM)
8. Save as project state

**Performance:**
1. Load project state
2. All containers load with beat grids
3. Click cells to play
4. Mix on phrase markers (magenta)
5. Waveforms scroll together
6. See builds/drops approaching
7. Professional DJ set with visual timing

### Tempo Experimentation

**Scenario:** Find best tempo for track combination

**Workflow:**
1. Load 2-3 tracks to LIVE grid
2. Play tracks together
3. Adjust master tempo with keyboard
4. Up/Down for fine tuning (±0.1 BPM)
5. Left/Right for coarse (±5 BPM)
6. Watch beat grids align/misalign
7. Find sweet spot where everything locks
8. Save tempo in project state

### Structure Analysis

**Scenario:** Learn track structure before mixing

**Workflow:**
1. Load track to LIVE grid cell
2. Click cell to play
3. Watch beat grid scroll:
   - Cyan bars = individual beats
   - Magenta = phrase boundaries (4 bars)
   - Bar numbers = navigation markers
4. Note where drops happen
5. Note where breaks happen
6. Plan mix points on phrase markers
7. Confident mixing during performance

---

## GETTING HELP

**Documentation:**
- F_UserGuide_All.md (comprehensive)
- C_Developer_Documentation.md (technical)
- B_Complete_Changelog.md (all 31 iterations)

**Website:**
https://www.itswessmithyo.com/

**Support:**
- No paid support (open source)
- "Ghost protocol" release
- Documentation-first

---

## SUMMARY

V3.2.0 delivers professional DJ visual timing:

**Ready Now:**
- Beat grid visualization ✅
- Master tempo control ✅
- Synced scrolling ✅
- Complete timing data ✅
- Bar-style waveforms ✅

**Coming Soon (V3.3.0+):**
- Scene launching
- Quantized triggers
- Crossfading
- MIDI control
- Ableton Link

**The Vision:**
"Anyone can be a OneManShYo" - solo DJ with live reactive visuals, professional timing reference, all in browser, free and open source.

---

**Welcome to V3.2.0! Your waveforms now speak the universal language of bars and beats.**
